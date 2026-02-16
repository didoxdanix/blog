---
title: "Ksplice no Exadata"
datePublished: Sat Dec 28 2024 23:27:23 GMT+0000 (Coordinated Universal Time)
cuid: cm58tad6b000309mbho8acyml
slug: ksplice-no-exadata
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1735428392391/bfd36eda-0109-448a-b7ad-a7a8001680d7.png

---

Normalmente quando precisamos corrigir qualquer problema no Kernel/OS do Exadata aplicamos o bundle patch de OS do exadata para corrigir o problema, mas você pode encontrar um bug/problema específico que já foi reconhecido pelo ksplice ou que a equipe de suporte da Oracle acabou de solucionar e já publicou no ksplice para resolver o “seu” problema.

Alguns alertas:

* Ksplice offline updates may be installed on **database nodes only**.
    
* Ksplice offline updates are supported for Exadata images 12.1.1.1.2 and later.
    
* Ksplice offline updates are supported only for kernel updates. Exadata does not support ksplice updates for user space packages.
    

        Este é um procedimento que o próprio suporte da Oracle (Doc ID 2207063.1) vai pedir para executá-lo, então execute-o apenas quando for requisitado pelo Suporte Oracle.        A razão mais comum pela qual a instalação do ksplice falha é que um ou mais módulos do kernel proprietários são carregados por um produto de segurança de terceiros que modificou o kernel de uma forma que entra em conflito com uma correção do ksplice recebida. O seguinte comando mostra módulos de kernel proprietários carregados:

```bash
[root@dbm0db02 ~]#  grep -l P /sys/module/*/taint | cut -f4 -d/ | egrep -v 'oracle(acfs|advm|oks)'
[root@dbm0db02 ~]#
```

Se não retornar nada podemos iniciar, caso retorne, favor consulte a nota [2207063.1](https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=288786106005096&parent=EXTERNAL_SEARCH&sourceId=HOWTO&id=2207063.1&_afrWindowMode=0&_adf.ctrl-state=x5vvl0vgf_119).

Considerando que o procedimento anterior foi executado com sucesso, vamos iniciar o procedimento.

