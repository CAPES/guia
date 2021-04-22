[[_TOC_]]

## Introdução
Um projeto permite que uma comunidade de usuários organize e gerencie
seu conteúdo isoladamente de outras comunidades.

Um Projeto do Openshift para a CAPES significa uma Aplicação em Um
determinado Ambiente, ou seja, cada aplicação deverá ter um Projeto para
cada Ambiente que a mesma é utilizada.

> Ex: A Aplicação SAP se utiliza dos ambientes de desenvolvimento,
> teste, homologação, pré produção e produção, portando ela terá os
> seguintes projetos criados:
> - sap-des
> - sap-test
> - sap-hom
> - sap-prod
> - sap-preprod

> Vide: [https://git.capes.gov.br/dti/orientacoes-gerais/guia/wikis/padrao-nomenclatura#openhsift]


## Passos para a criação de um projeto no Openshift

### 1. Logar no OpenShift
Utilize o commando `oc login` para realizar o login no cluster

```$ oc login https://openshift.capes.gov.br:443```

### 2. Criação do projeto
Crie o projeto através do comando `oc new-project` preenchendo as
seguintes variáveis:
- {{nome-projeto}}:  Nome do projeto [a-z0-9-] 
- {{Nome-projeto}}:  Nome que será exibido na aba Application Console do
  Openshift
- {{Descricao do Projeto}}:  Descrição do projeto que informa o nome da
  aplicação e em qual ambiente ela está inserida

```$ oc new-project  {{nome-projeto}} --display-name="{{Nome-projeto}}" --description="{{Descricao do Projeto}}"```

### 3. Adicionar o node-selector na namespace
Altere a namespace para adicionar os seguintes seletores:
- `environment=dht`:  Ambientes não produtivos
- `environment=prod`:  Ambientes produtivos
- `node-role.kubernetes.io/compute=true`:  Nós de computação (Workes),
  aplicações em geral
- `node-role.kubernetes.io/infra=true`:  Nós de infraestrutura, apenas
  para aplicações devops, ferramentas de arquitetura ou ferramentas do
  cluster

```$ oc edit ns {{nome-projeto}}```

Edite o arquivo aberto pelo editor colocando as informações sobre os
seletores

````yaml
metadata:
  annotations:
	openshift.io/node-selector: environment={{dht ou prod}},node-role.kubernetes.io/{{compute ou infra}}=true
````

### 4. Informações sobre a política de tráfego de acessos do namespace (EgressNetworkPolicy)
Ao término da criação do namespace automaticamente é criado o manifesto do tipo "EgressNetworkPolicy" que limita a saída dos "PODS" apenas para as redes internas (Produção, homologação, Banco de dados e etc..).
```- apiVersion: network.openshift.io/v1
  kind: EgressNetworkPolicy
  metadata:
    name: controle-saida
  spec:
    egress:
    - to:
        cidrSelector: 172.19.0.0/16
      type: Allow
    - to:
        cidrSelector: 0.0.0.0/0
      type: Deny
```
- `Não é permitido excluir o manifesto do namespace`
- `Se for necessário a alteração da regra abrir um CATI para equipe de infraestrutura`