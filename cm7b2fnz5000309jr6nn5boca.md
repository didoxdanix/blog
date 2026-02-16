---
title: "Múltiplos EXTTRAILs (DIRDAT) no mesmo Extract."
datePublished: Tue Feb 18 2025 22:38:24 GMT+0000 (Coordinated Universal Time)
cuid: cm7b2fnz5000309jr6nn5boca
slug: multiplos-exttrails-dirdat-no-mesmo-extract
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1739968547935/b8c1a772-ceac-4fe1-8d3a-5f3958bb7ab5.gif
tags: oracle, oracle-database, goldengate

---

Muitas vezes, ao configurar o extract, assumimos, de forma equivocada, que ele pode ter apenas um único EXTTRAIL (dirdat). Eu mesmo já tive essa impressão, pois muitos exemplos na documentação e em artigos seguem essa abordagem. Com isso, por hábito, acabamos incluindo vários owners dentro do mesmo extract, utilizando um único trail.

E foi assim até o dia em que me deparei com uma base que gerava, sabe-se lá quantos *dirdat* por minuto. O problema é que, por causa disso, os outros **owners**, que tinham poucas transações, começaram a apresentar um grande volume de **lag** nos seus **replicats**. No entanto, o **lag** não era causado pela quantidade de transações, e sim pelo grande volume de *dirdat* que o **replicat** precisava percorrer. Cada um desses *dirdat* continha poucas transações dos **owners** que geravam menos atividade, mas, devido ao alto volume geral, o processamento ficava atrasado.

A questão é: como resolver isso? Como separar os *dirdat* de forma que cada **owner** tenha suas operações registradas separadamente?

Primeiro, vou mostrar como o "padrão" é implementado e, em seguida, demonstrarei como realizar essa separação,

Criando o arquivo de parâmetro e\_prod.prm

```bash
GGSCI (777bca39a994) 2> dblogin USERIDALIAS ORIGEM19C
Successfully logged into database CDB$ROOT.

GGSCI (777bca39a994 as C##OGG@SRC19EE/CDB$ROOT) 5> edit param e_prod

EXTRACT E_PROD
USERIDALIAS ORIGEM19C



EXTTRAIL ./dirdat/aa

TABLE SRC19EEPDB1.OWNER1.*;
TABLE SRC19EEPDB1.OWNER2.*;
TABLE SRC19EEPDB1.OWNER3.*;
```

Vamos registrar e adicionar o Extract:

```bash
GGSCI (777bca39a994 as C##OGG@SRC19EE/CDB$ROOT) 10> REGISTER EXTRACT E_PROD DATABASE CONTAINER (SRC19EEPDB1) ;

2025-02-18 19:36:19  INFO    OGG-02003  Extract E_PROD successfully registered with database at SCN 21184522.

GGSCI (777bca39a994 as C##OGG@SRC19EE/CDB$ROOT) 11> ADD EXTRACT E_PROD, INTEGRATED TRANLOG, BEGIN NOW
EXTRACT (Integrated) added.

GGSCI (777bca39a994 as C##OGG@SRC19EE/CDB$ROOT) 12> ADD EXTTRAIL ./dirdat/aa, EXTRACT E_PROD
EXTTRAIL added.
```

Obtendo informações do Extract:

```bash
GGSCI (777bca39a994 as C##OGG@SRC19EE/CDB$ROOT) 13> info e_prod

EXTRACT    E_PROD    Initialized   2025-02-18 19:36   Status STOPPED
Checkpoint Lag       00:00:00 (updated 00:01:44 ago)
Log Read Checkpoint  Oracle Integrated Redo Logs
                     2025-02-18 19:36:52
                     SCN 0.0 (0)
```

Iniciando Extract:

