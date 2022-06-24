**Pilha Java para Arquitetura de Referência - versão 2.x**

# Leituras Recomendadas

- [Spring Boot Actuator][d08669b5]

  [d08669b5]: https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html "Spring Boot Actuator: Production-ready Features"

# Introdução

Existem diversos aspectos que uma aplicação em produção tem de lidar além de suas funcionalidades negociais. O Spring Boot disponibiliza uma diversidade de capacidades informacionais e adminstrativas comuns através do módulo `org.springframework.boot:spring-boot-starter-actuator`. É interessante que as aplicações façam uso dessas funcionalidades, principalmente em lugar de implementar sua própria versão de tais funcionalidades não negociais.

> As funcionalidades do Actuator são administrativas e sensíveis, por isso **é fundamental levar em consideração as configurações recomendadas para evitar que esses recursos fiquem publicamente acessíveis**.

# Dependências e Versionamentos

- `org.springframework.boot:spring-boot-starter-actuator`: `2.6.4`

# Tutoriais

## Maven

Adicionar no `pom.xml` a dependência `org.springframework.boot:spring-boot-starter-actuator` na versão adequada:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
  <version>${springboot-actuator.version}</version>
</dependency>
```

A propriedade `${springboot-actuator.version}` deve ser definida criando a tag `<springboot-actuator.version>` na tag `<properties>` do `pom.xml`. O valor dever ser o valor da versão compatível com a pilha[^obs-versao-no-archetype].

## Spring Boot

- Porta Alternativa
- Path Alternativo
- Swagger UI
  - Ativação da integração
  - CORS
- Liveness/Readiness probes para K8s

### Porta Alternativa

Por padrão os endpoints web do Spring Boot Actuator rodam na mesma porta da aplicação (`server.port`), em geral o melhor é colocar essas funcionalidades em outra porta, recomendamos a porta `8089`[^porta8089]. Conforme a documentação do Spring Boot Actuator para modificar a porta é preciso configurar a propriedade `management.server.port`

### Path Alternativo

Por padrão os endpoints web do Spring Boot Actuator são mapeados no (sub)path `/actuator`. Recomendamos modificar para `/manager`. A propriedade que configura esse mapeamento é `management.endpoints.web.base-path`

### Integração com Swagger UI via `springdoc-openapi`[^springdoc-actuator]

Caso o módulo `br.gov.capes.cgs.narq:swaggerui-starter` tenha sido incluído como dependência no `pom.xml` do projeto é possível visualizar e manipular os endpoints do Spring Boot Actuator através do Swagger UI. Algumas configurações são necessárias.

#### Ativação da integração

Para que o `springdoc-openapi` mapeie os endpoints é preciso ativar a propriedade `springdoc.show-actuator`.

> Quando `springdoc.show-actuator=true` o grupo `x-actuator` fica listado nos Swagger UI.

#### CORS

Seguindo a recomendação de disponibilizar os _endpoints_ do Spring Boot Actuator na porta `8089`, é necessário configurar o CORS para que o Swagger UI consiga realizar requisições. A propriedade `management.endpoints.web.cors.allowed-origins` deve ser configurada com protocolo e autority adequados.

> ##### localhost
> O localhost costuma ser o hostname padrão que se utiliza durante o desenvolvimento local. Levando em conta que os _endpoints_  funcionais estejam mapeados na porta `8080` (`server.port=8080` por padrão), o `management.endpoints.web.cors.allowed-origins` teria o valor `http://localhost:8089`
>
> ##### Kubernetes (OKD, OCP, K8s)
> Quando a aplicação é implantada no _cluster_ Kubernetes ele normalmente fica disponível num hostname, mas na porta `80` (HTTP), ou `443` (HTTPS). Isso deve ser levado em conta na hora de atribuir o `management.endpoints.web.cors.allowed-origins`.
> Outro aspecto a se levar em conta sobre a implantação em Kubernetes é de que pode ser necessário mapear Service e Route/Ingress para que a porta seja acessível e as requisições possam ser realizadas.

Também é necessário indicar quais os verbos HTTP devem ser permitidos via CORS. A propriedade que configura isso é `management.endpoints.web.cors.allowed-methods`. Uma configuração básica é habilitar apenas `GET`, mas dependendo dos endpoints configurados em `management.endpoints.web.exposure.include`, algumas funcionalidades podem precisar de outros verbos habilitados. A limitação dos verbos pode também ser uma savaguarda contra exploração indevida da aplicação[^exploit].

---

**Exemplo**[^application.yml]

```yaml
management:
  server:
    port: 8089
  endpoints:
    web:
      base-path: /manager
      cors:
        allowed-origins: http://localhost:8089
        allowed-methods: GET

springdoc:
  show-actuator: true
```

> A documentação de [Spring Boot Actuator][d08669b5] é fundamental para saber quais os _endpoints_ que se deseja ativos na aplicação e como fazer[^springboot-actuator-endpoints] (propriedade `management.endpoints.web.exposure.include`).

---

### Liveness/Readiness probes para K8s

O Actuator dispõe de _endpoints_ HTTP que podem ser utilizados pelos probes de containers do Kubernetes[^springboot-actuator-k8s-probes].

**Exemplo**[^exeplo-values-yaml]

```yaml
      livenessProbe:
        httpGet:
          path: "/manager/health/liveness"
          port: 8089
        failureThreshold: 3
        periodSeconds: 10

      readinessProbe:
        httpGet:
          path: "/manager/health/readiness"
          port: 8089
        failureThreshold: 3
        periodSeconds: 10
```

> O exemplo acima está seguindo as recomendações de porta e path.

## Java

# Notas e Referências

[^porta8089]: A porta 8089 é uma porta não atribuida no registro da IANA - https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.txt - visitado em 2022-03-22
[^springboot-actuator-k8s-probes]: 2.9. Kubernetes Probes - https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.kubernetes-probes
[^springboot-actuator-endpoints]: 2.2. Exposing Endpoints - https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.exposing
[^exeplo-values-yaml]: É um exemplo do que poderia ser adicionado nos arquivos `values-*.yaml` que configuram o deploy da aplicação através do [Helm Chart CAPES-Aplic](https://git.capes.gov.br/cgs/DEVOPS/helm/chart-capes-aplic)
[^exploit]: Ataques, como de DDoS, ou abusos, como coleta de dados sensíveis, são exemplos de exploração indevida das aplicações.
[^obs-versao-no-archetype]: A versão já está configurado no `pom.xml` que é gerado pelo arquétipo e a tag `<version>` e a tag de propriedade podem ser suprimidas nesse caso.
[^springdoc-actuator]: Actuator support - https://springdoc.org/#actuator-support
[^application.yml]: Configuração que fica no `application.yml`, normalmente definido como [ConfigMap][d74f04b0] nos `values-*.yaml`.

  [d74f04b0]: https://kubernetes.io/docs/concepts/configuration/configmap/ "ConfigMaps"
