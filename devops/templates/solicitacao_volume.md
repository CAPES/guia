## Introdução

Neste Guia será descrito o formato de como deve ser solicitado volumes para o Openshift.

A solicitação deve ser feita através de um CATI para a fila da INFRAESTRUTURA.

#### Padrão do Título

`[OPENSHIFT] - Criação de volume de storage <AMBIENTE> - <NOME/SIGLA APP>`

Deve ser criado somente dois volumes, um para o ambiente DHT (Desenvolvimento, homologação e teste) e outro PROD (Produção).

AMBIENTE: PROD ou DHT

### Template do conteúdo do chamado

```
Prezados,
 
Solicito a criação dos seguintes volumes de storage:
VOL_<SIGLA-APLICACAO>_OPENSHIFT_<TIPO>_<AMBIENTE>
** O 'TIPO' pode ser LOGS, ARQUIVOS ou algo que faça sentindo para o que for ser armazenado no volume

******
Deve ser disponibilizado o volume no SAMBA-GNS para os usuários abaixo:
- <lista de desenvolvedores da aplicação>
- yxz
- xpto
******
 
- Qual tamanho do espaço necessário?
R:
Para volume em desenvolvimento - 5gb (Sugestão)
Para volume em produção - 30gb (Sugestão)
 
-Caso o volume seja criado informar os IPs dos servidores? 
R:
<vide item abaixo>
 
- O espaço será liberado depois? 
R: Não
 
- Quantos servidores hoje existentes com o volume mapeado? 
R: Nenhum
 
- Em qual ambiente será criado o volume? 
R: <AMBIENTE>

- Existe algum outro volume DHT ou de Produção que o espaço possa ser
alocado para este ambiente? 
R: N/A
 
- Quem é o responsável imediato deste solicitação? 
R: <Gerente da Aplicação>
 
- Tem algum outro volume DHT ou produção  que não esteja sendo utilizado
e que possa ser liberado? 
R: N/A

- Tem algum volume DHT que  o espaço posso ser helenizado para criação de
outro?. 
R: Não

- Quanto tempo o volume ficará montado no ambiente de teste? 
R: N/A
```


### Máquinas de compute do Openshift
 
#### Desenvolvimento (DHT)
```
v-okd-compute01-h.hom.capes.gov.br

```

#### Ambiente Produtivo (PROD)
```
v-okd-compute01.capes.gov.br


```