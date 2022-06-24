**Pilha Java para Arquitetura de Referência - versão 2.x**

# Leituras Recomendadas

- [Spring Cloud Sleuth Reference Documentation](https://docs.spring.io/spring-cloud-sleuth/docs/current/reference/html/getting-started.html)
  - [Spring Cloud Sleuth - Getting Started](https://docs.spring.io/spring-cloud-sleuth/docs/current/reference/html/getting-started.html)
  - [Using Spring Cloud Sleuth](https://docs.spring.io/spring-cloud-sleuth/docs/current/reference/html/using.html)
- [Spring Cloud Sleuth OTel](https://github.com/spring-projects-experimental/spring-cloud-sleuth-otel)

# Introdução

Os _backends_ de "Aplicações WEB" muitas vezes são "sistemas distribuídos"[^dist-sys]. Essa é a arquitetura esperada para aplicações desenvolvidas/mantidas pela CGS. Apesar de suas vantagens, sistemas distribuídos apresentam suas próprias limitações e desafios. Uma das mais destacáveis está na área de **Observabilidade**[^observability]. Um dos aspectos é como entender a cadeia de interações entre as diferentes aplicações, serviços e ferramentas que compõem a plataforma.
Exemplificando com um cenário simplista: a interface faz uma chamada ao _backend_, que tem na sua fronteira um BFF[^bff], esse BFF distribui as chamadas entre uma diversidade de serviços negociais, provavelmente agregando e simplificando os dados de modo a otimizar o que é respondido para a interface. Cada um desses serviços negociais pode fazer chamadas a outros serviços negociais, consultar ou alterar diferentes bases de dados (relacionais ou não), adicionar eventos em _message brokers_, chamar serviços como de correio eletrônico, dentre outras. Conseguir rastrear quais os elementos ativados por uma dada requisição da interface é importante para garantir o funcionamento correto da aplicação, otimizar a arquitetura do _backend_ e depurar/"debugar".
Basicamente é necessário que conforme uma requisição seja ralizada todas as interações estejam associadas com tal requisição, que é o que chamamos de "**_tracing_ distribuído**"[^distributed-tracing]. As aplicações precisam ser instrumentadas para realizar tal proeza. Um aspecto a ser levado em consideração é que essas informações tem de ser agregadas e disponibilizadas para escrutínio. A CAPES escolheu o Zipkin[^zipkin] como ferramenta de agregação e gerenciamento. Escolhemos para essa versão da pilha  Java o _Spring Cloud Sleuth_[^spring-cloud-sleuth] com OpenTelemetry[^opentelemetry].

> O OpenTelemetry[^otel-docs] é um projeto incubado[^opentelemetry-cncf] pela CNCF[^cncf], que dentre outras coisas tem a ideia de ser compatível, mas substituir soluções como o Brave[^brave] ou OpenTracing[^opentracing].

# Dependências e Versionamentos

- Spring Cloud Sleuth: `3.1.1`
  - `org.springframework.cloud:spring-cloud-sleuth-otel-autoconfigure`: `1.1.0-M5`

# Tutoriais

## Maven

Adicionar no `pom.xml` a dependência `br.gov.capes.cgs.narq:tracing-starter` na versão adequada:

```xml
<dependency>
  <groupId>br.gov.capes.cgs.narq</groupId>
  <artifactId>tracing-starter</artifactId>
  <version>${tracing-starter.version}</version>
</dependency>
```

A propriedade `${tracing-starter.version}` deve ser definida criando a tag `<tracing-starter.version>` na tag `<properties>` do `pom.xml`. O valor dever ser o valor da versão da pilha[^obs-versao-no-archetype].

## Spring Boot

Comecemos com um exemplo de configuração e depois vamos discutir a respeito de tais configurações.

```yaml
spring:
  application:
    name: app-de-exemplo
  sleuth:
    sampler.probability: 1.0
    propagation:
      type: B3
    zipkin:
      base-url: http://localhost:9411
      sender.type: web
```

- `spring.application.name`: é necessário para que as informações sejam devidamente identificadas
- `spring.sleuth.sampler.probability`: é um valor entre 0 e 1.0 que define a proporção de operações que deve ser registradas (no caso da CAPES) no Zipkin
- `spring.sleuth.propagation.type`: quais tipos de propagação devem ser utilizados. É um campo que **deve** ser informado e deve ter `B3`[^b3] para mantermos compatibilidade com aplicações de outras linguagens, ou em outras versões da pilha
- `spring.sleuth.base-url`: é o endereço onde se encontra o Zipkin que agregará as informações
  - > No exemplo acima apontamos para um Zipkin rodando localmente (`localhost`). A DTI tem uma instalação de Zipkin para cada _stage_ de DHT (Desenvolvimento, Homologação, Testes) e de Produção. O valor deve ser configurado adequadamente.
- `spring.sleuth.sender.type`: é o tipo de comunicação que deve ser utilizada para registrar os dados no Zipkin. No caso da CAPES deve ser `web`

> ### `spring.sleuth.sampler.probability`
> Essa proprieade não estava devidamente integrada com o `org.springframework.cloud:spring-cloud-sleuth-otel-autoconfigure`, por isso implementamos que caso não seja definido, será tratado como se fosse `1.0`. O valor de `1.0` implica que todas as requisições serão reportadas ao Zipkin. Isso pode não ser o comportamento ideal para a aplicação, mas é o comportamento que estabelecemos para aplicações que façam uso desse módulo da pilha.
> Um valor de `0.0` faz com que nenhum tracing seja enviado ao Zipkin. Valores intermediários (maior que `0.0`, menor que `1.0`) vão ser enviados aleatoriamente na proporção apontada. Valores negativos são tratados como `0.0` e maiores que `1.0` tratados como se fossem `1.0`.
> Caso a aplicação defina seu próprio _bean_[^spring-bean] `Sampler`[^opentelemetry-sampler], essa configuração recua e a implementação da aplicação será utilizada.

## Java

### Criando e Encerrando `Spans`

É possível implementar programaticamente um controle fino de por onde o fluxo de processamento da _request_ passa **dentro** de uma aplicação, mas em geral isso é um exagero que estamos apenas mencionando levando em conta as leituras recomendadas.

### Definindo o _bean_ de `Sampler`[^opentelemetry-sampler] na aplicação

Um outro aspecto que pode ser mencionado diz respeito a aplicações implementarem sua própria lógica de `Sampler`[^opentelemetry-sampler]. Nesses casos fica sob responsabilidade da aplicação definir a estratégia e frequência adequada de _sampling_ do tracoing.

**Exemplo**:
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

import io.opentelemetry.sdk.trace.samplers.Sampler;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public Sampler sampler() {
        return Sampler.traceIdRatioBased(0.125d);
    }

}
```

> No exemplo acima a frequência é de `0.125` independentemente do que estiver definido na proprieade `spring.sleuth.sampler.probability`.

# Questões Conhecidas

## Integração com aplicações baseadas na versão 1.x da pilha

Devido a suporte nas versões das bibliotecas utilizadas na pilha 2.x foi necessário um ajuste na versão 1.x da pilha. **A partir da versão `1.2.1` o _tracing_ integra corretamente com aplicações baseadas na pilha 2.x**. Versões anteriores a `1.2.1` são incompatíveis e por isso o _tracing_ fica segmentada o que vai de encontro ao propósito de _tracing_ distribuído.

> É recomendável que todas as aplicações na versão 1.x da pilha atualizem para a versão `1.2.1`, para garantir a devida integração, mas principalmente se há interações entre aplicações baseadas nas pilhas 1.x e 2.x.

# Notas e Referências

[^dist-sys]: _What is a Distributed System, and How Does it Work?_ - https://www.confluent.io/learn/distributed-systems/
[^observability]: "_In control theory, observability is a measure of how well internal states of a system can be inferred from knowledge of its external_" - Observability - https://en.wikipedia.org/wiki/Observability
[^bff]: Pattern: Backends For Frontends - https://samnewman.io/patterns/architectural/bff/
[^zipkin]: Zipkin - https://zipkin.io/
[^distributed-tracing]:Pattern: Distributed tracing - https://microservices.io/patterns/observability/distributed-tracing.html
[^opentelemetry]: OpenTelemetry - https://opentelemetry.io/
[^spring-cloud-sleuth]: Spring Cloud Sleuth - https://spring.io/projects/spring-cloud-sleuth
[^cncf]: Cloud Native Computing Foundation - https://www.cncf.io/
[^opentelemetry-cncf]: https://www.cncf.io/projects/opentelemetry/
[^b3]:`B3` é o formato originado pelo Zipkin - https://github.com/openzipkin/b3-propagation
[^spring-bean]: Por exemplo, utilizando a anotação `@Bean` - https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html
[^opentelemetry-sampler]: `io.opentelemetry.sdk.trace.samplers.Sampler` - https://javadoc.io/doc/io.opentelemetry/opentelemetry-sdk-trace/1.12.0/io/opentelemetry/sdk/trace/samplers/Sampler.html
[^obs-versao-no-archetype]: A versão já está configurado no `pom.xml` que é gerado pelo arquétipo e a tag `<version>` e a tag de propriedade podem ser suprimidas nesse caso.
[^otel-docs]: Documentation - https://opentelemetry.io/docs/
[^brave]: Brave - https://github.com/openzipkin/brave
[^opentracing]: OpenTracing - https://opentracing.io/
