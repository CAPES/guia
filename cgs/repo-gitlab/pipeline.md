# Pipeline

A pipeline e seus componentes, Jobs e Stages, são definidos no arquivo `.gitlab-ci.yml`.



Na estrutura da Capes, o `.gitlab-ci.yml` deverá estar localizado no diretório `devops/`. Por padrão esse é um arquivo bloqueado e as informações contidas nele são as mesmas para todos os projetos.


`devops/.gitlab-ci.yml`

```
# Inclui os jobs
include:
  - project: 'cgs/DEVOPS/automations/gitlab-pipeline'
    file: '.gitlab-ci.yaml'
    ref: ocp
  - project: 'cgs/DEVOPS/automations/gitlab-pipeline'
    file: 'build.gitlab-ci.yaml'
    ref: ocp
  - project: 'cgs/DEVOPS/automations/gitlab-pipeline'
    file: 'deploy.gitlab-ci.yaml'
    ref: ocp
  - project: 'cgs/DEVOPS/automations/gitlab-pipeline'
    file: 'review.gitlab-ci.yaml'
    ref: ocp
  - project: 'cgs/DEVOPS/automations/gitlab-pipeline'
    file: 'tag.gitlab-ci.yaml'
    ref: ocp
```

### Fluxo de Eventos

```mermaid
graph LR
    A(Branch Feature) --> B(MR Branch Develop)
    B --> B1(Deploy Feature)
    B --> C(Aceite MR)
    C --> C1(Deploy Desenvolvimento)
    C --> C2(Deploy Teste - se houver)    
```

```mermaid
graph LR
    A(Branch Develop) --> B(MR Branch Master)
    A --> A1(Commit Branch Develop)
    A1 --> A2(Deploy Desenvolvimento) 
    A1 --> A3(Deploy Teste - se houver)
    B --> B1(Deploy Homologação)
    B --> C(Aceite MR)
    C --> C1(Deploy Pré-produção)
```


```mermaid
graph LR
    A(Branch Master) --> B(Geração de Tag)
    A --> A1(Commit Branch Master)
    A1 --> A2(Deploy Pré-produção - se houver) 
    B --> C(Play Manual Job)
    C --> E(Deploy Produção)    
```


#### Feature Branch
- `Commits` nessa `branch` **NÃO** acarretam nenhum evento de pipeline
- Solicitação de `MERGE REQUEST` para a branch `develop` executará a criação de ambiente dinâmico

#### Develop
- A aceitação de `MERGE REQUEST` ou a execução de algum `commit` na branch `develop`, executará a pipeline para `deploy` no ambiente de `teste` e `desevolvimento`
- Solicitação de `MERGE REQUEST` para a branch `master`, executará a pipeline para `deploy` no ambiente de `homologação`


#### Master
- A aceitação de ```MERGE REQUEST``` ao ```master```, irá executar a pipeline para `deploy` no ambiente de `pré-produção`

#### Tag
- A `criação de tag` servirá de gatilho para pipeline de `deploy` no ambiente de `produção`.
- O job de `deploy de produção` é `manual`, deverá ser executado apertando o `play` do job.


### Acompanhar Pipelines

As pipelines de um projeto git podem ser acompanhadas pelo menu `CI/CD >> Pipelines`

![Pipelines](./img/acompanhar-pipeline.png "Acompanhar Pipelines")