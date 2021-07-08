## Introdução

Neste Guia será descrito o passo a passo de como realizar a migração de uma aplicação do Openshift OKD para a nova infraestrutura do Openshift OCP.


### 1. Solicitação de Acesso ao OCP

Responsabilidade: Time Desenvolvimento - Gerente

- Solicitar acesso, via CATI, informando todos os usuários (login de rede) da equipe, para a fila de Infraestrutura. Caso ainda não tenha projeto no OCP, os chamados dos passos 1 e 2 podem ser os mesmos.

Responsabilidade: Time Infraestrutura

- Liberação de acesso ao Openshift OCP


### 2. Criação de projeto no OCP

Responsabilidade: Time Desenvolvimento

- Solicitar criação, via chamado, do projeto no Openshift OCP

Responsabilidade: Time Infraestrutura
- Criação do projeto no Openshift OCP;
- Criação do projeto no registry Harbor do OCP;
- Configuração das variáveis CI/CD de tokens (REGISTRY_TOKEN e REGISTRY_USER) no projeto git para acesso ao novo registry de imagens.

#### Exemplo de chamado:

```
Prezados,

Solicito criação do projeto <nome-projeto> no Openshift OCP.

Os usuários de rede abaixo devem ter acesso ao Openshift e a esse projeto.

Gerente: 
- José Ricardo - joser 

Equipe
- Ricardo José - ricardoj
- José Castro - josec 
```
*** O nome do projeto no Openshift deve ser o mesmo nome do projeto no GIT.


### 3. Alteração de pipeline

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

### 4. Apontamento de Charts

Responsabilidade: Time Desenvolvimento

No arquivo `devops/<backend|frontend>/<nome-projeto>/Chart.yaml` realizar a alteração do repositório de charts para cada chart dependente.

De: http://charts.paas.capes.gov.br/capes/infra

Para: http://charts.capes.gov.br/capes/infra

```
...
dependencies:
  - name: capes-aplic
    version: "0.6.2"
    repository: "http://charts.capes.gov.br/capes/infra"
    alias: app
...
```

### 5. Alteração de registry de imagem nos Dockerfiles

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


### 6. Alteração de dns da aplicação

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
