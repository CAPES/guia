**Pilha Java para Arquitetura de Referência - versão 2.x**

# Leituras Recomendadas

- [Feature Toggles (aka Feature Flags)](https://martinfowler.com/articles/feature-toggles.html)
- [GitLab - Feature Flags](https://docs.gitlab.com/ee/operations/feature_flags.html)
- [Apêndice 1 - Estratégias de _Deploy_/Entrega](./APENDICES/FEATURE-TOGGLES1.md)

# Introdução

_Feature Toggles_ (ou _Flags_) é um mecanismo para permitir que a aplicação rode sem que funcionalidades ou comportamentos estejam efetivamente ativos. Reforçamos as **Leituras Recomendadas**, mas podemos pensar nessa técnica como uma forma seletiva de "código morto". No seu formato mais simples é meramente um trecho de código envolvido por um _if_ ou _if/else_, que vai chavear se um determinado processamento vai ocorrer.

Pode parecer irrelevante algo que se resume a um desvio de fluxo (_if/else_), mas a ideia chave é utilidade empregada: separar o _deploy_ de uma funcionalidade, de sua disponibilização. _Deploy_  é traduzido como Implantação, e é o passo onde os artefatos que compõem um determinado _software_ são colocados num determinado contexto computacional em que tal _software_ fica hábil para utilização. Depois que é implantado o _software_ (ou determinadas funcionalidades dele), pode ainda não estar disponível.

No universo de aplicações web é muito comum e na maiora das vezes desejável, que o código com uma nova funcionalidade seja implantado bem antes de ser disponibilizado para as pessoas e aplicações cliente. A decisão de quando uma determinada funcionalidade vai ser "lançada" é uma decisão negocial. O time de desenvolvimento da aplicação tem a responsabilidade de desenvolver tal funcionalidade. Se a disponibilização é totalmente atrelada a implantação, revisões de datas podem criar pressões que dificultem ou até compromentam a Entrega Contínua[^cd].
_Feature Flags_ também podem ser importantes para os times de desenvolvimento. Existem estratégias de _Feature Flag_ que permitem que apenas um percentual das clientes (pessoas ou aplicações) tenham acesso a nova funcionalidade. Há até casos que uma estratégia com uma categoria ou lista específica de clientes possa ter acesso a nova funcionalidade (como por exemplo "_Beta Testers_").

> ### Sobre Estratégias de _Feature Flags_
> - Há casos por exemplo também onde se deseja validar a corretude da nova funcionalidade, ou o desempenho dela. Há cenários onde até mesmo há utilidade de DevOps para desligar uma determinada funcionalidade que esteja onerando a aplicação/plataforma durante um pico de utilização.
> - O importante é ter em mente que _Feature Toggles/Flags_ são temporários e devem ser retiradas conforme a funcionalidade estabilize.
> - **É importante não tentar utilizar _Feature Toggles/Flags_ para fazer algo perene de controle de perfis**. Esse tipo de coisa é mais adequadamente implementado com RBAC[^rbac]. É provavelmente adequado utilizar _Feature Flags_ e RBAC combinados, e eventualmente remover a _Feature Flag_.
> - **Reforçamos as Leituras Recomendadas para considerações a respeito de _Deploy e Entrega_ relacionando com _Feature Flags_[^vide-apendice1]**.

# Dependências e Versionamentos

- GitLab: `13.5`
- Unleash Client SDK para Java:
  - `io.getunleash:unleash-client-java`: `5.1.0`

> ### GitLab
> Estamos prevendo utilizar o próprio GitLab como servidor de _command and control_ do Unleash[^unleash].

# Tutoriais

## Maven

Adicionar no `pom.xml` a dependência `br.gov.capes.cgs.narq:feature-toggle-starter` na versão adequada:

```xml
<dependency>
  <groupId>br.gov.capes.cgs.narq</groupId>
  <artifactId>feature-toggle-starter</artifactId>
  <version>${feature-toggle-starter.version}</version>
</dependency>
```

A propriedade `${feature-toggle-starter.version}` deve ser definida criando a tag `<feature-toggle-starter.version>` na tag `<properties>` do `pom.xml`. O valor dever ser o valor da versão da pilha[^obs-versao-no-archetype].

## Spring Boot

O módulo `br.gov.capes.cgs.narq:feature-toggle-starter` é configurado nas propriedades `unleash.*`. Há 2 modos (`unleash.mode`) possíveis para o módulo:
- `GITLAB`
- `UNLEASH`

### Modo `GITLAB`

É o modo padrão, já que a expectativa é que seja utilizado o GitLab como servidor de controle de _Feature Flags_.

- `unleash.endpoint`: URL da API REST do Unleash Server, no modo `GITLAB` é **API URL**[^gitlab-credentials]
- `unleash.token`: no modo `GITLAB` é **Instance ID** no GitLab[^gitlab-credentials]
- `unleash.env`: no modo `GITLAB` é **Application name** no GitLab[^gitlab-credentials]

**Exemplo**:
```yaml
unleash:
  mode: gitlab
  endpoint: https://git.capes.gov.br/api/v4/feature_flags/unleash/2
  token: SyHHPwRf3mJ3TunTYyHU
  env: development
```

**Observe que essa configuração é necessária para que o controle através de _Feature Toggles_ seja possível**.

> Não vamos discutir o modo `UNLEASH`, mas ele está implementado como uma medida de contenção caso se deseje mudar do GitLab para um Unleash externo.

## Java

Em geral a única coisa que será preciso implementar no código Java é uma checagem do estado da _Feature Flag_.

**Exemplo**:
```java
import java.net.URI;
import java.net.URISyntaxException;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

import br.gov.capes.commons.Cor;
import br.gov.capes.pessoas.Pessoa.ConsultaPessoa;
import io.getunleash.Unleash;

@Repository
public class Pessoas {

    private static Logger logger = LoggerFactory.getLogger(Pessoas.class);

    @Autowired
    private Unleash unleash;

    public Pessoa get(Long id) {
        logger.debug("@get(id:Long='{}'):Pessoa", id);

        ConsultaPessoa consulta = this.novaConsulta(id);

        if(this.unleash.isEnabled("website")) {
            this.incluirWebsiteNoResultado(id, consulta);
        }

        if(this.unleash.isEnabled("cor-preferida")) {
            this.incluirCorPreferidaNoResultado(id, consulta);
        }

        Pessoa resultado = consulta.executar();
        return resultado;
    }

    //...

}
```

### Lista de Usuárias (IDs)

Caso o módulo `br.gov.capes.cgs.narq:seguranca-starter` esteja presente o Spring _bean_[^unleash-bean] de `Unleash`[^unleash-java-obj], terá no seu contexto a ID da usuária. O valor dessa ID será o valor da _claim_ `sub` do JWT.

### Configurações Avançadas

Os seguintes _beans_ são auto-configurados pelo módulo `br.gov.capes.cgs.narq:feature-toggle-starter`:

- `UnleashContextProvider`
  - Nome: `br.gov.capes::Unleash::context-provider`
- `UnleashConfig`
  - Nome: `br.gov.capes::Unleash::config`
- `Unleash`
  - Nome: `br.gov.capes::Unleash`
  - Escopo: `request`

Qualquer um desses _beans_ deve recuar caso a aplicação defina seu próprio _bean_ daquele tipo.

> #### `UnleashSettings`[^unlesash-settings]
> A classe `br.gov.capes.cgs.narq.featuretoggle.UnleashSettings` **não** mapeia todas as propriedades definidas pelo API de SDK Client do Unleash[^unleash-java-sdk]. No caso de ser necessário, as aplicações deveriam definir os _beans_ de acordo. Aplicações que queiram implementar seus pŕoprios _beans_ com minúcias de configuração provavelmente vão precisar implementar seu próprio mapeamento de propriedades (herança pode ser uma abordagem adequada nesse caso).

# Questões Conhecidas

## Integração com GitLab

- Até o momento[^gitlab-issue-35666], não há um modo de integrar as credenciais de Feature Toggles no pipeline[^repo-pipeline] de aplicações da CGS. **A solução é adicionar os valores de credenciais[^gitlab-credentials] diretamente nos ConfigMaps no respectivo `values-*.yaml`**.

# Notas e Referências

[^cd]: ContinuousDelivery - https://martinfowler.com/bliki/ContinuousDelivery.html
[^rbac]: Segurança - [_Roles_ (Papeis), _Scopes_, Permissões e `GrantedAuthority`](./SEGURANCA.md#roles-papeis-scopes-permissões-e-grantedauthority)
[^repo-pipeline]: Gitlab Pipeline - https://git.capes.gov.br/cgs/DEVOPS/automations/gitlab-pipeline
[^obs-versao-no-archetype]: A versão já está configurado no `pom.xml` que é gerado pelo arquétipo e a tag `<version>` e a tag de propriedade podem ser suprimidas nesse caso.
[^unleash]: Unleash - https://getunleash.io/
[^gitlab-credentials]: Get access credentials - https://docs.gitlab.com/ee/operations/feature_flags.html#integrate-feature-flags-with-your-application
[^unleash-java-obj]: Unleash.java - https://github.com/Unleash/unleash-client-java/blob/main/src/main/java/io/getunleash/Unleash.java
[^unleash-bean]: O _bean_ com nome `br.gov.capes::Unleash`
[^unlesash-settings]: Classe `br.gov.capes.cgs.narq.featuretoggle.UnleashSettings`
[^unleash-java-sdk]: Java SDK - https://docs.getunleash.io/sdks/java_sdk
[^vide-apendice1]: Vide [Apêndice 1 - Estratégias de _Deploy_/Entrega](./APENDICES/FEATURE-TOGGLES1.md)
[^helm]: The package manager for Kubernetes - https://helm.sh/
[^helm-charts]: Charts - https://helm.sh/docs/topics/charts/
[^gitlab-issue-35666]: Quando a Issue 35666 do GitLab for resolvida, a expectativa é que haja esse suporte - _Provide UNLEASH_URL and UNLEASH_INSTANCE_ID as CI/CD pre-defined environment variables_ - https://gitlab.com/gitlab-org/gitlab/-/issues/35666
