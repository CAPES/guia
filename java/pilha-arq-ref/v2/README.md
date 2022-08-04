# Índice da Documentação[^todo]

- [x] [Arquétipo de Aplicação Java](ARQUETIPO.md)
- [x] [Testes Automatizados](TESTES.md)
  - [ ] Integração com o `Sonar` _Quality Gate_
- [x] [WEB/HTTP (REST)](REST.md)
  - [x] [Cliente (`Feign`)](RESTCLIENT.md)
  - [x] Erros (RFC 7807)
- Persistência
  - [x] [SQL (ORM)](PERSISTENCIA-SQL.md)
  - NoSQL
    - [ ] Elasticsearch
    - [ ] Redis
    - [ ] MongoDB
- [x] [_Logging_](LOGS.md)
- [x] [Segurança](SEGURANCA.md)
- [x] [Mensageria](MENSAGERIA.md)
- [x] [_Live Documentation_ `(OpenAPI + Swagger UI)`](SWAGGER-UI.md)
- [x] [_Production-ready Features_ (`Spring Boot Actuator`)](ACTUATOR.md)
- [x] [_Tracing_ Distribuído](TRACING-DISTRIBUIDO.md)
- _Circuit Breaker_
  - [ ] "_Black Box_"(`Istio`)
  - [x] ["_White Box_"(`Resilience4J`)](./RESILIENCE4J.md)
- [x] [_Jobs_/Rotinas](JOBS.md)
- [ ] API Gateway
- [x] [_Feature Toggles_](FEATURE-TOGGLES.md)
- [x] [Envio de E-Mails](./EMAIL.md)
- _Backend For Frontend_ (BFF)
  - [x] [API/REST Proxy (`Spring Cloud Gateway`)](./API-ROUTER.md)
- Observabilidade & Monitoramento
  - [ ] Saúde
  - [x] [APM](./APM.md)

---

# Agrupamento/Classificação

Agrupamos ou classificamos os itens da seguinte forma:

### Básico

O Básico é o mínimo de _runtime_ que precisamos para criar aplicações de _Backend_ REST.

- [^aspecto-pilha]**Arquétipo de Aplicação Java**
- [^aspecto-pilha]**Teste Automatizados**[^testes]
- [^aspecto-pilha]**WEB/HTTP (REST)**
- [^aspecto-pilha]**Persistência** (SQL)
- [^aspecto-pilha]**Segurança**
- [^aspecto-pilha] [^aspecto-arq]**Mensageria**
- [^aspecto-pilha]**REST Client** (`Feign`)
- [^aspecto-pilha] [^aspecto-arq]**_Logging_**

### Complementar

O que é Complementar são aspectos importantes ou muito úteis, mas não impeditivos para a implementação de um _Backend_ REST.

- [^aspecto-pilha]**_Live Documentation_** (`Swagger UI`)
- [^aspecto-pilha]**Production-ready Features** (`Spring Boot Actuator`)
- [^aspecto-pilha]**API/REST Proxy** (`Spring Cloud Gateway`)
- [^aspecto-arq]**_Tracing_ Distribuído** (`Zipkin`)
- [^aspecto-arq]**_Job_/Rotina** (Kubernetes `CronJob`)

### Adicional

Estão dentro do escopo que queremos alcançar com a arquitetura de referência, mas hoje já vivemos sem eles nos clusters.

- [^aspecto-pilha] [^aspecto-arq]**Persistência** (NoSQL)
  - (?) `Elasticsearch`
  - (?) `Redis`
  - (?) `MongoDB`
- **_Circuit Breaker_**
  - [^aspecto-arq]"_Black Box_" (`Istio`)
  - [^aspecto-pilha]"_White Box_" (`Resilience4J`)
- [^aspecto-arq]**CI/CD** (aprimoramentos)
- **Solução de Garantia de Qualidade** (`Sonar` Quality Gate)
- [^aspecto-arq]**`Istio`** (_Discovery Service_, _Service Mesh_, _Circuit Breaker "Black Box"_ )
- [^aspecto-arq]**API Gateway** (`Kong`)
- [^aspecto-pilha] [^aspecto-arq]**Feature Toggle**
- [^aspecto-pilha] [^aspecto-arq]**Envio de E-Mails**
- [^aspecto-arq]**Observabilidade & Monitoramento** (APM?)

# _Roadmap_

- `2.0.0-M0`: Versão preliminar minimamente estruturada com a adesão e intenção de utilização de Spring Boot 2.x
- `2.0.0-M1`: Versão **Básica**
- `2.0.0-M2`: Versão com corretivas sobre a versão `2.0.0-M1`
- `2.0.0-M3`: Versão **Complementar**
- `2.0.0-M4`: Versão com corretivas sobre a versão `2.0.0-M3`
- `2.0.0`: Versão **Adicional**

A partir da versão `2.0.0` pretendemos seguir caminho usual de SemVer[^semver].

# Notes & References

[^todo]: Estamos marcando o que já tem implementação e documentação completa
[^contextos-de-endpoints]: `/rest`, `/feeds`, `refs`, `/diagnostics`, `/manager`
[^cobertura-existe-falta-extrair]: Já existe bateria de teste, falta coletar os valores
[^problem-spring-Web]: `org.zalando:problem-spring-web` https://github.com/zalando/problem-spring-web
[^doc-so-escrita]: Documentação apenas escrita, ainda não publicada
[^pipeline-cicd]: Impacto no Pipeline de desenvolvimento de aplicações. Isso normalmente envolve alterar o[ Pipeline de aplicações](https://xpto.com/cgs/DEVOPS/automations/gitlab-pipeline) ou [Chart `capes-aplic`](https://xpto.com/cgs/DEVOPS/helm/chart-capes-aplic)
[^testes]: _Frameworks_ e infraestrutura para desenvolvimento e execução de testes automatizados.
[^antes]: antes...
[^aspecto-arq]: Aspecto de Arquitetura
[^aspecto-pilha]: Aspecto de Pilha
[^cronjob-chart]: https://xpto.com/cgs/DEVOPS/helm/cronjob
[^semver]: _Semantic Versioning_ (Versionamento Semântico) - https://semver.org/
