---
title: "Como resolver o erro bash: fork: retry: Resource temporarily unavailable durante o I O Calibrate"
datePublished: Wed Oct 29 2025 17:17:45 GMT+0000 (Coordinated Universal Time)
cuid: cmhc9dtt5000002ifhnkbezmq
slug: como-resolver-o-erro-bash-fork-retry-resource-temporarily-unavailable-durante-o-i-o-calibrate
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1761758026501/9e87596c-73ff-4e3a-9294-9b429feea726.png
tags: bash-fork-retry-resource-temporarily-unavailable

---

Durante a execução do I/O Calibrate no Oracle 19.25 com Oracle Enterprise Linux 9 (OEL9), obtive a seguinte mensagem após o procedimento ter sido executado por 12 minutos.

```bash
SET SERVEROUTPUT ON
DECLARE
lat  INTEGER;
iops INTEGER;
mbps INTEGER;
BEGIN
DBMS_RESOURCE_MANAGER.CALIBRATE_IO (1, 10, iops, mbps, lat);
DBMS_OUTPUT.PUT_LINE ('max_iops = ' || iops);
DBMS_OUTPUT.PUT_LINE ('latency  = ' || lat);
DBMS_OUTPUT.PUT_LINE ('max_mbps = ' || mbps);
END;
/
```

Após aproximadamente 12 minutos, o erro abaixo passou a aparecer em qualquer comando executado no bash do Linux.

```bash
-bash: fork: retry: Resource temporarily unavailable
-bash: fork: retry: Resource temporarily unavailable
-bash: fork: retry: Resource temporarily unavailable
-bash: fork: retry: Resource temporarily unavailable
```

Inicialmente, considerei a hipótese de que a quantidade de processos tivesse excedido os limites configurados, porém, ao executar o comando:

```bash
[oracle@node1 limits.d]$ ps -ef | grep oracle | wc -l 
2413
```

Ok, em processos Oracle so tenho 2413 ativos, porque esta acontecendo isso…????

Mas outra coisa me chamou atenção no vmstat durante a execução/problema.

```bash
root@node1 ~]# vmstat 1 
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1 16449      0 418050816 226100 6115824    0    0  5457    16   58   56  1  1 60 37  0
 6 16446      0 418047904 226108 6116228    0    0 344089    89 132928 425784  3  2  0 95  0
 3 16454      0 418043136 226108 6116152    0    0 346177   634 119197 404523  2  2  0 95  0
 2 16455      0 418041888 226116 6116144    0    0 347153   236 82672 351030  0  2  0 98  0
 0 16442      0 418049952 226116 6116172    0    0 347193     2 82877 350616  0  2  0 98  0
 0 1293      0 418054656 226116 6116176    0    0 113793    13 64679 255913  0  2  0 98  0
 7 16440      0 417885696 226124 6116204    0    0 483856   622 149115 526313 11  4  2 83  0
 5 16443      0 417884416 226124 6116204    0    0 351673   633 140787 529131  6  3  0 91  0
 9 16441      0 417879456 226132 6116248    0    0 353833   322 141966 534006  6  3  0 91  0
 6 16448      0 417879200 226132 6116260    0    0 355265     2 142702 534330  6  3  0 90  0
 1 16451      0 417878208 226132 6116264    0    0 364186     2 147385 547224  7  3  0 90  0
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1761329820626/0c768d1b-d7b7-47a1-9133-d44655f161c1.png align="center")

Na saída do comando vmstat, o campo b, localizado sob a seção procs, representa o número de processos ou threads bloqueados que estão aguardando a conclusão de operações de I/O. No caso em análise, esse comportamento foi observado durante a execução do I/O Calibrate do Oracle, que realiza testes intensivos de leitura e escrita no storage para medir desempenho.

Em determinado momento, o valor do campo b atingiu 16.451 threads bloqueadas, indicando que milhares de threads do Oracle estavam simultaneamente em espera por I/O. Após ultrapassar a marca de 16.000, o ambiente começou a apresentar falhas, evidenciando um possível gargalo no subsistema de I/O. Depois de uma análise detalhada, identifiquei a causa raiz do problema…

```bash
 cat /etc/security/limits.d/oracle-database-preinstall-19c.conf

# oracle-database-preinstall-19c setting for nproc soft limit is 16384
oracle   soft   nproc    16384

# oracle-database-preinstall-19c setting for nproc hard limit is 16384
oracle   hard   nproc    16384
```

Pronto, aí estava o problema. No arquivo oracle-database-preinstall-19c.conf, o valor padrão é 16384, e durante o processo foram alocados 16451, ocasionando o erro de “fork”.

Diante disso, alterei os limites soft e hard para 32384, e o problema não voltou a ocorrer durante a execução do I/O Calibrate.

```bash
# refer orabug15971421 for more info.
oracle   soft   nproc    32384

# oracle-database-preinstall-19c setting for nproc hard limit is 16384
oracle   hard   nproc    32384
```

Pra garantir a aplicação eu reiniciei os hosts em modo rolling, apois o incremento dos valores o erro -bash: fork: retry: Resource temporarily unavailable nao ocorreu mais.

É isso pessoal, qualquer coisa só me chamar no [**LinkedIn**](https://www.linkedin.com/in/diogo-fernandess/) 🙂