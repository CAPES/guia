[[_TOC_]]

# Política de Uso do Openshift

## Introdução
Este tem por objetivo definir quem pode fazer o quê no ambiente do Openshift.

## Perfis padrões do Openshift

O Openshift traz por padrão em sua versão 3.11, que é a adotada pela CAPES, alguns perfils padrões que possuem as seguintes atribuições:

| Perfil | Descrição |
| :---: | :--- |
| **admin**             | Um gerente de projeto. Se usado em uma ligação local , um usuário administrador terá direitos para visualizar qualquer recurso no projeto e modificar qualquer recurso no projeto, exceto para a cota. |
| **basic-user**        | Um usuário que pode obter informações básicas sobre projetos e usuários. |
| **cluster-admin**     | Um superusuário que pode realizar qualquer ação em qualquer projeto. Quando vinculados a um usuário com uma vinculação local , eles têm controle total sobre a cota e todas as ações em todos os recursos do projeto. |
| **cluster-status**    | Um usuário que pode obter informações básicas de status do cluster. |
| **edit**              | Um usuário que pode modificar a maioria dos objetos em um projeto, mas não tem o poder de visualizar ou modificar roles or bindings. |
| **self-provisioner**  | Um usuário que pode criar seus próprios projetos. |
| **view**              | Um usuário que não pode fazer nenhuma modificação, mas pode ver a maioria dos objetos em um projeto. Eles não podem visualizar ou modificar roles or bindings. |
| **cluster-reader**    | Um usuário que pode ler, mas não visualizar, objetos no cluster. |


## Perfis da CAPES vs Perfis do Openshift

Para o contexto da CAPES ficou definido os seguintes papeis:

|                             | **admin** | **view**     | **cluster-admin** |
| ------                      | :------:  | :------:     | :------:          | 
| **Arquitetura**             |           |              | x                 |  
| **Devops**                  |           |              | x                 |
| **GCM**                     | x         |              |                   | 
| **Gerente do Projeto**      |           | x            |                   | 
| **Infraestrutura**          |           |              | x                 |  
| **Líder Técnico***          | x         |              |                   |
| **Time de Desenvolvimento** |           | x            |                   |

> * Administrador de Projeto (no máximo 2 líderes de projetos indicados pela coordenação imediata responsável pelo projeto). Nestes casos todos deve utilizar duplo fator de autenticação.


# Referências

- [Doc. Openshift](https://docs.openshift.com/container-platform/3.11/architecture/additional_concepts/authorization.html#roles)