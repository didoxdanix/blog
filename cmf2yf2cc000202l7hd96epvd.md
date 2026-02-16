---
title: "Ksplice: Configuração e Instalação em Ambientes Oracle Linux"
datePublished: Tue Sep 02 2025 19:41:27 GMT+0000 (Coordinated Universal Time)
cuid: cmf2yf2cc000202l7hd96epvd
slug: ksplice-configuracao-e-instalacao-em-ambientes-oracle-linux
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1756836676988/76b2de94-5886-4d2c-87e2-a504157b385f.png
tags: ksplice

---

Em um artigo anterior ([link aqui](https://diogofernandes.com.br/ksplice-no-exadata)), escrevi sobre como atualizar o Ksplice em um Exadata. Mas e quando falamos de máquinas “comuns”, como Dell, HP etc.? Em uma instalação simples, os pacotes do Ksplice não vêm configurados da mesma forma que nos Exadatas. Por isso, precisamos instalar tudo do zero e realizar o registro no ULN (Unbreakable Linux Network) para aplicar as correções das últimas CVEs.

Nos passos abaixo, vou mostrar como fazer todo o processo do zero: quais pacotes devem ser selecionados, tanto na parte do terminal quanto no site [linux.oracle.com](http://linux.oracle.com), para concluir o procedimento.

Vamos instalar agora o rhn-setup para registrarmos o sistema operacional e configurarmos o Ksplice.

```bash
[root@Oracle02 ~]# yum install rhn-setup -y 
Loaded plugins: langpacks, ulninfo
ol7_UEKR6                                                                                                                                                                                                                                                                                             | 3.0 kB  00:00:00     
ol7_latest                                                                                                                                                                                                                                                                                            | 3.6 kB  00:00:00     
(1/4): ol7_UEKR6/x86_64/updateinfo                                                                                                                                                                                                                                                                    | 1.3 MB  00:00:02     
(2/4): ol7_latest/x86_64/updateinfo                                                                                                                                                                                                                                                                   | 3.7 MB  00:00:04     
(3/4): ol7_latest/x86_64/primary_db                                                                                                                                                                                                                                                                   |  54 MB  00:00:41     
(4/4): ol7_UEKR6/x86_64/primary_db                                                                                                                                                                                                                                                                    |  83 MB  00:01:04     
Resolving Dependencies
--> Running transaction check
---> Package rhn-setup.x86_64 0:2.0.2-24.0.7.el7 will be updated
--> Processing Dependency: rhn-setup = 2.0.2-24.0.7.el7 for package: rhn-setup-gnome-2.0.2-24.0.7.el7.x86_64
---> Package rhn-setup.x86_64 0:2.0.2-24.0.11.el7 will be an update
--> Processing Dependency: rhn-client-tools = 2.0.2-24.0.11.el7 for package: rhn-setup-2.0.2-24.0.11.el7.x86_64
--> Running transaction check
---> Package rhn-client-tools.x86_64 0:2.0.2-24.0.7.el7 will be updated
--> Processing Dependency: rhn-client-tools = 2.0.2-24.0.7.el7 for package: rhn-check-2.0.2-24.0.7.el7.x86_64
---> Package rhn-client-tools.x86_64 0:2.0.2-24.0.11.el7 will be an update
---> Package rhn-setup-gnome.x86_64 0:2.0.2-24.0.7.el7 will be updated
---> Package rhn-setup-gnome.x86_64 0:2.0.2-24.0.11.el7 will be an update
--> Running transaction check
---> Package rhn-check.x86_64 0:2.0.2-24.0.7.el7 will be updated
---> Package rhn-check.x86_64 0:2.0.2-24.0.11.el7 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

=============================================================================================================================================================================================================================================================================================================================
 Package                                                                          Arch                                                                   Version                                                                            Repository                                                                  Size
=============================================================================================================================================================================================================================================================================================================================
Updating:
 rhn-setup                                                                        x86_64                                                                 2.0.2-24.0.11.el7                                                                  ol7_latest                                                                  94 k
Updating for dependencies:
 rhn-check                                                                        x86_64                                                                 2.0.2-24.0.11.el7                                                                  ol7_latest                                                                  58 k
 rhn-client-tools                                                                 x86_64                                                                 2.0.2-24.0.11.el7                                                                  ol7_latest                                                                 422 k
 rhn-setup-gnome                                                                  x86_64                                                                 2.0.2-24.0.11.el7                                                                  ol7_latest                                                                 160 k

Transaction Summary
=============================================================================================================================================================================================================================================================================================================================
Upgrade  1 Package (+3 Dependent packages)

Total size: 734 k
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Warning: RPMDB altered outside of yum.
  Updating   : rhn-client-tools-2.0.2-24.0.11.el7.x86_64                                                                                                                                                                                                                                                                 1/8 
  Updating   : rhn-setup-2.0.2-24.0.11.el7.x86_64                                                                                                                                                                                                                                                                        2/8 
  Updating   : rhn-setup-gnome-2.0.2-24.0.11.el7.x86_64                                                                                                                                                                                                                                                                  3/8 
  Updating   : rhn-check-2.0.2-24.0.11.el7.x86_64                                                                                                                                                                                                                                                                        4/8 
  Cleanup    : rhn-setup-gnome-2.0.2-24.0.7.el7.x86_64                                                                                                                                                                                                                                                                   5/8 
  Cleanup    : rhn-setup-2.0.2-24.0.7.el7.x86_64                                                                                                                                                                                                                                                                         6/8 
  Cleanup    : rhn-check-2.0.2-24.0.7.el7.x86_64                                                                                                                                                                                                                                                                         7/8 
  Cleanup    : rhn-client-tools-2.0.2-24.0.7.el7.x86_64                                                                                                                                                                                                                                                                  8/8 
  Verifying  : rhn-setup-gnome-2.0.2-24.0.11.el7.x86_64                                                                                                                                                                                                                                                                  1/8 
  Verifying  : rhn-check-2.0.2-24.0.11.el7.x86_64                                                                                                                                                                                                                                                                        2/8 
  Verifying  : rhn-setup-2.0.2-24.0.11.el7.x86_64                                                                                                                                                                                                                                                                        3/8 
  Verifying  : rhn-client-tools-2.0.2-24.0.11.el7.x86_64                                                                                                                                                                                                                                                                 4/8 
  Verifying  : rhn-setup-gnome-2.0.2-24.0.7.el7.x86_64                                                                                                                                                                                                                                                                   5/8 
  Verifying  : rhn-check-2.0.2-24.0.7.el7.x86_64                                                                                                                                                                                                                                                                         6/8 
  Verifying  : rhn-client-tools-2.0.2-24.0.7.el7.x86_64                                                                                                                                                                                                                                                                  7/8 
  Verifying  : rhn-setup-2.0.2-24.0.7.el7.x86_64                                                                                                                                                                                                                                                                         8/8 

Updated:
  rhn-setup.x86_64 0:2.0.2-24.0.11.el7                                                                                                                                                                                                                                                                                       

Dependency Updated:
  rhn-check.x86_64 0:2.0.2-24.0.11.el7                                                                 rhn-client-tools.x86_64 0:2.0.2-24.0.11.el7                                                                 rhn-setup-gnome.x86_64 0:2.0.2-24.0.11.el7                                                                

Complete!
[root@Oracle02 ~]#
```

Após instalar o rhn-setup, acesse o site [linux.oracle.com](http://linux.oracle.com), gere a chave de registro (Auth token) e guarde o codigo, pois ele irá parecer somente uma vez, essa chave vamos usar ela no campo senha, do proximo step.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756837962148/35075d2c-4965-4238-ab7a-27f1744301e4.png align="center")

Agora, vamos chamar o comando uln\_register

```bash
[root@Oracle02 ~]# uln_register
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756838766970/386ff695-ec41-4897-8a62-6e68c641ab95.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756838966228/4c223363-2d90-4cc8-9808-bfb156e81870.png align="center")

e vai aparecendo outras telas…

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756839013752/f1a7e121-a891-481a-a0ac-64124d6857e0.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756839061537/63914494-e91b-435a-86a9-ffea65a52280.png align="center")

e vai selecionando next até chegar nessa tela:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756839113477/b3a29109-095d-42d6-8b78-45a1253dabb5.png align="center")

next…

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756839147259/c62a3ca8-ff54-4e5b-b349-894c2a5b6617.png align="center")

next….

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756839171121/fc763234-2d2d-4a3e-8ce2-66ce53c103c1.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756839207167/b47abbbb-c030-4a03-9d52-121eadd622ad.png align="center")

Agora vamos acessar o site linux.oracle.com

A lista de hosts cadastrados estarão logo a abaixo. Acabamos de registrar o host O Oracle02 e vamos clicar nele:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756839340444/3e0ac957-b03f-4448-90bb-593c5cbc8f9c.png align="center")

Assim que clicar no host ela tera irá abrir:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756839560042/58380d8d-9fbc-4851-b526-84db9dc498cf.png align="center")

clique em Manage Subscritptions

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756839595311/eeac6a9c-bec7-47a2-85cb-3e70525053a1.png align="center")

Essa tela irá aparecer:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756839704936/3e86627b-10d0-4d34-9822-7c8b76819db8.png align="center")

assim que arrastar o ksplice aware pra direita clique em save subscription.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756839759914/9dfacb4a-410b-410a-984f-9d6fa9675fab.png align="center")

Chega de print, vamos agora para o terminal novamente.

Registro e gerenciamento de atualizações finalizado vamos instalar o ksplice.

```bash
[root@Oracle02 ~]# yum install ksplice uptrack
```

```bash
[root@Oracle02 ~]# yum install ksplice uptrack 
Loaded plugins: langpacks, rhnplugin, ulninfo
This system is receiving updates from ULN.
ol7_x86_64_ksplice                                                                                                                                                                                                                                                                                    | 3.0 kB  00:00:00     
ol7_x86_64_ksplice/updateinfo                                                                                                                                                                                                                                                                         | 9.6 kB  00:00:00     
ol7_x86_64_ksplice/primary_db                                                                                                                                                                                                                                                                         | 5.8 MB  00:00:03     
ol7_x86_64_userspace_ksplice                                                                                                                                                                                                                                                                          | 3.0 kB  00:00:00     
ol7_x86_64_userspace_ksplice/updateinfo                                                                                                                                                                                                                                                               |  90 kB  00:00:00     
ol7_x86_64_userspace_ksplice/primary_db                                                                                                                                                                                                                                                               | 328 kB  00:00:00     
Resolving Dependencies
--> Running transaction check
---> Package ksplice.x86_64 0:1.0.62-1.el7 will be installed
--> Processing Dependency: ksplice-core0 = 1.0.62-1.el7 for package: ksplice-1.0.62-1.el7.x86_64
--> Processing Dependency: ksplice-tools = 1.0.62-1.el7 for package: ksplice-1.0.62-1.el7.x86_64
---> Package uptrack.noarch 0:1.2.84-0.el7 will be installed
--> Processing Dependency: perl(Fatal) for package: uptrack-1.2.84-0.el7.noarch
--> Processing Dependency: perl-autodie for package: uptrack-1.2.84-0.el7.noarch
--> Running transaction check
---> Package ksplice-core0.x86_64 0:1.0.62-1.el7 will be installed
--> Processing Dependency: libboost_filesystem-mt.so.1.53.0()(64bit) for package: ksplice-core0-1.0.62-1.el7.x86_64
--> Processing Dependency: libboost_python-mt.so.1.53.0()(64bit) for package: ksplice-core0-1.0.62-1.el7.x86_64
--> Processing Dependency: libboost_regex-mt.so.1.53.0()(64bit) for package: ksplice-core0-1.0.62-1.el7.x86_64
---> Package ksplice-tools.x86_64 0:1.0.62-1.el7 will be installed
--> Processing Dependency: python-requests for package: ksplice-tools-1.0.62-1.el7.x86_64
---> Package perl-autodie.noarch 0:2.16-2.el7 will be installed
--> Running transaction check
---> Package boost-filesystem.x86_64 0:1.53.0-28.el7 will be installed
---> Package boost-python.x86_64 0:1.53.0-28.el7 will be installed
---> Package boost-regex.x86_64 0:1.53.0-28.el7 will be installed
---> Package python-requests.noarch 0:2.6.0-10.el7 will be installed
--> Processing Dependency: python-urllib3 >= 1.10.2-1 for package: python-requests-2.6.0-10.el7.noarch
--> Running transaction check
---> Package python-urllib3.noarch 0:1.10.2-7.0.1.el7 will be installed
--> Finished Dependency Resolution
Dependencies Resolved
=============================================================================================================================================================================================================================================================================================================================
 Package                                                                        Arch                                                                 Version                                                                          Repository                                                                        Size
=============================================================================================================================================================================================================================================================================================================================
Installing:
 ksplice                                                                        x86_64                                                               1.0.62-1.el7                                                                     ol7_x86_64_ksplice                                                                11 k
 uptrack                                                                        noarch                                                               1.2.84-0.el7                                                                     ol7_x86_64_ksplice                                                               157 k
Installing for dependencies:
 boost-filesystem                                                               x86_64                                                               1.53.0-28.el7                                                                    ol7_x86_64_latest                                                                 68 k
 boost-python                                                                   x86_64                                                               1.53.0-28.el7                                                                    ol7_x86_64_latest                                                                132 k
 boost-regex                                                                    x86_64                                                               1.53.0-28.el7                                                                    ol7_x86_64_latest                                                                295 k
 ksplice-core0                                                                  x86_64                                                               1.0.62-1.el7                                                                     ol7_x86_64_ksplice                                                               305 k
 ksplice-tools                                                                  x86_64                                                               1.0.62-1.el7                                                                     ol7_x86_64_ksplice                                                                94 k
 perl-autodie                                                                   noarch                                                               2.16-2.el7                                                                       ol7_x86_64_latest                                                                 77 k
 python-requests                                                                noarch                                                               2.6.0-10.el7                                                                     ol7_x86_64_latest                                                                 95 k
 python-urllib3                                                                 noarch                                                               1.10.2-7.0.1.el7                                                                 ol7_x86_64_latest                                                                102 k
Transaction Summary
=============================================================================================================================================================================================================================================================================================================================
Install  2 Packages (+8 Dependent packages)
Total download size: 1.3 M
Installed size: 5.6 M
Is this ok [y/d/N]: y
Downloading packages:
(1/10): boost-filesystem-1.53.0-28.el7.x86_64.rpm                                                                                                                                                                                                                                                     |  68 kB  00:00:00     
(2/10): boost-python-1.53.0-28.el7.x86_64.rpm                                                                                                                                                                                                                                                         | 132 kB  00:00:00     
(3/10): boost-regex-1.53.0-28.el7.x86_64.rpm                                                                                                                                                                                                                                                          | 295 kB  00:00:00     
(4/10): ksplice-1.0.62-1.el7.x86_64.rpm                                                                                                                                                                                                                                                               |  11 kB  00:00:00     
(5/10): ksplice-core0-1.0.62-1.el7.x86_64.rpm                                                                                                                                                                                                                                                         | 305 kB  00:00:00     
(6/10): ksplice-tools-1.0.62-1.el7.x86_64.rpm                                                                                                                                                                                                                                                         |  94 kB  00:00:00     
(7/10): perl-autodie-2.16-2.el7.noarch.rpm                                                                                                                                                                                                                                                            |  77 kB  00:00:00     
(8/10): python-requests-2.6.0-10.el7.noarch.rpm                                                                                                                                                                                                                                                       |  95 kB  00:00:00     
(9/10): python-urllib3-1.10.2-7.0.1.el7.noarch.rpm                                                                                                                                                                                                                                                    | 102 kB  00:00:00     
(10/10): uptrack-1.2.84-0.el7.noarch.rpm                                                                                                                                                                                                                                                              | 157 kB  00:00:00     
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                                                                                                        794 kB/s | 1.3 MB  00:00:01     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : boost-filesystem-1.53.0-28.el7.x86_64                                                                                                                                                                                                                                                                    1/10 
  Installing : boost-python-1.53.0-28.el7.x86_64                                                                                                                                                                                                                                                                        2/10 
  Installing : boost-regex-1.53.0-28.el7.x86_64                                                                                                                                                                                                                                                                         3/10 
  Installing : ksplice-core0-1.0.62-1.el7.x86_64                                                                                                                                                                                                                                                                        4/10 
  Installing : python-urllib3-1.10.2-7.0.1.el7.noarch                                                                                                                                                                                                                                                                   5/10 
  Installing : python-requests-2.6.0-10.el7.noarch                                                                                                                                                                                                                                                                      6/10 
  Installing : perl-autodie-2.16-2.el7.noarch                                                                                                                                                                                                                                                                           7/10 
  Installing : uptrack-1.2.84-0.el7.noarch                                                                                                                                                                                                                                                                              8/10 
  Installing : ksplice-tools-1.0.62-1.el7.x86_64                                                                                                                                                                                                                                                                        9/10 
  Installing : ksplice-1.0.62-1.el7.x86_64                                                                                                                                                                                                                                                                             10/10 
There are no existing modules on disk that need basename migration.
  Verifying  : python-requests-2.6.0-10.el7.noarch                                                                                                                                                                                                                                                                      1/10 
  Verifying  : perl-autodie-2.16-2.el7.noarch                                                                                                                                                                                                                                                                           2/10 
  Verifying  : ksplice-core0-1.0.62-1.el7.x86_64                                                                                                                                                                                                                                                                        3/10 
  Verifying  : ksplice-1.0.62-1.el7.x86_64                                                                                                                                                                                                                                                                              4/10 
  Verifying  : ksplice-tools-1.0.62-1.el7.x86_64                                                                                                                                                                                                                                                                        5/10 
  Verifying  : python-urllib3-1.10.2-7.0.1.el7.noarch                                                                                                                                                                                                                                                                   6/10 
  Verifying  : uptrack-1.2.84-0.el7.noarch                                                                                                                                                                                                                                                                              7/10 
  Verifying  : boost-regex-1.53.0-28.el7.x86_64                                                                                                                                                                                                                                                                         8/10 
  Verifying  : boost-python-1.53.0-28.el7.x86_64                                                                                                                                                                                                                                                                        9/10 
  Verifying  : boost-filesystem-1.53.0-28.el7.x86_64                                                                                                                                                                                                                                                                   10/10 
Installed:
  ksplice.x86_64 0:1.0.62-1.el7                                                                                                                                 uptrack.noarch 0:1.2.84-0.el7                                                                                                                                
Dependency Installed:
  boost-filesystem.x86_64 0:1.53.0-28.el7   boost-python.x86_64 0:1.53.0-28.el7   boost-regex.x86_64 0:1.53.0-28.el7   ksplice-core0.x86_64 0:1.0.62-1.el7   ksplice-tools.x86_64 0:1.0.62-1.el7   perl-autodie.noarch 0:2.16-2.el7   python-requests.noarch 0:2.6.0-10.el7   python-urllib3.noarch 0:1.10.2-7.0.1.el7  
Complete!
```

vamos atualizar as libs necessarias para o ksplice:

```bash
[root@Oracle02 ~]# yum --disablerepo=* --enablerepo=ol7_x86_64_userspace_ksplice update
```

Na parte de cima se atente a versao do seu O.S….  
OEL8= yum --disablerepo=\* --enablerepo=ol8\_x86\_64\_userspace\_ksplice update  
OEL9 = yum --disablerepo=\* --enablerepo=ol9\_x86\_64\_userspace\_ksplice update

```bash
[root@Oracle02 ~]# yum --disablerepo=* --enablerepo=ol7_x86_64_userspace_ksplice update
Loaded plugins: langpacks, rhnplugin, ulninfo
This system is receiving updates from ULN.
Resolving Dependencies
--> Running transaction check
---> Package glibc.x86_64 0:2.17-326.0.9.el7_9 will be updated
---> Package glibc.x86_64 2:2.17-326.0.9.ksplice1.el7_9.3 will be an update
--> Processing Dependency: ksplice-helper >= 1.0.51 for package: 2:glibc-2.17-326.0.9.ksplice1.el7_9.3.x86_64
--> Processing Dependency: ksplice-helper for package: 2:glibc-2.17-326.0.9.ksplice1.el7_9.3.x86_64
--> Processing Dependency: libksplice_helper.so()(64bit) for package: 2:glibc-2.17-326.0.9.ksplice1.el7_9.3.x86_64
---> Package glibc-common.x86_64 0:2.17-326.0.9.el7_9 will be updated
---> Package glibc-common.x86_64 2:2.17-326.0.9.ksplice1.el7_9.3 will be an update
---> Package glibc-devel.x86_64 0:2.17-326.0.9.el7_9 will be updated
---> Package glibc-devel.x86_64 2:2.17-326.0.9.ksplice1.el7_9.3 will be an update
---> Package glibc-headers.x86_64 0:2.17-326.0.9.el7_9 will be updated
---> Package glibc-headers.x86_64 2:2.17-326.0.9.ksplice1.el7_9.3 will be an update
---> Package openssl.x86_64 1:1.0.2k-19.0.1.el7 will be updated
---> Package openssl.x86_64 2:1.0.2k-26.ksplice1.el7_9 will be an update
---> Package openssl-libs.x86_64 1:1.0.2k-19.0.1.el7 will be updated
---> Package openssl-libs.x86_64 2:1.0.2k-26.ksplice1.el7_9 will be an update
--> Running transaction check
---> Package ksplice-helper.x86_64 0:1.0.62-1.el7 will be installed
--> Finished Dependency Resolution
Dependencies Resolved
=============================================================================================================================================================================================================================================================================================================================
 Package                                                                Arch                                                           Version                                                                                    Repository                                                                            Size
=============================================================================================================================================================================================================================================================================================================================
Updating:
 glibc                                                                  x86_64                                                         2:2.17-326.0.9.ksplice1.el7_9.3                                                            ol7_x86_64_userspace_ksplice                                                         3.7 M
 glibc-common                                                           x86_64                                                         2:2.17-326.0.9.ksplice1.el7_9.3                                                            ol7_x86_64_userspace_ksplice                                                          12 M
 glibc-devel                                                            x86_64                                                         2:2.17-326.0.9.ksplice1.el7_9.3                                                            ol7_x86_64_userspace_ksplice                                                         1.1 M
 glibc-headers                                                          x86_64                                                         2:2.17-326.0.9.ksplice1.el7_9.3                                                            ol7_x86_64_userspace_ksplice                                                         695 k
 openssl                                                                x86_64                                                         2:1.0.2k-26.ksplice1.el7_9                                                                 ol7_x86_64_userspace_ksplice                                                         494 k
 openssl-libs                                                           x86_64                                                         2:1.0.2k-26.ksplice1.el7_9                                                                 ol7_x86_64_userspace_ksplice                                                         1.2 M
Installing for dependencies:
 ksplice-helper                                                         x86_64                                                         1.0.62-1.el7                                                                               ol7_x86_64_userspace_ksplice                                                          21 k
Transaction Summary
=============================================================================================================================================================================================================================================================================================================================
Install             ( 1 Dependent package)
Upgrade  6 Packages
Total download size: 19 M
Is this ok [y/d/N]: y
Downloading packages:
No Presto metadata available for ol7_x86_64_userspace_ksplice
(1/7): glibc-2.17-326.0.9.ksplice1.el7_9.3.x86_64.rpm                                                                                                                                                                                                                                                 | 3.7 MB  00:00:02     
(2/7): glibc-common-2.17-326.0.9.ksplice1.el7_9.3.x86_64.rpm                                                                                                                                                                                                                                          |  12 MB  00:00:08     
(3/7): glibc-devel-2.17-326.0.9.ksplice1.el7_9.3.x86_64.rpm                                                                                                                                                                                                                                           | 1.1 MB  00:00:01     
(4/7): glibc-headers-2.17-326.0.9.ksplice1.el7_9.3.x86_64.rpm                                                                                                                                                                                                                                         | 695 kB  00:00:00     
(5/7): ksplice-helper-1.0.62-1.el7.x86_64.rpm                                                                                                                                                                                                                                                         |  21 kB  00:00:00     
(6/7): openssl-1.0.2k-26.ksplice1.el7_9.x86_64.rpm                                                                                                                                                                                                                                                    | 494 kB  00:00:00     
(7/7): openssl-libs-1.0.2k-26.ksplice1.el7_9.x86_64.rpm                                                                                                                                                                                                                                               | 1.2 MB  00:00:01     
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                                                                                                        1.2 MB/s |  19 MB  00:00:15     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : ksplice-helper-1.0.62-1.el7.x86_64                                                                                                                                                                                                                                                                       1/13 
  Updating   : 2:glibc-common-2.17-326.0.9.ksplice1.el7_9.3.x86_64                                                                                                                                                                                                                                                      2/13 
  Updating   : 2:glibc-2.17-326.0.9.ksplice1.el7_9.3.x86_64                                                                                                                                                                                                                                                             3/13 
  Updating   : 2:openssl-libs-1.0.2k-26.ksplice1.el7_9.x86_64                                                                                                                                                                                                                                                           4/13 
  Updating   : 2:glibc-headers-2.17-326.0.9.ksplice1.el7_9.3.x86_64                                                                                                                                                                                                                                                     5/13 
  Updating   : 2:glibc-devel-2.17-326.0.9.ksplice1.el7_9.3.x86_64                                                                                                                                                                                                                                                       6/13 
  Updating   : 2:openssl-1.0.2k-26.ksplice1.el7_9.x86_64                                                                                                                                                                                                                                                                7/13 
  Cleanup    : glibc-devel-2.17-326.0.9.el7_9.x86_64                                                                                                                                                                                                                                                                    8/13 
  Cleanup    : 1:openssl-1.0.2k-19.0.1.el7.x86_64                                                                                                                                                                                                                                                                       9/13 
  Cleanup    : glibc-headers-2.17-326.0.9.el7_9.x86_64                                                                                                                                                                                                                                                                 10/13 
  Cleanup    : 1:openssl-libs-1.0.2k-19.0.1.el7.x86_64                                                                                                                                                                                                                                                                 11/13 
  Cleanup    : glibc-common-2.17-326.0.9.el7_9.x86_64                                                                                                                                                                                                                                                                  12/13 
  Cleanup    : glibc-2.17-326.0.9.el7_9.x86_64                                                                                                                                                                                                                                                                         13/13 
  Verifying  : 2:glibc-2.17-326.0.9.ksplice1.el7_9.3.x86_64                                                                                                                                                                                                                                                             1/13 
  Verifying  : 2:glibc-devel-2.17-326.0.9.ksplice1.el7_9.3.x86_64                                                                                                                                                                                                                                                       2/13 
  Verifying  : 2:glibc-common-2.17-326.0.9.ksplice1.el7_9.3.x86_64                                                                                                                                                                                                                                                      3/13 
  Verifying  : 2:openssl-libs-1.0.2k-26.ksplice1.el7_9.x86_64                                                                                                                                                                                                                                                           4/13 
  Verifying  : ksplice-helper-1.0.62-1.el7.x86_64                                                                                                                                                                                                                                                                       5/13 
  Verifying  : 2:glibc-headers-2.17-326.0.9.ksplice1.el7_9.3.x86_64                                                                                                                                                                                                                                                     6/13 
  Verifying  : 2:openssl-1.0.2k-26.ksplice1.el7_9.x86_64                                                                                                                                                                                                                                                                7/13 
  Verifying  : glibc-headers-2.17-326.0.9.el7_9.x86_64                                                                                                                                                                                                                                                                  8/13 
  Verifying  : glibc-devel-2.17-326.0.9.el7_9.x86_64                                                                                                                                                                                                                                                                    9/13 
  Verifying  : glibc-common-2.17-326.0.9.el7_9.x86_64                                                                                                                                                                                                                                                                  10/13 
  Verifying  : glibc-2.17-326.0.9.el7_9.x86_64                                                                                                                                                                                                                                                                         11/13 
  Verifying  : 1:openssl-libs-1.0.2k-19.0.1.el7.x86_64                                                                                                                                                                                                                                                                 12/13 
  Verifying  : 1:openssl-1.0.2k-19.0.1.el7.x86_64                                                                                                                                                                                                                                                                      13/13 
Dependency Installed:
  ksplice-helper.x86_64 0:1.0.62-1.el7                                                                                                                                                                                                                                                                                       
Updated:
  glibc.x86_64 2:2.17-326.0.9.ksplice1.el7_9.3     glibc-common.x86_64 2:2.17-326.0.9.ksplice1.el7_9.3     glibc-devel.x86_64 2:2.17-326.0.9.ksplice1.el7_9.3     glibc-headers.x86_64 2:2.17-326.0.9.ksplice1.el7_9.3     openssl.x86_64 2:1.0.2k-26.ksplice1.el7_9     openssl-libs.x86_64 2:1.0.2k-26.ksplice1.el7_9    
Complete!
```

**E agora vamos para o grand finale: aplicar as correções de segurança (CVEs) com o Ksplice.**

```bash
yum install uptrack-updates-$(uname -r)
```

```bash
[49jdhygv] CVE-2021-20239: Information leak via cgroup BPF filter.
[36ap11tk] CVE-2021-3178: Path traversal vulnerability in NFSv3 filesystem.
[j5snfa4g] CVE-2020-27825: Race condition in kernel tracing buffers causes DoS.
[tlekvfju] CVE-2021-29154: Code execution in eBPF JIT compiler.
[oy5v22yn] Bad return value when adding an element to RAR Correctable Errors Collector.
[7108zh9o] CVE-2020-36310: Denial-of-service in KVM support due to a nested page fault.
[r8zov234] CVE-2021-31916: Information disclosure due to out-of-bounds writes in the Multi-device driver.
[i1vv7a8s] Improved update to CVE-2020-28374: Access control bypass when reading or writing TCM devices.
[j7hp93tv] CVE-2021-23133: Multiple vulnerabilities due to a race condition in SCTP
```

Pode acontecer durante a instalação dá esse aviso:

```bash

  Installing : uptrack-updates-5.4.17-2102.201.3.el7uek.x86_64-20241022-0.noarch                                                                                                                                                                                                                                         1/1 
It appears that another Uptrack process is currently running on this
system. Please wait a minute and try again.  If you are unable to
resolve this issue, please contact Oracle support.
  Verifying  : uptrack-updates-5.4.17-2102.201.3.el7uek.x86_64-20241022-0.noarch
```

Caso aconteça isso, espere 5 minutos e execute esse comando:

```bash
/usr/bin/python2 /usr/sbin/uptrack-upgrade -y --all
```

É isso, pessoal! Espero que este artigo ajude você a configurar o Ksplice no Oracle Linux.

E por último, mas não menos importante: para utilizar o Ksplice no Oracle Linux é necessário ter suporte ativo. Verifique se o seu CSI possui cobertura para o Ksplice no Oracle Linux.