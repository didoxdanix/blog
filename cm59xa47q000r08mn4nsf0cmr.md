---
title: "Modo Simulação"
datePublished: Sun Dec 29 2024 18:06:56 GMT+0000 (Coordinated Universal Time)
cuid: cm59xa47q000r08mn4nsf0cmr
slug: modo-simulacao
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1735495579461/3ef5f11b-bc62-462e-9d1f-0bda67d65704.png
tags: oracle, oracle-database, oracledatabasevault

---

Ok, você foi lá, fez toda a captura de privilégios usando o DBMS\_PRIVILEGE\_CAPTURE e outros métodos, mas mesmo assim fica aquela dúvida: E se aparecer algo novo?

Ao implementar regras com o Database Vault, essa é uma das principais preocupações. Porém, essa inquietação desaparece quando se descobre o modo simulation.

O modo simulation permite implementar uma security rule e colocar as command rule pertencente a ela em modo de simulação. Assim, se algum aplicativo violar a regra que você implementou, ele ainda conseguirá acessar os dados normalmente, mas você poderá identificar a violação por meio dos logs de auditoria. Esses logs mostram quem "violou" a regra que você mapeou anteriormente. Como você fez um longo trabalho de mapeamento, em teoria, apenas as exceções devem aparecer.

Com o modo simulation, você pode deixar a regra rodando durante uma semana, por exemplo, e depois verificar na view de auditoria as possíveis violações. Após esse período de testes, será possível ativar a regra de forma definitiva, sem o risco de interromper a produção.

Agora, vamos ver como isso funciona na prática.

No [artigo anterior](https://diogofernandes.com.br/criando-rules-set-no-oracle-database-vault), João não podia modificar a tabela **salário**. Então, o que faremos será alterar a **command rule** para que fique em modo **simulation**. Dessa forma, ela não impedirá o usuário João de modificar os dados, mas notificará o evento na **view DBA\_DV\_SIMULATION\_LOG**.

> Uma informação importante é que é a **command rule** que fica em modo **simulation**, e não a **security rule**.

Verificando se tem algum registro na DBA\_DV\_SIMULATION\_LOG

```bash
SQL> show user
USER is "DBV_ADMIN"
SQL> select * from dba_dv_simulation_log order by timestamp desc ; 

no rows selected
```

Confirmando que o usuário João não tem acesso de alteração a tabela salario.

```bash
SQL> show user
USER is "JOAO"

SQL> select * from app.salario ;

        ID NOME                                                  SALARIO
---------- -------------------------------------------------- --------------
         1 João Silva                                              3000
         2 Maria Oliveira                                          4500
         3 Carlos Santos                                           5000
         5 Diogo                                                   2700
         6 Luiza Ferreira                                          6100
         7 Fernanda Lima                                           3800
         8 Roberto Faria                                           2900
         9 Claudia Mendes                                          5300
        10 Tiago Barbosa                                           4700

SQL> update app.salario set salario=10000 where id=5 ; 
update app.salario set salario=10000 where id=5
           *
ERROR at line 1:
ORA-47306: 20002: Access denied: You do not have DML privileges.
```

Aqui está claro que não temos acesso: ORA-47306: 20002: Access denied: You do not have DML privileges.

Agora vou colocar a command rule de update em modo simulation, caso queria entender melhor como eu as criei o link do artigo anterior está [aqui](https://www.linkedin.com/pulse/criando-rules-set-oracle-database-vault-diogo-fernandes-oal6f/?trackingId=d7DvOEAgTOaS7e%2BgrgApKA%3D%3D).

```bash
SQL> BEGIN
  DVSYS.DBMS_MACADM.UPDATE_COMMAND_RULE(
    command         => 'UPDATE',
    rule_set_name   => 'RULE_SET_DML',
    object_owner    => 'APP',
    object_name     => 'SALARIO',
    enabled         => DBMS_MACUTL.G_SIMULATION -- <<< aqui esta a flag de simutaion.
  );
END;
/  2    3    4    5    6    7    8    9   10  

PL/SQL procedure successfully completed. 
```

Verificando Status da command rule:

```bash
SQL> col command for a13
col RULE_SET_NAME for a20
col object_owner for a10
col object_name for a20
col enable for a4
select COMMAND, RULE_SET_NAME,OBJECT_OWNER,OBJECT_NAME,ENABLED 
from DBA_DV_COMMAND_RULE 
where object_owner='APP';
```

![](https://media.licdn.com/dms/image/v2/D4D12AQFpmA_Wuy0fcA/article-inline_image-shrink_1500_2232/article-inline_image-shrink_1500_2232/0/1733780944667?e=1741219200&v=beta&t=ZE_euWfYqV9HCUH4bcvTxniBMKDs5vZPfgIDtqhCXzw align="left")

Podemos ver que o campo enable esta com o valor do **S**imulation.

Agora, tentaremos realizar o update na tabela APP.SALARIO utilizando o usuário João.

```bash
SQL> show user
USER is "JOAO"
SQL> update app.salario set salario=10000 where id=5 ; 

1 row updated.

SQL> commit ; 

Commit complete.
```

Agora que o update foi realizado com sucesso, vamos dar uma olhada na view de auditoria do DBV.

```bash
SELECT 
    USERNAME,
    COMMAND,
    VIOLATION_TYPE,
    OBJECT_OWNER,
    OBJECT_NAME,
    SQLTEXT,
    DATABASE_IP,
    MACHINE
FROM DBA_DV_SIMULATION_LOG; 
```

![](https://media.licdn.com/dms/image/v2/D4D12AQEXsVFv11JlKQ/article-inline_image-shrink_1500_2232/article-inline_image-shrink_1500_2232/0/1733781647616?e=1741219200&v=beta&t=WwwHmse8g4YbMFQ8FZnUFGuJ2e2DprdSlAZ9YYLU8wM align="left")

![](https://media.licdn.com/dms/image/v2/D4D12AQExDKfyhfPEKw/article-inline_image-shrink_1500_2232/article-inline_image-shrink_1500_2232/0/1733781671973?e=1741219200&v=beta&t=jONiCYVoR07ZSXoeiFuoblfz0OcyHY-wlECr8vMU120 align="left")

Como podemos ver, o DBV registrou com sucesso o acesso do usuário João à tabela APP.SALARIO. Esse modo de simulação, como falado antes, é muito útil para mapear e identificar acessos que não foram previstos no mapeamento inicial das permissões, sendo que somente os usuários que não têm permissão(que violaram a regra do DBV) serão registrados na view de auditoria.

Como sempre, recomendo validar as regras de forma exaustiva nos ambientes de homologação.