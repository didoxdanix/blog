---
title: "Erro ao criar volume ACFS no Oracle linux 9.5"
datePublished: Tue Oct 21 2025 18:39:41 GMT+0000 (Coordinated Universal Time)
cuid: cmh0wsdo4000002jrgxog2km9
slug: erro-ao-criar-volume-acfs-no-oracle-linux-95
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1761071895237/9501313d-805e-426d-b87e-9011034bcb39.jpeg
tags: oracle, clsu-00107, acfsoracle

---

Durante a configuração de um ACFS (Oracle ASM Cluster File System) em um ambiente com Oracle Linux 9.5 com grid 19c, me deparei com um erro ao tentar formatar o volume ASM. O comando executado foi:

```bash
/sbin/mkfs -t acfs /dev/asm/volume-157
```

E o erro retornado foi o seguinte:

```bash
mkfs.acfs: CLSU-00107: operating system function: read; failed with error data: 5; at location: ORF_0
mkfs.acfs: CLSU-00101: operating system error message: Input/output error
mkfs.acfs: ACFS-00526: read of volume disk header failed
mkfs.acfs: ACFS-01004: /dev/asm/volume-157 was not formatted.
```

Esse erro impedia a criação do sistema de arquivos e, inicialmente, parecia estar relacionado a permissões ou configuração incorreta do ASM. No entanto, após uma análise mais detalhada, identifiquei que se tratava de um bug conhecido documentado pela Oracle na nota ACFS Incompatibility With OEL8/9 UEK7 Kernel And ASMlib v3 (Doc ID 3076268.1).

## E qual era a causa disso?

O problema está relacionado a uma incompatibilidade entre o **kernel UEK7** (usado nas versões mais recentes do Oracle Linux 8 e 9) e o driver **ASMlib v3**.

Essa incompatibilidade afeta diretamente o processo de leitura do cabeçalho do volume durante o mkfs.acfs.

De acordo com a nota 3076268.1, as versões afetadas do Oracle Database RU (DBRU) incluem:

* 19.23.0.0.0 DBRU
    
* 19.24.0.0.0 DBRU
    
* 19.25.0.0.0 DBRU
    
* 19.26.0.0.0 DBRU
    
* 19.27.0.0.0 DBRU
    

Ou seja, todas essas versões exigem a aplicação do patch para garantir compatibilidade plena do ACFS com o kernel mais recente do Oracle Linux.

## **Solução**

A Oracle disponibilizou um patch corretivo (Patch 37405185) que resolve a incompatibilidade entre o ACFS, o kernel UEK7 e o ASMlib v3.

Antes da aplicação, é altamente recomendado realizar uma análise com o OPatchAuto:

```bash
$ORACLE_HOME/OPatch/opatchauto apply /tmp/37405185/  -oh /u01/app/19.0.0/grid -analyze
$ORACLE_HOME/OPatch/opatchauto apply /tmp/37405185/  -oh /u01/app/19.0.0/db -analyze
```

Se não houver conflitos, prossiga com a aplicação do patch. (Faça a análise para o grid\_home e db\_home.)

Em caso de RAC apliquem em todos os nós….

Aplicação do patch no Grid Infrastructure

```bash
[root@node1 tmp]# $ORACLE_HOME/OPatch/opatchauto apply /tmp/37405185/ -oh /u01/app/19.0.0/grid
```

Aplicação do patch no Oracle Database Home

```bash
[root@node1 tmp]# $ORACLE_HOME/OPatch/opatchauto apply /tmp/37405185/ -oh /u01/app/19.0.0/db
```

Apos a aplicação dos patches, a criação e montagem do ACFS foram concluídas normalmente no Oracle Linux 9.5, comprovando que o bug foi resolvido com a aplicação do patch 37405185.

```bash
/dev/asm/volume-157   30G  559M   30G   2% /VOLUME
```

## **Conclusão**

Esse caso reforça a importância de consultar a base de conhecimento da Oracle (MOS) sempre que um erro aparentemente genérico ocorre, principalmente em ambientes novos ou atualizados (como Oracle Linux 9.5 com UEK7).

Após aplicar o patch 37405185, o ACFS voltou a funcionar normalmente em ambiente Oracle 19.25.0.0.0 DBRU + Oracle Linux 9.5 (UEK7).

É isso pessoal, qualquer coisa só me chamar no [**LinkedIn**](https://www.linkedin.com/in/diogo-fernandess/) 🙂