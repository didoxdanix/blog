---
title: "Migrando VM'S do ODA/OVM para ODA/KVM"
datePublished: Sat Dec 28 2024 23:09:18 GMT+0000 (Coordinated Universal Time)
cuid: cm58sn3l7000809mn6fpiapqt
slug: migrando-vms-do-odaovm-para-odakvm
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1735427304359/a55a6dcf-c147-43f5-a4b3-ea85bc9f7bad.png

---

No post anterior aqui, abordamos sobre como configurar e gerenciar o KVM nas novas versões do ODA.

Agora, vamos falar um pouco sobre a migração de maquinas virtuais.

Como todos já sabem, a outra plataforma conhecida como “OVM” é baseada no Hypervisor XEN. E a grande pergunta que fica é … Como migrar as maquinas virtuais do “OVM” para o “KVM” ?

É isso meus amigos que vamos abordar neste artigo.

De forma bem simplificada e direta vamos aos detalhes da maquina virtual que vamos migrar do OVM para o KVM.

```bash
[root@odax5node1 ~]# oakcli show vm MVPEP-SML-TRN
The Resource is : MVPEP-SML-TRN
        AutoStart       :       restore        
        CPUPriority     :       100            
        Disks           :       |file:/OVS/Repositories/repo1/.ACFS
                                /snaps/MVPEP-SML-TRN/VirtualMachine
                                s/MVPEP-SML-TRN/7396bcf1382c42b1bf4
                                31ece9bcc23ee.img,xvda,w||,xvdb:/OV
                                S/Repositories/repo1/.ACFS/snaps/MV
                                PEP-SML-TRN/VirtualMachines/MVPEP-S
                                ML-TRN/cdrom,r|
        Domain          :       XEN_PVM        
        DriverDomain    :       False          
        ExpectedState   :       online        
        FailOver        :       true           
        IsSharedRepo    :       true           
        Keyboard        :       en-us          
        MaxMemory       :       32768M         
        MaxVcpu         :       24             
        Memory          :       32768M         
        Mouse           :       OS_DEFAULT     
        Name            :       MVPEP-SML-TRN  
        Networks        :       |bridge=net1|  
        NodeNumStart    :       1              
        OS              :       OL_5           
        PrefNodeNum     :       1              
        PrivateIP       :       None           
        ProcessorCap    :       0              
        RepoName        :       repo1          
        State           :       Online         
        TemplateName    :       otml_oraclelinux610
        VDisks          :       |oakvdk_disk120_mvpep-sml-trn_repo1
                                |              
        Vcpu            :       24             
        cpupool         :       default-unpinned-pool
        vncport         :       5901           

[root@odax5node1 ~]# oakcli show vdisk disk120_mvpep-sml-trn -repo repo1
The Resource is : disk120_mvpep-sml-trn_repo1
        Name            :       disk120_mvpep-sml-trn_repo1
        RepoName        :       repo1          
        Size            :       120G           
        Type            :       shared         
        VmAttached      :       1
```

A parte que nos interessa é literalmente esta:

Disco de Boot /OVS/Repositories/repo1/.ACFS/snaps/MVPEP-SML-TRN/VirtualMachines/MVPEP-SML-TRN/7396bcf1382c42b1bf431ece9bcc23ee.img Disco Secundario anexado /OVS/Repositories/repo1/.ACFS/snaps/oakvdk\_disk120\_mvpep-sml-trn/VirtualDisks/oakvdk\_disk120\_mvpep-sml-trn

```bash
[root@odax5node1 MVPEP-SML-TRN]# oakcli show repo repo1 -node 0
The Resource is : repo1_0
        AutoStart       :       restore        
        DG              :       DATA           
        Device          :       /dev/asm/repo1-341
        ExpectedState   :       Online         
        FreeSpace       :       236025.2M      
        MountPoint      :       /u01/app/sharedrepo/repo1
        Name            :       repo1_0        
        PFreeSpace      :       28.81%         
        RepoType        :       shared         
        Size            :       819200.0M      
        State           :       Online         
        Version         :       2
```

Até aqui tudo certo, temos todas as informações da VM.

Agora vamos para parte de “backup”

```bash
[root@odax5node1 MVPEP-SML-TRN]# oakcli stop vm MVPEP-SML-TRN
```

