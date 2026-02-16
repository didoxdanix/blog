---
title: "ExaCC – Discos Offline após Aplicação de Patch"
datePublished: Mon Apr 21 2025 02:14:23 GMT+0000 (Coordinated Universal Time)
cuid: cm9qg0e6c000109iidvsy3tda
slug: exacc-discos-offline-apos-aplicacao-de-patch
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1745201821970/fee74f15-92f4-41da-bd92-b5f6f731084f.png
tags: oracle, oracle-database, oraclecloudinfrastructure

---

Caso ainda não esteja familiarizado com o ExaCC, segue abaixo uma explicação rápida sobre como funciona o gerenciamento e aplicação de patches.

O ExaCC, por ser uma solução virtualizada, possui duas camadas principais:

* **Dom0**: camada base (bare metal), responsável pelo gerenciamento físico dos recursos.
    
* **DomU**: camada virtual onde, de fato, os bancos de dados residem e operam.
    

A aplicação de patches no **Dom0** é de responsabilidade exclusiva da Oracle, enquanto os patches no **DomU** são de responsabilidade do cliente (nossa).

Sabendo disso, em uma das janelas de manutenção programadas pela Oracle para aplicação de patch no Dom0 — que sempre haviam sido concluídas com sucesso — ocorreu um incidente em que **seis discos ficaram offline no ASM** ao final do processo.

Assim que concluiu o patch, seis discos do grupo RECO ficaram offline e ficavam oscilando entre seis discos offline e um disco. Achei esse comportamento estranho. Minha primeira ação foi verificar os discos presencialmente para ver se havia algum aviso, e chegando lá, tudo estava "verde", ou seja, tudo ok.

```plaintext
ASMCMD> lsdg
State    Type  Rebal  Sector  Logical_Sector  Block       AU   Total_MB    Free_MB  Req_mir_free_MB  Usable_file_MB  Offline_disks  Voting_files  Name
MOUNTED  HIGH  N         512             512   4096  4194304  000000000  000000000                0        000000000              0             Y  DATAC1/
MOUNTED  HIGH  N         512             512   4096  4194304  000000000   000000000               0        000000000              6             N  RECOC1/
```

Analisando o log do ASM encontrei o seguinte aviso:

```plaintext
ORA-15025: could not open disk "/dev/exadata_quorum/QD_RECOC1_XXXXXXXXXXX"
ORA-27041: unable to open file
Linux-x86_64 Error: 2: No such file or directory
Additional information: 3
WARNING: Read Failed. group:2 disk:18 AU:0 offset:0 size:4096
```

Sem delongas abri um chamado na Oracle informando todos os logs e o contexto do incidente e eles pediram pra executar esse comando:

```plaintext
asmcmd lsdsk |grep exadata_quorum|xargs ls -l
```

Ao executar tive a seguinte saída em 1 dos nós:

```plaintext

[grid@sv-xxxxxxxxxx ~]$ asmcmd lsdsk |grep exadata_quorum|xargs ls -l
ls: cannot access /dev/exadata_quorum/QD_RECOC1_XXXXXXXXXXX01: No such file or directory
lrwxrwxrwx 1 root root 8 Mar 25 12:25 /dev/exadata_quorum/QD_DATAC1_XXXXXXXXXXX02 -> …/dm-18
lrwxrwxrwx 1 root root 8 Mar 25 12:25 /dev/exadata_quorum/QD_DATAC1_XXXXXXXXXXX03 -> …/dm-16
lrwxrwxrwx 1 root root 8 Mar 25 12:25 /dev/exadata_quorum/QD_RECOC1_XXXXXXXXXXX04 -> …/dm-17
```

No nó 2, todos os discos do quorum estavam visíveis; já no nó 1, um dos membros não era exibido.

Ja com bastantes informações coletadas, nossos amigos indianos nos passaram a seguinte nota:

**Infrastructure Maintenance Fails During Cell Update Due to Missing Quorum Disk Device Links in a VM (Doc ID 2978693.1)**

A nota pede a execução dos seguintes comandos para os quorum discos serem “reapresentados ao O.S”

```plaintext
#udevadm control --reload-rules
#udevadm trigger
```

Após a execução do comando o diskgroup ja “flegou” o Rebal = Y e ja baixou os discos offline para 1

```plaintext
ASMCMD> lsdg
State    Type  Rebal  Sector  Logical_Sector  Block       AU   Total_MB    Free_MB  Req_mir_free_MB  Usable_file_MB  Offline_disks  Voting_files  Name
MOUNTED  HIGH  N         512             512   4096  4194304  000000000  000000000                0        000000000              0             Y  DATAC1/
MOUNTED  HIGH  Y         512             512   4096  4194304  000000000   000000000               0        000000000              1             N  RECOC1/
```

Depois de 2 horas, todos os discos estavam online novamente.

```plaintext
ASMCMD> lsdg
State    Type  Rebal  Sector  Logical_Sector  Block       AU   Total_MB    Free_MB  Req_mir_free_MB  Usable_file_MB  Offline_disks  Voting_files  Name
MOUNTED  HIGH  N         512             512   4096  4194304  000000000  000000000                0        000000000              0             Y  DATAC1/
MOUNTED  HIGH  N         512             512   4096  4194304  000000000   000000000               0        000000000              0             N  RECOC1/
```

Assim como a nota menciona, infelizmente o ambiente estava em uma das versões causadas pelo bug:

Exadata image version is 22.1.10.0.0, 22.1.13.0.0, 22.1.14.0.0, or 22.1.15.0.0.

O procedimento, como a nota reforça, pode ser executado “online” sem impacto a produção.

Espero que este artigo seja útil em situações futuras 🙂.