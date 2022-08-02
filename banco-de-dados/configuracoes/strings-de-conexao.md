# Índice
1. [Introdução](#Introdução)
1. [Oracle](#oracle)
1. [Postgres](#postgres)
1. [Sybase](#sybase)

<br>

# Introdução

Este documento destina-se a documentação das strings de conexões dos SGBDs utilizados na CAPES.

> OBS. A string de conexão JDBC está na sua forma básica e pode necessitar de ajustes a depender da aplicação que a utilizará.

<br><br>

# ORACLE

## CONEXÕES TNS ORACLE

### PRODUÇÃO

#### ADDCAPES

```bash
ADDCAPES=
  (DESCRIPTION=
    (ADDRESS=
      (PROTOCOL=TCP)
      (HOST=oraclesrv03.capes.gov.br)
      (PORT=1521)
    )
    (CONNECT_DATA=
      (SERVICE_NAME=addcapes_dr)
    )
  )
```

#### PROD

```bash
PROD=
  (DESCRIPTION=
    (ADDRESS=
      (PROTOCOL=TCP)
      (HOST=oraclesrv01.capes.gov.br)
      (PORT=1521)
    )
    (CONNECT_DATA=
      (SERVICE_NAME=prod_dr)
    )
  )

```

#### PROTECT
 
```bash
PROTECT=
  (DESCRIPTION=
    (ADDRESS=
      (PROTOCOL=TCP)
      (HOST=oraclesrv02.capes.gov.br)
      (PORT=1521)
    )
    (CONNECT_DATA=
      (SERVICE_NAME=protect_dr)
    )
  )
```
### DATA GUARD ESPELHO PRODUÇÃO

### PRODDG

```bash
PRODDG=
  (DESCRIPTION=
    (ADDRESS=
      (PROTOCOL=TCP)
      (HOST=oracledgsrv01.capes.gov.br)
      (PORT=1521)
    )
    (CONNECT_DATA=
      (SERVICE_NAME=proddg)
    )
  )
```
### PROTECTDG
```bash
PROTECTDG=
  (DESCRIPTION=
    (ADDRESS=
      (PROTOCOL=TCP)
      (HOST=oracledgsrv02.capes.gov.br)
      (PORT=1521)
    )
    (CONNECT_DATA=
      (SERVICE_NAME=protectdg)
    )
  )
 ```
### ADDCAPDG

```bash
ADDCAPDG=
  (DESCRIPTION=
    (ADDRESS=
      (PROTOCOL=TCP)
      (HOST=oracledgsrv03.capes.gov.br)
      (PORT=1521)
    )
    (CONNECT_DATA=
      (SERVICE_NAME=addcapdg)
    )
  )
```
### DESENVOLVIMENTO/HOMOLOGAÇÃO/TESTE/PRÉ-PRODUÇÃO
 
#### ADDCAPD
 
```bash
ADDCAPD=
  (DESCRIPTION=
    (ADDRESS=
      (PROTOCOL=TCP)
      (HOST=oracledhtsrv06.hom.capes.gov.br)
      (PORT=1521)
    )
    (CONNECT_DATA=
      (SERVICE_NAME=addcapd_dr)
    )
  )
```
#### PROTECTDG
```bash
PROTECTDG=
  (DESCRIPTION=
    (ADDRESS=
      (PROTOCOL=TCP)
      (HOST=oracledgsrv02.capes.gov.br)
      (PORT=1521)
    )
    (CONNECT_DATA=
      (SERVICE_NAME=protectdg)
    )
  )
``` 
#### DESENVOLVIMENTO
 
```bash
DSNV=
  (DESCRIPTION=
    (ADDRESS=
      (PROTOCOL=TCP)
      (HOST=oracledhtsrv01.hom.capes.gov.br)
      (PORT=1521)
    )
    (CONNECT_DATA=
      (SERVICE_NAME=dsnv_dr)
    )
  )
```

#### HOMOLOGAÇÃO 

```bash
HOM=
  (DESCRIPTION=
    (ADDRESS=
      (PROTOCOL=TCP)
      (HOST=oracledhtsrv02.hom.capes.gov.br)
      (PORT=1521)
    )
    (CONNECT_DATA=
      (SERVICE_NAME=hom_dr)
    )
  )
```
 
#### PEND

```bash
PEND=
  (DESCRIPTION=
    (ADDRESS=
      (PROTOCOL=TCP)
      (HOST=oracledhtsrv03.hom.capes.gov.br)
      (PORT=1521)
    )
    (CONNECT_DATA=
      (SERVICE_NAME=pend_dr)
    )
  )
```
  
 
#### PRÉ-PRODUÇÃO

```bash
PREPROD=
  (DESCRIPTION=
    (ADDRESS=
      (PROTOCOL=TCP)
      (HOST=oracledhtsrv04.hom.capes.gov.br)
      (PORT=1521)
    )
    (CONNECT_DATA=
      (SERVICE_NAME=preprod_dr)
    )
  )
```

#### TESTE  

```bash
TESTE=
  (DESCRIPTION=
    (ADDRESS=
      (PROTOCOL=TCP)
      (HOST=oracledhtsrv05.hom.capes.gov.br)
      (PORT=1521)
    )
    (CONNECT_DATA=
      (SERVICE_NAME=teste_dr)
    )
  )
```
 
## CONEXÕES JDBC ORACLE
 
### PRODUÇÃO

#### ADDCAPES
 
```bash
jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=oraclesrv03.capes.gov.br)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=addcapes_dr)))
```

#### PROD
 
```bash
jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=oraclesrv01.capes.gov.br)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=prod_dr)))
```
 
#### PROTECT
 
```bash
jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=oraclesrv02.capes.gov.br)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=protect_dr)))
```
### ESPELHO PRODUÇÃO JDBC

### PRODDG

```bash
jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=oracledgsrv01.capes.gov.br)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=proddg)))
 ```
### PROTECTDG

```bash
jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=oracledgsrv02.capes.gov.br)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=protectdg)))
 ```
### ADDCAPDG

```bash
jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=oracledgsrv03.capes.gov.br)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=addcapdg)))
```
### DESENVOLVIMENTO/HOMOLOGAÇÃO/TESTE/PRÉ-PRODUÇÃO

#### ADDCAPD 
 
```bash
jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=oracledhtsrv06.hom.capes.gov.br)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=addcapd_dr)))
```

#### DESENVOLVIMENTO 
 
```bash
jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=oracledhtsrv01.hom.capes.gov.br)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=dsnv_dr)))
```

#### HOMOLOGAÇÃO 
 
```bash
jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=oracledhtsrv02.hom.capes.gov.br)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=hom_dr)))
```

#### PEND 
 
```bash
jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=oracledhtsrv03.hom.capes.gov.br)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=pend_dr)))
```
 
#### PRÉ-PRODUÇÃO
 
```bash
jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=oracledhtsrv04.hom.capes.gov.br)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=preprod_dr)))
```

#### TESTE 
 
```bash
jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=oracledhtsrv05.hom.capes.gov.br)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=teste_dr)))
```


<br><br><br>

# POSTGRES
 
## CONEXÕES JDBC POSTGRES
 
### PRODUÇÃO
 
```bash
Host name: postgres.capes.gov.br
Port number: 5432
Banco: producao
ou
Host name: postgres9.capes.gov.br
Port number: 5432
Banco: producao

jdbc:postgresql://postgres.capes.gov.br:5432/producao
ou
jdbc:postgresql://postgres9.capes.gov.br:5432/producao
```
 
 #### ESPELHO DE PRODUÇÃO

```bash
Host name: postgresro.capes.gov.br
Port number: 5432
Banco: producao
 
jdbc:postgresql://postgresro.capes.gov.br:5432/producao
``` 
 

#### SISREL

```bash
Host name: v-postgresisrel01.capes.gov.br(melhor opção) ou XXX.XXX.XXX.XXX
Port number: 5432
Banco: sisrel2
 
jdbc:postgresql://v-postgresisrel01.capes.gov.br:5432/sisrel2
```
 
### DESENVOLVIMENTO/HOMOLOGACAO/TESTE/PREPROD
 
 
#### ENTERPRISEDB


##### DESENVOLVIMENTO ==>> ***DESATIVADO*** <<== 
~~Host name: v-des-enterprisedb01.des.capes.gov.br(melhor opção) ou XXX.XXX.XXX.XXX
Port number: 5432
Banco: Dsnv~~ 
 
~~jdbc:postgresql://v-des-enterprisedb01.des.capes.gov.br:5432/Dsnv~~

##### HOMOLOGACAO ==>> ***DESATIVADO*** <<== 
~~Host name: postgres.hom.capes.gov.br(melhor opção) ou XXX.XXX.XXX.XXX
Port number: 5432
Banco: homologacao~~ 
 
 
~~jdbc:postgresql://postgres.hom.capes.gov.br:5432/homologacao~~

##### PREPROD ==>> ***DESATIVADO*** <<== 
~~Host name: postgres.hom.capes.gov.br(melhor opção) ou XXX.XXX.XXX.XXX
Port number: 5432
Banco: preprod~~ 
 
 
~~jdbc:postgresql://postgres.hom.capes.gov.br:5432/preprod~~

##### TESTE  ==>> ***DESATIVADO*** <<==
~~Host name: postgres.teste.capes.gov.br(melhor opção) ou XXX.XXX.XXX.XXX
Port number: 5432
Banco: teste~~ 
 
~~jdbc:postgresql://postgres.teste.capes.gov.br:5432/teste~~
 
 
##### SISREL DES
 
Host name: v-postegres-sisrel-des.capes.gov.br (melhor opção) ou XXX.XXX.XXX.XXX
Port number: 5432
Banco: sisrel_des 
 
```bash
jdbc:postgresql://v-postegres-sisrel-des.capes.gov.br:5432/sisrel_des
```
 
##### POSTGRES9 DES
Host name: postgresdhtsrv01.hom.capes.gov.br
Port number: 5432
Banco: postgres, Dsnv, administrativo. etc 
 
```bash
jdbc:postgresql://postgresdhtsrv01.hom.capes.gov.br:5432/<banco de dados>
```
 
##### POSTGRES9 HOM
Host name: postgresdhtsrv02.hom.capes.gov.br
Port number: 5432
Banco: postgres, homologacao 
 
```bash
jdbc:postgresql:/postgresdhtsrv02.hom.capes.gov.br:5432/<banco de dados>
```

##### POSTGRES9 TESTE
Host name: postgresdhtsrv03.hom.capes.gov.br
Port number: 5432
Banco: postgres, teste, administrativo cadastrodepessoas, tst_scba, etc. 
 
```bash
jdbc:postgresql://postgresdhtsrv03.hom.capes.gov.br:5432/<banco de dados>
```

##### POSTGRES9 PREPROD
Host name: postgresdhtsrv04.hom.capes.gov.br
Port number: 5432
Banco: postgres, preprod 
 
```bash
jdbc:postgresql:/postgresdhtsrv04.hom.capes.gov.br:5432/<banco de dados>
```

<br><br><br>

# SYBASE

## CONEXÕES JDBC SYBASE
 
### PRODUÇÃO
 
#### PROD0
Host name: XXX.XXX.XXX.XXX
Port number: 5000
 
```bash
jdbc:jtds:sybase://XXX.XXX.XXX.XXX:XXXXX:/<NOME_DO_BANCO>
```
 
 
#### PROD1
Host name: XXX.XXX.XXX.XXX
Port number: 5000
 
```bash
jdbc:jtds:sybase://XXX.XXX.XXX.XXX:5000:/<nomedobanco>
```
 
 
#### PROD_DW
Host name: XXX.XXX.XXX.XXX
Port number: 6500
 
```bash
jdbc:jtds:sybase://XXX.XXX.XXX.XXX:6500:/<nomedobanco>
```
 
 
 
### DESENVOLVIMENTO
 
#### DESENV0
Host name: XXX.XXX.XXX.XXX
Port number: 5200
 
```bash
jdbc:jtds:sybase://XXX.XXX.XXX.XXX:5200:/<nomedobanco>
```
 
 
#### DESENV1
Host name: XXX.XXX.XXX.XXX
Port number: 5300
 
```bash
jdbc:jtds:sybase://XXX.XXX.XXX.XXX:5300:/<nomedobanco>
```
 
### HOMOLOGAÇÃO
 
#### HOM0
Host name: XXX.XXX.XXX.XXX
Port number: 5000
 
```bash
jdbc:jtds:sybase://XXX.XXX.XXX.XXX:5000:/<nomedobanco>
```
 
 
#### HOM1
Host name: XXX.XXX.XXX.XXX
Port number: 5100
 
```bash
jdbc:jtds:sybase://XXX.XXX.XXX.XXX:5100:/<nomedobanco>
```
