**Pilha Java para Arquitetura de Referência - versão 2.x**

# Leituras Recomendadas

- [Eventos & Mensageria entre aplicações](https://git.capes.gov.br/dti/orientacoes-gerais/guia/blob/master/arquitetura/definicoes/eventos/README.md)
- [ AMQP 0-9-1 Model Explained - RabbitMQ](https://www.rabbitmq.com/tutorials/amqp-concepts.html)

# Introdução

Para comunicação entre aplicações (ou entre instâncias de uma aplicação), é recomendado o estabelecimento de um mecanismo de mensageria. A pilha Java para a arquitetura de referência adere as demais indicações da arquitetura nesse aspecto utilizando AMQP e o suporte que o Spring Framework e o Spring Boot dão a essa tecnologia.

# Dependências e Versionamentos

- Especificação AMQP: `0.9.1`
- Spring AMQP: `2.3.10`

> ## JMS[^jms] & JTA[^jta]
> Apesar de ser possível utilizar o RAbbitMQ como JMS Provider, analisamos e concluímos que essa possibilidade só traria complicações desnecessárias. **Solicitamos que as aplicações façam uso de AMQP como protocolo de mensageria entre aplicações _e não tentem utilizar JMS_**.
> Além de abdicar de JMS, também queremos destacar que estamos abdicando de JTA. Não havia suporte a XA no plugin JMS do RabbitMQ[^rabbitmq-jms-no-xa], o que só reforça nosso entendimento de não usar XA e JTA por tabela.

# Tutoriais

Os tutoriais a seguir cobrem alguns dos aspectos de implementação.

## Maven

Adicionar no `pom.xml` a dependência `br.gov.capes.cgs.narq:mensageria-starter` na versão adequada:

```xml
<dependency>
  <groupId>br.gov.capes.cgs.narq</groupId>
  <artifactId>mensageria-starter</artifactId>
  <version>${mensageria-starter.version}</version>
</dependency>
```

A propriedade `${mensageria-starter.version}` deve ser definida criando a tag `<mensageria-starter.version>` na tag `<properties>` do `pom.xml`. O valor dever ser o valor da versão da pilha[^obs-versao-no-archetype].

## Spring Boot

É necessário configurar a localização e credenciais do usuário de aplicação.

### DHT

Há apenas um cluster de RabbitMQ como _Message Broker_ para DHT. A separação por "ambiente"/_stage_ é feita através de _Virtual Hosts_ da ferramenta[^rabbitmq-vhosts].

Exemplo - `application.yml`[^use-configmap]
```yaml
spring:
   rabbitmq:
      host: rabbitmq.hom.capes.gov.br
      virtual-host: des
      ssl.enabled: false
      username: siglaaplicacao
      password: **************
```

Os VHosts de DHT são:

- `des`
- `hom`
- `teste`

### Produção

Exemplo - `application.yml`[^use-configmap]
```yaml
spring:
   rabbitmq:
      host: rabbitmq.capes.gov.br
      ssl.enabled: false
      username: siglaaplicacao
      password: **************
```

## Java

O desenho da composição de _Exchanges_, Filas e _Routing keys_ é algo particular a cada aplicação. Vamos exemplificar apenas os aspectos de "**escutar uma fila**" ou "**enviar mensagens**".

### Escutar uma fila

Para "escutar uma fila" a forma mais simples no Spring AMQP é utilizar a anotação `@RabbitListener`[^spring-RabbitListener-javadoc].

#### Fila anônima

```java
import java.net.URISyntaxException;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import br.gov.capes.commons.MensagemEvento;

@Service
public class ProcessadorPublicacoes {

    private static Logger logger = LoggerFactory.getLogger(ProcessadorPublicacoes.class);

    private UltimosAvisos repositorio;

    @Autowired
    public ProcessadorPublicacoes(UltimosAvisos repositorio) {
        this.repositorio = repositorio;
    }

    @RabbitListener(
        bindings = {
            @QueueBinding(
                declare = "true",
                exchange = @Exchange(name = "narq.avisos-topx", type = ExchangeTypes.TOPIC),
                value = @Queue,
                key = "Aviso.publicado"
            )
        }
    )
    public void processar(MensagemEvento mensagem) throws URISyntaxException {
        logger.debug("@processar(mensagem:MensagemEvento='{}'):void", mensagem);
        //requisitar evento
        //requisitar dados complementares/satélites ao evento
        //...
    }

}
```

Utilizando `declare` com `"true"` na anotação `@QueueBinding`[^spring-QueueBinding-javadoc] para não implementar a declaração do _bean_ de `Binding`[^spring-Binding-javadoc].

> `MensagemEvento` é uma classe utilitária implementando o formato recomendado para mensagens[^formato-msg-event-capes]. O módulo `br.gov.capes.cgs.narq:mensageria-starter` já configura a integração com o contexto de _beans_ do Spring Framework de modo que são transitados em formato JSON utilizando **Jackson**[^fasterxml-jackson].

#### Fila nomeada

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Service;

@Service
public class ProcessadorModificacoes {

    private static Logger logger = LoggerFactory.getLogger(ProcessadorModificacoes.class);

    @RabbitListener(
        bindings = {
            @QueueBinding(
                exchange = @Exchange(name = "narq.avisos-topx", type = ExchangeTypes.TOPIC),
                value = @Queue(name = "avisosModificados"), //uma fila normal...
                key = { "Aviso.modificado" }
            )
        }
    )
    public void autoProcessar(Aviso mensagem) {
        logger.debug("@autoProcessar(mensagem:{}):void", mensagem.getClass());
        //processar...
    }

}
```

### Enviar mensagem

```java
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.atomic.AtomicLong;

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import br.gov.capes.commons.Link;
import br.gov.capes.commons.MensagemEvento;
import br.gov.capes.narq.prototipos.avisos.DadosAviso.Status;

//...

@Autowired
private RabbitTemplate rabbitTemplate;

//..

public void process(Status antigo, Aviso aviso) {
    Object mensagem = aviso;
    String chaveRota = "Aviso.modificado";
    if(aviso.getStatus() == Status.PUBLICADO) {
        mensagem = this.montarMensagemEventoPublicacao(aviso);
        chaveRota = "Aviso.publicado";
    }
    this.rabbitTemplate.convertAndSend("narq.avisos-tx", chaveRota, mensagem);
}

private MensagemEvento montarMensagemEventoPublicacao(Aviso aviso) {
    AvisoPublicado publicacao = this.novaPublicacao(aviso);
    String idMsg = publicacao.toString();
    Link link = Link.builder()
            .rel("http://narq.capes.gov.br/refs/events/Aviso/publicado")
            .href("http://localhost:8080/feeds/avisos/publicados/" + publicacao.getId())
        .build();
    return new MensagemEvento(idMsg, link);
}

//...

```

Para enviar a mensagem estamos fazendo uso do `RabbitTemplate`[^spring-RabbitTemplate-javadoc]. É importante que a mensagem seja emitida com a _Routing key_ correta. Não apontamos qualquer declaração do _Exchange_, pois ele normalmente ou é definido através das anotações de `@RabbitListener` ou como um _bean_ da aplicação.

Os exemplos devem ser suficientes para implementar o que está descrito como um possível desenho na documenação para utilização de eventos entre aplicações[^docs-eventos].

# Notas e Referências

[^rabbitmq-vhosts]: Virtual Hosts no RabbitMQ -  https://www.rabbitmq.com/vhosts.html
[^use-configmap]: O modo mais adequado de prover o arquivo de configuração para a aplicação é através de ConfigMap - https://docs.openshift.com/container-platform/3.11/dev_guide/configmaps.html
[^obs-versao-no-archetype]: A versão já está configurado no `pom.xml` que é gerado pelo arquétipo e a tag `<version>` e a tag de propriedade podem ser suprimidas nesse caso.
[^spring-RabbitListener-javadoc]: `org.springframework.amqp.rabbit.annotation.RabbitListener` - https://docs.spring.io/spring-amqp/docs/2.3.10/api/org/springframework/amqp/rabbit/annotation/RabbitListener.html
[^spring-QueueBinding-javadoc]: `org.springframework.amqp.rabbit.annotation.QueueBinding` - https://docs.spring.io/spring-amqp/docs/2.3.10/api/org/springframework/amqp/rabbit/annotation/QueueBinding.html
[^spring-Binding-javadoc]: `org.springframework.amqp.core.Binding` - https://docs.spring.io/spring-amqp/docs/2.3.10/api/org/springframework/amqp/core/Binding.html
[^formato-msg-event-capes]: Vide https://git.capes.gov.br/dti/orientacoes-gerais/guia/blob/melhoria_arquitetura-eventos/arquitetura/definicoes/eventos/README.md#131-publicar-evento
[^fasterxml-jackson]: Jackson Project - https://github.com/FasterXML/jackson
[^spring-RabbitTemplate-javadoc]: `org.springframework.amqp.rabbit.core.RabbitTemplate` - https://docs.spring.io/spring-amqp/docs/2.3.10/api/org/springframework/amqp/rabbit/core/RabbitTemplate.html
[^docs-eventos]: Eventos & Mensageria entre aplicações - https://git.capes.gov.br/dti/orientacoes-gerais/guia/blob/master/arquitetura/definicoes/eventos/README.md
[^rabbitmq-jms-no-xa]: https://www.rabbitmq.com/jms-client.html#limitations
[^jms]: Java Message Service - https://www.oracle.com/java/technologies/java-message-service.html
[^jta]: Java Transaction API - https://www.oracle.com/java/technologies/jta.html
