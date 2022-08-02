## Introdução

O _pipeline_ é o componente de alto nível da integração contínua, entrega e implantação.

Os _pipelines_ compreendem:

- _Jobs_ que definem o que executar. Por exemplo, compilação de código
  ou execução de teste.
- _Stages_ que definem quando e como executar. Por exemplo, esses testes
  são executados somente após a compilação do código.

Vários _Jobs_ no mesmo _Stage_ são executados pelos Runners em paralelo, se houver Runners simultâneos suficientes.

Se todos os _Jobs_ em um _Stage_:

- Com sucesso, o pipeline passa para o próximo estágio.
- Falha, o próximo _Stage_ não é (geralmente) executado e o _pipeline_ termina mais cedo.

> [Git Flow](ferramentas/git/norma-de-uso/Guia-de-uso-Git.md#git-flow)

## Configuração

Os scripts de pipeline foram definidos no repositório [gitlab-pipeline](https://xpto.com/cgs/DEVOPS/automations/gitlab-pipeline), portanto para a criacão de um pipeline padrão da CAPES basta incluí-los no _devops/.gitlab-ci.yaml_ do projeto.


### Exemplo

```yaml
# Inclui os jobs
include:
  - project: 'cgs/DEVOPS/automations/gitlab-pipeline'
    file: '.gitlab-ci.yaml'
  - project: 'cgs/DEVOPS/automations/gitlab-pipeline'
    file: 'build.gitlab-ci.yaml'
  - project: 'cgs/DEVOPS/automations/gitlab-pipeline'
    file: 'deploy.gitlab-ci.yaml'
  - project: 'cgs/DEVOPS/automations/gitlab-pipeline'
    file: 'review.gitlab-ci.yaml'
  - project: 'cgs/DEVOPS/automations/gitlab-pipeline'
    file: 'tag.gitlab-ci.yaml'

```


## Referências
- [Guia de Uso](/ferramentas/git/norma-de-uso/Guia-de-uso-Git.md)