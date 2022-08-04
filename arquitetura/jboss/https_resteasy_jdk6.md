# Usando BouncyCastle para uma consulta HTTPS com RestEasy no JDK 6

Solução (JCE do BouncyCastle)

Esta é uma versão simplificada do código que estou usando para realizar solicitações HTTPS com RestEasy. No projeto estou usando JBOSS AS 5.2 e JDK 1.6.0_45.

```
	ResteasyClient client = new ResteasyClientBuilder().build();		
		this.instancia = client.target(<url>)
				.proxy(<clase de ponto entrada>);
```
Isso costumava funcionar bem para conectar ao nosso servidor (e ainda funciona se eu usar http como protocolo) até recentemente, quando algumas atualizações de segurança foram feitas.

Agora ele está apresentando os erros " _Could not generate DH keypair_ " e " _Prime size must be multiple of 64,
and can only range from 512 to 1024 (inclusive)_ " mencionados nesta pergunta:

```
Java: Why does SSL handshake give 'Could not generate DH keypair' exception?
```
Este erro ocorre porque o site SFTP está usando uma chave SSL com comprimento de 2048, no Java 7
oferece suporte a chaves de até 1024 de comprimento, mas para corrigir e atender a solicitação atualize para
Java 8 ou superior (versão estável).

Caso não queira atualizar a versão, você poder usar a implementação JCE do BouncyCastle.

## Solução (JCE do BouncyCastle)

Precisamos adicione a linha abaixo ao arquivo java.security, que geralmente está localizado em
$JAVA_HOME/jre/lib/security:

```
Exemplo:
#
# List of providers and their preference orders (see above):
#
security.provider.1=sun.security.provider.Sun
security.provider.2=org.bouncycastle.jce.provider.BouncyCastleProvider
security.provider.3=sun.security.rsa.SunRsaSign
security.provider.4=com.sun.net.ssl.internal.ssl.Provider
security.provider.5=com.sun.crypto.provider.SunJCE
security.provider.6=sun.security.jgss.SunProvider
security.provider.7=com.sun.security.sasl.Provider
security.provider.8=org.jcp.xml.dsig.internal.dom.XMLDSigRI
security.provider.9=sun.security.smartcardio.SunPCSC
security.provider.10=sun.security.mscapi.SunMSCAPI
```
Pronto, finalizamos a configuração do JCE do BouncyCastle, RestEasy e Commons HttpClient. Antes de começarmos ajustar o código, precisamos adicionar as dependências necessárias ao nosso arquivo pom.xml:

```
<dependency>
			<groupId>org.jboss.resteasy</groupId>
			<artifactId>resteasy-jaxrs</artifactId>
			<version>3.0.11.Final</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.jboss.resteasy</groupId>
			<artifactId>resteasy-client</artifactId>
			<version>3.0.11.Final</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpcore</artifactId>
			<version>4.4.1</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpclient</artifactId>
			<version>4.4.1</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.bouncycastle</groupId>
			<artifactId>bcprov-jdk15to18</artifactId>
			<version>1.64</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.bouncycastle</groupId>
			<artifactId>bctls-jdk15to18</artifactId>
			<version>1.64</version>
			<scope>provided</scope>
		</dependency>
```

PS:  Essas versões são as últimas compatíveis com o JAVA 1.6 e que possuem todas as classes necessárias para a implementação. Além disso, quaisquer outras versões das bibliotecas acima devem ser excluídas das dependências no pom. No eclipse isso é facilmente realizado ao se utilizar a ferramenta "Dependency Hierarchy", que permite visualizar toda a hierarquia dos jars.

Feito, agora preciso usar o Bouncy Castle porque o Java 1.6 não oferece suporte a TLS1.1 / TLS1.2 portanto, a primeira etapa é estender a classe SSLSocketFactory.

Segue o exemplo abaixo:

