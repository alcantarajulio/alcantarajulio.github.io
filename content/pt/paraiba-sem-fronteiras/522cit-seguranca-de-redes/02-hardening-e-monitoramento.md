---
title: "Hardening e monitoramento"
description: "Desligar o que ninguém usa, autenticar o roteamento, sincronizar o relógio e fazer o registro chegar a algum lugar."
date: 2026-08-19
chapter: 2
translationKey: "psf-522-02"
weight: 20
tags: ["hardening", "ospf", "ntp", "syslog", "snmp"]
---

O capítulo anterior trancou a porta da frente. Sobra o resto do equipamento: uma dúzia
de serviços que a Cisco liga de fábrica por conveniência, um protocolo de roteamento que
aceita qualquer vizinho que se apresente, um relógio que atrasa alguns segundos por dia e
um registro de eventos que morre junto com o equipamento quando ele reinicia.

## Superfície de ataque é a soma do que está ligado

Cada serviço ativo é uma porta que alguém pode bater, uma implementação que pode ter
falha e uma linha a mais na sua lista de coisas para acompanhar. O roteador de fábrica
responde a coisas que ninguém usa há vinte anos.

O CDP anuncia modelo, versão de IOS, endereço de gerência e nome do equipamento para
qualquer um no segmento. Para o técnico que documenta a rede isso economiza horas. Para
quem faz reconhecimento antes de um ataque, também. O servidor HTTP embutido aceita
autenticação em texto claro. Os "small servers" respondem echo, discard e chargen, que
serviam para diagnóstico em 1983 e hoje servem de amplificador em ataque de negação de
serviço. O roteamento pela origem deixa quem envia o pacote escolher o caminho, o que
contorna a sua topologia de filtragem.

O critério é simples de enunciar e chato de aplicar: desligue tudo, ligue de volta só o
que você usa e consegue justificar.

## O plano de controle acredita em quem se apresenta

Um protocolo de roteamento sem autenticação aceita adjacência com qualquer vizinho que
fale a mesma língua na mesma área. Quem conecta um equipamento no segmento anuncia uma
rota melhor para a sua sub-rede de servidores e passa a receber o tráfego. O nome disso
é sequestro de rota, e o efeito vai de buraco negro a interceptação silenciosa, porque
nada impede o atacante de encaminhar o tráfego adiante depois de olhá-lo.

A autenticação de vizinhança resolve o caso. Os dois lados compartilham uma chave, cada
mensagem carrega um resumo criptográfico calculado com ela, e o roteador descarta o que
não bate. As versões antigas usam MD5, e as atuais aceitam cadeias de chave com SHA, que
ainda permitem rotação sem derrubar a adjacência.

## Sem relógio comum não existe investigação

Registro de evento sem hora confiável é anedota. Quando o firewall marca 14:03, o
roteador marca 13:58 e o servidor marca 14:11, ninguém reconstrói a ordem dos fatos, e
a ordem dos fatos é o que separa a causa da consequência.

O NTP resolve a sincronização e cria um problema novo, porque um atacante que controla a
fonte de tempo reescreve a linha do tempo da sua investigação, faz certificado expirado
parecer válido e desalinha janelas de bloqueio. Por isso o NTP também se autentica: o
cliente só aceita resposta de servidor que prove conhecer a chave.

Duas escolhas acompanham. Registre em UTC ou marque o fuso de forma explícita, porque
correlacionar log de três países com horário local e sem fuso custa uma tarde. E use
carimbo com milissegundos, já que eventos automatizados acontecem dentro do mesmo
segundo.

## Registro que fica no equipamento não é registro

O buffer local some no reinício, e o primeiro movimento de quem invade é reiniciar ou
limpar. Log serve para depois, então precisa sair do equipamento enquanto ainda existe.

O syslog classifica cada mensagem em oito níveis, de 0 para emergência a 7 para
depuração. Mandar tudo no nível 7 para o servidor gera volume que ninguém lê e esconde
o que importa. Mandar só o nível 0 esconde a tentativa de login que precede o incidente.
O ponto de equilíbrio costuma ficar em informativo, nível 6, com depuração ligada apenas
durante uma investigação.

O SNMP merece a mesma atenção. As versões 1 e 2c autenticam por community string, que
viaja em texto claro e funciona como senha compartilhada com permissão de leitura ou de
escrita sobre o equipamento. A versão 3 acrescenta autenticação por usuário e cifra do
conteúdo. Não existe motivo defensável para SNMP v2c em rede nova.

