**Pilha Java para Arquitetura de Referência - versão 2.x**

# Leituras Recomendadas

- [Spring Boot Reference Documentation - Working with SQL Databases](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.sql)
- [Hibernate ORM 5.4.32.Final User Guide](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html)[^hibernate-jp-guide]

# Introdução

Para persistir dados as aplicações devem integrar com algum recurso/serviço[^persist-on-fs]. A persistência de dados preferida para aplicações desenvolvidas pela CGS é em base base relacional ("SQL"), especificamente base de dados no SGBD Oracle.

Por isso ser lugar extremamente comum a pilha Java adere a convenção padrão do Spring Boot para integração com base de dados SQL e a extende para facilitar alguns casos que por ventura venham a ocorrer.

> ## XA
> Apesar de historicamente a CAPES (especificamente a CGS da DTI) ter adotado XA por padrão como um facilitador para garantir a consistência de dados em múltiplos bancos, encaramos problemas com o SGBD Oracle o que fez essa versão da pilha recuar da utilização de XA. A priori fica a cargo da implementação da aplicação tomar as devidas medidas para lidar com a integridade quando estiver manipulando mais de uma base de dados. Cada conexão com banco de dados tem seu próprio tratamento transacional, não havendo garantia ACID[^acid] para uma _request_ que manipule mais de uma base de dados.
> ## JTA
> Por conta disso as aplicações baseadas na pilha não tem suporte XA, o que faz com que as operações JDBC e JMS contidas numa mesma _request_ rodem em escopos transacionais distintos sem garantias ACID, sendo responsabilidade da implementação da aplicação lidar com erros e tomar as medidas compensatórias necessárias.
>
> ---
> Casos específicos podem ser conversados com o Núcleo de Arquitetura (NARQ) da CGS, mas a implementação e configuração de XA e JTA não será adicionada a pilha.
>

# Dependências e Versionamentos

- Especificação ORM: JPA (`2.2`)
  - Implementação ORM: Hibernate (`5.4.32.Final`)
- JDBC: `4.3`
  - Drivers
    - Oracle: `com.oracle.database.jdbc:ojdbc11:21.3.0.0`
    - Postgres: `org.postgresql:postgresql:42.2.23`
  - Conection Pool: HikariCP (`4.0.3`)

# Tutoriais

Essa secção vai conter as informações de como configurar e implementar a utilização de persitência de dados SQL:

- Configurações do Maven
- Configurações do Spring Boot
- Implementação em Java

## Maven

Adicionar no `pom.xml` a dependência `br.gov.capes.cgs.narq:persistencia-starter` na versão adequada:

```xml
<dependency>
  <groupId>br.gov.capes.cgs.narq</groupId>
  <artifactId>persistencia-starter</artifactId>
  <version>${persistencia-starter.version}</version>
</dependency>
```

A propriedade `${persistencia-starter.version}` deve ser definida criando a tag `<persistencia-starter.version>` na tag `<properties>` do `pom.xml`. O valor dever ser o valor da versão da pilha[^obs-versao-no-archetype].

## Spring Boot

### Aplicação "Mono" Banco

Aplicações que fazem apenas uma conexão com base de dados SQL podem utilizar o padrão de configuração do Spring Boot `spring.datasource.*` no arquivo `application.yml`.

Exemplo:
```yaml
spring:
   datasource:
      url: jdbc:oracle:thin:@oracledhtsrv01.hom.capes.gov.br:1521/dsnv_dr
      username: usuario
      password: senha
      driver-class-name: oracle.jdbc.driver.OracleDriver
```

As configurações de JPA (`spring.jpa.*`, `spring.jpa.properties.*` e `spring.jpa.hibernate.*`) podem ser utilizadas.

Exemplo:
```yaml
spring:
  jpa:
    show-sql: true
    properties:
      hibernate:
        xml_mapping_enabled: false
```

