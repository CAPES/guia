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

### Eventos

```mermaid
graph LR
    B(Feature Branch) --> C(MR Develop)
    C --> C1(Deploy Ambiente Dinâmico)
    C --> D(Aceite MR)
    D --> E(Commit Develop)
    E --> E1(Deploy Desenvolvimento)
    E --> F(MR Master)
    F --> F1(Deploy Homologação)
    F --> G(Aceite MR)
    G --> G1(Deploy Pré-prod)
    G --> H(Geração de Tag)
    H --> I(Deploy Produção)    
```