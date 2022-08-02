## Introdução

O Harbor é um _registry_ de imagem de contêiner de código aberto que
protege imagens com controle de acesso baseado em função, verifica
imagens em busca de vulnerabilidades e assina imagens como confiáveis.

## Instalação e Configuração

> Vide: [Helm Chart](https://xpto.com/cgs/DEVOPS/helm/harbor)

## Procedimentos Operacionais

### Criar Projeto no Harbor

> Vide:
> [Criar Projeto no Harbor](devops/procedimentos-operacionais/criar-projeto-harbor.md)

## Política de Uso

### Regras e Normas

- Armazenamento das imagens​
- Análise de vuneralibilidade​
- Controle de acesso​

#### Endereços
- Portal​: [https://harbor.paas.capes.gov.br/​]

- Registry​: `registry.paas.capes.gov.br​`

#### Configurações dos Projetos

Um projeto no Harbor contém todos os repositórios de um aplicativo.
Nenhuma imagem pode ser enviada para o Harbor antes que o projeto seja
criado. O RBAC é aplicado a um projeto. Existem dois tipos de projetos
no Harbor:
- Público : todos os usuários têm o privilégio de leitura de um projeto público; é conveniente compartilhar alguns repositórios com outros dessa maneira.
- Privado : um projeto privado só pode ser acessado por usuários com privilégios adequados.

As propriedades do projeto podem ser alteradas clicando em
"Configuração": 
- Para tornar todos os repositórios do projeto acessíveis a todos,
  marque a `Public` caixa de seleção.
- Para impedir que imagens não assinadas no projeto sejam baixadas,
  marque a `Enable content trust` caixa de seleção.
- Para impedir que imagens vulneráveis ​​do projeto sejam baixadas,
  marque a `Prevent vulnerable images from running`caixa de seleção e
  altere o nível de gravidade das vulnerabilidades. As imagens não podem
  ser obtidas se o nível for igual ou superior ao nível selecionado no
  momento.
- Para ativar uma verificação imediata de vulnerabilidade em novas
  imagens enviadas para o projeto, marque a `Automatically scan images
  on push` caixa de seleção.

#### Usuários

Harbor gerencia imagens através de projetos. Os usuários podem ser
adicionados a um projeto como um membro com uma das três funções
diferentes:
- Guest : o convidado tem privilégio somente leitura para um projeto especificado.
- Developer : o desenvolvedor possui privilégios de leitura e gravação para um projeto.
- Master : o Master possui permissões elevadas além das do 'Developer',
  incluindo a capacidade de digitalizar imagens, visualizar _jobs_ de
  replicação e excluir imagens e Helm Charts.
- ProjectAdmin : Ao criar um novo projeto, você receberá a função
  "ProjectAdmin" no projeto. Além de privilégios de leitura e gravação,
  o "ProjectAdmin" também possui alguns privilégios de gerenciamento,
  como adicionar e remover membros, iniciando uma verificação de
  vulnerabilidade.

Além das três funções acima, existem duas funções no nível do sistema:
- SysAdmin : "SysAdmin" tem mais privilégios. Além dos privilégios
  mencionados acima, o "SysAdmin" também pode listar todos os projetos,
  definir um usuário comum como administrador, excluir usuários e
  definir a política de verificação de vulnerabilidades para todas as
  imagens. O projeto público "library" também pertence ao administrador.
- Anonymous : Quando um usuário não está logado, ele é considerado um usuário "Anônimo". Um usuário anônimo não tem acesso a projetos privados e tem acesso somente leitura a projetos públicos.


##### Permissões

Os usuários têm habilidades diferentes, dependendo da função em que
estão em um projeto.

Em projetos públicos, todos os usuários poderão ver a lista de
repositórios, imagens, vulnerabilidades de imagem, helm charts e versões
de helm chart, imagens pull, imagens de novo rótulo (precisa de
permissão `push` para a imagem de destino), baixar helm charts, baixar
versões de helm chart.

O administrador do sistema tem todas as permissões para o projeto.

##### Permissões de membros do projeto

A tabela a seguir mostra os vários níveis de permissão do usuário em um projeto.


| Ação                                                      | Limited Guest | Guest | Developer | Master | Project Admin |
|:----------------------------------------------------------|:--------------|:------|:----------|:-------|:--------------|
| Veja as configurações do projeto                          | ✓             | ✓     | ✓         | ✓      | ✓             |
| Edite as configurações do projeto                         |               |       |           |        | ✓             |
| Veja uma lista de membros do projeto                      |               | ✓     | ✓         | ✓      | ✓             |
| Criar / editar / excluir membros do projeto               |               |       |           |        | ✓             |
| Veja uma lista de logs do projeto                         |               | ✓     | ✓         | ✓      | ✓             |
| Veja uma lista de replicações do projeto                  |               |       |           | ✓      | ✓             |
| Veja uma lista de trabalhos de replicação de projeto      |               |       |           |        | ✓             |
| Veja uma lista de rótulos de projeto                      |               |       |           | ✓      | ✓             |
| Criar / editar / excluir rótulos de projeto               |               |       |           | ✓      | ✓             |
| Veja uma lista de repositórios                            | ✓             | ✓     | ✓         | ✓      | ✓             |
| Crie repositórios                                         |               |       | ✓         | ✓      | ✓             |
| Editar / excluir repositórios                             |               |       |           | ✓      | ✓             |
| Veja uma lista de imagens                                 | ✓             | ✓     | ✓         | ✓      | ✓             |
| Voltar a marcar a imagem                                  |               | ✓     | ✓         | ✓      | ✓             |
| Puxar imagem                                              | ✓             | ✓     | ✓         | ✓      | ✓             |
| Imagem push                                               |               |       | ✓         | ✓      | ✓             |
| Digitalizar / excluir imagem                              |               |       |           | ✓      | ✓             |
| Veja uma lista de vulnerabilidades de imagem              | ✓             | ✓     | ✓         | ✓      | ✓             |
| Veja o histórico de criação de imagens                    | ✓             | ✓     | ✓         | ✓      | ✓             |
| Adicionar / remover etiquetas da imagem                   |               |       | ✓         | ✓      | ✓             |
| Veja uma lista de gráficos de leme                        | ✓             | ✓     | ✓         | ✓      | ✓             |
| Download de gráficos de leme                              | ✓             | ✓     | ✓         | ✓      | ✓             |
| Carregar gráficos de leme                                 |               |       | ✓         | ✓      | ✓             |
| Excluir gráficos de leme                                  |               |       |           | ✓      | ✓             |
| Veja uma lista de versões do gráfico de leme              | ✓             | ✓     | ✓         | ✓      | ✓             |
| Baixe versões do gráfico de leme                          | ✓             | ✓     | ✓         | ✓      | ✓             |
| Carregar versões do gráfico de leme                       |               |       | ✓         | ✓      | ✓             |
| Excluir versões do gráfico de leme                        |               |       |           | ✓      | ✓             |
| Adicionar / remover etiquetas da versão do Helm Char      |               |       | ✓         | ✓      | ✓             |
| Veja uma lista de robôs de projeto                        |               |       |           | ✓      | ✓             |
| Criar / editar / excluir robôs do projeto                 |               |       |           |        | ✓             |
| Consulte lista de permissões CVE configurada              | ✓             | ✓     | ✓         | ✓      | ✓             |
| Criar / editar / remover lista de permissões do CVE       |               |       |           |        | ✓             |
| Ativar / desativar webhooks                               |               |       | ✓         | ✓      | ✓             |
| Criar / excluir regras de retenção de tags                |               |       | ✓         | ✓      | ✓             |
| Ativar / desativar regras de retenção de tags             |               |       | ✓         | ✓      | ✓             |
| Veja cotas do projeto                                     | ✓             | ✓     | ✓         | ✓      | ✓             |
| Editar cotas de projeto                                   |               |       |           |        |               |


## Referências
- [Chart do Harbor](https://xpto.com/cgs/DEVOPS/helm/harbor)
- [Github do Harbor](https://github.com/goharbor/harbor)
- [Documentação do Harbor](https://github.com/goharbor/harbor/tree/master/docs)
- [Guia de Instalação do Harbor](https://github.com/goharbor/harbor/blob/master/docs/installation_guide.md)