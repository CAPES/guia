# Requisitos necessários para Aplicações OpenShift
O objetivo deste documento é descrever os requisitos mínimos para que uma aplicação da CAPES seja migrada para o ambiente Openshift. Este documento foi criado com base no manifesto The Twelve-Factor app (12factor)¹, que detalha boas práticas comuns a aplicações web modernas.


1. Base de código[^1]
   1. A base do código deve estar no GitLab (https://git.capes.gov.br/)
   1. A aplicação deve ter apenas uma base de código e, partindo dessa, ser implantada em distintos ambientes.

1. Dependência[^3]
   1. Declaração de todas as dependências necessárias para executar o código
   1. As dependências deverão ser declaradas nos seguintes arquivos:
        1. composer.json (PHP)
        1. package.json (Javascript)
        1. pom.xml (Java)
        1. requirements.txt (Python)
   1. As dependências ou libs não declaradas em um dos itens acima deverá ser colocada na imagem Docker base da aplicação.
   
1. Configurações[^1]
   1. Viabilizar a configuração da aplicação sem a necessidade de modificar o código
   1. As configurações deverão ser definidas através de variáveis de ambiente ou arquivos, desde que os mesmos não façam parte do código fonte da aplicação
   1. Deverá ser solicitado à equipe de banco de dados a criação de um resource do cofre de senha específico para o OpenShift.
   
1. Serviços de Apoio[^1]
   1. A aplicação deve estar pronta para receber parâmetros que configurem os serviços corretamente e, assim, possibilitem o consumo de aplicações necessárias à solução proposta.
   1. Não devem ser utilizados IPs como meio de configuração dos serviços, pois em algum momento os mesmos poderão rodar em containers.
   
1. Construa, lance, execute[^1]
   1. Construa - converter código do repositório em pacote executável. Nesse processo se obtém as dependências, compila-se o binário e os ativos do código.
   1. Lance - pacote produzido na fase construir é combinado com a configuração. O resultado é o ambiente completo, configurado e pronto para ser colocado em execução.
   1. Execute (também conhecido como “runtime”) - inicia a execução do lançamento (aplicação + configuração daquele ambiente), com base nas configurações específicas do ambiente requerido.
   
1. Processos[^3]
   1. A aplicação possa atender a picos de demandas com inicialização automática de novos processos, sem afetar seu comportamento.
   1. A boa prática indica que processos de aplicações são stateless (não armazenam estado) e share-nothing. Quaisquer dados que precisem persistir devem ser armazenados em serviço de apoio stateful (armazena o estado), normalmente é usado uma base de dados.
   
1. Vínculo de portas[^1]
   1. A aplicação em questão seja autocontido e dependa de um servidor de aplicação, tal como Jboss, Tomcat e afins. O software deve exportar um serviço HTTP e lidar com as requisições que chegam por ele. Significa que, qualquer aplicação adicional é desnecessária para o código estar disponível à comunicação externa.
   1. Tradicionalmente, a implantação de artefato em servidor de aplicação, tal como Tomcat e Jboss, exige a geração de um artefato e, esse, é enviado para o serviço web em questão. Mas no modelo de container Docker, a idéia é que o artefato do processo de implantação seja o próprio contêiner.
   
1. Concorrência[^3]
   1. A solução deve ser capaz de iniciar novos processos da mesma aplicação, caso necessário, sem afetar o produto
   
1. Descartabilidade[^3]
   1. A aplicação deve ser capaz de remover rapidamente processos defeituosos. Com objetivo de evitar que o serviço prestado seja dependente das instâncias que o servem.
   
1. Paridade entre desenvolvimento/produção[^1]
   1. A imagem seja construída e apenas seu status modifique para ser posta em produção. O código atual já está pronto para esse comportamento, assim, não há muito a ser modificado para garantir a boa prática.
   
1. Logs[^3]
   1. A aplicação não deve gerenciar ou rotear arquivos de log, mas deve ser depositados sem qualquer esquema de buffer na saída padrão (STDOUT).
   
1. Processos de administração[^1]
   1. Processos de administração executados em ambientes idênticos ao utilizado no código em execução, seguindo todas as práticas apresentadas acima.
   1. A aplicação deverá disponibilizar um serviço de Web Cenários para a monitoração do Zabbix.

---
[^1]: Itens obrigatórios de responsabilidade do time do projeto.
[^2]: Itens obrigatórios de responsabilidade do time DevOps.
[^3]: Itens recomendados, porém não obrigatórios.

---
# Referências
- [12factor](https://12factor.net/)