Maquina parada vamos copiar discos.

```bash
[root@odax5node1 MVPEP-SML-TRN]# cd /u01/app/sharedrepo/repo1

[root@odax5node1 repo1]# cp .ACFS/snaps/MVPEP-SML-TRN/VirtualMachines/MVPEP-SML-TRN/7396bcf1382c42b1bf431ece9bcc23ee.img /backup/
[root@odax5node1 repo1]# cp .ACFS/snaps/oakvdk_disk120_mvpep-sml-trn/VirtualDisks/oakvdk_disk120_mvpep-sml-trn /backup/
```

Copia finalizada, vamos sair do X5 “OVM” e vamos para o X8 KVM. OBS: Não copie nenhum arquivo para o VMSTORAGE/REPOSITORIO, neste local apenas arquivos das VM'S.

```bash
[root@odax8godata ovmstorage]# cd /backup

[root@odax8godata backup]# ll -rths
total 241G
121G -rw------- 1 root root 120G Feb 12 13:08 7396bcf1382c42b1bf431ece9bcc23ee.img
121G -rw-r--r-- 1 root root 120G Feb 12 13:56 oakvdk_disk120_mvpep-sml-trn
 64K drwx------ 2 root root  64K Feb 18 00:56 lost+found
 52K drwxr-xr-x 4 root root  20K Feb 18 19:24 old

[root@odax8godata backup]# qemu-img info 7396bcf1382c42b1bf431ece9bcc23ee.img
image: 7396bcf1382c42b1bf431ece9bcc23ee.img
file format: raw
virtual size: 120G (128849018880 bytes)
disk size: 120G

[root@odax8godata backup]# qemu-img info oakvdk_disk120_mvpep-sml-trn
image: oakvdk_disk120_mvpep-sml-trn
file format: raw
virtual size: 120G (128849018880 bytes)
disk size: 120G
```

Agora vamos renomear os discos para ficar mais fácil a compreensão.

```bash
[root@odax8godata backup]# mv 7396bcf1382c42b1bf431ece9bcc23ee.img DISCO_DE_BOOT.ovm
[root@odax8godata backup]# mv oakvdk_disk120_mvpep-sml-trn DISCO_SECUNDARIO.ovm
[root@odax8godata backup]# ll -rths
total 241G
121G -rw------- 1 root root 120G Feb 12 13:08 DISCO_DE_BOOT.ovm
121G -rw-r--r-- 1 root root 120G Feb 12 13:56 DISCO_SECUNDARIO.ovm
 64K drwx------ 2 root root  64K Feb 18 00:56 lost+found
 52K drwxr-xr-x 4 root root  20K Feb 18 19:24 old
 



[root@odax8godata backup]# qemu-img info DISCO_DE_BOOT.ovm
image: DISCO_DE_BOOT.ovm
file format: raw
virtual size: 120G (128849018880 bytes)
disk size: 120G

[root@odax8godata backup]# qemu-img info DISCO_SECUNDARIO.ovm
image: DISCO_SECUNDARIO.ovm
file format: raw
virtual size: 120G (128849018880 bytes)
disk size: 120G
```

Pronto, temos algumas informações relevantes sobre os discos, agora vamos iniciar a conversão do disco de OVM/XEN para KVM.

Antes de iniciar, precisamos saber pra qual tipo de disco vamos converter e quais são as propriedades do mesmo, para saber isso peguei informações de um disco já existente que foi criando de forma nativa pelo ODACLI.

```bash
[root@odax8godata vm_MACHINE19C]#  qemu-img info MACHINE19C
image: MACHINE19C
file format: qcow2
virtual size: 200G (214748364800 bytes)
disk size: 16G
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: true
    refcount bits: 16
    corrupt: false
	
[root@odax8godata vdisk_VDISK1]# qemu-img info VDISK1
image: VDISK1
file format: raw
virtual size: 300G (322122547200 bytes)
disk size: 300G
```

Aqui identificamos que não precisaremos converter o disco secundario ou seja o VDISK, vamos precisar converter apenas o disco de boot.

Tendo em mãos todas as informações necessarias vamos iniciar a conversão dos discos (OBS: NUNCA FAÇA ISSO DIRETAMENTE COM O DISCO DA MAQUINA VIRTUAL DO OVM/XEN, SEMPRE EXECUTE ISSO COM UM BACKUP DO DISCO/VM).

