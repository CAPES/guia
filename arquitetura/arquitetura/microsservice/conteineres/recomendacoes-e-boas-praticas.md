# Contêineres - Recomendações & Boas Práticas

Contêineres são uma tecnologia poderosa e genérica, que permite uma vasta gama de possibilidades e permite as mais diversas formas de disponibilizar software.

Toda essa liberdade pode levar a alguns abismos inesperados.

Vamos catalogar aqui algumas boas práticas que encaramos como importantes e que deveriam ser utilizadas no desenvolvimento de aplicações no ecossistema da CAPES.

A recomendação é que as aplicações sejam aderentes a essas práticas e se adequem a elas o mais rápido possível.

# Geral

1. Evite Ctrl+C & Ctrl+V inescrupulosamente
2. Prefira `ENTRYPOINT`

## 1. Evite Ctrl+C & Ctrl+V inescrupulosamente

Poderia até seguir sem falar, mas é preciso tomar cuidado com o o que copiamos de algum lugar e colamos nos nossos projetos.

> O código fonte ao longo desse artigo mesmo é basicamente incompleto e serve de ilustração

1. Tome cuidado com as versões das imagens base ao copiar de um outro `Dockerfile`
2. Tome cuidado com as referências ao copiar configurações de um arquivo de `values-*.yml` para outro, diversas configurações podem ser específicas daquela implantação (_deployment_)

## 2. Prefira `ENTRYPOINT`

Usar `ENTRYPOINT`em vez de `CMD` afinal a imagem é o servidor de uma aplicação de backend

Exemplo (_Dockerfile_):

```Dockerfile
# ...
ENTRYPOINT ["./app.jar", "--spring.jta.atomikos.datasource.oracle.password=${DB_PASS}"]
```

# Java

## Maven

1. Utilizar um profile chamado "docker" para otimizar o artefato para a execução a aplicação em conteiner

### 1. Profile "docker"

Um [profile Maven](https://maven.apache.org/guides/introduction/introduction-to-profiles.html) pode ser utilizado para fazer ajustes no artefato construído pelo Maven de modo a rodar melhor dentro de um conteiner.

> Cada projeto vai ter seus próprios requisitos de como ajustar o artefato para rodar "conteneirizado"

1. `Dockerfile`: Modificar o estágio de construção para utilizar o profile Maven
2. `pom.xml`: Adicionar o profile _docker_ e fazer as configurações para o projeto

Exemplo (_Dockerfile_):
```Dockerfile
FROM registry.paas.capes.gov.br/base/maven-3.6-jdk-8:1.0.4 as build
#...
RUN mvn -B -Pdocker -Dmaven.test.skip=true package
#...
```

Exemplo (_pom.xml_):
```xml
<!-- ... -->
  <profiles>
    <profile>
      <id>docker</id>
      <build>
        <!-- ... -->
      </build>
    </profile>
  </profiles>
<!-- ... -->
```


## Aplicações Spring Boot

1. Rode a o JAR como executável
2. Configure a utilização de memória da JVM[^jvm-mem-at-docker]
3. Configure a JVM para rodar com o “Horário de Brasília”
4. Configure a variável de ambiente `JAVA_OPTS` nos arquivos de values
5. (Opcional) Minimize a quantidade de argumentos passada para o `ENTRYPOINT`

### 1. Rode o JAR como executável

Exemplo:
```Dockerfile
#...
ENTRYPOINT ["./app.jar", "--spring.jta.atomikos.datasource.oracle.password=${DB_PASS}"]
```

Isso permite melhor controle da aplicação pela pilha spring boot através do orquestrador, sem precisar reconstruir a imagem. Essa prática é uma direta consequência da recomendação sobre **Configurações** no [_Twelve Factors_](https://xpto.com/dti/orientacoes-gerais/guia/wikis/Devops/primeiros-passos/12factor).

### 2. Configure a utilização de memória da JVM[^jvm-mem-at-docker]

Adicione na variável de ambiente `JAVA_OPTS`

```shell
-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=1
```

Essa configuração permite uma utilização mais eficaz da memória disponibilizada no conteiner/pod.

### 3. Configure a JVM para rodar com o "Horário de Brasília"

```shell
-Duser.timezone=America/Sao_Paulo
```

### 4. Configure a variável de ambiente `JAVA_OPTS` nos arquivos de values

Nos arquivos de _values-\*.yml_ em `devops/backend/` configurar a variável de ambiente

Exemplo (_values-prod.yml_):
```yaml
app:
#...
  deployment:
  #...
    containers:
    #...
      environments:
      #...
      - name: JAVA_OPTS
        value: -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=1 -XX:+UseSerialGC -Duser.timezone=America/Sao_Paulo -Doracle.jdbc.timezoneAsRegion=false -Djava.security.egd=file:/dev/./urandom
```

Isso permite que o contexto da aplicação possa ser manipulado diretamente no orquestrador (k8s/Kubernetes, OKD/Openshift), sem que seja necessário modificar a base de código para parametrizar a execução

### 5. Minimize a quantidade de argumentos passada para o `ENTRYPOINT`

Para melhorar a legibilidade e aumentar o alinhamento com a recomendação de configuração é bom que o `ENTRYPOINT` tenha o mínimo de argumentos. Ele pode chegar ao ponto de ser apenas

```Dockerfile
ENTRYPOINT ['./app.jar']
```

1. Config Map com app.conf
2. Exportar senhas como variáveis de ambiente do Spring Boot

#### 5.1. Config Map com app.conf

_"Comming Soon..."_

#### 5.2. Exportar senhas como variáveis de ambiente do Spring Boot

_"Comming Soon..."_

## JBoss

TODO

## Outros contextos Java

Outras práticas que podem ser adotadas quando a aplicação ou sua construção é baseada em Java

### Utilizar a variável de ambiente `JAVA_TOOL_OPTIONS`[^java-tool-options]

Análogo com a opção de `JAVA_OPTS` que aplicações Spring Boot desde a versão 6 da JVM[^java-6-tool-options] a variável de ambiente `JAVA_TOOL_OPTIONS` pode ser utilizada para configurar argumentos para a JVM como por exemplo _agentlibs_

Exemplo (_values-prod.yml_):
```yaml
app:
#...
  deployment:
  #...
    containers:
    #...
      environments:
      #...
      - name: JAVA_TOOL_OPTIONS
        value: -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=1 -XX:+UseSerialGC -Duser.timezone=America/Sao_Paulo -Doracle.jdbc.timezoneAsRegion=false -Djava.security.egd=file:/dev/./urandom
```

Essa variável pode ser utilizada em conjunto ou como opção a utilização de `JAVA_OPTS` para aplicações na pilha Spring Boot.

# PHP

TODO

# Python

TODO

# Referências & Notas

[^jvm-mem-at-docker]: https://medium.com/@yortuc/jvm-memory-allocation-in-docker-container-a26bbce3a3f2
[^java-6-tool-options]: https://docs.oracle.com/javase/6/docs/platform/jvmti/jvmti.html#tooloptions
[^java-tool-options]: https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/envvars002.html
