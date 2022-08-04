# Pilha Spring Boot - Orientações Técnicas

## Swagger-UI
### Exibição de versão implantada

Aplicações implantadas no cluster OKD/Openshift utilizam uma arquitetura e ciclo de desenvolvimento diferente das que são implantadas, por exemplo, pelo Jenkins.

Nessa arquitetura a versão acaba não sendo um valor codificado no artefato final, mas sim controlado através da variável `VERSAO`.

A partir da versão 1.1.7 da pilha, a versão exibida no Swagger-UI é que está definida na variável de ambiente. Caso a variável não esteja disponível, o valor vai ser extraído normalmente do jar que é o artefato da aplicação. **Para que versão exibida seja a mesma da tag implantada, basta que a aplicação seja modificada para usar a versão 1.1.7, ou maior, da pilha como dependência**.