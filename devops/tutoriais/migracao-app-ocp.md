## Introdução

Neste Guia será descrito o passo a passo de como realizar a migração de uma aplicação do Openshift OKD para a nova infraestrutura do Openshift OCP.


### 1. Criação de projeto no OCP

Responsabilidade: Time Desenvolvimento / Time GCM

Realizar a solicitação da criação do projeto no Openshift OCP. Essa solicitação deve ser feita para o time de GCM. Com essa solicitação, será criado o projeto no Openshift, o projeto no registry Harbor do OCP além de reconfigurar as variáveis de tokens (REGISTRY_TOKEN e REGISTRY_USER) no projeto git para acesso ao novo registry de imagens.


### 2. Alteração de pipeline

Responsabilidade: Time Desenvolvimento

No arquivo `devops/gitlab-ci.yaml` do projeto git, deve ser realizado o apontamento para a pipeline do OCP. Para isso, basta incluir, em todos os jobs, a linha `ref: ocp`, conforme exemplo abaixo. Com isso, o projeto git já estará utilizando a pipeline de deploy do OCP.

```
include:
  - project: 'cgs/DEVOPS/automations/gitlab-pipeline'
    file: '.gitlab-ci.yaml'
    ref: ocp
  - project: 'cgs/DEVOPS/automations/gitlab-pipeline'
    file: 'build.gitlab-ci.yaml'
    ref: ocp
```

### 3. Apontamento de Charts

Responsabilidade: Time Desenvolvimento

No arquivo `devops/<backend|frontend>/<nome-projeto>/Chart.yaml` realizar a alteração do repositório de charts para cada chart dependente.

De: http://charts.apps.ocp.capes.gov.br/capes/infra

Para: http://charts.capes.gov.br/capes/infra

```
...
dependencies:
  - name: capes-aplic
    version: "0.6.1"
    repository: "http://charts.capes.gov.br/capes/infra"
    alias: app
...
```

### 4. Alteração de registry de imagem nos Dockerfiles

Responsabilidade: Time Desenvolvimento

Em todas as camadas da aplicação (backend|frontend), alterar os Dockerfiles realizando a mudança do dns do registry de imagens. Qualquer apontamento deve ser feito para o dns `registry.capes.gov.br`.

Exemplo:
```
FROM registry.capes.gov.br/base/composer:1.0.0 as vendor
RUN composer install
...
FROM registry.capes.gov.br/replicas/php:7.3-cli
WORKDIR /sistema
...
```


### 5. Alteração de dns da aplicação

Responsabilidade: Time Desenvolvimento

Nos values-<des|prod>.yaml de configuração de deploy, realizar a alteração dos dns de DHT para o wildcard `*.dht.ocp.capes.gov.br`.

Exemplo:
```
...
route:
  - name: route-app-sample
    enable: true
    hostname: app-sample-des.dht.ocp.capes.gov.br
    servicePort: http
    path: /
    tls:
      insecureEdgeTerminationPolicy: Redirect
      termination: edge
...
```