---
> ### Oracle
> Para que a aplicação conecte ao Oracle é necessário a dependência `com.oracle.database.jdbc:ojdbc11`. A versão preferida já está configurada para aplicações que usem o _parent_ `br.gov.capes:arquitetura-backend-spring-boot`, como é o caso das aplicações criadas pelo arquétipo Maven nem é preciso selecionar a dependência, ela já é dependência transitiva obrigatória.
> ### Postgres
> Para que a aplicação conecte ao Postgres é necessário adicionar a dependência `org.postgresql:postgresql` ao `pom.xml`. A versão preferida já está configurada para aplicações que usem o _parent_ `br.gov.capes:arquitetura-backend-spring-boot`, como é o caso das aplicações criadas pelo arquétipo Maven, adicionar a dependência:
> ```xml
> <dependency>
>   <groupId>org.postgresql</groupId>
>   <artifactId>postgresql</artifactId>
> </dependency>
> ```

---

### Aplicação Multi Banco

Caso seja imprescindível que a aplicação utilize mais de um banco de dados relacional[^mais-de-1-banco] é possível utilizar a extensão que criamos. A extensão é configurada através da propriedade raiz `databases`.

A propriedade hierarquicamente abaixo de `databases` é o "nome" que se quer dar aquele banco. Depois do nome é possível estabelecer uma mesma árvore de `datasource.*` e `jpa.*` (conforme em `spring.datasource.*` e `spring.jpa.*`). As configurações em `databases.<nome>.datasource` seriam as mesmas que em `spring.datasource` e em `databases.<nome>.jpa` as mesmas que em `spring.jpa`[^spring-boot-data-properties].

> #### Não misture `databases.<nome>.*` e a configuração básica do Spring
> Não coloque no mesmo arquivo de configuração `databases.<nome>.*` e `spring.datasource.*` (nem `spring.jpa.*`). Essas configurações não se misturam bem, alguns dos _beans_ podem recuar e outros não, deixando o conjunto final num estado problemático e inutilizável. Quando usar **apenas um** banco utilize a configuração padrão, nos casos **excepcionais** onde mais de um banco precise ser utilizado `databases.<nome>.*` pode ser utilizado.

#### Exemplos

**application.yml**

```yaml
databases:
  oracle:
    datasource:
      url: jdbc:oracle:thin:@oracledhtsrv01.hom.capes.gov.br:1521/dsnv_dr
      username: usuaria
      password: senha
      driver-class-name: oracle.jdbc.driver.OracleDriver
    jpa:
      packages-to-scan: br.gov.capes.corporativo
      properties:
        hibernate:
          xml_mapping_enabled: false
  #segundo banco
  postgres:
    datasource:
      url: jdbc:postgresql://postgresdhtsrv01.hom.capes.gov.br:5432/Dsnv
      username: usr_narq
      password: qrWAN_12nr_RsUu
      driver-class-name: org.postgresql.Driver
    jpa:
      packages-to-scan: br.gov.capes.legado
      properties:
        hibernate.xml_mapping_enabled: false
```

**application.properties**
```properties
databases.oracle.datasource.url=jdbc:oracle:thin:@oracledhtsrv01.hom.capes.gov.br:1521/dsnv_dr
databases.oracle.datasource.username=usuaria
databases.oracle.datasource.password=senha
databases.oracle.datasources.driver-class-name=oracle.jdbc.driver.OracleDriver

databases.oracle.jpa.packages-to-scan=br.gov.capes.corporativo
databases.oracle.jpa.properties.hibernate.xml_mapping_enabled=false

#segundo banco
databases.postgres.datasource.url=jdbc:postgresql://postgresdhtsrv01.hom.capes.gov.br:5432/Dsnv
databases.postgres.datasource.username=usuaria
databases.postgres.datasource.password=senha
databases.postgres.datasources.driver-class-name=org.postgresql.Driver

databases.postgres.jpa.packages-to-scan=br.gov.capes.legado
databases.postgres.jpa.properties.hibernate.xml_mapping_enabled=false
```

