# Aplicações Java EE com frontend Angular

## Utilizando `gulp`[^gulpjs]

- A versão do gulp depende da versão do `node`/`npm` que a aplicação utiliza
- A configuração de versão de `node`/`npm` **deve** ser feita utilizando o `nvm`[^nvm]

### Jobs Jenkins

> Reponsável: **GCM**

#### Job de Release

Basicamente deve seguir a estrutura básica de construção de releases de projetos "Mavenizados"[^jenkins-job-release-mvn]

- **Build a maven2/3 project**
- **Build**
  - **Goals and options**
- **Build Environment**
  - [x] Configure release build
  - **Release parameters**
    - branch com a base de código
    - releaseVersion
    - developmentVersion
    - username
    - password
  - After successful release build
    - **Invoke top-level Maven targets**
  - After failed release build
    - **Invoke top-level Maven targets**

##### Build a maven2/3 project

O Job deve ser criado como um de projeto Maven.

> Colocar um print? [Colocar uma nota recomendando copiar do projeto maven Release]

##### Build

O build usa a convenção de que configurações especiais para a geração de releases devem utilizar um profile Maven chamado `release`, o qual deve ser definido e configurado nos `pom.xml` do projeto.

###### Goals and options

Rodar os alvos `release:clean` e `release:prepare` mais todos os devidos parâmetros, como a ativação do profile `release`.

**Exemplo:**
```sh
-Prelease  release:clean release:prepare -D${username} -D{password} -Dbranch.url=http://svn.capes.gov.br/svn/CATALOGO-TESES/branches/${escolha}
```

> Os jobs de muitos projetos foram configurados para rodar como build os alvos `clean` e `install` e configuram um **Invoke top-level Maven targets** num 'Before release build'. Isso é desnecessário. Os alvos do plugin de release passam por todas as fases necessárias para a construção. Utilizar a configuração faz com que o projeto seja construído 3 vezes. O ideal seria que o job de release construísse os artefatos do projeto uma única vez, o que essa configuração também não alcaça, já que roda 2 vezes a construição dos artefatos, uma em `release:prepare` e outra em `release:perform`.

**Parâmetros:**
- `-Prelease`: ativa o profile `release`;
- `-Dbranch.url=http://svn.capes.gov.br/svn/CATALOGO-TESES/branches/${escolha}`: configura a propriedade `branch.url`, com uma URI composta pelo caminho de branches do projeto e a escolha selacionada na solicitação de execução do job.
  - No exemplo está o caminho para o Catalogo de Teses, modificar para a URI adequada.

##### Build Environment

###### After successful release build

Adicionar um **Invoke top-level Maven targets** utilizando o alvo de `release:perform` e passando todos os parâmetros necessários, incluindo a ativação do profile `release`.

**Exemplo:**
```sh
-Prelease release:perform -D${username} -D{password} -Dbranch.url=http://svn.capes.gov.br/svn/CATALOGO-TESES/branches/${escolha} -Dmaven.test.skip.exec=true
```

###### After failed release build

Adicionar um **Invoke top-level Maven targets** utilizando o alvo de `release:rollback` e passando todos os parâmetros necessários, incluindo a ativação do profile `release`.

**Exemplo:**
```sh
-Prelease  release:rollback -D${username} -D{password}
```

> O `release:rollback` reverte a versão no `pom.xml` do projeto, mas não elimina a tag criada para guardar essa versão da base de código. Essa tag provavelmente tem de ser removida manualmente, ou adicionado uma invocação do `svn` para realizar essa deleção - ainda não testamos essa automatização.

#### Job de Deploy

> Seguem a estrutura utilizando o script

- **Build**
  - **Artifact Resolver**
- **Post-build Actions**
  - **Post build task**

##### Build

###### Artifact Resolver

- [x] Fail on error
- Release update policy: _Always_
- Snapshot update policy: _Always_
  - **Os jobs de deploy em Desenvolvimento _principalmente_ devem ter essa configuração como _Always_, pois é o ambiente que recebe SNAPSHOTS**

##### Post-build Actions

###### Post build task

- Script:
  1. Remover a versão atual do(s) artefatos(s) na pasta de implantação dos artefatos
  2. Copiar o(s) artefatos resolvidos para pasta de implantação
  3. Reiniciar os servidor de aplicação
  - **Exemplo**:
