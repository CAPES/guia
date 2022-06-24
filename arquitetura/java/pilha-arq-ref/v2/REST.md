**Pilha Java para Arquitetura de Referência - versão 2.x**

# Leituras Recomendadas

- [Definições para construção de API REST](https://git.capes.gov.br/dti/orientacoes-gerais/guia/blob/master/arquitetura/rest-apis.md)

# Introdução

O foco da pilha Java para a Arquitetura de Referência é estabelecer os padrões e tecnologias para a implementação de aplicações que definam APIs REST como _backends_. Por isso é fundamental um módulo que facilite a criação de serviços Web/HTTP.

A ideia geral é se apoiar no `org.springframework.boot:spring-boot-starter-web` com algumas pré-configurações:

- CORS[^cors] habilitado para todos os verbos HTTP e _endpoints_
- Configurações de serialização e desserialização de JSON
- RFC 7807[^rfc7807] para retornar erros

# Dependências e Versionamentos

- Spring Boot Web Starter: `2.5.6`
- RFC 7807[^rfc7807]
  - `org.zalando:problem`: `0.27.1`
  - `org.zalando:problem-spring-web`: `0.28.0-RC0`

# Tutoriais

## Maven

Adicionar no `pom.xml` a dependência `br.gov.capes.cgs.narq:logging-starter` na versão adequada:

```xml
<dependency>
  <groupId>br.gov.capes.cgs.narq</groupId>
  <artifactId>web-starter</artifactId>
  <version>${web-starter.version}</version>
</dependency>
```

A propriedade `${web-starter.version}` deve ser definida criando a tag `<web-starter.version>` na tag `<properties>` do `pom.xml`. O valor dever ser o valor da versão da pilha[^obs-versao-no-archetype].

## Spring Boot

Não criamos nenhuma extensão para o módulo `br.gov.capes.cgs.narq:web-starter`. O restante das configurações de HTTP pode ser utilizado como é convencional no Spring Boot.

## Java

### Serialização/Desserialização JSON

O `ObjectMapper`[^jackson-objectmapper-javadoc] pode ser obtido com um `@Autowired`[^spring-autoWired-javadoc] num componente Spring[^esteriotipos-contam].

```java

//...

    @Autowired
    private ObjectMapper mapper;

//...

```

### Utilizando o padrão do RFC 7807

**Exemplo 1**
```java
import java.net.URI;
import java.net.URISyntaxException;
import java.util.ArrayList;
import java.util.Optional;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

import br.gov.capes.cgs.narq.commons.ProblemInfo;

//...

    @Autowired
    private Entidades entidades;

    @PostMapping(
        consumes = {
            MediaType.APPLICATION_JSON_VALUE
        },
        produces = {
            MediaType.APPLICATION_JSON_VALUE,
            MediaType.APPLICATION_PROBLEM_JSON_VALUE
        }
    )
    public ResponseEntity<?> post(@RequestBody(required = false) String dados) {
        Optional<Entidade> nova =this.entidades.criar(dados);
        return nova.map(this::criada)
                .orElseGet(() -> this.dadosInvalidos(dados));
    }

    private ResponseEntity<Object> criada(Entidade nova) {
        URI location;
        try {
            location = new URI("/rest/entidades/" + nova.getId());
        } catch (URISyntaxException e) {
            throw new IllegalArgumentException(e);
        }
        return ResponseEntity.created(location).body(nova);
    }

    private ResponseEntity<Object> dadosInvalidos(String dados) {
        ProblemInfo problem;
        try {
            problem = ProblemInfo.of()
                    .type("/refs/errors/DadosInvalidos")
                    .status(HttpStatus.BAD_REQUEST)
                    .title("Conteúdo inválido")
                    //não tem detail..., mas tem um extensão
                    .extension("inconsistencias", this.entidades.inconsistencias(dados))
                .build();
        } catch (URISyntaxException e) {
            throw new IllegalArgumentException(e);
        }
        return ResponseEntity.badRequest()
                .contentType(MediaType.APPLICATION_PROBLEM_JSON)
            .body(problem);
    }

//...

```

- A classe `Entidades` é apenas um exemplo de um serviço (`@Service`[^spring-service-javadoc]) ou repositório (`@Repository`[^spring-repository-javadoc]) sendo utilizado na camada REST.
- No exemplo não utilizamos exceções para controlar o fluxo, o `ResponseEntity`[^spring-responseentity-javadoc] devido é retornado caso seja um sucesso ou algum erro
  - No caso de uma `URISyntaxException`[^urisyntaxexception-javadoc] é que o erro borbulharia até o "tratador de exceções"

### Extendendo `Problem`[^zalando-problem-javadoc]

A classe `br.gov.capes.cgs.narq.commons.ProblemInfo` provavelmente é geral o suficiente para a vasta maioria dos casos do _payload_ necessário como resposta.

A serialização/desserialização JSON de um `? extends Problem` para `ProblemInfo` e vice versa pode ser simplificado usando a classe abstrata `br.gov.capes.cgs.narq.commons.AbstractProblem`:

```java
import java.net.URI;
import java.net.URISyntaxException;
import java.util.List;

import org.springframework.http.HttpStatus;

import br.gov.capes.cgs.narq.commons.AbstractProblem;
import lombok.AccessLevel;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;

@Getter
@ToString(callSuper = true)
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public class DadosInvalidos extends AbstractProblem {

    static final String URI_STR = "/refs/errors/DadosInvalidos";
    static final URI URI;
    static final String TITLE = "Conteúdo inválido";

    static {
        try {
            URI = new URI(URI_STR);
        } catch (URISyntaxException e) {
            throw new IllegalStateException(e);
        }
    }

    @Setter(value = AccessLevel.PRIVATE)
    private List<String> inconsistencias;

    @Builder(builderMethodName = "of")
    private DadosInvalidos(List<String> inconsistencias) throws URISyntaxException {
        this.type = URI;
        this.title = TITLE;
        this.status = HttpStatus.BAD_REQUEST;
        this.instance = null;
        this.inconsistencias = inconsistencias;
    }

}
```

> #### Adicionando outro "tratador de exeções"
> Em geral não deveria ser necessário adicionar outros `@ControllerAdvice`[^spring-controlleradvice-javadoc]. Pondere se exceções não estão sendo utilizadas para controlar o fluxo da aplicação (o que é considerado uma má prática em Java).

### Erros "Genéricos"

Caso uma exceção diferente de `ProblemInfoException` seja lançada um erro "genérico" e mínimo será emitido pelo "tratador de erros":

```json
{
  "type": "about:blank",
  "title": "Internal Server Error",
  "detail": null,
  "status": 500
}
```

Esse _payload_ se alinha com o RFC 7807[^rfc7807] no sentido que é considerado um _payload_ sem mais informação do que o próprio HTTP indicaria.

> ### Exceções da versão 1.x
> As exceções do pacote `br.gov.capes.utils.exceptions` já haviam sido depreciadas e foram removidas dessa versão da pilha. Lembramos que o controle de fluxo negocial não deve ser feito através de exceções. [O Hystrix também foi retirado da pilha 2.x em favor do Resilience4J](./RESTCLIENT.md). Também observamos que aquelas exceções apesar de se proporem "negociais", vazavam para outras camadas como a de apresentação (no caso a camada HTTP REST). Em geral podemos considerar um abuso de generalização criar exceções negociais, que inclusive costumam tanger o desenvolvimento para a prática de utilizar exceções para o controle de fluxo negocial. Esse assunto tem algumas polêmicas, mas em geral vamos recomendar que as aplicações criem suas próprias exceções quando for adequado, ou utilizem as da Standard Edition.

# Cuidados

## RFC 7807

- O cabeçalho HTTP `Content-Type` para retornar um `Problem` (como `ProblemInfo`) **DEVE** ser `application/problem+json`. **NUNCA** retorne outro Media Type
- Cuidado com o conteúdo retornado! Conforme o RFC 7807[^rfc7807] insiste, **a função desse Media Type não é _debbug_**, ele deve servir como uma interface comum que pode ser utilizada programaticamente para que o cliente reaja adequadamente a erros. Não insira pilhas de erro/_stacktraces_ ou dados sensíveis.
- As aplicações **NUNCA** deveriam emitir `ProblemInfoException` fora da camada HTTP/REST (usualmente o que é anotado com `@RestController`[^spring-restcontroller-javadoc])

# Notas e Referências

[^cors]: Cross-Origin Resource Sharing (CORS) - https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
[^rfc7807]: Problem Details for HTTP APIs - https://datatracker.ietf.org/doc/html/rfc7807
[^spring-restcontroller-javadoc]: `org.springframework.web.bind.annotation.RestController` - https://docs.spring.io/spring-framework/docs/current/javadoc-api/index.html?org/springframework/web/bind/annotation/RestController.html
[^obs-versao-no-archetype]: A versão já está configurado no `pom.xml` que é gerado pelo arquétipo e a tag `<version>` e a tag de propriedade podem ser suprimidas nesse caso.
[^zalando-problem-javadoc]: `org.zalando.problem.Problem` - https://javadoc.io/static/org.zalando/problem/0.27.1/index.html?org/zalando/problem/Problem.html
[^spring-controlleradvice-javadoc]: `org.springframework.web.bind.annotation.ControllerAdvice` - https://docs.spring.io/spring-framework/docs/current/javadoc-api/index.html?org/springframework/web/bind/annotation/ControllerAdvice.html
[^jackson-objectmapper-javadoc]: `com.fasterxml.jackson.databind.ObjectMapper` - https://fasterxml.github.io/jackson-databind/javadoc/2.12/com/fasterxml/jackson/databind/ObjectMapper.html
[^spring-autoWired-javadoc]: `org.springframework.beans.factory.annotation.Autowired` - https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/annotation/Autowired.html
[^esteriotipos-contam]: Anotações de esteriótipos DDD como `@Service`, `@Repository` além de outros tipos de componentes como `@RestController` são definidas como `@Component`
[^spring-service-javadoc]: `org.springframework.stereotype.Service` - https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Service.html
[^spring-repository-javadoc]: `org.springframework.stereotype.Repository` - https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Repository.html
[^spring-responseentity-javadoc]: `org.springframework.http.ResponseEntity` - https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/ResponseEntity.html
[^urisyntaxexception-javadoc]: `java.net.URISyntaxException` - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/net/URISyntaxException.html
