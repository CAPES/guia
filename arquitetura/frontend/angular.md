# Template
Um componente Angular poderá ser criado usando como base o projeto [Angular component Template](https://xpto.com/cgs/narq/componentes-javascript/angular-component-template.git), o qual é detalhado a seguir.

## Estrutura de pastas do projeto
* devops
* system

## devops
Contém todos os arquivos referentes ao processo de *buid*, *depoly*, *ci/cd*.

Originalmente, o pipeline é composto por dois estágios: *Tests* e *Tag*.

O primeiro é executado sempre que houver um *merge_requests*. Neste momento, serão executados os scripts `devops/pipeline-scripts/yarn-install-deps.sh` e `devops/pipeline-scripts/npm-compile-project.sh`.

### devops/.gitlab-ci.yml
Contém as configurações do pipeline.

## system
Contém arquivos/pastas do componente.
Sob esta pasta, a estruturação é livre, salvo exista alguma padronização.

Se fizer uso do [Angular component Template](https://xpto.com/cgs/narq/componentes-javascript/angular-component-template.git), configure os seguintes aquivos:

### system/package.json

| linha | Propriedade                                | Examplo                                               |
| ---:  | ---                                        | :---                                                  |
|  2    | `"name": "@capes/<component-name>"`        | `"name": "@capes/count-paragraph"`                    |
|  4    | `"description": "<component-description>"` | `"description": "Contador de parágrafos de um texto"` |
| 12    | `"author": "auth-name - <author-mail>"`    | `"author": "J. Augusto - <augustowebd@gmail.com>"`    |
| 15    | `"package-name": "package-version"`        | `"@angular/core": "~8.2.14",`                         |
| 18    | `"package-dev-name": "package-version"`    | `"typescript": "~3.4.0"`                              |

**NOTA:** Não altere a versão do componente, ela será definida pelo processo de deploy.

### system/README.md

Descreva o componente, bem como ele deve ser usadado exmplificando com *snippets*.

**NOTA**: Para que o pipeline seja executado corretamente, é-se necessário: 
i) atender as regras de uso do Git [Guia de uso Git](https://xpto.com/dti/orientacoes-gerais/guia/wikis/Guia-de-uso-Git); 
ii) o projeto do componente deverá está sob o grupo [Componentes Javascript](https://xpto.com/cgs/narq/componentes-javascript).

# Fluxo de buid da aplicações
- A codificação de cada componente deverá iniciar com a criação de seu repositório no git, na [seção Javascript](https://xpto.com/cgs/narq/componentes-javascript).
- **Após a conclusão de sua codificação**, realizar solicitação de _Merge Request_ à Arquitetura.
- O _Merge Request_ sendo aprovado, o pipeline enviará o código para o [Sinopia Capes](http://npm.ci.capes.gov.br/) com o nome informado na linha 2 do arquivo `system/package.json` - `"name": "@capes/<component-name>"`