O `<nome>` utilizado abaixo/depois de `databases` será o prefixo para a definição de _alias_ para os beans criados:

- `<nome>`: Datasource (não tem sufixo). Ex.: `oracle`
- `<nome>-em`: Entity Manager. Ex.: `pessoas-em`
  ```java
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.beans.factory.annotation.Qualifier;
  //...

  @Autowired
  @Qualifier("pessoas-em")
  private EntityManager em;
  ```
- `<nome>-tm`: Transaction Manager. Ex.: `oracle-tm`
  ```java
  @Transactional(transactionManager = "postgres-tm")
  public void limpar(Object chave) {
    //...
  }
  ```


### Java

Recomendamos as seguintes opções para implementação de consultas a base de dados:

1. Spring Data Specification
2. JPA Criteria
3. JPQL
4. _Native Queries_

> #### :warning: :no_entry_sign: Atenção
> Desencorajamos a definição de _queries_ derivadas pelos nomes de métodos, assim como herdar de `org.springframework.data.repository.Repository` (e outras interfaces e classes como `org.springframework.data.repository.CrudRepository`, `org.springframework.data.jpa.repository.JpaRepository`, `org.springframework.data.jpa.repository.support.SimpleJpaRepository`). Também removemos `br.gov.capes.servicos.utils.persistencia.Repositorio` e `br.gov.capes.servicos.utils.persistencia.SpringJPADAO` que havia na versão `1.x` da pilha. Incentivamos que os Repositórios sejam construídos utilizando o padrão de _Specification_ de DDD[^ddd] e tenham uma interface reduzida e específica aos objetos que manipula.
>
> **Também recomendamos _muita_ cautela na utilização de herança**[^comp-over-inher]. Criar sua própria implementação de "Repositorio" que faça as operações de CRUD e fazer todos os repositórios herdarem dela sem nenhum critéiro é uma prática ruim. Nem todo _Repository_ tem as mesmas lógicas de  consulta (até o `findById` pode não ser relevante para algum), de ordenação, ou operações de escrita (ex.: alguns deletam, outros não; alguns deletam logicamente, outros fisicamente; etc.). **Por mais mínimo que seja o conjunto de métodos/operações da interface do "_Reporitory_ base", se pelo menos um dos Repositórios implementados não utiliza qualquer um dos métodos herdados, esse "_Repository_ base" está inchado**.

---
> #### Observação
> Não vamos nos estender em relação a utilização de JPA. O objetivo dessa documentação **não** é ensinar JPA e sim priorizar as formas de utilização, listar e expor o que é desencorajado. Em suma consolidar e sintetizar as práticas que esperamos ver implementadas apoiadas nas escolhas e opiniões para a pilha Java para a Arquitetura de Referência.
---

### 1. Spring Data Specification

_Specification_ é um padrão de DDD. Como a pilha é fortemente baseada em Spring, não vemos problema em utilizar a implementação do _framework_: `org.springframework.data.jpa.domain.Specification`.

Para incentivar o uso de composição a pilha dispõe de algumas classes utilitárias que podem ser utilizadas para implementar os casos mais comuns de utilização de _Specification_:

- **`br.gov.capes.cgs.narq.jpa.Listagem`** para listar um conjunto de entidades com base numa _Specification_;
- **`br.gov.capes.cgs.narq.jpa.Selecao`** para selecionar uma única entidade com base numa _Specification_.

#### Exemplos

Com uma implementação mínima da entidade `Pessoa`:

```java
package br.gov.capes.pessoas;

import java.time.LocalDateTime;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

import lombok.AccessLevel;
import lombok.Getter;
import lombok.ToString;

@Entity
@ToString
@Table(schema = "CORPORATIVO", name = "PESSOA")
public class Pessoa {

  //Normalmente esse construtor seria privado, mas para o EntityManager#getReference funcionar
  //precisa que será pelo menos default
  Pessoa() {
    super();
  }

  @Id
  @Column(name = "ID_PESSOA")
  @Getter(value = AccessLevel.PACKAGE) //Visibilidade para operações do repositório Pessoas
  private Long id;

  @Getter
  @Column(name = "NM_PESSOA")
  private String nome;

  @Column(name = "DS_IDENTIFICADOR_REGISTRADO")
  private String identificador;
}
```