```bash
GGSCI (777bca39a994 as C##OGG@SRC19EE/CDB$ROOT) 14> start e_prod

Sending START request to MANAGER ...
EXTRACT E_PROD starting

GGSCI (777bca39a994) 3> info e_prod, detail

EXTRACT    E_PROD    Last Started 2025-02-18 19:45   Status RUNNING
Checkpoint Lag       00:00:00 (updated 00:00:09 ago)
Process ID           78817
Log Read Checkpoint  Oracle Integrated Redo Logs
                     2025-02-18 19:46:14
                     SCN 0.21225987 (21225987)

  Target Extract Trails:

  Trail Name                                       Seqno        RBA     Max MB Trail Type

  ./dirdat/aa                                          0     409753        500 EXTTRAIL
```

Vamos popular as tabelas dos respectivos owners e acompanhar a gravação:

```bash
SQL> BEGIN
    FOR i IN 1001..2000 LOOP
        INSERT INTO OWNER1.TABELA_EXEMPLO (ID, NOME, DATA_CRIACAO)
        VALUES (i, 'Registro ' || i, SYSDATE - DBMS_RANDOM.VALUE(1, 365));
        
        INSERT INTO OWNER2.TABELA_EXEMPLO (ID, NOME, DATA_CRIACAO)
        VALUES (i, 'Registro ' || i, SYSDATE - DBMS_RANDOM.VALUE(1, 365));
         2   
        INSERT INTO OWNER3.TABELA_EXEMPLO (ID, NOME, DATA_CRIACAO)
        VALUES (i, 'Registro ' || i, SYSDATE - DBMS_RANDOM.VALUE(1, 365));
        
        -- Faz o commit a cada 100 registros para evitar consumo excessivo de UNDO
        IF MOD(i, 100) = 0 THEN
  3              COMMIT;
        END IF;
    END LOOP;
    
    COMMIT;
END;
/  4    5    6    7    8    9   10   11   12   13   14   15   16   17   18   19   20  

PL/SQL procedure successfully completed.
```

Podemos ver que todos os registros foram para um único dirdat, o “aa”.

```bash
[oracle@777bca39a994 dirdat]$ ls -lrt
total 808
-rw-r----- 1 oracle oinstall 824151 Feb 18 19:50 aa000000000
```

Nesse momento, o Extract está operando dessa forma:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1739915787220/2e1c940c-e888-4108-b0b0-1e5ebade2d58.gif align="center")

Como podemos ver acima, o extract está escrevendo a captura dos 3 owners em um único arquivo `./dirdat/aa`. A seguir, vamos ver como separar para que cada owner escreva em um dirdat separado.

Agora vamos adicionar o Extract de maneira que cada owner “escreva“ no seu próprio dirdat.

```bash
GGSCI (777bca39a994) 2> edit param e_prod

EXTRACT E_PROD
USERIDALIAS ORIGEM19C



EXTTRAIL ./dirdat/aa
TABLE SRC19EEPDB1.OWNER1.*;

EXTTRAIL ./dirdat/ab
TABLE SRC19EEPDB1.OWNER2.*;

EXTTRAIL ./dirdat/ac
TABLE SRC19EEPDB1.OWNER3.*;
```

Agora vamos adicionar o Extract

```bash
GGSCI (777bca39a994 as C##OGG@SRC19EE/CDB$ROOT) 4> REGISTER EXTRACT E_PROD DATABASE CONTAINER (SRC19EEPDB1) ;

2025-02-18 20:09:24  INFO    OGG-02003  Extract E_PROD successfully registered with database at SCN 21234029.


GGSCI (777bca39a994 as C##OGG@SRC19EE/CDB$ROOT) 5> ADD EXTRACT E_PROD, INTEGRATED TRANLOG, BEGIN NOW
EXTRACT (Integrated) added.
```

Agora, assim como separamos os exttrail no arquivo de parâmetros, devemos adicionar os 3 ao extract também.

```bash
GGSCI (777bca39a994 as C##OGG@SRC19EE/CDB$ROOT) 6> ADD EXTTRAIL ./dirdat/aa, EXTRACT E_PROD
EXTTRAIL added.

GGSCI (777bca39a994 as C##OGG@SRC19EE/CDB$ROOT) 7> ADD EXTTRAIL ./dirdat/ab, EXTRACT E_PROD
EXTTRAIL added.

GGSCI (777bca39a994 as C##OGG@SRC19EE/CDB$ROOT) 8> ADD EXTTRAIL ./dirdat/ac, EXTRACT E_PROD
```

