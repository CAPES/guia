# Visão Geral
No gerenciamento de um ambiente computacional com certa complexidade, o gerenciamento das senhas de serviço (senhas não pessoais) que integram os serviços, sistemas e bancos de dados, se faz necessário. O uso de planilhas, e-mail e ferramentas descentralizadas trazem vulnerabilidades problemas na gestão de forma profissional. Com isto, a DTI/CGII implantou um **[cofre de senhas](https://git.capes.gov.br/cgii/seguranca/pmp)** (acesso restrito), utilizando o software **Password Manager Pro** (PMP), que centraliza tais dados sensíveis e permite o compartilhamento das contas com colaboradores e também através do uso de API, para as automações necessárias.

<br><br>

# Normas Gerais de Uso

* **Complexidade da senha (PORTARIA EM CONSTRUÇÃO)** - define o número mínimo de caracteres que a senha deve conter.

* **Cadastro e Compartilhamento da senha** - Cada área - CGS (sistemas), CGII (infraestrutura) e NDAC (Banco de Dados) - tem um administrador que é responsáveis por gerenciar o cadastro e compartilhamento necessário das senhas. Via de regra, o compartilhamento é feito da seguinte forma:

  * **Para acesso ao banco de dados** - as senhas que são cadastradas no *datasource* da aplicação são criadas pela área de **banco de dados** e compartilhadas com:

    * **Equipe GCM** - para a configuração das aplicações de ambiente DHT (não produção).
* **Equipe Infraestrutura** - para a configuração das aplicações de ambiente de produção.
    * **Automação via API** - para autoconfiguração
* **Contas locais, super usuários e senhas de certificados ** - as senhas locais de administração (root, admin, sa, administrador...), normalmente criadas na instalação de um serviço, precisam estar registradas no cofre de senhas e serão compartilhadas apenas com os administradores respectivos daquela ferramenta, **quando estritamente necessário**. Os sistemas que permitem a criação de contas pessoais (ou integradas ao AD), com perfil de administrador devem ser utilizadas sempre que possível.
  
* **Alteração / Exclusão da senha** - a alteração da senha requer atualização tanto no cofre de senhas quanto na aplicação, sendo necessário o [registro de uma mudança](https://git.capes.gov.br/cgii/ccm/gmud/wikis/home), aplicando-se também para o caso de desativação de uma senha.

> :blue_book: As exceções a estas regras devem ser registradas via CATI sendo necessário autorização da coordenação.

As normas específicas de uso podem ser encontradas na [documentação da solução](https://git.capes.gov.br/cgii/seguranca/pmp/wikis/home) (acesso restrito).