```bash
[root@odax8godata backup]# qemu-img convert -p -f raw -O qcow2 -o lazy_refcounts=on DISCO_DE_BOOT.ovm DISCO_DE_BOOT.qcow2
    (100.00/100%)

[root@odax8godata backup]# ll -rths
total 355G

121G -rw------- 1 root root 120G Feb 12 13:08 DISCO_DE_BOOT.ovm
121G -rw-r–r-- 1 root root 120G Feb 12 13:56 DISCO_SECUNDARIO.ovm
64K drwx------ 2 root root  64K Feb 18 00:56 lost+found
52K drwxr-xr-x 4 root root  20K Feb 18 19:24 old
112G -rw-r–r-- 1 root root 112G Feb 18 19:43 DISCO_DE_BOOT.qcow2
```

```bash
[root@odax8godata backup]# qemu-img info DISCO_DE_BOOT.qcow2
image: DISCO_DE_BOOT.qcow2
file format: qcow2
virtual size: 120G (128849018880 bytes)
disk size: 112G
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: true
    refcount bits: 16
    corrupt: false
```

```bash
[root@odax8godata backup]# qemu-img info DISCO_SECUNDARIO.ovm
image: DISCO_SECUNDARIO.ovm
file format: raw
virtual size: 120G (128849018880 bytes)
disk size: 120G
```

Disco convertido vamos iniciar a criação da VM.

Primeiro vamos criar um vdisk de 120GB (O tamanho tem quer ser exatamente o mesmo do disco/OVM). Esse disco será substituído pelo “DISCO\_SECUNDARIO.img”

```bash
[root@odax8godata vdisk_VDISK1]# odacli create-vdisk -n DISCO_SECUNDARIO -sh -s 120G -vms OVMSTORAGE

Job details                                                      
----------------------------------------------------------------
                     ID:  db9330e0-a124-49f0-9ffd-5b92e868c8d5
            Description:  VM disk DISCO_SECUNDARIO creation
                 Status:  Created
                Created:  February 20, 2021 2:12:38 AM BRT
                Message:  

Task Name                                Start Time                          End Time                            Status    
---------------------------------------- ----------------------------------- ----------------------------------- ----------



[root@odax8godata ~]# cd /u05/app/sharedrepo/ovmstorage/.ACFS/snaps/vdisk_DISCO_SECUNDARIO
[root@odax8godata vdisk_DISCO_SECUNDARIO]# ll -h
total 120G
-rw-r--r-- 1 root root 120G Feb 20 02:15 DISCO_SECUNDARIO
```

```bash
[root@odax8godata vdisk_DISCO_SECUNDARIO]# odacli create-vm -n MVPEP-SML-TRN -cp POOL12CORES -vc 24 -m 32G -vms OVMSTORAGE -s 120G -vd DISCO_SECUNDARIO -vn VNET1 -src /u01/V1003434-01.iso

Job details                                                      
----------------------------------------------------------------
                     ID:  6d3e30a1-ac79-485c-b29f-00a9ddbbc22f
            Description:  VM MVPEP-SML-TRN creation
                 Status:  Created
                Created:  February 20, 2021 2:28:48 AM BRT
                Message:  

Task Name                                Start Time                          End Time                            Status    
---------------------------------------- ----------------------------------- ----------------------------------- ----------
```

```bash
[root@odax8godata vdisk_DISCO_SECUNDARIO]# odacli list-vms
Name                  VM Storage            Node             Current State    Target State     Created                  Updated                
--------------------  --------------------  ---------------  ---------------  ---------------  -----------------------  -----------------------
MVPEP-SML-TRN         OVMSTORAGE            odax8godata     ONLINE           ONLINE           2021-02-20 02:28:59 BRT  2021-02-20 02:28:59 BRT
```

