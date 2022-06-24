**Pilha Java para Arquitetura de Referência - versão 2.x**

# Leituras Recomendadas

- [Writing Clean Tests – Small Is Beautiful](https://www.petrikainulainen.net/programming/testing/writing-clean-tests-small-is-beautiful/)[^testes-aninhados-junit5]
- [Apache Maven Surefire](https://maven.apache.org/surefire/)
  - [Maven Surefire Plugin](https://maven.apache.org/surefire/maven-surefire-plugin/index.html)
  - [Maven Failsafe Plugin](https://maven.apache.org/surefire/maven-failsafe-plugin/index.html)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Spring Boot - Testing][dcab3733]

  [dcab3733]: https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing "Spring Boot - Core Features - Testing"

# Introdução

Toda aplicação deveria ter testes automatizados escritos e executados durante seu processo de desenvolvimento[^obs1]

> ## Conceitualização
>
> Conceitos como as diferentes categorias de testes (**Testes de Unidade**, **Testes de Integração**, **Testes _End-to-End_**/**E2E**, **Testes _Edge-to-Edge_**/**e2e**, **Testes de Carga**, **Testes de Stress**, etc.) podem ser mencionados ou brevemente explicados nesse texto, mas a devida documentação desses conceitos foge ao escopo desse material. **Recomendamos _veementemente_ a familiarização dos desenvolvedores com as práticas e conceitos de testes automatizados**.

# Dependências e Versionamentos

- Framework de Testes: JUnit 5 (`5.7.2`)
  - Jupiter: `5.7.2`
  - Platform: `1.7.2`
- Framework de Testes: Apache Maven Surefire
  - Maven Surefire Plugin: `3.0.0-M5`
  - Maven Failsafe Plugin: `3.0.0-M5`
- Framework de _mocking_: Mockito (`3.11.2`)
  - `org.mockito:mockito-core`
  - `org.mockito:mockito-junit-jupiter`
- JMeter: `5.4.1`
  - Plugin Maven: `com.lazerycode.jmeter:jmeter-maven-plugin:3.4.0`

# Tutoriais

## Maven

Adicionar no `pom.xml` a dependência `br.gov.capes.cgs.narq:testes` na versão adequada:

```xml
<dependency>
  <groupId>br.gov.capes.cgs.narq</groupId>
  <artifactId>testes</artifactId>
  <version>${testes.version}</version>
  <scope>test</scope>
</dependency>
```

A propriedade `${testes.version}` deve ser definida criando a tag `<testes.version>` na tag `<properties>` do `pom.xml`. O valor dever ser o valor da versão da pilha[^obs-versao-no-archetype].

### _Parent_ `br.gov.capes:arquitetura-backend-spring-boot`

Caso o projeto tenha sido configurado para utilizar o _parent_ da pilha, como no caso em que é criado através do arquétipo[^obs-versao-no-archetype], algumas configurações e premissas estão estabelecidas:

#### Testes Unitários

- Execução de **Testes Unitários** faz parte do ciclo de construção[^maven-lifecycles]
- Arquivos escaneados como suítes de **Testes Unitários**:
  - `**/Test*.java`
  - `**/*Test.java`
  - `**/*Tests.java`
  - `**/*TestCase.java`

#### Testes de Integração

- Execução de **Testes de Integração** faz parte do ciclo de construção[^maven-lifecycles]
- Arquivos escaneados como suítes de **Testes de Integração**:
  - `**/IT*.java`
  - `**/*IT.java`
  - `**/*ITCase.java`

#### Spring Boot Test Starter

Para utilizar os starter de testes do Spring Boot adicionar no `pom.xml`:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>
```

> `org.springframework.boot:spring-boot-starter-test` é uma dependência opcional de `br.gov.capes.cgs.narq:testes`, por isso tem de ser declarado. No caso de projetos gerados a partir do arquétipo descomentar a dependência no `pom.xml` criado.

## Spring Boot

Não adicionamos extensões/configurações especiais que tenham de ser configuradas para o Spring Boot. Para _features_ avançadas que facilitem testes de integração de aplicações Spring Boot insistimos na recomendação da leitura de ["Spring Boot - Testing"][dcab3733].

## Java

Adotamos como _framework_ de testes o JUnit na versão 5.

### JUnit 5

#### Testes Hierarquizados

Reforçamos a utilidade da leitura de [Writing Clean Tests – Small Is Beautiful](https://www.petrikainulainen.net/programming/testing/writing-clean-tests-small-is-beautiful/), mas destacamos os seguintes pontos:

- Casos de teste com basicamente uma assertiva;
- Hierarquização de configuração.

A API de testes do JUnit 5 inclui a anotação `@Nested`[^junit5-nested-javadoc] que permite estruturar os testes conforme os pontos destacados. **Recomendamos fortemente essa estilistica de testes**.

```java
import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Test;

class RestEndpointTest {

  @BeforeAll
  static void setUpBeforeClass() throws Exception {
  }

  @AfterAll
  static void tearDownAfterClass() throws Exception {
  }

  @BeforeEach
  void setUp() throws Exception {
  }

  @AfterEach
  void tearDown() throws Exception {
  }

  @Test
  void test() {
    fail("Not yet implemented");
  }

  @Nested
  public class Listagem {

    @BeforeEach
    void setUp() throws Exception {
      //...
    }

    @Nested
    public class SemFiltros {

      @BeforeEach
      void setUp() throws Exception {
        //...
      }

      @Test
      void test() throws Exception {
        //...
      }

    }

    @Nested
    public class ComFiltros {

      @BeforeEach
      void setUp() throws Exception {
        //...
      }

      @Nested
      public class DeData {

        @BeforeEach
        void setUp() throws Exception {
          //...
        }

        @Test
        void test() throws Exception {
          //...
        }

        @Nested
        public class Invalidas {

          @Nested
          public class ForaDoPeriodo {

            @BeforeEach
            void setUp() throws Exception {
              //...
            }

            @Test
            void test() throws Exception {
              //...
            }

          }

        }

      }

    }

  }

}
```

#### _Extensions_

Um dos adventos do JUnit 5 foram as _Extensions_[^junit5-user-guide] que podem ser utilizadas para adicionar diversas capacidades as suítes de testes. A utilização de _Extensions_ é trivial através da utilização da anotação `@ExtendWith`[^junit5-ExtendWith-javadoc].

Algumas das extensões que consideramos úteis para o desenvolvimento de aplicações na Arquitetura de Referência:

- Mockito
- Spring Boot

---

##### Mockito Extention

Mockito[^mockito] é um framework que serve para mitigar a necessidade de acessar elementos fora da unidade que está sendo testada.

```java
import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.junit.jupiter.MockitoExtension;

@ExtendWith(MockitoExtension.class)
class MockitoTest {

  @Mock
  private Simples instancia;

  @BeforeAll
  static void setUpBeforeClass() throws Exception {
  }

  @AfterAll
  static void tearDownAfterClass() throws Exception {
  }

  @BeforeEach
  void setUp() throws Exception {
    Mockito.when(this.instancia.getName()).thenReturn("hello");
  }

  @AfterEach
  void tearDown() throws Exception {
  }

  @Test
  void test() {
    assertEquals("hello", this.instancia.getName());
  }

}
```

##### Spring Boot Test & Spring Boot Extention

O _starter_ `org.springframework.boot:spring-boot-starter-test` disponibiliza uma porção de utilitários para testes que fazem uso de funcionalidades e capacidades dos _frameworks_ Spring.

- Os testes utilizando essas capacidades são testes "_edge to edge_" (no mínimo, ou até "_end to end_"), então devem ser executados pelo `failsafe` na fase `integration-test`
    - `**/*IT.java`

**Injeção de Dependências**

Fazendo corretamente é possível injetar os _beans_ do contexto do Spring nas classes de teste:

```java
import static org.junit.jupiter.api.Assertions.assertNotNull;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.test.context.junit.jupiter.SpringExtension;

import com.fasterxml.jackson.databind.ObjectMapper;

@ExtendWith(SpringExtension.class)
class SpringBootedIT {

  @TestConfiguration
  static class Configs {

    @Bean
    public ObjectMapper mapper() {
      return new ObjectMapper();
    }

  }

  @Autowired
  private ObjectMapper mapper;

  @Test
  void test() {
    assertNotNull(this.mapper);
  }

}
```

> **`@SpringBootTest` vs `@ExtendWith(SpringExtention.class)`**
> `@SpringBootTest` necessita de uma aplicação (algo anotado com `@SpringBootApplication`) e vai basicamente carregar todo seu contexto de aplicação. Caso seja necessário apenas algumas como injeção de dependências (como quando usando `@Autowired`), utilizar `@ExtendWith(SpringExtention.class)` é provavelmente suficiente.
> A implementação de `@SpringBootTest` incorpora `@ExtendWith(SpringExtention.class)` e outras anotações. Normalmente será mais prático simplesmente utilizar `@SpringBootTest`.

**Mock Beans**

É possível fazer com que determinados _beans_ sejam estabelecidos como _mocks_ utilizando a anotação `@MockBean`:

```java
import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mockito;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.core.env.Environment;
import org.springframework.test.context.junit.jupiter.SpringExtension;

@ExtendWith(SpringExtension.class)
class SpringBootedAndMockedIT {

  @MockBean
  private Environment env;

  @BeforeAll
  static void setUpBeforeClass() throws Exception {
  }

  @AfterAll
  static void tearDownAfterClass() throws Exception {
  }

  @BeforeEach
  void setUp() throws Exception {
    Mockito.when(this.env.getProperty("chave")).thenReturn("valor");
  }

  @AfterEach
  void tearDown() throws Exception {
  }

  @Test
  void test() {
    assertEquals("valor", this.env.getProperty("chave"));
  }

}
```

Se um determinado **componente** tem uma dependência `@Autowired` ela pode ser satisfeita através de um atributo anotado com `@MockBean`. Assim o comportamento da dependência injetada pode ser controlada pelo **Mockito**, para tornar o comportamento do **componente** correto e previsível/determinístico, o que é necessário para que a bateria de testes seja reproduzível.

#### Casos de teste desabilitados

Diante da necessidade de desabilitar casos de teste é importante ter o seguinte em mente:

- **Evite utilizar** `@Disabled`[^junit5-disabled-javadoc] **em casos ou suítes de teste**
- Utilize tags[^junit5-user-guide] para apontar testes que não devem ser executados durante a construção através de Maven

> O _parent_ `br.gov.capes:arquitetura-backend-spring-boot` já tem configurado como grupo padrão de exclusão `non-maven`. Suítes ou casos de testes marcados como desse grupo serão ignorados no ciclo de construção[^maven-lifecycles].

##### Dasabilitar testes via tag[^obs-tags-junit-maven]

Para utilizar a tag `non-maven`[^obs-archetype] para remover casos da bateria de testes executada pelo Maven:

- Adicionar a anotação `@Tag("non-maven")`[^junit5-tag-javadoc] nos casos ou suítes de testes

**Desabilitar toda a suíte de testes**
```java
@Tag("non-maven")
class TestSimples {

  @Test
  void test() {
    fail("A suíte de testes desse caso só deve ser executada via...");
  }

}
```

**Desabilitar casos de testes específicos**
```java
class TestSimples {

  @Tag("non-maven")
  @Test
  void test() {
    assertTrue(false, "Esse caso de testes só dever ser executado via...");
  }

}
```

## JMeter

**Apache JMeter**[^apache-jmeter] é uma ferramenta para execução de cenários de testes. Utiliza uma abordagem completamente diferente de JUnit. JMeter é utilizado para testes funcionais _end-to-end_ e mensuração de performance. É uma ferramenta propícia para testes de perfomance, carga, stress.

---

### Maven

Recomendamos a integração do JMeter ao ciclo de construção do Maven[^jmeter-maven-plugin], para que esse tipo de bateria de testes possa ser incluido/uitilizado em Pipeline ou executado de maneira direta.

Recomendamos isolar as configurações de JMeter num **profile** para que execução desse tipo de teste seja deliberado e focado.

```xml
<profile>
  <id>jmeter</id>
  <properties>
    <jmeter-maven-plugin.version>3.4.0</jmeter-maven-plugin.version>
    <apache-jmeter.version>5.4.1</apache-jmeter.version>
    <jmeter-tests-dir>${project.build.directory}/jmeter/</jmeter-tests-dir>
  </properties>
  <build>
    <testResources>
      <testResource>
        <directory>${project.basedir}/src/test/resources</directory>
        <filtering>true</filtering>
        <targetPath>${jmeter-tests-dir}</targetPath>
        <includes>
          <include>*.jmx</include>
          <include>*.csv</include>
        </includes>
      </testResource>
    </testResources>
    <pluginManagement>
      <plugins>
        <plugin>
          <groupId>com.lazerycode.jmeter</groupId>
          <artifactId>jmeter-maven-plugin</artifactId>
          <version>${jmeter-maven-plugin.version}</version>
          <configuration>
            <jmeterVersion>${apache-jmeter.version}</jmeterVersion>
            <testFilesDirectory>${jmeter-tests-dir}</testFilesDirectory>
          </configuration>
          <executions>
            <execution>
              <id>configuration</id>
              <goals>
                <goal>configure</goal>
              </goals>
            </execution>
            <execution>
              <id>jmeter-check-results</id>
              <goals>
                <goal>results</goal>
              </goals>
            </execution>
          </executions>
        </plugin>
      </plugins>
    </pluginManagement>
    <plugins>
      <plugin>
        <groupId>com.lazerycode.jmeter</groupId>
        <artifactId>jmeter-maven-plugin</artifactId>
        <executions>
          <execution>
            <id>executar-jmeter</id>
            <phase>integration-test</phase>
            <goals>
              <goal>jmeter</goal>
            </goals>
            <configuration>
              <overrideRootLogLevel>debug</overrideRootLogLevel>
              <generateReports>true</generateReports>
            </configuration>
          </execution>
        </executions>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <configuration>
          <skipExec>true</skipExec>
        </configuration>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-failsafe-plugin</artifactId>
        <configuration>
          <skipExec>true</skipExec>
        </configuration>
      </plugin>
    </plugins>
  </build>
</profile>
```

- Desligamos a execução de testes de unidade (**surefire**) e de integração (**failsafe**);
- Fazemos uma filtragem dos arquivos `*.jmx` e `*.csv` para substituir variáveis que podem ser configuradas via Maven
  - Inclusive especificamente por **profile**;
- Por alguma razão estranha dessa versão do plugin (`com.lazerycode.jmeter:jmeter-maven-plugin:3.4.0`), o _goal_ precisa estar numa `<execution>` com o valor de `<id>` sendo **exatamente** `configuration`
- Para executar é preciso ativar o _profile_ na execução: `mvn -Pjmeter clean verify`

> Uma forma de segregar/selecionar diferentes cenários de testes ("carga", "stress", "performance de consulta", "performance de criação") é fazer as configurações fundamentais num _profile_ `jmeter` e criar outros outros _profiles_ para com o plugin executando o _goal_ `jmeter` configurada com a tag `<testFilesIncluded>`:
```xml
<profile>
  <id>performance</id>
  <build>
    <plugins>
      <plugin>
        <groupId>com.lazerycode.jmeter</groupId>
        <artifactId>jmeter-maven-plugin</artifactId>
        <executions>
          <execution>
            <id>executar-jmeter</id>
            <goals>
              <goal>jmeter</goal>
            </goals>
            <configuration>
              <overrideRootLogLevel>debug</overrideRootLogLevel>
              <generateReports>true</generateReports>
              <testFilesIncluded>
                <testFilesIncluded>performance-consulta-simples.jmx</testFilesIncluded>
                <testFilesIncluded>performance-consulta-detalhada.jmx</testFilesIncluded>
                <testFilesIncluded>performance-inclusao.jmx</testFilesIncluded>
                <testFilesIncluded>performance-inclusao-em-lote.jmx</testFilesIncluded>
              </testFilesIncluded>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</profile>
```
> Com isso podemos rodar essa bateria de testes de performance com: `mvn -Pjmeter,performance clean verify`

#### Como executar o JMeter GUI através do Maven

Também podemos abrir a interface gráfica do JMeter para desenvolver e ajustar os cenários de teste (arquivos `*.jmx`):
`mvn -Pjmeter jmeter:configure jmeter:gui -DguiTestFile=src/test/resources/performance.jmx`

- O parâmetro `-DguiTestFile=` pode ser utilizado para já abrir um determinado arquivo

---

Como configurar e utilizar o JMeter foge ao escopo dessa documentação, mas existe [uma vasta documentação pra isso](https://jmeter.apache.org/usermanual/index.html).

# Notas e Referências

[^obs1]: Mais do que uma mera opinião, em geral a CGS tem estabelecido em contrato com seus fornecedores esse tipo de exigência
[^obs-versionamento]: Em caso de inconsistência entre as versões apontas aqui, a implementação tem precedência. Favor informar a arquitetura nesses casos: lista.arquitetura@capes.gov.br
[^testes-aninhados-junit5]: Já faz parte da distribuição de JUnit 5 um mecanismo de aninhamento de classes de teste
[^obs-archetype]: Esse comportamento já está configurado no `pom.xml` que é gerado pelo arquétipo
[^junit5-user-guide]: Veja a documentação de JUnit 5: https://junit.org/junit5/docs/current/user-guide/
[^obs-tags-junit-maven]: Testes excluídos através de tags nem são contados como desabilitados
[^junit5-tag-javadoc]: `org.junit.jupiter.api.Tag`: https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/Tag.html
[^junit5-disabled-javadoc]: `org.junit.jupiter.api.Disabled`: https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/Disabled.html
[^traducacao-goals]: "Alvos"
[^maven-profiles]: Maven Profiles: https://maven.apache.org/guides/introduction/introduction-to-profiles.html
[^e2e-test-cosmic]: Essa é uma denominação que tipifica testes: https://www.cosmicpython.com/book/chapter_03_abstractions.html#_implementing_our_chosen_abstractions
[^junit5-nested-javadoc]:`org.junit.jupiter.api.Nested`: https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/Nested.html
[^mockito]: https://site.mockito.org/
[^junit5-ExtendWith-javadoc]: `org.junit.jupiter.api.extension.ExtendWith`: https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/extension/ExtendWith.html
[^apache-jmeter]: Apache JMeter: https://jmeter.apache.org/
[^jmeter-maven-plugin]: JMeter Maven Plugin: https://github.com/jmeter-maven-plugin/jmeter-maven-plugin
[^obs-versao-no-archetype]: A versão já está configurado no `pom.xml` que é gerado pelo arquétipo e a tag `<version>` e a tag de propriedade podem ser suprimidas nesse caso.
[^maven-lifecycles]: O Maven tem 3 ciclos padrões `clean` (limpeza), `build` (**construção**), `site` (web site do projeto) - https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html
