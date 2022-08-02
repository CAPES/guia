> **Versão PMP** - Você está lendo a documentação de uma versão antiga. Leia a [mais recente](cofre-senhas.md).

- [Visão Geral](#visão-geral)
- [Normas Gerais de Uso](#normas-gerais-de-uso)
  - [Nomenclatura](#nomenclatura)
    - [Caminho dos Segredos](#caminho-dos-segredos)
    - [Nome dos Usuários (nas Integrações via API)](#nome-dos-usuários-nas-integrações-via-api)
- [Como Acessar o PMP](#como-acessar-o-pmp)
  - [Interface Web](#interface-web)
    - [Segundo Fator de Autenticação](#segundo-fator-de-autenticação)
  - [API](#api)
    - [Ver os Recursos Compartilhados com o Usuário](#ver-os-recursos-compartilhados-com-o-usuário)
    - [Consulta IDs através do resource e usuário](#consulta-ids-através-do-resource-e-usuário)
    - [Ver uma Senha de uma Conta](#ver-uma-senha-de-uma-conta)
- [Integração via OpenShift (Chart)](#integração-via-openshift-chart)

<br><br>

# Visão Geral
No gerenciamento de um ambiente computacional com certa complexidade, o gerenciamento das senhas de serviço (senhas não pessoais) que integram os serviços, sistemas e bancos de dados, se faz necessário. O uso de planilhas, e-mail e ferramentas descentralizadas trazem vulnerabilidades problemas na gestão de forma profissional. 

Com isto, a DTI/CGII implantou um **[cofre de senhas](https://xpto.com/cgii/seguranca/pmp)** (acesso restrito), utilizando o software **PMP (Password Manager Pro** da empresa Manage Engine, que centraliza tais dados sensíveis e permite o compartilhamento das contas com colaboradores e também através do uso de API, para as automações necessárias.

<br><br>

# Normas Gerais de Uso

* **Complexidade da senha (PORTARIA EM CONSTRUÇÃO)** - define o número mínimo de caracteres que a senha deve conter.

* **Cadastro e Compartilhamento da senha** - Cada área - CGS (sistemas), CGII (infraestrutura) e NDAC (Banco de Dados) - tem um administrador que é responsáveis por gerenciar o cadastro e compartilhamento necessário das senhas. Via de regra, o compartilhamento é feito da seguinte forma:
    * **Para acesso ao banco de dados** - as senhas que são cadastradas no *datasource* da aplicação são criadas pela área de **banco de dados** e compartilhadas com:
      
      * **Equipe GCM** - para a configuração das aplicações de ambiente DHT (não produção).
      * **Equipe Infraestrutura** - para a configuração das aplicações de ambiente de produção.
      * **Automação via API** - para autoconfiguração

<br>

* **Alteração / Exclusão da senha** - a alteração da senha requer atualização tanto no cofre de senhas quanto na aplicação, sendo necessário o [registro de uma mudança](https://xpto.com/cgii/ccm/gmud/wikis/home), aplicando-se também para o caso de desativação de uma senha.

* **Contas locais, super usuários e senhas de certificados** - as senhas locais de administração (root, admin, sa, administrador...), normalmente criadas na instalação de um serviço, precisam estar registradas no cofre de senhas e serão compartilhadas apenas com os administradores respectivos daquela ferramenta, **quando estritamente necessário**. Os sistemas que permitem a criação de contas pessoais (ou integradas ao AD), com perfil de administrador devem ser utilizadas sempre que possível.


> :blue_book: As exceções a estas regras devem ser registradas via CATI sendo necessário autorização da coordenação.

<br>

## Nomenclatura 
### Caminho dos Segredos
O caminho onde os segredos ficam armazenados segue a seguinte nomenclatura abaixo. Sendo um exemplo, o `PROD_Antimalware-Mcafee`, onde as senhas de produção do sistema de antimalware, da empresa Mcafee. Logo, todos os segredos (usuários/senhas) ficarão armazenadas dentro de recurso (*resource name*).

> Nota: No cofre de senhas (PMP) o caminho é chamado de *resource name*.

```
XXX_AAA-BBB
```

Onde:
* XXX: prefixo que indica o ambiente (vide tabela abaixo)
* AAA: Indica o serviço/sistema a qual aquele recurso contém informações. Exemplo: Antimalware, PortalCapes, Wireless, Oracle, Intranet
* BBB: Indica o produto daquele serviço ou o fabricante. Exemplo: Microsoft, Google, Itautec


| **Prefixo XXX <br>(Ambiente)** | **Descrição** |
| ------------ | ------------- |
| `PROD`      | Prefixo para os segredos do ambiente de produção |
| `PREPROD`   | Prefixo para os segredos do ambiente de pre-produção|
| `HOM`       | Prefixo para os segredos do ambiente de homologação|
| `DES`       | Prefixo para os segredos do ambiente de desenvolvimento|
| `TESTE`     | Prefixo para os segredos do ambiente de testes|

<br>

### Nome dos Usuários (nas Integrações via API)
Para que a integração com o cofre de senhas, através de esteiras de desenvolvimento, fosse feita de forma padronizada, tornando algumas configurações transparentes para o desenvolvedor, foi definido o padrão a seguir. Sendo um exemplo, o `WEB_FINANCEIRO-WEB`, indicando 

> Nota: No cofre de senhas (PMP) o caminho é chamado de *resource name*.

```
SSS_PPP ou SSS-OOO_PPP
```

Onde:
* SSS: prefixo usado para identificar o tipo de usuário (Vide tabela abaixo)
  * OOO: nome **opcinal** também considerado como parte do prefixo
* PPP: nome exatamente igual ao projeto no repositório (Git)




| **Prefixo SSS <br>(Tipo da Conta)** | **Descrição** |
| ------------ | ------------- |
| `WEB`      | Prefixo do usuário que indica uma conta para uso de aplicação web |
| `SRV`      | Prefixo do usuário que indica uma conta de serviço|

<br><br>

# Como Acessar o PMP
Veja abaixo as formas para acessar os segredos na ferramenta PMP.

> :blue_book: Com base na norma acima, somente usuários previamente cadastradas podem acessar. As solicitações são feitas pelo CATI.

<br>

## Interface Web
O acesso a ferramenta PMP é feito da seguinte forma:
* Gerenciador de Senhas (PMP) - https://pmp.capes.gov.br:7272
  * **Usuário**: adm.<usuário> (conta administrativa)
  * **Senha**: <senha_da_rede>
  * **Segundo Fator de Autenticação (TFA - Two Facto Authentication)**: utilize um dos aplicativo abaixo

### Segundo Fator de Autenticação
Para uso do mecanismo de segundo fator de autenticação um dos aplicativos abaixo precisam ser utilizados:

* **[Authy](https://authy.com/) (Preferencial)** -  Recomenda-se instalá-lo no dispositivo móvel particular e de uso restrito.
  * [Android](https://play.google.com/store/apps/details?id=com.authy.authy)
  * [IOS](https://itunes.apple.com/us/app/authy/id494168017)
<br>

* **[Google Authenticator](https://support.google.com/accounts/answer/1066447?co=GENIE.Platform%3DAndroid&hl=pt-BR)** - recomenda-se instalá-lo no dispositivo móvel particular e de uso restrito.
  * [Android](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2&hl=pt_BR)
  * [IOS](https://itunes.apple.com/br/app/google-authenticator/id388497605?mt=8)
<br>

> :warning: **Sincronia de Data/Hora** - A autenticação com segundo fator é sensível ao sincronismo de data/hora. Certifique que seu dispositivo está sincronizado (consulte a hora certa em https://ntp.br/.
<br>

Caso seja necessário re-criar o segundo fator de autenticação no dispositivo (troca de aparelho) entre em contato com a CGII.


<br>

## API
O uso de API permite ver, alterar, trocar os recursos (usuários/senhas) dentro do PMP, porém as aplicações na CAPES até o momento são apenas para ver uma senha. 
> Para a listagem completa, vide [documentação do fabricante](https://www.manageengine.com/products/passwordmanagerpro/help/restapi.html).


> :blue_book: Os exemplos abaixo utilizam a chave (token) como uma variável de ambiente (`$TOKENPMP`), porém poderia ser informado o token na própria requisição, porém não recomendado.

<br>

### Ver os Recursos Compartilhados com o Usuário 
Considerando que já existem recursos (usuários/senhas) compartilhados com o usuário da API. Veja como listar quais são.

* **Exemplo**
```
curl -k -H "AUTHTOKEN:$TOKENPMP" https://pmp.capes.gov.br:7272/restapi/json/v1/resources
```

Saída:
```
{"operation":{"result":{"message":"Resources fetched successfully","status":"Success"},"Details":[{"RESOURCE DESCRIPTION":"","RESOURCE TYPE":"Windows","RESOURCE ID":"1","RESOURCE NAME":"TESTE_testeAPI","NOOFACCOUNTS":"2"}],"name":"GET RESOURCES","totalRows":1}}
```


<br>

### Consulta IDs através do resource e usuário

* **Exemplos**
```
curl -k -H "AUTHTOKEN:$TOKENPMP" https://pmp.capes.gov.br:7272/restapi/json/v1/resources/1/accounts
```

Saída:
```
{"operation":{"result":{"message":"Resource details with account list fetched successfully","status":"Success"},"Details":{"LOCATION":"","RESOURCE DESCRIPTION":"","RESOURCE TYPE":"Windows","RESOURCE URL":"","DOMAIN NAME":"","RESOURCE ID":"1","ACCOUNT LIST":[{"AUTOLOGONSTATUS":"User is not allowed to automatically logging in to remote systems in mobile","IS_TICKETID_REQD_ACW":"false","PASSWDID":"1","ISFAVPASS":"false","IS_TICKETID_REQD_MANDATORY":"false","ACCOUNT ID":"1","AUTOLOGONLIST":["Windows Remote Desktop","RDP Console Session"],"ACCOUNT NAME":"user1","PASSWORD STATUS":"****","IS_TICKETID_REQD":"false","ACCOUNT PASSWORD POLICY":"CAPES","ISREASONREQUIRED":"false"},{"AUTOLOGONSTATUS":"User is not allowed to automatically logging in to remote systems in mobile","IS_TICKETID_REQD_ACW":"false","PASSWDID":"2","ISFAVPASS":"false","IS_TICKETID_REQD_MANDATORY":"false","ACCOUNT ID":"2","AUTOLOGONLIST":["Windows Remote Desktop","RDP Console Session"],"ACCOUNT NAME":"user2","PASSWORD STATUS":"****","IS_TICKETID_REQD":"false","ACCOUNT PASSWORD POLICY":"CAPES","ISREASONREQUIRED":"false"}],"RESOURCE NAME":"TESTE_testeAPI","DEPARTMENT":"","DNS NAME":"","RESOURCE OWNER":"admin","RESOURCE PASSWORD POLICY":"CAPES"},"name":"GET RESOURCE ACCOUNTLIST"}}
```

<br>

### Ver uma Senha de uma Conta 
Para isto, é necessário saber qual é o ID do recursos e o ID da conta.

* **Exemplo**
```
curl -k -H "AUTHTOKEN:$TOKENPMP" https://pmp.capes.gov.br:7272/restapi/json/v1/resources/1/accounts/2/password
```

Saída:
```
{"operation":{"result":{"message":"Password fetched successfully","status":"Success"},"Details":{"PASSWORD":"XXXXXXXX"},"name":"GET PASSWORD"}}
```
<br><br><br>





<br><br>

# Integração via OpenShift (Chart)

A integração entre o Openshift e o Cofre de Senhas é feita através de um Operator do Kubernetes.
Foi criado o chart de `cofresenha` para facilitar a utilização dessa integração. Veja [como usar](/devops/orientacoes-tecnicas/cofre-senhas.md).