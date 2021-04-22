[[_TOC_]]

# Introdução

Este documento destina-se a documentação das strings de conexões dos SGBDs utilizados na CAPES.

> OBS. A string de conexão JDBC está na sua forma básica e pode necessitar de ajustes a depender da aplicação que a utilizará.
 
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


 

# POSTGRES
 
## CONEXÕES JDBC POSTGRES
 
### PRODUÇÃO
 
#### ENTERPRISEDB

Host name: f-prod-postgres01.fc.capes.gov.br(melhor opção) ou 172.19.100.240
Port number: 5432
Banco: producao
 
```bash
jdbc:postgresql://f-prod-postgres01.fc.capes.gov.br:5432/producao
```
 
 
Host name: postgresro.capes.gov.br(melhor opção) ou 172.19.100.155
Port number: 5432
Banco: producao  ==>> Banco D-1
 
```bash
jdbc:postgresql://postgresro.capes.gov.br:5432/producao
```
 
 
#### SISREL

Host name: v-postgresisrel01.capes.gov.br(melhor opção) ou 172.19.100.203
Port number: 5432
Banco: sisrel2
 
```bash
jdbc:postgresql://v-postgresisrel01.capes.gov.br:5432/sisrel2
```
 

#### POSTGRES9
Host name: postgres9.capes.gov.br(melhor opção) ou 172.19.212.10
Port number: 5432
Banco: postgres
 
```bash
jdbc:postgresql://postgres9.capes.gov.br:5432/postgres
```
 
 #### ESPELHO DE PRODUÇÃO

Host name: 172.19.100.155
Port number: 5432
Banco: producao
 
```bash
jdbc:postgresql://172.19.100.155:5432/producao
```
 
### DESENVOLVIMENTO/HOMOLOGACAO/TESTE/PREPROD
 
 
#### ENTERPRISEDB


##### DESENVOLVIMENTO
Host name: v-des-enterprisedb01.des.capes.gov.br(melhor opção) ou 172.19.100.177
Port number: 5432
Banco: Dsnv 
 
```bash
jdbc:postgresql://v-des-enterprisedb01.des.capes.gov.br:5432/Dsnv
```
 
##### HOMOLOGACAO
Host name: postgres.hom.capes.gov.br(melhor opção) ou 172.19.100.179
Port number: 5432
Banco: homologacao 
 
```bash
jdbc:postgresql://postgres.hom.capes.gov.br:5432/homologacao
```
 
##### PREPROD
Host name: postgres.hom.capes.gov.br(melhor opção) ou 172.19.100.179
Port number: 5432
Banco: preprod 
 
```bash
jdbc:postgresql://postgres.hom.capes.gov.br:5432/preprod
```
 
##### TESTE
Host name: postgres.teste.capes.gov.br(melhor opção) ou 172.19.100.121
Port number: 5432
Banco: teste 
 
```bash
jdbc:postgresql://postgres.teste.capes.gov.br:5432/teste
```
 
 
##### SISREL DES
 
Host name: v-postegres-sisrel-des.capes.gov.br (melhor opção) ou 172.19.100.154
Port number: 5432
Banco: sisrel_des 
 
```bash
jdbc:postgresql://v-postegres-sisrel-des.capes.gov.br:5432/sisrel_des
```
 
##### POSTGRES9 DES
Host name: postgres9.des.capes.gov.br (melhor opção) ou 172.19.128.22
Port number: 5432
Banco: postgres 
 
```bash
jdbc:postgresql://postgres9.des.capes.gov.br:5432/postgres
```
 
##### POSTGRES9 HOM
Host name: postgres9.hom.capes.gov.br (melhor opção) ou 172.19.126.31
Port number: 5432
Banco: postgres 
 
```bash
jdbc:postgresql://postgres9.hom.capes.gov.br:5432/postgres
```
 
# SYBASE

## CONEXÕES JDBC SYBASE
 
### PRODUÇÃO
 
#### PROD0
Host name: 172.19.100.214
Port number: 5000
 
```bash
jdbc:jtds:sybase://172.19.100.214:5000:/<NOME_DO_BANCO>
```
 
 
#### PROD1
Host name: 172.19.100.215
Port number: 5000
 
```bash
jdbc:jtds:sybase://172.19.100.215:5000:/<nomedobanco>
```
 
 
#### PROD_DW
Host name: 172.19.100.213
Port number: 6500
 
```bash
jdbc:jtds:sybase://172.19.100.213:6500:/<nomedobanco>
```
 
 
 
### DESENVOLVIMENTO
 
#### DESENV0
Host name: 172.19.100.130
Port number: 5200
 
```bash
jdbc:jtds:sybase://172.19.100.130:5200:/<nomedobanco>
```
 
 
#### DESENV1
Host name: 172.19.100.136
Port number: 5300
 
```bash
jdbc:jtds:sybase://172.19.100.136:5300:/<nomedobanco>
```
 
### HOMOLOGAÇÃO
 
#### HOM0
Host name: 172.19.100.130
Port number: 5000
 
```bash
jdbc:jtds:sybase://172.19.100.133:5000:/<nomedobanco>
```
 
 
#### HOM1
Host name: 172.19.100.136
Port number: 5100
 
```bash
jdbc:jtds:sybase://172.19.100.136:5100:/<nomedobanco>
```
