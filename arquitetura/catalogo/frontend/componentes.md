[[_TOC_]]
# Catálogo de interface
Um catálogo tem a finalidade de definir padrões que podem ser reaproveitados para o desenvolvimento de sistemas em um âmbito mais geral, tendo como principal foco o padrão de experiência disponibilizado, isso permite uma melhor uniformização e aproveitamento do desenvolvimento, liberando os desenvolvedores de criar soluções próprias, também cria uma linguagem única que pode ser compartilhada entre os times de desenvolvimento e especificações, fazendo com que os casos de uso, tenham uma melhor legibilidade.

## Padrões
* Seleção Simples
> Descrever
* Seleção Múltipla
> Descrever
* Busca
> Descrever
* Busca Avançada
> Descrever
Mestre Detalhe
> Descrever

# Catálogo de componentes
Um catálogo de componentes tem como finalidade padronizar os componentes existentes além de documentar o seu uso dando exemplos de utilização e em qual cenário um determinado componente é mais adequado.

Os componentes deste catálogo estão desenvolvidos em angular, como directivas, filtros entre outros componentes da arquitetura client-side angular, os componentes especificados aqui estão distribuídos em 2 categorias distintas: Componentes visuais, Componentes não visuais.

## Componentes Visuais
* Alert
* Autocomplete
* Checkbox
* Datepicker
* Dropdown
* Input
* Mask
* Menu
* Message
* Modal
* Navbar
* Painel
* ProgressBar
* Radio
* Select
* Tab
* Upload

## Componentes não visuais
Datasource
Visão geral

O componente datasource tem a finalidade de encapsular na view um tratador de requisições ajax, expondo uma api para manipulação de request, com suporte ao path param.

**Atributos**