```bash
[root@odax8godata vdisk_DISCO_SECUNDARIO]# odacli describe-vm -n MVPEP-SML-TRN
VM details                                                                      
--------------------------------------------------------------------------------
                       ID:  f18f916b-19ae-4bfd-af33-f4a49f67f625
                     Name:  MVPEP-SML-TRN
                  Created:  2021-02-20 02:28:59 BRT
                  Updated:  2021-02-20 02:28:59 BRT
               VM Storage:  OVMSTORAGE
              Description:  NONE
                  VM size:  120.00 GB
                   Source:  V1003434-01.iso
                  OS Type:  NONE
               OS Variant:  NONE
        Graphics settings:  vnc,listen=0.0.0.0
             Display Port:  :0

 Status                   
--------------------------
             Current node:  odax8godata
            Current state:  ONLINE
             Target state:  ONLINE

 Parameters               
--------------------------
           Preferred node:  NONE
              Boot option:  NONE
               Auto start:  YES
                Fail over:  NO

                            Config                     Live                     
                            -------------------------  -------------------------
                   Memory:  32.00 GB                   32.00 GB                 
               Max Memory:  32.00 GB                   32.00 GB                 
               vCPU count:  24                         24                       
           Max vCPU count:  24                         24                       
                 CPU Pool:  POOL12CORES                POOL12CORES              
        Effective CPU set:  16-27,48-59                16-27,48-59              
                    vCPUs:  0:16-27,48-59              0:16-27,48-59            
                            1:16-27,48-59              1:16-27,48-59            
                            2:16-27,48-59              2:16-27,48-59            
                            3:16-27,48-59              3:16-27,48-59            
                            4:16-27,48-59              4:16-27,48-59            
                            5:16-27,48-59              5:16-27,48-59            
                            6:16-27,48-59              6:16-27,48-59            
                            7:16-27,48-59              7:16-27,48-59            
                            8:16-27,48-59              8:16-27,48-59            
                            9:16-27,48-59              9:16-27,48-59            
                            10:16-27,48-59             10:16-27,48-59           
                            11:16-27,48-59             11:16-27,48-59           
                            12:16-27,48-59             12:16-27,48-59           
                            13:16-27,48-59             13:16-27,48-59           
                            14:16-27,48-59             14:16-27,48-59           
                            15:16-27,48-59             15:16-27,48-59           
                            16:16-27,48-59             16:16-27,48-59           
                            17:16-27,48-59             17:16-27,48-59           
                            18:16-27,48-59             18:16-27,48-59           
                            19:16-27,48-59             19:16-27,48-59           
                            20:16-27,48-59             20:16-27,48-59           
                            21:16-27,48-59             21:16-27,48-59           
                            22:16-27,48-59             22:16-27,48-59           
                            23:16-27,48-59             23:16-27,48-59           
                   vDisks:  DISCO_SECUNDARIO:vdb       DISCO_SECUNDARIO:vdb     
                vNetworks:  VNET1:52:54:00:68:26:a9    VNET1:52:54:00:68:26:a9
```

Maquina criada, vamos para-la, para substituir pelos disco convertido.

```bash
[root@odax8godata vdisk_DISCO_SECUNDARIO]# odacli stop-vm -n MVPEP-SML-TRN

Job details                                                      
----------------------------------------------------------------
                     ID:  e318afa6-9190-41e2-8167-d2f24a165928
            Description:  VM MVPEP-SML-TRN stop
                 Status:  Success
                Created:  February 20, 2021 2:30:58 AM BRT
                Message:  

Task Name                                Start Time                          End Time                            Status    
---------------------------------------- ----------------------------------- ----------------------------------- ----------
Validate dependency resources            February 20, 2021 2:30:58 AM BRT    February 20, 2021 2:30:58 AM BRT    Success   
Stop VM                                  February 20, 2021 2:30:58 AM BRT    February 20, 2021 2:31:03 AM BRT    Success
```

```bash
[root@odax8godata vdisk_DISCO_SECUNDARIO]# odacli list-vms 
Name                  VM Storage            Node             Current State    Target State     Created                  Updated                
--------------------  --------------------  ---------------  ---------------  ---------------  -----------------------  -----------------------
MVPEP-SML-TRN         OVMSTORAGE                             OFFLINE          OFFLINE          2021-02-20 02:28:59 BRT  2021-02-20 02:28:59 BRT
```

