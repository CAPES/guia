[[_TOC_]]
## [Boas práticas](#boas-praticas)
## [Otimizações](#otimizacoes)
## [Apoio ao desenvolvimento](#apoio-ao-desenvolvimento)
## [Autenticação e Autorização](#autenticacao-e-autorizacao)
## [Logging](#logging)
Lidar com o erros é algo que todo desenvolvedor deve ser capaz de fazer. 

Erros que são gerados no backend como de uma aplicação php ou java, estão sob um ambiente controlado, geralmente, o log gerado é direcionado para arquivos ou para um agregador de logs.

Entretanto não temos este mesmo benefício no frontend de aplicações SPA, uma vez que as aplicações são executadas no browser. O erro disparado, geralmente, é tratado pelo navegador, que envia o log para o console do browser.

![400px-DTI_Arquitetura_Frontend_angular_logging-erros-browser](uploads/06c327d71dc056590b04796b112ea2d0/400px-DTI_Arquitetura_Frontend_angular_logging-erros-browser.jpg)

Para que o time responsável pela manutenção da aplicação seja notificado que ocorreu um erro, é necessário a notificação a um servidor agregador de logs que tem o papel de receber os logs gerados no frontend e repassar para o backend para armazenamento e processamento.

Outra responsabilidade do servidor é o processamento do sourcemap para gerar uma _stacktrace_ com o erro localizado corretamente. Abaixo vemos a solução proposta:

![800px-DTI_Arquitetura_Frontend_angular_solucao-logging-javascript](uploads/9fe8e51db8f2122daac455b42a35f720/800px-DTI_Arquitetura_Frontend_angular_solucao-logging-javascript.png)

## [Acessibilidade](#acessibilidade)
## [Internacionalização](#internacionalizacao)
## [Localização](#localizacao)
## [Aplicações web progressivas](#aplicacoes-web-progressivas)
## [Testes](#testes)
## [UI/UX](ui-ux)
## [Caching](caching)
## [Variáveis de ambiente](#variaveis-de-ambiente)
## [Utilitários](#utilitarios)
## [Esquemas](#esquemas)

# Referências
* [https://medium.com/@tomastrajan/️-how-to-create-your-first-custom-angular-schematics-with-ease-️-bca859f3055d](Customização por esquemas em angular)
* [https://blog.angular.io/schematics-an-introduction-dc1dfbc2a2b2](schematics)
