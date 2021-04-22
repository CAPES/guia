# Parametrização das aplicações no Openshift.

## Motivação

As aplicações novas ou migradas devem priorizar o armazenamento dos parâmetros de sistema como variáveis de ambiente, em detrimento ao armazenamento em banco de dados ou em arquivos de texto.<br/>
A motivação para isto são várias, o artigo sobre os 12 fatores [^1] descreve as vantagens de escalabilidade ao armazenar as configurações em *env vars*. Com este esquema de configuração, a imagem Docker construída pode ser implantada em qualquer ambiente (dht ou produção) sem alteração de código.

## Exemplos de parâmetros

Os seguintes parâmetros podem ser transformados em variáveis de ambiente individuais:

* Endereços de serviços <br/>
* Senhas de banco (no formato Secret) <br/>
* Configurações do SSO <br/>  

## Spring Boot

No caso de aplicações Spring Boot, a configuração principal (application.yml), deve ser montada através de um config-map por ambiente.<br/>
As senhas de banco devem ser armazenadas no cofre de senhas e obtidas como um Secret (via cofre-senha-operator).


## Parâmetros dinâmicos (casos especiais)

Existem parâmetros cujos valores precisam ser aplicados sem restart da aplicação (data de eventos, flag de manutenção do sistema, etc). Esse tipo de parâmetro pode ficar no banco de dados para fácil update e não representa uma quebra dos princípios dos 12 fatores. O importante é garantir que estes parâmetros não sejam impeditivos para a inicialização da aplicação.

## Feature toggle

A parametrização de casos de uso utilizando feature toggles deve ser feita utilizando mecanismo próprio. Não é adequado armazenar esse tipo de parâmetro em banco de dados, uma vez que isto impacta na independencia de deploy da aplicação, nem este deve ser armazenado em variáveis de ambiente, uma vez que obriga o restart da aplicação. <br/>
Consultar os projetos disponibilizados [^2] e [^3].

## Referências

[^1]: https://12factor.net/pt_br/config
[^2]: https://git.capes.gov.br/cgs/narq/pocs/feature-toggle/capes-feature-toggle-service
[^3]: https://git.capes.gov.br/cgs/narq/pocs/feature-toggle/capes-feature-toggle-client