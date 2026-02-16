---
title: "Escolhendo o DBID"
datePublished: Sun Feb 15 2026 20:16:03 GMT+0000 (Coordinated Universal Time)
cuid: cmlo6qytx000i02jo77yx59jb
slug: escolhendo-o-dbid
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1771188111254/5a650f43-d873-412d-b15c-eac841b17677.png

---

Alguns ERP’s têm o seu licenciamento vinculado ao DBID do banco Oracle, até aí tudo bem, até o dia em que o banco precisa ser migrado. Se for via Data Guard, tranquilo, mas há meios de upgrade, por exemplo, que podem mudar o DBID, e depois da migração isso vira um “problema”, principalmente em ERP’s que têm dezenas de arquivos com esse número de DBID para vincular o licenciamento, então é melhor trocar o DBID. Não vou entrar no mérito se é certo ou não, mas às vezes é o que temos que fazer.

Recordo-me de que a primeira vez que vi esse procedimento foi há 12 anos no blog [oraclehome.com.br](https://oraclehome.com.br/2014/09/11/escolhendo-meu-dbid/), inclusive, se você está iniciando a carreira no Oracle, sugiro ler todos os artigos que têm lá. E nesse mesmo artigo fui encaminhado para um artigo da [Pythian](https://www.pythian.com/blog/how-to-choose-your-oracle-database-id-dbid) em 2009, ou seja, não espere encontrar uma nota aqui, isso aqui é literalmente pergaminho do Oracle rs.

# <mark>Não faça isso em produção sem validar 500 vezes na sua base de homologação/testes.</mark>

Avisos dados, vamos lá.

Primeiro, você tem que colocar a sua base em estado MOUNT.

```bash
Connected to:
Oracle Database 23ai Free Release 23.0.0.0.0 - Develop, Learn, and Run for Free
Version 23.5.0.24.07

SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 FREEPDB1                       READ WRITE NO
SQL> select dbid from v$database ; 

      DBID
----------
1452480138

SQL> shut immediate ; 
Database closed.
Database dismounted.
ORACLE instance shut down.
```

O shutdown tem que ser immediate, não rode esse procedimento com shutdown abort.

Como podemos ver, o DBID até o momento é 1452480138.

Agora vamos iniciar o banco em MOUNT.

```bash
SQL> startup mount ; 
ORACLE instance started.

Total System Global Area 1603726640 bytes
Fixed Size                  5360944 bytes
Variable Size             436207616 bytes
Database Buffers         1157627904 bytes
Redo Buffers                4530176 bytes
Database mounted.
SQL> 
SQL> 
SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       MOUNTED
         3 FREEPDB1                       MOUNTED
```

Pronto, startup feito em mount. Agora vamos para mágica.

O script abaixo que devemos executar:

```bash
set serveroutput on

declare
  v_chgdbid   binary_integer;
  v_chgdbname binary_integer;
  v_skipped   binary_integer;
  v_new_db_name varchar2(9);
  v_old_db_name varchar2(9);
  v_new_dbid    number;
  v_old_dbid    number;
  w_action      varchar2(255);


begin
     w_action:='Recuperando DBID Atual.' ;
     select dbid, name, name into v_old_dbid, v_new_db_name, v_old_db_name  from v$database;
     select TO_NUMBER('&NOVO_DBID') into v_new_dbid from dual;

     w_action:='Executando a Procedure (dbms_backup_restore.nidbegin).';
     dbms_output.put_line('New NAME='||V_NEW_DB_NAME);
     dbms_output.put_line('Old NAME='||V_OLD_DB_NAME);
     dbms_output.put_line('New DBID='||V_NEW_DBID);
     dbms_output.put_line('Old DBID='||V_OLD_DBID);

     dbms_backup_restore.nidbegin(V_NEW_DB_NAME,V_OLD_DB_NAME,V_NEW_DBID,V_OLD_DBID,0,0,10);

     w_action:='Executando a Procedure (dbms_backup_restore.nidprocesscf).';
     dbms_backup_restore.nidprocesscf( v_chgdbid,v_chgdbname);

     dbms_output.put_line('ControlFile.......: ');
     dbms_output.put_line('  => Change Name..: '  ||to_char(v_chgdbname));
     dbms_output.put_line('  => Change DBID..: '  ||to_char(v_chgdbid));

     w_action := 'Alterando os Datafiles, procedure (dbms_backup_restore.nidprocessdf).';
     for i in (select file#,name from v$datafile)
     loop
        dbms_output.put_line('DataFile..........: '  ||i.name);
        dbms_output.put_line('  => Skipped......: '  ||to_char(v_skipped));
        dbms_output.put_line('  => Change Name..: '  ||to_char(v_chgdbname));
        dbms_output.put_line('  => Change DBID..: '  ||to_char(v_chgdbid));
        dbms_backup_restore.nidprocessdf(i.file#,0, v_skipped,v_chgdbid,v_chgdbname);
     end loop;

     w_action := 'Alterando os Tempfiles, procedure (dbms_backup_restore.nidprocessdf).';
     for i in (select file#,name from v$tempfile)
     loop
        dbms_output.put_line('TempFile..........: '  ||i.name);
        dbms_output.put_line('  => Skipped......: '  ||to_char(v_skipped));
        dbms_output.put_line('  => Change Name..: '  ||to_char(v_chgdbname));
        dbms_output.put_line('  => Change DBID..: '  ||to_char(v_chgdbid));
        dbms_backup_restore.nidprocessdf(i.file#,1,v_skipped,v_chgdbid,v_chgdbname);
     end loop;
  dbms_backup_restore.nidend;
end;
/
```

Ao executar, ele vai pedir para digitar o DBID que você quer setar:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1771181899022/7857e48e-844b-4a2b-ac00-4c9a2295deb9.png align="center")

