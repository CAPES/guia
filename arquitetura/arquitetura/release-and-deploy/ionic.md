
# Construção & Implantação (Release & Deploy) de aplicações Ionic[^ionic-1] [^ionic-2]

O jobs de _release_ e _deploy_ de aplicações Ionic através do Jenkins da CAPES devem seguir algumas características. As configurações **Jenkins** são de responsabilidade de GCM, as de **Projeto** do time de desenvolvimento da aplicação. Essas configurações devem estar devidamente alinhadas para o correto funcionamento dos jobs.

[[_TOC_]]

# Jenkins

No Jenkins é preciso:

- Preparar os workers (slaves) que vão realizar as construções;
- Configurar os jobs

## worker (slave)

Instalamos alguns scripts no slave que roda esse tipo de construção:

- `/opt/jenkins-scripts/ionic/pom.xml`
- `/opt/jenkins-scripts/ionic/mvn-assembly.xml`
- `/opt/jenkins-scripts/ionic/build.sh`

### `/opt/jenkins-scripts/ionic/pom.xml`

É o arquivo de configuração que estabelece as configurações fundamentais para a criação de artefatos zip a partir do [plugin assembly](https://maven.apache.org/plugins/maven-assembly-plugin/index.html).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>
  <packaging>pom</packaging>

  <groupId>br.gov.capes.narq</groupId>
  <artifactId>ionic-app</artifactId>
  <version>0.0.0-SNAPSHOT</version>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-assembly-plugin</artifactId>
        <version>3.3.0</version>
        <executions>
          <execution>
            <id>zip</id>
            <phase>package</phase>
            <goals>
              <goal>single</goal>
            </goals>
            <configuration>
              <appendAssemblyId>false</appendAssemblyId>
              <outputDirectory>${PWD}/target</outputDirectory>
              <descriptors>
                <descriptor>/opt/jenkins-scripts/ionic/mvn-assembly.xml</descriptor>
              </descriptors>
              <finalName>${ionic.env}</finalName>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>

</project>
```

### `/opt/jenkins-scripts/ionic/mvn-assembly.xml`

O arquivo complementar do Maven para a criação dos arquivos zip

```xml
<assembly
    xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2 http://maven.apache.org/xsd/assembly-1.1.2.xsd">

  <id>zip</id>
  <includeBaseDirectory>false</includeBaseDirectory>

  <formats>
    <format>zip</format>
  </formats>
  <fileSets>
    <fileSet>
      <directory>${PWD}/www/${ionic.env}</directory>
      <outputDirectory></outputDirectory>
    </fileSet>
  </fileSets>
</assembly>

```

### `/opt/jenkins-scripts/ionic/build.sh`

É o arquivo que realmente controla todo ciclo de construção e upload dos artefatos maven.

```shell
#!/bin/bash

_stop=0
if [ -z "$MVN_GROUP_ID" ] ; then
    echo "Não foi informado o groupId" 1>&2 ;
    _stop=$((_stop + 1))
fi

if [ -z "$MVN_ARTIFACT_ID" ] ; then
    echo "Não foi informado o artifactId" 1>&2 ;
    _stop=$((_stop + 1))
fi

MVN_VERSION=$Versao
if [ -z "$MVN_VERSION" ] ; then
    MVN_VERSION=${env['Versao']}
    if [ -z "$MVN_VERSION" ] ; then
        echo "Não foi informado a versão" 1>&2 ;
        _stop=$((_stop + 1))
    fi
fi
if [ $_stop -gt 0 ] ; then
    exit $_stop;
fi


qtd=0
for i in $(ls env/environment.*.json) ; do
    qtd=$((qtd + 1))
done

if [ $qtd -lt 1 ] ; then
    echo "Não há arquivos de environment em env/" 1>&2 ;
    _stop=$((_stop + 1));
    exit $_stop;
fi

if [ -z "$NEXUS_RELEASE_URI" ] ; then
    NEXUS_RELEASE_URI=http://nexus.capes.gov.br/nexus/content/repositories/releases
fi

if [ -z "$NPM_SCRIPT" ] ; then
    NPM_SCRIPT=browser:build
fi

NODE_VERSION=${NODE_VERSION/v/}
if [ -z "$NODE_VERSION" ] ; then
    NODE_VERSION=v10.16.3
else
    NODE_VERSION="v$NODE_VERSION"
fi

IONIC_CLI_VERSION=${IONIC_CLI_VERSION/v/}
if [ -z "$IONIC_CLI_VERSION" ] ; then
    IONIC_CLI_VERSION=6.12.2
fi

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"

nvm install $NODE_VERSION && npm install -g @ionic/cli@$IONIC_CLI_VERSION
if [ $? -ne 0 ]  ; then
    _stop=$((_stop + 1));
    exit $_stop;
fi

mvn --global-settings /opt/apache-maven-3.0.5/conf/settings.xml --settings /opt/apache-maven-3.0.5/conf/settings.xml --file /opt/jenkins-scripts/ionic/pom.xml clean ;
if [ $? -ne 0 ]  ; then
    _stop=$((_stop + 1)) ;
    exit $_stop ;
fi


count=0
for i in $(ls env/environment.*.json) ; do
    _env=${i/env\/environment./} ;
    _env=${_env/.json/} ;
    npm install && npm --verbose run $NPM_SCRIPT:$_env --wwwDir www/$_env &&
    mvn --global-settings /opt/apache-maven-3.0.5/conf/settings.xml --settings /opt/apache-maven-3.0.5/conf/settings.xml --file /opt/jenkins-scripts/ionic/pom.xml -Dionic.env=$_env package &&
    count=$((count + 1)) ;
done


if [ $count -ne $qtd ] ; then
    echo "Nem todos os classifiers foram gerados. count=$count, qtd=$qtd" 1>&2 ;
    _stop=$((_stop + 1)) ;
    exit $_stop ;
fi


function _mvn_deploy_file() {
    mvn --global-settings /opt/apache-maven-3.0.5/conf/settings.xml --settings /opt/apache-maven-3.0.5/conf/settings.xml deploy:deploy-file -DgroupId=$MVN_GROUP_ID -DartifactId=$MVN_ARTIFACT_ID -Dversion=$MVN_VERSION -Dclassifier=${_env} -Dfile=target/${_env}.zip -DrepositoryId=releases -Durl="$NEXUS_RELEASE_URI" -DgeneratePom=$MVN_GEN_POM -Dpackaging=zip
}


MVN_GEN_POM=true
if [ -f "target/prod.zip" ] ; then
    _env=prod
    _mvn_deploy_file &&
    MVN_GEN_POM=false
fi
if [ $? -ne 0 ]  ; then
    _stop=$((_stop + 1));
    exit $_stop ;
fi


for i in $(ls env/environment.*.json) ; do
    _env=${i/env\/environment./} ;
    _env=${_env/.json/} ;
    if [ $_env != "prod" ] ; then
        _mvn_deploy_file &&
        MVN_GEN_POM=false ;
        if [ $? -ne 0 ]  ; then
            _stop=$((_stop + 1));
            exit $_stop ;
        fi
    fi
done
```

## Jobs

### Release

Com os arquivos (`pom.xml`, `mvn-assembly.xml` e `build.sh`) devidamente instalados é possível crair jobs que utilizem essa infraestrutura.

Caso seja necessário configurar um novo job de _release_ de uma aplicação Ionic é possível se basear no job [IonicWebAppReleaseJob](http://jenkins.capes.gov.br/job/IonicWebAppReleaseJob/).

Como o job de _release_ se baseia no script `/opt/jenkins-scripts/ionic/build.sh` diversas configurações podem ser utilizadas, inclusive sendo parâmetros de construção que podem ser informados pelos usuários:

- `MVN_GROUP_ID`: Define o groupId[^mvn-relationships] do artefato final
  - **Sem default**
- `MVN_ARTIFACT_ID`: Define o artifactId[^mvn-relationships] do artefato final
  - **Sem default**
- `Versao` ou `env['Versao']`: Define a versão da _release_ e do artefato final
  - **Sem default**
  - O valor é reatribuído para `MVN_VERSION`
- `NEXUS_RELEASE_URI`: Define a URI base do reposítório onde o artefato será armazenado
  - **Default**: `http://nexus.capes.gov.br/nexus/content/repositories/releases`
- `NPM_SCRIPT`: Define o prefixo de scripts do npm que serão utilizados para construir os artefatos
  - **Default**: `browser:build`
- `NODE_VERSION`: Versão do **Node.js** a ser utilizada (e por consequência do **npm**)
  - **Default**: `v10.16.3`
  - Pode ser informado com ou sem o `v` de prefixo (ex.: `v10.16.3` ou `10.16.3`)
- `IONIC_CLI_VERSION`: Versão da cli do Ionic a ser utilizada
  - **Default**: `6.12.2`
  - Utiliza uma instalação "global" de `@ionic/cli` naquela versão de **Node.js**, controlada por **nvm**

Os valores **Default** servem de alternativa caso os valores resolvam para vazio por alguma razão. Valores **Sem default** são obrigatórios e tem de ser informados de alguma maneira.

### Deploy (Implantação)

Cada job de _deploy_ basicamente pode resolver um dos artefatos criados durante o job de _release_ (cada ambiente deve ter um **classifier**[^mvn-relationships] correspondente criado) e descompactar o conteúdo do arquivo baixado na devida localização.

Caso seja necessário configurar um novo job de _deploy_ de uma aplicação Ionic é possível se basear no job [IonicWebAppDeploy](http://jenkins.capes.gov.br/job/IonicWebAppDeploy/).

# Projeto

Para que a construção funcione corretamente o projeto tem de seguir alguns padrões.

## package.json

Para cada arquivo `env/environment.*.json` deve haver no `package.json` um script `browser:build:$env`[^build-prefix] onde `$env` é cada valor da expansão de `*`. Exemplo:

O seguinte conjunto de arquivos
```
./env/
├── environment.dev.json
├── environment.hmg.json
├── environment.local.json
├── environment.prod.json
└── environment.teste.json
```
Vai _tentar_ criar 5 **classifiers**:
- **dev**
- **hmg**
- local
- **prod**
- **teste**

Provavelmente apenas os **classifiers** em negrito terão jobs de implantação (_deploy_) configurados no Jenkins.

Para que o job funcione é preciso algo como o a seguir no arquivo `package.json`:

```json
"scripts": {
    "ionic": "ionic",
    "build": "ionic-app-scripts build",
    "browser:build:local": " cross-env ENV=local ionic build ",
    "browser:build:prod": " cross-env ENV=prod ionic build ",
    "browser:build:dev": " cross-env ENV=dev ionic build ",
    "browser:build:teste": " cross-env ENV=teste ionic build ",
    "browser:build:hmg": " cross-env ENV=hmg ionic build ",
  },
```

> Mesmo que o **classifier** _local_ nunca seja utilizado por um job de _deploy_, se o arquivo `env/environment.local.json` existir, é necessário que exista um script correspondente no `package.json` ou a construção vai falhar.

# Notas e Referências

[^ionic-1]: https://ionicframework.com/
[^ionic-2]: Ênfase em construções web
[^mvn-relationships]: https://maven.apache.org/pom.html#pom-relationships
[^build-prefix]: O prefixo `browser:build` pode ser alterado no job pelo valor que preenche a variável `NPM_SCRIPT`, alinhar com GCM caso necessário
