## Introdução

As aplicações implantadas no Openshift por padrão estão configuradas para salvar o log do STDOUT na pilha EFK (Elasticsearch, Fluentd, Kibana). Atualmente esses logs possuem a validade de 1 ano.
Além dos logs salvos por padrão, as aplicações podem ser customizadas para enviar logs no formato GELF para a instância do Graylog de produção.

## Customização dos logs

Na aplicação InfoPes, foi feita uma customização para envio de logs específicos de auditoria ao Graylog. A configuração é feita através do arquivo logback.xml.
Cada ambiente dht está configurado com uma cópia do arquivo, customizando os caminhos de servidor e porta do graylog.

```logback.xml
<configuration>
	<appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender" >
		<layout class="ch.qos.logback.classic.PatternLayout">
			<Pattern>
				%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n
			</Pattern>
		</layout>
	</appender>

	<appender name="ASYNC_CONSOLE" class="ch.qos.logback.classic.AsyncAppender">
		<appender-ref ref="CONSOLE" />
	</appender>
	
	<appender name="GELF" class="de.siegmar.logbackgelf.GelfTcpAppender">
		<graylogHost>${GRAYLOG_HOST:-siem.hom.capes.gov.br}</graylogHost>
		<graylogPort>${GRAYLOG_PORT:-12205}</graylogPort>
		<connectTimeout>15000</connectTimeout>
		<reconnectInterval>300</reconnectInterval>
		<maxRetries>2</maxRetries>
		<retryDelay>3000</retryDelay>
		<poolSize>2</poolSize>
		<poolMaxWaitTime>5000</poolMaxWaitTime>
		<encoder class="de.siegmar.logbackgelf.GelfEncoder">
			<includeRawMessage>false</includeRawMessage>
			<includeMarker>true</includeMarker>
			<includeMdcData>true</includeMdcData>
			<includeCallerData>false</includeCallerData>
			<includeRootCauseData>false</includeRootCauseData>
			<includeLevelName>false</includeLevelName>
			<shortPatternLayout class="ch.qos.logback.classic.PatternLayout">
				<pattern>%m%nopex</pattern>
			</shortPatternLayout>
			<fullPatternLayout class="ch.qos.logback.classic.PatternLayout">
				<pattern>%m%n</pattern>
			</fullPatternLayout>
			<numbersAsString>false</numbersAsString>
			<staticField>app_name:infopes</staticField>
			<staticField>environment:localhost</staticField>
			<staticField>os_arch:${os.arch}</staticField>
			<staticField>os_name:${os.name}</staticField>
			<staticField>os_version:${os.version}</staticField>
		</encoder>
	</appender>
	
	<appender name="ASYNC_GELF" class="ch.qos.logback.classic.AsyncAppender">
    	<appender-ref ref="GELF" />
    </appender>
    
    <!-- Logs de acesso não são enviados para o stdout, apenas para o graylog -->
	<logger name="infopes.registroacesso" additivity="false">
		<appender-ref ref="ASYNC_GELF" />
	</logger>

	<root level="INFO">
		<appender-ref ref="ASYNC_CONSOLE" />
	</root>
</configuration>
```

## Log em arquivo

Além do formato GELF, a configuração do logback escreve os logs em arquivo. O formato foi padronizado para montagem no diretório /sistema/logs. Os logs em arquivo devem ser montados em um PersistentVolume próprio do namespace. A montagem do volume é feita via pipeline da aplicação.

## Débitos técnicos

As dependências maven do logback deverão ser incorporadas na próxima versão planejada da pilha Spring Boot.


