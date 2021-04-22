## Introdução

A integracão entre o Openshift e o Cofre de Senhas é feita através de um Operator do Kubernetes.
Foi criado o chart de `cofresenha` para facilitar a utilização dessa integração.


## Configuração
Como o cofresenha é um chart template, ele deve ser utilizado como dependencia nos chart dummy específicos de cada aplicação. No arquivo `Chart.yaml`, incluir o cofresenha como dependencia da aplicação.
```yaml
dependencies:
    repository: "http://charts.paas.capes.gov.br/capes/infra"
  - name: cofresenha
    version: "0.1.0"
    repository: "http://charts.paas.capes.gov.br/capes/infra"

```

## Exemplo

Arquivo localizado em: `devops/backend|frontend/aplicacao/values-[ambiente].yaml`

```yaml
cofresenha:
  enable: true
  secretName: database-secret
  accountName: WEBAPP
  ambiente: DES
  sgbd: ORACLE
```


## Referências
- [Chart do cofre de senha](https://git.capes.gov.br/cgs/DEVOPS/helm/chart-cofresenha)
- [Operator do cofre de senha](https://git.capes.gov.br/cgs/DEVOPS/helm/chart-cofresenha-operator)