O repositório `Pessoas` poderia ser implementado com uma listagem por nome e uma seleção pelo identificador registrado:

```java
package br.gov.capes.pessoas;

import java.util.List;

import javax.persistence.EntityManager;
import javax.persistence.criteria.CriteriaBuilder;
import javax.persistence.criteria.CriteriaQuery;
import javax.persistence.criteria.Root;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.stereotype.Repository;

import br.gov.capes.cgs.narq.jpa.Listagem;

@Repository
public class Pessoas {

  private static Logger logger = LoggerFactory.getLogger(Pessoas.class);

  private EntityManager em;
  private Listagem<Pessoa> listagem;

  @Autowired
  public Pessoas(@Qualifier("oracle-em") EntityManager em) {
    logger.debug("new Pessoas(em:EntityManager='{}')", em);
    this.listagem = new Listagem<Pessoa>(Pessoa.class);
    this.em = em;
  }

  public List<Pessoa> buscar(Specification<Pessoa> spec) {
    return this.listagem.com(this.em).execute(spec);
  }

  public static Specification<Pessoa> peloIdetificadorRegistrado(String chave) {
    return (Root<Pessoa> root, CriteriaQuery<?> query, CriteriaBuilder cb) -> {
      return cb.equal(root.get("identificador"), chave);
    };
  }

  public static Specification<Pessoa> peloNome(String chave) {
    return (Root<Pessoa> root, CriteriaQuery<?> query, CriteriaBuilder cb) -> {
      return cb.like(root.get("nome"), chave);
    };
  }

}
```

> ##### APENAS UMA invocação de `execute(Specification<T> scpe)` por instância
>  **`br.gov.capes.cgs.narq.jpa.Listagem`** e **`br.gov.capes.cgs.narq.jpa.Selecao`** são instâncias de uso único. O método `execute(Specification<T> scpe)` só pode ser invocado quando a instância está no estado **`PRONTA`**.

### 2. JPA Criteria

Algumas vezes não é viável ou legível implementar determinadas consultas como _Specification_, nesse caso a primeira alternativa recomendada é utilizar mais diretamente a API de JPA Criteria[^oracle-jpa-criteria] [^baeldung-jpa-criteria].

#### Exemplo: Qual o tamanho da família "Silva"?

Vamos exemplificar fazendo uma consulta contra a entidade `Pessoa` (mapeada anteriormente). Qual o tamanho da família "Silva"? Ou: quantas pessoas na base tem o sobrenome Silva?
Ter o sobrenome Silva significa ter ele no meio do nome completo ('PRENOME **SILVA** OUTROS_SOBRENOMES') ou terminar em Silva ('PRENOME OUTROS_SOBRENOMES **SILVA**'). Traduzindo isso para Java utilizando a API de Criteria:

```java
import javax.persistence.EntityManager;
import javax.persistence.Query;
import javax.persistence.criteria.CriteriaBuilder;
import javax.persistence.criteria.CriteriaQuery;
import javax.persistence.criteria.Predicate;
import javax.persistence.criteria.Root;

//...

CriteriaBuilder cb = this.em.getCriteriaBuilder(); //this.em é a referência a instância de EntityManager
CriteriaQuery<Long> query = cb.createQuery(Long.class);
Root<Pessoa> root = query.from(Pessoa.class);

Predicate temSilva = cb.like(cb.upper(root.get("nome")), "% SILVA %");
Predicate terminaEmSilva = cb.like(cb.upper(root.get("nome")), "% SILVA");

query = query.select(cb.count(root));
query = query.where(cb.or(temSilva, terminaEmSilva));

Long total = this.em.createQuery(query).getSingleResult();
```

