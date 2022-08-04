**Pilha Java para Arquitetura de Referência - versão 2.x**

# Leituras Recomendadas

- [Spring Cloud OpenFeign](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/)

# Introdução

É comum que seja necessário que um serviço seja cliente de outro. Nos casos onde essa necessidade é de natureza síncrona, em que a requisição que está sendo processada depende do resultado de uma requisição a outra API REST, é interessante fazer uso de um cliente para requisições REST. Selecionamos o Feign para isso, por inclusive fazer parte do ecossistema de Spring Boot através do projeto Spring Cloud. Tinhamos a ideia inicial de abdicar de tudo do Spring Cloud, mas por pragmatismo e simplicidade vamos continuar usando alguns recursos como a integração e autoconfiguração com o OpenFeign.

# Dependências e Versionamentos

- Spring Cloud OpenFeign: `3.0.4`
  - Open Feign: `10.12`
- Spring Cloud Circuit Breaker Resilience4J: `2.0.2`
  - Resilience4J: `1.7.0`
- **Opcionais**
  - Spring Cloud Contract
    - spring-cloud-contract-wiremock: `3.0.2`
    - spring-cloud-contract-stub-runner: `3.0.2`

# Tutoriais

## Maven

Adicionar no `pom.xml` a dependência `br.gov.capes.cgs.narq:restclient-starter` na versão adequada:

```xml
<dependency>
  <groupId>br.gov.capes.cgs.narq</groupId>
  <artifactId>restclient-starter</artifactId>
  <version>${restclient-starter.version}</version>
</dependency>
```

A propriedade `${restclient-starter.version}` deve ser definida criando a tag `<restclient-starter.version>` na tag `<properties>` do `pom.xml`. O valor dever ser o valor da versão da pilha[^obs-versao-no-archetype].

### Remoção do _Circuit Breaker_

O módulo de _Circuit Breaker_ do Spring Cloud é carregado junto com o de OpenFeign, caso queria remover esse módulo do _runtime_ da aplicação, é possível excluir ele no `pom.xml`:

```xml
<dependency>
  <groupId>br.gov.capes.cgs.narq</groupId>
  <artifactId>restclient-starter</artifactId>
  <version>${restclient-starter.version}</version>
  <exclusions>
    <exclusion>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

### Testes

É importante testar a integração com o Feign, principalmente se houver uso de _Fallbacks_. Por isso `br.gov.capes.cgs.narq:restclient-starter` tem como dependência opcional Spring Cloud Contract[^spring-cloud-contract]. Para utilizá-las para testes:

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-contract-wiremock</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-contract-stub-runner</artifactId>
  <scope>test</scope>
</dependency>
```

O versionamento já foi adequado em `br.gov.capes.cgs.narq:restclient-starter`.

## Spring Boot

Não estabelecemos nenhuma extensão, nem pré-configuração diferente do que já é definido no Spring Cloud. Vamos apenas mencionar alguns aspectos importantes de configuração.

### _Circuit Breaker_

A utilização da dependência `br.gov.capes.cgs.narq:restclient-starter` seleciona o Resilience4J como _Circuit Breaker_, para ativar a utilização dele no Feign é preciso ativar a configuração `feign.circuitbreaker.enabled`:

**`application.yml`**
```yaml
feign:
   circuitbreaker:
      enabled: true
```

> Observe que os projetos criados através do arquétipo tem essa propriedade configurada para false, mesmo que o módulo não esteja adicionado no `pom.xml`.

### _Timeouts_

É possível que os _timeouts_ das requisições, principalmente com a ativação do _Circuit Breaker_, sejam muito curtos para as requisições realizadas[^prefira-async]. Nesses casos existem algumas propriedades que podem ser configuradas:

- `feign.client.config.default.connect-timeout`: limite **em milisegundos** de espera para que a conexão com o destino seja estabelecida, **10 segundos** por _default_;
- `feign.client.config.default.read-timeout`: o limite **em milisegundos** de espera para obter o conteúdo da resposta, **60 segundos** por _default_;
- `feign.client.config.<nome-instancia>.connect-timeout`: análogo a `feign.client.config.default.connect-timeout`, mas não existe _default_, herda do global;
- `feign.client.config.<nome-instancia>.read-timeout`: análogo a `feign.client.config.default.read-timeout`, mas não existe _default_, herda do global.