1º vamos acessar o site: [https://linux.oracle.com](https://linux.oracle.com/) (Logue com a conta da sua organização).

2º Vá em Channels

![enter image description here](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhNOBVqzSQcBW-qAbUUGNiIbxkT0MLZpqJo0a4aZb8MqR1XxzfYamOMo8EDpZBCbjratG6l_KJxoiOQ27s2tYVSBxBzbBSRck4MRQelEvHIfkuhQd8lf8MY8sO9ABrez8cvSwGAtmhfWfYe/s567/Imagem1.png align="left")

3 º Selecione a arquitetura:

![enter image description here](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhwNzRTPmIuylI-teSUULQLFVLpUsSBqkovFKAs9MzdGzQnoIi1B0l0gXkn8KbctfvPl69TaaRNQTvv7xgG4oQrjTfixK-fGNQAOSppSjqNDu64CvJ2sbk0IXd6r2brg836LkS1lExRMOMj/s567/Imagem2.png align="left")

4 º Procure pelo pacote “Ksplice for Oracle Linux (x86\_64)”

![enter image description here](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhFMxP3oArTkTbgpLePMRjw8gf9yZJzDKRPVOfzDTEyE7pea2ZouHf1fOtlH4qeQf9dhTnMb_HWSMpMuF8rHAcDQAPRHcG1szzvx3AY7kkM7l0ZG-hSmq88qAnsXIlibjNXS2fH-6EJp3AM/s303/Imagem3.png align="left")

5º Depois de clicar em Ksplice for Oracle Linux 7 (x86\_64), clique em Channel packages.

![enter image description here](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgI8pqjV0tAQB2rY5KOCUBI-JBxFTkCwmMYhih14YwYtBOEahCGoZaF4WXHoKuC4MDevWiP51VeImDbbqVq74Bcr57fy0-IFTkeZcj3o5SbywyTxlLuJY1ZNIxdcna05ARba9D5CnhRWsPM/s567/Imagem4.png align="left")

6º Agora vamos baixar o pacote exato do seu kernel atual:

```bash
[root@dbm0db02 ~]# uname -r

4.14.35-1902.303.5.3.el7uek.x86_64
```

Vamos dar um CTRL+F e baixar exatamente o package que contem o resultado do “uname -r”

![enter image description here](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEirvQZe8MUJk8C-0UVw-sKPyCjlIkKsWW6c974rpRqOt133aaeX5UpBcZw94jw8JqoEnk6q_5bfva7KYsLJ1CIcZvtaWB1GIVrbFd_C0L39r7tzgz00Qm1BMUDCOSpiT1dhFkvUmLEYwQgH/s567/Imagem5.png align="left")

Transfira para o dbnodes e vamos iniciar o procedimento:

```bash
[root@dbm0db02 ~]# ll -rths
total 50M
 50M -rw-r--r-- 1 root    root     50M May 22 18:09 uptrack-updates-4.14.35-1902.303.5.3.el7uek.x86_64-20210512-0.noarch.rpm
```

Como a nota [2207063.1 recomend](https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=288786106005096&parent=EXTERNAL_SEARCH&sourceId=HOWTO&id=2207063.1&_afrWindowMode=0&_adf.ctrl-state=x5vvl0vgf_119)a, antes de iniciarmos a instalação do ksplice precisamos remover o pacote “exadata-sun.\*computenode-exact”.

```bash
[root@dbm0db02 ~]#  yum list installed | grep 'exadata-sun.*computenode-exact'
exadata-sun-computenode-exact.noarch       20.1.0.0.0.200616-1         installed

[root@dbm0db02 ~]# yum erase exadata-sun-computenode-exact.noarch
Resolving Dependencies
--> Running transaction check
---> Package exadata-sun-computenode-exact.noarch 0:20.1.0.0.0.200616-1 will be erased
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================================================================================================================================== Package                                                       Arch                                   Version                                              Repository                                 Size
===========================================================================================================================================================================================================Removing:
 exadata-sun-computenode-exact                                 noarch                                 20.1.0.0.0.200616-1                                  installed                                 0.0  

Transaction Summary
===========================================================================================================================================================================================================Remove  1 Package

Installed size: 0  
Is this ok [y/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Erasing    : exadata-sun-computenode-exact-20.1.0.0.0.200616-1.noarch                                                                                                                                1/1 
  Verifying  : exadata-sun-computenode-exact-20.1.0.0.0.200616-1.noarch                                                                                                                                1/1 

Removed:
  exadata-sun-computenode-exact.noarch 0:20.1.0.0.0.200616-1                                                                                                                                               

Complete!
```

Pacote removido! Vamos a instalação…

```bash
[root@dbm0db02 ~]# ll -rths
total 50M
50M -rw-r--r-- 1 root    root     50M May 22 18:09 uptrack-updates-4.14.35-1902.303.5.3.el7uek.x86_64-20210512-0.noarch.rpm

[root@dbm0db02 ~]# yum install /root/uptrack-updates-4.14.35-1902.303.5.3.el7uek.x86_64-20210512-0.noarch.rpm
Examining /root/uptrack-updates-4.14.35-1902.303.5.3.el7uek.x86_64-20210512-0.noarch.rpm: uptrack-updates-4.14.35-1902.303.5.3.el7uek.x86_64-20210512-0.noarch
Marking /root/uptrack-updates-4.14.35-1902.303.5.3.el7uek.x86_64-20210512-0.noarch.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package uptrack-updates-4.14.35-1902.303.5.3.el7uek.x86_64.noarch 0:20210512-0 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================================================================================================================================== Package                                                          Arch                 Version                   Repository                                                                           Size
===========================================================================================================================================================================================================Installing:
 uptrack-updates-4.14.35-1902.303.5.3.el7uek.x86_64               noarch               20210512-0                /uptrack-updates-4.14.35-1902.303.5.3.el7uek.x86_64-20210512-0.noarch                51 M

Transaction Summary
===========================================================================================================================================================================================================Install  1 Package

Total size: 51 M
Installed size: 51 M
Is this ok [y/d/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : uptrack-updates-4.14.35-1902.303.5.3.el7uek.x86_64-20210512-0.noarch                                                                                                                    1/1 
The following steps will be taken:
Install [edt48ylb] Add ftrace safety guard for existing Ksplice updates.
Install [bipgvpw6] Known exploit detection.
Install [lylzl1sj] Known exploit detection for CVE-2017-7308.
Install [snvyltlq] Known exploit detection for CVE-2018-14634.
Install [e2hf1ats] KPTI enablement for Ksplice.
Install [1teh5owg] Known exploit detection for CVE-2018-18445.
Install [44121bvv] Double free with SCSI LSI MPT Fusion SAS attach error.
Install [trh9jthq] CVE-2019-3846: Heap overflow when parsing BSS descriptor in Marvell WiFi-Ex driver.
Install [3cqjhtx3] CVE-2019-19054: Denial-of-service in the cx2388x tv card driver.
Install [7q5nae2r] CVE-2019-15214: Use-after-free when connecting ALSA cards.
Install [ff6vfhdf] CVE-2019-19536: Information leak when initializing PCAN-USB device.
Install [ixsnbim2] CVE-2019-7308: Out-of-bounds speculation in BPF verifier.
Install [2lm6v1an] CVE-2020-12770: Information leak/DoS in SCSI generic userspace write.
Install [67w4v90f] CVE-2020-12464: Use-after-free in USB scatter-gather library.
Install [4tnet5w5] CVE-2019-19534: Information leak using PEAK PCAN-USB/USB Pro interfaces for CAN 2.0b/CAN-FD.
Install [9teqocf3] CVE-2020-12653: Denial-of-service when scanning for APs in mwifiex driver.
Install [qdojji7h] CVE-2019-3846: Heap overflow when parsing fields in Marvell WiFi-Ex driver.
Install [mcajph0v] CVE-2020-12654: Denial-of-service when querying WMM status in mwifiex driver.
Install [3u8vegty] CVE-2020-10757: Flaw in DAX page mapping allows privilege escalation.
Install [nlf0zswi] CVE-2020-10711: NULL pointer dereference when using CIPSO network packet labeling.
Install [ptsm3729] CVE-2019-19533: Information leak in Technotrend/Hauppauge USB DEC driver.
Install [5pv0zta2] CVE-2020-12655: Denial-of-service when syncing data on XFS filesystem.
Install [aadmn3e5] Use-after-free when freeing received data over RDS socket.
Install [c7221zqe] Buffer overflow when dumping registers in LSI Logic MegaRAID SAS RAID driver.
Install [9elx8r6h] CVE-2020-12652: Denial-of-service in the Fusion MTP driver.
Install [hwmxykva] Poor NFS performances caused by excessive attribute revalidation.
Install [tgthi987] Denial-of-service when freezing and unfreezing an XFS filesystem.
Install [cya3ogzy] Improved fix for CVE-2020-2732 when booting nested guests.
Install [e966zc48] Denial-of-service in the Infiniband driver when referencing a node.
Install [cvdwwuzd] Use-after-free in the Infiniband driver when releasing resources.
Install [kjk2vpux] Race condition when sending IB subnet MAD causes denial-of-service.
Install [hiaoskn9] CVE-2020-10766: Information leak using Spectre V4 variant.
Install [t439t60g] NULL-pointer dereference when shutting down DSA switch.
Install [t4avel7c] CVE-2019-19447: Use-after-free when unmounting corrupt ext4 filesystem.
Install [r4pnegf6] CVE-2020-10732: Information leak in corefiles in per-thread info.
Install [e9o5is2i] CVE-2019-19062: Denial-of-service in the crypto subsystem.
Install [q7lx9ooy] CVE-2019-16234: NULL pointer dereference when registering Intel Wireless WiFi driver.
Install [d1l1wlz8] Use-after-free when releasing clocks in PTP clock driver.
Install [3adwuqzu] CVE-2019-19037: Denial-of-service when handling empty directories in ext4 filesystem.
Install [mx9ibtwj] CVE-2019-16232: NULL pointer dereference when registering Marvell Libertas 8385/8686/8688 SDIO 802.11b/g cards.
Install [jm6yd4li] Memory corruption during cgroup destruction with PSI enabled.
Install [bb0tzpz2] Kernel crash in guest VM with machine check exception.
Install [clivbxob] CVE-2019-20811: Denial-of-service in network device sysfs system.
Install [tsh471fw] Add bit for guest kernel to handle kernel panic without host intervention.
Install [mqhc4bmw] Don't return an ACK on some RDMA netlink operations.
Install [4ixr7sgq] CVE-2018-20169: Missing bound check when reading extra USB descriptors.
Install [3nkinjqi] CVE-2018-1000026: Denial-of-service when receiving invalid packet on bnx2x network card.
Install [5f5w0oz6] CVE-2018-18281: Information leak in mremap syscall.
Install [1a5n75aj] CVE-2019-19063: Denial-of-service in the rtlwifi driver.
Install [19lbejmu] CVE-2019-0136: Denial-of-service in Intel(R) wifi driver.
Install [anxgy28r] CVE-2018-20784: Denial-of-service in task scheduling.
Install [1y4ivn6l] CVE-2018-20976: Use-after-free when mounting XFS filesystem.
Install [dhmgixyw] CVE-2015-2150: Denial-of-service in Xen host from the guest.
Install [jurvatrf] CVE-2019-19523: Use-after-free when disconnecting ADU USB devices.
Install [3tcll1ov] CVE-2018-16882: Privilege escalation in nested Intel KVM interrupts.
Install [od6vms9i] CVE-2019-19052: Memory leak when opening USB Socket CAN device driver.
Install [byd1fjvz] CVE-2019-15927: Out-of-bounds accesses in usb audio driver.
Install [rtg2jjko] CVE-2019-9506: Information disclosure when transmitting over bluetooth.
Install [hgb8rgcc] CVE-2019-5108: Denial-of-service of a wireless access point during roaming of a station.
Install [okxy9ag9] CVE-2020-10751: SELinux bypass in netlink message validation.
Install [l8qz0cf1] CVE-2019-15918: Out-of-bounds access during CIFS mount.
Install [fhoaweza] CVE-2019-2024: Use-after-free when disconnecting a Empia EM28xx USB device.
Install [gphk4k5r] CVE-2020-13974: Integer overflow in virtual terminal keyboard interface.
Install [iivjues9] CVE-2019-19528: Denial-of-service when disconnecting IO Warrior USB device.
Install [11uqhn6c] CVE-2020-12114: Race condition in mountpoint counter causes DoS.
Install [b9b9f3kf] CVE-2019-19807: Use-after-free when registering timer in ALSA driver.
Install [hgpl3vss] CVE-2019-15218: Denial-of-service in Siano Mobile Digital TV USB tuner probing.
Install [109qn2xz] CVE-2019-19530: Denial-of-service in USB CDC-ACM probing.
Install [bhcvif0m] CVE-2020-11565: Out-of-bounds access when mounting tmpfs.
Install [mveq6q0v] CVE-2019-2101: Information leak when initializing a usb video device.
Install [96jk0uhc] CVE-2019-15117: Out-of-bounds access when parsing USB descriptor in ALSA USB driver.
Install [ajidrtfz] Improved fix for CVE-2018-17972: Information leak in /proc kernel stack dumps.
Install [uzg2h1sb] CVE-2019-19066: Denial-of-service int SCSI bfa driver.
Install [5d8o75u5] CVE-2019-15118: Stack overflow when checking input source type in ALSA USB driver.
Install [kj8qwnw5] CVE-2019-19051: Memory leak when changing power status of Intel Wireless WiMAX Connection 2400 driver.
Install [dldi73w2] CVE-2018-1129: Signature check bypass of cephx message.
Install [6g7nqusv] CVE-2019-3900: Infinite loop in vhost_net driver under heavy load.
Install [fp67ub1v] CVE-2020-1749: Information disclosure in IPv6 IPSec tunneling.
Install [jqobfvky] CVE-2019-11487: Invalid memory access when overflowing pages refcount.
Install [o3vwjkah] CVE-2019-18805: Denial-of-service in IPv4 round trip time configuration.
Install [r6xwmr9a] CVE-2019-19535, CVE-2019-19536: Information leak when initializing PCAN-USB device.
Install [eirwnk56] CVE-2017-18552: Memory corruption in the RDS protocol.
Install [8yot03uh] CVE-2019-15921: Denial-of-service in generic netlink socket family.
Install [2thbpnu3] CVE-2019-20812: Soft lockup in packet sockets with zero timeout.
Install [7c33rkhk] CVE-2019-9458: Use-after-free in V4L2 event subscription.
Install [4grifv3a] CVE-2019-9455: Information leak in V4L2 when setting output buffer size.
Install [fb5eodub] CVE-2019-19073, CVE-2019-19074: Denial-of-service in the ath9k wireless driver.
Install [samo6fmn] CVE-2020-10720: Use-after-free in generic receive offload fragmentation.
Install [ah33pp3h] CVE-2020-0305: Use-after-free when failing to open file on character device.
Install [gm7wtw1w] CVE-2020-12771: Deadlock during BCache node coalesce failure.
Install [ap0dbd8r] CVE-2019-15902: Bounds-check bypass in sys_ptrace().
Install [32yey8b2] CVE-2019-10220: Privileges escalation when parsing directory from a bad SMB server.
Install [t36cih2j] CVE-2020-8992: Deadlock with too big journal size on ext4 filesystem.
Install [l16s2z79] CVE-2020-10769: Out-of-bounds memory access in authenticated encryption key parsing.
Install [h2zoj70m] CVE-2014-9900: Information disclosure in Wake-On-LAN driver.
Install [jgoq29rm] Improved fix for CVE-2019-19768: Use-after-free when reporting an IO trace.
Install [baczzgxp] CVE-2019-19642: Denial-of-service in kernel relay file open path.
Install [rb014za7] Incorrect reporting of Process Address Space ID on AMD systems.
Install [9npfv9rb] Connection failure after RDS peer reboot.
Install [b5lxrr63] CVE-2020-24394: Information leak when exporting a filesystem over NFS.
Install [grdlpgdc] CVE-2019-17075: Denial-of-service in Chelsio T4/T5 RDMA TPT entries.
Install [scfnppae] CVE-2019-16746: Buffer overflow when receiving beacon over wireless network.
Install [s30q18pv] CVE-2020-14331: Out-of-bounds writes in ioctls of Console display driver.
Install [gljcytwa] CVE-2020-16166: Confidentiality vulnerability in the generation of the device ID.
Install [bz7pfng5] CVE-2019-3874: Denial-of-service by consuming a large amount of memory using SCTP socket.
Install [kkm3c4kt] CVE-2020-10781: Denial-of-service using Zram hot_add file sysfs entry.
Install [6mcud0xp] CVE-2019-17133: Denial-of-service in WiFI SIOCGIWESSID ioctl().
Install [6gp9cs5a] CVE-2018-14613: Multiple denial-of-services in the btrfs when mounting crafted images.
Install [b872hl5n] CVE-2019-14898: Denial-of-service when writing to file-max sysctl.
Install [q4ua18de] Channel recovery on transmition timeout in the Mellanox MLX5E driver.
Install [hu41z827] CVE-2019-18885: Denial-of-service in BTRFS extent verification.
Install [l3qgca72] CVE-2020-10767: Information leak using Spectre V2 attack due to IBPB being disabled.
Install [hzi9zld7] Denial-of-service when changing a paging attribute to non cachable.
Install [sygd7ev6] CVE-2020-25212: Out-of-bounds writes in RPC operations of Network File System.
Install [hgwk3m10] CVE-2018-20669: Privilege escalation in ioctl of i915 driver.
Install [8w3s90uz] Avoid page fault when updating the AMD IOMMU interrupt table.
Install [qhqi45un] CVE-2020-14386: Memory corruption when receiving a packet.
Install [h4n2u6me] CVE-2020-25284: Permission bypass when creating or removing a Rados block device.
Install [fih2hrl8] CVE-2020-25285: Denial-of-service when concurrently updating huge page sysctl parameters.
Install [3cogmz8j] CVE-2020-14314: Out-of-bounds memory read when splitting a directory block in the Ext4 filesystem.
Install [ig7kio2w] Use-after-free in the Oracle ASM driver when handling a query operation.
Install [7oirdwih] Re-factor memory cgroup statistic calculation.
Install [at9agcxc] Disable infiniband completion queue time stamping.
Install [eto8igfv] CVE-2019-19448: Use-after-free in Btrfs filesystem with a crafted btrfs filesystem image.
Install [rtplzq9k] CVE-2020-25641: Denial-of-service in biovec when zero-length biovec is issued.
Install [tqhmm9xx] CVE-2020-25643: Memory corruption in WAN HDLC-PPP due to missing error checking.
Install [stv272u2] CVE-2019-16089: Denial-of-service while checking NBD netlink status.
Install [dqw87z0a] CVE-2020-25211: Denial-of-service in Netfilter due to out-of-bounds memory access.
Install [fu9299l2] CVE-2020-14385: Denial of service in XFS filesystem.
Install [bua1sg0m] CVE-2019-19377: Use-after-free when unmounting a BTRFS image.
Install [oxfogcv8] CVE-2020-14356: NULL-pointer dereference in cgroupv2.
Install [l9d34kbr] CVE-2020-14390: Memory corruption when resizing the framebuffer.
Install [l6yeduhy] Race condition during iommu shutdown during a kernel panic.
Install [b2sygna8] CVE-2020-25645: Possible information leak between encrypted geneve endpoints.
Install [9pbc8q7r] CVE-2020-8694: Platypus Attack Mitigation.
Install [fecvmf3c] Clean up ftrace safety guard for existing Ksplice updates.
Install [n8ho14r0] Canceled RDS operations may still be executed.
Install [qzcfgssm] Use-after-free due to incorrect RDS operation status.
Install [i9zgalwk] Memory corruption when processing RDS extension headers.
Install [f5nnackz] CPU resource exhaustion when shrinking hash tables.
Install [4w115e3d] CVE-2020-12352: Information leak when handling AMP packets in Bluetooth stack.
Install [8927melz] Guest VM leaks bits into host control register, causing host to panic.
Install [pc7kfj5f] CVE-2019-19816: Invalid memory accesses during btrfs filesystem sync.
Install [gof9iub0] CVE-2020-25656: Use-after-free in console subsystem.
Install [nvdwvczo] CVE-2020-25668: Race condition when sending ioctls to a virtual terminal.
Install [iqua3ysi] CVE-2020-25704: Denial-of-service in the performance monitoring subsystem.
Install [pc4efmwh] CVE-2020-27675: Race condition when reconfiguring para-virtualized Xen devices.
Install [518x0407] CVE-2020-28974: Invalid memory access when manipulating framebuffer fonts.
Install [om575fk9] CVE-2020-28374: Access control bypass when reading or writing TCM devices.
Install [smtq0o16] CVE-2020-25705: ICMP rate-limiter can indirectly leak UDP port information.
Install [gxgp7md3] CVE-2020-28915: Information leak due to out-of-bounds read in Framebuffer Console.
Install [1ayz8ukr] CVE-2020-14351: Privilege escalation in perf subsystem due to use-after-free.
Install [jw5a37tf] CVE-2020-29569: Use-after-free when disconnecting Xen block devices.
Install [e1qgj5oi] Invalid bonding state with some network interfaces.
Install [4d68bkuy] Memory corruption in RDMA IO buffers.
Install [sxbva55v] Recover from memory pressure in the network layer.
Install [2hog7v7x] Flush the ARP cache when an RDMA interface changes its hardware address.
Install [tqysap9f] Avoid unneeded BUG_ON when closing RDS connections.
Install [9014r0f1] CVE-2020-15436: Use-after-free in blk device locks allows privilege escalation.
Install [5nnh5ltk] Buffer overflow when parsing some /proc/sys entries.
Install [7bfftrhd] CVE-2020-36158: Buffer overflow when creating an ad-hoc network.
Install [ddty3naj] Restrict NLM interval based host rebinding to UDP.
Install [deyfa0l1] CVE-2020-29660: Use-after-free in tty subsystem.
Install [hmbv5wjj] Possible missing files when iterating NFSv4 directories.
Install [2inwipov] CVE-2019-19947: Information leak in CAN Kvaser memory allocations.
Install [iahmns7p] CVE-2020-10768: Information leak using Spectre V2 gadgets due to incorrect prctl configuration.
Install [67ds1ur7] CVE-2020-24490: Privilege escalation in Bluetooth subsystem due to heap buffer overflow.
Install [h7cc0lf4] CVE-2019-18808: Memory leak in CCP device driver with invalid hash type.
Install [dp49y68f] CVE-2020-12351: Denial-of-service in L2CAP bluetooth driver.
Install [6vd15fp9] CVE-2021-26931, XSA-362: Mishandling of errors causes DoS of Xen backend.
Install [mbihi3b6] CVE-2021-26930, XSA-365: Bad error handing of blkback grant references.
Install [juvadtfe] CVE-2021-26932, XSA-361: Denial-of-host-service by malicious Xen frontend.
Install [s2yrku37] CVE-2019-19770: use-after-free in the debugfs from blktrace.
Install [cz2s2q2d] Improved update to CVE-2020-28915: Information leak due to out-of-bounds read in Framebuffer Console.
Install [bt9gct3d] Use-after-free in the networking TAP driver when handling a frame.
Install [bk6uwd1c] Migration failure in the Infiniband driver when an interface comes up after initialization.
Install [3r5qfqjf] Unecessary delays when allocation a virtual host SCSI device.
Install [oe1kvz5a] Avoid delaying the processing of completions in the infiniband driver.
Install [egsy21yt] Possible race condition whilst disconnecting SUNRPC connections.
Install [4za2h4oi] Avoid excessive memory usage from the infiniband driver.
Install [3pg4dv4d] High CPU utilization caused by lock contention in the zone page allocator.
Install [8f5avd0t] Possible kernel panic during IMPI reboot.
Install [45wwoufj] CVE-2021-3348: Use-after-free due to bad locking in Network block device.
Install [jqmhutsm] CVE-2021-3347: Privilege escalation in the Fast Userspace Mutexes.
Install [flm81vdd] CVE-2020-16120: Read permission bypass with overlay filesystem.
Install [mxu6ucv9] CVE-2021-27363, CVE-2021-27364, CVE-2021-27365: Priviledge escalation in iSCSI subsystem.
Install [ezqm5dui] Known exploit detection for CVE-2016-5195.
Install [71me89jl] Known exploit detection for CVE-2019-9213.
Install [ktm7khfr] CVE-2021-28038: Mishandling of errors causes DoS of Xen backend.
Install [jhckhjpw] Reduce allocation latency in Infiniband driver.
Install [ibyh7gbw] CVE-2020-27170, CVE-2020-27171: Information disclosure in BPF verifier.
Install [j1fc3g2n] CVE-2021-29605: Denial-of-Service in netfilter subsystem.
Install [qq5lrnft] Denial-of-service in the OCFS2 filesystem when setting file attributes
Install [evpvzwgz] CVE-2021-28688, XSA-371: Xen Hypervisor persistant grant leakage.
Install [226k0cjc] CVE-2021-28971: Denial-of-Service in Intel PEBS performance monitoring.
Install [hu2zpwc7] CVE-2021-28964: Race condition in btrfs filesystem.
Install [rg1g0t6f] CVE-2021-3428: Denial-of-Service in ext4 subsystem.
Install [jcfl6cfs] CVE-2021-29154: Code execution in eBPF JIT compiler.
Installing [edt48ylb] Add ftrace safety guard for existing Ksplice updates.
Installing [bipgvpw6] Known exploit detection.
Installing [lylzl1sj] Known exploit detection for CVE-2017-7308.
Installing [snvyltlq] Known exploit detection for CVE-2018-14634.
Installing [e2hf1ats] KPTI enablement for Ksplice.
Installing [1teh5owg] Known exploit detection for CVE-2018-18445.
Installing [44121bvv] Double free with SCSI LSI MPT Fusion SAS attach error.
Installing [trh9jthq] CVE-2019-3846: Heap overflow when parsing BSS descriptor in Marvell WiFi-Ex driver.
Installing [3cqjhtx3] CVE-2019-19054: Denial-of-service in the cx2388x tv card driver.
Installing [7q5nae2r] CVE-2019-15214: Use-after-free when connecting ALSA cards.
Installing [ff6vfhdf] CVE-2019-19536: Information leak when initializing PCAN-USB device.
Installing [ixsnbim2] CVE-2019-7308: Out-of-bounds speculation in BPF verifier.
Installing [2lm6v1an] CVE-2020-12770: Information leak/DoS in SCSI generic userspace write.
Installing [67w4v90f] CVE-2020-12464: Use-after-free in USB scatter-gather library.
Installing [4tnet5w5] CVE-2019-19534: Information leak using PEAK PCAN-USB/USB Pro interfaces for CAN 2.0b/CAN-FD.
Installing [9teqocf3] CVE-2020-12653: Denial-of-service when scanning for APs in mwifiex driver.
Installing [qdojji7h] CVE-2019-3846: Heap overflow when parsing fields in Marvell WiFi-Ex driver.
Installing [mcajph0v] CVE-2020-12654: Denial-of-service when querying WMM status in mwifiex driver.
Installing [3u8vegty] CVE-2020-10757: Flaw in DAX page mapping allows privilege escalation.
Installing [nlf0zswi] CVE-2020-10711: NULL pointer dereference when using CIPSO network packet labeling.
Installing [ptsm3729] CVE-2019-19533: Information leak in Technotrend/Hauppauge USB DEC driver.
Installing [5pv0zta2] CVE-2020-12655: Denial-of-service when syncing data on XFS filesystem.
Installing [aadmn3e5] Use-after-free when freeing received data over RDS socket.
Installing [c7221zqe] Buffer overflow when dumping registers in LSI Logic MegaRAID SAS RAID driver.
Installing [9elx8r6h] CVE-2020-12652: Denial-of-service in the Fusion MTP driver.
Installing [hwmxykva] Poor NFS performances caused by excessive attribute revalidation.
Installing [tgthi987] Denial-of-service when freezing and unfreezing an XFS filesystem.
Installing [cya3ogzy] Improved fix for CVE-2020-2732 when booting nested guests.
Installing [e966zc48] Denial-of-service in the Infiniband driver when referencing a node.
Installing [cvdwwuzd] Use-after-free in the Infiniband driver when releasing resources.
Installing [kjk2vpux] Race condition when sending IB subnet MAD causes denial-of-service.
Installing [hiaoskn9] CVE-2020-10766: Information leak using Spectre V4 variant.
Installing [t439t60g] NULL-pointer dereference when shutting down DSA switch.
Installing [t4avel7c] CVE-2019-19447: Use-after-free when unmounting corrupt ext4 filesystem.
Installing [r4pnegf6] CVE-2020-10732: Information leak in corefiles in per-thread info.
Installing [e9o5is2i] CVE-2019-19062: Denial-of-service in the crypto subsystem.
Installing [q7lx9ooy] CVE-2019-16234: NULL pointer dereference when registering Intel Wireless WiFi driver.
Installing [d1l1wlz8] Use-after-free when releasing clocks in PTP clock driver.
Installing [3adwuqzu] CVE-2019-19037: Denial-of-service when handling empty directories in ext4 filesystem.
Installing [mx9ibtwj] CVE-2019-16232: NULL pointer dereference when registering Marvell Libertas 8385/8686/8688 SDIO 802.11b/g cards.
Installing [jm6yd4li] Memory corruption during cgroup destruction with PSI enabled.
Installing [bb0tzpz2] Kernel crash in guest VM with machine check exception.
Installing [clivbxob] CVE-2019-20811: Denial-of-service in network device sysfs system.
Installing [tsh471fw] Add bit for guest kernel to handle kernel panic without host intervention.
Installing [mqhc4bmw] Don't return an ACK on some RDMA netlink operations.
Installing [4ixr7sgq] CVE-2018-20169: Missing bound check when reading extra USB descriptors.
Installing [3nkinjqi] CVE-2018-1000026: Denial-of-service when receiving invalid packet on bnx2x network card.
Installing [5f5w0oz6] CVE-2018-18281: Information leak in mremap syscall.
Installing [1a5n75aj] CVE-2019-19063: Denial-of-service in the rtlwifi driver.
Installing [19lbejmu] CVE-2019-0136: Denial-of-service in Intel(R) wifi driver.
Installing [anxgy28r] CVE-2018-20784: Denial-of-service in task scheduling.
Installing [1y4ivn6l] CVE-2018-20976: Use-after-free when mounting XFS filesystem.
Installing [dhmgixyw] CVE-2015-2150: Denial-of-service in Xen host from the guest.
Installing [jurvatrf] CVE-2019-19523: Use-after-free when disconnecting ADU USB devices.
Installing [3tcll1ov] CVE-2018-16882: Privilege escalation in nested Intel KVM interrupts.
Installing [od6vms9i] CVE-2019-19052: Memory leak when opening USB Socket CAN device driver.
Installing [byd1fjvz] CVE-2019-15927: Out-of-bounds accesses in usb audio driver.
Installing [rtg2jjko] CVE-2019-9506: Information disclosure when transmitting over bluetooth.
Installing [hgb8rgcc] CVE-2019-5108: Denial-of-service of a wireless access point during roaming of a station.
Installing [okxy9ag9] CVE-2020-10751: SELinux bypass in netlink message validation.
Installing [l8qz0cf1] CVE-2019-15918: Out-of-bounds access during CIFS mount.
Installing [fhoaweza] CVE-2019-2024: Use-after-free when disconnecting a Empia EM28xx USB device.
Installing [gphk4k5r] CVE-2020-13974: Integer overflow in virtual terminal keyboard interface.
Installing [iivjues9] CVE-2019-19528: Denial-of-service when disconnecting IO Warrior USB device.
Installing [11uqhn6c] CVE-2020-12114: Race condition in mountpoint counter causes DoS.
Installing [b9b9f3kf] CVE-2019-19807: Use-after-free when registering timer in ALSA driver.
Installing [hgpl3vss] CVE-2019-15218: Denial-of-service in Siano Mobile Digital TV USB tuner probing.
Installing [109qn2xz] CVE-2019-19530: Denial-of-service in USB CDC-ACM probing.
Installing [bhcvif0m] CVE-2020-11565: Out-of-bounds access when mounting tmpfs.
Installing [mveq6q0v] CVE-2019-2101: Information leak when initializing a usb video device.
Installing [96jk0uhc] CVE-2019-15117: Out-of-bounds access when parsing USB descriptor in ALSA USB driver.
Installing [ajidrtfz] Improved fix for CVE-2018-17972: Information leak in /proc kernel stack dumps.
Installing [uzg2h1sb] CVE-2019-19066: Denial-of-service int SCSI bfa driver.
Installing [5d8o75u5] CVE-2019-15118: Stack overflow when checking input source type in ALSA USB driver.
Installing [kj8qwnw5] CVE-2019-19051: Memory leak when changing power status of Intel Wireless WiMAX Connection 2400 driver.
Installing [dldi73w2] CVE-2018-1129: Signature check bypass of cephx message.
Installing [6g7nqusv] CVE-2019-3900: Infinite loop in vhost_net driver under heavy load.
Installing [fp67ub1v] CVE-2020-1749: Information disclosure in IPv6 IPSec tunneling.
Installing [jqobfvky] CVE-2019-11487: Invalid memory access when overflowing pages refcount.
Installing [o3vwjkah] CVE-2019-18805: Denial-of-service in IPv4 round trip time configuration.
Installing [r6xwmr9a] CVE-2019-19535, CVE-2019-19536: Information leak when initializing PCAN-USB device.
Installing [eirwnk56] CVE-2017-18552: Memory corruption in the RDS protocol.
Installing [8yot03uh] CVE-2019-15921: Denial-of-service in generic netlink socket family.
Installing [2thbpnu3] CVE-2019-20812: Soft lockup in packet sockets with zero timeout.
Installing [7c33rkhk] CVE-2019-9458: Use-after-free in V4L2 event subscription.
Installing [4grifv3a] CVE-2019-9455: Information leak in V4L2 when setting output buffer size.
Installing [fb5eodub] CVE-2019-19073, CVE-2019-19074: Denial-of-service in the ath9k wireless driver.
Installing [samo6fmn] CVE-2020-10720: Use-after-free in generic receive offload fragmentation.
Installing [ah33pp3h] CVE-2020-0305: Use-after-free when failing to open file on character device.
Installing [gm7wtw1w] CVE-2020-12771: Deadlock during BCache node coalesce failure.
Installing [ap0dbd8r] CVE-2019-15902: Bounds-check bypass in sys_ptrace().
Installing [32yey8b2] CVE-2019-10220: Privileges escalation when parsing directory from a bad SMB server.
Installing [t36cih2j] CVE-2020-8992: Deadlock with too big journal size on ext4 filesystem.
Installing [l16s2z79] CVE-2020-10769: Out-of-bounds memory access in authenticated encryption key parsing.
Installing [h2zoj70m] CVE-2014-9900: Information disclosure in Wake-On-LAN driver.
Installing [jgoq29rm] Improved fix for CVE-2019-19768: Use-after-free when reporting an IO trace.
Installing [baczzgxp] CVE-2019-19642: Denial-of-service in kernel relay file open path.
Installing [rb014za7] Incorrect reporting of Process Address Space ID on AMD systems.
Installing [9npfv9rb] Connection failure after RDS peer reboot.
Installing [b5lxrr63] CVE-2020-24394: Information leak when exporting a filesystem over NFS.
Installing [grdlpgdc] CVE-2019-17075: Denial-of-service in Chelsio T4/T5 RDMA TPT entries.
Installing [scfnppae] CVE-2019-16746: Buffer overflow when receiving beacon over wireless network.
Installing [s30q18pv] CVE-2020-14331: Out-of-bounds writes in ioctls of Console display driver.
Installing [gljcytwa] CVE-2020-16166: Confidentiality vulnerability in the generation of the device ID.
Installing [bz7pfng5] CVE-2019-3874: Denial-of-service by consuming a large amount of memory using SCTP socket.
Installing [kkm3c4kt] CVE-2020-10781: Denial-of-service using Zram hot_add file sysfs entry.
Installing [6mcud0xp] CVE-2019-17133: Denial-of-service in WiFI SIOCGIWESSID ioctl().
Installing [6gp9cs5a] CVE-2018-14613: Multiple denial-of-services in the btrfs when mounting crafted images.
Installing [b872hl5n] CVE-2019-14898: Denial-of-service when writing to file-max sysctl.
Installing [q4ua18de] Channel recovery on transmition timeout in the Mellanox MLX5E driver.
Installing [hu41z827] CVE-2019-18885: Denial-of-service in BTRFS extent verification.
Installing [l3qgca72] CVE-2020-10767: Information leak using Spectre V2 attack due to IBPB being disabled.
Installing [hzi9zld7] Denial-of-service when changing a paging attribute to non cachable.
Installing [sygd7ev6] CVE-2020-25212: Out-of-bounds writes in RPC operations of Network File System.
Installing [hgwk3m10] CVE-2018-20669: Privilege escalation in ioctl of i915 driver.
Installing [8w3s90uz] Avoid page fault when updating the AMD IOMMU interrupt table.
Installing [qhqi45un] CVE-2020-14386: Memory corruption when receiving a packet.
Installing [h4n2u6me] CVE-2020-25284: Permission bypass when creating or removing a Rados block device.
Installing [fih2hrl8] CVE-2020-25285: Denial-of-service when concurrently updating huge page sysctl parameters.
Installing [3cogmz8j] CVE-2020-14314: Out-of-bounds memory read when splitting a directory block in the Ext4 filesystem.
Installing [ig7kio2w] Use-after-free in the Oracle ASM driver when handling a query operation.
Installing [7oirdwih] Re-factor memory cgroup statistic calculation.
Installing [at9agcxc] Disable infiniband completion queue time stamping.
Installing [eto8igfv] CVE-2019-19448: Use-after-free in Btrfs filesystem with a crafted btrfs filesystem image.
Installing [rtplzq9k] CVE-2020-25641: Denial-of-service in biovec when zero-length biovec is issued.
Installing [tqhmm9xx] CVE-2020-25643: Memory corruption in WAN HDLC-PPP due to missing error checking.
Installing [stv272u2] CVE-2019-16089: Denial-of-service while checking NBD netlink status.
Installing [dqw87z0a] CVE-2020-25211: Denial-of-service in Netfilter due to out-of-bounds memory access.
Installing [fu9299l2] CVE-2020-14385: Denial of service in XFS filesystem.
Installing [bua1sg0m] CVE-2019-19377: Use-after-free when unmounting a BTRFS image.
Installing [oxfogcv8] CVE-2020-14356: NULL-pointer dereference in cgroupv2.
Installing [l9d34kbr] CVE-2020-14390: Memory corruption when resizing the framebuffer.
Installing [l6yeduhy] Race condition during iommu shutdown during a kernel panic.
Installing [b2sygna8] CVE-2020-25645: Possible information leak between encrypted geneve endpoints.
Installing [9pbc8q7r] CVE-2020-8694: Platypus Attack Mitigation.
Installing [fecvmf3c] Clean up ftrace safety guard for existing Ksplice updates.
Installing [n8ho14r0] Canceled RDS operations may still be executed.
Installing [qzcfgssm] Use-after-free due to incorrect RDS operation status.
Installing [i9zgalwk] Memory corruption when processing RDS extension headers.
Installing [f5nnackz] CPU resource exhaustion when shrinking hash tables.
Installing [4w115e3d] CVE-2020-12352: Information leak when handling AMP packets in Bluetooth stack.
Installing [8927melz] Guest VM leaks bits into host control register, causing host to panic.
Installing [pc7kfj5f] CVE-2019-19816: Invalid memory accesses during btrfs filesystem sync.
Installing [gof9iub0] CVE-2020-25656: Use-after-free in console subsystem.
Installing [nvdwvczo] CVE-2020-25668: Race condition when sending ioctls to a virtual terminal.
Installing [iqua3ysi] CVE-2020-25704: Denial-of-service in the performance monitoring subsystem.
Installing [pc4efmwh] CVE-2020-27675: Race condition when reconfiguring para-virtualized Xen devices.
Installing [518x0407] CVE-2020-28974: Invalid memory access when manipulating framebuffer fonts.
Installing [om575fk9] CVE-2020-28374: Access control bypass when reading or writing TCM devices.
Installing [smtq0o16] CVE-2020-25705: ICMP rate-limiter can indirectly leak UDP port information.
Installing [gxgp7md3] CVE-2020-28915: Information leak due to out-of-bounds read in Framebuffer Console.
Installing [1ayz8ukr] CVE-2020-14351: Privilege escalation in perf subsystem due to use-after-free.
Installing [jw5a37tf] CVE-2020-29569: Use-after-free when disconnecting Xen block devices.
Installing [e1qgj5oi] Invalid bonding state with some network interfaces.
Installing [4d68bkuy] Memory corruption in RDMA IO buffers.
Installing [sxbva55v] Recover from memory pressure in the network layer.
Installing [2hog7v7x] Flush the ARP cache when an RDMA interface changes its hardware address.
Installing [tqysap9f] Avoid unneeded BUG_ON when closing RDS connections.
Installing [9014r0f1] CVE-2020-15436: Use-after-free in blk device locks allows privilege escalation.
Installing [5nnh5ltk] Buffer overflow when parsing some /proc/sys entries.
Installing [7bfftrhd] CVE-2020-36158: Buffer overflow when creating an ad-hoc network.
Installing [ddty3naj] Restrict NLM interval based host rebinding to UDP.
Installing [deyfa0l1] CVE-2020-29660: Use-after-free in tty subsystem.
Installing [hmbv5wjj] Possible missing files when iterating NFSv4 directories.
Installing [2inwipov] CVE-2019-19947: Information leak in CAN Kvaser memory allocations.
Installing [iahmns7p] CVE-2020-10768: Information leak using Spectre V2 gadgets due to incorrect prctl configuration.
Installing [67ds1ur7] CVE-2020-24490: Privilege escalation in Bluetooth subsystem due to heap buffer overflow.
Installing [h7cc0lf4] CVE-2019-18808: Memory leak in CCP device driver with invalid hash type.
Installing [dp49y68f] CVE-2020-12351: Denial-of-service in L2CAP bluetooth driver.
Installing [6vd15fp9] CVE-2021-26931, XSA-362: Mishandling of errors causes DoS of Xen backend.
Installing [mbihi3b6] CVE-2021-26930, XSA-365: Bad error handing of blkback grant references.
Installing [juvadtfe] CVE-2021-26932, XSA-361: Denial-of-host-service by malicious Xen frontend.
Installing [s2yrku37] CVE-2019-19770: use-after-free in the debugfs from blktrace.
Installing [cz2s2q2d] Improved update to CVE-2020-28915: Information leak due to out-of-bounds read in Framebuffer Console.
Installing [bt9gct3d] Use-after-free in the networking TAP driver when handling a frame.
Installing [bk6uwd1c] Migration failure in the Infiniband driver when an interface comes up after initialization.
Installing [3r5qfqjf] Unecessary delays when allocation a virtual host SCSI device.
Installing [oe1kvz5a] Avoid delaying the processing of completions in the infiniband driver.
Installing [egsy21yt] Possible race condition whilst disconnecting SUNRPC connections.
Installing [4za2h4oi] Avoid excessive memory usage from the infiniband driver.
Installing [3pg4dv4d] High CPU utilization caused by lock contention in the zone page allocator.
Installing [8f5avd0t] Possible kernel panic during IMPI reboot.
Installing [45wwoufj] CVE-2021-3348: Use-after-free due to bad locking in Network block device.
Installing [jqmhutsm] CVE-2021-3347: Privilege escalation in the Fast Userspace Mutexes.
Installing [flm81vdd] CVE-2020-16120: Read permission bypass with overlay filesystem.
Installing [mxu6ucv9] CVE-2021-27363, CVE-2021-27364, CVE-2021-27365: Priviledge escalation in iSCSI subsystem.
Installing [ezqm5dui] Known exploit detection for CVE-2016-5195.
Installing [71me89jl] Known exploit detection for CVE-2019-9213.
Installing [ktm7khfr] CVE-2021-28038: Mishandling of errors causes DoS of Xen backend.
Installing [jhckhjpw] Reduce allocation latency in Infiniband driver.
Installing [ibyh7gbw] CVE-2020-27170, CVE-2020-27171: Information disclosure in BPF verifier.
Installing [j1fc3g2n] CVE-2021-29605: Denial-of-Service in netfilter subsystem.
Installing [qq5lrnft] Denial-of-service in the OCFS2 filesystem when setting file attributes
Installing [evpvzwgz] CVE-2021-28688, XSA-371: Xen Hypervisor persistant grant leakage.
Installing [226k0cjc] CVE-2021-28971: Denial-of-Service in Intel PEBS performance monitoring.
Installing [hu2zpwc7] CVE-2021-28964: Race condition in btrfs filesystem.
Installing [rg1g0t6f] CVE-2021-3428: Denial-of-Service in ext4 subsystem.
Installing [jcfl6cfs] CVE-2021-29154: Code execution in eBPF JIT compiler.
Your kernel is fully up to date.
Effective kernel version is 4.14.35-2047.503.1.el7uek
  Verifying  : uptrack-updates-4.14.35-1902.303.5.3.el7uek.x86_64-20210512-0.noarch                                                                                                                    1/1 

Installed:
  uptrack-updates-4.14.35-1902.303.5.3.el7uek.x86_64.noarch 0:20210512-0    
Verificando correções aplicadas:

[root@dbm0db02 ~]# uptrack-show
Installed updates:
[bipgvpw6] Known exploit detection.
[lylzl1sj] Known exploit detection for CVE-2017-7308.
[snvyltlq] Known exploit detection for CVE-2018-14634.
[e2hf1ats] KPTI enablement for Ksplice.
[1teh5owg] Known exploit detection for CVE-2018-18445.
[44121bvv] Double free with SCSI LSI MPT Fusion SAS attach error.
[trh9jthq] CVE-2019-3846: Heap overflow when parsing BSS descriptor in Marvell WiFi-Ex driver.
[3cqjhtx3] CVE-2019-19054: Denial-of-service in the cx2388x tv card driver.
[7q5nae2r] CVE-2019-15214: Use-after-free when connecting ALSA cards.
[ff6vfhdf] CVE-2019-19536: Information leak when initializing PCAN-USB device.
[ixsnbim2] CVE-2019-7308: Out-of-bounds speculation in BPF verifier.
[2lm6v1an] CVE-2020-12770: Information leak/DoS in SCSI generic userspace write.
[67w4v90f] CVE-2020-12464: Use-after-free in USB scatter-gather library.
[4tnet5w5] CVE-2019-19534: Information leak using PEAK PCAN-USB/USB Pro interfaces for CAN 2.0b/CAN-FD.
[9teqocf3] CVE-2020-12653: Denial-of-service when scanning for APs in mwifiex driver.
[qdojji7h] CVE-2019-3846: Heap overflow when parsing fields in Marvell WiFi-Ex driver.
[mcajph0v] CVE-2020-12654: Denial-of-service when querying WMM status in mwifiex driver.
[3u8vegty] CVE-2020-10757: Flaw in DAX page mapping allows privilege escalation.
[nlf0zswi] CVE-2020-10711: NULL pointer dereference when using CIPSO network packet labeling.
[ptsm3729] CVE-2019-19533: Information leak in Technotrend/Hauppauge USB DEC driver.
[5pv0zta2] CVE-2020-12655: Denial-of-service when syncing data on XFS filesystem.
[aadmn3e5] Use-after-free when freeing received data over RDS socket.
[c7221zqe] Buffer overflow when dumping registers in LSI Logic MegaRAID SAS RAID driver.
[9elx8r6h] CVE-2020-12652: Denial-of-service in the Fusion MTP driver.
[hwmxykva] Poor NFS performances caused by excessive attribute revalidation.
[tgthi987] Denial-of-service when freezing and unfreezing an XFS filesystem.
[cya3ogzy] Improved fix for CVE-2020-2732 when booting nested guests.
[e966zc48] Denial-of-service in the Infiniband driver when referencing a node.
[cvdwwuzd] Use-after-free in the Infiniband driver when releasing resources.
[kjk2vpux] Race condition when sending IB subnet MAD causes denial-of-service.
[hiaoskn9] CVE-2020-10766: Information leak using Spectre V4 variant.
[t439t60g] NULL-pointer dereference when shutting down DSA switch.
[t4avel7c] CVE-2019-19447: Use-after-free when unmounting corrupt ext4 filesystem.
[r4pnegf6] CVE-2020-10732: Information leak in corefiles in per-thread info.
[e9o5is2i] CVE-2019-19062: Denial-of-service in the crypto subsystem.
[q7lx9ooy] CVE-2019-16234: NULL pointer dereference when registering Intel Wireless WiFi driver.
[d1l1wlz8] Use-after-free when releasing clocks in PTP clock driver.
[3adwuqzu] CVE-2019-19037: Denial-of-service when handling empty directories in ext4 filesystem.
[mx9ibtwj] CVE-2019-16232: NULL pointer dereference when registering Marvell Libertas 8385/8686/8688 SDIO 802.11b/g cards.
[jm6yd4li] Memory corruption during cgroup destruction with PSI enabled.
[bb0tzpz2] Kernel crash in guest VM with machine check exception.
[clivbxob] CVE-2019-20811: Denial-of-service in network device sysfs system.
[tsh471fw] Add bit for guest kernel to handle kernel panic without host intervention.
[mqhc4bmw] Don't return an ACK on some RDMA netlink operations.
[4ixr7sgq] CVE-2018-20169: Missing bound check when reading extra USB descriptors.
[3nkinjqi] CVE-2018-1000026: Denial-of-service when receiving invalid packet on bnx2x network card.
[5f5w0oz6] CVE-2018-18281: Information leak in mremap syscall.
[1a5n75aj] CVE-2019-19063: Denial-of-service in the rtlwifi driver.
[19lbejmu] CVE-2019-0136: Denial-of-service in Intel(R) wifi driver.
[1y4ivn6l] CVE-2018-20976: Use-after-free when mounting XFS filesystem.
[dhmgixyw] CVE-2015-2150: Denial-of-service in Xen host from the guest.
[jurvatrf] CVE-2019-19523: Use-after-free when disconnecting ADU USB devices.
[3tcll1ov] CVE-2018-16882: Privilege escalation in nested Intel KVM interrupts.
[od6vms9i] CVE-2019-19052: Memory leak when opening USB Socket CAN device driver.
[byd1fjvz] CVE-2019-15927: Out-of-bounds accesses in usb audio driver.
[rtg2jjko] CVE-2019-9506: Information disclosure when transmitting over bluetooth.
[hgb8rgcc] CVE-2019-5108: Denial-of-service of a wireless access point during roaming of a station.
[okxy9ag9] CVE-2020-10751: SELinux bypass in netlink message validation.
[l8qz0cf1] CVE-2019-15918: Out-of-bounds access during CIFS mount.
[fhoaweza] CVE-2019-2024: Use-after-free when disconnecting a Empia EM28xx USB device.
[gphk4k5r] CVE-2020-13974: Integer overflow in virtual terminal keyboard interface.
[iivjues9] CVE-2019-19528: Denial-of-service when disconnecting IO Warrior USB device.
[11uqhn6c] CVE-2020-12114: Race condition in mountpoint counter causes DoS.
[b9b9f3kf] CVE-2019-19807: Use-after-free when registering timer in ALSA driver.
[hgpl3vss] CVE-2019-15218: Denial-of-service in Siano Mobile Digital TV USB tuner probing.
[109qn2xz] CVE-2019-19530: Denial-of-service in USB CDC-ACM probing.
[bhcvif0m] CVE-2020-11565: Out-of-bounds access when mounting tmpfs.
[mveq6q0v] CVE-2019-2101: Information leak when initializing a usb video device.
[96jk0uhc] CVE-2019-15117: Out-of-bounds access when parsing USB descriptor in ALSA USB driver.
[ajidrtfz] Improved fix for CVE-2018-17972: Information leak in /proc kernel stack dumps.
[uzg2h1sb] CVE-2019-19066: Denial-of-service int SCSI bfa driver.
[5d8o75u5] CVE-2019-15118: Stack overflow when checking input source type in ALSA USB driver.
[kj8qwnw5] CVE-2019-19051: Memory leak when changing power status of Intel Wireless WiMAX Connection 2400 driver.
[dldi73w2] CVE-2018-1129: Signature check bypass of cephx message.
[6g7nqusv] CVE-2019-3900: Infinite loop in vhost_net driver under heavy load.
[fp67ub1v] CVE-2020-1749: Information disclosure in IPv6 IPSec tunneling.
[jqobfvky] CVE-2019-11487: Invalid memory access when overflowing pages refcount.
[o3vwjkah] CVE-2019-18805: Denial-of-service in IPv4 round trip time configuration.
[r6xwmr9a] CVE-2019-19535, CVE-2019-19536: Information leak when initializing PCAN-USB device.
[eirwnk56] CVE-2017-18552: Memory corruption in the RDS protocol.
[8yot03uh] CVE-2019-15921: Denial-of-service in generic netlink socket family.
[2thbpnu3] CVE-2019-20812: Soft lockup in packet sockets with zero timeout.
[7c33rkhk] CVE-2019-9458: Use-after-free in V4L2 event subscription.
[4grifv3a] CVE-2019-9455: Information leak in V4L2 when setting output buffer size.
[fb5eodub] CVE-2019-19073, CVE-2019-19074: Denial-of-service in the ath9k wireless driver.
[samo6fmn] CVE-2020-10720: Use-after-free in generic receive offload fragmentation.
[ah33pp3h] CVE-2020-0305: Use-after-free when failing to open file on character device.
[gm7wtw1w] CVE-2020-12771: Deadlock during BCache node coalesce failure.
[ap0dbd8r] CVE-2019-15902: Bounds-check bypass in sys_ptrace().
[32yey8b2] CVE-2019-10220: Privileges escalation when parsing directory from a bad SMB server.
[t36cih2j] CVE-2020-8992: Deadlock with too big journal size on ext4 filesystem.
[l16s2z79] CVE-2020-10769: Out-of-bounds memory access in authenticated encryption key parsing.
[h2zoj70m] CVE-2014-9900: Information disclosure in Wake-On-LAN driver.
[jgoq29rm] Improved fix for CVE-2019-19768: Use-after-free when reporting an IO trace.
[baczzgxp] CVE-2019-19642: Denial-of-service in kernel relay file open path.
[rb014za7] Incorrect reporting of Process Address Space ID on AMD systems.
[9npfv9rb] Connection failure after RDS peer reboot.
[b5lxrr63] CVE-2020-24394: Information leak when exporting a filesystem over NFS.
[grdlpgdc] CVE-2019-17075: Denial-of-service in Chelsio T4/T5 RDMA TPT entries.
[scfnppae] CVE-2019-16746: Buffer overflow when receiving beacon over wireless network.
[s30q18pv] CVE-2020-14331: Out-of-bounds writes in ioctls of Console display driver.
[gljcytwa] CVE-2020-16166: Confidentiality vulnerability in the generation of the device ID.
[bz7pfng5] CVE-2019-3874: Denial-of-service by consuming a large amount of memory using SCTP socket.
[kkm3c4kt] CVE-2020-10781: Denial-of-service using Zram hot_add file sysfs entry.
[6mcud0xp] CVE-2019-17133: Denial-of-service in WiFI SIOCGIWESSID ioctl().
[6gp9cs5a] CVE-2018-14613: Multiple denial-of-services in the btrfs when mounting crafted images.
[b872hl5n] CVE-2019-14898: Denial-of-service when writing to file-max sysctl.
[q4ua18de] Channel recovery on transmition timeout in the Mellanox MLX5E driver.
[hu41z827] CVE-2019-18885: Denial-of-service in BTRFS extent verification.
[l3qgca72] CVE-2020-10767: Information leak using Spectre V2 attack due to IBPB being disabled.
[hzi9zld7] Denial-of-service when changing a paging attribute to non cachable.
[sygd7ev6] CVE-2020-25212: Out-of-bounds writes in RPC operations of Network File System.
[hgwk3m10] CVE-2018-20669: Privilege escalation in ioctl of i915 driver.
[qhqi45un] CVE-2020-14386: Memory corruption when receiving a packet.
[h4n2u6me] CVE-2020-25284: Permission bypass when creating or removing a Rados block device.
[fih2hrl8] CVE-2020-25285: Denial-of-service when concurrently updating huge page sysctl parameters.
[3cogmz8j] CVE-2020-14314: Out-of-bounds memory read when splitting a directory block in the Ext4 filesystem.
[ig7kio2w] Use-after-free in the Oracle ASM driver when handling a query operation.
[7oirdwih] Re-factor memory cgroup statistic calculation.
[at9agcxc] Disable infiniband completion queue time stamping.
[eto8igfv] CVE-2019-19448: Use-after-free in Btrfs filesystem with a crafted btrfs filesystem image.
[rtplzq9k] CVE-2020-25641: Denial-of-service in biovec when zero-length biovec is issued.
[tqhmm9xx] CVE-2020-25643: Memory corruption in WAN HDLC-PPP due to missing error checking.
[stv272u2] CVE-2019-16089: Denial-of-service while checking NBD netlink status.
[dqw87z0a] CVE-2020-25211: Denial-of-service in Netfilter due to out-of-bounds memory access.
[fu9299l2] CVE-2020-14385: Denial of service in XFS filesystem.
[bua1sg0m] CVE-2019-19377: Use-after-free when unmounting a BTRFS image.
[oxfogcv8] CVE-2020-14356: NULL-pointer dereference in cgroupv2.
[l9d34kbr] CVE-2020-14390: Memory corruption when resizing the framebuffer.
[l6yeduhy] Race condition during iommu shutdown during a kernel panic.
[b2sygna8] CVE-2020-25645: Possible information leak between encrypted geneve endpoints.
[9pbc8q7r] CVE-2020-8694: Platypus Attack Mitigation.
[edt48ylb] Add ftrace safety guard for existing Ksplice updates.
[fecvmf3c] Clean up ftrace safety guard for existing Ksplice updates.
[n8ho14r0] Canceled RDS operations may still be executed.
[qzcfgssm] Use-after-free due to incorrect RDS operation status.
[i9zgalwk] Memory corruption when processing RDS extension headers.
[8w3s90uz] Avoid page fault when updating the AMD IOMMU interrupt table.
[f5nnackz] CPU resource exhaustion when shrinking hash tables.
[4w115e3d] CVE-2020-12352: Information leak when handling AMP packets in Bluetooth stack.
[8927melz] Guest VM leaks bits into host control register, causing host to panic.
[pc7kfj5f] CVE-2019-19816: Invalid memory accesses during btrfs filesystem sync.
[gof9iub0] CVE-2020-25656: Use-after-free in console subsystem.
[nvdwvczo] CVE-2020-25668: Race condition when sending ioctls to a virtual terminal.
[iqua3ysi] CVE-2020-25704: Denial-of-service in the performance monitoring subsystem.
[518x0407] CVE-2020-28974: Invalid memory access when manipulating framebuffer fonts.
[om575fk9] CVE-2020-28374: Access control bypass when reading or writing TCM devices.
[anxgy28r] CVE-2018-20784: Denial-of-service in task scheduling.
[smtq0o16] CVE-2020-25705: ICMP rate-limiter can indirectly leak UDP port information.
[gxgp7md3] CVE-2020-28915: Information leak due to out-of-bounds read in Framebuffer Console.
[1ayz8ukr] CVE-2020-14351: Privilege escalation in perf subsystem due to use-after-free.
[jw5a37tf] CVE-2020-29569: Use-after-free when disconnecting Xen block devices.
[e1qgj5oi] Invalid bonding state with some network interfaces.
[4d68bkuy] Memory corruption in RDMA IO buffers.
[sxbva55v] Recover from memory pressure in the network layer.
[2hog7v7x] Flush the ARP cache when an RDMA interface changes its hardware address.
[tqysap9f] Avoid unneeded BUG_ON when closing RDS connections.
[9014r0f1] CVE-2020-15436: Use-after-free in blk device locks allows privilege escalation.
[5nnh5ltk] Buffer overflow when parsing some /proc/sys entries.
[7bfftrhd] CVE-2020-36158: Buffer overflow when creating an ad-hoc network.
[ddty3naj] Restrict NLM interval based host rebinding to UDP.
[deyfa0l1] CVE-2020-29660: Use-after-free in tty subsystem.
[hmbv5wjj] Possible missing files when iterating NFSv4 directories.
[2inwipov] CVE-2019-19947: Information leak in CAN Kvaser memory allocations.
[iahmns7p] CVE-2020-10768: Information leak using Spectre V2 gadgets due to incorrect prctl configuration.
[67ds1ur7] CVE-2020-24490: Privilege escalation in Bluetooth subsystem due to heap buffer overflow.
[h7cc0lf4] CVE-2019-18808: Memory leak in CCP device driver with invalid hash type.
[dp49y68f] CVE-2020-12351: Denial-of-service in L2CAP bluetooth driver.
[6vd15fp9] CVE-2021-26931, XSA-362: Mishandling of errors causes DoS of Xen backend.
[mbihi3b6] CVE-2021-26930, XSA-365: Bad error handing of blkback grant references.
[juvadtfe] CVE-2021-26932, XSA-361: Denial-of-host-service by malicious Xen frontend.
[s2yrku37] CVE-2019-19770: use-after-free in the debugfs from blktrace.
[cz2s2q2d] Improved update to CVE-2020-28915: Information leak due to out-of-bounds read in Framebuffer Console.
[bt9gct3d] Use-after-free in the networking TAP driver when handling a frame.
[bk6uwd1c] Migration failure in the Infiniband driver when an interface comes up after initialization.
[3r5qfqjf] Unecessary delays when allocation a virtual host SCSI device.
[oe1kvz5a] Avoid delaying the processing of completions in the infiniband driver.
[egsy21yt] Possible race condition whilst disconnecting SUNRPC connections.
[4za2h4oi] Avoid excessive memory usage from the infiniband driver.
[3pg4dv4d] High CPU utilization caused by lock contention in the zone page allocator.
[8f5avd0t] Possible kernel panic during IMPI reboot.
[45wwoufj] CVE-2021-3348: Use-after-free due to bad locking in Network block device.
[jqmhutsm] CVE-2021-3347: Privilege escalation in the Fast Userspace Mutexes.
[flm81vdd] CVE-2020-16120: Read permission bypass with overlay filesystem.
[mxu6ucv9] CVE-2021-27363, CVE-2021-27364, CVE-2021-27365: Priviledge escalation in iSCSI subsystem.
[pc4efmwh] CVE-2020-27675: Race condition when reconfiguring para-virtualized Xen devices.
[ezqm5dui] Known exploit detection for CVE-2016-5195.
[71me89jl] Known exploit detection for CVE-2019-9213.
[ktm7khfr] CVE-2021-28038: Mishandling of errors causes DoS of Xen backend.
[jhckhjpw] Reduce allocation latency in Infiniband driver.
[ibyh7gbw] CVE-2020-27170, CVE-2020-27171: Information disclosure in BPF verifier.
[j1fc3g2n] CVE-2021-29605: Denial-of-Service in netfilter subsystem.
[qq5lrnft] Denial-of-service in the OCFS2 filesystem when setting file attributes
[evpvzwgz] CVE-2021-28688, XSA-371: Xen Hypervisor persistant grant leakage.
[226k0cjc] CVE-2021-28971: Denial-of-Service in Intel PEBS performance monitoring.
[hu2zpwc7] CVE-2021-28964: Race condition in btrfs filesystem.
[rg1g0t6f] CVE-2021-3428: Denial-of-Service in ext4 subsystem.
[jcfl6cfs] CVE-2021-29154: Code execution in eBPF JIT compiler.

Effective kernel version is 4.14.35-2047.503.1.el7uek
[root@dbm0db02 ~]# uptrack-show --available
Available updates:
None
```

Sempre consulte a nota 2207063.1 antes de iniciar o processo acima descrito, algo pode mudar com o lançamento de novas versões.

Espero que aproveitem :)