> **Criteria API é uma opção viável e cabível para escrita de dados (delete[^jpa-criteria-delete], update[^jpa-criteria-update]).**
> Em geral as manipulações normais de entidades[^jpa-em-persist] [^jpa-em-merge] [^jpa-em-remove] pelo `EntityManager` devem ser suficientes.

### 3. JPQL

Uma alternativa a API de Criteria é a JPQL. Reescrevendo a consulta de "Qual o tamanho da família Silva?" com JPQL temos o seguinte:

```java
import java.io.File;
import java.nio.file.Files;

import javax.persistence.EntityManager;
import javax.persistence.Query;

import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;

//...

Resource resource = new ClassPathResource("quantosSilvas.jpql");
File file = resource.getFile();
String jpql = new String(Files.readAllBytes(file.toPath()));

Query query = this.em.createQuery(jpql, Long.class);
Long total = (Long) query.getSingleResult();
```

Conteúdo de `quantosSilvas.jpql`
```sql
select count(*)
from Pessoa p
where upper(p.nome) like '% SILVA %'
      or upper(p.nome) like '% SILVA'
```

> JPQL é obviamente menos orientado a objeto, envolvendo por vezes manipulação de _strings_. O que podemos recomendar é evitar casos de manipulação de _strings_ de JPQL. Esse tipo de situação onde envolve a manipulação condicional para a geração da consulta é um caso onde se deveria utilizar a API de Criteria. Consultas fixas podem ser externalizadas em arquivos `.jpql` e mantidos como "_resources_" da aplicação (como demonstrado no exemplo). Essa separação se dá para evitar misturar mais de um linguagem num mesmo arquivo. Arquivos `.java` deveriam ter código Java e evitar qualquer outra linguagem.

> #### JPQL não é HQL: JPA vs Hibernate
> Apesar da implementação de JPA selecionada ser o Hibernate, é importante que não sejam utilizados recursos e capacidades do Hibernate que não sejam cobertos pela especificação JPA, uma dessas coisas é HQL. Apesar de todo JPQL ser um HQL válido, nem todo HQL é um JPQL válido[^hql-vs-jpql]. Utilize a especificação, não a implementação ("desenvolva orientado a interface não a implementação").

### 4. _Native Queries_

Em último caso podem ser utilizadas "Consultas Nativas" para fazer uso de capacidade específicas de SQL (seja do padrão ou extensões do banco em questão).

Vamos exemplificar consultando os "10 mais novos Silvas" da nossa base de dados:

```java
import java.io.File;
import java.nio.file.Files;
import java.util.List;

import javax.persistence.EntityManager;

import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;

//...

Resource resource = new ClassPathResource("novosSilvas.sql");
File file = resource.getFile();
String sql = new String(Files.readAllBytes(file.toPath()));

List<?> dezNovosSilvas = this.em.createNativeQuery(sql).setMaxResults(10).getResultList();
```

> **Novamente** recomendamos _evitar a todo custo manipular strings_, prefira deixar consultas fixas em arquivos `.sql` e carregar o conteúdo deles.

Conteúdo de `novosSilvas.sql`:

```sql
select
  p.ID_PESSOA as ID,
  p.NM_PESSOA as NOME,
  tir.DS_TIPO_IDENTIFICADOR as TIPO_DOCUMENTO,
  id.DS_IDENTIFICADOR_REGISTRADO as NUMERO_DOCUMENTO,
  pf.DT_NASCIMENTO as DATA_NASCIMENTO
from CORPORATIVO.PESSOA p
  inner join CORPORATIVO.IDENTIFICADOR_REGISTRADO id
    on id.ID_PESSOA = p.ID_PESSOA
      and id.ID_TIPO_IDENTIFICADOR = p.ID_TIPO_IDENTIFICADOR
  inner join CORPORATIVO.TIPO_IDENTIFICADOR_REGISTRADO tir
    on tir.ID_TIPO_IDENTIFICADOR = p.ID_TIPO_IDENTIFICADOR
  inner join CORPORATIVO.PESSOA_FISICA pf
    on pf.ID_PESSOA_FISICA = p.ID_PESSOA
where upper(p.NM_PESSOA) like '% SILVA %'
  or upper(p.NM_PESSOA) like '% SILVA'
order by pf.DT_NASCIMENTO desc, p.ID_PESSOA desc
```

