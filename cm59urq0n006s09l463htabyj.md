---
title: "Oracle Database Vault: Uma verdadeira "Muralha" para seu banco de dados"
datePublished: Sun Dec 29 2024 16:56:39 GMT+0000 (Coordinated Universal Time)
cuid: cm59urq0n006s09l463htabyj
slug: oracle-database-vault-uma-verdadeira-muralha-para-seu-banco-de-dados
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1735491346938/13c1fc22-ce2a-439c-a5f0-6333f1a2b295.png
tags: oracle, oracle-database, oracledatabasevault

---

Até aqui, tínhamos liberdade para fazer qualquer coisa com o usuário SYS, com certeza podíamos dizer que ele era o 'Zeus' do banco de dados Oracle. Só que, no momento em que ativamos o Database Vault, algumas regras mudam. Vamos habilitar o Database Vault e observar na prática como ele se comporta no banco de dados.

Primeiro vamos verificar se o Database Vault e o Label Security estão "instalados" no banco de dados.

```bash
 SQL> SELECT  comp_name, version, status FROM dba_registry WHERE comp_name in( 'Oracle Database Vault', 'Oracle Label Security');

Component                                    Version           Status
-------------------------------------------- ----------------- -----------------
Oracle Label Security                        19.0.0.0.0        VALID
Oracle Database Vault                        19.0.0.0.0        VALID
```

Já que os componentes estão instalados podemos prosseguir...

Mas antes gostaria de colocar uma observação, nem todo o banco tem o DBV e Label security instalados, principalmente aqueles bancos que foram migrados do 11G para o 19C. Em breve disponibilizarei um link nesse artigo demonstrando como instalar esses componentes "do zero".

Sem delongas, vamos para a parte boa.

O Database Vault precisa de 2 usuários, de forma macro, esses usuários são o de administração e o de gerenciamento. "NAO PERCA A SENHA DESSES USUÁRIOS, ISSO PODE TE DAR UMA GRANDE DOR DE CABEÇA".

Vamos la:

```bash
CREATE PROFILE PRF_DATABASE_VAULT LIMIT password_life_time UNLIMITED;

CREATE USER DBV_ADMIN IDENTIFIED BY WElcome1#_ PROFILE PRF_DATABASE_VAULT;

CREATE USER DBV_ADMIN_BACKUP IDENTIFIED BY WElcome1#_ PROFILE PRF_DATABASE_VAULT;

CREATE USER DBV_MGR IDENTIFIED BY WElcome1#_ PROFILE PRF_DATABASE_VAULT;

CREATE USER DBV_MGR_BACKUP IDENTIFIED BY WElcome1#_ PROFILE PRF_DATABASE_VAULT;

GRANT DV_OWNER TO DBV_ADMIN;
GRANT DV_ADMIN TO DBV_ADMIN;
GRANT CONNECT TO DBV_ADMIN;
GRANT CREATE SESSION TO DBV_ADMIN;

GRANT DV_ACCTMGR TO DBV_MGR;
GRANT CONNECT TO DBV_MGR;
GRANT CREATE SESSION TO DBV_MGR;


GRANT DV_OWNER TO DBV_ADMIN_BACKUP;
GRANT DV_ADMIN TO DBV_ADMIN_BACKUP;
GRANT CONNECT TO DBV_ADMIN_BACKUP;
GRANT CREATE SESSION TO DBV_ADMIN_BACKUP;

GRANT DV_ACCTMGR TO DBV_MGR_BACKUP;
GRANT CONNECT TO DBV_MGR_BACKUP;
GRANT CREATE SESSION TO DBV_MGR_BACKUP;
```

Anteriormente disse que são 2 usuários para o Database Vault, um admin e outro de MGR, por precaução criamos um usuário backup, para caso algo ocorra com os principais usuários.

```bash
SQL> CREATE PROFILE PRF_DATABASE_VAULT LIMIT password_life_time UNLIMITED;

Profile created.

SQL> 
SQL> CREATE USER DBV_ADMIN IDENTIFIED BY WElcome1#_ PROFILE PRF_DATABASE_VAULT;

User created.

SQL> 
SQL> CREATE USER DBV_ADMIN_BACKUP IDENTIFIED BY WElcome1#_ PROFILE PRF_DATABASE_VAULT;

User created.

SQL> 
SQL> CREATE USER DBV_MGR IDENTIFIED BY WElcome1#_ PROFILE PRF_DATABASE_VAULT;

User created.

SQL> 
SQL> CREATE USER DBV_MGR_BACKUP IDENTIFIED BY WElcome1#_ PROFILE PRF_DATABASE_VAULT;

User created.

SQL> GRANT DV_OWNER TO DBV_ADMIN;

Grant succeeded.

SQL> GRANT DV_ADMIN TO DBV_ADMIN;

Grant succeeded.

SQL> GRANT CONNECT TO DBV_ADMIN;

Grant succeeded.

SQL> GRANT CREATE SESSION TO DBV_ADMIN;

Grant succeeded.

SQL> 
SQL> GRANT DV_ACCTMGR TO DBV_MGR;

Grant succeeded.

SQL> GRANT CONNECT TO DBV_MGR;

Grant succeeded.

SQL> GRANT CREATE SESSION TO DBV_MGR;

Grant succeeded.

SQL> 
SQL> 
SQL> GRANT DV_OWNER TO DBV_ADMIN_BACKUP;

Grant succeeded.

SQL> GRANT DV_ADMIN TO DBV_ADMIN_BACKUP;

Grant succeeded.

SQL> GRANT CONNECT TO DBV_ADMIN_BACKUP;

Grant succeeded.

SQL> GRANT CREATE SESSION TO DBV_ADMIN_BACKUP;

Grant succeeded.

SQL> 
SQL> GRANT DV_ACCTMGR TO DBV_MGR_BACKUP;

Grant succeeded.

SQL> GRANT CONNECT TO DBV_MGR_BACKUP;

Grant succeeded.

SQL> GRANT CREATE SESSION TO DBV_MGR_BACKUP;

Grant succeeded.
```

