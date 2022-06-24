**Pilha Java para Arquitetura de Referência - versão 2.x**

# Leituras Recomendadas

- [SkyWalking OAP server and UI](https://skywalking.apache.org/docs/main/v9.1.0/readme/)
- [SkyWalking Java Agent](https://skywalking.apache.org/docs/skywalking-java/v8.10.0/readme/)

# Introdução

_Application Performance Management_ (APM) é a ideia de monitorar e gerenciar a performance e disponibilidade de aplicações. Especificamente para a CAPES selecionamos como ferramenta o Apache SkyWalking[^SkyWalking]. Por ser uma ferramenta FOSS[^foss], vimos como uma solução mais em conta para a instituição, mas levando em conta todo o respaldo que os projetos da Fundação Apache trazem.

Há duas implantações do SkyWalking:

- Uma para produção: skywalking.capes.gov.br
- Uma para DHT: skywalking.hom.capes.gov.br

Esses servidores agregam os dados enviados pelas aplicações. Há uma diversidade de opções de como coletar os dados para serem agregados no Apache SkyWalking, mas a pilha optou por utilizar o agente Java da solução[^SW-java-agent].

> #### ATENÇÃO
> A integração com o APM é uma das integrações que é _opt out_ nessa versão, **então é preciso configurar para _NÃO_ utilizar**; por padrão as aplicações baseadas na pilha vão tentar integrar o que pode gerar erros de funcionamento caso as configurações necessárias não sejam informadas.

# Dependências e Versionamentos

- SkyWalking Java Agent: `8.1.0`
- Imagem java 17:
  - `registry.capes.gov.br/base/java-17`: `1.4.0`
  - `registry.capes.gov.br/base/java-17-slim`: `1.4.0`
- chart-capes-aplic: `0.6.4`
- `br.gov.capes:arquitetura-backend-spring-boot`[^parent-vers]
  - `br.gov.capes.cgs.narq:apm-skywalking`

> Destacamos que essa integração precisa de outros elementos além da própria pilha, no caso versões mínimas da imagem base para aplicações Java e versões mínimas do chart de configuração de aplicações da CGS.

# Tutoriais

## Com _parent_ padrão

### Maven

No `pom.xml` deve haver:

```xml
<parent>
    <groupId>br.gov.capes</groupId>
    <artifactId>arquitetura-backend-spring-boot</artifactId>
    <version>2.0.0</version>
    <relativePath />
  </parent>
```

Na tag `version` deve estar o **valor** da versão da pilha (não pode ser definido como uma propriedade, estamos apenas exemplificando acima).

Caso a aplicação esteja utilizando `br.gov.capes:arquitetura-backend-spring-boot` como _parent_ (como é o padrão para aplicações criadas através do [arquétipo](./ARQUETIPO.md)), só é preciso definir as variáveis de ambiente.


### Helm Chart - chart-capes-aplic

Nos respectivos arquivos de _values_ é preciso definir as variáveis de ambiente:

- `SW_AGENT_COLLECTOR_BACKEND_SERVICES`: Indicar `hostname:porta`. O valor padrão é `skywalking.hom.capes.gov.br:11800`, logo **é necessário** informar o de produção no arquivo `values-prod.yaml`
- `SW_AGENT_GROUP`: indicamos que o grupo seja a denominação do "sistema". É comum que um "sistema" seja composto por várias aplicações, como uma GUI (web), um BFF, e uma multitude de MSS (serviços). Caso não seja indicado, vai ser preenchido com o nome do namespace onde a aplicação está sendo implantada
- `APP_NAME`: A denominação da aplicação

> A variável `SW_AGENT_NAME` tem a precedência respeitada, se ela estiver definida será ela que será utilizada. É uma forma de controlar mais diretamente o nome do serviço sendo observado

- `SW_AGENT_NAMESPACE`: a imagem Java base, caso essa variável não esteja presente o `name` do _namespace_ onde está sendo implantado (através da variável `K8S_NAMESPACE`[^k8s-namespace])

**exemplo**
```yaml
# ...
app:
  deployment:
    containers:
      image: sigla-aplicacao/sigla-aplicacao-backend
      imagePullPolicy: Always
      environments:
      - name: APP_NAME
        value: 'sigla-aplicacao-backend'
      - name: SW_AGENT_GROUP
        value: 'sigla-aplicacao'
      - name: SW_AGENT_COLLECTOR_BACKEND_SERVICES
        value: "skywalking.capes.gov.br:11800"
```

O exemplo acima demonstra uma configuração de produção. As aplicações em DHT podem usar o servidor `skywalking.hom.capes.gov.br`. A interface web da aplicação está disponível na porta `8080` ([http://skywalking.hom.capes.gov.br:8080](http://skywalking.hom.capes.gov.br:8080)).
As aplicações se conectam ao SkyWalking pela porta `11800`.

#### Instalação alternativa

Caso a imagem da aplicação não seja a imagem base para Java, ou por alguma razão tenha sido feita uma instalação alternativa do agente, é possível definir a variável de ambiente `SW_AGENT_JAVA_PATH`, apontando para o jar do agente. Essa variável tem precedência no nosso script de inicialização para apontar a locallização do agente instalado. _Essa mudança pode impactar que outro arquivo de configuração vai ser utilizado, um que provavelmente não faz uso de nossas configurações personalizadas e variáveis extras_.[^instalacoes-alternativas]

### Dockerfile

Esperamos que o Dockerfile rode a aplicação como um jar executável

```Dockerfile
ENTRYPOINT ["./app.jar"]
```
A diretiva acima supõe que os padrões estejam sendo seguidos, modo que a o `WORKDIR` da imagem é onde o jar executável da aplicação está e ele foi nomeado como `app.jar`.

## Sem o _parent_ padrão

Caso não seja utilizado `br.gov.capes:arquitetura-backend-spring-boot` como _parent_ do projeto, o script de inicialização do jar executável não é a versão modificada que ativa o agente e calcula alguns valores default. _**A recomendação é utilizar o padrão com o jar executável**_, mas se por alguma razão isso não for possível ou adequado, fica sob responsabilidade do time de desenvolvimento adotar uma estratégia equivalente, como:

- Adicionar `-javaagent:/opt/apache/skywalking/agents/java/skywalking-agent.jar` na variável `JAVA_OPTS`;
- Alterar a configuração `embeddedLaunchScript` do plugin `org.springframework.boot:spring-boot-maven-plugin`, para um script que façam as manipulações necessárias
- Adicionar a parametrização do Dockerfile

As variáveis de ambiente para configuração do Agente Java[^SW-java-agent-configs] provavelmente tem de ser defenidas ou calculadas adequadamente.
> As variáveis `SW_AGENT_GROUP` e `SW_GROUPED_AGENT_NAME` não fazem parte do conjunto padrão do Apache SkyWalking, foram definidas e são utilizadas por nossos scripts e configurações personalizadas.

---

## Não integrar com o APM

Nos casos onde não seja adequado integrar com o APM provavelmente a forma mais simples é remover o arquivo `/opt/apache/skywalking/agents/java/skywalking-agent.jar` da imagem. _Caso esse arquivo não exista **e a variável `SW_AGENT_JAVA_PATH` não esteja definida**[^SW-agentpath-unset], o script de inicialização que criamos não roda a JVM com o agente atrelado_. Nos casos onde a inicialização da JVM não esteja sendo feito pelo script que criamos (como é o caso de não usar `br.gov.capes:arquitetura-backend-spring-boot` como _parent_, ou definir a propriedade maven `launch-script.path`, ou a configuração `embeddedLaunchScript` do plugin `org.springframework.boot:spring-boot-maven-plugin`) o controle de qual script será utilizado para inicializar a aplicação é do time e pode simplesmente não implementar a parametrização do Agente Java do Apache SkyWalking para a execução da JVM.

---

# Notas e Referências

[^parent-vers]: Qualquer _snapshot_ ou _release_ que já contenha o módulo `br.gov.capes.cgs.narq:apm-skywalking`
[^k8s-namespace]: Essa é uma variável automática que foi defenida no chart a partir da versão `0.6.4`
[^SW-java-agent-configs]: Table of Agent Configuration Properties -
 https://skywalking.apache.org/docs/skywalking-java/v8.10.0/en/setup/service-agent/java-agent/configurations/
[^SkyWalking]: Apache SkyWalking - https://skywalking.apache.org/
[^foss]: Free and Open-Source Software
[^SW-java-agent]: SkyWalking Java Agent - https://skywalking.apache.org/docs/skywalking-java/v8.10.0/readme/
[^SW-agentpath-unset]: Ela normalmente não está
[^instalacoes-alternativas]: Instalações alternativas são responsabilidade do time de desenvolvimento da aplicação. O time fica responsável por todas as variáveis e detalhes de integração da aplicação com o agente, além dos corretos apontamentos para o APM adequado.
