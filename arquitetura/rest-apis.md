As palavras "**DEVE**" expressa única opção de realização, "**NÃO DEVE**"" (= **!DEVE**) expressa que não se deve realizar desta forma, "**REQUERIDO**" indica que o item é requerido,  "**PODE**" pode-se realizar de tal forma, desde que não Infrinja nenhuma outra regra; "**NÃO PODE**" (= **!PODE**)[^rfc2119].

# Leituras Recomendadas

1. REST API Design Rulebook[^rest-api-rulebook]
2. REST in Practice[^rest-in-practice]
3. RESTful Web APIs[^restful-Web-apis]

> Embora essa documentação almeje por suficiência, essas foram as leituras principais que inspiraram o conteúdo e recomendamos sua assimilação por serem mais profundas e extensas cobrindo muito mais do que podemos fazer por aqui.

# Introdução

Esse texto documenta conceitos, práticas e recomendações para a implementação de sistemas distribuídos fazendo uso de **Protocolo de Transferência de Hipermídia**[^http].

Para saber se esta arquitetura se aplica corretamente na resolução de seu problema, atente para o princípios/restrições impostos pelo **REST**:

- **_Client-Server_**: trate preocupações do cliente no lado do cliente e as do servidor no lado do servidor. Com isso, tem-se que, no lado do cliente, melhora a portabilidade de interface entre várias plataformas. Já no lado do servidor, ganha-se na escalabilidade e, por consequência, disponibilidade.
- **_Stateless_**: cada solicitação do _Client_ ao _Server_ **DEVE** conter todas as informações necessárias para o completo atendimento da requisição. Nenhum contexto armazenado no servidor deverá (= **NÃO DEVERÁ**) ser necessário para o atendimento da requisição. Nota-se que o gerenciamento da Sessão recai aos cuidados daquele que faz a requisição - _Client_.
- **_Cacheable_**: é-se exigido que os dados de uma resposta sejam implicita ou exeplicitamente  marcados como passível ou não de cache. Esta prerrogativa permitirá que o cliente melhore sua política de cache.
- **_Uniform interface_**: Ao aplicar o princípio de generalidade da engenharia de software à interface do componente, a arquitetura geral do sistema é simplificada e a visibilidade das interações é aprimorada. Para obter uma interface uniforme, são necessárias várias restrições arquiteturais para orientar o comportamento dos componentes. O **REST** é definido por quatro restrições de interface:
  1. identificação de recursos;
  2. manipulação de recursos através de representações;
  3. mensagens auto-descritivas; e,
  4. hipermídia como o mecanismo do estado do aplicativo.
- **_Layered system_**: o estilo do sistema em camadas permite que uma arquitetura seja composta de camadas hierárquicas, restringindo o comportamento do componente, de modo que cada componente não possa "ver" além da camada imediata com a qual está interagindo.

_**RE**presentacional **S**tate  **T**ransfer_[^rest-traducao] (**REST**) é um estilo de arquitetura de software onde se usa o protocolo **HTTP** para interagir com **Representações** de **Recursos**.

# Formato de URI[^uri-rfc]

_**U**niform **R**esource **I**dentifier_[^uri-traducao] é um uma sintaxe especificada para identificar um **Recurso**. O protocolo **HTTP** utiliza **URI** para localizar[^url-obs] as **Representações** e realizar as interações especificadas no protocolo.
**URIs** são primordiais para a definição de uma API **REST**. As Representações de um Recurso numa API REST são usualmente conhecidas como _endpoints_. Para padronizar o estabelecimento desses _endpoints_ vamos fazer algumas especificações e definições. Essas regras e restrições esperam tanto atender a elementos das especificações tecnológicas utilizadas (HTTP, JSON, etc.), quanto criar uma padronização e familiaridade para os desenvolvedores que precisaram implementar integrações com essas APIs.

Recapitulando que o formato geral de URI é:
`URI = schema "://" authority "/" path [ "?" query ] [ "#" fragment ]`

- **REGRA**: URIs **NÃO DEVE** conter barra a direita de `path`
  - Certo: `http://afazeres.capes.gov.br`
  - Errado: `http://afazeres.capes.gov.br/`
  - Certo: `http://afazeres.capes.gov.br/feeds`
  - Errado: `http://afazeres.capes.gov.br/feeds/`
  - Certo: `http://afazeres.capes.gov.br/feeds?id=1234567890`
  - Errado: `http://afazeres.capes.gov.br/feeds/?id=1234567890`
- **REGRA**: hífen (`-`) **DEVE** ser utilizado para merlhora a legibilidade da URI[^kebab-case] (em vez de fazer uso de espaçõs ou sublinhado)
- **REGRA**: **DEVE** usar letras minúsculas na URI
  - O RFC 3986[^uri-rfc] define os URIs que diferenciam maiúsculas de minúsculas, exceto os componentes do esquema e do host. Por exemplo:
    1. `http://api.afazeres.capes.gov.br/rest/usuarios/123/notificacoes-ativas`
    2. `HTTP://API.AFAZERES.CAPES.GOV.BR/rest/usuarios/123/notification-activate`
    3. `http://api.afazeres.capes.gov.br/rest/usuarios/123/Notification-activate`
    4. `http://api.afazeres.capes.gov.br/rest/usuarios/123/Notification-Activate`
    5. `http://api.afazeres.capes.gov.br/rest/usuarios/123/NOTIFICATION-ACTIVATE`
    - Conforme a especificação (RFC 3986), as URIs 1 e 2 são identicas;
    - Segundo essa nossa **REGRA** apenas 1 está Certa.
