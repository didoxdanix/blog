---
title: "Hot Backup de Máquinas Virtuais no ODA com KVM"
datePublished: Fri Sep 12 2025 13:00:53 GMT+0000 (Coordinated Universal Time)
cuid: cmfguigoo000302l5d7lr6pfz
slug: hot-backup-de-maquinas-virtuais-no-oda-com-kvm
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1757610058461/ece2667a-0a4a-4337-a6b3-4ec0abd588fb.jpeg
tags: oracle, kvm

---

(Safe Harbor) Antes de iniciarmos, informo que este procedimento **não** está documentado em nenhuma nota oficial da Oracle (MOS), embora eu mencione algumas referências ao longo do texto. Cheguei a este procedimento após conversar com vários profissionais de KVM e após extensa leitura em fóruns; ele foi testado diversas vezes antes da publicação deste artigo. Ainda assim, recomendo cautela: valide primeiro em ambiente de testes/homologação e só então aplique no ambiente de produção, após **testes exaustivos**. Já que o Safe Harbor foi pronunciado, bora colocar a mão na massa!!!

Hoje trago para vocês uma forma diferente de realizar backups no Oracle Database Appliance (ODA). Este artigo será dividido em duas partes: uma destinada às VMs Linux e outra às VMs Windows.

Normalmente, as notas de backup sugerem duas abordagens:

1. Parar a máquina e copiar os discos.
    
2. Congelar a maquina, criar um snapshot do volume ACFS e copiar os arquivos para outro disco, conforme descrito na nota **2779329.1**.
    

A primeira opção é, sem dúvida, a mais rápida e segura. Sempre que possível, escolha essa abordagem. Porém, na prática, raramente temos a chance de parar a máquina, e quase sempre precisamos realizar o backup das VMs em modo hot (a quente).

De acordo com a nota **2779329.1**, para esse cenário é necessário criar snapshots do ACFS, e mesmo assim os discos são colocados em modo “Freeze". E aí surgem dois problemas:

• Discos em locais diferentes: no ODA, caso exista um segundo disco anexado à VM, ele é armazenado em uma área distinta do disco de boot. Isso significa que será necessário criar dois snapshots em momentos diferentes, gerando inconsistências de data e hora, o que pode se tornar uma verdadeira dor de cabeça.

• Gestão dos snapshots: após o backup, é preciso excluir os snapshots do ACFS. Essa atividade, além de repetitiva, aumenta bastante a carga de trabalho administrativo.

E, ao realizar os backups pelas formas mencionadas acima, em determinado momento me surgiu a seguinte pergunta:

## Existe alguma forma de os utilitários do KVM realizarem o backup em vez de depender apenas de procedimentos via storage?

Uma vez que a pergunta ficou ‘instalada’ na minha cabeça, comecei a conversar com alguns mestres em KVM e encontrei a seguinte solução.

```bash
virsh snapshot-create-as --domain linux_vm --name FILE_STAGE_BACKUP.TEMP --disk-only --atomic --quiesce
```

Mas calma, não vá executando o comando logo de cara (rs). Ainda temos um longo caminho de explicação e é preciso cautela quanto ao comando que será utilizado.

Agora vamos começar a analisar as estruturas da VM no KVM/ODA. Os comandos apresentados aqui podem ser executados tanto em um ODA quanto em um ambiente de KVM Server.

O primeiro passo é compreender quantos discos a VM possui. Para verificar isso, vamos executar o seguinte comando:

```bash
root@kvm-001:/kvm# virsh domblklist linux_vm
 Target   Source
---------------------------------
 vda      /kvm/linux_boot
 vdb      /kvm/disco_secundario
```

Como podemos ver, temos 2 discos: o disco de boot e o disco *secundário*. Neste caso, o ambiente que estou utilizando é um KVM Server, mas a mesma regra se aplica ao ODA.

A outra verificação necessária é confirmar se o *KVM guest agent* está instalado na máquina virtual. Para isso, utilizamos o seguinte comando:

```bash
[root@guestvm ~]# rpm -qa | grep guest
qemu-guest-agent-9.0.0-10.el9_5.x86_64
```

Agora que já verificamos as informações sobre a quantidade de discos e a presença do *guest agent*, vamos de fato falar sobre o hot-backup.

Chegou o momento de executar o comando para iniciarmos o *backup* a quente. Esse comando, na prática, faz com que o próprio KVM crie o *snapshot* dos discos — e não o ACFS. Dessa forma, a máquina continuará online e todas as operações passarão a ser gravadas em um novo arquivo.

```bash
root@kvm-001:/kvm# virsh snapshot-create-as --domain linux_vm --name FILE_STAGE_BACKUP.TEMP --disk-only --atomic --quiesce
Domain snapshot FILE_STAGE_BACKUP.TEMP created
```