```sh
sudo rm -f /CAPES/AplWeb/Desenv/Java/catalogo-teses/deployments/*.war* || true
sudo cp -u -f catalogo-teses.war /CAPES/AplWeb/Desenv/Java/catalogo-teses/deployments/ || true

/opt/restart_cluster.sh des.capes.gov.br catalogo-teses
```
- [x] Run script only if all previous steps were successful
- [x] Escalate script execution status to job status

### Confugurações Maven

> Reponsável: **Time de Desenvolvimento**

> Informar as limitações conhecidas para explicar os motivos
> versão do plugin de release

Para que o projeto possa ser devidamente construído as configurações do Maven nos arquivos `pom.xml` devem estar alinhadas com as convenções adotadas no Job de Release.

> Muito provavelmente as configurações mencionadas vão exigir adaptações e extensões, por isso é fundamental o bom entendimento do funcionamento do Maven[^maven-docs]

#### Configurações Básicas

- Configurar a exibição de SQL pelo Hibernate nos logs
- Configuração do `<scm>`
- Versão do plugin `org.apache.maven.plugins:maven-release-plugin`
- Limpeza da pasta de `build` do **Angular**
- Configurar invocação de `npm install`
- Configurar invocação do `gulp`
- Profile `release`
  - Reconfigurar a verbosidade do Hibernate
  - Reconfigurar a utilização de `npm install` e `gulp prod`

##### Configurar a exibição de SQL pelo Hibernate nos logs

Vamos utilizar uma funcionalidade fundamental do Maven para controlar o valor da propriedade `hibernate.show_sql` no arquivo `META-INF/persistence.xml` através da "filtragem" de arquivos de _resources_. Em `META-INF/persistence.xml` definimos a propriedade para ser substituida pelo valor de uma propriedade no Maven:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.0"
  xmlns="http://java.sun.com/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="
        http://java.sun.com/xml/ns/persistence
        http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd">
  <persistence-unit name="default" transaction-type="JTA">
    <!-- outras configurações -->
    <properties>
      <!-- outras propriedades... -->
      <property name="hibernate.show_sql" value="${hibernate.show-sql}"/>
    </properties>
  </persistence-unit>
</persistence>
```

A versão (`version`) do `persistence` depende da versão do JBoss/JavaEE sendo utilizada.

Configurar a proprieda em `pom.xml` e configurar para que esse resource seja "filtrado":

```xml
  <properties>
    <!-- provavelmente mais propriedades -->
    <hibernate.show-sql>true</hibernate.show-sql>
  </properties>
  <!-- outras configurações -->
  <build>
    <finalName>catalogo-teses</finalName>
    <resources>
      <!-- recursos modificáveis pelo Maven -->
      <resource>
        <directory>${project.basedir}/src/main/resources</directory>
          <filtering>true</filtering>
          <includes>
            <include>META-INF/persistence.xml</include>
          </includes>
        </resource>
      <!-- recursos NÃO-modificáveis pelo Maven -->
      <resource>
        <directory>${project.basedir}/src/main/resources</directory>
        <filtering>false</filtering>
        <excludes>
          <!-- Excluir os arquivos incluídos no outro recurso -->
          <exclude>&gt;META-INF/persistence.xml</exclude>
        </excludes>
      </resource>
    </resources>
  <!-- outras configurações -->
  </build>
  <!-- outras configurações -->
```

##### Configuração do `<scm>`

O Job de Release define uma propriedade `branch.url`, por isso vamos utilizar essa propriedade para configurar a tag XML de SCM:

```xml
  <!-- ${branch.url} é reconfigurado no job de release no Jenkins -->
  <scm>
    <connection>scm:svn:${branch.url}</connection>
    <developerConnection>scm:svn:${branch.url}</developerConnection>
    <url>${branch.url}</url>
  </scm>