- **REGRA**: Extensão de arquivo DEVE ser suprimida
  - Extensão de arquivo **NÃO DEVE** ser usada para indicar o formado dos dados
  - O _Client_ **DEVE** ser instruído a usar **HTTP Header** `Accept-Type` para lidar com o tipo de dado retornado
  - Certo: `http://api.afazeres.capes.gov.br/rest/usuarios/123/notifications`
  - Errado: `http://api.afazeres.capes.gov.br/rest/usuarios/123/notifications.json`

## Desenho de `authority` em URI

- **REGRA**: **DEVE** ser usados nomes de subdomínios consistentes  para APIs
  - O domínio de nível superior e os nomes dos primeiros subdomínios (por exemplo, `api.capes.gov.br`) de uma API devem identificar seu proprietário do serviço. O nome completo do domínio de uma API deve adicionar um subdomínio chamado api.
  - Certo: `http://api.afazeres.capes.gov.br`
  - Errado: `http://afazeres.api.capes.org.br`
  - Errado: `http://afazeres.capesapi.capes.org.br`
  - Errado: `http://afazeres.apicapes.capes.org.br`

# Arquétipos de Recursos

Apesar de URI não fazer qualquer distinção entre os recursos apontados no _endpoints_, é conveniente estabelecer alguns arquétipos para facilitar a vida dos desenvolvedores.

## _Collection_

Um _Collection Resource_ representa um diretório de recursos gerenciado pelo servidor.  Clients podem propor novos recursos a serem adicionados a uma coleção. No entanto, cabe à coleção optar por criar um novo recurso, ou não. Um recurso de coleção escolhe o que deseja conter e também decide os URIs de cada recurso contido.
**Exemplos**:
- `http://api.afazeres.capes.gov.br/rest/usuarios`
- `http://api.afazeres.capes.gov.br/rest/usuarios/{uuid}/afazeres`[^uri-template]
- `http://api.afazeres.capes.gov.br/rest/labels`

## _Store_

Um _Store_ é um repositório de recursos gerenciado pelo _Client_. Um recurso de armazenamento permite que um cliente da API inclua recursos, recupere-os e decida quando excluí-los. Por conta própria, _Store_ não criam novos recursos; portanto, um _Store_ nunca gera novos URIs. Em vez disso, cada recurso armazenado possui um URI que foi escolhido por um cliente quando foi inicialmente colocado na _Store_.
**Exemplos**:
- No _Store_ `http://api.afazeres.capes.gov.br/rest/usuarios/{uuid}/emails` o cliente poderia adicionar um Recurso `http://api.afazeres.capes.gov.br/rest/usuarios/{uuid}/emails/eu%40exemplo.org.br`[^eu-at-exemplo]

## _Document_

Um recurso _Document_ é um conceito singular que é semelhante a uma instância de objeto ou registro de banco de dados. A representação de estados de um _Document_, geralmente, inclui campos com valores e links para outros recursos relacionados.
**Exemplos** (cada URI abaixo identifica um _Document Resource_):
- `http://api.afazeres.capes.gov.br/rest/usuarios/{uuid}`
- `http://api.afazeres.capes.gov.br/rest/usuarios/{uuid}/contato`
- `http://api.afazeres.capes.gov.br/rest/labels/{luid}`

Um **documento** pode ter recursos filhos que representam seus conceitos subordinados específicos[^ddd-aggregate].

Com a capacidade de reunir vários tipos de Resource em um único pai, um Document é um candidato lógico para o recurso raiz de uma API REST, também conhecido como **`docroot`**. O URI de exemplo abaixo identifica a **`docroot`**, que é o ponto de entrada para "Afazeres" REST API:
- `http://api.afazeres.capes.gov.br`

## _Controller_

Um _Controller Resource_[^nao-mvc] modela um conceito procedural. Eles são como funções executáveis, com parâmetros e valores de retorno; entradas e saídas. Como o uso de formulários HTML de um aplicativo Web tradicional, uma API REST depende de _Controller_ resources para executar ações específicas do aplicativo que não podem ser mapeadas logicamente para um dos métodos padrão _CRUD_[^crud].
Os nomes dos controllers, geralmente, aparecem como o último segmento em um caminho de URI, sem recursos filhos para segui-los na hierarquia. O exemplo abaixo mostra um recurso do controlador que permite que um client reenvie um alerta para um usuário:
- `http://api.afazeres.capes.gov.br/rest/notificacoes/{nuid}/enviar`

# Desenho de `path` de URIs

Cada segmento do `path` da URI, separado por barras (`/`), representa uma oportunidade de design. A atribuição de valores significativos a cada segmento de caminho ajuda a comunicar claramente a estrutura hierárquica do design do modelo de recurso de uma API REST.

![](uri-path-design.png)

- **REGRA**: URI representando um recurso _Document_ **DEVE** ser nomeado com um substantivo ou frase substantiva singular
  - `http://api.afazeres.capes.gov.br/rest/usuarios/{uuid}`
- **REGRA**: URI representando um recurso _Collection_ **DEVE** ser nomeado com um substantivo ou frase substrantiva plural
  - `http://api.afazeres.capes.gov.br/rest/usuarios`
- **REGRA**: URI representando um recurso _Store_ **DEVE** ser nomeado com um substantivo ou frase substrantiva plural
  - `http://api.afazeres.capes.gov.br/rest/usuarios/{uuid}/emails`
- **REGRA**: URI representando um recurso _Controller_ **DEVE** ser nomedo com um verbo ou frase verbal
  - `http://api.afazeres.capes.gov.br/rest/usuarios/{uuid}/ativar`