```
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.InetAddress;
import java.net.InetSocketAddress;
import java.net.Socket;
import java.net.UnknownHostException;
import java.security.Principal;
import java.security.SecureRandom;
import java.security.Security;
import java.security.cert.CertificateException;
import java.security.cert.CertificateFactory;
import java.util.Hashtable;
import java.util.LinkedList;
import java.util.List;

import javax.net.ssl.HandshakeCompletedEvent;
import javax.net.ssl.HandshakeCompletedListener;
import javax.net.ssl.SSLPeerUnverifiedException;
import javax.net.ssl.SSLSession;
import javax.net.ssl.SSLSessionContext;
import javax.net.ssl.SSLSocket;
import javax.net.ssl.SSLSocketFactory;
import javax.security.cert.X509Certificate;

import org.bouncycastle.crypto.tls.Certificate;
import org.bouncycastle.crypto.tls.CertificateRequest;
import org.bouncycastle.crypto.tls.DefaultTlsClient;
import org.bouncycastle.crypto.tls.ExtensionType;
import org.bouncycastle.crypto.tls.TlsAuthentication;
import org.bouncycastle.crypto.tls.TlsClientProtocol;
import org.bouncycastle.crypto.tls.TlsCredentials;
import org.bouncycastle.jce.provider.BouncyCastleProvider;

public class TSLSocketConnectionFactory extends SSLSocketFactory {

	//////////////////////////////////////////////////////////////////////////////////////////////////////////////
	// Adding Custom BouncyCastleProvider
	///////////////////////////////////////////////////////////////////////////////////////////////////////////////
	static {
		if (Security.getProvider(BouncyCastleProvider.PROVIDER_NAME) == null)
			Security.addProvider(new BouncyCastleProvider());
	}

	//////////////////////////////////////////////////////////////////////////////////////////////////////////////
	// HANDSHAKE LISTENER
	///////////////////////////////////////////////////////////////////////////////////////////////////////////////
	public class TLSHandshakeListener implements HandshakeCompletedListener {
		@Override
		public void handshakeCompleted(HandshakeCompletedEvent event) {

		}
	}

	//////////////////////////////////////////////////////////////////////////////////////////////////////////////
	// SECURE RANDOM
	///////////////////////////////////////////////////////////////////////////////////////////////////////////////
	private SecureRandom _secureRandom = new SecureRandom();

	//////////////////////////////////////////////////////////////////////////////////////////////////////////////
	// Adding Custom BouncyCastleProvider
	///////////////////////////////////////////////////////////////////////////////////////////////////////////////
	@Override
	public Socket createSocket(Socket socket, final String host, int port, boolean arg3) throws IOException {
		if (socket == null) {
			socket = new Socket();
		}
		if (!socket.isConnected()) {
			socket.connect(new InetSocketAddress(host, port));
		}

		final TlsClientProtocol tlsClientProtocol = new TlsClientProtocol(socket.getInputStream(),
				socket.getOutputStream(), _secureRandom);
		return _createSSLSocket(host, tlsClientProtocol);

	}

	//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
	// SOCKET FACTORY METHODS
	//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
	@Override
	public String[] getDefaultCipherSuites() {
		return null;
	}

	@Override
	public String[] getSupportedCipherSuites() {
		return null;
	}

	@Override
	public Socket createSocket(String host, int port) throws IOException, UnknownHostException {
		return null;
	}

	@Override
	public Socket createSocket(InetAddress host, int port) throws IOException {
		return null;
	}

	@Override
	public Socket createSocket(String host, int port, InetAddress localHost, int localPort)
			throws IOException, UnknownHostException {
		return null;
	}

	@Override
	public Socket createSocket(InetAddress address, int port, InetAddress localAddress, int localPort)
			throws IOException {
		return null;
	}

	//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
	// SOCKET CREATION
	//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

	private SSLSocket _createSSLSocket(final String host, final TlsClientProtocol tlsClientProtocol) {
		return new SSLSocket() {
			private java.security.cert.Certificate[] peertCerts;

			@Override
			public InputStream getInputStream() throws IOException {
				return tlsClientProtocol.getInputStream();
			}

			@Override
			public OutputStream getOutputStream() throws IOException {
				return tlsClientProtocol.getOutputStream();
			}

			@Override
			public synchronized void close() throws IOException {
				tlsClientProtocol.close();
			}

			@Override
			public void addHandshakeCompletedListener(HandshakeCompletedListener arg0) {

			}

			@Override
			public boolean getEnableSessionCreation() {
				return false;
			}

			@Override
			public String[] getEnabledCipherSuites() {
				return new String[] { "" };
			}

			@Override
			public String[] getEnabledProtocols() {
				return new String[] { "" };
			}

			@Override
			public boolean getNeedClientAuth() {
				return false;
			}

			@Override
			public SSLSession getSession() {
				return new SSLSession() {

					@Override
					public int getApplicationBufferSize() {
						return 0;
					}

					@Override
					public String getCipherSuite() {
						return "";
					}

					@Override
					public long getCreationTime() {
						throw new UnsupportedOperationException();
					}

					@Override
					public byte[] getId() {
						throw new UnsupportedOperationException();
					}

					@Override
					public long getLastAccessedTime() {
						throw new UnsupportedOperationException();
					}

					@Override
					public java.security.cert.Certificate[] getLocalCertificates() {
						throw new UnsupportedOperationException();
					}

					@Override
					public Principal getLocalPrincipal() {
						return null;
					}

					@Override
					public int getPacketBufferSize() {
						throw new UnsupportedOperationException();
					}

					@Override
					public X509Certificate[] getPeerCertificateChain() throws SSLPeerUnverifiedException {
						// TODO Auto-generated method stub
						return null;
					}

					@Override
					public java.security.cert.Certificate[] getPeerCertificates() throws SSLPeerUnverifiedException {
						return peertCerts;
					}

					@Override
					public String getPeerHost() {
						throw new UnsupportedOperationException();
					}

					@Override
					public int getPeerPort() {
						return 0;
					}

					@Override
					public Principal getPeerPrincipal() throws SSLPeerUnverifiedException {
						return null;
						// throw new UnsupportedOperationException();

					}

					@Override
					public String getProtocol() {
						return null;
					}

					@Override
					public SSLSessionContext getSessionContext() {
						throw new UnsupportedOperationException();
					}

					@Override
					public Object getValue(String arg0) {
						throw new UnsupportedOperationException();
					}

					@Override
					public String[] getValueNames() {
						throw new UnsupportedOperationException();
					}

					@Override
					public void invalidate() {
						throw new UnsupportedOperationException();

					}

					@Override
					public boolean isValid() {
						throw new UnsupportedOperationException();
					}

					@Override
					public void putValue(String arg0, Object arg1) {
						throw new UnsupportedOperationException();

					}

					@Override
					public void removeValue(String arg0) {
						throw new UnsupportedOperationException();

					}

				};
			}

			@Override
			public String[] getSupportedProtocols() {
				return null;
			}

			@Override
			public boolean getUseClientMode() {
				return false;
			}

			@Override
			public boolean getWantClientAuth() {

				return false;
			}

			@Override
			public void removeHandshakeCompletedListener(HandshakeCompletedListener arg0) {

			}

			@Override
			public void setEnableSessionCreation(boolean arg0) {

			}

			@Override
			public void setEnabledCipherSuites(String[] arg0) {

			}

			@Override
			public void setEnabledProtocols(String[] arg0) {

			}

			@Override
			public void setNeedClientAuth(boolean arg0) {

			}

			@Override
			public void setUseClientMode(boolean arg0) {

			}

			@Override
			public void setWantClientAuth(boolean arg0) {

			}

			@Override
			public String[] getSupportedCipherSuites() {
				return null;
			}

			@Override
			public void startHandshake() throws IOException {
				tlsClientProtocol.connect(new DefaultTlsClient() {
					@Override
					public Hashtable<Integer, byte[]> getClientExtensions() throws IOException {
						Hashtable<Integer, byte[]> clientExtensions = super.getClientExtensions();
						if (clientExtensions == null) {
							clientExtensions = new Hashtable<Integer, byte[]>();
						}

						// Add host_name
						byte[] host_name = host.getBytes();

						final ByteArrayOutputStream baos = new ByteArrayOutputStream();
						final DataOutputStream dos = new DataOutputStream(baos);
						dos.writeShort(host_name.length + 3); // entry size
						dos.writeByte(0); // name type = hostname
						dos.writeShort(host_name.length);
						dos.write(host_name);
						dos.close();
						clientExtensions.put(ExtensionType.server_name, baos.toByteArray());
						return clientExtensions;
					}

					@Override
					public TlsAuthentication getAuthentication() throws IOException {
						return new TlsAuthentication() {

							@Override
							public void notifyServerCertificate(Certificate serverCertificate) throws IOException {

								try {
									CertificateFactory cf = CertificateFactory.getInstance("X.509");
									List<java.security.cert.Certificate> certs = new LinkedList<java.security.cert.Certificate>();
									for (org.bouncycastle.asn1.x509.Certificate c : serverCertificate
											.getCertificateList()) {
										certs.add(cf.generateCertificate(new ByteArrayInputStream(c.getEncoded())));
									}
									peertCerts = certs.toArray(new java.security.cert.Certificate[0]);
								} catch (CertificateException e) {
									System.out.println("Failed to cache server certs" + e);
									throw new IOException(e);
								}

							}

							@Override
							public TlsCredentials getClientCredentials(CertificateRequest arg0) throws IOException {
								return null;
							}

						};

					}

				});

			}

		};// Socket

	}
}

```
Configuração efetuada na inicialização e utilização do serviço:

