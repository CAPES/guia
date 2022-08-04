**Pilha Java para Arquitetura de Referência - versão 2.x**

> # NOTA
> **Não incluir informação sensível nos logs das aplicações. Senhas, segredos e informações pessoais de usuários não devem ser emitidas nos logs.**

# Leituras Recomendadas

- **[Especificação de formatos de logs para aplicações da CGS](/arquitetura/definicoes/logging/README.md)**
- [Spring Boot Logging](https://docs.spring.io/spring-boot/docs/2.6.1/reference/htmlsingle/#features.logging)
- [Manual SLF4J](https://www.slf4j.org/manual.html)
- [Manual Logback](https://logback.qos.ch/manual/)

# Introdução

A prática de logs de aplicações é algo muito comum e essencial para a observabilidade da saúde e bom funcionamento das aplicações além da eventual utilidade na depuração de erros e incidentes.

É fundamental que as aplicações registrem seus logs no nível e com a composição adequada.

## Conceitos

- **_Logging Agent_** ("Coletor de logs"): É a ferramenta utilizada para obter ("coletar" :sweat_smile:) os logs que estão sendo adicionados a um "_stream_"
- **_Logging Backend_** ("Agregador de logs"): É a ferramenta que recebe os logs (talves de um coletor) e os armazena de uma forma duradoura e disponível

> Implantamos o **Fluent Bit**[^fluentbit] como _Logging Agent_ do cluster OCP e a ferramenta de _Logging Backend_ da DTI é o **Graylog**[^graylog].


## Arquiteturas disponíveis

Essa versão da pilha Java para a Arquitetura de Referência é agressivamente apoiada na infraestrutura de orquestração e conteinerização de aplicações baseada em Kubernetes (e especificamente em Openshift). Esse tipo de abordagem tem algumas recomendações de como arquitetar o manuseio dos logs das aplicações. O Kubernetes descreve 3[^k8s-logging-architecture], dos quais só implementamos 2:

1. **_Logging agent_ no _node_**;
2. Logs expostos diretamente pela aplicação.

O _The Twelve-Factor App_ é ainda mais restritivo[^12factor-logs], indicando que as aplicações devem apenas emitir seus logs na saída padrão ("`STDOUT`"), conforme a primeira opção.

Recomendamos que as aplicações sejam construídas utilizando a primeira abordagem (**_Logging agent_ no _node_**), o que diminui o esforço de configuração na aplicações. Esperamos que isso fique evidente na secção de [Tutoriais](#tutoriais).

> Em casos específicos pode ser alinhado que a aplicação envie diretamente seus logs para o Graylog sem o intermédio do _Logging Agent_, mas esses casos devem ser alinhados.
> **Os logs não devem ser armazenados em arquivos em volumes externos montados no contêiner da aplicação**.

## Formatos dos Logs

Para que o coletor de logs se comporte corretamente e os dados consigam ser recebidos pelo agregador é necessário que o formato seja estável e bem definido. O Graylog aceita diversos formatos de log como _input_, como o Syslog do RFC 5424[^rfc-syslog], sob diferentes protocolos de comunicação (UDP, TCP, HTTP, AMQP, Kafka). Achamos melhor seguir o padrão GELF[^gelf] que tem um _payload_ estruturado. A grosso modo um _payload_ GELF é um _layout_ de um objeto JSON, com campos e tipos de valores bem definidos.
Temos um formato baseado numa estrutura JSON especificado pelo NARQ que é o formato padrão para logs que vão ser coletados. O coletor implantado no cluster OCP (Fluent Bit) foi configurado para lidar com esse formato e o transformar em GELF, por isso chamamos esse formato de `CgsGelfPrecursor`. O GELF que esperamos no Graylog contém uma diversidade de campos adicionais específicos para a CGS.

# Dependências e Versionamentos

- Spring Boot Logging Starter: `2.5.6`
- SLF4J: `1.7.32`
- Logback:
  - Logback Classic: `1.2.9`
  - Logback JSON Classic: `0.1.5`
  - Logback GELF: `4.0.5`

# Tutoriais

## Maven

Adicionar no `pom.xml` a dependência `br.gov.capes.cgs.narq:logging-starter` na versão adequada:

```xml
<dependency>
  <groupId>br.gov.capes.cgs.narq</groupId>
  <artifactId>logging-starter</artifactId>
  <version>${logging-starter.version}</version>
</dependency>
```

A propriedade `${persistencia-starter.version}` deve ser definida criando a tag `<persistencia-starter.version>` na tag `<properties>` do `pom.xml`. O valor dever ser o valor da versão da pilha[^obs-versao-no-archetype].

## Spring Boot

Definimos e pré-configuramos 2 _appenders_[^logback-appenders], que podem ser reconfigurados através de propriedades do Spring Boot:

- `CONSOLE`
- `GELF`

Em geral as configurações padrões devem ser adequadas e suficientes sem que haja necessidade de qualquer intervervenção do time de desenvolvimento (por exemplo em configMaps).

### `CONSOLE` _Appender_

Por padrão os logs em formato `CgsGelfPrecursor` já estão ativados e configurados para serem emitidos num _appender_ no formato esperado pelo _Logging Agent_ do cluster OCP.

```yaml
logging:
  console:
    enabled: true
    title:
      pattern: "%m%nopex"
    timestamp:
      pattern: "yyyy-MM-dd'T'HH:mm:ss.SSS"
    message-field:
      class: "br.gov.capes.cgs.narq.logging.TextFormat"
      pattern: "%m%n%ex{short}"
```

> O YAML acima exemplifica as propriedades disponíveis com seus respectivos valores padrões.

### `GELF` _Appender_

Para os casos extraordinários onde se mostre necessário que a aplicação envie os logs diretamente para o Graylog, já temos no módulo `br.gov.capes.cgs.narq:persistencia-starter` um suporte pré-configurado. Isso é realizado através de um _appender_[^logback-appenders], que pode ser ativado pela propriedade `logging.gelf.enabled`.

```yaml
logging:
  gelf:
    enabled: false
    host: localhost
    port: 12201
    encoder:
      includes:
        raw-message: false
        marker: false
        mdc-data: true
        caller-data: false
        root-cause: false
        level-name: true
      pattern:
        short: "%m%nopex"
        full: "%m%ex{short}"
        exception: "%ex{full}"
```

> O YAML acima exemplifica as propriedades disponíveis com seus respectivos valores padrões.

Os campos que apontam para o Graylog (`logging.gelf.host` e `logging.gelf.port`) provavelmente precisam ser ajustados para apontar corretamente para um Graylog apto a receber GELF via TCP.

> Apesar de `logging.gelf.enabled=true` não ser efetivamente incompatível com `logging.console.enabled=true`, o mais provável é que os dois _appenders_ não devam ser utilizados simultaneamente na mesma aplicação.

## Java

Utilizar a SLF4J[^slf4j] para emitir os logs:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Exemplo {

    private static Logger logger = LoggerFactory.getLogger(Exemplo.class);

    //...

        Object arg;
        Exception ex;

        //...

        logger.trace("tracing");
        logger.trace("tracing: {}", arg);
        logger.trace("tracing: {}", arg, ex);
        logger.debug("debuging");
        logger.debug("debuging: {}", arg);
        logger.debug("debuging: {}", arg, ex);
        logger.info("informing");
        logger.info("informing: {}", arg);
        logger.info("informing: {}", arg, ex);
        logger.warn("warning");
        logger.warn("warning: {}", arg);
        logger.warn("warning: {}", arg, ex);
        logger.error("erring");
        logger.error("erring: {}", arg);
        logger.error("erring: {}", arg, ex);

    //...

}
```

A utilização do SLF4J foge ao escopo dessa documentação.

### Mensagens Estruturadas

Por padrão o campo `message` no JSON emitido pelo `CONSOLE` _appender_ é uma `string`. Caso seja necessário adicionar mais conteúdo estruturado em JSON aos logs emitidos é possível fazer com que o campo `message` tenha um objeto JSON. Modificando o valor da propriedade `logging.console.message-field.class` para `br.gov.capes.cgs.narq.logging.JsonFormat` o campo `message` passa a conter um `br.gov.capes.cgs.narq.logging.MessageFieldAsJson`. No campo `info` é possível haver `null`, um objeto (`{}`), ou um array de objetos JSON (`[]`), dependendo da quantidade de instâncias de `br.gov.capes.cgs.narq.logging.MessageFieldExtraInfo`[^MessageFieldExtraInfo] que for passada para o método de log.

```java
import br.gov.capes.cgs.narq.logging.MessageFieldExtraInfo;

import lombok.Builder;
import lombok.Getter;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Builder
@Getter
class JwtAudit implements MessageFieldExtraInfo {

    private String jwtAud;
    private String jwtSub;

}

public class Exemplo {

    private static Logger logger = LoggerFactory.getLogger(Exemplo.class);

    //...

        Object arg;
        JwtAudit extraInfos;
        Exception ex;

        //...

        logger.info("informing: {}", arg, extraInfos, ex);

}
```

#### Cuidados

- A instância de `MessageFieldExtraInfo` não precisa ter um _placeholder_ (`{}`) no parâmetro de `format` do método de log. Isso na verdade só vai fazer com que o `toString()` da instância seja incluído no campo `text` do objeto JSON o que provavelmente é indesejado além de um desperdício
- Sempre deixar a exceção (caso haja) como último argumento do método de log
- `br.gov.capes.cgs.narq.logging.MessageFieldExtraInfo` é apenas uma "interface de marcação". As instâncias devem ser objetos concretos que possam ser esternalizados em JSON através da Jackson[^jackson]

> # Lembrete
> **Não incluir informação sensível nos logs das aplicações. Senhas, segredos e informações pessoais de usuários não devem ser emitidas nos logs.**

# Notas e Referências

[^k8s-logging-architecture]: Kubernetes Logging Architecture - https://kubernetes.io/docs/concepts/cluster-administration/logging/
[^12factor-logs]: XI. Logs (The Twelve-Factor App) - https://12factor.net/logs
[^rfc-syslog]: RFC 5424 - The Syslog Protocol - https://datatracker.ietf.org/doc/html/rfc5424
[^gelf]: Graylog Extended Log Format - https://docs.graylog.org/docs/gelf
[^obs-versao-no-archetype]: A versão já está configurado no `pom.xml` que é gerado pelo arquétipo e a tag `<version>` e a tag de propriedade podem ser suprimidas nesse caso.
[^logback-appenders]: Logback Manual - Chapter 4: Appenders -
https://logback.qos.ch/manual/appenders.html
[^slf4j-javadoc-logger]: `org.slf4j.Logger` - https://www.slf4j.org/api/index.html?org/slf4j/Logger.html
[^slf4j]: Simple Logging Facade for Java (SLF4J) - https://www.slf4j.org/
[^MessageFieldExtraInfo]: É apenas uma interface de marcação
[^fluentbit]: Fluent Bit - https://fluentbit.io/
[^graylog]: Graylog - https://www.graylog.org/
[^jackson]: Jackson Project - https://github.com/FasterXML/jackson