- **REGRA**: Partes variáveis que formam o URI PODERÃO ser substituídas por valores
  - `http://api.afazeres.capes.gov.br/rest/usuarios/{uuid}`
  - `http://api.afazeres.capes.gov.br/rest/usuarios/{uuid}/tarefas/{tuid}`
  - **REGRA**: Externalizar os ideintificadores de documentos como UUID[^uuid-rfc] [^uuid-obs1]
    - Recursos secundários ("_subpaths_") de um documento podem até externalizar IDs em suas representações desde que isso seja seguro o suficiente mesmo em caso de ataques utilizando _loops_ indesejados para interagir com a API[^bad-loop-exemplo]
- **REGRA**: URI que represente _CRUD_ **NÃO DEVE** conter identificadores do tipo da operação
  - Correto: `POST /usuarios`
  - Correto: `GET /usuarios/{uuid}`
  - Correto: `DELETE /usuarios/{uuid}`
  - Errado: `POST /registrarUsuario`
  - Errado: `GET /deleteUsuario?{uuid}`
  - Errado: `DELETE /deleteUsuario?{uuid}`

# Desenho de `query` de URI

Recapitulando: `URI = scheme "://" authority "/" path [ `**`"?" query`**` ] [ "#" fragment ]`

- **REGRA**: O componente `query`, na URI, **PODERÁ** ser usado para filtrar _Collection_ ou `Store`
  - `GET /usuarios?filtro1=valor1`
  - `GET /usuarios?filtro1=valor1&filtro2=valor2`
- **REGRA**: O componente `query`, na URI, `PODERÁ` ser usado para paginar `Collection` ou `Store`
  - `pageSize`: especifica a quantidade máxima de elementos recuperados a cada requisição
  - `pageStartIndex`: iniciando em **zero**, indica o índice do primeiro elemento recuperado
  - `/users?pageSize={num}&pageStartIndex={num}`
> Quando a complexidade dos requisitos de paginação (ou filtragem) de numa requisição exceder os recursos de formatação simples da parte `query`, considere[^graphql] projetar um _Controller_. Por exemplo, o _Controller_ a seguir pode aceitar entradas mais complexas por meio do corpo da requisição de uma solicitação ao invés de fazer parte de `query` do URI:
> - `POST /rest/usuarios/buscar`
>
> Todavia, os cuidados inerentes a cache da consulta deverão ser observados.

# Desenho da Interação através de HTTP

#### HTTP/1.1

**HTTP/1.1** é uma das versões mais difundidas do protocolo HTTP. Vamos pincelar alguns pontos sobre o protocolo e estabelecer mais algumas regras.

## Request Methods[^http-req-methods-traducao]

`Request-Line = `**`Method`** `SP Request-URI SP HTTP-Version CRLF`[^http-req-line-rfc]

Cada método do protocolo tem um propósito:
- `GET`: recuperar uma representação do estado de um recurso;
- `HEAD`: recuperar os metadados associados ao estado do recurso;
- `PUT`: atualizar um recurso;
- `DELETE`: remove um recurso de seu pai;
- `POST`:
  - Criar um novo recurso dentro de uma coleção
  - Executar controladores
- `OPTIONS`: recuperar os métodos disponíveis para cada recurso
- `PATCH`: realizar modificaçõe parciais num recurso[^http-patch-rfc]

- **REGRA**: `GET` e `POST` **NÃO DEVEM** ser usados para encapsular outros métodos de solicitação
- **REGRA**: `GET` `DEVE` ser usado para recuperar uma representação de um recurso
- **REGRA**: `HEAD` **DEVE** ser usado para recuperar cabeçalhos de resposta
- **REGRA**: `PUT` **DEVE** ser usado para atualizar recursos mutáveis
- **REGRA**: `POST` **DEVE** ser usado para criar um Recurso em uma _Collection_
- **REGRA**: `POST` **DEVE** ser usado para executar _Controller_
- **REGRA**: `DELETE` **DEVE** ser usado para remover um recurso de seu pai
  - Um cliente usa `DELETE` para solicitar que um recurso seja completamente removido de seu pai, que geralmente é uma coleção ou armazenamento. Depois que uma solicitação `DELETE` é processada para um determinado recurso, o recurso não pode mais ser encontrado pelos clientes. Portanto, qualquer tentativa futura de recuperar a representação de estado do recurso, usando `GET` ou `HEAD`, deve resultar em um status `404` ("_Not Found_") retornado pela API. O método `DELETE` possui semântica muito específica no HTTP, que não deve ser sobrecarregada ou resiginificada pelo design de uma API REST. Especificamente, uma API não deve distorcer o significado pretendido de `DELETE`, mapeando-a para uma ação menor que deixe o recurso e seu URI disponível para os clientes. Por exemplo, se uma API deseja fornecer uma exclusão "soft" ou alguma outra interação de mudança de estado, deve empregar um recurso especial do controlador e direcionar seus clientes para usar `POST` em vez de `DELETE` para interagir.
- **REGRA**: `OPTIONS` deve ser usada para recuperar os metadados que descrevem as interações disponíveis dos recursos
  - O Client poderá usar o método OPTIONS para recuperar os metadados do recurso que incluem no header: `Allow: GET, PUT, DELETE`. Ainda, `OPTIONS`, pode incluir um corpo que inclua mais detalhes sobre cada opção de interação.

## Response Status Code[^resp-status-code-traducao]

- **REGRA**: `200` ("OK") **DEVE** ser usado para indicar sucesso de forma genérica
  - Ao contrário do código de status `204`, uma resposta `200` **DEVE** incluir um corpo de resposta.
