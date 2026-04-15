---
title: "Ksplice: Atualizando CVEs"
datePublished: 2026-04-15T23:14:38.865Z
cuid: cmo0o3wcw000u1qmdc65wb0o3
slug: ksplice-atualizando-cves
cover: https://cdn.hashnode.com/uploads/covers/677046341c3ce68f37f2ae3c/8290bce2-e206-4ad1-96c5-721556a85558.png

---

No meu outro artigo sobre Ksplice ([aqui](https://diogofernandes.com.br/ksplice-configuracao-e-instalacao-em-ambientes-oracle-linux)), demonstrei como instalar, configurar e aplicar os pacotes de segurança (CVEs) do Ksplice.

Neste artigo, vou mostrar como atualizar esses pacotes, como instalar a versão mais recente do Ksplice e como manter o sistema operacional com o nível de segurança mais elevado possível.

Com tudo já devidamente configurado, conforme demonstrado no [artigo anterior](https://diogofernandes.com.br/ksplice-configuracao-e-instalacao-em-ambientes-oracle-linux), o processo de manutenção é bastante simples e envolve apenas três etapas:

Primeiro Passo: Verificar a versão atualmente instalada.

```bash
[root@srvora02 ~]# uptrack-uname -r
5.15.0-302.167.6.1.el9uek.x86_64
```

Segundo Passo: Instalar a versão mais recente disponível.

```bash
/usr/bin/python2 /usr/sbin/uptrack-upgrade -y --all
```

Terceiro Passo: Acompanhar a atualização.

```bash
The following steps will be taken:
Remove [lx0gzdnf] CVE-2025-38457: Null pointer dereference in QoS and/or fair queueing driver.
Remove [iw8wjvh1] CVE-2025-38477: Use-after-free in Quick Fair Queueing scheduler (QFQ) driver.
Remove [gqkk45sd] CVE-2025-38107: Integer overflow in Enhanced transmission selection scheduler (ETS).
Remove [387haf5e] CVE-2025-38193: Integer overflow in Stochastic Fairness Queueing (SFQ) driver.
Remove [cdo10nt9] CVE-2025-38083, CVE-2025-38108: Integer underflow in multiple network schedulers.
Remove [7izd5nbg] CVE-2025-37890, CVE-2025-38000, CVE-2025-38001, CVE-2025-38350: Use-after-free in HFSC network scheduler.
Remove [11p6pga8] CVE-2025-37798, CVE-2025-38177, CVE-2025-37932, CVE-2025-37953: Use-after-free in multiple network schedulers.
Install [aostn7o7] Protective update for Null pointer dereference in RDS Protocol driver.
Install [2p8a0k0d] Improvement update for Null pointer dereference in RDS Protocol driver.
Install [jvz2patb] CVE-2025-37798, CVE-2025-38177, CVE-2025-37932, CVE-2025-37953: Use-after-free in multiple network schedulers.
Install [8sqc7c0k] CVE-2025-37890, CVE-2025-38000, CVE-2025-38001, CVE-2025-38350: Use-after-free in HFSC network scheduler.
Install [ly6x1hsg] CVE-2025-38083, CVE-2025-38108: Integer underflow in multiple network schedulers.
Install [igyv8377] CVE-2025-38193: Integer overflow in Stochastic Fairness Queueing (SFQ) driver.
Install [rnpo6r7o] CVE-2025-38107: Integer overflow in Enhanced transmission selection scheduler (ETS).
Install [iw8wjvh1] CVE-2025-38477: Use-after-free in Quick Fair Queueing scheduler (QFQ) driver.
Install [pc74lj2k] CVE-2025-38457: Null pointer dereference in QoS and/or fair queueing driver.
Install [dnrtq922] CVE-2025-38468: Kernel oops in Hierarchical Token Bucket network scheduler.
Install [dqagdwkf] CVE-2025-38684: Null pointer dereference in Packet Scheduler subsystem.
Install [d6kl1l8h] CVE-2025-38024: Use-after-free in Software RDMA over Ethernet (RoCE) driver.
Install [kugcl16z] CVE-2025-38622: Kernel oops in UDP networking stack.
Install [f218n88x] CVE-2025-40019: Missing check in block encryption ESSIV support.
Install [tdluc30k] CVE-2025-39885: Deadlock in OCFS2 filesystem driver.
Install [6wokg8qv] CVE-2025-39955, CVE-2025-40186: Dropped packets in TCP/IP networking driver.
Install [d9c9apzb] CVE-2025-40027: Use-after-free in 9P protocol stack.
Install [nua1eqsl] CVE-2025-40018: Use-after-free in IP virtual server (FTP) driver.
Install [f10dn542] CVE-2025-40115: Use-after-free in LSI Fusion-MPT SAS driver.
Install [dgxqpxvg] CVE-2025-40055: Use-after-free in OCFS2 filesystem.
Install [ogor6blh] CVE-2025-40048: Hang in Generic Hyper-V VMBus userspace I/O driver.
Install [i6gtoxbd] CVE-2025-40044: Out-of-bounds memory access in UDF filesystem.
Install [m0lj1tzl] CVE-2025-40153: Soft lockup in HugeTLB filesystem.
Install [c6fyocml] CVE-2025-40111: Use-after-free in VMware vGPU driver.
Install [e1f00opd] CVE-2025-40187: Null pointer dereference in SCTP Protocol driver.
Install [kd50bj1e] CVE-2025-40205: Out-of-bounds memory access in BTRFS filesystem.
Install [8ai5p4ry] CVE-2025-40194: Kernel crash in Intel P state control driver.
Install [5ckw5kra] CVE-2025-40204: Variable-time MAC check in SCTP Protocol driver.
Install [gbwhmb1k] CVE-2025-40134: Null pointer dereference in multi-device driver (RAID/LVM).
Install [hwzbva2o] CVE-2025-40178: Null pointer dereference in core kernel.
Install [6kzoiuc0] CVE-2025-40173: Use-after-free in IPv6 networking stack.
Install [hmql959q] CVE-2025-40105: Memory leak in core filesystem layer.
Install [h6ubsv5w] CVE-2025-40026: Race in x86 KVM.
Install [k9l2nqad] Logic error in Transport Layer Security driver.
Install [j8sn6w7i] CVE-2025-40087: Remote null pointer dereference in Parallel NFS FlexFile server driver.
Install [h52c77mt] CVE-2025-40240: Remote null pointer dereference in SCTP Protocol driver.
Install [kar08ekj] CVE-2025-38678: Kernel oops in Netfilter driver.
Install [qtz7bm22] CVE-2025-40233: Kernel crash in OCFS2 filesystem driver.
Install [nqh664s7] CVE-2025-40231: Deadlock in Virtual Socket protocol driver.
Install [p7vygkv1] CVE-2025-22056: Out-of-bounds memory access in Netfilter tunnel driver.
Install [i7n5xk5p] CVE-2024-50257: Use-after-free in Netfilter Xtables driver.
Install [fu6dwsct] CVE-2025-40248: Use-after-free in Virtual Socket protocol driver.
Install [d8vo4m2h] CVE-2025-40271: Use-after-free in /proc filesystem driver.
Install [5g7smk83] CVE-2025-40280: Use-after-free in TIPC Protocol driver.
Install [pyek1z1n] Known exploit detection for CVE-2023-4015.
Install [4f9bsw52] Known exploit detection for CVE-2023-5197.
Install [2lggz22s] Performance improvement for kallsyms lookup.
Install [roubjmj4] CVE-2025-40258: Use-after-free in MPTCP: Multipath TCP driver.
Install [a09xhhyp] CVE-2025-68209: Null pointer dereference in Mellanox devices driver.
Install [h1y3arqq] CVE-2025-40256, CVE-2025-40215: Race condition in XFRM subsystem.
Install [md0rp234] CVE-2025-38571: Use-after-free in NFS over TLS.
Install [k2js8ari] CVE-2025-38566: Remote denial-of-service in SUNRPC.
Install [smeektn8] CVE-2025-40083: Null pointer dereference in Quick Fair Queueing scheduler (QFQ) driver.
Install [itlqzz4z] CVE-2025-40281: Out-of-bounds memory access in SCTP Protocol driver.
Install [fcc5gg0s] CVE-2025-40277: Out-of-bounds memory access in VMware graphics driver.
Install [pj9mhss8] CVE-2025-40273: Kernel oops in NFS server for NFS version 4 driver.
Install [5xelf337] CVE-2025-68301: Out-of-bounds memory access in aQuantia AQtion driver.
Install [i9jxp7n1] CVE-2023-24023: Authentication bypass when pairing Bluetooth devices.
Install [afhh5976] CVE-2025-40040: Kernel panic in KSM for page merging driver.
Install [a6sme3hu] CVE-2024-37354: Denial-of-service in Btrfs filesystem driver.
Install [g5zqvsnh] Performance regression in TCP/IP networking driver.
Install [kkpy2t0b] CVE-2025-40264: Null pointer dereference in BladeEngine driver.
Install [k0mbwxcn] CVE-2025-40272: Use-after-free in secret memory management.
Install [ibpiufuf] CVE-2025-68295: Memory leak in SMB/CIFS client driver.
Install [f3abo2gy] CVE-2024-47666: Kernel crash in PMC-Sierra SPC 8001 SAS/SATA Based Host Adapter driver.
Install [16mqfnr3] CVE-2025-40257: Use-after-free in MPTCP: Multipath TCP driver.
Install [r7uqufh0] CVE-2024-43888: Use-after-free in LRU list implementation.
Install [r9z9ei4b] CVE-2025-40149: Use-after-free in Transport Layer Security HW offload driver.
Install [dsc70x0h] Use-after-free in x86 KVM.
Install [c1ej2xno] CVE-2025-71066: Use-after-free in ETS network scheduler.
Install [m8f04wlo] CVE-2025-68813: Null pointer dereference in IP virtual server driver.
Install [2mzhguj3] CVE-2025-68776: Null pointer dereference in High-availability Seamless Redundancy (HSR & PRP) driver.
Install [3nb1rndm] CVE-2025-71147: Memory leak in TPM-based trusted keys driver.
Install [19b8c1q1] CVE-2025-68818: Null pointer dereference in QLogic QLA2XXX Fibre Channel driver.
Install [kt172cv8] CVE-2025-71104: Hard lockup in KVM.
Install [8zwirihy] CVE-2025-68788: Information leak in fsnotify.
Install [eh3y43j5] CVE-2025-71131: Use-after-free in Sequence Number IV Generator driver.
Install [id0t4yrc] CVE-2025-71097: Reference count leak in TCP/IP networking driver.
Install [jz3bgg48] CVE-2025-71084: Reference count leak in InfiniBand driver.
Install [mktplwnd] CVE-2025-38022: Use-after-free in InfiniBand driver.
Install [2q9l5k7x] CVE-2025-71068: Out-of-bounds memory access in RPC-over-RDMA transport driver.
Install [g6a9bz2q] CVE-2025-71120: Null pointer dereference in SunRPC GSS.
Install [86zo4w8h] CVE-2025-68803: Access control violation in NFS server driver.
Install [kae6ypxh] CVE-2025-38129: Use-after-free in Networking driver.
Install [dsv9zfll] CVE-2024-46830: Memory corruption in Kernel-based Virtual Machine (KVM) driver.
Install [4n99njmf] CVE-2024-36903: Information leak in IPv6 networking support.
Install [dj33fek5] CVE-2026-22998: Null pointer dereference in NVME subsystem.
Install [k6avmbn4] CVE-2026-23074: Use-after-free in TEQL network scheduler.
Install [t81xh89q] CVE-2025-68764: Insufficient privilege checks in NFS client driver.
Install [gewbr09k] CVE-2026-22988, CVE-2025-71098: Kernel panic in IPv6 GRE tunnel driver.
Install [7hm82298] CVE-2022-49465: Use-after-free in Block layer bio throttling driver.
Install [kz7fb0p7] CVE-2026-23001: Use-after-free in MAC-VLAN driver.
Install [4ndlu6l2] CVE-2026-23060: Null pointer dereference in Authenc driver.
Install [jhpwhouq] CVE-2026-22976: Null pointer dereference in QFQ network scheduler.
Install [gicuj166] CVE-2025-71182: Denial-of-service in SAE J1939 driver.
Install [q90ed9uh] CVE-2025-40110: Null pointer dereference in DRM driver for VMware virtual GPUs.
Install [fihi4pp8] CVE-2024-36927: Use of uninitialized memory in TCP/IP networking driver.
Install [oo19cbu5] CVE-2026-23120: Data race in Layer Two Tunneling Protocol (L2TP) driver.
Install [i7uyijjk] CVE-2026-23105: Undefined behavior in QFQ network scheduler.
Install [6ujas1b3] CVE-2023-53520: Kernel crash in Bluetooth subsystem.
Install [qkduksn0] CVE-2026-23099: Out-of-bounds memory access in Bonding driver.
Install [3e6v26ph] CVE-2026-22977: Kernel panic in Networking driver.
Install [7p3npkzk] CVE-2025-71194: Deadlock in Btrfs filesystem driver.
Install [avkzvsdh] CVE-2026-23097: Deadlock in Page migration driver.
Install [fk4hzrxx] CVE-2026-23011: Kernel panic in IP: GRE tunnels over IP driver.
Install [83qy1cn4] CVE-2026-23111: Use-after-free in Netfilter driver.
Install [hgz2po8t] CVE-2026-23209: Use-after-free in MAC-VLAN driver.
Install [oey64tu5] Known exploit detection for CVE-2026-23209.
Removing [lx0gzdnf] CVE-2025-38457: Null pointer dereference in QoS and/or fair queueing driver.
Removing [iw8wjvh1] CVE-2025-38477: Use-after-free in Quick Fair Queueing scheduler (QFQ) driver.
Removing [gqkk45sd] CVE-2025-38107: Integer overflow in Enhanced transmission selection scheduler (ETS).
Removing [387haf5e] CVE-2025-38193: Integer overflow in Stochastic Fairness Queueing (SFQ) driver.
Removing [cdo10nt9] CVE-2025-38083, CVE-2025-38108: Integer underflow in multiple network schedulers.
Removing [7izd5nbg] CVE-2025-37890, CVE-2025-38000, CVE-2025-38001, CVE-2025-38350: Use-after-free in HFSC network scheduler.
Removing [11p6pga8] CVE-2025-37798, CVE-2025-38177, CVE-2025-37932, CVE-2025-37953: Use-after-free in multiple network schedulers.
Installing [aostn7o7] Protective update for Null pointer dereference in RDS Protocol driver.
Installing [2p8a0k0d] Improvement update for Null pointer dereference in RDS Protocol driver.
Installing [jvz2patb] CVE-2025-37798, CVE-2025-38177, CVE-2025-37932, CVE-2025-37953: Use-after-free in multiple network schedulers.
Installing [8sqc7c0k] CVE-2025-37890, CVE-2025-38000, CVE-2025-38001, CVE-2025-38350: Use-after-free in HFSC network scheduler.
Installing [ly6x1hsg] CVE-2025-38083, CVE-2025-38108: Integer underflow in multiple network schedulers.
Installing [igyv8377] CVE-2025-38193: Integer overflow in Stochastic Fairness Queueing (SFQ) driver.
Installing [rnpo6r7o] CVE-2025-38107: Integer overflow in Enhanced transmission selection scheduler (ETS).
Installing [iw8wjvh1] CVE-2025-38477: Use-after-free in Quick Fair Queueing scheduler (QFQ) driver.
Installing [pc74lj2k] CVE-2025-38457: Null pointer dereference in QoS and/or fair queueing driver.
Installing [dnrtq922] CVE-2025-38468: Kernel oops in Hierarchical Token Bucket network scheduler.
Installing [dqagdwkf] CVE-2025-38684: Null pointer dereference in Packet Scheduler subsystem.
Installing [d6kl1l8h] CVE-2025-38024: Use-after-free in Software RDMA over Ethernet (RoCE) driver.
Installing [kugcl16z] CVE-2025-38622: Kernel oops in UDP networking stack.
Installing [f218n88x] CVE-2025-40019: Missing check in block encryption ESSIV support.
Installing [tdluc30k] CVE-2025-39885: Deadlock in OCFS2 filesystem driver.
Installing [6wokg8qv] CVE-2025-39955, CVE-2025-40186: Dropped packets in TCP/IP networking driver.
Installing [d9c9apzb] CVE-2025-40027: Use-after-free in 9P protocol stack.
Installing [nua1eqsl] CVE-2025-40018: Use-after-free in IP virtual server (FTP) driver.
Installing [f10dn542] CVE-2025-40115: Use-after-free in LSI Fusion-MPT SAS driver.
Installing [dgxqpxvg] CVE-2025-40055: Use-after-free in OCFS2 filesystem.
Installing [ogor6blh] CVE-2025-40048: Hang in Generic Hyper-V VMBus userspace I/O driver.
Installing [i6gtoxbd] CVE-2025-40044: Out-of-bounds memory access in UDF filesystem.
Installing [m0lj1tzl] CVE-2025-40153: Soft lockup in HugeTLB filesystem.
Installing [c6fyocml] CVE-2025-40111: Use-after-free in VMware vGPU driver.
Installing [e1f00opd] CVE-2025-40187: Null pointer dereference in SCTP Protocol driver.
Installing [kd50bj1e] CVE-2025-40205: Out-of-bounds memory access in BTRFS filesystem.
Installing [8ai5p4ry] CVE-2025-40194: Kernel crash in Intel P state control driver.
Installing [5ckw5kra] CVE-2025-40204: Variable-time MAC check in SCTP Protocol driver.
Installing [gbwhmb1k] CVE-2025-40134: Null pointer dereference in multi-device driver (RAID/LVM).
Installing [hwzbva2o] CVE-2025-40178: Null pointer dereference in core kernel.
Installing [6kzoiuc0] CVE-2025-40173: Use-after-free in IPv6 networking stack.
Installing [hmql959q] CVE-2025-40105: Memory leak in core filesystem layer.
Installing [h6ubsv5w] CVE-2025-40026: Race in x86 KVM.
Installing [k9l2nqad] Logic error in Transport Layer Security driver.
Installing [j8sn6w7i] CVE-2025-40087: Remote null pointer dereference in Parallel NFS FlexFile server driver.
Installing [h52c77mt] CVE-2025-40240: Remote null pointer dereference in SCTP Protocol driver.
Installing [kar08ekj] CVE-2025-38678: Kernel oops in Netfilter driver.
Installing [qtz7bm22] CVE-2025-40233: Kernel crash in OCFS2 filesystem driver.
Installing [nqh664s7] CVE-2025-40231: Deadlock in Virtual Socket protocol driver.
Installing [p7vygkv1] CVE-2025-22056: Out-of-bounds memory access in Netfilter tunnel driver.
Installing [i7n5xk5p] CVE-2024-50257: Use-after-free in Netfilter Xtables driver.
Installing [fu6dwsct] CVE-2025-40248: Use-after-free in Virtual Socket protocol driver.
Installing [d8vo4m2h] CVE-2025-40271: Use-after-free in /proc filesystem driver.
Installing [5g7smk83] CVE-2025-40280: Use-after-free in TIPC Protocol driver.
Installing [pyek1z1n] Known exploit detection for CVE-2023-4015.
Installing [4f9bsw52] Known exploit detection for CVE-2023-5197.
Installing [2lggz22s] Performance improvement for kallsyms lookup.
Installing [roubjmj4] CVE-2025-40258: Use-after-free in MPTCP: Multipath TCP driver.
Installing [a09xhhyp] CVE-2025-68209: Null pointer dereference in Mellanox devices driver.
Installing [h1y3arqq] CVE-2025-40256, CVE-2025-40215: Race condition in XFRM subsystem.
Installing [md0rp234] CVE-2025-38571: Use-after-free in NFS over TLS.
Installing [k2js8ari] CVE-2025-38566: Remote denial-of-service in SUNRPC.
Installing [smeektn8] CVE-2025-40083: Null pointer dereference in Quick Fair Queueing scheduler (QFQ) driver.
Installing [itlqzz4z] CVE-2025-40281: Out-of-bounds memory access in SCTP Protocol driver.
Installing [fcc5gg0s] CVE-2025-40277: Out-of-bounds memory access in VMware graphics driver.
Installing [pj9mhss8] CVE-2025-40273: Kernel oops in NFS server for NFS version 4 driver.
Installing [5xelf337] CVE-2025-68301: Out-of-bounds memory access in aQuantia AQtion driver.
Installing [i9jxp7n1] CVE-2023-24023: Authentication bypass when pairing Bluetooth devices.
Installing [afhh5976] CVE-2025-40040: Kernel panic in KSM for page merging driver.
Installing [a6sme3hu] CVE-2024-37354: Denial-of-service in Btrfs filesystem driver.
Installing [g5zqvsnh] Performance regression in TCP/IP networking driver.
Installing [kkpy2t0b] CVE-2025-40264: Null pointer dereference in BladeEngine driver.
Installing [k0mbwxcn] CVE-2025-40272: Use-after-free in secret memory management.
Installing [ibpiufuf] CVE-2025-68295: Memory leak in SMB/CIFS client driver.
Installing [f3abo2gy] CVE-2024-47666: Kernel crash in PMC-Sierra SPC 8001 SAS/SATA Based Host Adapter driver.
Installing [16mqfnr3] CVE-2025-40257: Use-after-free in MPTCP: Multipath TCP driver.
Installing [r7uqufh0] CVE-2024-43888: Use-after-free in LRU list implementation.
Installing [r9z9ei4b] CVE-2025-40149: Use-after-free in Transport Layer Security HW offload driver.
Installing [dsc70x0h] Use-after-free in x86 KVM.
Installing [c1ej2xno] CVE-2025-71066: Use-after-free in ETS network scheduler.
Installing [m8f04wlo] CVE-2025-68813: Null pointer dereference in IP virtual server driver.
Installing [2mzhguj3] CVE-2025-68776: Null pointer dereference in High-availability Seamless Redundancy (HSR & PRP) driver.
Installing [3nb1rndm] CVE-2025-71147: Memory leak in TPM-based trusted keys driver.
Installing [19b8c1q1] CVE-2025-68818: Null pointer dereference in QLogic QLA2XXX Fibre Channel driver.
Installing [kt172cv8] CVE-2025-71104: Hard lockup in KVM.
Installing [8zwirihy] CVE-2025-68788: Information leak in fsnotify.
Installing [eh3y43j5] CVE-2025-71131: Use-after-free in Sequence Number IV Generator driver.
Installing [id0t4yrc] CVE-2025-71097: Reference count leak in TCP/IP networking driver.
Installing [jz3bgg48] CVE-2025-71084: Reference count leak in InfiniBand driver.
Installing [mktplwnd] CVE-2025-38022: Use-after-free in InfiniBand driver.
Installing [2q9l5k7x] CVE-2025-71068: Out-of-bounds memory access in RPC-over-RDMA transport driver.
Installing [g6a9bz2q] CVE-2025-71120: Null pointer dereference in SunRPC GSS.
Installing [86zo4w8h] CVE-2025-68803: Access control violation in NFS server driver.
Installing [kae6ypxh] CVE-2025-38129: Use-after-free in Networking driver.
Installing [dsv9zfll] CVE-2024-46830: Memory corruption in Kernel-based Virtual Machine (KVM) driver.
Installing [4n99njmf] CVE-2024-36903: Information leak in IPv6 networking support.
Installing [dj33fek5] CVE-2026-22998: Null pointer dereference in NVME subsystem.
Installing [k6avmbn4] CVE-2026-23074: Use-after-free in TEQL network scheduler.
Installing [t81xh89q] CVE-2025-68764: Insufficient privilege checks in NFS client driver.
Installing [gewbr09k] CVE-2026-22988, CVE-2025-71098: Kernel panic in IPv6 GRE tunnel driver.
Installing [7hm82298] CVE-2022-49465: Use-after-free in Block layer bio throttling driver.
Installing [kz7fb0p7] CVE-2026-23001: Use-after-free in MAC-VLAN driver.
Installing [4ndlu6l2] CVE-2026-23060: Null pointer dereference in Authenc driver.
Installing [jhpwhouq] CVE-2026-22976: Null pointer dereference in QFQ network scheduler.
Installing [gicuj166] CVE-2025-71182: Denial-of-service in SAE J1939 driver.
Installing [q90ed9uh] CVE-2025-40110: Null pointer dereference in DRM driver for VMware virtual GPUs.
Installing [fihi4pp8] CVE-2024-36927: Use of uninitialized memory in TCP/IP networking driver.
Installing [oo19cbu5] CVE-2026-23120: Data race in Layer Two Tunneling Protocol (L2TP) driver.
Installing [i7uyijjk] CVE-2026-23105: Undefined behavior in QFQ network scheduler.
Installing [6ujas1b3] CVE-2023-53520: Kernel crash in Bluetooth subsystem.
Installing [qkduksn0] CVE-2026-23099: Out-of-bounds memory access in Bonding driver.
Installing [3e6v26ph] CVE-2026-22977: Kernel panic in Networking driver.
Installing [7p3npkzk] CVE-2025-71194: Deadlock in Btrfs filesystem driver.
Installing [avkzvsdh] CVE-2026-23097: Deadlock in Page migration driver.
Installing [fk4hzrxx] CVE-2026-23011: Kernel panic in IP: GRE tunnels over IP driver.
Installing [83qy1cn4] CVE-2026-23111: Use-after-free in Netfilter driver.
Installing [hgz2po8t] CVE-2026-23209: Use-after-free in MAC-VLAN driver.
Installing [oey64tu5] Known exploit detection for CVE-2026-23209.
Your kernel is fully up to date.
Effective kernel version is 5.15.0-318.199.3.2.1.el9uek
```

Quarto Passo: Confirmar a instalação e validar os CVEs aplicados.

```bash
Antes: 

[root@srvora02 ~]# uptrack-uname -r
5.15.0-302.167.6.1.el9uek.x86_64

[root@srvora02 ~]# uptrack-show | wc -l 
330

Agora:

[root@srvora02 ~]# uptrack-uname -r
5.15.0-318.199.3.2.1.el9uek.x86_64

[root@srvora02 ~]# uptrack-show | wc -l 
431
```

  
É isso pessoal! Espero que este artigo ajude você a atualizar seu Ksplice. Qualquer coisa, só chamar no [**linkedin**](https://www.linkedin.com/in/diogo-fernandess/) 🙂