```bash
[root@odax8godata backup]# cd /backup/

[root@odax8godata backup]# ll -h
total 352G
-rw------- 1 root root 120G Feb 12 13:08 DISCO_DE_BOOT.ovm
-rw-r--r-- 1 root root 112G Feb 18 19:43 DISCO_DE_BOOT.qcow2
-rw-r--r-- 1 root root 120G Feb 19 10:44 DISCO_SECUNDARIO.ovm



[root@odax8godata backup]# qemu-img info DISCO_DE_BOOT.qcow2
image: DISCO_DE_BOOT.qcow2
file format: qcow2
virtual size: 120G (128849018880 bytes)
disk size: 112G
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: true
    refcount bits: 16
    corrupt: false

[root@odax8godata backup]# qemu-img info  DISCO_SECUNDARIO.ovm
image: DISCO_SECUNDARIO.ovm
file format: raw
virtual size: 120G (128849018880 bytes)
disk size: 120G
```

```bash
[root@odax8godata ~]# cd /u05/app/sharedrepo/ovmstorage/.ACFS/snaps/vm_MVPEP-SML-TRN
[root@odax8godata vm_MVPEP-SML-TRN]# ll -h
total 476M
-rw------- 1 root root 121G Feb 20 02:28 MVPEP-SML-TRN
-rw-r--r-- 1 root root 4.3K Feb 20 02:28 MVPEP-SML-TRN_live.xml
-rw-r--r-- 1 root root 3.7K Feb 20 02:28 MVPEP-SML-TRN.xml

[root@odax8godata ~]# cd /u05/app/sharedrepo/ovmstorage/.ACFS/snaps/vdisk_DISCO_SECUNDARIO
[root@odax8godata vdisk_DISCO_SECUNDARIO]# ll -h
total 120G
-rw-r--r-- 1 qemu qemu 120G Feb 20 02:15 DISCO_SECUNDARIO
```

Agora vamos substituir os seguintes arquivos…

MVPEP-SML-TRN —&gt;&gt;&gt; DISCO\_DE\_BOOT.qcow2 DISCO\_SECUNDARIO —&gt;&gt;&gt; DISCO\_SECUNDARIO.ovm

```bash
[root@odax8godata discos_antes_migracao]# cd  /backup/
[root@odax8godata backup]# mkdir discos_antes_migracao
      
[root@odax8godata backup]# mv /u05/app/sharedrepo/ovmstorage/.ACFS/snaps/vm_MVPEP-SML-TRN/MVPEP-SML-TRN /backup/discos_antes_migracao/
[root@odax8godata backup]# mv /u05/app/sharedrepo/ovmstorage/.ACFS/snaps/vdisk_DISCO_SECUNDARIO/DISCO_SECUNDARIO /backup/discos_antes_migracao/
[root@odax8godata backup]# ll -h /backup/discos_antes_migracao/
total 121G
-rw-r--r-- 1 qemu qemu 120G Feb 20 02:15 DISCO_SECUNDARIO
-rw------- 1 root root 121G Feb 20 02:28 MVPEP-SML-TRN
```

Discos "padrões" movidos, agora vamos colocar no devido lugar o disco de boot convertido e o disco secundário/vdisk que veio do ODA X5/OVM.

