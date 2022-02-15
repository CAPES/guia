# Visão Geral da Infraestrutura Tecnológica da CAPES
Aqui estão listadas as principais orientações da DTI no que tange o uso da infraestrutura do parque tecnológico do órgão, segmentado pelas vertentes abaixo.

<br><br>

# Automação
* [Ansible](automacao/ansible.md) - criação de ambientes e configuração de serviços utilizando receitas em **Ansible**.


<br><br>

# Segurança
Diretrizes sobre o assunto de segurança da informação, sobre:

* [Acesso SSH](seguranca/acesso-ssh.md) - método para **acesso, via protocolo ssh, nos servidores**, através de um gateway, utilizando login de rede.

* [Anti-SPAM](seguranca/antispam/README.md) - veja como utilizar o filtro de e-mails não solicitados (SPAM).

* [Logs de Eventos - SIEM](seguranca/logs.md) - envio e gerenciamento dos **logs** dos servidores, serviços e sistemas para a ferramenta de correlacionamento de eventos SIEM - *Security information and event management*.

* **Segredos e Senhas**
  * [Cofre de Senhas](seguranca/cofre-senhas.md) - gestão das **senhas de serviço** utilizadas para interconexão entre sistemas, serviços e bancos de dados, incluindo-se certificados digitais, tokens e outros segredos.
  * [Política de Senhas](seguranca/politica-senhas.md) - diretrizes utilizadas para a gestão das senhas (usuários e contas de serviço) do ambiente e procedimentos de alteração das senhas.