Como dito anteriormente, esse comando colocará todos os discos em modo *snapshot*, criando novos apontamentos para eles. Assim, o disco boot\_linux será transformado em boot\_linux.FILE\_STAGE\_BACKUP.TEMP e o disco\_secundario em disco\_secundario.FILE\_STAGE\_BACKUP.TEMP, como demonstrado na saída abaixo:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757542246305/cf346e85-3604-404e-99b5-6dfb6cc0a197.png align="center")

```bash
root@kvm-001:/kvm# virsh snapshot-create-as --domain linux_vm --name FILE_STAGE_BACKUP.TEMP --disk-only --atomic --quiesce
Domain snapshot FILE_STAGE_BACKUP.TEMP created
root@kvm-001:/kvm# ls -lrt
total 6488960
drwxr-xr-x 2 root         root       4096 Sep 10 13:54 outras_maquinas
-rw------- 1 libvirt-qemu kvm    35651584 Sep 10 19:07 disco_secundario
-rw------- 1 libvirt-qemu kvm  6605111296 Sep 10 19:07 linux_boot
-rw------- 1 libvirt-qemu kvm     1835008 Sep 10 19:10 linux_boot.FILE_STAGE_BACKUP.TEMP
-rw------- 1 libvirt-qemu kvm     2031616 Sep 10 19:10 disco_secundario.FILE_STAGE_BACKUP.TEMP
```

Dessa forma, as escritas passam a ser direcionadas para um novo ponto, encerrando as operações no linux\_boot e iniciando em linux\_boot.FILE\_STAGE\_BACKUP.TEMP e disco\_secundario.FILE\_STAGE\_BACKUP.TEMP.

A partir desse momento, podemos copiar o disco para o diretório de nossa preferência.

```bash
root@kvm-001:/kvm# scp linux_boot disco_secundario root@serverbackup:/backup
```

Agora vamos resumir o que aconteceu:

1. Primeiro, criamos um *snapshot* online da VM, forçando-a a escrever tudo em novos arquivos, com o sufixo .FILE\_STAGE\_BACKUP.TEMP.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757542087989/091cd73a-af77-4ee6-a0b1-a707b40414ec.png align="center")

2. Ao executar o comando de *backup*, os arquivos da VM (linux\_boot e disco\_secundario) ficaram em um ponto consistente, permitindo que fossem copiados de forma segura para outro local.
    
3. Por fim, copiamos os arquivos para outro servidor.
    

Pronto! Até aqui realizamos o *backup*, mas agora precisamos executar alguns passos para que as escritas voltem a ser direcionadas inteiramente para o disco linux\_boot, e não mais para o linux\_boot.FILE\_STAGE\_BACKUP.TEMP.

Como criamos o snap, vamos agora verificar e deletar apenas os metadados do mesmo.

```bash
root@kvm-001:/kvm# virsh snapshot-list linux_vm
 Name                     Creation Time               State
---------------------------------------------------------------------
 FILE_STAGE_BACKUP.TEMP   2025-09-10 19:07:30 -0300   disk-snapshot
```

Caminho atual de onde o VM estava escrevendo:

```bash
root@kvm-001:/kvm# virsh domblklist linux_vm
 Target   Source
--------------------------------------------------------
 vda      /kvm/linux_boot.FILE_STAGE_BACKUP.TEMP
 vdb      /kvm/disco_secundario.FILE_STAGE_BACKUP.TEMP
```

Agora chegamos a um ponto importante: neste momento vamos deletar apenas os METADADOS do *snapshot*, de forma que os arquivos ainda permanecerão no local.(sempre verifique e o sintaxe --metadata esta no final do comando)

```bash
root@kvm-001:/kvm# virsh snapshot-delete --domain linux_vm FILE_STAGE_BACKUP.TEMP --metadata
Domain snapshot FILE_STAGE_BACKUP.TEMP deleted
```

Podemos ver agora que não existe mais snapshot:

```bash
root@kvm-001:/kvm# virsh snapshot-list linux_vm
 Name   Creation Time   State
-------------------------------
```

Podemos observar que a máquina ainda continua apontando para os discos do *snapshot*, mesmo após a exclusão:

```bash
root@kvm-001:/kvm# virsh domblklist linux_vm
 Target   Source
--------------------------------------------------------
 vda      /kvm/linux_boot.FILE_STAGE_BACKUP.TEMP
 vdb      /kvm/disco_secundario.FILE_STAGE_BACKUP.TEMP
```

Nesse momento a VM esta nessa situação:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757599891798/31e1c379-bba0-4737-b7de-33941a9bb72a.png align="center")

Agora temos um impasse: o *metadata* do *snapshot* foi deletado, mas a máquina ainda está apontando para o arquivo gerado. Como fazer para voltar ao disco original sem perder as transações que ocorreram nesse período?

Para resolver isso, precisamos realizar um *merge* dos discos. No nosso caso, a VM possui dois discos: vda e vdb.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757600020196/06bc7873-d1c0-40b0-93c6-0db4570ce6b9.png align="center")

Nesse momento, precisamos realizar o *merge* do disco /kvm/linux\_boot.FILE\_STAGE\_BACKUP.TEMP para o disco /kvm/linux\_boot, e do disco /kvm/disco\_secundario.FILE\_STAGE\_BACKUP.TEMP para /kvm/disco\_secundario.