```
import javax.ws.rs.NotFoundException;

import org.apache.http.client.HttpClient;
import org.apache.http.conn.ssl.DefaultHostnameVerifier;
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
import org.apache.http.impl.client.HttpClientBuilder;
import org.jboss.resteasy.client.jaxrs.ResteasyClient;
import org.jboss.resteasy.client.jaxrs.ResteasyClientBuilder;
import org.jboss.resteasy.client.jaxrs.engines.ApacheHttpClient4Engine;
import org.jboss.resteasy.plugins.providers.jackson.CustomResteasyJacksonJsonProvider;

	private <classe de ponto de entrada> getServico(String authToken) {
		ApacheHttpClient4Engine engine = new ApacheHttpClient4Engine(getHttpClient());

		ResteasyClient client = new ResteasyClientBuilder().httpEngine(engine).build();
		client.register(new AuthHeadersRequestFilter(authToken));
		client.register(CustomResteasyJacksonJsonProvider.class);
		return client.target(<url>).proxy(<classe de ponto de entrada>);
	}
	private static HttpClient getHttpClient() {

		try {
			SSLConnectionSocketFactory socketFactory = new SSLConnectionSocketFactory(new TSLSocketConnectionFactory(),
					new String[] { "TLSv1.2" }, null, new DefaultHostnameVerifier());

			HttpClient httpClient = HttpClientBuilder.create().setSSLSocketFactory(socketFactory).build();

			return httpClient;

		} catch (Exception e) {
			LOGGER.error("Erro ao tentar criar cliente com TLS 1.2 e BouncyCastle", e);
			
			return HttpClientBuilder.create().build();
		}
	}
```

