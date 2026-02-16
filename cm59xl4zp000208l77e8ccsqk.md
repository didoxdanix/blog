---
title: "Expandindo Storage no ODA X8M"
datePublished: Sun Dec 29 2024 18:15:30 GMT+0000 (Coordinated Universal Time)
cuid: cm59xl4zp000208l77e8ccsqk
slug: expandindo-storage-no-oda-x8m
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1735495979813/caf92469-ff2e-4659-ab96-47a932b14cbd.png
tags: oracle, oracle-database, oracledatabaseappliance, oda

---

Expandir o storage em alguns appliances pode ser um pouco complicado. Mas esse não é o caso do ODA (isso, é claro, desde que você não se depare com um bug no caminho rs).

No artigo de hoje, vamos expandir um ODA que já estava com 8 discos e que agora será expandido para 12 discos, atingindo assim sua capacidade total.

Sem mais delongas, vamos para a prática!

![Article content](https://media.licdn.com/dms/image/v2/D4D12AQFu53x4F98yTA/article-inline_image-shrink_1500_2232/article-inline_image-shrink_1500_2232/0/1734370248247?e=1759968000&v=beta&t=wZcuNuGkVtKbF8YERBR38ixjSJ_KvLu8xVZ6NxhsgaY align="left")

Acima está o mapa das posições dos discos do ODA X8M. A contagem das posições começa na parte inferior, da esquerda para a direita. No meu caso, apenas as duas primeiras fileiras estavam preenchidas com discos, e ficaram totalmente preenchidas após a adição dos outros quatro.

> Adicione um disco por vez, espere de 2 até 5 minutos até aparecer dentro de /dev/nvme\*
> 
> ![Article content](https://media.licdn.com/dms/image/v2/D4D12AQFAGKLmFAaFMQ/article-inline_image-shrink_1500_2232/article-inline_image-shrink_1500_2232/0/1734370664849?e=1759968000&v=beta&t=E-mVJYHqBdxLUgS8K_Z1AMoB9pHh85_vQx_SLfY2z90 align="left")

Podemos ver que, do disco 8 ao disco 11, ainda não temos as partições p1 e p2 presentes nos outros discos já configurados. Ótimo! Isso indica que os discos estão "limpos".

Ativando os discos.

```bash
[root@odaxxxx01 ~]# odaadmcli power disk on pd_08
Disk 'pd_08' already powered on
[root@odaxxxx01 ~]# odaadmcli power disk on pd_09
Disk 'pd_09' already powered on
[root@odaxxxx01 ~]# odaadmcli power disk on pd_10
Disk 'pd_10' already powered on
[root@odaxxxx01 ~]# odaadmcli power disk on pd_11
Disk 'pd_11' already powered on
```

> Após adicionar todos os discos, aguarde cerca de 20 minutos antes de iniciar a expansão. Esse tempo é necessário para que o OAK processe todas as informações sobre os discos.

Antes de expandir vamos olhar a posição atual dos discos.

```bash
[root@odaxxxx01 ~]# odaadmcli show disk
        NAME            PATH            TYPE            STATE           STATE_DETAILS

        pd_00           /dev/nvme0n1    NVD             ONLINE          Good
        pd_01           /dev/nvme2n1    NVD             ONLINE          Good
        pd_02           /dev/nvme6n1    NVD             ONLINE          Good
        pd_03           /dev/nvme4n1    NVD             ONLINE          Good
        pd_04           /dev/nvme1n1    NVD             ONLINE          Good
        pd_05           /dev/nvme3n1    NVD             ONLINE          Good
        pd_06           /dev/nvme7n1    NVD             ONLINE          Good
        pd_07           /dev/nvme5n1    NVD             ONLINE          Good
```

Sim, os discos pd\_08, pd\_09, pd\_10 e pd\_11 não estão aparecendo aqui. Nada de pânico! Eles só vão aparecer após o comando "expand storage". Então, vamos lá!

```bash
 [root@odaxxxx01 bin]# odaadmcli expand storage -ndisk 4
Precheck passed.
Check the progress of expansion of storage by executing 'odaadmcli show disk'
Waiting for expansion to finish ...
```

Durante o acompanhamento, veremos vários estágios dos discos, incluindo, algumas vezes, o status INVALID. Não entre em pânico! Apenas aguarde a tela de expansão ser concluída.

Durante o procedimento:

![Article content](https://media.licdn.com/dms/image/v2/D4D12AQEa5JwRpU4Njg/article-inline_image-shrink_1500_2232/article-inline_image-shrink_1500_2232/0/1734371885164?e=1759968000&v=beta&t=tQi0dZNyPeGJoRRXNHKZqbwgJ98T6T8bP6RasDQwghY align="left")

![Article content](https://media.licdn.com/dms/image/v2/D4D12AQFN06HkSBt8xw/article-inline_image-shrink_1500_2232/article-inline_image-shrink_1500_2232/0/1734371695191?e=1759968000&v=beta&t=0lzYrbnXeFG3qcDDE-tvkJH-5CPGmZC9t5KFFmPathI align="left")

![](https://media.licdn.com/dms/image/v2/D4D12AQEa5JwRpU4Njg/article-inline_image-shrink_1500_2232/article-inline_image-shrink_1500_2232/0/1734371885164?e=1741219200&v=beta&t=mp2nM2nW7vSyiMorhzLmLJwfwpMRHZWmcfa1VYfF5Mo align="left")

![](https://media.licdn.com/dms/image/v2/D4D12AQFN06HkSBt8xw/article-inline_image-shrink_1500_2232/article-inline_image-shrink_1500_2232/0/1734371695191?e=1741219200&v=beta&t=M09XQ0dkxn5jbI7aZdW01pE0RE5NlP3U4PuYWVoiwW0 align="left")

Assim que o procedimento conclui:

![Article content](https://media.licdn.com/dms/image/v2/D4D12AQEzb6joxyMgGQ/article-inline_image-shrink_1500_2232/article-inline_image-shrink_1500_2232/0/1734371724153?e=1759968000&v=beta&t=RAGqZKv0-oW-ewkzZMZ1rbOals4pbTNvO3yHs7yGEj0 align="left")

Agora podemos ver que todos os discos estão aparecendo no comando odaadmcli show disk. Isso indica que todos os discos foram configurados com sucesso.

O comando odaadmcli expand storage -ndisk 4, que foi utilizado para a expansão, será concluído automaticamente. Não feche até a finalização.

PS:(A parte do comando -ndisk 4, indica a quantidade de disco que vai expandir, se for apenas 2 discos, alterar o valor de 4 para 2, por exemplo.)

Uma vez concluído, o ASM iniciará o rebalance, e a expansão estará concluída.

![Article content](https://media.licdn.com/dms/image/v2/D4D12AQHMihxOqaNHUg/article-inline_image-shrink_1500_2232/article-inline_image-shrink_1500_2232/0/1734372176803?e=1759968000&v=beta&t=rqwj97SIH2bSvzvyLNHNXudr2xs63A00Jnr3fI55tKQ align="left")

Antes de iniciar, recomendo seguir estas três etapas:

1. Rode o comando ./orachk -nordbms (como root) para verificar se está tudo certo com o appliance.
    
2. Execute o comando odaadmcli show server e certifique-se de que o status está healthy (saudável).
    
3. Por último, mas não menos importante, antes de iniciar a expansão(10 dias antes ou até mais), abra um proactive SR no Oracle Support e explique todo o seu plano de expansão. Os engenheiros podem aprovar ou identificar alguma possível "falha" no seu procedimento.