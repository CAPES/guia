# JMX Remoto no JBoss EAP 6 [^1]

## Configuração do Servidor
1. Habilitar o remoting-connector do JMX
   1. No arquivo `${JBOSS_HOME}/capes/configuration/standalone-capes.xml` modificar a o subsystem **urn:jboss:domain:jmx**

      Modificar a tag **remoting-connector** com **use-management-endpoint="true**":
```
<remoting-connector use-management-endpoint="true" />
```
2. Reiniciar as instâncias

## Conectando ao JMX
##### JCONSOLE
O JBoss recomenda utilizar utilizar um script de configuração próprio, localizado em `${JOBSS_HOME}/bin/jconsole.sh`. O ideal é utilizar uma JVM da versão de Java 7, para isso basta setar antes a variável **JAVA_HOME** antes de invocar o jconsole:
<pre>
JAVA_HOME=/usr/lib/jvm/jdk1.7.0_45 /opt/jboss/bin/jconsole.sh
</pre>

---

##### VisualVM
O JBOss indica utilizar uma configuração especial do classpath atravś do parâmetro **-cp:a** para invocar o VisuamVM. Também é o ideal utilizar o VisuamVM da JVM da versão de Java 7:
<pre>
/usr/lib/jvm/jdk1.7.0_45/bin/jvisualvm -cp:a /opt/jboss/bin/client/jboss-client.jar
</pre>

---

Tanto para o JConsole quanto para o VisualVM basta utilizar a conexão JMX no formato: `service:jmx:remoting-jmx://${IP_BIND_MANAGEMENT}:9999`
Onde o valor de **${IP_BIND_MANAGEMENT}** é determinado na subida do JBoss através do parâmetro `-Djboss.node.name=${IP_BIND_MANAGEMENT}`. Nos scripts de inicialização utilizados na CAPES ele sobe para o mesmo IP de bind da aplicação.

###### Referências
[^1]: https://docs.tibco.com/pub/mdm/9.0.0/doc/html/GUID-7E2C2C30-50EF-4903-B4FA-EEDF6F457249.html