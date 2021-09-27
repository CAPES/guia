## Introdução

Neste Guia será descrito o passo a passo de como realizar a migração de uma aplicação do Openshift OKD para a nova infraestrutura do Openshift OCP.

[Vídeo tutorial](https://drive.google.com/file/d/1wh2OgwpNtp6l2QP5ZdacGpEMugcd6Uy1/view?usp=sharing)

## Pré-requisitos

### a) Atualização para o CofreSenha 2.0

Como pré-requisito para migração ao OCP, é necessário que a aplicação já esteja utilizando a versão 2.0 do Cofre de Senhas.

Ver documentação de [implantação cofre de senha 2.0](../orientacoes-tecnicas/cofre-senhas.md)

### b) Revisão das URIs de conexão com banco de dados

Revisar nos arquivos `values-*.yaml` as _strings_ das conexões com os bancos de dados para utilizarem os valores documentados pelo time de banco de dados - [Strings de conexão](../../banco-de-dados/configuracoes/strings-de-conexao.md).

## Passo a passo de Migração

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

Solicito criação do projeto <sigla-projeto> no Openshift OCP.

Os usuários de rede abaixo devem ter acesso ao Openshift e a esse projeto.

Gerente:
- José Ricardo - joser

Equipe
- Ricardo José - ricardoj
- José Castro - josec
```
*** O nome do projeto no Openshift deve ser o mesmo nome do projeto no GIT. Enviar a sigla do projeto no Git.




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

### 7. Migração de Persistent Volumes

Responsabilidade: Time Desenvolvimento

Solicitar via chamado CATI, a criação dos PVs (Persistent Volumes) de dht e produção no Openshift OCP.

Enviar a lista dos nomes dos PVs no chamado. Os nomes podem ser vistos nos arquivos values-<des|prod>.yaml

```
app:
...
  pvc:
  - pvcname: sap-des-log-pvc
    accessMode: ReadWriteMany
    size: 1Gi
    volumeName: sap-des-log-pv
  - pvcname: sap-des-arquivos-pvc
    accessMode: ReadWriteMany
    size: 1Gi
    volumeName: sap-des-arquivos-pv

```

Exemplo de chamado:

```
Prezados,

Solicito criação dos PVs do projeto '<nome-projeto>' no Openshift OCP

Volumes a serem criados:
- sap-des-log-pv
- sap-des-arquivos-pv

```

Responsabilidade: Time Infraestrutura
- Criação dos PVs no Openshift OCP

### 8. Validar integrações e regras de firewall

Responsabilidade: Time Desenvolvimento

Após migração da aplicação, é necessário a validação de todas as integrações da aplicação. Em caso de falha, mas especificamente time out, o problema pode ser de firewall.

Para solicitar uma nova regra de firewall, deve ser aberto um chamado para o time de infraestrutura informando a ORIGEM e DESTINO da conexão.

Exemplos:

#### 8.1 Comunicação sentido [OCP >> Aplicação/Serviço externo]
```
Origem:
[Compute Nodes OCP (DHT/PROD-INT/PROD-EXT)]

Destino:
Host: xpto.capes.gov.br
Port: 80 e 443
```
#### 8.2 Comunicação sentido [Aplicação/Serviço externo >> OCP]

```
Origem:
Host: xpto.capes.gov.br

Destino:
[VIP OCP de (DHT/PROD-INT/PROD-EXT)]
```



### 9. Testar DNS em produção

Responsabilidade: Time Desenvolvimento

Para testar a aplicação em produção no Openshift OCP, basta realizar o deploy normalmente no ambiente de produção com a criação da tag.

Após o deploy, é necessário configurar o dns interno da máquina para que o o dns da aplicação será apontada para os ips do Openshift OCP.

No linux editar o arquivo `/etc/hosts` e inserir o dns da aplicação e o ip do OCP. Já no windows o arquivo é o `C:\Windows\System32\drivers\etc\hosts`.

```
...
## Para aplicações internas
172.19.208.52 xpto.capes.gov.br

## Para aplicações externas
172.19.208.60 xpto.capes.gov.br
...
```

*** Lembrar de remover a linha após os testes.

### 10. Implantar em produção

Responsabilidade: Time Desenvolvimento

Abrir chamado CATI para a infraestrutura solicitando a mudança de DNS.

```
Prezados,

Para finalizar a migração da aplicação XPTO para o Openshift OCP.

Solicito alteração do DNS abaixo.

DNS:
xpto.capes.gov.br

Destino:
[VIP OCP de (PROD-INT/PROD-EXT)]
```
