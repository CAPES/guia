**Pilha Java para Arquitetura de Referência - versão 2.x**

# Leituras Recomendadas

- [Apêndice 1 - Permissionamento & Segurança nas Aplicações de _Backend_ na Arquitetura de Referência da CGS](./APENDICES/SEGURANCA1.md)

# Introdução

_Segurança_ é uma palavra que costuma ser sobrecarregada de significados. No contexto dessa documentação queremos dar ênfase principalmente na capacidade de restringir numa aplicação o acesso a determinados dados e operações a usuárias autênticadas e devidamente autorizadas. Muitas vezes isso é chamado de "_permissionamento_".

# Dependências e Versionamentos

- **Tecnologias & Padrões**
  - OAuth: `2.0`
  - JWT (Json Web Token)[^jWt]
- **Framework**: Spring Security (`5.5.3`)

# _Design_ da Solução

As requisições feitas as aplicações Java devem ser feitas com o cabeçalho HTTP `Authorization` do tipo `Bearer` carregando um **JWT** compacto. Esse JWT é emitido pelo SSO da CAPES[^sso-capes] através de fluxos de OAuth 2.

# Tutoriais

> Não faz parte do escopo desse tópico de documentação explicar ou recomendar como deve ser feita a integração com o SSO ou como deve ser mantido e utilizado o JWT pelas clientes de aplicações Java baseadas na versão 2.x da pilha.
> Uma exceção é que explicaremos como configurar para integrar Swagger-UI disponibilizado pelo módulo `swagger`.

## Maven

Adicionar o módulo `br.gov.capes.cgs.narq:seguranca-starter` como dependência do projeto:

```xml
<dependency>
    <groupId>br.gov.capes.cgs.narq</groupId>
    <artifactId>seguranca-starter</artifactId>
    <version>${seguranca-starter.version}</version>
</dependency>
```

A propriedade `${seguranca-starter.version}` deve ser definida criando a tag `<seguranca-starter.version>` na tag `<properties>` do `pom.xml`. O valor dever ser o valor da versão da pilha[^obs-versao-no-archetype].

## Configurações Spring Boot

Conforme `br.gov.capes.cgs.narq:seguranca-starter` é adicionado ao `pom.xml` da aplicação é necessário configurar sua utilização:

- `spring.security.oauth2.resourceserver.jwt.public-key-location`: caminho para localização do arquivo com a chave pública do SSO - **obrigatório**
  - Esse arquivo pode até estar disponível no classpath da aplicação (ex.: `classpath:sso-capes.pub`), mas recomendamos que seja um arquivo montado no pod de acordo com o ambiente do SSO que se espera
  - Essa configuração não aceita um certificado como era o caso de `seguranca.certificado-sso` na versão 1.x
- `spring.security.oauth2.resourceserver.jwt.jws-algorithm`: o algoritmo de criptografia utilizado. No caso o SSO da CAPES utiliza `RS512`, então esse valor que deve ser configurado - **obrigatório**

### Extensões da CAPES

- `spring.security.capes-extensions.enabled`: para desabilitar as configurações esse valor pode ser definido como `false`
- `spring.security.capes-extensions.sigla-aplicacao`: A sigla da aplicação conforme no Segurança (normalmente é a sigla no Catálogo de Sistemas) - **obrigatório**
- `spring.security.capes-extensions.endpoint`: A URI do endpoint do Web Service do Segurança - **obrigatório**

**Exemplo no `application.yml`**[^app-yaml-deve-ser-configmap] da aplicação
```yaml
spring:
   security:
      capes-extensions:
         endpoint: https://des.capes.gov.br/seguranca-services/SegurancaService?wsdl
         sigla-aplicacao: APP
      oauth2:
         resourceserver:
            jwt:
               jws-algorithm: RS512
               public-key-location: classpath:sso-capes.pub
```

## Java

### Anotações

- Anotações para acessar dados do JWT
- Anotações para controlar acesso/execução de métodos