O *merge* deve ser feito disco por disco. No nosso caso, a VM possui 2 discos, mas se a sua VM tiver 5 discos, será necessário executar o *merge* em cada um deles. Como aqui são apenas 2 discos, vamos ao comando:

Primeiro do VDA:

```bash
root@kvm-001:/kvm# virsh blockcommit linux_vm vda --active --verbose --pivot
Block commit: [100.00 %]
Successfully pivoted
```

Depois do VDB:

```bash
root@kvm-001:/kvm# virsh blockcommit linux_vm vdb --active --verbose --pivot
Block commit: [100.00 %]
Successfully pivoted
```

Pronto, agora vamos ver onde o KVM esta escrevendo as operações das VMs:

```bash
root@kvm-001:/kvm# virsh domblklist linux_vm
 Target   Source
---------------------------------
 vda      /kvm/linux_boot
 vdb      /kvm/disco_secundario
```

Pronto, ao executar os comando de merge, ele sai dessa situação:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757600392395/7aa4747e-2c3f-4844-a16f-9377902cb808.png align="center")

E retorna para essa situação:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757600538212/f57e720d-21d2-43dd-b847-e20da5a65835.png align="center")

Podemos ver agora, que as escritas de fatos voltaram para os arquivos “originais”:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757600625046/0e40a59d-421f-41b6-a8ec-7ae453b591e5.png align="center")

Mas, como podemos observar, os arquivos do *snapshot* ainda continuam no diretório. Nesse caso, por segurança, recomendo movê-los para um diretório temporário e, após algum tempo de validação, deletá-los.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757600755847/423d4cc6-0bb2-4f2f-90df-32ba31580467.png align="center")

```bash
root@kvm-001:/kvm# ls -lrt
total 7177096
drwxr-xr-x 2 root         root       4096 Sep 10 13:54 outras_maquinas
-rw------- 1 libvirt-qemu kvm   575406080 Sep 11 11:16 linux_boot.FILE_STAGE_BACKUP.TEMP
-rw------- 1 libvirt-qemu kvm    68747264 Sep 11 11:16 disco_secundario.FILE_STAGE_BACKUP.TEMP
-rw------- 1 libvirt-qemu kvm  6631194624 Sep 11 11:22 linux_boot
-rw------- 1 libvirt-qemu kvm    73924608 Sep 11 11:22 disco_secundario
root@kvm-001:/kvm# mkdir retencao
root@kvm-001:/kvm# mv linux_boot.FILE_STAGE_BACKUP.TEMP retencao/
root@kvm-001:/kvm# mv disco_secundario.FILE_STAGE_BACKUP.TEMP retencao/
root@kvm-001:/kvm# ls -lrt
total 6548156
drwxr-xr-x 2 root         root       4096 Sep 10 13:54 outras_maquinas
drwxr-xr-x 2 root         root       4096 Sep 11 11:25 retencao
-rw------- 1 libvirt-qemu kvm  6631194624 Sep 11 11:25 linux_boot
-rw------- 1 libvirt-qemu kvm    73924608 Sep 11 11:25 disco_secundario
```

Pronto, pessoal! Neste artigo descrevi os procedimentos para realizar o *backup online* de uma VM Linux. No próximo artigo, irei abordar o *backup online* de máquinas Windows.

Problemas conhecidos:

Existe um bug na estrutura do KVM (em *guest VMs* Linux). Ao criar um *snapshot* em máquinas que possuem mais de um disco, pode ocorrer o seguinte erro:

```bash
root@kvm-001:/kvm# virsh snapshot-create-as --domain linux_vm --name FILE_STAGE_BACKUP.TEMP --disk-only --atomic --quiesce
error: internal error: unable to execute QEMU agent command 'guest-fsfreeze-freeze': failed to open /disco_secundario: Permission denied
```

Esse erro pode ser contornado conforme a nota Doc ID 3087488.1 com seguinte comando (Comando dentro das *guest VMs* Linux):

```bash
[root@guestvm ~]# setsebool -P virt_qemu_ga_read_nonsecurity_files 1
[root@guestvm ~]# sestatus -b |grep qemu
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757602081854/f5a8bd44-1ed8-466b-87c4-25c65884a732.png align="center")

OLVM: internal error: unable to execute QEMU agent command 'guest-fsfreeze-freeze': failed to open /: Permission denied (Doc ID 3087488.1)

Após a implementação da solução descrita na nota, o comando para criar o *snapshot* será concluído com sucesso.

**Observação:** todos esses procedimentos foram testados em ambientes de *guest VMs* com partições dos tipos **XFS** e **EXT4**. Caso a sua *guest VM* utilize outro sistema de arquivos, recomendo realizar testes exaustivos antes de aplicar em produção.

É isso pessoal, qualquer coisa só me chamar no [linkedin](https://www.linkedin.com/in/diogo-fernandess/) 🙂