```bash
root@odax8godata backup]# cd /backup/
[root@odax8godata backup]# ll -h
total 352G
-rw-r--r-- 1 root root 112G Feb 18 19:43 DISCO_DE_BOOT.qcow2
-rw-r--r-- 1 root root 120G Feb 19 10:44 DISCO_SECUNDARIO.ovm

[root@odax8godata backup]# cp DISCO_DE_BOOT.qcow2 /u05/app/sharedrepo/ovmstorage/.ACFS/snaps/vm_MVPEP-SML-TRN/MVPEP-SML-TRN
[root@odax8godata backup]# cp DISCO_SECUNDARIO.ovm /u05/app/sharedrepo/ovmstorage/.ACFS/snaps/vdisk_DISCO_SECUNDARIO/DISCO_SECUNDARIO
[root@odax8godata backup]# ll -h /u05/app/sharedrepo/ovmstorage/.ACFS/snaps/vm_MVPEP-SML-TRN/
total 112G
-rw-r--r-- 1 root root 112G Feb 20 02:52 MVPEP-SML-TRN
-rw-r--r-- 1 root root 4.3K Feb 20 02:28 MVPEP-SML-TRN_live.xml
-rw-r--r-- 1 root root 3.7K Feb 20 02:28 MVPEP-SML-TRN.xml

[root@odax8godata backup]# ll -h /u05/app/sharedrepo/ovmstorage/.ACFS/snaps/vdisk_DISCO_SECUNDARIO/
total 121G
-rw-r--r-- 1 root root 120G Feb 20 02:56 DISCO_SECUNDARIO

[root@odax8godata ~]# cd /u05/app/sharedrepo/ovmstorage/.ACFS/snaps/vm_MVPEP-SML-TRN/

[root@odax8godata vm_MVPEP-SML-TRN]# ll -rths
total 112G
4.0K -rw-r--r-- 1 root root 3.7K Feb 20 02:28 MVPEP-SML-TRN.xml
 12K -rw-r--r-- 1 root root 4.3K Feb 20 02:28 MVPEP-SML-TRN_live.xml
112G -rw-r--r-- 1 root root 112G Feb 20 02:52 MVPEP-SML-TRN

[root@odax8godata vm_MVPEP-SML-TRN]# chmod 600 MVPEP-SML-TRN

[root@odax8godata vm_MVPEP-SML-TRN]# ll -rths
total 112G
4.0K -rw-r--r-- 1 root root 3.7K Feb 20 02:28 MVPEP-SML-TRN.xml
 12K -rw-r--r-- 1 root root 4.3K Feb 20 02:28 MVPEP-SML-TRN_live.xml
112G -rw------- 1 root root 112G Feb 20 02:52 MVPEP-SML-TRN

[root@odax8godata ~]# cd /u05/app/sharedrepo/ovmstorage/.ACFS/snaps/vdisk_DISCO_SECUNDARIO/

[root@odax8godata vdisk_DISCO_SECUNDARIO]# ll -rths
total 121G
121G -rw-r--r-- 1 root root 120G Feb 20 02:56 DISCO_SECUNDARIO

[root@odax8godata vdisk_DISCO_SECUNDARIO]# chown qemu:qemu DISCO_SECUNDARIO

[root@odax8godata vdisk_DISCO_SECUNDARIO]# ll -rths
total 121G
121G -rw-r--r-- 1 qemu qemu 120G Feb 20 02:56 DISCO_SECUNDARIO
```

Discos devidamente substituídos, vamos iniciar a VM.

```bash
[root@odax8godata vdisk_DISCO_SECUNDARIO]# odacli list-vms
Name                  VM Storage            Node             Current State    Target State     Created                  Updated                
--------------------  --------------------  ---------------  ---------------  ---------------  -----------------------  -----------------------
MVPEP-SML-TRN         OVMSTORAGE                             OFFLINE          OFFLINE          2021-02-20 02:28:59 BRT  2021-02-20 02:28:59 BRT
```

```bash
[root@odax8godata vdisk_DISCO_SECUNDARIO]# odacli start-vm -n MVPEP-SML-TRN
 
Job details                                                      
----------------------------------------------------------------
                     ID:  2d234ec2-df58-44cd-b603-a00b3c972acf
            Description:  VM MVPEP-SML-TRN start
                 Status:  Success
                Created:  February 20, 2021 3:07:43 AM BRT
                Message:  

Task Name                                Start Time                          End Time                            Status    
---------------------------------------- ----------------------------------- ----------------------------------- ----------
Validate dependency resources            February 20, 2021 3:07:43 AM BRT    February 20, 2021 3:07:43 AM BRT    Success   
Start VM                                 February 20, 2021 3:07:43 AM BRT    February 20, 2021 3:07:51 AM BRT    Success   
Save live VM configuration in ACFS       February 20, 2021 3:07:51 AM BRT    February 20, 2021 3:07:51 AM BRT    Success   
Save live VM configuration in metadata   February 20, 2021 3:07:51 AM BRT    February 20, 2021 3:07:51 AM BRT    Success
```