> As anotações `br.gov.capes.servicos.utils.seguranca.RequerLogin` e `br.gov.capes.servicos.utils.seguranca.RequerPermissao` foram removidas em favor das anotações compatíveis com Spring Security.

#### Anotações para acessar dados do JWT

Existem algumas anotações que podem ser utilizadas para que os dados do JWT sejam carregados em referências. Os dados do JWT emito pelo SSO devem ficar no `Bearer` do cabeçalho `Authorization`. A aplicação cliente, como, por exemplo, uma aplicação HTML/JS/CSS ou outro serviço, deve ser responsável por mandar esses cabeçalhos devidamente preenchidos.

> O SSO emite um JWT através de um de seus fluxo de autenticação.

##### `@AuthenticationPrincipal`

A anotação `AuthenticationPrincipal`[^javadoc-auth-principal] pode ser utilizada para que o Spring injete uma instância de `Jwt`[^javadoc-jWt] na referência anotada. O objeto `Jwt` dá acesso ao conteúdo do **JWT** recebido. Conforme definido pelo SSO da CAPES o login está no  _claim_[^jWt-claims] `sub` ("_Subject_")[^iana-jWt-claims]

```java
import org.springframework.http.ResponseEntity;
import org.springframework.http.ResponseEntity.BodyBuilder;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping(path = "/rest/exemplo")
public class ExemploEndpoint {

  @GetMapping
  public ResponseEntity<?> get(@AuthenticationPrincipal Jwt jwt) {
    BodyBuilder builder = ResponseEntity.ok();
    //...
    return builder.build();
  }

}
```
##### `@JsonWebToken`

Caso queira injetar um objeto `Jwt` como atributo de um _bean_, em vez de utilizar `@AuthenticationPrincipal`, utilizar a anotação de conveniência `@br.gov.capes.cgs.narq.seguranca.JsonWebToken`:

```java
import br.gov.capes.cgs.narq.seguranca.JsonWebToken;

import org.springframework.security.oauth2.jwt.Jwt;

    //...

    @JsonWebToken
    private Jwt jwt;

    //...
```

> O `@AuthenticationPrincipal` não é permitida em atributos de classe. Por conta da natureza do "Escopo de _Request_" do JWT no cabeçalho de `Authorization`, é mais adequado receber como parâmetro do método em vez de injetar como atributo de classe. Pode ser tornar moroso ter de sempre receber como parâmetro por isso a anotação de conveniência `@JsonWebToken`.

##### `@Login`

A única informação normalmente relevante para aplicações no JWT emitido pelo SSO da CAPES é o _Subject_(`sub`) que contém o _login_ utilizado para autenticar. Esse é um dado da base de dados corporativa[^use-servicos-corporativos] e pode ser utilizada para consultar informações associadas a pessoa. Para obter essa informação diretamente como parâmetro de um método num _bean_ criamos a anotação de conveniência `@br.gov.capes.cgs.narq.seguranca.JsonWebToken.Login`.

```java
import org.springframework.web.bind.annotation.GetMapping;

import br.gov.capes.cgs.narq.seguranca.JsonWebToken.Login;

  //..

    @GetMapping
    public String get(@Login String username) {
      //..
    }

    //..

```

---

> #### `@AuthenticationPrincipal`, `@JsonWebToken` e `@Login` em requisições não autenticadas
> Caso o JWT não esteja no Bearer do cabeçalho `Authorization` comportamentos inconsistentes podem ocorrer, por isso é conveniente que endpoints que processam requisições autenticadas, não sejam reutilizados para requisições sem autenticação[^use-anotacaoes-de-auth].
> No caso de não haver autenticação conforme descrita:
> - O parâmetro anotado com `@AuthenticationPrincipal` resolve para `null`;
> - O atributo anotado com `@JsonWebToken` continua "não `null`", mas praticamente todos os seus métodos retornam `null`[^vide-NenhumJWT];
> - A tentativa de injeção no parâmetro anotado com `@Login` falha e resulta num erro inesperado.

---

### Anotações para controlar acesso/execução de métodos

