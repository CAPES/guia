[[_TOC_]]

## Introdução

Este documento define regras para a criação de DNS para tanto para as aplicações novas, que serão implantadas no OpenShift, quanto para as aplicações legadas que estão sendo migradas para o mesmo.

## Balanceadores

Atualmente o Openshift possui três tipos de balanceadores

- **DHT**: São os balanceadores para os ambientes de desenvolvimento, teste, homologação e pré-produção
- **Produção Interno**: São os balanceadores para o ambiente de produção de aplicações de consumo interno da CAPES
- **Produção Público**: São os balanceadores para o ambiente de produção de aplicações públicas da CAPES, ou seja, aplicações expostas para a internet


|Tipo|Host|IP| VIP|                                                       
|:---|:---|:---|:---|                                                     
|DHT|v-okd-bal-dht01.hom.capes.gov.br | 172.19.206.142|172.19.206.146|   
|DHT|v-okd-bal-dht02.hom.capes.gov.br | 172.19.206.143|172.19.206.147|   
|DHT|v-okd-bal-dht03.hom.capes.gov.br | 172.19.206.144|172.19.206.148|   
|DHT|v-okd-bal-dht04.hom.capes.gov.br | 172.19.206.145|172.19.206.149|   
|Produção Interno|v-okd-bal-in-prod01.capes.gov.br | 172.19.204.31| 172.19.204.29|
|Produção Interno|v-okd-bal-in-prod02.capes.gov.br | 172.19.204.32| 172.19.204.30|
|Produção Público|v-okd-bal-prod01.capes.gov.br |172.19.204.21|172.19.204.25|
|Produção Público|v-okd-bal-prod02.capes.gov.br |172.19.204.22|172.19.204.26|
|Produção Público|v-okd-bal-prod03.capes.gov.br |172.19.204.23|172.19.204.27|
|Produção Público|v-okd-bal-prod04.capes.gov.br |172.19.204.24|172.19.204.28|


## DNS

### Ambiente DHT

Dados os padrões de nomenclatura[^padroes] estabelecidos, os ambientes de desenvolvimento, teste, homologação e pré-produção serão classificados em Aplicações Legadas, ou seja, foram implantadas anteriormente ao OpenShift, e Aplicações novas, que já foram concebidas para ambiente conteinerizado.

#### Aplicações Legadas

Deverão ser alterados os DNS atuais para apontar para os VIPS dos balanceadores de DHT fazendo Round-robin

> Ex: 
> - sigla-da-aplicacao.hom.capes.gov.br -> 172.19.206.146
> - sigla-da-aplicacao.hom.capes.gov.br -> 172.19.206.147
> - sigla-da-aplicacao.hom.capes.gov.br -> 172.19.206.148
> - sigla-da-aplicacao.hom.capes.gov.br -> 172.19.206.149

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
> - sigla-da-aplicacao.capes.gov.br -> 172.19.204.29
> - sigla-da-aplicacao.capes.gov.br -> 172.19.204.30

Além disso deve-se criar uma rota do Openshift no padrão estabelecido.

> Ex: 
> - sigla-da-aplicacao.capes.gov.br

#### Ambiente de Produção Público

Deverão ser criados/alterados os DNS para apontar para os VIPS dos balanceadores de Produção Público fazendo Round-robin

> Ex: 
> - sigla-da-aplicacao.capes.gov.br -> 172.19.204.25
> - sigla-da-aplicacao.capes.gov.br -> 172.19.204.26
> - sigla-da-aplicacao.capes.gov.br -> 172.19.204.27
> - sigla-da-aplicacao.capes.gov.br -> 172.19.204.28


Além disso deve-se criar uma rota do Openshift no padrão estabelecido.

> Ex: 
> - sigla-da-aplicacao.capes.gov.br


## Referências

- [^padroes]: [Padrão de nomenclatura](/padrao-nomenclatura)