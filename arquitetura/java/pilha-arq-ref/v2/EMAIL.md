**Pilha Java para Arquitetura de Referência - versão 2.x**

# Leituras Recomendadas

- [Spring Boot Reference - IO - 4. Sending Email](https://docs.spring.io/spring-boot/docs/current/reference/html/io.html#io.email)
  - [Spring Framework - Integration - 6. Email](https://docs.spring.io/spring-framework/docs/5.3.20/reference/html/integration.html#mail)
- [GreenMail](https://greenmail-mail-test.github.io/greenmail/)
  - [Using JUnit5 extension](https://greenmail-mail-test.github.io/greenmail/#ex_junit_extension)

# Introdução

**E-Mail é uma das peças fundamentais da Internet**. É uma forma clássica de informação ser disponbilizada de maneira assincrona/assimétrica para usuárias de aplicações e plataformas.

> ## Uso Consciente
> Aplicações devem evitar enviar mensagens de e-mail para suas usuárias. Mensagens geradas automaticamente devem ter uma relevância e usualmente implicarem que as recipientes de tais mensagens devam (ou pelo menos possam) tomar alguma atitude em relação ao conteúdo transmitido.
> A DTI (ainda) não tem uma política de recomendações quanto a produção e emissão de e-mails automaticamente por aplicações, mas quando tiver, _essas recomendações devem ser seguidas fielmente_.

Seguindo a ideia de se apoiar o melhor possível nas ferramentas já disponibilizadas pelos _frameworks_ base (Spring, Spring Boot, Spring Cloud), não criamos um módulo na pilha, mas vamos documentar aqui alguns aspectos técnicos e recomendações de implementação.

## E-Mails são efeitos colaterais assíncronos

Em geral é importante manter em mente a ideia de que e-mails deveriam ser gerados como um efeito colateral assíncrono a operação que os exige. Um `HTTP POST`  não deveria esperar para um e-mail fosse gerado, enviado e confirmado pelo SMTP antes de retornar. Um e-mail de notificação é (normalmente) algo satélite ao que foi manipulado pelo _endpoint_ REST. É bem possível que esse efeito colateral de emissão de e-mail seja realizado por uma instância de um serviço especializado, os endpoints REST rodam numa aplicação diferente da que processa/gera e-mails e integra com o SMTP da CAPES. Provavelmente faz sentido ser um microsserviço separado, que inclusive integraria com o microsserviço de endpoints REST através de [Mensageria](./MENSAGERIA.md).

# Dependências e Versionamentos

- `org.springframework.boot:spring-boot-starter-mail`: `2.6.6`
  - `com.sun.mail:jakarta.mail`: `1.6.7`
- GreenMail (_testes_)
  - `com.icegreen:greenmail-junit5`: `1.6.9`

# Tutoriais

## Maven

Adicionar no `pom.xml` a dependência `org.springframework.boot:spring-boot-starter-mail` na versão adequada:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-mail</artifactId>
  <version>${spring-boot-starter-mail.version}</version>
</dependency>
```

A propriedade `${spring-boot-starter-mail.version}` deve ser definida criando a tag `<spring-boot-starter-mail.version>` na tag `<properties>` do `pom.xml`. O valor dever ser o valor da versão da pilha[^obs-versao-no-archetype].

### Testes

A recomendação é utilizar GreenMail[^greenmail] para mimetizar um servidor SMTP. `com.icegreen:greenmail-junit5` está incluso na árvores de dependências de `br.gov.capes.cgs.narq:testes`, sendo suficiente incluir [esse módulo como uma dependência de testes](./TESTES.md).

## Spring Boot

As propriedades `spring.mail.*` podem ser adicionadas normalmente no YAML de configurações[^application-yaml-configmap].

**Exemplo**:
```yaml
spring:
  mail:
    host: smtp.capes.gov.br
    port: 25
    protocol: smtp
```

- **Não há um servidor de e-mail de DHT**: o serviço de SMTP é único[^multiplos-smtp], e as aplicações devem implementar como lidam para que mensagens de diferentes _stages_/ambientes sejam reconhecíveis como tais;
- **Não é preciso autenticação**: a integração pela porta 25 é protegida através do _firewall_ da instituição, as aplicações não precisam de credenciais, ou segredos[^K8s-secrets], para integrar.

## Java

Uma razão de não criarmos um módulo na pilha para aplicar configurações e abstrações específicas é de **evitar extrapolar generalização**[^overgeneralization]. É bem possível que todas as diferentes comunicações por e-mail de uma aplicação sejam realizadas por um único `@Service`[^spring-boot-service-stereotype].

```java
import javax.mail.internet.MimeMessage;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.stereotype.Service;

@Service
public class ServicoEmail {

    private JavaMailSender emailSender;

    @Autowired
    public ServicoEmail(JavaMailSender emailSender) {
      this.emailSender = emailSender;
   }

   //... (métodos implementando o envio de diferentes mensagens/notificações)

}
```

Não vamos entrar nas minúcias de criação de `MimeMessage`[^javax-mail-mime-msg-javadoc] `JavaMailSender.createMimeMessage()`[^spring-java-mail-sender-javadoc], ou envio de mensagens `JavaMailSender.send(MimeMessage mimeMessage)`[^spring-java-mail-sender-javadoc], etc.. Reforçamos as **Leituras Recomendadas** para isso.

### Testes de integração (GreenMail & JUnit 5)

Um aspecto que merece destaque é como realizar testes de integração do envio de e-mails. **Não tente integrar ao serviço de SMTP da CAPES para testes de integração**, _mesmo que tenha credenciais para contornar o bloqueio da porta 25 pelo firewall_.

O uso de GreenMail[^greenmail] para testes é recomenado pelo próprio Spring[^test-mail-server-deprecated].

```java
import static org.junit.Assert.assertEquals;

import javax.mail.Address;
import javax.mail.internet.MimeMessage;

import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.RegisterExtension;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import com.icegreen.greenmail.configuration.GreenMailConfiguration;
import com.icegreen.greenmail.junit5.GreenMailExtension;
import com.icegreen.greenmail.util.GreenMailUtil;
import com.icegreen.greenmail.util.ServerSetupTest;

import br.gov.capes.sdr.servico.EmailServico;

@SpringBootTest(
    properties = {
        "spring.mail.host=localhost",
        "spring.mail.port=3025", //porta configurada por ServerSetupTest.SMTP
    }
)
class ServicoEmailIT {

    private static String UM_DESTINATARIO = "fulano.da.silva@capes.gov.br";
    private static String UM_ASSUNTO = "Teste SDR E-mail";
    private static String UM_CORPO = "Envio email Simples";

    @Autowired
    private ServicoEmail servicoEmail;

    @RegisterExtension
    static GreenMailExtension greenMail = new GreenMailExtension(ServerSetupTest.SMTP)
        .withConfiguration(GreenMailConfiguration.aConfig())
        .withPerMethodLifecycle(true);

    private MimeMessage email;

    @BeforeAll
    static void setUpBeforeClass() throws Exception {
    }

    @AfterAll
    static void tearDownAfterClass() throws Exception {
    }

    @BeforeEach
    void setUp() throws Exception {
        EmailDeNotificacao mail = null;
        //mail = ...
        this.servicoEmail.enviarEmailSimples(mail);

        this.email = greenMail.getReceivedMessages()[0];
    }

    @AfterEach
    void tearDown() throws Exception {
    }

    @Test
    void testAssunto() throws Exception {
        assertEquals(UM_ASSUNTO, this.email.getSubject());
    }

    @Test
    void testDestinatarios() throws Exception {
        Address[] destinatarios = this.email.getAllRecipients();
        assertEquals(1, destinatarios.length);
        assertEquals(UM_DESTINATARIO, destinatarios[0].toString());
    }

    @Test
    void testCorpo() throws Exception {
        assertEquals(UM_CORPO, GreenMailUtil.getBody(this.email));
    }

}

```

- É interessante que suítes de testes de integração para serem executados pelo Maven Failsafe[^mvn-failsafe-plugin]. Para isso a classe deve terminar em `IT` (em vez de `Test`);
- As propriedades `spring.mail.*` podem ser descritas na classe de teste, sem precisar de um arquivo de propriedades (`application.properties` ou `application.yml`/`application.yaml`)

# Notas e Referências

[^overgeneralization]: Tem o ditado que diz que "Toda generalização é burra, inclusive essa". Generalizações costumam se apoiar em [DRY][b0506572], mas não comprometendo Coesão. Generalizações exageradas normalmente implicam código morto nos objetos. Se as instâncias do objeto concreto não utilizam o método, provavelmente a generalização dele é exagerada.

  [b0506572]: https://wiki.c2.com/?DontRepeatYourself "Don't Repeat Yourself"
[^obs-versao-no-archetype]: A versão já está configurado no `pom.xml` que é gerado pelo arquétipo e a tag `<version>` e a tag de propriedade podem ser suprimidas nesse caso.
[^application-yaml-configmap]: Adequadamente num ConfigMap do Kubernetes nos `values-*.yaml` do Helm Chart
[^multiplos-smtp]: Há na verdade vários servidores de SMTP e a aplicação deve apontar para o adequado.
[^K8s-secrets]:
[^spring-boot-service-stereotype]: `@org.springframework.stereotype.Service` - https://docs.spring.io/spring-framework/docs/5.3.18/javadoc-api/org/springframework/stereotype/Service.html
[^javax-mail-mime-msg-javadoc]: `javax.mail.internet.MimeMessage` - https://docs.oracle.com/javaee/6/api/javax/mail/internet/MimeMessage.html
[^spring-java-mail-sender-javadoc]: `org.springframework.mail.javamail.JavaMailSender` - https://docs.spring.io/spring-framework/docs/5.3.18/javadoc-api/org/springframework/mail/javamail/JavaMailSender.html
[^greenmail]: GreenMail - https://greenmail-mail-test.github.io/greenmail/
[^test-mail-server-deprecated]:  `org.springframework.integration.test.mail.TestMailServer` foi depreciado em favor do GreenMail - https://docs.spring.io/spring-integration/api/org/springframework/integration/test/mail/TestMailServer.html
[^mvn-failsafe-plugin]: Maven Failsafe Plugin - https://maven.apache.org/surefire/maven-failsafe-plugin/
