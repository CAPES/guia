# Introdução

Essa documentação aborda os aspectos técnicos da integracão, via API, entre o Openshift e o Cofre de Senhas que é feita através de um Operator do Kubernetes. 
Foi criado o chart de `cofresenha` para facilitar a utilização dessa integração.

## Normas de Uso
Leia sobre as [diretrizes definidas sobre o uso de cofre de senhas](/infraestrutura/seguranca/cofre-senhas.md).

<br><br>

# Como funciona

O primeiro passo é solicitar o cadastro do segredo/senha no PMP Cofre de Senhas da Capes. 
Após o cadastro do segredo, é necessário realizar a configuração do chart no repositório git do projeto.

Quando tudo estiver configurado, a integração funciona da seguinte forma:
- O operator, através das configurações inseridas no chart, se conectar na API do cofre de senhas (PMP) e buscar o segredo solicitado.
- Após encontrado o segredo, o operator cria um objeto `secret` no repositório do projeto. A secret é formada por uma combinação de `chave: valor`, sendo a chave definida nas configurações do chart e o valor é o segredo registrado no cofre de senhas.
- Com a `secret` criada, é possível acessá-la pela aplicação inserindo o seu valor em uma variável de ambiente.

<br><br>

# Configuração
Como o `cofresenha` é um chart template, ele deve ser utilizado como dependencia nos chart dummy específicos de cada aplicação. No arquivo `Chart.yaml`, incluir o `cofresenha` como dependencia da aplicação.


Caso a aplicação já esteja no OCP (Openshift 4.7+), deve ser utilizado da seguinte forma:

```yaml
dependencies:
  - name: cofresenha
    version: "2.0.0"
    repository: "http://charts.capes.gov.br/capes/infra"
```

Se a aplicação ainda esteja no OKD (Openshift 3.11), o repositório deve ser:

```yaml
dependencies:
  - name: cofresenha
    version: "2.0.0"
    repository: "http://charts.paas.capes.gov.br/capes/infra"
```


<br><br>

# Utilização

Após a configuração da dependencia do chart, para de fato utilizar a funcionalidade, é necessário inserir as seguintes configurações nos arquivos de values. 

Arquivo localizado em: `devops/backend|frontend/aplicacao/values-[ambiente].yaml`

Exemplo:
```yaml
cofresenha:
  enable: true
  secretName: database-cofre-secret
    
  secretsList:
    - name: "password"
      resourceName: "DES_ORACLE_OSHIFT"
      prefixo: "WEB"
```


| Parâmetro                                    | Descrição                                                                                    | Default                                              |
| -------------------------------------------- | -------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| `enable`                                     | Habilita a integração do cofre de senhas.                                                    | `true`                                               |
| `secretName`                                 | Nome da secret que será criada no namespace do projeto. Pode ser qualquer valor que faça sentido.               | `cofre-secret`                                          |
| `secretsList`                                | Listagem de todos os segredos a serem buscados no cofre de senhas (PMP)                            | `[]`                                                 |
| `secretsList.name`                           | Nome da key a ser criada na secret                                                                    | ` `                                                 |
| `secretsList.resourceName`                   | Nome do Resource Name cadastro no cofre de senhas                                                                    | ` `                                                 |
| `secretsList.prefixo`                        | Prefixo do User Account do cofre de senhas. Para este campo no cofre de senha, deve seguir o padrão PREFIXO_<NOME_APLICACAO>. Valores possívels: WEB e SRV                                                                     | ` `                                                 |


Para acessar o valor do segredo, no exemplo abaixo é criado a variável de ambiente `DATABASE_PASSWORD`.

```yaml
...
deployment:
...
    containers:
      environments:
      - name: DATABASE_PASSWORD
        valueFrom:
          secretKeyRef:
            name: database-cofre-secret
            key: password
...
```

<br><br>
# Referências
- [Chart do cofre de senha](https://git.capes.gov.br/cgs/DEVOPS/helm/chart-cofresenha)
- [Operator do cofre de senha](https://git.capes.gov.br/cgs/DEVOPS/helm/chart-cofresenha-operator)