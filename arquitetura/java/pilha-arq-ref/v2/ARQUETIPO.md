**Pilha Java para Arquitetura de Referência - versão 2.x**

# Leituras Recomendadas

- [Apache Maven - Introduction to Archetypes](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html)
- [Estrutura de pastas nos repositórios de aplicações da CGS](https://xpto.com/dti/orientacoes-gerais/guia/blob/master/ferramentas/version-control-system/guia-de-uso-git.md#reposit%C3%B3rio)

# Introdução

Um dos módulos Maven da pilha é um arquétipo para criar o _layout_ de arquivos e configurações básicas para novas aplicações. Então é possível criar uma estrutura mínima de uma aplicação já baseada na pilha na forma de um projeto "mavenizado".

# Dependências e Versionamentos

- Java: `SE 17` (Major: `61`)
- Maven: `3.8.5`
  - Plugins:
    - `org.apache.maven.plugins:maven-archetype-plugin`: `3.2.0`

> ## Versões na "_toolchain_"
> A versão 2.x da foi compilada em Java 17 e se espera que as aplicações utilizem essa mesma versão. Para que os módulos e configurações de construção funcionem adequadamente é necessário utilizar Maven `3.8.5` ou superior.

# Tutoriais

## Maven

> IDEs como Eclipse e IntelliJ são capazes de utilizar o arquétipo para criar os arquivos, mas como IDE é uma escolha primordialmente pessoal dos desenvolvedores, vamos apenas ilustrar usando a CLI do Maven (`mvn`).

**Exemplo**
```
mvn archetype:generate -B \
  -DarchetypeGroupId=br.gov.capes.cgs.narq \
  -DarchetypeArtifactId=archetype-minimal \
  -DarchetypeVersion=2.0.0-SNAPSHOT \
  -DgroupId=br.gov.capes \
  -DartifactId=c-a-s-b-app \
  -Dversion=1.0-SNAPSHOT \
  -Djava-path=/usr/lib/jvm/java-11-openjdk-amd64/bin/java
```

- `-B`: modo "batch"/"não iterativo" que não vai solicitar por dados ausentes
- `-DarchetypeGroupId=...`: o grupo do arquétipo a ser utilizado. **DEVE** ser `br.gov.capes.cgs.narq`
- `-DarchetypeArtifactId=...`: o artefato do arquétipo a ser utilizado. **DEVE** ser `archetype-minimal`
- `-DarchetypeVersion=...`: a versão do arquétipo a ser utilizado. Utilizar uma versão disponível da pilha
- `-DgroupId=...`: o grupo do projeto a ser criado pelo arquétipo. No mínimo `br.gov.capes`
- `-DartifactId=...`: a identificação da aplicação como **artefato** Maven
- `-Dversion=`: a versão no qual o projeto Maven será criado
- `-Djava-path=...`: caminho para o binário da JVM que se deseja que esse projeto Maven utilize[^mvn-configs]

**Outras propridades que podem ser utilizadas como parâmetros**:
- `git`: repositório git associado com o projeto
  - Exemplo: `-Dgit=git://xpto.com:cgs/narq/frameworks/java/capes-arquitetura-spring-boot.git`
- `descricao`: Descrição do projeto. Será colocada na tag `<description>`
  - Exemplo: `-Ddescricao="Uma descrição minimalista da aplicação."`
- `javac-path`: O caminho para o binário do compilador Java que se deseja que esse projeto Maven utilize[^mvn-configs]
  - Exemplo: `-Djavac-path="/usr/bin/javac"`

> Por enquanto estamos provendo apenas o arquétipo mínimo `br.gov.capes.cgs.narq:archetype-minimal`. Temos o receio de que as aplicações sejam criadas com um conjunto de módulos maior do que o necessário para seu domínio negocial e desenho de solução, por isso essa ideia de um arquétipo "mínimo" ser a base de todas as aplicações e elas adicionarem os módulos da pilha conforme se mostre necessário.

# Cuidados

- O projeto/módulo Maven criado a partir do arquétipo já contém uma suíte de teste com 01 caso de teste, _**intencionalmente**_ falhando. Lembre de escrever sua devida súite de testes e retirar essa falha intencional.

# Notas e Referências

[^mvn-configs]: Pasta `.mvn` e seus arquivos, como `.mvn/jvm.config` - https://maven.apache.org/configure.html