O `<nome-instancia>` é o mesmo valor em `name`/`value` na anotação `@FeignClient`. Assim é posível estabelecer um ajuste fino, com um valor _default_ adequado e valores especiais em casos particularmente lentos.
O limite de instância tem prioridade, então ele será respeitado.

#### _Timeouts_ no _Circuit Breaker_

É possível configurar `resilience4j.timelimiter.configs.default.timeout-duration` para estabelecer um limite compatível com as configurações do Feign, pois se _Circuit Breaker_ estiver ativo e for menor que o do Feign, o _Fallback_ será invocado.

> Não há suporte padrão a _timeouts_ de instância do Resilience4J no Spring CLoud OpenFeign, por isso apenas o limite global é respeitado.

#### Exemplo

**`application.yml`**
```yaml
feign:
  client:
    config:
      default:
        connect-timeout: 60000
        read-timeout: 10000
      <instancia>:
        connect-timeout: 120000
        read-timeout: 20000
resilience4j:
  timelimiter:
    configs:
      default:
        timeout-duration: 125000
```

> No exemplo acima o timeout global do _Circuit Breaker_ tem de ser maior que o timeout da instância do FeignClient.

## Java

Para habilitar os clientes Feign é preciso que a anotação `@EnableFeignClients` seja processada. O lugar mais comum de fazer isso é na _main class_ da aplicação.

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.openfeign.EnableFeignClients;

@EnableFeignClients
@SpringBootApplication
public class RestClientApplication {

  public static void main(String[] args) {
    SpringApplication.run(RestClientApplication.class, args);
  }

}
```

É usual anotar com `@EnableFeignClients` a mesma classe anotada com `@SpringBootApplication` se essa classe estiver num pacote que é "pai" de todos os `@FeignClient` da aplicação. Caso uma outra classe de `@Configuration` seja utilizada é importante utilizar os parâmetros de `@EnableFeignClients` como `basePackages`.

### _Circuit Breaker Fallbacks_

Com o _Circuit Breaker_ habilitado é possível criar _Fallbacks_ para os clientes Feign.

**`Pessoas.java`**
```java
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

@FeignClient(name = "pessoas", url = "${services.pessoas.base-uri}", path = "/pessoas", fallback = PessoasFallback.class)
public interface Pessoas {

  @RequestMapping(path = "/{id}", produces = MediaType.APPLICATION_JSON_VALUE)
  Pessoa get(@PathVariable("id") Long id);

}
```

**`PessoasFallback.java`**
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

@Component
public class PessoasFallback implements Pessoas {

  private static Logger logger = LoggerFactory.getLogger(PessoasFallback.class);

  public PessoasFallback() {
    logger.debug("new PessoasFallback()");
  }

  @Override
  public Pessoa get(Long id) {
    logger.debug("@get(id:long='{}')", id);
    Pessoa desconhecida = new Pessoa();
    desconhecida.setId(-1L);
    return desconhecida;
  }

}
```

 **As implementações de _Fallback_ precisam ser _beans_ do Spring para funcionar**. Provavelmente a forma mais fácil de fazer isso é anotar a classe com `@Component`[^spring-component-annotation] conforme no exemplo acima.

### Invocação em paralelo

Quando o processamento de uma resposta depende de mais de uma requisição via Feign é interessante processar tantas chamadas quanto possível em paralelo. A forma mais direta de fazer isso com Java 8+ é utilizando `CompletableFuture<T>`[^CompletableFuture-javadoc]. Vamos fazer um exemplo de brinquedo reutilizando o FeignClient `Pessoas` definido anteriormente:

