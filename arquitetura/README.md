# Introdução

Seção destinada a documentação e definição do Núcleo de Arquitetura.

# Definições

- [Definições para construção de API REST](rest-apis.md)
- [Eventos & Mensageria entre aplicações](definicoes/eventos/README.md)
- [Microsserviços](microsservices.md)
- [Especificações de Monitoramento de Aplicações](arquitetura/monitoramento-aplicacoes.md)
- [Formato de Logging](definicoes/logging/README.md)

# Tecnologias

## Backend

### Java
- Pilha na Arquitetura de Referência
  - [Versão 1][0ce381c9]
    - Leitura recomendada: [Entendendo o Projeto](https://wiki.capes.gov.br/index.php/DTI:Arquitetura_Servicos_Java_Guia_Programacao_Entendendo_Projeto)
  - [Versão 2](java/pilha-arq-ref/v2/README.md)
- JBoss
  - [JMX remoto no JBoss EAP 6](jboss/jmx-remote-jboss-eap6.md)
- Spring Boot
  - [Orientações Técnicas](java/springboot/orientacoes-tecnicas.md)
  - [Logging de aplicações](java/springboot/logback.md)

  [0ce381c9]: https://wiki.capes.gov.br/index.php/DTI:Arquitetura_Servicos_Java "Ligação externa - Wiki da DTI"

### PHP

- [Laravel](php/laravel)
- Zend
- Symfony


## [Frontend](frontend/README.md)
- [Angular](frontend/angular.md)

## Ferramentas

  - [Nexus - Gerenciador de Respositórios de Artefatos](ferramentas/nexus.md)

# Instruções Técnicas

  - [Construção & Implantação de Aplicações](arquitetura/arquitetura/release-and-deploy.md)