- **REGRA**: `200` ("OK") **NÃO DEVE** ser usado para comunicar erros na resposta
- **REGRA**: `201` ("Created") **DEVE** ser usado para indicar a criação bem-sucedida de recursos
- **REGRA**: `202` ("Accepted") **DEVE** ser usado para indicar o início "bem-sucedido" de uma ação assíncrona
  - Somente _Controller_ podem enviar respostas com status `202`.
- **REGRA**: `204` ("No Content") **PODE** ser usado quando o corpo da resposta estiver intencionalmente vazio
  - Por exemplo se uma _Store_ estiver vazia, ou se os filtros aplicados na consulta não retornam resultados
  - `204` é uma resposta de **Sucesso**
- **REGRA**: `301` ("Moved Permanently") **PODE** ser usado para indicar alteração de localização de um recurso
- **REGRA**: `303` ("See Other") **PODE** ser usado para indicar ao cliente acompnhar o resultado noutra URI
- **REGRA**: `304` ("Not Modified") **PODE** ser usado para preservar o uso desnecessário de banda
  - A diferença chave entre `204` e `304` é que o primeiro, intencionamento, não possui um corpo de resposta, quanto que o segundo tem conteúdo, mas não não difere do existente no _Cache_ em posse do _Client_.
- **REGRA**: `400` ("Bad Request") **PODE** ser usada para indicar falha não específica
  - Para erros na categoria `4xx`, o corpo da resposta pode conter um documento descrevendo o erro do cliente (a menos que o método de solicitação seja HEAD).
    - O corpo de uma respota `4xx` deve seguir a especificação de **Problem Details for HTTP APIs**[^rfc-7807]

## Desenho do Metadata

- **REGRA**: `Content-Type` **DEVE** ser usado
- **REGRA**: `Content-Length` **DEVE** ser usado
- **REGRA**: `Last-Modified`[^last-mod-obs1] **DEVE** ser usado na resposta da requisição
- **REGRA**: `ETag` **PODE** ser usado na reposta da requisição
- **REGRA**: `Location` **DEVE** ser para especificar a URI do recurso criado
- **REGRA**: `Cache-Control`, `Expires`, e `Date` **DEVEM** ser usados para auxiliar no controle de _Cache_
  - `Cache-Control: max-age=60, must-revalidate`
  - `Date: Tue, 15 Nov 1994 08:12:31 GMT`
  - `Expires: Thu, 01 Dec 1994 16:00:00 GMT`
- **REGRA**: `Cache-Control`, `Expires`, e `Date` **PODEM** ser usados para desencorajar o armazenamento em _Cache_
- **REGRA**: _Caching_ **DEVE** ser encorajado sempre que não prejudicar a integradidado da funcionalidade
- **REGRA**: Cabeçalhos de expiração de _Cache_ **DEVEM** ser usados com `200` respostas ("OK")
- **REGRA**: Instrução de expiração de _Cache_ **PODE** ser usados em resposta `3xx` e `4xx`

## Desenho de Representação

- **REGRA**: JSON **DEVE** ser suportado para representação de recursos
- **REGRA**: JSON **DEVE** ser bem formado
- **REGRA**: XML e outros formatos **PODEM** opcionalmente ser usados para representação de recursos
- **REGRA**: Envelopes adicionais não **DEVEM** ser criados
  - Uma API REST deve aproveitar a mensagem "envelope" fornecida pelo HTTP. Em outras palavras, o corpo deve conter uma representação do estado do recurso, sem nenhum _wrapper_ adicional orientado ao transporte.
- **REGRA**: Um esquema consistente **DEVE** ser usado para representar links
- **REGRA**: Um link **DEVE** ser incluído para o elemento corrente (`self`[^iana-link-rel] [^rfc-Web-link] link) no corpo da mensagem de resposta
- **REGRA**: Minimize o número de URIs da API anunciados como "ponto de entrada"
- **REGRA**: Os links **DEVEM** ser usados para anunciar as ações disponíveis de um recurso

## Representação de _Media Type_

- **REGRA**: Um esquema consistente **DEVE** ser usado para representar formatos de tipo de mídia

## Representação de Erros

- **REGRA**: O esquema _Problem Details for HTTP APIs_[^rfc-7807] DEVE ser utilizado
  - É um esquema **consistente** ser usado para representar erros
- **REGRA**: Um esquema consistente **DEVE** ser usado para representar respostas a erros

## Responsabilidades do Cliente

O tipo de cliente HTTP, mais difundido é o _web browser_ (navegador web). Clientes para APIs REST podem não ser diretametne _browsers_, podem ser:
- Aplicações web (HTML+CSS+JS)
- Outras aplicações REST
- etc..


Então para uma integração e utilização devida com os servidores de APIs REST os clientes também precisam seguir algumas recomendações, e os servidores devem ser consistentes e adequados ao lidarem com elas.

### Versionamento

- **REGRA**: Novos URIs **DEVEM** ser usados para introduzir novos conceitos
  - Um recurso é um modelo semântico, como um pensamento sobre uma coisa. A forma e o estado representacional de um recurso podem mudar com o tempo, mas o identificador deve abordar consistentemente o mesmo pensamento, que nenhum outro URI pode identificar. Além disso, todos os personagens no URI de um recurso contribuem para sua identidade. Portanto, a versão de uma API REST, ou qualquer um de seus recursos, normalmente não deve ser significada em um URI. Por exemplo, incluir um indicador de versão, como v2, em um URI indica que o próprio conceito possui várias versões, o que geralmente não é a intenção.
- **REGRA**: Os esquemas **DEVEM** ser usados para gerenciar versões de formulário representacionais
- **REGRA**: _Entity tags_ (`ETag`)[^etag] **DEVEM**  ser usadas para gerenciar versões de estado representacional

