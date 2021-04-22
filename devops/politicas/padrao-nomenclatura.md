## Introdução

Este documento define regras e normas de padrão de nomenclatura para as
diversas ferramentas em uso atualmente na CAPES. Assume-se que o leitor
tem prévio conhecimento na ferramenta mencionada e suas restrições de
nomenclatura, portanto onde não estiver escrito os critérios de
aceitação que a ferramenta impõe, entende-se que o padrão será o _Kebab
Case_ sem preposições, sem acentos ou caracteres especiais. Cada
ferramenta define o tamanho máximo de caracteres que poderão compor o
nome de um Objeto, portanto consulte a documentação de cada uma delas.

As definições mostrarão tabelas da ferramentas e os objetos referente a
elas, a definição do padrão semântico da CAPES e um exemplo baseado em
uma aplicação fictícia.

Nos exemplos abaixo iremos trabalhar com uma aplicação fictícia chamada
**"Cadastro de usuários"** do sistema também fictício cujo nome é
**"SISCADUS"**, portanto estes nomes em _Kebab Case_ são
cadastro-usuario e siscadus respectivamente.

### Legendas

#### Siglas dos Ambientes

| Ambiente        | Sigla   |
|:----------------|:--------|
| Desenvolvimento | des     |
| Teste           | test    |
| Homologação     | hom     |
| Pré Produção    | preprod |
| Produção        | prod    |

#### Siglas das Coordenações

| Coordenação                                             | Sigla |
|:--------------------------------------------------------|:------|
| Coordenação-Geral de Sistemas                           | CGS   |
| Coordenação de Sistemas de Apoio à Educação             | CSAE  |
| Coordenação de Sistemas de Auxílios, Bolsas e Convênios | CSAB  |
| Coordenação de Sistemas de Avaliação da Pós-Graduação   | CSAPG |


> Vide: [http://intranet.capes.gov.br/index.php/dti/organograma-dti]

## Padrão de nomenclatura 


### Git


| Objeto         | Padrão                                                                                   | Exemplo                                         |
|:---------------|:-----------------------------------------------------------------------------------------|:------------------------------------------------|
| Repositório    | Coordenação Geral/Coordenação de Sistema/Nome do Sistema[^sistema]/Aplicação[^aplicacao] | cgs/csae/siscadus/cadastro-usuario              |
| Feature Branch | feature/Número do Redmine-Nome da Tarefa                                                 | feature/1234-alterar-cadastro-usuario[^feature] |
| Tag            | **v**MAJOR.MINOR.PATCH[^semver]                                                          | v1.0.1                                          |


#### Gitlab Variáveis

| Prefixo         | Sigla   |
|:----------------|:--------|
| CI_             | Variáveis de origem do próprio Gitlab CI     |
| CGS_            | Variáveis globais dos projetos da CGS. Essas variáveis estão definidas na aba `CI/CD` projeto `CGS`    |
| REPO_           | Variáveis locais do projeto. Essas variáveis estão definidas na aba `CI/CD` do próprio projeto.    |
| LOCAL_          | Variáveis locais definidas no próprio job. |


### Harbor

| Objeto      | Padrão                                                       | Exemplo                         |
|:------------|:-------------------------------------------------------------|:--------------------------------|
| Projeto     | Nome do Sistema[^sistema]                                    | siscadus                        |
| Repositório | Nome do Sistema[^sistema]/Nome do Imagem[^imagem-docker]:tag | siscadus/cadastro-usuario:1.0.1 |


### OpenShift


| Objeto            | Padrão                                   | Exemplo               |
|:------------------|:-----------------------------------------|:----------------------|
| Projeto[^projeto] | Aplicação[^aplicacao]- Sigla do Ambiente | cadastro-usuario-prod |

### URLs


| Ambiente          | Padrão                                   | Exemplo                       |
|:------------------|:-----------------------------------------|:------------------------------|
|DHT - Openshift    | sigla-ambiente.cloud.capes.gov.br        | siprec-des.cloud.capes.gov.br |
|DHT - Tradicional  | sigla.ambiente.capes.gov.br              | siprec.des.capes.gov.br       |
|PRODUÇÃO           | sigla.capes.gov.br                       | siprec.capes.gov.br           |



## Referências

[^sistema]: Nome do Sistema no Portal Gestão em _Kebab Case_
[^aplicacao]: Nome da Aplicação no Portal Gestão em _Kebab Case_.
[^feature]: A palavra feature é obrigatória seguida de /
[^semver]: Em conformidade com o padrão semver [https://semver.org/]
[^imagem-docker]: Nome da Imagem docker
[Padrão de Nomes de Imagem](devops/ferramentas-servicos/docker.md#nome-da-imagem)
[^projeto]: Os nomes dos projetos no OpenShift somente poderão conter letras
  minúsculas, números e traços. Eles não podem começar ou terminar com
  um traço.

