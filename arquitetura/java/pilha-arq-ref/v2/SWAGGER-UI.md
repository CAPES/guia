**Pilha Java para Arquitetura de Referência - versão 2.x**

# Leituras Recomendadas

- [OpenAPI 3.0 Specification](https://swagger.io/specification/)
- [springdoc-openapi - OpenAPI 3 Library for spring-boot](https://springdoc.org/)
  - [GitHub Repo](https://github.com/springdoc/springdoc-openapi)
- [Swagger 2.X Annotations](https://github.com/swagger-api/swagger-core/wiki/Swagger-2.X---Annotations)
- **[Definições para construção de API REST](https://xpto.com/dti/orientacoes-gerais/guia/blob/doc_pilha_java_2/arquitetura/rest-apis.md)**[^rest-api-specs]

# Ferramentas úteis

- [Swagger UI](https://swagger.io/tools/swagger-ui/)
- [Swagger Editor](https://swagger.io/tools/swagger-editor/)
  - [Live Demo](https://editor.swagger.io/)

# Introdução

Uma **API REST** precisa ser bem definida[^rest-api-specs] e devidamente documentada. **OpenAPI**[^openapi] é um formato de descrição de APIs REST. Com um formato bem definido de descrição é possível estabelecer uma "documentação viva" para a API REST da aplicação se valendo de outras ferramentas. Uma ferramenta que faz um ótimo trabalho para documentação e testes de uma API REST descrita em OpenAPI é o **Swagger UI**. A utilidade e praticidade da ferramenta é considerável por isso temos ela incluída na pilha.

# Dependências e Versionamentos[^obs-versionamento]

- SpringDoc OpenAPI: `1.5.10`
  - Swagger Core[^sWagger-core-jakarta]: `2.1.10`
    - Swagger Models: `io.swagger.core.v3:swagger-models-jakarta`
    - Swagger Annotations: `io.swagger.core.v3:swagger-annotations-jakarta`
    - Swagger Integration: `io.swagger.core.v3:swagger-integration-jakarta`
  - Swagger UI: `3.51.1`

# Tutoriais

## Maven

Adicionar no `pom.xml` a dependência `br.gov.capes.cgs.narq:swaggerui-starter` na versão adequada:

```xml
<dependency>
  <groupId>br.gov.capes.cgs.narq</groupId>
  <artifactId>swaggerui-starter</artifactId>
  <version>${swaggerui-starter.version}</version>
</dependency>
```

A propriedade `${swaggerui-starter.version}` deve ser definida criando a tag `<swaggerui-starter.version>` na tag `<properties>` do `pom.xml`. O valor dever ser o valor da versão da pilha[^obs-versao-no-archetype].

## Spring Boot & Springdoc

Há duas estratégias para que a especificação em OpenAPI que alimenta o Swagger UI:

1. Gerada a partir das anotações de mapeamento do Spring e do Swagger Core;
2. Carregada através de um arquivo YAML

O funcionamento padrão é gerar a especificação através das anotações, se em vez disso deseje utilizar um arquivo YAML predeterminado apontar o caminho/URI na propriedade `springdoc.capes-extensions.spec-file`.

**Exemplo**:
```yaml
springdoc:
  capes-extensions:
    specs-file: classpath:openapi.yaml
```
> No caso do exemplo foi utilizada uma URI para indicar que o arquivo está disponível no _classpath_ da aplicação (provavelmente empacotado no arquivo `.jar`). Não faça acesso a recursos fora do próprio servidor pra isso (não use `http://`, `https://`, etc.), apenas o que esteja no próprio sistema de arquivos do _host_, ou classpath da aplicação.

### Integração com o SSO da CAPES

A integração com o SSO já está pré-pronta ao utilizar `br.gov.capes.cgs.narq:swaggerui-starter`, tanto que a propriedade `springdoc.swagger-ui.oauth.endpoint`[^springdoc.sWagger-ui.oauth.endpoint] é **obrigatória** quando o Swagger UI está habilitado.
A propriedade `springdoc.swagger-ui.oauth.clientId` preenche o `client_id` na modal de autorização.

**Exemplo**:
```yaml
springdoc:
  swagger-ui:
    oauth:
      endpoint: https://des.capes.gov.br/sso/oauth
      clientId: uma-app.capes.gov.br
```

> #### Configurações no SAdmin
> Para integrar o Swagger UI com o SSO é necessário fazer algumas configurações no SAdmin:
> - Fluxo **Implicit**: adicionar o fluxo **Implicit** nos **FLUXOS PERMITIDOS**;
> - URI de redirecionamento: Adicionar a URI de redirecionamento
>   - O formato do _Path_ é `/swagger-ui/oauth2-redirect.html`
>     - Exemplo: http://localhost:8080/swagger-ui/oauth2-redirect.html
>     - Exemplo: http://uma-app-des.dht.ocp.capes.gov.br/swagger-ui/oauth2-redirect.html

### Grupos Padrões

Nossa pré-configuração estabelece alguns grupos para alguns contextos usuais e prováveis:
- **rest**: para os URI _paths_ `/rest/**`
  - _default_: habilitado
  - `springdoc.capes-extensions.groups.rest=disabled` para desabilitar
- **feeds**: para os URI _paths_ `/feeds/**`
  - _default_: desabilitado
  - `springdoc.capes-extensions.groups.feeds=enabled` para habilitar
- **refs**: para os URI _paths_ `/refs/**`
  - _default_: desabilitado
  - `springdoc.capes-extensions.groups.refs=enabled` para habilitar
- **diagnostis**: para os URI _paths_ `/diagnostis`, `/diagnostis/**`
  - _default_: desabilitado
  - `springdoc.capes-extensions.groups.diagnostis=enabled` para habilitar

**Exemplo**:
```yaml
springdoc:
  capes-extensions:
    groups:
      rest: enabled
      feeds: enabled
      refs: enabled
      diagnostics: enabled
```

### Desabilitar o Swagger UI

O Swagger UI provavelmente não deveria estar disponível em produção por ser uma ferramenta de documentação e testes.

Para desabilitar o Swagger UI há 2 alternativas:
1. `springdoc.swagger-ui.enabled=false`
2. `springdoc.api-docs.enabled=false` (também desabilita a publicação da especificação em OpenAPI)

> A configuração `springdoc.capes-extensions.enabled=false` apenas desabilita as configurações e capacidades especificas da CAPES, como a pre-configuração de integração com o SSO.

Essa versão `2.x` da pilha Java para a Arquitetura de Referência se apoia agressivamente na concepção de que as aplicações serão conteinerizadas e orquestradas. No caso as configurações específicas para produção devem ser devidamente feitas em ConfigMaps[^k8s-configmap]. A expectativa é de que as aplicações sejam implantadas através do Pipeline[^pipeline-apps] padrão de aplicações que utiliza o Helm Chart **capes-aplic**[^chart-capes-aplic], o que permite a definição ConfigMaps adequados para cada namespace.

## Java - Documentando com as anotações de Spring Web e Swagger Core

Por padrão o SpringDoc gera o arquivo de especificação com base no código fonte da aplicação, balizado pelas anotações do Spring Web e do Swagger Core.

**Anotações Spring Web**


_Annotation_  | _Target_[^interface-target]  | Finalidade  | Exemplo
--|---|---|--
`@RestController`[^spring-restcontroller-javadoc] | Classe | Estabelece uma classe como controladora de requisições REST.
`@RequestMapping`[^spring-requestmapping-javadoc] | Classe | Usamos para definir o endpoint raiz do `@RestController` | `@RequestMapping("/rest/pessoas") public class RestEndpoints`
`@GetMapping`[^spring-getmapping-javadoc] | Método | Usamos para processar requisições `HTTP GET`. | `@GetMapping public ResponseEntity<?> metodo()`
`@PostMapping`[^spring-postmapping-javadoc] | Método | Usamos para processar requisições `HTTP POST` | `@PostMapping public ResponseEntity<?> metodo()`
`@PutMapping`[^spring-putmapping-javadoc] | Método | Usamos para processar requisições `HTTP PUT` | `@PutMapping public ResponseEntity<?> metodo()`
`@DeleteMapping`[^spring-deletemapping-javadoc] | Método | Usamos para processar `HTTP DELETE` | `@DeleteMapping public ResponseEntity<?> metodo()`
`@PatchMapping`[^spring-patchmapping-javadoc] | Método | Usamos para processar `HTTP PATCH` | `@PatchMapping public ResponseEntity<?> metodo()`
`@RequestParam`[^spring-requestparam-javadoc] | Parâmetro | Usamos para mapear um HTTP _query param_, campos de formulário e partes de um _multipart_ (HTTP POST) | `@GetMapping public ResponseEntity<?> metodo(@RequestParam boolean todos)`
`@PathVariable`[^spring-pathvariable-javadoc] | Parâmetro | Usado para mapear o valor de URI _template_ do `value` no `path` da anotação de processamento de método HTTP | `@GetMapping("/imagens/{id}") public ResponseEntity<?> metodo(@PathVariable("id") Long id)`
`@RequestHeader`[^spring-requestheader-javadoc] | Parâmetro | Usamos para acessar informações que são envidas nos cabeçalhos HTTP.
`@RequestBody`[^spring-requestbody-javadoc] | Parâmetro | Usamos para acessar o _payload_ enviado na request (POST, PUT, PATCH)

- **_Target_**: Onde recomendamos utilizar a anotação[^interface-target]

> Se for absolutamente necessário processar outros métodos HTTP que não tem anotação própria (`HEAD`, `OPTIONS`, `TRACE`), a alternativa é anotar o método com `@RequestMapping`:
> ```java
> @RequestMapping(method = RequestMethod.HEAD)
> public ResponseEntity<?> head()
> ```

### Um Exemplo...

```java
import java.net.URI;

import org.springframework.http.HttpHeaders;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PatchMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/rest/public")
public class PublicRestEndpoints {

  @RequestMapping(method = RequestMethod.HEAD)
  public Object head() {
    //...
  }

  @GetMapping
  public Object get() {
    //...
  }

  @PostMapping(consumes = "application/vnd.capes.gov.br+json;profile=http://rest.capes.gov.br/refs/exemplo/Coisa")
	public Object post(@RequestBody Coisa treco) {
    //...
  }

  @DeleteMapping
  public Object delete(@RequestHeader(HttpHeaders.REFERER) URI origem) {
    //...
  }

  @PatchMapping
  public Object patch() {
    //...
  }

  @PutMapping
  public Object put() {
    //...
  }

}
```

- `value` de `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping` em métodos é concatenado com o `value` do `@RequestMapping` da classe
  - `@RequestMapping("/rest/public")` + `@GetMapping("/imagens")` vira o URI _path_ `/rest/public/imagens`

Para enriquecer e detalhar o resultado no Swagger UI, é possível utilizar as anotações de **Swagger Annotations**:

  _Annotation_  | _Target_[^interface-target]  | Finalidade
--|---|--
~~`@OpenAPIDefinition`~~[^sWagger-openapidef-javadoc] | Classe | **Não usamos essa anotação**, pois as informações dela são disponibilizadas através dos beans do _starter_. Adicionar essa anotação pode não ter efeito ou causar inconsistências em como o Swagger UI renderiza |
`@Operation`[^sWagger-operation-javadoc] | Método |  Usada para descrever uma operação |
`@ApiResponse`[^sWagger-apiresponse-javadoc] | Método  | Usada para explicar um retorno que pode ser dado |
`@Parameter`[^sWagger-parameter-javadoc] | Parâmetro | Usada para explicar/descrever um parâmetro na API REST  |
`@RequestBody`[^sWagger-requestbody-javadoc] [^requestbody-spring-vs-sWagger] | Parâmetro | Usada para explicar/descrever o _payload_ que deve ser enviado na requisição

> As anotações do Swagger Core, são utilizadas em conjunto com as anotações funcionais do Spring. **Tanto quanto possível a informação deve vir das anotações Spring**, pois elas definem comportamento e não apenas documentação.

Há diversas outras anotações no Swagger Core. Reforçamos e reiteramos a recomendação de leitura da documentação de [Swagger 2.X Annotations](https://github.com/swagger-api/swagger-core/wiki/Swagger-2.X---Annotations), para o entendimento copleto e mais adequado.

# Questões Conhecidas

# Notas e Referências

[^rest-api-specs]: Apesar de tangente a utilização de OpenAPI e Swagger UI consideramos fundamental que as APIs REST da CAPES sigam as recomendações:  https://xpto.com/dti/orientacoes-gerais/guia/blob/doc_pilha_java_2/arquitetura/rest-apis.md
[^springdoc]: https://springdoc.org/
[^1]: Esse artefato está disponível como uma das dependências seletivas de `br.gov.capes.cgs.narq:capes-arquitetura-spring-boot-web-starter`
[^obs-versao-no-archetype]: A versão já está configurado no `pom.xml` que é gerado pelo arquétipo e a tag `<version>` e a tag de propriedade podem ser suprimidas nesse caso.
[^sWagger-core-jakarta]: Estamos usando a versão com namespace `jakarta.*` em vez do namespace `javax.*`
[^openapi]: OpenAPI Initiative - https://www.openapis.org/
[^springdoc.sWagger-ui.oauth.endpoint]: Mesmo tendo definido `springdoc.capes-extensions.*` achamos mais natural e organizado adicionar a propriedade `endpoint` que aponta para a URI do SSO em `springdoc.swagger-ui.oauth.*`.
[^interface-target]: Não necessariamente todos os `ElementType` válidos para a anotação
[^spring-requestbody-javadoc]: `org.springframework.web.bind.annotation.RequestBody` - https://docs.spring.io/spring-framework/docs/5.3.12/javadoc-api/org/springframework/web/bind/annotation/RequestBody.html
[^sWagger-requestbody-javadoc]: `io.swagger.v3.oas.annotations.parameters.RequestBody` - https://docs.swagger.io/swagger-core/v2.1.10/apidocs/io/swagger/v3/oas/annotations/parameters/RequestBody.html
[^requestbody-spring-vs-sWagger]: Uma infeliz colisão de nomes, mas como o foco de integração do Swagger Core era com o JAX-RS não havia essa colisão
[^sWagger-openapidef-javadoc]: `io.swagger.v3.oas.annotations.OpenAPIDefinition` - https://docs.swagger.io/swagger-core/v2.1.10/apidocs/io/swagger/v3/oas/annotations/OpenAPIDefinition.html
[^sWagger-operation-javadoc]:`io.swagger.v3.oas.annotations.Operation` - https://docs.swagger.io/swagger-core/v2.1.10/apidocs/io/swagger/v3/oas/annotations/Operation.html
[^sWagger-apiresponse-javadoc]: `io.swagger.v3.oas.annotations.responses.ApiResponse` - https://docs.swagger.io/swagger-core/v2.1.10/apidocs/io/swagger/v3/oas/annotations/responses/ApiResponse.html
[^sWagger-parameter-javadoc]: `io.swagger.v3.oas.annotations.Parameter` - https://docs.swagger.io/swagger-core/v2.1.10/apidocs/io/swagger/v3/oas/annotations/Parameter.html
[^spring-restcontroller-javadoc]: `org.springframework.web.bind.annotation.RestController` - https://docs.spring.io/spring-framework/docs/5.3.12/javadoc-api/org/springframework/web/bind/annotation/RestController.html
[^spring-requestmapping-javadoc]: `org.springframework.web.bind.annotation.RequestMapping` - https://docs.spring.io/spring-framework/docs/5.3.12/javadoc-api/org/springframework/web/bind/annotation/RequestMapping.html
[^spring-getmapping-javadoc]: `org.springframework.web.bind.annotation.GetMapping` - https://docs.spring.io/spring-framework/docs/5.3.12/javadoc-api/org/springframework/web/bind/annotation/GetMapping.html
[^spring-postmapping-javadoc]: `org.springframework.web.bind.annotation.PostMapping` - https://docs.spring.io/spring-framework/docs/5.3.12/javadoc-api/org/springframework/web/bind/annotation/PostMapping.html
[^spring-putmapping-javadoc]: `org.springframework.web.bind.annotation.PutMapping` - https://docs.spring.io/spring-framework/docs/5.3.12/javadoc-api/org/springframework/web/bind/annotation/PutMapping.html
[^spring-deletemapping-javadoc]: `org.springframework.web.bind.annotation.DeleteMapping` - https://docs.spring.io/spring-framework/docs/5.3.12/javadoc-api/org/springframework/web/bind/annotation/DeleteMapping.html
[^spring-patchmapping-javadoc]: `org.springframework.web.bind.annotation.PatchMapping` - https://docs.spring.io/spring-framework/docs/5.3.12/javadoc-api/org/springframework/web/bind/annotation/PatchMapping.html
[^spring-requestparam-javadoc]: `org.springframework.web.bind.annotation.RequestParam` - https://docs.spring.io/spring-framework/docs/5.3.12/javadoc-api/org/springframework/web/bind/annotation/RequestParam.html
[^spring-pathvariable-javadoc]: `org.springframework.web.bind.annotation.PathVariable` - https://docs.spring.io/spring-framework/docs/5.3.12/javadoc-api/org/springframework/web/bind/annotation/PathVariable.html
[^spring-requestheader-javadoc]: `org.springframework.web.bind.annotation.RequestHeader` - https://docs.spring.io/spring-framework/docs/5.3.12/javadoc-api/org/springframework/web/bind/annotation/RequestHeader.html
[^k8s-configmap]: Kubernetes ConfigMaps - https://kubernetes.io/pt-br/docs/concepts/configuration/configmap/
[^pipeline-apps]: **Gitlab Pipeline** é a pipeline padrão para aplicações - https://xpto.com/cgs/DEVOPS/automations/gitlab-pipeline
[^chart-capes-aplic]: Helm Chart Capes Aplic - https://xpto.com/dti/orientacoes-gerais/guia/blob/master/devops/orientacoes-tecnicas/chart-capes-aplic.md
[^obs-versionamento]: Em caso de inconsistência entre as versões apontas aqui, a implementação tem precedência. Favor informar a arquitetura nesses casos: lista.arquitetura@capes.gov.br