```

#####  Versão do plugin `org.apache.maven.plugins:maven-release-plugin`

A configuração de propriedade só com o plugin `org.apache.maven.plugins:maven-release-plugin` em versões um pouco, masi modernas. Digamos que selecionamos a versão `2.5.1` dele, temos de configurar isso em `pom.xml`:
```xml
  <!-- outras configurações -->
  <build>
    <!-- outras configurações -->
    <plugins>
      <!-- outros pluigins -->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-release-plugin</artifactId>
      </plugin>
    </plugins>
    <!-- outras configurações -->
    <pluginManagement>
      <plugins>
        <!-- outros pluigins -->
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-release-plugin</artifactId>
          <version>2.5.1</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
  <!-- outras configurações -->
```

##### Limpeza da pasta de `build` do **Angular**

Digamos que o conteúdo Angular esteja na pasta `src/main/frontend`. Nela que ficaram coisas como `package.json` e `package-lock.json`. A pasta de `build` do Angular provavelmente será uma subpasta dela. Por ser uma subpasta é importate que a pasta `build` em `src/main/frontend` seja ignorada pelo Sistema de Versionamento de Código (arquivo `.gitignore` ou propriedade `svn:ignore`) - **caso não seja devidamente ignorado é bem possível que o alvo `release:prepare` invocado no Job falhe ao detectar modificações não "commitadas"**.
Vamos configurar em `pom.xml` uma execução do plugin `org.apache.maven.plugins:maven-clean-plugin` para lidar com a pasta `build`:

```xml
  <!-- outras configurações -->
  <properties>
    <!-- provavelmente mais propriedades -->
    <frontend.src.dir>src/main/frontend</frontend.src.dir>
  </properties>
  <!-- outras configurações -->
  <build>
    <!-- outras configurações -->
    <plugins>
      <!-- outros pluigins -->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-clean-plugin</artifactId>
        <executions>
          <execution>
            <id>clean-js</id>
            <phase>clean</phase>
            <goals>
              <goal>clean</goal>
            </goals>
            <configuration>
              <filesets>
                <fileset>
                  <directory>${frontend.src.dir}/build</directory>
                </fileset>
              </filesets>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
    <!-- outras configurações -->
    <pluginManagement>
      <plugins>
        <!-- outros pluigins -->
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.1.0</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
  <!-- outras configurações -->
```

A propriedade `frontend.src.dir` será útil em outros pontos

> **Por que não vamos limpar a pasta `node_modules`?**
> Essa pasta costuma ser massiva, e rodar um `npm install` com ela vazia costuma ser muito demorado. O que poderia ser feito seria adicionar uma execução especial para isso[^mvn-plugin-execution] `mvn clean:clean@clean-node-modules`.

##### Configurar invocação de `npm install`

É preciso rodar um `npm install` na pasta `src/main/frontend`, que definimos na propriedade `frontend.src.dir`. Vamos configurar o plugin `org.codehaus.mojo:exec-maven-plugin` em `pom.xml` para realizar isso:

```xml
  <!-- outras configurações -->
  <properties>
    <!-- provavelmente mais propriedades -->
    <npm-install-phase>generate-resources</npm-install-phase>
  </properties>
  <!-- outras configurações -->
  <build>
    <!-- outras configurações -->
    <plugins>
      <!-- outros pluigins -->
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <executions>
          <execution>
            <id>npm-install</id>
            <phase>${npm-install-phase}</phase>
            <goals>
              <goal>exec</goal>
            </goals>
            <configuration>
              <executable>npm</executable>
              <arguments>
                <argument>install</argument>
              </arguments>
              <workingDirectory>${frontend.src.dir}</workingDirectory>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
    <!-- outras configurações -->
    <pluginManagement>
      <plugins>
        <!-- outros pluigins -->
        <plugin>
          <groupId>org.codehaus.mojo</groupId>
          <artifactId>exec-maven-plugin</artifactId>
          <version>3.0.0</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
  <!-- outras configurações -->