Essas anotações podem ser colocadas em métodos para controlar a execução deles diante de determinadas regras específicas como a presença de determinadas autorizações, ou a pertinência a determinados grupos.

#### `@PreAuthorize` e `@PosAuthorize`

As anotações `@PreAuthorize`[^javadoc-preauth] e `@PosAuthorize`[^javadoc-postauth] podem ser utilizadas para verificar se requisitos de permissionamento estão satisfeitos antes de retornar uma resposta para a cliente.

> Por conveniência criamos uma meta anotação `@br.gov.capes.cgs.narq.IsAuthenticated` que faz `@PreAuthorize("isAuthenticated()")`:
>
> ---
> ```java
> import org.springframework.http.ResponseEntity;
> import org.springframework.http.ResponseEntity.BodyBuilder;
> import org.springframework.security.access.prepost.PreAuthorize;
> import org.springframework.security.core.annotation.AuthenticationPrincipal;
> import org.springframework.security.oauth2.jwt.Jwt;
> import org.springframework.web.bind.annotation.GetMapping;
> import org.springframework.web.bind.annotation.RequestMapping;
> import org.springframework.web.bind.annotation.RestController;
>
> import br.gov.capes.narq.security.IsAuthenticated;
>
> @RestController
> @RequestMapping(path = "/rest/exemplo")
> public class ExemploEndpoint {
>
>   @GetMapping
>   @IsAuthenticated
>   public ResponseEntity<?> get(@AuthenticationPrincipal Jwt jwt) {
>     BodyBuilder builder = ResponseEntity.ok();
>     //...
>     return builder.build();
>   }
>
> }
> ```
> ---



##### _Roles_ (Papeis), _Scopes_, Permissões e `GrantedAuthority`

O **Segurança CAPES** define um sistema de RBAC[^rbac]. A ideia de RBAC é que as permissões de acesso não seja associadas com usuários, mas com _Roles_ (Perfis). Um Perfil está associado com uma coleção de **Permissões**. Essas permissões podem estar associadas a **Objetos** (não é o conceito de Objetos de Orientação a Objetos) e **Operações**. No mundo REST essa associação é naturalmente mapeável para Endpoints e Verbos HTTP. É fácil imaginar que uma determinada Permissão de `GET` no _endpoint_ `/rest/exemplo` e uma outra de `POST` nesse mesmo _endpoint_ `/rest/exemplo`. São duas permissões que podem ser dadas alternativa ou simultaneamente a diversos Perfis.

A implementação do **Segurança CAPES** desvia na nomenclatura, utilizando `Grupo` em lugar de `Role`, e `Transacao` em vez de `Permission`.

Não existe um formato padronizado para informar _roles_ e _permissions_ de RBAC nos _Claims_ de JWT. O JWT emitido pelo SSO da CAPES hoje em dia[^hoje-em-dia] nem tem o _claim_ `scope`. É importante notar que _Scopes_ de OAuth e os elementos de RBAC não tem um mapeamento natural, nem padronizado[^vide-apendice1].

O Spring Security utiliza uma abstração[^spring-security-grantedauthority] para lidar com o esquema de permissionamento: `Authentication`[^javadoc-authentication] tem uma coleção de `GrantedAuthority`[^javadoc-grantedauthority]. Já há algumas convenções adotadas para o valor de `getAuthority()` da lista _authorities_ de `Authentication` no Spring Boot 2:
- _Roles_ tem o prefixo `ROLE_`;
- _Scopes_ vindos do JWT ficam com o prefixo `SCOPE_`.

Convencionamos para as permissões derivadas das transações do Segurança CAPES o formato **`$SIGLA_APP`::`$COD_CASO_USO`::`$DESC_TRANSACAO`** onde:
- `$SIGLA_APP` é a sigla da aplicação conforme em `spring.security.capes-extensions.sigla-aplicacao`;
- `$COD_CASO_USO` é o código do Caso de Uso conforme cadastrado no **Segurança CAPES**[^relacional-casouso];
- `$DESC_TRANSACAO` é a descrição da Transação conforme cadastrado no **Segurança CAPES**[^relacional-transacao].

