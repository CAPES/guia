**Pilha Java para Arquitetura de Referência - versão 2.x**

# Leituras Recomendadas

- [CAPES Cronjob HelmChart](https://xpto.com/cgs/DEVOPS/helm/cronjob/blob/master/README.md)
- [Eventos & Mensageria entre aplicações](https://xpto.com/dti/orientacoes-gerais/guia/blob/master/arquitetura/definicoes/eventos/README.md)
- [Mensageria](MENSAGERIA.md)

# Introdução

Muitas vezes, aplicações precisam realizar processamentos períodicos sem que haja uma interferêcia de usuários. Isso é o que Jobs/Rotinas são: implementações que periodicamente realizam processamentos de dados.

> ## _Jobs_ vs _Eventos_
> Muitos dos casos que são resolvidos com rotinas, deveriam ser resolvidos com eventos. Jobs podem ser uma resposta (inadequada) para uma nessidade "assíncrona".

Essa versão da pilha é agressivamente orientada a conteinerização de aplicações e orquestração de contêineres, por isso **NÃO** vamos recomendar o uso da solução Spring para rotinas. **Recomendamos a solução Kubernetes/Openshift para isso**: `CronJob`[^k8s-cronjob] [^ocp-cronjob].

# Dependências e Versionamentos

- chart-cronjob: `0.1.3`[^chart-cronjob]

# Tutoriais

Há pelo menos 2 formas de implementar as rotinas:

- Invocação a Endpoint REST
- Aplicação processa e encerra

## Invocação a Endpoint REST

- [WEB/HTTP (REST)](REST.md)
- [Definições para construção de API REST](https://xpto.com/dti/orientacoes-gerais/guia/blob/master/arquitetura/rest-apis.md)

Nesse caso o processamento desejado fica como efeito colateral da invocação de uma _endpoint_ REST, e o `CronJob` apenas executa periódicamente a invocação desse(s) _endpoint(s)_.
Uma vantagem dessa abordagem é a possibilidade desse endpoint estar disponível para ser invocado por outros clientes que não sejam o `CronJob`. A execução arbitrária fora da agenda pode ser útil ou necessária. Outra possível vantagem é de manter uma base de código mais coesa, uma vez que provavelmente a aplicação que executa o processamento rotineiro é a mesma que implementa as outras operações funcionais. Uma possível desvantagem é que que há uma integração de rede entre a rotina e o que por ela deveria ser processado o que pode dificultar a depuração da aplicação.

### Helm

É preciso adicionar no `Chart.yaml` a dependência do chart de _cron job_:
```yaml
dependencies:
- name: cronjob
  version: 0.1.3
  repository: http://charts.capes.gov.br/capes/infra
```

E configurar adequadamente os arquivos de `values-*.yaml`:

- **Imagem**: a imagem pode ser uma imagem como uma `netshoot` (`replicas/netshoot`); ou uma criada, personalizada pelo time
  - O template do chart já pressupõe a URI do ChartMuseum da CAPES como prefixo, por isso usar apenas o _path_ (`replicas/netshoot`, `myapp/myapp-backend`, etc.)
- **ConfigMaps** & **Volumes**: utilizar ConfigMaps e montagem deles em volumes de maneira adequada. O script que autentica no SSO e executa a invocação do _endpoint_ pode ser definido como um ConfigMap e montado no local adequado

---

#### SSO da CAPES

O SSO da CAPES tem disponível o fluxo de autenticação _Client Credentials_, que pode ser utilizado para autenticar o job. É preciso configurar o "Provedor de Serviço" para aceitar esse fluxo e copiar a chave gerada para uma Secret[^k8s-secret] [^ocp-secret], para que seja possível disponibilizar para o pod que realiza a request ao _endpoint_.

> É razoável que o serviço exija autenticação para a execução do processamento rotineiro. Isso evita que chamadas indesejadas ou equivocadas sejam realizadas.

#### Exemplo de script de job

```sh
#!/bin/sh

BASIC_CREDENTIALS=$(echo -n $CREDENTIALS | base64 | tr -d '\n')

# Obter o token do SSO
curl -X POST -H "Authorization: Basic $BASIC_CREDENTIALS" -H 'application/x-www-form-urlecoded' 'https://sso.capes.gov.br/sso/oauth?grant_type=client_credentials' | jq -r '.access_token' > /tmp/bearer_token.txt

if [ $? -ne 0 ]; then
  echo "Erro ao tentar obter o token de OAuth" > /dev/stderr
  exit 2;
fi

if [ -r /tmp/bearer_token.txt ]; then
  # Invocar o processamento dos arquivos
  if [ $(curl -X POST -o /dev/stderr -w '%{http_code}' -H "Authorization: Bearer $(cat /tmp/bearer_token.txt)" -H 'Accept: application/json;profile="http://myapp.capes.gov.br/refs/UmServico",application/problem+json' 'http://myapp-prod-backend-app.myapp-prod.svc/rest/um-servico') -eq 200 ]; then
    exit 0;
  else
    exit 1
  fi
else
  echo "Não é possível ler do arquivo com o token" > /dev/stderr
  exit 3
fi

```

> Utilizar `netshoot` ou imagens baseadas nele dá acesso a ferramentas como `curl`[^man-curl] e `jq`[^man-jq] que são necessárias para realizar as requisições HTTP e manipulações de JSON.

- `CREDENTIALS` é uma variável de ambiente que deve conter o valor copiado do SAdmin
  - É preciso codificar em base64 para enviar como Basic de `Authorization`
  - O valor de `CREDENTIALS` é carregado por uma secret
- É importante fazer bom uso do DNS e roteamento interno ao cluster:
  - invocar o serviço: `<nome-do-servico>.<nome-do-namespace>.svc`

---

## Aplicação processa e encerra

Também é possível escrever uma aplicação Spring Boot que inicia, realiza algum processamento e depois apenas encerra. Nesse caso como não há nenhuma forma de interagir com a aplicação o mais natural é que o `CronJob` rode uma imagem da própria aplicação. Essa abordagem pode ser considerada mais adequada para otimização de recursos. Exemplo: se a aplicação tiver 5 pods para serviços, mas apenas um deles vai executar eventualmente a rotina, a implementação da rotina está inflando todos os pods que não fazem uso da funcionalidade.

### Java

Nesse caso podemos implementar o processamento no método `run(ApplicationArguments args)` da interface `ApplicationRunner`[^springboot-apprunner]

**Exemplo.java**:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ExemploApplication implements ApplicationRunner {

  private static Logger logger = LoggerFactory.getLogger(ExemploApplication.class);

  public static void main(String[] args) {
    SpringApplication.run(ExemploApplication.class, args);
  }

  @Override
  public void run(ApplicationArguments args) throws Exception {
    //implementação do processamento rotineiro
  }

}
```

A implementação de `ApplicationRunner` precisa ser um _bean_ do Spring. Como o que é anotado com `@SpringBootApplication` é um `@Component`[^spring-component-annotation], um dos modos mais simples é fazer a própria _main class_ implementar `ApplicationRunner`, desde que as recomendações e boas práticas de implementação sejam seguidas (KISS, DRY, complexidade ciclomática, encapsulamento, coesão, tamanho de métodos, etc.).

> ##### `br.gov.capes.cgs.narq:web-starter` & `org.springframework.boot:spring-boot-starter-web`
> Observar que a presença dessas dependências no `pom.xml` cria uma aplicação web que fica aguardando requisições e não encerra após a conclusão do método `run(ApplicationArguments args)`. Então essas dependências são incompatíveis com implementações de aplicações que sejam **apenas** rotinas cuja imagem será **executada** num `CronJob`.

### Helm

Nesse caso provavelmente o job vai ficar num reposítório separado (pelo menos numa camada separada) e deve ter seu próprio Dockerfile.

> #### 02 camadas por repo, 01 Dockerfile por camada
> Lembrar que o pipeline de aplicações da CAPES só aceita 2 camadas por repositório e apenas um Dockerfile por camada. Então se já existe um serviço na camanda _backend_, o job deveria ser colocado na camada _frontend_, o que é bizarro e aponta para ter a base de código do job mantida num repositório separado.
> ##### Invocação a Endpoint REST -> Apenas mais um chart
> Uma outra vantagem de utilizar **Invocação a Endpoint REST** é que desde que a imagem não seja personalizada (utilizando apenas um script via ConfigMap, por exemplo) a configuração da rotina é apenas mais um container no _backend_, gerenciado por um chart, configurado no mesmo aquivo de values[^ex-values].

É preciso adicionar a dependência no arquivo `Chart.yaml` da camada da mesma forma que para **Invocação a Endpoint REST**.

Em vez de um `ConfigMap` com o script que invoca o _endpoint_ que processa a rotina provavelmente será necessário criar um com o `application.yml` e montá-lo do modo adequado, como é usual com contêineres que rodam aplicações Spring Boot.

# Notas e Referências

[^k8s-cronjob]: CronJob - https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/
[^ocp-cronjob]: Running tasks in pods using jobs - https://docs.openshift.com/container-platform/4.7/nodes/jobs/nodes-nodes-jobs.html
[^chart-cronjob]: README - https://xpto.com/cgs/DEVOPS/helm/cronjob/blob/master/README.md
[^springboot-apprunner]: `org.springframework.boot.ApplicationRunner` - https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/ApplicationRunner.html
[^spring-component-annotation]: `org.springframework.stereotype.Component` https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Component.html
[^man-curl]:curl.1 the man page - https://curl.se/docs/manpage.html
[^man-jq]: jq Manual - https://stedolan.github.io/jq/manual/
[^k8s-secret]: Secrets - https://kubernetes.io/docs/concepts/configuration/secret/
[^ocp-secret]: Understanding secrets - https://docs.openshift.com/container-platform/4.7/nodes/pods/nodes-pods-secrets.html#nodes-pods-secrets-about_nodes-pods-secrets
[^ex-values]: Exemplo de configuração de values para cronjob - [values-prod.yaml](./APENDICES/values-prod.yaml)
