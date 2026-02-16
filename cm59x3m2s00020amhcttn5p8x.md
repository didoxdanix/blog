---
title: "Criando Rules SET no Oracle Database Vault"
datePublished: Sun Dec 29 2024 18:01:53 GMT+0000 (Coordinated Universal Time)
cuid: cm59x3m2s00020amhcttn5p8x
slug: criando-rules-set-no-oracle-database-vault
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1735495163989/c5bbe008-6cf8-4929-bed7-41d2078bdb36.png
tags: oracle, oracle-database, oracledatabasevaul

---

Nos artigos anteriores ([artigo 1](https://diogofernandes.com.br/oracle-database-vault-uma-verdadeira-muralha-para-seu-banco-de-dados) e [artigo 2](https://diogofernandes.com.br/instalando-database-vault)), mostrei como configurar e, em alguns casos, até como instalar o Oracle Database Vault.

Neste artigo, abordaremos os Rule Sets do DBV e veremos como eles podem proteger seus dados contra acessos não autorizados.

Mas antes, mostrarei a hierarquia dos Rule Sets no DBV, que é estruturada da seguinte forma:

![Article content](https://media.licdn.com/dms/image/v2/D4D12AQGLJq6aqM01MQ/article-inline_image-shrink_1500_2232/article-inline_image-shrink_1500_2232/0/1733165928207?e=1752105600&v=beta&t=an1LZlMeRGbO_0c2es8L3sJhGZFithj14N076R3XpdQ align="left")

* **Rule Set (Conjunto de Regras):** É um agrupamento de condições ou regras que podem ser aplicadas em diferentes contextos no Database Vault. Ele define critérios que devem ser atendidos para permitir ou negar uma ação, como acessar um dado ou executar um comando.
    
* **Command Rule (Regra de Comando):** [São reg](https://www.linkedin.com/pulse/oracle-database-vault-uma-verdadeira-muralha-para-seu-diogo-fernandes-lhhaf/)ras [aplicad](https://www.linkedin.com/pulse/instalando-database-vault-diogo-fernandes-pa2df/)as a comandos específicos do banco de dados, como SELECT, UPDATE, ou GRANT. Elas permitem restringir o uso desses comandos com base em condições configuradas no Rule Set.
    
* **Rule (Regra):** É a condição individual que compõe um Rule Set. Pode ser uma verificação simples, como confirmar se um usuário pertence a um grupo específico ou se uma conexão vem de um IP autorizado.
    

Obs.: Existem os Factors, mas estes serão tema de outro artigo. :)

Para demonstrar as permissões, temos o seguinte cenário:

* O **Owner APP**, que possui a tabela salario.
    
* Os usuários **Maria** e **João**, que têm permissão de leitura e escrita na tabela.
    

```bash
SQL> conn maria
Enter password: 
Connected.
SQL> show user
USER is "MARIA"
SQL> select * from app.salario ; 

        ID NOME                                                  SALARIO
---------- ----------------------------------------- ----------------------------- 
         1 João Silva                                               3000
         2 Maria Oliveira                                           4500
         3 Carlos Santos                                            5000
         4 Ana Costa                                                3200
         5 Pedro Almeida                                            2700
         6 Luiza Ferreira                                           6100
         7 Fernanda Lima                                            3800
         8 Roberto Faria                                            2900
         9 Cláudia Mendes                                           5300
        10 Tiago Barbosa                                            4700

10 rows selected.

SQL> delete app.salario where id=1 ; 

1 row deleted.

SQL> rollback ; 

Rollback complete.

SQL> conn joao
Enter password: 
Connected.
SQL> show user
USER is "JOAO"
SQL> select * from app.salario ; 

        ID NOME                                                  SALARIO
---------- ------------------------------------------- -------------------------- 
         1 João Silva                                               3000
         2 Maria Oliveira                                           4500
         3 Carlos Santos                                            5000
         4 Ana Costa                                                3200
         5 Pedro Almeida                                            2700
         6 Luiza Ferreira                                           6100
         7 Fernanda Lima                                            3800
         8 Roberto Faria                                            2900
         9 Cláudia Mendes                                           5300
        10 Tiago Barbosa                                            4700

10 rows selected.

SQL> delete app.salario where id=1 ; 

1 row deleted.

SQL> rollback ;

Rollback complete.
```

Como podemos ver acima, ambos têm permissão de "leitura e escrita" na tabela.

Agora, vamos criar uma regra no DBV que limita João a ter apenas acesso de SELECT, enquanto Maria poderá realizar as demais alterações.

Criando as roles "vazias":

```bash
SQL> conn / as sysdba 
Connected.
SQL> show user
USER is "SYS"
SQL> CREATE ROLE DBV_SELECT;

Role created.

SQL> CREATE ROLE DBV_DML;

Role created.
```

Obs.: As roles acima não possuem nenhuma permissão; elas serão utilizadas apenas como uma **FLAG**.

Criando a regra de Select:

```bash
SQL> show user
USER is "DBV_ADMIN"

SQL> BEGIN
  DVSYS.DBMS_MACADM.CREATE_RULE_SET(
    rule_set_name    => 'RULE_SET_SELECT',
    description      => 'Rule Set enabled for SELECT operations',
    enabled          => DVSYS.DBMS_MACUTL.G_YES,
    eval_options     => DBMS_MACUTL.G_RULESET_EVAL_ALL,
    audit_options    => DBMS_MACUTL.G_RULESET_AUDIT_FAIL,
    fail_options     => DBMS_MACUTL.G_RULESET_FAIL_SHOW,
    fail_message     => 'Access denied: You do not have SELECT privileges',
    fail_code        => -20001,
    handler_options  => DBMS_MACUTL.G_RULESET_HANDLER_OFF,
    handler          => NULL
  );
END;
/  2    3    4    5    6    7    8    9   10   11   12   13   14   15  

PL/SQL procedure successfully completed.
```

Adicionando "command rule" a Rule SET :

```bash
BEGIN
  DVSYS.DBMS_MACADM.CREATE_COMMAND_RULE(
    command         => 'SELECT',
    rule_set_name   => 'RULE_SET_SELECT',
    object_owner    => 'APP',
    object_name     => 'SALARIO',
    enabled         => DVSYS.DBMS_MACUTL.G_YES
  );
END;
/
```

Criando a rule:

```bash
QL> BEGIN
  DVSYS.DBMS_MACADM.CREATE_RULE(
    rule_name => 'RULE_SELECT',
    rule_expr => 'DVSYS.DBMS_MACUTL.USER_HAS_ROLE_VARCHAR(''DBV_SELECT'',''"''||dvsys.dv_login_user||''"'') = ''Y'''
  );
END;
/  2    3    4    5    6    7  

PL/SQL procedure successfully completed.
```

Agora, vamos fechar o "ciclo". P[recisamo](https://www.linkedin.com/pulse/oracle-database-vault-uma-verdadeira-muralha-para-seu-diogo-fernandes-lhhaf/)s a[dicionar](https://www.linkedin.com/pulse/instalando-database-vault-diogo-fernandes-pa2df/) a **Rule** a **Rule\_Set**, pois é dessa forma que o DBV "entenderá" que criamos uma condição específica para permitir que o usuário execute SELECT na tabela salario.

No nosso caso, configuramos o DBV para o se[guinte:](https://www.linkedin.com/pulse/instalando-database-vault-diogo-fernandes-pa2df/)

> O usuário precisa ter a role **DBV\_SELECT** para poder executar SELECT na tabela.

A role, por si só, não concede nenhum privilégio; ela foi criada como uma **role "vazia"** para ser usada como condição para realizar o SELECT na tabela.

No entanto, no DBV, você pode criar outras condições, como:

* Permitir acesso somente a partir de um IP específico.
    
* Restringir o acesso a determinados horários[.](https://www.linkedin.com/pulse/instalando-database-vault-diogo-fernandes-pa2df/)
    
* Outros
    

Adicionando a rule a rule\_set:

```bash
SQL> BEGIN
  DVSYS.DBMS_MACADM.ADD_RULE_TO_RULE_SET(
    rule_set_name => 'RULE_SET_SELECT',
    rule_name     => 'RULE_SELECT'
  );
END;
/  2    3    4    5    6    7  

PL/SQL procedure successfully completed.
```

Ainda não atribuímos a role **DBV\_SELECT** nem para João, nem para Maria. Antes disso, eles conseguiam executar SELECT na tabela. Vamos testar agora para ver o que acontece...

```bash
SQL> show user 
USER is "JOAO"
SQL> select * from app.salario ;
select * from app.salario
                  *
ERROR at line 1:
ORA-47306: 20001: Access denied: You do not have SELECT privileges


SQL> conn maria 
Enter password: 
Connected.
SQL> show user
USER is "MARIA"
SQL> select * from app.salario ;
select * from app.salario
                  *
ERROR at line 1:
ORA-47306: 20001: Access denied: You do not have SELECT privileges
```

Agora vamos atribuir a role aos usuários:

```bash
SQL> conn / as sysdba 
Connected.
SQL> grant DBV_SELECT to joao ;

Grant succeeded.

SQL> grant DBV_SELECT to maria ; 

Grant succeeded.
```

> "Mas Diogo, você concedeu o acesso da role pelo usuário SYS. Qualquer pessoa com acesso ao usuário SYS poderia conceder a role e obter acesso não autorizado."

Sim, isso é verdade. No entanto, no DBV, você pode limitar quem pode conceder **grants** e até configurar **REALMs** para isolar o ambiente. Para não deixar este artigo muito extenso, estou mantendo o usuário SYS ainda com essa "permissão". Porém, em um próximo artigo, mostrarei como configurar o DBV para restringir a concessão de **grants** a usuários específicos e como criar e gerenciar **REALMs**.

Agora vamos ver se os usuários voltaram acessar os dados:

```bash
SQL> 
SQL> 
SQL> conn maria 
Enter password: 
Connected.
SQL> select * from app.salario ;

        ID NOME                                                  SALARIO
---------- --------------------------------------------------  ------------
         1 João Silva                                              3000
         2 Maria Oliveira                                          4500
         3 Carlos Santos                                           5000
         4 Ana Costa                                               3200
         5 Pedro Almeida                                           2700
         6 Luiza Ferreira                                          6100
         7 Fernanda Lima                                           3800
         8 Roberto Faria                                           2900
         9 Cláudia Mendes                                          5300
        10 Tiago Barbosa                                           4700

10 rows selected.

SQL> conn joao
Enter password: 
Connected.
SQL> select * from app.salario ;

        ID NOME                                                  SALARIO
---------- -------------------------------------------------- --------------
         1 João Silva                                               3000
         2 Maria Oliveira                                           4500
         3 Carlos Santos                                            5000
         4 Ana Costa                                                3200
         5 Pedro Almeida                                            2700
         6 Luiza Ferreira                                           6100
         7 Fernanda Lima                                            3800
         8 Roberto Faria                                            2900
         9 Cláudia Mendes                                           5300
        10 Tiago Barbosa                                            4700

10 rows selected.
```

Agora vamos implementar a regr[a DML e de](https://www.linkedin.com/pulse/oracle-database-vault-uma-verdadeira-muralha-para-seu-diogo-fernandes-lhhaf/)i[xar soment](https://www.linkedin.com/pulse/instalando-database-vault-diogo-fernandes-pa2df/)e a Maria com permissões de alterar os dados.

```bash
SQL> show user
USER is "DBV_ADMIN"
SQL> 
SQL> 
SQL> 
SQL> BEGIN
  DVSYS.DBMS_MACADM.CREATE_RULE_SET(
    rule_set_name    => 'RULE_SET_DML',
    description      => 'Rule Set enabled for DML operations',
    enabled          => DVSYS.DBMS_MACUTL.G_YES,
    eval_options     => DBMS_MACUTL.G_RULESET_EVAL_ALL,
    audit_options    => DBMS_MACUTL.G_RULESET_AUDIT_FAIL,
    fail_options     => DBMS_MACUTL.G_RULESET_FAIL_SHOW,
    fail_message     => 'Access denied: You do not have DML privileges.',
    fail_code        => -20002,
    handler_options  => DBMS_MACUTL.G_RULESET_HANDLER_OFF,
    handler          => NULL
  );
END;
/  2    3    4    5    6    7    8    9   10   11   12   13   14   15  

PL/SQL procedure successfully completed.

SQL> BEGIN
  DVSYS.DBMS_MACADM.CREATE_COMMAND_RULE(
    command         => 'INSERT',
    rule_set_name   => 'RULE_SET_DML',
    object_owner    => 'APP',
    object_name     => 'SALARIO',
    enabled         => DVSYS.DBMS_MACUTL.G_YES
  );
END;
/  2    3    4    5    6    7    8    9   10  

PL/SQL procedure successfully completed.

SQL> BEGIN
  DVSYS.DBMS_MACADM.CREATE_COMMAND_RULE(
    command         => 'UPDATE',
    rule_set_name   => 'RULE_SET_DML',
    object_owner    => 'APP',
    object_name     => 'SALARIO',
    enabled         => DVSYS.DBMS_MACUTL.G_YES
  );
END;
/  2    3    4    5    6    7    8    9   10  

PL/SQL procedure successfully completed.

SQL> BEGIN
  DVSYS.DBMS_MACADM.CREATE_COMMAND_RULE(
    command         => 'DELETE',
    rule_set_name   => 'RULE_SET_DML',
    object_owner    => 'APP',
    object_name     => 'SALARIO',
    enabled         => DVSYS.DBMS_MACUTL.G_YES
  );
END;
/  2    3    4    5    6    7    8    9   10  

PL/SQL procedure successfully completed.

SQL> BEGIN
  DVSYS.DBMS_MACADM.CREATE_RULE(
    rule_name => 'RULE_DML',
    rule_expr => 'DVSYS.DBMS_MACUTL.USER_HAS_ROLE_VARCHAR(''DBV_DML'',''"''||dvsys.dv_login_user||''"'') = ''Y'''
  );
END;
/  2    3    4    5    6    7  

PL/SQL procedure successfully completed.

SQL> 
SQL> BEGIN
  DVSYS.DBMS_MACADM.ADD_RULE_TO_RULE_SET(
    rule_set_name => 'RULE_SET_DML',
    rule_name     => 'RULE_DML'
  );
END;
/  2    3    4    5    6    7  

PL/SQL procedure successfully completed.
```

Concedendo a permissão apenas pa[ra Maria](https://www.linkedin.com/pulse/oracle-database-vault-uma-verdadeira-muralha-para-seu-diogo-fernandes-lhhaf/):

```bash
SQL> grant DBV_DML to maria ; 

Grant succeeded.

SQL> conn joao 
Enter password: 
Connected.

SQL> show user
USER is "JOAO"

SQL> delete from app.salario where id=4 ; 
delete from app.salario where id=4
                *
ERROR at line 1:
ORA-47306: 20002: Access denied: You do not have DML privileges.


SQL> update app.salario set nome='Diogo' where id=4 ; 
update app.salario set nome='Diogo' where id=4
           *
ERROR at line 1:
ORA-47306: 20002: Access denied: You do not have DML privileges.


SQL> insert into app.salario values (11,'Diogo',1000) ; 
insert into app.salario values (11,'Diogo',1000)
                *
ERROR at line 1:
ORA-47306: 20002: Access denied: You do not have DML privileges.

SQL> conn maria
Enter password: 
Connected.
SQL> show user
USER is "MARIA"
SQL> delete from app.salario where id=4 ; 

1 row deleted.

SQL> update app.salario set nome='Diogo' where id=5 ; 

1 row updated.

SQL>  insert into app.salario values (11,'Diogo',1000) ; 

1 row created.

SQL> commit ;

Commit complete.
```

O Database Vault é ótimo em "delimitar" espaço, uma vez que as regras de acesso aos dados são bem desenhadas com a equipe de compliance e segurança. As restrições podem ser implementadas com sucesso no seu ambiente. Esta é uma pequena demonstração de como o DBV atua. Como sempre, recomendo que, valide exaustivamente no seu ambiente de homologação antes de aplicar em produção.