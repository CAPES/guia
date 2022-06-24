**Pilha Java para Arquitetura de Referência - versão 2.x**

# Leituras Recomendadas

- [Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/)

# Introdução

Algumas vezes uma aplicação pode servir de _proxy_ para outra, se espera que a aplicação um receba requisições e as direcione (_route_) para a outra. Uma dos cenários mais comuns pra isso é o caso de _Backend-for-Frontend_ (BFF)[^bff-ref1] [^bff-ref2]. Implementar o simples redirecionamento de uma aplicação para outra pode envolver implementar os métodos que expõem o _endpoint_ em uma aplicação, criar os objetos que representam os conceitos que serão transitados (_payloads_ de _request_ e de _response_), criar um mapeamento do _endpoint_ da outra aplicação[^ex-usando-feign]. Toda essa implementação tem a chance de conter pequenos erros de implementação.

> #### Cuidado
> Apesar de reduzir muito do "_boiler plate_" para o cenário de puro redirecionamento de requisições, as configurações de Spring Cloud Gateway, também são detalhes de implementação que também estão sujeitos a erro. Outro aspecto a se levar em consideração é que algumas das capacidades que se espera de um BFF como simplificar, transformar, ou agregar múltiplas requisições podem ser complexas de implementar através do sistema declarativo de YAML que o mecanismo de configuração de aplicações Spring Boot utilizam. Também pode haver casos que alguns elementos são na verdade objetos Java que devem ser implementados para estender os comportamentos do Spring Cloud Gateway.

# Dependências e Versionamentos

- Spring Cloud Gateway: `3.1.1`

# Tutoriais

## Maven

Adicionar no `pom.xml` a dependência `br.gov.capes.cgs.narq:proxy-gateway-starter` na versão adequada:

```xml
<dependency>
  <groupId>br.gov.capes.cgs.narq</groupId>
  <artifactId>proxy-gateway-starter</artifactId>
  <version>${proxy-gateway-starter.version}</version>
</dependency>
```

A propriedade `${proxy-gateway-starter.version}` deve ser definida criando a tag `<proxy-gateway-starter.version>` na tag `<properties>` do `pom.xml`. O valor dever ser o valor da versão da pilha[^obs-versao-no-archetype].

### Swagger UI

O módulo `br.gov.capes.cgs.narq:swaggerui-starter` não é compatível com o módulo `br.gov.capes.cgs.narq:proxy-gateway-starter`. Para obter o mesmo comportamente é preciso utilizar o mótulo alternativo `br.gov.capes.cgs.narq:swaggerui-reactive-starter`:

```xml
<dependency>
  <groupId>br.gov.capes.cgs.narq</groupId>
  <artifactId>swaggerui-reactive-starter</artifactId>
  <version>${swaggerui-reactive-starter.version}</version>
</dependency>
```

A propriedade `${swaggerui-reactive-starter.version}` deve ser definida criando a tag `<swaggerui-reactive-starter.version>` na tag `<properties>` do `pom.xml`. O valor dever ser o valor da versão da pilha[^obs-versao-no-archetype].

## Spring Boot

Insistimos na recomendação de leitura da [documentação de referência do Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/) para as diversas capacidades e minúcias do projeto. Vamos dar um exemplo simples de como mapear _endpoints_ REST.

- `http://stuff.capes.gov.br/rest/local-stuff` retorna uma lista de "Stuff";
- `https://stuff.capes.gov.br/rest/local-stuff/{id}` retorna a "Stuff" de ID igual a `{id}`.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: list-of-stuff
        uri: http://stuff.capes.gov.br:8080
        predicates:
        - name: Path
          args:
            patterns:
            - "/rest/stuff"
          matchTrailingSlash: false
        filters:
        - RewritePath=/rest/stuff, /rest/local-stuff
      - id: get-stuff
        uri: http://stuff.capes.gov.br:8080
        predicates:
        - name: Path
          args:
            patterns:
            - "/rest/stuff/{segment}"
          matchTrailingSlash: false
        filters:
        - RewritePath=/rest/stuff/?(?<segment>.*), /rest/local-stuff/$\{segment}
```

Para funcionar corretamente é necessário que a aplicação seja configurada como _reactive_, através da propriedade `spring.main.web-application-type`, mas essa propriedade já é pré-configurada pelo módulo `br.gov.capes.cgs.narq:proxy-gateway-starter`.

> ### URIs de Services do K8s são preferíveis
> Onde possível, é preferível mapear `spring.cloud.gateway.routes[*].uri` para URIs de Services do Kubernetes[^k8s-services-uris]. Isso torna as chamadas mais ágeis, evitando saltos de rede desnecessários. É particularmente verdade se o Spring Cloud Gateway (pelo módulo `br.gov.capes.cgs.narq:swaggerui-starter`) estiver sendo utilizado para implementar um BFF.

## Java

A expectativa geral é que não seja necessário realizar implementação Java para lidar com os mapeamentos. Para saber das possibilidades insistimos na recomendação da leitura da [documentação do Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/).

# Questões Conhecidas

- O **Spring Cloud Gateway** utiliza o **WebFlux**[^spring-Webflux], que utiliza a abordagem `REACTIVE`, o que torna a utilização em conjunto com alguns outros componentes limitada
  - O **Spring Cloud OpenFeign**[^feign-starter] só tem suporte MVC (`SERVLET`) até o momento[^hoje].

> #### Spring Cloud Gateway e Spring Cloud OpenFeign são incompatíveis
> Até o momento não temos uma solução para a utilização conjunta das duas tecnologias, por isso a recomendação que damos é escolher apenas _Spring Cloud Gateway_ ou apenas _Spring Cloud OpenFeign_.

- As rotas definidas no Spring Cloud Gateway **não são mapeadas** para o Swagger UI disponibilizado por `br.gov.capes.cgs.narq:swaggerui-starter`[^sWaggerui-starter], nem por `br.gov.capes.cgs.narq:swaggerui-reactive-starter`.

# Notas e Referências
[^bff-ref1]: Pattern: Backends For Frontends - https://samnewman.io/patterns/architectural/bff/
[^bff-ref2]: Pattern: API Gateway / Backends for Frontends - https://microservices.io/patterns/apigateway.html
[^ex-usando-feign]: Por exemplo usando um _client feign_ - [REST Client](./RESTCLIENT.md)
[^spring-Webflux]: Spring WebFlux - https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux
[^hoje]: 2022-04-05
[^sWaggerui-starter]: Veja [_Live Documentation_ através de OpenAPI + `Swagger UI`](SWAGGER-UI.md)
[^feign-starter]: Provido através da dependência `br.gov.capes.cgs.narq:restclient-starter` - [Cliente REST](RESTCLIENT.md)
[^obs-versao-no-archetype]: A versão já está configurado no `pom.xml` que é gerado pelo arquétipo e a tag `<version>` e a tag de propriedade podem ser suprimidas nesse caso.
[^k8s-services-uris]: O formato indicado pela documentação do OpenSHift é `<service>.<pod_namespace>.svc.cluster.local` - https://docs.openshift.com/container-platform/3.11/architecture/networking/networking.html - mas os pods costumam ter configurações de resolução de rede em que basta `<service>.<pod_namespace>.svc` como formato de URI do Service - https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#namespaces-of-services