O retorno padrão é um _array_ de `Object` o que não é muito prático de manipular. Podemos mapear o resultado dessa consulta para um tipo com com propriedades e comportamentos - classe `br.gov.capes.pessoas.Silva`:

```java
package br.gov.capes.pessoas;

import java.time.LocalDate;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.ToString;

@ToString
@AllArgsConstructor
public class Silva {

  private Long id;

  @Getter
  private String nome;

  @Getter
  private LocalDate dataNascimento;

}

```

Adicionando a anotação `@SqlResultSetMapping`[^SqlResultSetMapping-javadoc] em `Pessoa`:

```java
@SqlResultSetMapping(
  name = Pessoa.SILVA,
  classes = {
    @ConstructorResult(
      targetClass = Silva.class,
      columns = {
        @ColumnResult(name = "ID", type = Long.class),
        @ColumnResult(name = "NOME", type = String.class),
        @ColumnResult(name = "DATA_NASCIMENTO", type = LocalDate.class),
      }
    )
  }
)
```

Podemos rodar a consulta fazendo uso desse mapeamento:

```java
import java.io.File;
import java.nio.file.Files;
import java.util.List;

import javax.persistence.EntityManager;

import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;

//...

Resource resource = new ClassPathResource("novosSilvas.sql");
File file = resource.getFile();
String sql = new String(Files.readAllBytes(file.toPath()));

@SuppressWarnings("unchecked")
List<Silva> dezNovosSilvas = this.em.createNativeQuery(sql, Pessoa.SILVA).setMaxResults(10).getResultList();
```

---

### "_Named Queries Annotations_"

As anotações `@NamedQuery`[^namedquery-javadoc] (agregada por `@NamedQueries`[^namedqueries-javadoc]) e `@NamedNativeQuery`[^namednativequery-javadoc] (agregada por `@NamedNativeQueries`[^namednativequeries-javadoc]) exigem que a string com a consulta seja um dos atributos (`query`) da anotação. Essas anotações são pré-processadas e ficam no escopo da implementação de JPA, o que pode dar um pequeno ganho de performance. É importante notar que esse pré-processamento impacta na inicialização da aplicação e na _baseline_ da quatidade de memória da aplicação. **O mais importante é que essas consultas não sejam longas para não comprometer a legibilidade** e levar em conta que elas inevitavelmente enxertam código em outra linguagem nos arquivos `.java`.

Se anotarmos `Pessoa` com:
```java
@NamedNativeQueries({
  @NamedNativeQuery(
    name = Pessoa.QTD_PESSOAS_FISICAS,
    query = "select count(*) from CORPORATIVO.PESSOA p where p.TP_PESSOA = 'F'"
  )
})
@NamedQueries(
  @NamedQuery(
    name = Pessoa.PRIMEIRA_PESSOA,
    query = "select p from Pessoa p where p.id = 1"
  )
)
```

Podemos realizar as seguintes consultas:
```java
this.em.createNamedQuery(Pessoa.QTD_PESSOAS_FISICAS).getSingleResult()

Pessoa primeira = this.em.createNamedQuery(Pessoa.PRIMEIRA_PESSOA, Pessoa.class).getSingleResult();
```

> ### Não utilizar a anotação `@Query`[^spring-query-annotation] do Spring
> A anotação `@Query` é utilizada para definir consultas em métodos de `Repository` o que já mencionamos como desaconselhado.

---

# Questões Conhecidas

# Notas e Referências