## Recuperar o equipamento depois do estrago

O IOS guarda uma cópia protegida da imagem e da configuração, invisível para o comando
de listagem e resistente ao apagamento por quem tem privilégio total. Isso não impede a
invasão. Encurta o tempo entre o estrago e a volta do serviço, que é uma métrica que o
seu chefe entende.

Existe também um assistente que aplica de uma vez um conjunto grande dessas medidas.
Ele serve bem em laboratório e em equipamento novo. Em produção, leia o que ele mudou
antes de salvar, porque ele desliga serviços que a sua rede talvez use.

## Prática

O cenário é o mesmo do capítulo anterior, agora com dois roteadores rodando OSPF na área
0 e um servidor de syslog e NTP na faixa de gerência.

### Desligar serviços

```text
R1(config)# no cdp run
R1(config)# no ip http server
R1(config)# ip http secure-server
R1(config)# no service pad
R1(config)# no ip bootp server
R1(config)# no ip source-route
R1(config)# no service tcp-small-servers
R1(config)# no service udp-small-servers
R1(config)# no ip domain-lookup
```

Nas interfaces voltadas para fora, desligue o que ajuda o reconhecimento e o
redirecionamento:

```text
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# no ip proxy-arp
R1(config-if)# no ip redirects
R1(config-if)# no ip unreachables
R1(config-if)# no ip mask-reply
```

Se precisar do CDP em algum ponto, desligue por interface em vez de manter global:

```text
R1(config-if)# no cdp enable
```

### Autenticar o OSPF

```text
R1(config)# interface gigabitEthernet 0/0
R1(config-if)# ip ospf message-digest-key 1 md5 Ospf#Redes2026
R1(config-if)# exit
R1(config)# router ospf 1
R1(config-router)# area 0 authentication message-digest
```

Aplique nos dois lados. Chave diferente entre vizinhos derruba a adjacência, e o
sintoma aparece em `show ip ospf neighbor` como vizinho que nunca sai de INIT.

### Sincronizar o tempo com autenticação

```text
R1(config)# ntp authentication-key 1 md5 Ntp#Redes2026
R1(config)# ntp authenticate
R1(config)# ntp trusted-key 1
R1(config)# ntp server 10.20.0.10 key 1
R1(config)# ntp update-calendar
R1(config)# clock timezone BRT -3
```

### Registrar para fora

```text
R1(config)# service timestamps log datetime msec localtime show-timezone
R1(config)# service timestamps debug datetime msec
R1(config)# logging buffered 16384 informational
R1(config)# logging host 10.20.0.10
R1(config)# logging trap informational
R1(config)# logging source-interface loopback 0
```

A interface de origem fixa evita que a mensagem chegue ao servidor com um endereço
diferente a cada caminho, o que atrapalha filtro e correlação.

### SNMP com versão 3

```text
R1(config)# snmp-server view LEITURA iso included
R1(config)# snmp-server group MONITORA v3 priv read LEITURA
R1(config)# snmp-server user nagios MONITORA v3 auth sha Snmp#Auth2026 priv aes 128 Snmp#Priv2026
```

Se houver community string antiga sobrando, remova. Ela continua valendo enquanto
existir.

### Proteger imagem e configuração

```text
R1(config)# secure boot-image
R1(config)# secure boot-config
R1# show secure bootset
```

### Conferir

```text
R1# show ip ospf interface gigabitEthernet 0/0
R1# show ip ospf neighbor
R1# show ntp associations detail
R1# show ntp status
R1# show logging
R1# show snmp user
R1# show secure bootset
R1# show ip interface gigabitEthernet 0/1 | include proxy|redirect|unreachable
```

O `show ntp status` precisa mostrar o relógio sincronizado e o estrato. Servidor não
autenticado aparece em `show ntp associations detail` sem a marca de autenticado, e o
cliente segue usando a hora dele se você esquecer o `ntp trusted-key`.

## Laboratórios do curso

Quem tiver conta no Networking Academy encontra o assunto nos laboratórios 6.2.7,
6.3.6, 6.3.7, 6.6.4, 6.7.11 e 6.7.12 do módulo 6.

O próximo capítulo tira as credenciais de dentro do equipamento e as centraliza em um
servidor, que é o que torna gerenciável uma rede com mais de três roteadores.