> No contexto de REST o que se manipula e se interage é com uma **Representação**  de um **Recurso**. Um recurso pode ter diferentes versões por 2 motivos:
> - Mudança de seu estado interno;
> - Mudança de sua estrutura.
>
> Uma mudança de estado interno é o que se espera com a mudança de um valor de um campo, adição ou subtração de uma informação. Esse tipo de versionamento pode ser otimizado com `Etag`. Vamos reconhecer que uma boa `ETag` deve ser calculável rapidamente e estar disponível de maneira mais ágil que consultar o estado real do conceito e tudo isso impacta no projeto e na implementação da aplicação/API.
> Uma mudança de estrutura normalmente envolve a adição ou subtração de campos (não do conteúdo de campos já existentes). Isso pode impactar os **Clientes** da API. Existem mudanças retrocompatíveis, como normalmente é a adição de campos. Remoção de campos normalmente cria uma quebra de compatibilidade. Mudança de nomenclatura de campos ou reestruturamento de JSON podem ser considerados extritamente como remoções de campos seguidas de adições, o que torna as coisas normalmente incompatíveis por conta de remoção. O caminho natural é lidar com diferentes versões de **Representação** de **Recursos** no caso de introdução de mudanças incompatíveis[^versioning-greg-young].
>
> **"Be conservative in what you send, be liberal in what you accept"**[^robustness-principle]
>
> A ideia de "seja conservador no que manda, seja liberal no que aceita" é uma mentalidade de desenvolvimento que pode ser considerada nesse escopo de versionamento de **Representações**. Algo a se ter em consideração é que os papeis de remetente (_sender_) e destinatário (_receiver_) são espelhados no esquema **cliente/servidor**. O cliente envia uma requisição (_request_) e aceita uma resposta (_response_). O servidor envia uma _response_ e aceita _request_. Ambos atuam nos dois papeis ao longo das interações. O Servidor/API deve mandar respostas bem formadas, com os _Media Types_ precisos e específicos. Mas como os papeis se espelham o cliente também deveria ser cuidadoso ao enviar seus _payloads_, utilizando os cabeçahos corretos e o mais precisamente compativeis com o conteúdo enviado. Normalmente é entendido um imbalanço entre o cliente e o servidor. _A API normalmente tem de acomodar e sofrer mais que os clientes_.

## Composição da Representação na Resposta

- **REGRA**: O componente `query` de um URI **PODERÁ** ser usado para indicar respostas parciais desejadas
  - _Request_
  ```http
  GET /rest/usuarios/123?fields=(prenome,dataNascimento) HTTP/1.1
  Host: api.afazeres.capes.gov.br
  ```
  - _Response_[^content-type-obs1]
  ```http
  HTTP/1.1 200 OK
  Content-Type: application/vnd.capes.gov.br+json;profile="https://rest.capes.gov.br/refs/afazeres/Usuario"
  {
    prenome" : "Fulana",
    "dataNascimento" : "1992-07-31"
  }
  ```

- **REGRA**: O componente `query` de um URI **PODE** ser usado para indicar respostas parciais indesejadas
  - _Request_[^content-type-obs2]
  ```http
  GET /rest/usuarios/123?fields=!(endereco,afazeres!(quarta,sexta)) HTTP/1.1
  Host: api.afazeres.capes.gov.br
  ```
  - _Response_
  ```http
  HTTP/1.1 200 OK
  Content-Type: application/vnd.capes.gov.br+json;profile="https://rest.capes.gov.br/refs/afazeres/Usuario"
  {
    "prenome" : "Fulana",
    "dataNascimento" : "1992-07-31",
    "afazeres" : {
      "segunda" : {
        "links" : {
          "daily" : {
            "href" : "http://api.afazeres.capes.gov.br/rest/afazeres/narq-daily-meeting",
            "rel" : "https://rest.capes.gov.br/refs/afazeres/Afazer"
          },
          # Outros afazeres regulares de segunda
        }
      },
      # Afazeres regulares em outros dias exceto quarta e sexta
    },
    # Outros campos de Pessoa, exceto endereço
    "links" : {
      # Links da pessoa
    }
  }
  ```
- **REGRA**: O componente `query` de um URI **PODE** ser usado para incorporar recursos vinculados
  - _Request_
  ```http
  GET /rest/usuarios/123?embed=(ferias) HTTP/1.1
  Host: api.afazeres.capes.gov.br
  ```
  - _Response_[^content-type-obs3]
  ```http
  HTTP/1.1 200 OK
  Content-Type: application/vnd.capes.gov.br+json;profile="https://rest.capes.gov.br/refs/afazeres/Usuario"
  {
    "prenome" : "Fulana",
    "dataNascimento" : "1992-07-31",
    "ferias" : [
      {
        "id" : "ferias-123-2020-2021",
        "name" : "Férias do período 2020 à 2021",
        "links" : {
          "self" : {
            "href" : "http://api.afazeres.capes.gov.br/rest/afazeres/ferias-123-2020-2021",
            "rel" : "https://rest.capes.gov.br/refs/commons/self"
          }
        }
      }
    ]
    # Outros campos...
    "links" : {
      "self" : {
        "href" : "http://api.afazeres.capes.gov.br/rest/usuarios/123",
        "rel" : "https://rest.capes.gov.br/refs/afazeres/Pessoa"
      },
      # Outros links...
    }
  }
  ```
---
### Sobre `Content-Type`

- **REGRA**: _Media Types_ específicos **DEVEM** ser utilizados[^vide-leituras-1]