Classe que registra o token no "bearer" de autorização (quando o serviço já está integrado ao SSO)
```
import java.io.IOException;

import javax.ws.rs.client.ClientRequestContext;
import javax.ws.rs.client.ClientRequestFilter;
import javax.ws.rs.core.HttpHeaders;

public class AuthHeadersRequestFilter implements ClientRequestFilter {

	private final String authToken;

	public AuthHeadersRequestFilter(String authToken) {
		this.authToken = authToken;
	}

	@Override
	public void filter(ClientRequestContext requestContext) throws IOException {
		requestContext.getHeaders().add(HttpHeaders.AUTHORIZATION, "Bearer " + authToken);
	}

}

```

Classe que faz o parse do json e mapeia para classe java com os dados do serviço
```
import java.io.IOException;
import java.io.OutputStream;
import java.lang.annotation.Annotation;
import java.lang.reflect.Type;

import javax.ws.rs.Consumes;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.MultivaluedMap;
import javax.ws.rs.ext.MessageBodyReader;
import javax.ws.rs.ext.MessageBodyWriter;
import javax.ws.rs.ext.Provider;

import org.jboss.resteasy.annotations.ConfigSerialization;

import com.fasterxml.jackson.core.JsonEncoding;
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.ObjectWriter;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.databind.ser.FilterProvider;
import com.fasterxml.jackson.databind.ser.impl.SimpleFilterProvider;
import com.fasterxml.jackson.datatype.joda.JodaModule;
import com.fasterxml.jackson.jaxrs.json.JacksonJsonProvider;
import com.fasterxml.jackson.jaxrs.json.annotation.EndpointConfig;
import com.fasterxml.jackson.jaxrs.json.util.AnnotationBundleKey;

import br.gov.capes.cadastropessoas.ServicoCadastroPessoas;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Provider
@Consumes({ "application/json", "text/json" })
@Produces({ "application/json", "text/json" })
public class CustomResteasyJacksonJsonProvider extends JacksonJsonProvider implements MessageBodyReader<Object>, MessageBodyWriter<Object> {

	private static Logger LOG = LoggerFactory.getLogger(CustomResteasyJacksonJsonProvider.class);

	@Override
	public ObjectMapper locateMapper(Class<?> type, MediaType mediaType) {
		ObjectMapper mapper = new ObjectMapper();
		mapper.registerModule(new JodaModule());
		mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
		mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
		mapper.enable(DeserializationFeature.ACCEPT_SINGLE_VALUE_AS_ARRAY);
		mapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
		mapper.enable(DeserializationFeature.USE_JAVA_ARRAY_FOR_JSON_ARRAY);
		mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);

		return mapper;
	}

	@Override
	public void writeTo(Object value, Class<?> type, Type genericType, Annotation[] annotations, MediaType mediaType, MultivaluedMap<String, Object> httpHeaders, OutputStream entityStream) {
		AnnotationBundleKey key = new AnnotationBundleKey(annotations);
		EndpointConfig endpoint;
		synchronized (_writers) {
			endpoint = _writers.get(key);
		}
		ObjectMapper mapper = null;
		// not yet resolved (or not cached any more)? Resolve!
		if (endpoint == null) {
			mapper = locateMapper(type, mediaType);

			endpoint = EndpointConfig.forWriting(mapper, annotations, this._jsonpFunctionName);
			// and cache for future reuse
			synchronized (_writers) {
				_writers.put(key.immutableKey(), endpoint);
			}
		}

		ObjectWriter writer = endpoint.getWriter();

		/*
		 * 27-Feb-2009, tatu: Where can we find desired encoding? Within HTTP
		 * headers?
		 */
		
		OutputStreamWrapper saida = new OutputStreamWrapper(entityStream);

		JsonEncoding enc = findEncoding(mediaType, httpHeaders);
		@SuppressWarnings("deprecation")
		JsonGenerator jsonGenerator = null;

		try {
			jsonGenerator = writer.getJsonFactory().createJsonGenerator(saida, enc);

			// Want indentation?
			if (writer.isEnabled(SerializationFeature.INDENT_OUTPUT)) {
				jsonGenerator.useDefaultPrettyPrinter();
			}
			// 04-Mar-2010, tatu: How about type we were given? (if any)
			JavaType rootType = null;

			/*
			 * 10-Jan-2011, tatu: as per [JACKSON-456], it's not safe to just
			 * force root type since it prevents polymorphic type serialization.
			 * Since we really just need this for generics, let's only use
			 * generic type if it's truly generic.
			 */
			// generic types are other impls of 'java.lang.reflect.Type'
			if (genericType != null && value != null && genericType.getClass() != Class.class) {
				/*
				 * This is still not exactly right; should root type be further
				 * specialized with 'value.getClass()'? Let's see how well this
				 * works before trying to come up with more complete solution.
				 */
				rootType = writer.getTypeFactory().constructType(genericType);
				/*
				 * 26-Feb-2011, tatu: To help with [JACKSON-518], we better
				 * recognize cases where type degenerates back into
				 * "Object.class" (as is the case with plain TypeVariable, for
				 * example), and not use that.
				 */
				if (rootType.getRawClass() == Object.class) {
					rootType = null;
				}
			}

			// Most of the configuration now handled through EndpointConfig,
			// ObjectWriter
			// but we may need to force root type:
			if (rootType != null) {
				writer = writer.withType(rootType);
			}
			// and finally, JSONP wrapping, if any:
			Object jsonWrapper = endpoint.applyJSONP(value);

			ConfigSerialization configSerialization = null;
			if (annotations != null && annotations.length > 0) {
				configSerialization = findAnnotation(ConfigSerialization.class, annotations);
			}

			if (configSerialization != null) {
				mapper.addMixInAnnotations(Object.class, PropertyFilterMixIn.class);

				FilterProvider filters = new SimpleFilterProvider().addFilter("filter properties by name", new IncludeExcludeFieldFilter(configSerialization));
				writer = writer.with(filters);
			}

			writer.writeValue(jsonGenerator, jsonWrapper);

			StringBuffer sb = ServicoCadastroPessoas.userThreadLocal.get();

			if(sb == null) {
				sb = new StringBuffer();
			}

			sb.append(jsonWrapper.getClass().getSimpleName()+saida.toRest());

			ServicoCadastroPessoas.userThreadLocal.set(sb);
		}catch (IOException e){
			LOG.error(e.getMessage(), e);
		} finally {

			if (saida != null) {
				try {
					saida.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			if(jsonGenerator != null){
				try {
					jsonGenerator.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
	}

	@SuppressWarnings("unchecked")
	private <T> T findAnnotation(Class<?> class1, Annotation[] annotations) {
		for (Annotation ann : annotations) {
			if (ann.annotationType().equals(class1)) {
				return (T) ann;
			}	
		}	

		return null;
	}
}

```


fontes:
https://stackoverflow.com/questions/6851461/why-does-ssl-handshake-give-could-not-generate-dh-keypair-exception
https://stackoverflow.com/questions/32230342/getting-could-not-generate-dh-keypair-exception
https://stackoverflow.com/questions/40536615/could-not-generate-dh-keypair-in-java-
https://access.redhat.com/solutions/
https://qastack.com.br/programming/6851461/why-does-ssl-handshake-give-could-not-generate-dh-keypair-exception
https://access.redhat.com/documentation/pt-br/red_hat_jboss_enterprise_application_platform/6.4/html/migration_guide/chap-migrate_your_application
https://developer.jboss.org/thread/
https://access.redhat.com/solutions/
https://stackoverflow.com/questions/29405727/java-lang-noclassdeffounderror-org-bouncycastle-jce-provider-bouncycastleprovid

Autor: Celso Queiroz