```bash
Terminal rodando o script:

SQL> set serveroutput on

declare
  v_chgdbid   binary_integer;
  v_chgdbname binary_integer;
  v_skipped   binary_integer;
  v_new_db_name varchar2(9);
  v_old_db_name varchar2(9);
  v_new_dbid    number;
  v_old_dbid    number;
  w_action      varchar2(255);


begin
     w_action:='Recuperando DBID Atual.' ;
     select dbid, name, name into v_old_dbid, v_new_db_name, v_old_db_name  from v$database;
     select TO_NUMBER('&NOVO_DBID') into v_new_dbid from dual;

     w_action:='Executando a Procedure (dbms_backup_restore.nidbegin).';
     dbms_output.put_line('New NAME='||V_NEW_DB_NAME);
     dbms_output.put_line('Old NAME='||V_OLD_DB_NAME);
     dbms_output.put_line('New DBID='||V_NEW_DBID);
     dbms_output.put_line('Old DBID='||V_OLD_DBID);

     dbms_backup_restore.nidbegin(V_NEW_DB_NAME,V_OLD_DB_NAME,V_NEW_DBID,V_OLD_DBID,0,0,10);

     w_action:='Executando a Procedure (dbms_backup_restore.nidprocesscf).';
     dbms_backup_restore.nidprocesscf( v_chgdbid,v_chgdbname);

     dbms_output.put_line('ControlFile.......: ');
     dbms_output.put_line('  => Change Name..: '  ||to_char(v_chgdbname));
     dbms_output.put_line('  => Change DBID..: '  ||to_char(v_chgdbid));

     w_action := 'Alterando os Datafiles, procedure (dbms_backup_restore.nidprocessdf).';
     for i in (select file#,name from v$datafile)
     loop
        dbms_output.put_line('DataFile..........: '  ||i.name);
        dbms_output.put_line('  => Skipped......: '  ||to_char(v_skipped));
        dbms_output.put_line('  => Change Name..: '  ||to_char(v_chgdbname));
        dbms_output.put_line('  => Change DBID..: '  ||to_char(v_chgdbid));
        dbms_backup_restore.nidprocessdf(i.file#,0, v_skipped,v_chgdbid,v_chgdbname);
     end loop;

     w_action := 'Alterando os Tempfiles, procedure (dbms_backup_restore.nidprocessdf).';
     for i in (select file#,name from v$tempfile)
     loop
        dbms_output.put_line('TempFile..........: '  ||i.name);
        dbms_output.put_line('  => Skipped......: '  ||to_char(v_skipped));
        dbms_output.put_line('  => Change Name..: '  ||to_char(v_chgdbname));
        dbms_output.put_line('  => Change DBID..: '  ||to_char(v_chgdbid));
        dbms_backup_restore.nidprocessdf(i.file#,1,v_skipped,v_chgdbid,v_chgdbname);
     end loop;
  dbms_backup_restore.nidend;
end;
/SQL> SQL>   2    3    4    5    6    7    8    9   10   11   12   13   14   15   16   17   18   19   20   21   22   23   24   25   26   27   28   29   30   31   32   33   34   35   36   37   38   39   40   41   42   43   44   45   46   47   48   49   50   51   52   53  
Enter value for novo_dbid: 777777
old  15:      select TO_NUMBER('&NOVO_DBID') into v_new_dbid from dual;
new  15:      select TO_NUMBER('777777') into v_new_dbid from dual;
New NAME=FREE
Old NAME=FREE
New DBID=777777
Old DBID=1452480138
ControlFile.......:
=> Change Name..: 0
=> Change DBID..: 1
DataFile..........: /opt/oracle/oradata/FREE/system01.dbf
=> Skipped......:
=> Change Name..: 0
=> Change DBID..: 1
DataFile..........: /opt/oracle/oradata/FREE/pdbseed/system01.dbf
=> Skipped......: 0
=> Change Name..: 0
=> Change DBID..: 1
DataFile..........: /opt/oracle/oradata/FREE/sysaux01.dbf
=> Skipped......: 0
=> Change Name..: 0
=> Change DBID..: 1
DataFile..........: /opt/oracle/oradata/FREE/pdbseed/sysaux01.dbf
=> Skipped......: 0
=> Change Name..: 0
=> Change DBID..: 1
DataFile..........: /opt/oracle/oradata/FREE/users01.dbf
=> Skipped......: 0
=> Change Name..: 0
=> Change DBID..: 1
DataFile..........: /opt/oracle/oradata/FREE/pdbseed/undotbs01.dbf
=> Skipped......: 0
=> Change Name..: 0
=> Change DBID..: 1
DataFile..........: /opt/oracle/oradata/FREE/undotbs01.dbf
=> Skipped......: 0
=> Change Name..: 0
=> Change DBID..: 1
DataFile..........: /opt/oracle/oradata/FREE/FREEPDB1/system01.dbf
=> Skipped......: 0
=> Change Name..: 0
=> Change DBID..: 1
DataFile..........: /opt/oracle/oradata/FREE/FREEPDB1/sysaux01.dbf
=> Skipped......: 0
=> Change Name..: 0
=> Change DBID..: 1
DataFile..........: /opt/oracle/oradata/FREE/FREEPDB1/undotbs01.dbf
=> Skipped......: 0
=> Change Name..: 0
=> Change DBID..: 1
DataFile..........: /opt/oracle/oradata/FREE/FREEPDB1/users01.dbf
=> Skipped......: 0
=> Change Name..: 0
=> Change DBID..: 1
TempFile..........: /opt/oracle/oradata/FREE/temp01.dbf
=> Skipped......: 0
=> Change Name..: 0
=> Change DBID..: 1
TempFile..........: /opt/oracle/oradata/FREE/pdbseed/temp01.dbf
=> Skipped......: 0
=> Change Name..: 0
=> Change DBID..: 1
TempFile..........: /opt/oracle/oradata/FREE/FREEPDB1/temp01.dbf
=> Skipped......: 0
=> Change Name..: 0
=> Change DBID..: 1

PL/SQL procedure successfully completed.
```

Após concluir, abra o banco com OPEN RESETLOGS.

```bash
SQL> alter database open resetlogs ; 

Database altered.
```

Status do banco:

```bash
SQL> alter database open resetlogs ; 

Database altered.

SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 FREEPDB1                       READ WRITE NO
SQL> select dbid from v$database ; 

      DBID
----------
    777777
```

Pronto, agora o banco esta com o DBID de sua escolha.

Espero que este artigo possa te ajudar em migrações futuras. Qualquer coisa, só chamar no [**linkedin**](https://www.linkedin.com/in/diogo-fernandess/) 🙂

**PS: Em caso de DB produção, faça backup full imediatamente após a alteração.**

**Créditos:**

*Artigo no* [*oraclehome.com.br*](https://oraclehome.com.br/2014/09/11/escolhendo-meu-dbid/) *escrito por* [*Anderson Graf*](https://www.linkedin.com/in/andersongraf/)*. (2014)*

*Artigo da* [*Pythian*](https://www.pythian.com/blog/how-to-choose-your-oracle-database-id-dbid) *(2009)*