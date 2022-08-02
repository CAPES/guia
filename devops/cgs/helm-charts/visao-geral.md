# Helm Charts


## O que é o Helm?

O Helm é uma ferramenta utilizada para definir, instalar e atualizar aplicações em cluster Kubernetes. O Helm usa um formato de empacotamento chamado `chart`.

Um `chart` é uma pasta que contém uma coleção de arquivos que descrevem um conjunto relacionado de objetos do Kubernetes. Um único chart pode ser usado para implantar algo simples, como um pod memcached, ou algo complexo, como uma pilha completa de aplicativos web com servidores HTTP, bancos de dados, caches e assim por diante.

> Para mais informações sobre Helm Charts, consulte a [documentação oficial](https://helm.sh/docs/).

## Utilizando Charts

Para a implantação de aplicações no ambiente Openshift na Capes, utiliza-se o conceitos de `chart dummy`, onde o chart da aplicação não utiliza nenhum tipo de template, somente charts dependentes.

Estrutura de um chart dummy:

```
└── aplicacao-xpto/
    ├── Chart.yaml              # Arquivo yaml que contém informações sobre o chart e suas dependencias
    ├── values.yaml             # OPCIONAL: Arquivo de configurações default para todos os ambientes
    ├── values-<ambiente>.yaml  # Configurações específicas por ambiente
```

### Utilizando um chart como dependencia

Um `chart dummy` funciona a base de outros charts como dependencia, ou seja, ele é um chart que utiliza outros charts para subir determinados recursos ou aplicações.

Exemplo de um chart dummy de uma aplicação `xpto` utilizando a versão `1.2.0` do chart `dep-chart`:

`chart.yaml`
```
apiVersion: v2
appVersion: "1.0.0"
description: Chart da aplicação xpto
name: xpto-backend
version: 1.0.0
dependencies:
  - name: dep-chart        # Nome do chart dependente, é utilizando como atributo chave nos arquivos values
    version: "1.2.0"       # Número da versão do chart
    alias: dep-chart       # OPCIONAL: Apelido para o chart, caso exista, substitue o nome do chart nos values
    repository: "http://charts.capes.gov.br/capes/infra"    

```

`values.yaml`
```
dep-chart:
    atributo1: "valor1"
    atributo2: 
        subatributo: "xyz"
    atributo3: "valor3"
```



## Principais Charts

Chart | Descrição
----- | -----------
[capes-aplic](https://xpto.com/cgs/DEVOPS/helm/chart-capes-aplic) | É o chart template padrão de aplicações em container da Capes.
[cofre-senha](https://xpto.com/cgs/DEVOPS/helm/chart-cofresenha) | Chart de integração entre o Openshift OCP e o Cofre de Senhas (Vault Hashicorp)
[cronjob](https://xpto.com/cgs/DEVOPS/helm/cronjob) | Agendador de tarefas/jobs. Um job cria um ou mais Pods para realizar determinadas tarefas.
[memcached](https://xpto.com/cgs/DEVOPS/helm/memcached) | Chart para utilização do memcached em aplicações PHP