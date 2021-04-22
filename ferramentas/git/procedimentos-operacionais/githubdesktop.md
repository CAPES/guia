[[_TOC_]]

# Como Utilizar o GitHub Desktop

Alguns procedimentos na utilização do GitHub Desktop.





## Login com TFA Habilitado

Para usuários que estão utilizando o recurso de segundo fator de autenticação (TFA) em sua conta, o método de abrir um repositório no Git da Capes precisa de alguns passos extras, pois o GitHub Desktop apena "pergunta" a senha ao tentar acessar o repositório.

**Personal Access Tokens** 

Você precisa criar um *token* de acesso que será utilizado como uma senha. Para isso, nas configurações do Git, na sessão de [*token* de acesso nas configurações da sua conta](https://git.capes.gov.br/profile/personal_access_tokens), crie um *token* com o escopo um dos escopos: ` read_repository ` ou ` write_repository `.

> O *token* gerado não ficará visível no sistema. Despois de gerado, salve em lugar seguro, pois poderá ser utilizado para **acessar seus repositórios sem o TFA**. 
![image](uploads/0ddead9c897876392a8173d9376420ad/image.png)


**Clone de um Repositório** 

Agora no GitHub Desktop, ao clonar um repositório na tela de login/senha, preencha com seu login e no lugar da senha, coloque o ***token*** que foi gerado.