Criamos duas implementações de `GrantedAuthority`:
- `RoleAuthority` para mapear os Grupos do **Segurança CAPES** em _Roles_;
- `AutorizacaoAuthority` para mapear as Transações do **Segurança CAPES**.

Elas implementam as convenções mencionadas, para _scopes_ apenas reutilizamos o mapeamento padrão do Spring Security/Spring Boot, que os mapeia para `SimpleGrantedAuthority`.

Como os `RoleAuthority` seguem a convenção de nomeclatura de `getAuthority()` as expressões `hasRole(String role)` e `hasAnyRole(String…​ roles)` podem ser utilizadas sem o prefixo `ROLE_`[^spring-security-el-common-built-in].

# Notas e Referências

[^jWt]: RFC 7519 - JSON Web Token (JWT) - https://datatracker.ietf.org/doc/html/rfc7519
[^sso-capes]: SSO da CAPES - https://sso.capes.gov.br
[^obs-versao-no-archetype]: A versão já está configurado no `pom.xml` que é gerado pelo arquétipo e a tag `<version>` e a tag de propriedade podem ser suprimidas nesse caso.
[^app-yaml-deve-ser-configmap]: Esses valores devem ser devidamente configurados em `values-*.yaml` como ConfigMaps - https://docs.openshift.com/container-platform/4.7/nodes/pods/nodes-pods-configmaps.html
[^javadoc-jWt]: Javadoc - org.springframework.security.oauth2.jwt.Jwt - https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/oauth2/jwt/Jwt.html
[^jWt-claims]: JWT Claims - RFC 7519 - https://datatracker.ietf.org/doc/html/rfc7519#section-4
[^iana-jWt-claims]: JSON Web Token Claims - IANA - https://www.iana.org/assignments/jwt/jwt.xhtml
[^javadoc-preauth]: Javadoc - `org.springframework.security.access.prepost.PreAuthorize` - https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/access/prepost/PreAuthorize.html
[^javadoc-postauth]: Javadoc - `org.springframework.security.access.prepost.PostAuthorize` - https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/access/prepost/PostAuthorize.html
[^rbac]: Role-Based Access Control (Controle de Acesso Baseado em Perfil)
[^hoje-em-dia]: 2021-11-23
[^spring-security-grantedauthority]: https://www.baeldung.com/spring-security-granted-authority-vs-role#grantedauthority
[^javadoc-grantedauthority]: Javadoc - `org.springframework.security.core.GrantedAuthority` - https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/core/GrantedAuthority.html
[^javadoc-authentication]: Javadoc - `org.springframework.security.core.Authentication` - https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/core/Authentication.html
[^relacional-casouso]: Coluna `CO_CASO_USO` na tabela `SEGURANCA.CASO_USO`
[^relacional-transacao]: Coluna `DS_TRANSACAO` na tabela `SEGURANCA.TRANSACAO`
[^spring-security-el-common-built-in]: https://docs.spring.io/spring-security/site/docs/current/reference/html5/#el-common-built-in
[^vide-apendice1]: Vide [Apêndice 1 - Permissionamento & Segurança nas Aplicações de _Backend_ na Arquitetura de Referência da CGS](./APENDICES/SEGURANCA1.md)
[^javadoc-auth-principal]: Javadoc - org.springframework.security.core.annotation.AuthenticationPrincipal - https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/core/annotation/AuthenticationPrincipal.html
[^use-servicos-corporativos]: Utilize serviços corporativos como o **Cadastro de Pessoas** para obter dados corporatívos, mas **não** implemente aplicações dependendo de acesso direto a base de dados do "`CORPORATIVO`".
[^use-anotacaoes-de-auth]: User anotações como as mencionadas, `@@IsAuthenticated`, `@PreAuthorize`
[^vide-NenhumJWT]: Vide `br.gov.capes.cgs.narq.seguranca.NenhumJt`