```

A propriedade `npm-install-phase` será útil para controlar as operações no contexto do Job de Release quando o profile `release` estiver ativo.

##### Configurar invocação do `gulp`

As configurações específicas do `gulp` não vão ser discutidas aqui, mas digamos que o comando `gulp prod` seja o que compila o Angular para Javascript com todas as otimizações desejadas. Vamos configurar a invocação desse comando em `pom.xml` através de uma execução no plugin `org.codehaus.mojo:exec-maven-plugin` (análogo ao que foir feito pra `npm install`):

```xml
  <!-- outras configurações -->
  <properties>
    <!-- provavelmente mais propriedades -->
    <gulp-prod-phase>generate-resources</gulp-prod-phase>
  </properties>
  <!-- outras configurações -->
  <build>
    <!-- outras configurações -->
    <plugins>
      <!-- outros pluigins -->
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <executions>
          <!-- outras execuções -->
          <execution>
            <id>gulp-prod</id>
            <phase>${gulp-prod-phase}</phase>
            <goals>
              <goal>exec</goal>
            </goals>
            <configuration>
              <executable>gulp</executable>
              <arguments>
                <argument>prod</argument>
              </arguments>
              <workingDirectory>${frontend.src.dir}</workingDirectory>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
    <!-- outras configurações -->
  </build>
  <!-- outras configurações -->
```

A propriedade `npm-install-phase` será útil para controlar as operações no contexto do Job de Release quando o profile `release` estiver ativo.

##### Profile `release`

###### Reconfigurar a verbosidade do Hibernate

Alterar a configuração do profile `release` em `pom.xml`

```xml
  <profiles>
    <profile>
      <id>release</id>
      <properties>
        <hibernate.show-sql>false</hibernate.show-sql>
      </properties>
    </profile>
  </profiles>
```

###### Reconfigurar a utilização de `npm install` e `gulp prod`

É necessário utilizar uma estratégia alternativa para a execução no Job de Release. Como cada aplicação pode utilizar uma versão diferente de `node`/`npm`, devemos utilizar o `nvm` para ativar a versão correta durante a execução do Job de Release.

O Job roda numa estação GNU/Linux, por isso vamos acrescentar na raiz do projeto (ou módulo) um script `release.sh`:
```sh
export NVM_DIR="$HOME/.nvm" && [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
nvm use 6.10.2

npm install
gulp prod
```
Selecionando a versão de `node`/`npm` que o projeto deseja utilizar, `v6.10.2` no caso do exemplo.

Adicionar a redefinição do valor das propriedades de phase das execuções de `npm install` e `gulp` no profile `release` em `pom.xml`:

```xml
  <profiles>
    <profile>
      <id>release</id>
      <properties>
        <hibernate.show-sql>false</hibernate.show-sql>
        <gulp-prod-phase>none</gulp-prod-phase>
        <npm-install-phase>none</npm-install-phase>
      </properties>
    </profile>
  </profiles>
```

Adicionar a configuração da execução do script `release.sh`:
```xml
<profiles>
    <profile>
      <id>release</id>
      <properties>
        <hibernate.show-sql>false</hibernate.show-sql>
        <gulp-prod-phase>none</gulp-prod-phase>
        <npm-install-phase>none</npm-install-phase>
      </properties>
      <build>
        <plugins>
          <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>exec-maven-plugin</artifactId>
            <executions>
              <execution>
                <id>gui-release</id>
                <phase>generate-resources</phase>
                <goals>
                  <goal>exec</goal>
                </goals>
                <configuration>
                  <executable>bash</executable>
                  <arguments>
                    <argument>${project.basedir}/release.sh</argument>
                  </arguments>
                  <workingDirectory>${frontend.src.dir}</workingDirectory>
                </configuration>
              </execution>
            </executions>
          </plugin>
        </plugins>
      </build>
    </profile>
  </profiles>
```

> **Por que não utilizamos o script `release.sh` para a construção padrão, de SNAPSHOTs, por exemplo?**
> Pois isso traria impactos para a execução em ambiente Windows. Da forma que está estruturado desde que os binários de `npm` e `gulp` estejam disponíveis no ambiente.

# Notas e Referências

[^jenkins-job-release-mvn]: Como apoio/exemplo criamos um job de exemplo: http://jenkins.capes.gov.br/job/Maven%20Release/configure
[^maven-docs]: https://maven.apache.org/
[^gulpjs]: https://gulpjs.com/
[^nvm]: https://github.com/nvm-sh/nvm
[^mvn-plugin-execution]: Aparentemente só disponível a partir do Maven `3.3.1`, utilizamos o `3.0.5` no Jenkins - https://stackoverflow.com/questions/3166538/how-to-execute-maven-plugin-execution-directly-from-command-line