| Atributo  | Obrigatório | Default   | Exemplo                                       | Descrição  |
| :---      | :---        | :---      | :---                                          | :---       |
| uri       | Sim         |  n/a      | [Link](http://api.capes.gov.br/rest/servico)  | Este atributo é o identificador do componente toda api do componente será exposta via atributo id       |

**Eventos**

| Evento       		| Descrição   																								|
| :---         		| :---        																								|
| on-success		| Evento disparado quando ocorre um sucesso no envio da requisição ajax, qualquer que seja essa requisição.	|
| on-fail			| Evento disparado quando ocorre um erro no envio da requisição ajax, qualquer que seja essa requisição.	|
| on-success-get	| Evento disparado quando ocorre um sucesso no envio da requisição GET via ajax.							|
| on-success-post	| Evento disparado quando ocorre um sucesso no envio da requisição POST via ajax.							|
| on-success-put	| Evento disparado quando ocorre um sucesso no envio da requisição PUT via ajax.							|
| on-success-delete	| Evento disparado quando ocorre um sucesso no envio da requisição DELETE via ajax.							|
| on-fail-get		| Evento disparado quando ocorre um falha no envio da requisição GET via ajax.								|
| on-fail-post		| Evento disparado quando ocorre um falha no envio da requisição POST via ajax.								|
| on-fail-put		| Evento disparado quando ocorre um falha no envio da requisição PUT via ajax.								|
| on-fail-delete	| Evento disparado quando ocorre um falha no envio da requisição DELETE via ajax.							|


**API do componente**
**Método:** *get*
---
**Descrição:** 
Método que efetua uma chamada http do tipo get para o componente especificado.

**Como usar:**
Este método recebe um objeto que representa a requisição http get do angular, entretanto ele adiciona suporte a pathParams, que são parametros do path. Abaixo vemos alguns exemplos de código:

**Utilização simples**

```html
  <ct-datasource id="cidades" uri="rest/cidades"></ct-datasource>
   <button ng-click="cidades.get()">carregar cidades</button>
  <p/>
  <table class="table">
    <tr><th>#</th><th>Nome</th></tr>
    <tr ng-repeat="cidade in cidades.response.data">
      <td>{{cidade.id}}</td>
      <td>{{cidade.nome}}</td>
    </tr>
  </table>
```
[visualizar snippet no jsbin](http://jsbin.ci.capes.gov.br/bup/2/edit)

Esse código cria um datasource chamado **cidades** que fará a requisição para **rest/cidades**, ao clicar no botão **carregar cidades**, é acionada a api do datasource que efetua a chamada get() correspondente usando o nome do componente, **cidades.get()** O resultado da requisição com sucesso ou erro, é armazenado no componente na variável result no formato do angular

```javascript
{
  "data":[
    {
      "id":1,
      "nome":"DF"
    },
    {
      "id":2,
      "nome":"SP"
    }
  ],
  "status":200,
  "statusText":"OK"
}
```
---

**Utilização com pathParams**

```html
  <ct-datasource id="cidade" uri="rest/cidades/{id}">
  </ct-datasource>
 
  <input ng-model="idCidade" placeholder="Informe o código"/>
  <button ng-click="cidade.get({pathParams:{id:idCidade}})">
    carregar cidade
  </button>
 
  <br/>
  Você carregou a cidade 
  {{cidade.response.data.id}}  
  {{cidade.response.data.nome}}
```
[visualizar snippet no jsbin](http://jsbin.ci.capes.gov.br/goh/1/edit?html,output)

Esse código cria um datasource chamado **cidade** ao informar o id no input da cidade com o valor 1 por exemplo, e clicar no botão **carregar cidade** será executada uma requisição para **rest/cidade/1**, passando o parametro pathParams com o id configurado com o valor que foi informado no input, **cidade.get({pathParams:{ id: idCidade }})** O resultado da requisição com sucesso ou erro, é armazenado no componente na variável result no formato do angular

---

**Utilização com query string params**

```html
<ct-datasource id="cidade" uri="rest/cidades">
  </ct-datasource>
 
  <input ng-model="idCidade" placeholder="Informe o código"/>
  <button ng-click="cidade.get({params:{id:idCidade}})">
    carregar cidade
  </button>
 
  <br/>
  Você carregou a cidade 
  {{cidade.response.data.id}}  
  {{cidade.response.data.nome}}
````
[visualizar snippet no jsbin](http://jsbin.ci.capes.gov.br/rir/1/edit?html,output)

Esse código cria um datasource chamado **cidade** ao informar o id no input da cidade com o valor 1 por exemplo, e clicar no botão **carregar cidade** será executada uma requisição para **rest/cidades?id=1**, passando o parametro params com o id configurado com o valor que foi informado no input, ```javascript cidade.get({params:{ id: idCidade }})``` O resultado da requisição com sucesso ou erro, é armazenado no componente na variável result no formato do angular

```javascript
{
  "data":{
    "id":1,
    "nome":"Brasilia"
  },
  "status":200,
  "statusText":"OK"
}
```

--- 
**Método:** *post*

Descrição: Método que efetua uma chamada http do tipo post pelo componente.

Como usar:

Este método recebe um objeto que representa a requisição http post do angular, entretanto ele adiciona suporte a pathParams, que são parametros do path exatamente igual ao metodo get.

**Utilização**

```html
 <ct-datasource id="cidades" uri="rest/cidades">
  </ct-datasource>
   <input ng-model="form_nome" placeholder="Informe o nome" />
   <button ng-click="cidades.post({data:{nome:form_nome}})">
    salvar cidade
  </button>
   <p/>
 
 
  <button ng-click="cidades.get()">carregar cidades</button>
  <p/>
  <table class="table">
    <tr><th>#</th><th>Nome</th></tr>
    <tr ng-repeat="cidade in cidades.response.data">
      <td>{{cidade.id}}</td>
      <td>{{cidade.nome}}</td>
    </tr>
  </table>
```

[visualizar snippet no jsbin](http://jsbin.ci.capes.gov.br/nus/1/edit?html,output)

Esse código cria um datasource chamado **cidades** que fará a requisição para **rest/cidades**, ao clicar no botão **carregar cidades**, é acionada a api do datasource que efetua a chamada get() correspondente usando o nome do componente, **cidades.get()** O resultado da requisição com sucesso ou erro, é armazenado no componente na variável result no formato do angular

```javascript
{
  "data":[
    {
      "id":1,
      "nome":"DF"
    },
    {
      "id":2,
      "nome":"SP"
    }
  ],
  "status":200,
  "statusText":"OK"
}
```

Também efetua o salvamento via POST de uma nova cidade, para enviar alguma informação na requisição post, é necessário preencher o parametro data com o valor que deve ser enviado. Neste caso temos um formulário que altera um nome e envia via post os dados.