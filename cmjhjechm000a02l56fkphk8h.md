---
title: "Processo ASR consumindo 600% de CPU no ODA."
datePublished: Mon Dec 22 2025 19:16:21 GMT+0000 (Coordinated Universal Time)
cuid: cmjhjechm000a02l56fkphk8h
slug: processo-asr-consumindo-600-de-cpu-no-oda
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1766430845582/e27e7faa-0c92-44ff-802e-81b73cb79f34.png

---

Esses dias, recebi um alerta no meu celular de um ODA de um cliente informando que o “load” estava mais alto do que o normal e, em seguida, outro alerta de “alta utilização de CPU”.

Meu primeiro pensamento foi: é algo no banco. Porém, ao abrir o oratop, não vi nada além do normal. Então, fui para o Linux e, ao executar o comando top, me deparei com a seguinte situação:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1766404544668/38702c65-7a7e-4fd8-b8ae-603396a1c6d7.png align="center")

Peguei o pid e foi ver de que processo se tratava…

```bash
root      4023     1  0 Aug20 ?        1-04:30:36 /opt/oracle/dcs/java/1.8.0_261/bin/java -Xms512m -Xmx1536m -Dlog4j.configurationFile=/opt/asrmanager/configuration/log4j2.xml -Dasr.log.level=info -Dasr.log.filecount=5 -Dauditlog.days=30 -cp /opt/asrmanager/felix-framework/bin/felix.jar org.apache.felix.main.Main /opt/asrmanager/bundle-cache/ -b /opt/asrmanager/lib/asrstart
```

Ali! ASR Manager…

Para quem não sabe, o ASR Manager abre chamados automaticamente com a Oracle em caso de alguma falha de hardware e também notifica o e-mail do administrador sobre a falha.

Este é um problema antigo no ODA: esse processo causa alta utilização de CPU, mas, na verdade, isso é um efeito colateral de outros problemas. No início, eu achava que era alguma falha da Oracle no momento de configurar o ASR, e até pode ser, mas, em 90% das vezes, isso é apenas um sintoma de outra coisa, e podem ser duas situações.

1 - O seu ODA não está conseguindo sair para os endereços da Oracle para onde as informações são enviadas. O endereço é [transport.oracle.com](http://transport.oracle.com), e existem algumas portas que precisam ser liberadas, conforme o link abaixo recomenda.

[Doc ASR](https://docs.oracle.com/en/engineered-systems/oracle-database-appliance/19.23/daten/configuring-and-using-oracle-auto-service-request-asr1.html?utm_source=chatgpt.com#GUID-A1DA06C1-5EDE-4EDF-A6ED-9C1D749E6E3E)

2 - A outra possibilidade é o cliente estar sem suporte, e o ASR ficar tentando o tempo todo fazer a comunicação com a Oracle, mas, como está sem suporte, o handshake não é concluído com sucesso.

Felizmente, após alguns testes, descobri que o problema do cliente era que haviam bloqueado a saída para o endereço [transport.oracle.com](http://transport.oracle.com) vinda do ODA. Então, nesse caso, eu matei o PID e reiniciei os serviços do ASR após a liberação do endereço/portas no firewall, e tudo deu certo.

```bash
kill -9 4023
cd /opt/asrmanager/bin
./asr restart
ASR Manager is stopped.
ASR Manager (pid 37354) is RUNNING.
```

Mas e se o seu caso for o 2?

O caso 2 é mais comum do que imaginamos, pois estamos falando de um hardware que foi modernizado com NVMe ali por volta de 2016/2017 e que até hoje continua em funcionamento. Em cerca de 80% dos casos, não houve um único aviso de falha de hardware. Com o passar do tempo, e devido à idade do equipamento, os clientes foram renovando seus parques e realocando esses appliances para bases de teste, ambientes de DR, BI para leitura, etc. (Sabemos como funciona na vida real…).

E, se o seu caso for esse, temos uma única opção: deletar a configuração.

```bash
odacli delete-asr
```

Pronto, assim você encerra o processo do ASR e não terá mais aquele problema de ele entrar em loop e tentar se conectar com a Oracle o tempo todo. (Só faça isso se seu oda não tiver mais suporte!!!)

Espero ter ajudado e qualquer coisa só me chamar no [linkedin](https://www.linkedin.com/in/diogo-fernandess/) :)