Pronto, agora vamos iniciar o Extract.

```bash
GGSCI (777bca39a994 as C##OGG@SRC19EE/CDB$ROOT) 9> start e_prod

Sending START request to MANAGER ...
EXTRACT E_PROD starting

GGSCI (777bca39a994 as C##OGG@SRC19EE/CDB$ROOT) 10> info e_prod

EXTRACT    E_PROD    Last Started 2025-02-18 20:12   Status RUNNING
Checkpoint Lag       00:03:22 (updated 00:00:01 ago)
Process ID           78851
Log Read Checkpoint  Oracle Integrated Redo Logs
                     2025-02-18 20:09:34
                     SCN 0.0 (0)


GGSCI (777bca39a994 as C##OGG@SRC19EE/CDB$ROOT) 11> info e_prod, detail

EXTRACT    E_PROD    Last Started 2025-02-18 20:12   Status RUNNING
Checkpoint Lag       00:03:22 (updated 00:00:06 ago)
Process ID           78851
Log Read Checkpoint  Oracle Integrated Redo Logs
                     2025-02-18 20:09:34
                     SCN 0.0 (0)

  Target Extract Trails:

  Trail Name                                       Seqno        RBA     Max MB Trail Type

  ./dirdat/aa                                          0       1289        500 EXTTRAIL  
  ./dirdat/ab                                          0       1289        500 EXTTRAIL  
  ./dirdat/ac                                          0       1289        500 EXTTRAIL
```

Pronto, agora vamos popular as tabelas de origem novamente para vermos como serão gerados os dirdats:

```bash
SQL> BEGIN
    FOR i IN 2001..3000 LOOP
        INSERT INTO OWNER1.TABELA_EXEMPLO (ID, NOME, DATA_CRIACAO)
        VALUES (i, 'Registro ' || i, SYSDATE - DBMS_RANDOM.VALUE(1, 365));
        
        INSERT INTO OWNER2.TABELA_EXEMPLO (ID, NOME, DATA_CRI  2  ACAO)
        VALUES (i, 'Registro ' || i, SYSDATE - DBMS_RANDOM.VALUE(1, 365));
        
        INSERT INTO OWNER3.TABELA_EXEMPLO (ID, NOME, DATA_CRIACAO)
        VALUES (i, 'Registro ' || i, SYSDATE - DBMS_RANDOM.VALUE(1, 365));
        
        -- Faz o commit a cada 100 registros para evitar consumo excessivo de UNDO
  3          IF MOD(i, 100) = 0 THEN
            COMMIT;
        END IF;
    END LOOP;
    
    COMMIT;
END;
/  4    5    6    7    8    9   10   11   12   13   14   15   16   17   18   19   20  

PL/SQL procedure successfully
```

Verificando os dirdats gerados:

```bash
GGSCI (777bca39a994 as C##OGG@SRC19EE/CDB$ROOT) 12> exit
[oracle@777bca39a994 dirdat]$ ls -lrt
total 420
-rw-r----- 1 oracle oinstall 140230 Feb 18 20:21 ac000000000
-rw-r----- 1 oracle oinstall 140230 Feb 18 20:21 ab000000000
-rw-r----- 1 oracle oinstall 140230 Feb 18 20:21 aa000000000
```

Pronto, agora nosso Extract está operando dessa forma:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1739916154136/908b14fb-d9e3-46a1-a5c9-64f8545984c3.gif align="center")

Na imagem acima, agora podemos ver que o OWNER1 está “escrevendo” no `./dirdat/aa` o OWNER2 no `./dirdat/ab` e o OWNER3 no `./dirdat/ac`

É isso, pessoal, espero que este artigo tenha contribuído de alguma forma para o seu projeto atual de OGG ou nos próximos 🙂.