[^acid]: Atomicity, Consistency, Isolation, Durability (Atomicidade, consistência, Isolamento, Durabilidade) - https://en.wikipedia.org/wiki/ACID
[^comp-over-inher]: Nem apenas pelo princípio de "_Composition over Inheritance_" - http://wiki.c2.com/?CompositionInsteadOfInheritance
[^persist-on-fs]: Nem que o recurso seja o sistema de arquivos de onde a aplicação roda.
[^mais-de-1-banco]: Consideramos utilização se a aplicação se conecta e realiza operações de DML ou SQL através daquela conexão.
[^oracle-jpa-criteria]: Oracle JPA Tutorial - https://docs.oracle.com/javaee/7/tutorial/persistence-criteria.htm#GJITV
[^baeldung-jpa-criteria]: Baeldung's JPA Criteria Queries - https://www.baeldung.com/hibernate-criteria-queries
[^spring-query-annotation]: `org.springframework.data.jpa.repository.Query` - https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/Query.html
[^namedquery-javadoc]: `javax.persistence.NamedQuery` - https://docs.jboss.org/hibernate/jpa/2.2/api/javax/persistence/NamedQuery.html
[^namedqueries-javadoc]: `javax.persistence.NamedQueries` - https://docs.jboss.org/hibernate/jpa/2.2/api/javax/persistence/NamedQueries.html
[^namednativequery-javadoc]: `javax.persistence.NamedNativeQuery` - https://docs.jboss.org/hibernate/jpa/2.2/api/javax/persistence/NamedNativeQuery.html
[^namednativequeries-javadoc]: `javax.persistence.NamedNativeQueries` - https://docs.jboss.org/hibernate/jpa/2.2/api/javax/persistence/NamedNativeQueries.html
[^SqlResultSetMapping-javadoc]: `javax.persistence.SqlResultSetMapping` - https://docs.jboss.org/hibernate/jpa/2.2/api/javax/persistence/SqlResultSetMapping.html
[^hql-vs-jpql]: HQL and JPQL - https://docs.jboss.org/hibernate/orm/5.5/userguide/html_single/Hibernate_User_Guide.html#hql
[^hibernate-jp-guide]: Apesar de ser um manual de Hibernate toca em muitos dos aspectos da utilização de JPA
[^obs-versao-no-archetype]: A versão já está configurado no `pom.xml` que é gerado pelo arquétipo e a tag `<version>` e a tag de propriedade podem ser suprimidas nesse caso.
[^jpa-criteria-delete]: `<T> CriteriaDelete<T> CriteriaBuilder.createCriteriaDelete​(java.lang.Class<T> targetEntity)` - https://docs.jboss.org/hibernate/jpa/2.2/api/javax/persistence/criteria/CriteriaBuilder.html#createCriteriaDelete-java.lang.Class-
[^jpa-criteria-update]: `<T> CriteriaUpdate<T> CriteriaBuilder.createCriteriaUpdate​(java.lang.Class<T> targetEntity)` - https://docs.jboss.org/hibernate/jpa/2.2/api/javax/persistence/criteria/CriteriaBuilder.html#createCriteriaUpdate-java.lang.Class-
[^jpa-em-persist]: `void EntityManager.persist​(java.lang.Object entity)` - https://docs.jboss.org/hibernate/jpa/2.2/api/javax/persistence/EntityManager.html#persist-java.lang.Object-
[^jpa-em-merge]: `<T> T EntityManager.merge​(T entity)` - https://docs.jboss.org/hibernate/jpa/2.2/api/javax/persistence/EntityManager.html#merge-T-
[^jpa-em-remove]: `void EntityManager.remove(java.lang.Object entity)`
[^ddd]: Domain-driven design - https://www.dddcommunity.org/learning-ddd/what_is_ddd/
[^crud]: Create, Read, Update, Delete - https://developer.mozilla.org/en-US/docs/Glossary/CRUD
[^spring-boot-data-properties]: Spring Boot Data Properties - https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties.data
