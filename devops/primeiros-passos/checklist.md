# Objetivo
O checklist abaixo tem como objetivo conhecer a aplicação que será migrada ou criada no ambiente Openshift. Envie o mesmo para devops@capes.gov.br com o título "[OPENSHIFT] Criação de Ambiente - <SIGLA_APLICACAO>/<NOME_APLICACAO>"

# Checklist
| Responsável | Informação    | Exemplo | 
|:--------------|:--------------|:-------|
|Infraestrutura | Quantidade de Máquinas (Produção)	| Ex: 1|
|Infraestrutura | Quantidade de CPU por máquina (Produção)	| Ex: 2 CPU|
|Infraestrutura | Quantidade de Memória  por máquina (Produção)	| Ex: 8GB|
|Time de Desenvolvimento | Implantação ou Migração	| Ex: Implantação/Migração|
|Time de Desenvolvimento | Endereço GIT	| Ex: https://git.capes.gov.br/cgs/CSAE/sistema/aplicacao.git|
|Time de Desenvolvimento | Id da aplicação no Gestão	| Ex: 2982 |
|Time de Desenvolvimento | SSO - Provedor de serviço	| Ex: aplic.capes.gov.br|
|Time de Desenvolvimento | SSO - Tipo de Autenticação	| Ex: OAUTH|
|Time de Desenvolvimento | SSO - Fluxos	| Ex: Implicit|
|Time de Desenvolvimento | SGBDs	| Ex: ORACLE e POSTGRESQL|
|Time de Desenvolvimento | Usuários SGBDs	| Ex: WEBAPP (ORACLE) E WEBCORPORATIVO (POSTGRESQL)|
|Time de Desenvolvimento | Servidor de Cache	| Ex: Memcached, Redis e FileSystem|
|Time de Desenvolvimento | Bancos de dados NoSQL	| Ex: ElasticSearch|
|Time de Desenvolvimento | Plataforma Backend - Linguagem e Versão	| Ex: Java JDK 6, PHP FPM 7, Python 3.7|
|Time de Desenvolvimento | Plataforma Frontend - Linguagem e Versão	| Ex: Angular - 7 e node - 10|
|Time de Desenvolvimento | Lista de Serviços de Apoio - URL	| Ex: https://interno.capes.gov.br/cadastropessoas|
|Time de Desenvolvimento | Lista de Apis e/ou Serviços - URL	| Ex: https://aplicacao.capes.gov.br/service/WebService?wsdl, https://aplicacao.capes.gov.br/api/bolsas|
|Time de Desenvolvimento | Lista de Aplicações que consomem os Serviços | Ex: SCBA e SUCUPIRA|
|Time de Desenvolvimento | Arquivos de configuração	| Ex: .env, aplic-ds.xml, sso.properties e services.properties|
|Time de Desenvolvimento | Portas Expostas	| Ex: 80 e 9000|
|Time de Desenvolvimento | WebServer - Versão	| Ex: Apache 2.4, JBoss EAP 5.2.0|
|Time de Desenvolvimento | Tipos de Log	| Ex: Arquivo e Graylog|
|Time de Desenvolvimento | Volumes Utilizados	| Ex: /CAPES/AplWeb/,  /CAPES/Arquivos e /CAPES/Arquivos/libs |
|Time de Desenvolvimento | Ambientes Utilizados	| Ex: Desenvolvimento, Teste, Homologação, PréProd e Produção|
|Time de Desenvolvimento | Stateful(Armazena Estado)/StateLess(Não Armazena Estado)	| Ex: Stateful|
|Time de Desenvolvimento | Apenas para aplicações Stateful, que estados são armazenados no Servidor	| Ex: Sessão do usuário e Cache|
|Time de Desenvolvimento | Todas as dependências estão listas no projeto(composer.json, requirements.txt, package.json ou pom.xml)	| Ex: Sim/Não|
|Time de Desenvolvimento | Há alguma dependência de serviço por IP? Qual?	| Ex: Sim/Não|
|Time de Desenvolvimento | Há algum procedimento de deploy ou build que atualmente é realizado de forma manual? Qual?	| Ex: Sim/Não|
|Time de Desenvolvimento | Há monitoria através de Web Cenário	| Ex: Sim/Não|
|Time de Desenvolvimento | Json da monitoria	| Ex: {"status":"UP","resources":[{"name":"SERVICO_LINHA_DIRETA","type":"application","status":"UP"},{"name":"SERVICO_SCBA","type":"application","status":"UP"},{"name":"SERVICO_SADMIN","type":"application","status":"UP"},{"name":"SERVICO_SICAPES","type":"application","status":"UP"},{"name":"SERVICO_SUCUPIRA_PROGRAMAS","type":"application","status":"UP"},{"name":"SERVICO_SUCUPIRA_DOCENTES","type":"application","status":"UP"},{"name":"BANCO_DADOS","type":"db","status":"UP"}]}|
|Time de Desenvolvimento | Qual a cobertura de testes unitários	| Ex: 50%|
|Time de Desenvolvimento | Há testes de integração?	| Ex: Sim/Não|
|Time de Desenvolvimento | Qual o padrão de versionamento da aplicação	| Ex: SemVer MAJOR.MINOR.PATCH|
|Time de Desenvolvimento | Aplicação possui serviços e execuções em background? Qual?	| Ex: Sim/Não|
|Time de Desenvolvimento | Utiliza alguma ferramenta APM? Qual?	| Ex: Sim/Não|
|Time de Desenvolvimento | Utiliza alguma ferramenta de depuração? Qual?	| Ex: Sim/Não|
|Time de Desenvolvimento | DNS da aplicação	| Ex: app.capes.gov.br|
|Time de Desenvolvimento | Onde a aplicação ficará disponível	| Ex: Interno(CAPES e/ou Ip: XXX.XXX.XXX.XXX) e/ou Externamente|