```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
//...
import org.springframework.beans.factory.annotation.Autowired;
//...

//...

  @Autowired
  private Pessoas pessoas;

//...

  private List<String> paralelo(Long de, Long ate) throws InterruptedException, ExecutionException {
    List<String> nomes = new ArrayList<String>((int)(ate - de + 1));
    @SuppressWarnings("unchecked")
    CompletableFuture<Pessoa>[] futuros = new CompletableFuture[(int)(ate - de + 1)];
    for(long i = de; i <= ate; i++) {
      final long id = i;
      int j = (int)(id - de);
      futuros[j] = CompletableFuture.supplyAsync(() -> this.pessoas.get(id));
    }
    CompletableFuture.allOf(futuros).join();
    for(CompletableFuture<Pessoa> futuro : futuros) {
      nomes.add(futuro.get().getNome());
    }
    return nomes;
  }

//...

```

### Testes

Podemos utilizar o WireMock[^Wiremock] para se passar pelo servidor que está sendo acessado via Feign:

```java
import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.cloud.contract.wiremock.AutoConfigureWireMock;
import org.springframework.core.io.Resource;
import org.springframework.util.StreamUtils;

import com.github.tomakehurst.wiremock.WireMockServer;
import com.github.tomakehurst.wiremock.client.WireMock;
import com.google.common.base.Charsets;

@SpringBootTest(
  properties = {
    "services.pessoas.base-uri=http://localhost:${wiremock.server.port}"
  }
)
@AutoConfigureWireMock(port = 0)
class PessoasFallbackIT {

  @Autowired
  private WireMockServer server;

  @Value("classpath:wiremock/stubs/pessoas/1000.json")
  private Resource umaPessoa;

  @Autowired
  private Pessoas pessoas;

  @BeforeAll
  static void setUpBeforeClass() throws Exception {
  }

  @AfterAll
  static void tearDownAfterClass() throws Exception {
  }

  @BeforeEach
  void setUp() throws Exception {
    this.server.stubFor(
      WireMock.get(WireMock.urlEqualTo("/pessoas/1000"))
      .willReturn(
        WireMock.aResponse()
          .withStatus(200)
          .withHeader("Content-Type", "application/json")
          .withBody(StreamUtils.copyToString(this.umaPessoa.getInputStream(), Charsets.UTF_8))
      )
    );
  }

  @AfterEach
  void tearDown() throws Exception {
  }

  @Test
  void test() throws Exception {
    assertEquals("FULANO DA SILVA SAURO", pessoas.get(1000L).getNome());
  }
}
```

A anotação `@AutoConfigureWireMock` com `port` igual a zero coloca o WireMock para ouvir uma porta aleatória. Utilizamos `${wiremock.server.port}` ao setar a propriedade `services.pessoas.base-uri` na anotação `@SpringBootTest` para que o _Feign Client_ utilize o WireMock - `@FeignClient(name = "pessoas", url = "${services.pessoas.base-uri}"`.
O arquivo `src/test/resources/wiremock/stubs/pessoas/1000.json` é adicionado no classpath de testes, por isso carregamos como um `Resource`. O conteúdo dele é meramente o JSON que esperamos de retorno:

**`src/test/resources/wiremock/stubs/pessoas/1000.json`**
```json
{
  "id": 1000,
  "nome": "FULANO DA SILVA SAURO"
}
```

> Nada impede os times de utilizarem recursos como a JSON API do WireMock. Apenas ilustramos um dos modos de utilizar a ferramenta para fazer testes. Inclusive seria bem mais factual que essas configurações fossem feitas em suítes de testes de integração que testassem os clientes do _Feign Client_, não diretamente como suítes de testes dos próprios clientes isoladamente.

# Notas e Referências

[^obs-versao-no-archetype]: A versão já está configurado no `pom.xml` que é gerado pelo arquétipo e a tag `<version>` e a tag de propriedade podem ser suprimidas nesse caso.
[^spring-cloud-contract]: Spring Cloud Contract - https://spring.io/projects/spring-cloud-contract
[^Wiremock]: WireMock - http://wiremock.org/
[^resilience4j-springboot2-configs]: https://resilience4j.readme.io/docs/getting-started-3#configuration
[^prefira-async]: Nesses casos seria interessante também considerar requisições assíncronas que retornem HTTP 201 ou 202.
[^CompletableFuture-javadoc]: `java.util.concurrent.CompletableFuture<T>` - https://docs.oracle.com/javase/8/docs/api/index.html?java/util/concurrent/CompletableFuture.html
[^spring-component-annotation]: `org.springframework.stereotype.Component` https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Component.html
