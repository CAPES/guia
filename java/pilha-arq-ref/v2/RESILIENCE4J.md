**Pilha Java para Arquitetura de Referência - versão 2.x**

# Leituras Recomendadas

- [Spring Cloud Circuit Breaker](https://spring.io/projects/spring-cloud-circuitbreaker)
  - [Configuring Resilience4J Circuit Breakers](https://docs.spring.io/spring-cloud-circuitbreaker/docs/current/reference/html/#configuring-resilience4j-circuit-breakers)
  - [Cloud Native Applications - Spring Cloud Circuit Breaker](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#spring-cloud-circuit-breaker)
- [Resilience4j  User Guide](https://resilience4j.readme.io/v1.7.0/docs)

# Introdução

Resilience4J[^r4j] é uma alternativa ao Netflix Hystrix[^hystrix]. Essa versão `2.x` da pilha Java para a Arquitetura de Referência se apoia agressivamente na concepção de que as aplicações serão conteinerizadas e orquestradas. A expectativa é que diversos aspectos tenham sido movidos para a infraestrutura da plataforma (Kubernetes/OpenShift), por isso a necessidade de implementar esses aspectos diretamente na aplicação deveria ser escassa. Nos casos onde seja preciso minúcia que não seja possível obter no escopo de solução de orquestração e conteinerização, a recomendação é utilizar o módulo de _Circuit Breaker_ do Spring Cloud.

> ## Cliente HTTP REST (Feign)
> A necessidade de implementar clientes de APIs REST deveria ser suprida com Feign e o módulo da pilha que disponibiliza os clientes Feign já está integrada com Resilience4J para prover _Circuit Breaker_[^pilha-rest-client].

# Dependências e Versionamentos

- `org.springframework.cloud:spring-cloud-starter-circuitbreaker-resilience4j`: `2.1.1`
  - `io.github.resilience4j:resilience4j-spring-boot2`: `1.7.0`
    - `io.github.resilience4j:resilience4j-spring`: `1.7.0`

# Tutoriais

## Maven

Adicionar no `pom.xml` a dependência `org.springframework.cloud:spring-cloud-starter-circuitbreaker-resilience4j` na versão adequada:

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
  <version>${spring-cloud-resilience4j.version}</version>
</dependency>
```

A propriedade `${spring-cloud-resilience4j.version}` deve ser definida criando a tag `<spring-cloud-resilience4j.version>` na tag `<properties>` do `pom.xml`. O valor dever ser o valor da versão da pilha[^obs-versao-no-archetype].

## Spring Boot

Não criamos configurações nem propriedades adicionais ao  _Spring Cloud Circuit Breaker_. Com exceção de [integrações com APIs REST, que devem ser feitas com _Feign Clients_](./RESTCLIENT.md), as configurações de Spring Boot e Spring Cloud vão obedecer o que está descrito nas **Leituras Recomendadas**, que insistimos que sejam lidas.

## Java

Enfatizamos que o uso de _Circuit Breaker_ para [integração com outras APIs REST deve ser feito utilizando _Feign Clients_](./RESTCLIENT.md). Outros casos parecem ser raros, de tal forma que vamos apenas reforçar as **Leituras Recomendadas**.

# Notas e Referências

[^r4j]: Resilience4J - https://github.com/resilience4j/resilience4j
[^obs-versao-no-archetype]: A versão já está configurado no `pom.xml` que é gerado pelo arquétipo e a tag `<version>` e a tag de propriedade podem ser suprimidas nesse caso.
[^pilha-rest-client]: Clientes REST com Feign - [Cliente (`Feign`)](./RESTCLIENT.md)
[^hystrix]: Hystrix: Latency and Fault Tolerance for Distributed Systems - https://github.com/Netflix/Hystrix
