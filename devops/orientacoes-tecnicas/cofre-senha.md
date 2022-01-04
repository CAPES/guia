# Introdução

Essa documentação aborda os aspectos técnicos da integração, via API, entre o Openshift e o Cofre de Senhas (ferramenta Vault da Hashicorp) que é feita através de um [Operator do Kubernetes](https://git.capes.gov.br/cgs/DEVOPS/helm/chart-cofresenha-operator).
Foi criado o [*chart*](https://git.capes.gov.br/cgs/DEVOPS/helm/chart-cofresenha) nomeado de `cofresenha` para facilitar a utilização dessa integração.

<br>

## Norma de Uso
Leia sobre as [diretrizes definidas sobre o uso de Cofre de Senhas](/infraestrutura/seguranca/cofre-senhas.md).

<br>

# Como funciona

1. Solicitar via chamado (CATI) o cadastro do segredo (senha, token, certificado...) no Cofre de Senhas da CAPES, caso ainda não tenha sido.
> **IMPORTANTE**: Jamais informe os segredos no chamado.
2. Após o cadastro do segredo, é necessário realizar a [configuração](#configuração) do *chart* no repositório Git do seu projeto.

Em resumo, quando tudo estiver configurado, a integração funcionará da seguinte forma:
- O *operator*, através das configurações inseridas no *chart*, se conectará via API do Cofre de Senhas em busca do segredo solicitado (senha, token, certificado...).
- Após encontrado o segredo, o *operator* cria um objeto do tipo `secret` no *namespace* da aplicação no Openshift. A `secret` é formada por uma combinação de `chave:valor`, sendo a **chave** definida nas configurações do *chart* e o **valor** é o segredo previamente cadastrado no Cofre de Senhas.
- Com o objeto `secret` criado, é possível acessá-la pela aplicação inserindo o seu valor em uma variável de ambiente.

<br>

# Configuração

Como o `cofresenha` é um *chart* modelo, ele deve ser utilizado como dependencia nos *chart dummy* específicos de cada aplicação. No git do seu projeto, no arquivo `Chart.yaml`, deve-se incluir o *chart* `cofresenha` como dependencia da aplicação, como mostrado abaixo:

```yaml
dependencies:
  - name: cofresenha
    version: "3.0.0"
    repository: "http://charts.capes.gov.br/capes/infra"
```

## Pré-requisitos

### Aplicações Java 8 (ou superior)

Aplicações em Java 8 ou superior podem utilizar a biblioteca ojdbc8 (biblioteca de JDBC da Oracle, compilada com major version 52). Isso pode contornar o problema de incompatibilidade com **usernames com mais de 30 caracteres** que pode ocorrer ao seguir os padrões de nomenclatura necessários para a integração com o Cofre de Senhas.


No `pom.xml`, deve-se utilizar a dependência do driver JDBC da Oracle para:

```yaml
<dependency>
    <groupId>com.oracle.database.jdbc</groupId>
    <artifactId>ojdbc8</artifactId>
    <version>19.3.0.0</version>
</dependency>
```

> Lembrar de remover ou excluir das dependências transitivas outras versões do driver JDBC da Oracle (`ojbc6` por exemplo).
> A partir da versão 1.2.0 da pilha Java, o driver já é o `ojdb8` numa versão adequada.

<br>

# Utilização

Após a configuração da dependencia do *chart*, é necessário inserir as seguintes configurações nos arquivos `devops/backend|frontend/aplicacao/values-<AMBIENTE>.yaml`

No exemplo abaixo, três segredos serão consultados via Cofre de Senhas:
```yaml
cofresenha:
  enable: true
  secretName: cofresenha-secret

  secretsList:
    - nome: "password_oracle"
      tipo: "banco_dados/oracle"
      chave: "WEB_APP"
      ambiente: "teste"
  
    - nome: "password_mysql"
      tipo: "banco_dados/mysql"
      chave: "WEB_APP"
      ambiente: "teste"
  
    - nome: "token"
      tipo: "automacao/cicd"
      chave: "token_api"
      ambiente: "teste"      
```


| **Parâmetro**          | **Descrição**                                                                                    | **Padrões**                                              |
| ---------------------- | -------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| `enable`               | Habilita a integração do Cofre de Senhas. | `true`                                               |
| `secretName`           | Nome do objeto do tipo `secret` que será criado no *namespace* da aplicação no Openshift. Deve ser qualquer nome que faça sentido. | `cofresenha-secret` |
| `secretsList`          | Listagem de todos os segredos (senhas, tokens, certificados e etc) a serem buscados no Cofre de Senhas (Vault) | - |
| `secretsList.nome`     | Nome da chave a ser criada no objeto `cofresenha-secret`, na qual conterá o valor do segredo. Deve ser qualquer nome que faça sentido.   | `password ` ou <br>`senha` ou <br> `token`                                                 |
| `secretsList.tipo`     | Tipo do segredo a ser buscado. Informado pela equipe que fez o cadastro no Cofre de Senhas.<br>Definições da [Norma de Uso](/infraestrutura/seguranca/cofre-senhas.md). Alguns tipos: <br> `banco_dados/oracle` ou<br>`banco_dados/mysql` ou<br>`automacao/cicd`  | **consultar** |
| `secretsList.chave`    | Nome da chave do segredo. Informado pela equipe que fez o cadastro no Cofre de Senhas. | **consultar**|
| `secretsList.ambiente` | Nome do ambiente do respectivo segredo.  | <br>`teste` ou <br>`des` ou<br>`hom` ou<br>`preprod` ou <br>`prod`


No exemplo abiaxo, para acessar os valores das `secret` são criadas as variável de ambiente `DATABASE_PASSWORD_ORACLE`, `DATABASE_PASSWORD_MYSQL` e `TOKEN_API`.

```yaml
...
deployment:
...
    containers:
      environments:
      - name: DATABASE_PASSWORD_ORACLE
        valueFrom:
          secretKeyRef:
            name: cofresenha-secret
            key: password_oracle

      - name: DATABASE_PASSWORD_MYSQL
        valueFrom:
          secretKeyRef:
            name: cofresenha-secret
            key: password_mysql

      - name: TOKEN_API
        valueFrom:
          secretKeyRef:
            name: cofresenha-secret
            key: token
...
```