O cabeçalho de `Content-Type` é normalmente utilizado de maneira simplista ou até ingênua. Um boa API REST deve ter o objetivo de se valer de **HATEOAS**[^hateoas]. Isso implica que o formato de hipermídia utilizado deve ser consistente e rico para representar o domínio negocial. Alguns formatos de hipermídia como **Atom**[^rfc-atom] são amplamente capazes de lidar com uma vastidão de domínios, até com alguma possibilidade de extensão. Um detalhe é que **Atom** é codificado em XML, o _Media Type_[^iana-media-types] normalmente é `application/atom+xml`[^rfc-atom] ou `application/atomsvc+xml`[^rfc-atompub]. Outros formatos como **OData**[^odata], **JSON Schema**[^json-schema], **WRML**[^WRML] já foram propostos para dar suporte a codificação em JSON, mas algumas condiderações devem ser feitas:
- OData apesar de utilizar _parameters_[^rfc-media-type-params], mesmo que não aprovados pela IANA[^iana-media-types-parameters], como `odata.metadata`, tem como _Media Type_ `application/json`, que não é um formato de hipermídia[^vide-leituras-1];
- JSON Schema apesar de bastante difundido, até hoje não passou de um internet draft[^json-schema-internet-draf02];
- WRML nunca foi aprovado pela IANA[^iana-media-types]

Alguns formatos de _Media Type_ são adequados para casos um pouco mais específicos:
- `application/problem+json`[^rfc-7807-media-type] é o formato proposto pelo RFC 7807[^rfc-7807] e tem até um mecanismo de extensão que alinhado com os demais princípios recomendados atende adequadamente o reporte de erros
- `application/json-patch+json` definido em **_JavaScript Object Notation (JSON) Patch_**[^rfc-6902] para submissão de conteúdo codificado em JSON na semântica do verbo `PATCH` de HTTP
  - `application/merge-patch+json` é um outro formato compatível com a utilização do verbo `PATCH`

**A alternativa é que as aplicações definam algo na _vendor tree_**[^rfc-media-type-vendor-tree].
Nos exemplos dessa documentação temos utilizado `application/vnd.capes.gov.br+json`, com o _parameter_ `profile`[^profile-parameter-obs1], apontando para um URI. Esse ainda não é um formato que tenha realmente sido especificado pelo NARQ. No momento não temos a certeza de que tal especificação será realizada. Apesar disso algumas coisas do espírito desse exemplo podem ser abraçadas:
- Estruturado em JSON (sufixo "`+json`");
- Parâmetro `profile` apontando para uma URI;
  - A URI de `profile` deve conter alguma forma de documentação ou metamodelo do conteúdo
- Esperar que eventualmente a NARQ especifique um catálogo de _profiles_ onde essas documentações/metamodelos fiquem disponíveis (talvez `https://rest.capes.gov.br/refs` como nos exemplos)

Alternativamente os times podem desenvolver seus próprios "_vendors_" (exemplo `vnd.csapg.capes.gov.br+json`, `vnd.csab.capes.gov.br+xml`, `vnd.scba.capes.gov.br+html`) e ter seu próprio catálogo (exemplo `http://siapg.capes.gov.br/refs`):
- **REGRA**: As aplicações **PODEM** implementar seus _Media Types_