```bash
[root@odax8godata vdisk_DISCO_SECUNDARIO]# odacli list-vms
Name                  VM Storage            Node             Current State    Target State     Created                  Updated                
--------------------  --------------------  ---------------  ---------------  ---------------  -----------------------  -----------------------
MVPEP-SML-TRN         OVMSTORAGE            odax8godata     ONLINE           ONLINE           2021-02-20 02:28:59 BRT  2021-02-20 03:07:51 BRT

[root@odax8godata vdisk_DISCO_SECUNDARIO]# odacli describe-vm -n MVPEP-SML-TRN
VM details                                                                      
--------------------------------------------------------------------------------
                       ID:  f18f916b-19ae-4bfd-af33-f4a49f67f625
                     Name:  MVPEP-SML-TRN
                  Created:  2021-02-20 02:28:59 BRT
                  Updated:  2021-02-20 03:07:51 BRT
               VM Storage:  OVMSTORAGE
              Description:  NONE
                  VM size:  120.00 GB
                   Source:  V1003434-01.iso
                  OS Type:  NONE
               OS Variant:  NONE
        Graphics settings:  vnc,listen=0.0.0.0
             Display Port:  :0

 Status                   
--------------------------
             Current node:  odax8godata
            Current state:  ONLINE
             Target state:  ONLINE

 Parameters               
--------------------------
           Preferred node:  NONE
              Boot option:  NONE
               Auto start:  YES
                Fail over:  NO

                            Config                     Live                     
                            -------------------------  -------------------------
                   Memory:  32.00 GB                   32.00 GB                 
               Max Memory:  32.00 GB                   32.00 GB                 
               vCPU count:  24                         24                       
           Max vCPU count:  24                         24                       
                 CPU Pool:  POOL12CORES                POOL12CORES              
        Effective CPU set:  16-27,48-59                16-27,48-59              
                    vCPUs:  0:16-27,48-59              0:16-27,48-59            
                            1:16-27,48-59              1:16-27,48-59            
                            2:16-27,48-59              2:16-27,48-59            
                            3:16-27,48-59              3:16-27,48-59            
                            4:16-27,48-59              4:16-27,48-59            
                            5:16-27,48-59              5:16-27,48-59            
                            6:16-27,48-59              6:16-27,48-59            
                            7:16-27,48-59              7:16-27,48-59            
                            8:16-27,48-59              8:16-27,48-59            
                            9:16-27,48-59              9:16-27,48-59            
                            10:16-27,48-59             10:16-27,48-59           
                            11:16-27,48-59             11:16-27,48-59           
                            12:16-27,48-59             12:16-27,48-59           
                            13:16-27,48-59             13:16-27,48-59           
                            14:16-27,48-59             14:16-27,48-59           
                            15:16-27,48-59             15:16-27,48-59           
                            16:16-27,48-59             16:16-27,48-59           
                            17:16-27,48-59             17:16-27,48-59           
                            18:16-27,48-59             18:16-27,48-59           
                            19:16-27,48-59             19:16-27,48-59           
                            20:16-27,48-59             20:16-27,48-59           
                            21:16-27,48-59             21:16-27,48-59           
                            22:16-27,48-59             22:16-27,48-59           
                            23:16-27,48-59             23:16-27,48-59           
                   vDisks:  DISCO_SECUNDARIO:vdb       DISCO_SECUNDARIO:vdb     
                vNetworks:  VNET1:52:54:00:68:26:a9    VNET1:52:54:00:68:26:a9
```

![img1](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgpDTHBZOGifqzZpX5YS8JxDpzbzEmUOCPjtTdJNiKCKZFLCxrzKJLmYClaU3Fz072jZxeRRQuOL7ZUuv_GRE9EC5XYB547UMtUdMeDir4TjREHhjnesEJBeLgnsU2j2SxhqrsDZ5mAelhD/s850/pos+migracao.png align="left")

![img2](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgX5umtwuVQXXfX1skPzy04GmMomwm0g8I_LZBiJcftyPKGSKVKljp4hyJDMURrrzSMy4mPxJDV0nLS5kHCmvG0OZSEoX9dsB4qrgquCYzU2Vuqnsvl3ZYXddCBX00e40t5J4GFg0Bpwo11/s797/pos+migracao+1.png align="left")

![img3](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjhlbQ8mEPAkqWRP2vVZYTV-unUyySaYi00ABsP68SzA37QnDFYWgABL_2FzTtJj0wsPgylcQZqc1pwc1XaQp5BDvqtQcyba8H7tTz4tABsMJYU7Z0EQK25tPaw32AvIToLTmWvxpveTWik/s826/pos+migracao+2.png align="left")