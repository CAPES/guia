> **Formato de Logging**

# Leituras Recomendadas

- [Graylog Docs](https://docs.graylog.org/docs)
  - Graylog Extended Log Format (GELF)[^gelf]

# Introdução

Essa documentação visa estabelecer 2 formatos associados com a prática de emissão e armazenamento de logs de aplicações produzidas ou mantidas pela CGS: `CgsGelfPrecursor` e `CgsGelf`.

## Infraestrutura

### Agregador de Logs

Estamos visando que os logs das aplicações da CGS serão mantidos na ferramenta Graylog[^todo-1]. Esperamos que os logs de aplicaçõesda CGS no Graylog tenham uma estrutura básica derivada do formato padrão GELF[^gelf].

### Coletores de Logs

#### Fluent Bit

Aplicações da CGS que estejam na arquitetura conteinerizada são esperadas a seguir os preceitos do _The Twelve-Factor App_[^12-factors] [^12-factors-logs].

#### Outros

Outras arquiteturas podem ser forçadas a usarem outras soluções, mas o princípio é que os logs registrados no Graylog sejam aderentes ao formato `CgsGelf`, se possível as aplicações devem emitir em `CgsGelfPrecursor` e o coletor fica responável por transformar de um formato para outro além de enviar para o Graylog.

> #### Protocolo de transporte do GELF
> Em geral vamos esperar que o Graylog esteja escutando para receber GELF, **especificamente `CgsGelf`**, via TCP na porta 12201. Desvios disso devem ser evitados e negociados com a CGII e NARQ.

# CgsGelfPrecursor

Formato de log que as aplicações devem emitir em sua saída padrão. É o formato esperado pelos coletores de logs.

- Cada log deve ser uma _string_ de uma linha em formato JSON.


```yaml
timestamp:
  formatted: string (RFC 5424)
  unixEpoch: number (float)
level: string
thread: string
logger: string
mdc:
  spanId: string (B3)
  traceId: string (B3)
  parentId: string (B3)
  http_method: string (método HTTP)
  http_uri: string (URI)
  referrer: string
  origin_addr: string (endereço IP)
syslogLevel: integer (syslog level)
app:
  name: string
  instance: string
os:
  name: string
  version: string
  arch: string
pid: integer
message: string | objeto JSON
stacktrace: string
```

- **B3**: "BigBrotherBird" é nome original do projeto Zipkin e tem uma especificação para os campos de _Trace Id_, _Span Id_, _Parent Id_;

# CgsGelf

Mensagem GELF com campos adicionais predefinidos a ser recebido pelo Gralog.

```yaml
version: string ("1.1")
timestamp: number (float)
host: string (host de URI | endereço IP)
level: integer (syslog level)
short_message: string
full_message: string
_stacktrace: string
_trace_id: string (B3)
_span_id: string (B3)
_parent_span_id: string (B3)
_http_method: string (método HTTP)
_http_uri: string (URI)
_referrer: string
_origin_addr: string (endereço IP)
_logger_name: string
_level_name: string
_app_instance: string
_app_name: string
_pid: integer
_thread_name: string
_os_name: string
_os_version: string
_os_arch: string
```

- `host` de URI: conforme indicado no RFC 3986[^rfc-uri-host]

# Mapeamento CgsGelfPrecursor -> CgsGelf

`CgsGelfPrecursor` | `CgsGelf` | Tipo | Formato | Descrição
-|-|-|-|-
**-** | `version`<span style="color:red">*</span> | string (UTF-8) | Fixo | (`"1.1"`) Versão da especificação de GELF
`timestamp.unixEpoch` | `timestamp`<span style="color:red">*</span> | `timestamp.unixEpoch` | number | epoch UNIX | Segundos desde o "epoch UNIX" com opcionalmente decimal para milisegundos. Esperamos que os logs das aplicações da CGS tenham precisão temporal de milisegundos (3 casas decimais)
**-** | `host`<span style="color:red">*</span> | string (UTF-8) | hostname | O nome do _host_ (RFC 3986) da fonte/aplicação que emitiu a mensagem
`syslogLevel` | `level` | number | syslog level | O nível equivalente ao padrão syslog
`title` | `short_message`<span style="color:red">*</span> | string (UTF-8) | `%m%nopex`[^logback-patter-syntax] | Uma curta mensagem descritiva. Esse campo não deveria conter "novas linhas" (`\n`)
`message` | `full_message` | string (UTF-8)  | `%m%n%ex{short}`[^logback-patter-syntax] | Uma longa mensagem. Pode conter novas linhas, e, por exemplo, um `backtrace/stacktrace` (ou parte dele, há o campo `stacktrace`/`_stacktrace` para esse propósito)
`stacktrace` | `_stacktrace` | string (UTF-8) | `%ex%n{full}`[^logback-patter-syntax] | A pilha de erro, se houver
  |   |   |   |
`mdc.traceId` | `_trace_id`  | string (ASCII) | `[a-f0-9]{16,32}` | O 'trace ID' de _Tracing_ Distribuído (B3 do OpenZipkin)
`mdc.spanId` | `_span_id`  | string (ASCII) | `[a-f0-9]{16}` | O 'Span ID' de _Tracing_ Distribuído (B3 do OpenZipkin)
`mdc.parentId` | `_parent_span_id` | string (ASCII) | `[a-f0-9]{16}` | "O 'Parent Span ID' de _Tracing_ Distribuído (B3 do OpenZipkin)"
  |   |   |   |
`mdc.http_method` | `_http_method` | string (ASCII) | enum | Nome do método HTTP invocado contra a aplicação que gerou a emissão desse log
`mdc.http_uri` | `_http_uri` | string (ASCII) | URI | Endpoint solicitado que desencadeou o processamento que gerou o evento de log
`mdc.referrer` | `_referrer`  | string (ASCII) | HTTP Header Referer  | Identificação da origem que fez a chamada ao endpoint
`mdc.origin_addr` | `_origin_addr`  | string (ASCII) | IP | O "número" do IP de onde partiu a solicitação
  |   |   |   |
`logger` | `_logger_name` | string (UTF-8) | |
`level` |  `_level_name`  | string (ASCII) | enum |
  |   |   |   |
`app.instance` | `_app_instance` | string (UTF-8) |   | Nome (hostname) da instância rodando a aplicação
`app.name` | `_app_name`  | string (UTF-8) |   | Nome da aplicação
  |   |   |   |
`pid` | `_pid`  | number |   | PID do processo da aplicação rodando o processamento que gerou o evento de log
`thread` | `_thread_name`  | string (UTF-8) |   | Nome da thread rodando o processamento que gerou o evento de log
`os.name` | `_os_name`  | string (UTF-8) |   | Nome do Sistema Operacional (É o que no Java se esperaria de `System.getProperty("os.name")`)
`os.version` | `_os_version` | string (UTF-8) |   | Versão do Sistema Operacional (É o que no Java se esperaria de `System.getProperty("os.version")`)
`os.arch` | `_os_arch` | string (UTF-8) |   | Arquitetura do sistema operacional (É o que no Java se esperaria de `System.getProperty("os.arch")`)

## Mapeamento "Syslog Level" -> `CgsGelfPrecursor.level`/`CgsGelf._level_name`

Syslog | Java[^logback-2-gelf] | PHP[^psr-3] | Python
-------|-----------------------|-------------|-------
7      | `TRACE`,<br/>`DEBUG`  | `DEBUG`     |
6      | `INFO`                | `INFO`      |
5      | -                     | `NOTICE`    |
4      | `WARN`                | `WARNING`   |
3      | `ERROR`               | `ERROR`     |
2      | -                     | `CRITICAL`  |
1      | -                     | `ALERT`     |
0      | -                     | `EMERGENCY` |

# Notas & Referências

[^todo-1]: #TODO(theom): colocar o link para documentação do Graylog.
[^gelf]: Graylog Extended Log Format (GELF) - https://docs.graylog.org/docs/gelf
[^12-factors]: The Twelve-Factor App - https://12factor.net/
[^12-factors-logs]: The Twelve-Factor App - Logs - https://12factor.net/logs
[^rfc-uri-host]: RFC 3986 - host Component - https://datatracker.ietf.org/doc/html/rfc3986#section-3.2.2
[^logback-2-gelf]: https://logging.paluch.biz/syslog-level-mapping.html
[^osiegmar-logback-gelf]: Integração Logback com GELF - https://github.com/osiegmar/logback-gelf
[^logback-patter-syntax]: Sintaxe - https://logback.qos.ch/manual/layouts.html#conversionWord
[^psr-3]: PSR 3 - Logger Interface - https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-3-logger-interface.md