> Devido a uma miopia quando implementamos a versão 1.x[^Wiki-pilha-java] da pilha Java[^arq-ref-java] da Arquitetura de Referência[^arq-ref], não impomos a regra discutida nessa secção. Os times que desenvolveram aplicações utilizando essa versão da pilha podem contar com uma certa leniência a respeito da aderência a essa regra e outras regras de [Desenho de Representação](#desenho-de-representação) que apontam na mesma direção.

#### Negociação de Conteúdo

Recapitulando sobre as regras de [Versionamento](#versionamento) é visível que pode ser necessário conviver com diferentes versões do esquema de Representação de um dado conceito no mesmo Recurso.

- **REGRA**: Aplicações **DEVEM** utilizar `Content-Type` para negociação de diferentes Representações de um Recurso
  - Certo:
  ```http
  GET /rest/usuarios?prenome=Fulana HTTP/1.1
  Host api.afazeres.capes.gov.br
  Accept: application/vnd.capes.gov.br+json;profile="https://rest.capes.gov.br/refs/afazeres/Pessoa2",application/problem+json
  ```
- **REGRA**: Aplicações **NÃO PODEM** utilizar esquemas de versionamento no `path` da URI
  - Errado: `http://api.afazeres.capes.gov.br/rest/v1/usuarios`
  - Errado: `http://api.afazeres.capes.gov.br/rest/usuarios/123/v1`
- **REGRA**: Aplicações **NÃO PODEM** definir versionamento no `query` URI
  - Errado: `http://api.afazeres.capes.gov.br/rest/usuarios?version=v1`

---

# Miscelância de Recomendações

Algumas práticas podem facilitar a implementação ou a manutenção do código que implementa algumas operações, mas não vamos colocar como regras rígidas. Então as palavras chaves vão ser **EVITE** e **PREFIRA**.

- **RECOMENDAÇÃO**: **EVITE** o encadeamento de IDs no `path` da URI dos _endpoints_
  - Evite: `http://api.afazeres.capes.gov.br/rest/usuarios/{uuid}/afazeres/{auid}/notificacoes/{nuid}/enviar`
  - **PREFIRA**: `http://api.afazeres.capes.gov.br/rest/notificacoes/{nuid}/enviar`

  Provavelmente o `nuid` vai ser globalmente única, pelo menos no contexto de **Notificações**, mesmo que uma **Notificação** esteja associada exclusivamente[^ddd-aggregate] a um único **Afazer** e seu `auid`, que por sua vez esteja associado exclusivamente a um único **Usuário** com seu `uuid`.
  Em geral o que se espera é que a pessoa identificada na requisição (através do cabeçalho `Authorization`) tem permissão para executar (ou re-executar) o envio de uma dada **Notificação**, seja por ser um usuário com perfil gerencial, ou a pessoa **Usuária** efetivamente associada com aquela **Notificação**.
- **RECOMENDAÇÃO**: **PREFIRA** que haja um `path` base para as interações REST[^path-contexto]
  - `http://api.afazeres.capes.gov.br/rest`
  - Outros contextos que podemos imaginar:
    - `http://api.afazeres.capes.gov.br/diagnostics`: para monitoramento da "saúde" da aplicação[^diagnostics-spec]
    - `http://api.afazeres.capes.gov.br/refs`: para documentações e catálogo de metamodelos dos _Media Types_ da aplicação
    - `http://api.afazeres.capes.gov.br/feeds`: para _feeds_ de eventos
    - `http://api.afazeres.capes.gov.br/graphql`: para GraphQL[^graphql]
    - `ws://api.afazeres.capes.gov.br/websockets`: para WebSockets[^rfc-Web-sockets]
- **RECOMENDAÇÃO**: Ao implementar seu _Media Type_ **PREFIRA** versionar o esquema de Representação em um parâmetro _profile_[^vnd-capes]
  - **RECOMENDAÇÃO**: **PREFIRA** utilizar um sufixo numérico para indicar uma versão de esquema de Representação
    - Prefira: `profile="https://rest.capes.gov.br/refs/afazeres/Usuario5"`
    - Evite: `profile="https://rest.capes.gov.br/refs/afazeres/Usuario-v5"` (sufixo com traço e letra)
    - Evite: `profile="https://rest.capes.gov.br/refs/afazeres/Usuario1.5"` (sufixo com ponto)
    - Evite: `profile="https://rest.capes.gov.br/refs/afazeres/UsuarioNovo"` (sufixo com não numérico)
    - Evite: `profile="https://rest.capes.gov.br/refs/afazeres/Usuario2021-06-15"` (sufixo com data)
  - **RECOMENDAÇÃO**: **PREFIRA** ter uma versão "_ever green_" sem sufixo numérico que seja um _alias_ para a versão preferida pela API
    - `profile="https://rest.capes.gov.br/refs/afazeres/Usuario"`
    - **RECOMENDAÇÃO**: **EVITE** retornar o valor _ever green_
      - Evite retornar:
      ```http
      HTTP/1.1 200 OK
      Content-Type: application/vnd.capes.gov.br+json;profile="https://rest.capes.gov.br/refs/afazeres/Pessoa"
      ```
      - Prefira retornar:
      ```http
      HTTP/1.1 200 OK
      Content-Type: application/vnd.capes.gov.br+json;profile="https://rest.capes.gov.br/refs/afazeres/Pessoa3"
      ```
    > Para evitar futuras confusões a primeira versão já deveria deveria ter um sufixo numérico que tem a versão _ever green_ como _alias_ (o _alias_ `Usuario` nasce pra versão `Usuario0`).
    > Quando um cliente recebe um _profile_ diferente da versão que ele acredita entender, por exemplo, receber `Usuario1` em vez de `Usuario0` ele pode tentar [negociar](#negociação-de-conteúdo) com a API pela versão compatível, por isso é importante que os _alias_ de _ever green_ sejam evitados ao implementar os decodificadores/processadores nos **Clientes** da API.


# Notas e Referências

[^rest-api-rulebook]: ISBN: 9781449310509 - https://www.oreilly.com/library/view/rest-api-design/9781449317904/
[^rest-in-practice]: ISBN: 9780596805821 - https://www.oreilly.com/library/view/rest-in-practice/9781449383312/
[^restful-Web-apis]: ISBN: 9781449358068 - https://www.oreilly.com/library/view/restful-web-apis/9781449359713/
[^rfc2119]: RFC 2119 - https://tools.ietf.org/html/rfc2119
[^http]: "_HyperText Transfer Protocol_" (HTTP)
[^rest-traducao]: "Transferência Representacional de Estado"
[^uri-rfc]: RFC 3986 - https://datatracker.ietf.org/doc/html/rfc3986
[^uri-traducao]: Identificador Uniforme de Recurso
[^url-obs]: Apesar de URL (_Uniform Resource Locator_) ser provavelmente mais familiar, não há necessidade de utilizar essa terminologia, vamos dar preferência a **URI**.
[^kebab-case]: Esse formato é conhecido como _kebab-case_ em contexto de programação
[^uri-template]: Esse formato entre "chaves" (`{uuid}`) pode ser encarado como _URI Template_. RFC 6570 - https://datatracker.ietf.org/doc/html/rfc6570
[^eu-at-exemplo]: `eu%40exemplo.org.br` é **`eu@exemplo.org.br`** codificado para URIs conforme o RFC 3986 determina
[^nao-mvc]: Não confundir nem misturar com o _Controller_ de **MVC**.
[^crud]: _Create, Read, Update, and Delete_ (Criar, Ler, Atualizar e Deletar) é o padrão fundamental de persistência de dados.
[^uuid-rfc]: RFC 4122 - https://datatracker.ietf.org/doc/html/rfc4122
[^uuid-obs1]: Isso provavelmente implica em incluir uma associação persistente na base de dados, por exemplo em bases relacionsi adicionar uma coluna na tabela que grava dados de uma representação de recurso
[^bad-loop-exemplo]: Exemplo um _loop_ que tente deletar todas as tarefas de uma usuária "`03a3c514-ce2d-11eb-b8bc-0242ac130003`", tentando todos os `tuid` de `1` até `1000000` em http://api.afazeres.capes.gov.br/rest/usuarios/{uuid}/tarefas/{tuid}`
[^graphql]: Um formato alternativo a REST que tem tido destaque para consultas complexas é o GraphQL - https://graphql.org/ - que poderia por exemplo ser implementado no _endpoint_ `http://api.afazeres.capes.gov.br/graphql`
[^http-req-methods-traducao]: Métodos de Requisições. Como o modelo REST é um modelo "cliente-servidor" o servidor provê a representação e processa as transformações através de requisições que são feitas por algum dos métodos de HTTP
[^http-req-line-rfc]: https://datatracker.ietf.org/doc/html/rfc2616#page-35
[^http-patch-rfc]: RFC 5789 - https://datatracker.ietf.org/doc/html/rfc5789
[^resp-status-code-traducao]: Códigos de Status de Resposta
[^rfc-7807]: RFC 7807 - https://datatracker.ietf.org/doc/html/rfc7807
[^last-mod-obs1]: Nem sempre essa informação está disponível na base de dados, mas é algo que deve ser projetado para novos conceitos e para a evolução e manutenção dos vigentes
[^iana-link-rel]: IANA Link Relations - https://www.iana.org/assignments/link-relations/link-relations.xhtml
[^rfc-atom]: The Atom Syndication Format. RFC 4287 - https://datatracker.ietf.org/doc/html/rfc4287
[^rfc-Web-link]: RFC 5988 - https://datatracker.ietf.org/doc/html/rfc5988
[^content-type-obs1]: O `Content-Type` poderia informar qual a projeção de campos que está sendo retornada: `Content-Type: application/vnd.capes.gov.br+json;profile="https://rest.capes.gov.br/refs/afazeres/Pessoa";fields="(prenome,dataNascimento)"`
[^content-type-obs2]: O `Content-Type` poderia informar qual a projeção de campos que está sendo retornada: `Content-Type: application/vnd.capes.gov.br+json;profile="https://rest.capes.gov.br/refs/afazeres/Pessoa";fields="!(endereco,afazeres!(quarta,sexta))"`
[^content-type-obs3]: O `Content-Type` poderia informar qual a junção que está sendo retornada: `Content-Type: application/vnd.capes.gov.br+json;profile="https://rest.capes.gov.br/refs/afazeres/Pessoa";embed="(ferias)"`
[^hateoas]: _Hypermedia as the Engine of Application State_. Vide as leituras recomendadas para entender o conceito[^rest-in-practice].
[^odata]: https://www.odata.org/
[^json-schema]: https://json-schema.org/
[^WRML]: https://github.com/wrml/wrml
[^rfc-atompub]: Atom Publishing Protocol (AtomPub). RFC 5023 -
https://datatracker.ietf.org/doc/html/rfc5023
[^iana-media-types]: IANA Media Types - https://www.iana.org/assignments/media-types/media-types.xhtml. RFC 6838 - https://datatracker.ietf.org/doc/html/rfc6838
[^rfc-media-type-params]: https://datatracker.ietf.org/doc/html/rfc6838#section-4.3
[^json-schema-internet-draf02]: https://datatracker.ietf.org/doc/html/draft-handrews-json-schema
[^iana-media-types-parameters]: https://www.iana.org/assignments/media-types-parameters/media-types-parameters.xhtml
[^vide-leituras-1]: Vide leituras recomendadas para entender[^rest-in-practice]
[^rfc-7807-media-type]: https://datatracker.ietf.org/doc/html/rfc7807#section-6.1
[^rfc-6902]: RFC 6902 - https://datatracker.ietf.org/doc/html/rfc6902
[^arq-ref]: https://intranet.capes.gov.br/diretoria-de-tecnologia-da-informacao-dti/mais/procedimentos-e-normas/item/download/174_3004d8de7d62295b09dc06c4080d9188
[^arq-ref-java]: https://intranet.capes.gov.br/diretoria-de-tecnologia-da-informacao-dti/mais/procedimentos-e-normas/item/download/175_469cc4bab52d92baa9ef33d27b5a7757
[^Wiki-pilha-java]: https://wiki.capes.gov.br/index.php/DTI:Arquitetura_Servicos_Java
[^rfc-media-type-vendor-tree]: https://datatracker.ietf.org/doc/html/rfc6838#section-3.2
[^profile-parameter-obs1]: O parâmetro `profile` está cadastrado na IANA como um _Link Relation_[^iana-link-rel]. Vide as leituras recomendas para entender melhor[^restful-Web-apis]
[^ddd-aggregate]: O que em DDD provavelmente seria um Aggregate - https://martinfowler.com/bliki/DDD_Aggregate.html
[^path-contexto]: Esse primeiro elemento de `path` é usualmente conhecido como "contexto"
[^rfc-Web-sockets]: RFC 6455 - https://datatracker.ietf.org/doc/html/rfc6455
[^diagnostics-spec]: Conforme [especificado](arquitetura/arquitetura/monitoramento-aplicacoes.md) pelo NARQ
[^versioning-greg-young]: https://leanpub.com/esversioning/read tem uma extensa discussão a respeito de versionamento de Eventos que pode ser facilmente aproveitada
[^robustness-principle]: "Princípio da Robustez". RFC 1122 - https://datatracker.ietf.org/doc/html/rfc1122#section-1.2.2
[^vnd-capes]: Até o aparecimento de um _Media Type_ especificado ou sancionado pelo NARQ que especifique categoricamente como apontar e versionar o esquema de Representação de um Recurso. Nesse ponto a **RECOMENDAÇÃO** de **PREFIRA**, tende a se tornar uma **REGRA** de **DEVE**.
[^etag]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag
