---
title: "Instalando Database Vault"
datePublished: Sun Dec 29 2024 16:59:58 GMT+0000 (Coordinated Universal Time)
cuid: cm59uvzlj000209kzf7eq1tys
slug: instalando-database-vault
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1735491535450/68c50847-ed42-4c76-a0f6-61bbd805d51e.png
tags: oracle, oracle-database, oracledatabasevault

---

No [artigo anterior](https://diogofernandes.com.br/oracle-database-vault-uma-verdadeira-muralha-para-seu-banco-de-dados), demonstrei como configurar o Database Vault em um ambiente.

No entanto, em alguns ambientes 19c que foram migrados da versão 11g, o Database Vault pode não estar instalado. Mas como posso verificar isso?

```bash
sqlplus / as sysdba 

SQL*Plus: Release 19.0.0.0.0 - Production on Tue Oct 29 20:58:51 2024
Version 19.19.0.0.0

Copyright (c) 1982, 2022, Oracle.  All rights reserved.


Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.19.0.0.0

SQL> set pages 50
SQL> set line 200
SQL> col comp_name for a50
SQL> SELECT  comp_name, version, status FROM dba_registry WHERE comp_name in( 'Oracle Database Vault', 'Oracle Label Security');

no rows selected

SQL> SELECT  comp_name, version FROM dba_registry  ;

COMP_NAME                                                                   VERSION       
-------------------------------------------------- --------------
Oracle Database Catalog Views                                   19.0.0.0.0    
Oracle Database Packages and Types                        19.0.0.0.0    
JServer JAVA Virtual Machine                                      19.0.0.0.0    
Oracle XDK                                                                     19.0.0.0.0    
Oracle Database Java Packages                                  19.0.0.0.0    
OLAP Analytic Workspace                                            19.0.0.0.0    
Oracle Real Application Clusters                                 19.0.0.0.0    
Oracle Workspace Manager                                         19.0.0.0.0    
Oracle Text                                                                     19.0.0.0.0    
Oracle XML Database                                                   19.0.0.0.0    
Oracle Multimedia                                                         19.0.0.0.0    
Spatial                                                                             19.0.0.0.0    
Oracle OLAP API                                                            19.0.0.0.0    
Oracle Application Express                                           5.0.4.00.12   

14 rows selected.
```

Como podemos ver acima, este é um banco 19C e não temos o DBV instalado.

O Database vault é dependente do Oracle Label Security, então vamos "instalar" ele primeiro e depois o Database Vault.

Preparação para instalar o Oracle Label Security

Criação de Tablespace's exclusivas para esses itens:

```bash
CREATE TABLESPACE TBS_DBV DATAFILE SIZE 1G AUTOEXTEND ON NEXT 1G ;

CREATE TEMPORARY TABLESPACE TBS_DBV_TEMP TEMPFILE SIZE 1G AUTOEXTEND ON NEXT 1G ;
```

Verificando....

```bash
SQL> select TABLESPACE_NAME from dba_tablespaces where TABLESPACE_NAME in ('TBS_DBV','TBS_DBV_TEMP') ;

TABLESPACE_NAME
------------------------------
TBS_DBV
TBS_DBV_TEMP 
```

Criando um restore point que nunca é demais....

```bash
create restore point PRE_DBV guarantee flashback database; 
```

Agora vamos iniciar o procedimento de instalação do DBV, este procedimento está na nota: How To Enable/Install/Uninstall Database Vault in oracle database? (Doc ID 2112167.1)

Instalando o Label Security (Este procedimento demora em torno de 10 a 20 min)

```bash
@$ORACLE_HOME/rdbms/admin/catols.sql
```

Depois da execução do script acima, saia no sqlplus e logue novamente para executar os seguintes comandos:

```bash
exec lbacsys.configure_ols
exec lbacsys.ols_enforcement.enable_ols

SQL> exec lbacsys.configure_ols

PL/SQL procedure successfully completed.

SQL> exec lbacsys.ols_enforcement.enable_ols

PL/SQL procedure successfully completed. 
```

Vamos verificar agora se o label secury foi instalado com sucesso:

```bash
set line 200
set pages 45
col comp_name for a50
SELECT  comp_name, version, status FROM dba_registry WHERE comp_name in( 'Oracle Database Vault', 'Oracle Label Security');

COMP_NAME                                          VERSION                        STATUS
-------------------------------------------------- ------------------------ 
Oracle Label Security                              19.0.0.0.0                     VALID 
```

Agora vamos instalar o Database Vault, o primeiro parâmetro é a Tablespace permanente e a segunda temporária que criamos no início do artigo.

```bash
@$ORACLE_HOME/rdbms/admin/catmac.sql TBS_DBV TBS_DBV_TEMP
```

```bash
sqlplus / as sysdba 

SQL*Plus: Release 19.0.0.0.0 - Production on Tue Oct 29 22:36:23 2024
Version 19.19.0.0.0

Copyright (c) 1982, 2022, Oracle.  All rights reserved.


Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.19.0.0.0

SQL> select TABLESPACE_NAME from dba_tablespaces where TABLESPACE_NAME in ('TBS_DBV','TBS_DBV_TEMP') ;

TABLESPACE_NAME
------------------------------
TBS_DBV
TBS_DBV_TEMP

SQL> show user 
USER is "SYS"
-- A execução deste script demora por volta de 10 min.

SQL> @$ORACLE_HOME/rdbms/admin/catmac.sql TBS_DBV TBS_DBV_TEMP

Ultimas linhas do spool:

Commit complete.

SQL> 
SQL> @?/rdbms/admin/sqlsessend.sql
SQL> Rem
SQL> Rem $Header: rdbms/admin/sqlsessend.sql /main/3 2018/07/25 13:50:02 surman Exp $
SQL> Rem
SQL> Rem sqlsessend.sql
SQL> Rem
SQL> Rem Copyright (c) 2013, 2018, Oracle and/or its affiliates.
SQL> Rem All rights reserved.
SQL> Rem
SQL> Rem    NAME
SQL> Rem      sqlsessend.sql - SQL session end
SQL> Rem
SQL> Rem    DESCRIPTION
SQL> Rem      Any commands which should be run at the end of all oracle
SQL> Rem      supplied scripts.
SQL> Rem
SQL> Rem    NOTES
SQL> Rem      See sqlsessstart.sql for the corresponding start script.
SQL> Rem
SQL> Rem    BEGIN SQL_FILE_METADATA
SQL> Rem    SQL_SOURCE_FILE: rdbms/admin/sqlsessend.sql
SQL> Rem    SQL_SHIPPED_FILE: rdbms/admin/sqlsessend.sql
SQL> Rem    SQL_PHASE: MISC
SQL> Rem    SQL_STARTUP_MODE: NORMAL
SQL> Rem    SQL_IGNORABLE_ERRORS: NONE
SQL> Rem    END SQL_FILE_METADATA
SQL> Rem
SQL> Rem    MODIFIED   (MM/DD/YY)
SQL> Rem    surman      05/04/18 - 27464252: Update SQL_PHASE
SQL> Rem    surman      03/08/13 - 16462837: Common start and end scripts
SQL> Rem    surman      03/08/13 - Created
SQL> Rem
SQL> 
SQL> alter session set "_ORACLE_SCRIPT" = false;
```

Instalação concluída vamos verificar os status do database vault:

```bash
set line 200
set pages 45
col comp_name for a50
SELECT  comp_name, version, status FROM dba_registry WHERE comp_name in( 'Oracle Database Vault', 'Oracle Label Security');


COMP_NAME                                          VERSION                        STATUS
-------------------------------------------------- -----------------------------
Oracle Label Security                              19.0.0.0.0                     VALID
Oracle Database Vault                             19.0.0.0.0                     VALID 
```

Pronto! Caso você queira habilitar/configurar o Database Vault no seu banco de dados, dá uma olhada [nesse link](https://diogofernandes.com.br/oracle-database-vault-uma-verdadeira-muralha-para-seu-banco-de-dados):

Algumas observações:

1 - A instalação do Label Security em instâncias que estão em RAC e com PDB pode causar um "stuck" na segunda instância. Tive esse problema em versões abaixo da 19.20, mas, até onde verifiquei, ele foi resolvido a partir da versão 19.20.

2 - Caso algum objeto fique inválido, execute o UTLRP. Recomendo fazer um backup da tabela DBA\_OBJECTS com a condição WHERE STATUS='INVALID' para verificar posteriormente se todos os objetos permanecem "estáveis" após a instalação do DBV.

Como sempre, recomendo realizar validações rigorosas em homologações e nunca executar nada diretamente em produção.