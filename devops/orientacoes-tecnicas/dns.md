## Introdução

Este documento define regras para a criação de DNS para tanto para as aplicações novas, que serão implantadas no OpenShift, quanto para as aplicações legadas que estão sendo migradas para o mesmo.

## Balanceadores

Atualmente o Openshift possui três tipos de balanceadores

- **DHT**: São os balanceadores para os ambientes de desenvolvimento, teste, homologação e pré-produção
- **Produção Interno**: São os balanceadores para o ambiente de produção de aplicações de consumo interno da CAPES
- **Produção Público**: São os balanceadores para o ambiente de produção de aplicações públicas da CAPES, ou seja, aplicações expostas para a internet


|Tipo|Host|IP| VIP|                                                       
|:---|:---|:---|:---|                                                     



## DNS

### Ambiente DHT

Dados os padrões de nomenclatura[^padroes] estabelecidos, os ambientes de desenvolvimento, teste, homologação e pré-produção serão classificados em Aplicações Legadas, ou seja, foram implantadas anteriormente ao OpenShift, e Aplicações novas, que já foram concebidas para ambiente conteinerizado.

#### Aplicações Legadas

Deverão ser alterados os DNS atuais para apontar para os VIPS dos balanceadores de DHT fazendo Round-robin

> Ex: 
> - sigla-da-aplicacao.hom.capes.gov.br -> XXX.XXX.XXX.XXX

Além disso deve-se criar duas rotas no Openshift no padrão estabelecido.

> Ex: 
> - sigla-da-aplicacao.hom.capes.gov.br
> - sigla-da-aplicacao-hom.cloud.capes.gov.br

#### Aplicações Novas

Não há necessicade de criaćão de DNS haja vista que há o wildcard *.cloud.capes.gov.br, basta criar uma rota do Openshift no padrão estabelecido.

> Ex: 
> - sigla-da-aplicacao-hom.cloud.capes.gov.br


### Ambiente de Produção

Para o ambiente de Produção temos dois cenários o de Produção interno e Produção Público

#### Ambiente de Produção Interno

Deverão ser criados/alterados os DNS para apontar para os VIPS dos balanceadores de Produção Interno fazendo Round-robin

> Ex: 
> - sigla-da-aplicacao.capes.gov.br -> XXX.XXX.XXX.XXX

Além disso deve-se criar uma rota do Openshift no padrão estabelecido.

> Ex: 
> - sigla-da-aplicacao.capes.gov.br

#### Ambiente de Produção Público

Deverão ser criados/alterados os DNS para apontar para os VIPS dos balanceadores de Produção Público fazendo Round-robin

> Ex: 
> - sigla-da-aplicacao.capes.gov.br -> XXX.XXX.XXX.XXX


Além disso deve-se criar uma rota do Openshift no padrão estabelecido.

> Ex: 
> - sigla-da-aplicacao.capes.gov.br


## Referências

- [^padroes]: [Padrão de nomenclatura](devops/politicas/padrao-nomenclatura.md)