Agora vamos ativar o DBV, como eu disse no início desse artigo o Database Vault vai mudar completando as permissões no banco, um exemplo que o SYS vai se tornar "um usuário mortal apenas", nenhum usuário terá mais super poderes, então homologue bastante o Database Vault

Antes de ativar em produção. Avisos feitos. Vamos la!

```bash
verificando status atual:


select * from cdb_dv_status where name in ('DV_CONFIGURE_STATUS','DV_ENABLE_STATUS') ; 
NAME                Status                CON_ID
------------------- ----------------- ----------
DV_CONFIGURE_STATUS FALSE                      0
DV_ENABLE_STATUS    FALSE                      0
```

```bash
Criando um restore point que nunca é demais:

SQL> create restore point PRE_DBV guarantee flashback database;

Restore point created.
```

```bash
configurando DBV:

BEGIN
 DVSYS.CONFIGURE_DV (
   dvowner_uname         => 'DBV_ADMIN',
   dvacctmgr_uname       => 'DBV_MGR');
 END;
/
```

```bash
 SQL> select * from cdb_dv_status where name in ('DV_CONFIGURE_STATUS','DV_ENABLE_STATUS') ; 

NAME                                            STATUS        
-------------------                  --------------
DV_CONFIGURE_STATUS            TRUE          
DV_ENABLE_STATUS                    FALSE 
```

O configure esta como true, vamos ativar o dbv.

```bash
ativando dbv:

conn dbv_admin/WElcome1#_

EXEC DBMS_MACADM.ENABLE_DV;

SQL> conn dbv_admin/WElcome1#_
Connected.
SQL> 
SQL> 
SQL> 
SQL> show user
USER is "DBV_ADMIN"
SQL> EXEC DBMS_MACADM.ENABLE_DV;

PL/SQL procedure successfully completed.

SQL>  select * from cdb_dv_status where name in ('DV_CONFIGURE_STATUS','DV_ENABLE_STATUS') ; 

NAME                STATUS          CON_ID
------------------- ----------- ------------------
DV_CONFIGURE_STATUS TRUE              0
DV_ENABLE_STATUS    FALSE             0
```

Mesmo depois do "activate" DBV status está como FALSE, para ativarmos precisamos reiniciar a instância...

Estamos vamos la:

```bash
SQL> shut immediate ; 
startup ; 
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> ORACLE instance started.

Total System Global Area 2365584096 bytes
Fixed Size                  8942304 bytes
Variable Size             520093696 bytes
Database Buffers         1828716544 bytes
Redo Buffers                7831552 bytes
Database mounted.
Database opened.

SQL> select * from cdb_dv_status where name in ('DV_CONFIGURE_STATUS','DV_ENABLE_STATUS') ; 

NAME                STATUS                   CON_ID
------------------- -------------------- ----------
DV_CONFIGURE_STATUS TRUE                          0
DV_ENABLE_STATUS    TRUE                          0 
```

Pronto, DBV ativado.

Antes, podíamos criar usuários normalmente:

```bash
SQL> show user      
USER is "SYS"
SQL> create user dbv_teste_sys identified by WElcome1#_ ; 

User created.

SQL> drop user dbv_teste_sys ;

User dropped.
```

Agora vamos tentar criar um usuário com o DBV ativado:

```bash
SQL> show user
USER is "SYS"
SQL> create user dbv_teste_sys identified by WElcome1#_ ;      
create user dbv_teste_sys identified by WElcome1#_
*
ERROR at line 1:
ORA-01031: insufficient privileges
```

para criar usuários agora apenas com o usuário DBV\_MGR

```bash
conn DBV_MGR/WElcome1#_

SQL> show user
USER is "DBV_MGR"
SQL>  create user dbv_teste_sys identified by WElcome1#_ ; 

User created. 
```

Antes de ativar o DBV, homologue profundamente no seu ambiente e fique atento as permissões que serão revogadas das roles. [Nesse link](https://docs.oracle.com/en/database/oracle/oracle-database/19/dvadm/what-to-expect-after-you-enable-oracle-database-vault.html#GUID-C91754D9-950D-4E1E-A697-B26D4C4D3B9C) você conseguirá ver todas as roles e as melhores praticas para o mesmo.