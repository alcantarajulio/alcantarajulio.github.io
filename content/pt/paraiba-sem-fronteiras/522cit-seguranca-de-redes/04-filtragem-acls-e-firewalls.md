---
title: "Filtragem de tráfego: ACLs e firewalls"
description: "Da lista de controle sem estado ao firewall com tabela de sessão e às políticas por zona no IOS."
date: 2026-08-19
chapter: 4
translationKey: "psf-522-04"
weight: 40
tags: ["acl", "firewall", "zpf", "dmz"]
---

Até aqui o alvo foi o equipamento. Agora o alvo é o tráfego que passa por ele, e a
pergunta muda: em vez de quem entra no roteador, quem atravessa a rede e para onde.

## A lista de acesso e suas quatro regras

Uma ACL é uma sequência ordenada de instruções de permissão e negação que o roteador
avalia contra cada pacote. Quatro comportamentos explicam quase todo erro de laboratório.

A avaliação acontece de cima para baixo e para na primeira linha que casa. Ordem errada
produz regra inalcançável, e o caso clássico é negar um host depois de já ter permitido
a sub-rede inteira.

Toda ACL termina com uma negação implícita que você não vê no `show`. Lista com apenas
uma linha de negação bloqueia tudo, porque não há nada permitindo o resto.

O casamento usa máscara curinga, que é o inverso da máscara de rede. Onde o bit vale
zero, o pacote precisa bater. Onde vale um, o roteador ignora.

E o sentido importa. A mesma lista aplicada na entrada ou na saída de uma interface
produz efeitos diferentes, porque a decisão acontece antes ou depois do roteamento.

Sobre onde colocar, a regra tradicional diz que a ACL padrão vai perto do destino,
porque ela só olha o endereço de origem e filtrar cedo derrubaria tráfego legítimo para
outros destinos. A ACL estendida vai perto da origem, porque ela distingue destino e
porta, e descartar cedo poupa a largura de banda de todo o caminho.

## O problema do retorno

Uma ACL não guarda memória. Cada pacote é avaliado como se fosse o primeiro, o que cria
um problema assim que você tenta permitir que a rede interna acesse a internet: a
requisição sai, e a resposta chega em uma porta alta arbitrária que a sua lista não
previu.

A primeira solução foi permitir o retorno olhando os bits de controle do TCP. Um pacote
que pertence a uma conversa já iniciada carrega marcadores que o primeiro pacote não
carrega, e a ACL passa a exigir esses marcadores. Funciona, e é frágil: quem forja os
bits atravessa, e o UDP não tem estado nenhum para inspecionar.

A resposta melhor foi guardar estado.

## Firewall com estado

Um firewall com estado mantém uma tabela de conexões ativas. Quando o pacote sai, ele
registra origem, destino, portas e o estado da conversa. Quando a resposta chega, ele
compara com a tabela: pertence a uma sessão que ele mesmo viu nascer, então passa. Não
pertence, então cai.

A diferença prática é que você escreve a regra em um sentido só, e o retorno acontece
sem que você o descreva. A diferença conceitual é que o firewall passa a entender
conversa, e não pacote isolado.

Vale situar as famílias, porque a prova cobra. O filtro de pacotes olha cabeçalho e nada
mais. O firewall com estado acrescenta a tabela de sessão. O gateway de aplicação, ou
proxy, termina a conexão de um lado e abre outra do outro, o que permite inspecionar o
conteúdo ao custo de desempenho e de transparência. E o NAT não é firewall, embora
esconda a topologia interna: ele traduz endereço, e a proteção que ele oferece é efeito
colateral, não projeto.

## Zonas em vez de interfaces

O modelo antigo do IOS aplicava inspeção interface a interface. Ele funciona, e envelhece
mal: cada interface nova exige revisar as regras de todas as outras.

O modelo por zonas inverte a lógica. Você declara zonas, coloca cada interface em uma
zona, e escreve a política entre pares de zonas. Quatro comportamentos definem o modelo.

Tráfego entre duas interfaces da mesma zona passa sem política. Tráfego entre zonas
diferentes é negado até existir um par de zonas com política que o permita. A política é
unidirecional, então permitir de dentro para fora não permite de fora para dentro. E
tráfego entre uma interface em zona e outra fora de qualquer zona é descartado, o que
produz o sintoma mais comum do laboratório: você configura tudo, esquece de colocar uma
interface na zona, e a rede para.

A zona própria do roteador merece nota. O tráfego destinado ao próprio equipamento, como
a sua sessão SSH, pertence a uma zona especial que por padrão permite tudo. No momento
em que você escreve uma política envolvendo essa zona, o padrão deixa de valer, e é
possível cortar a própria administração.

## A DMZ e por que ela existe

Um servidor que precisa ser alcançado da internet não pode morar na rede interna, porque
comprometê-lo daria ao atacante um pé dentro. Ele também não pode morar fora, porque
você perde o controle sobre ele.

A DMZ é a terceira zona: alcançável de fora com regras estreitas, alcançável de dentro,
e proibida de iniciar conexão para dentro. Essa última regra é a que importa. Se o
servidor web cair, o atacante que estiver nele não consegue abrir sessão contra o banco
de dados interno, porque a política nega o sentido.

## Prática

Três interfaces no R1: `g0/0` para a rede interna, `g0/1` para a internet, `g0/2` para a
DMZ com um servidor web em 192.168.100.10.

### ACL estendida nomeada

```text
R1(config)# ip access-list extended BORDA-IN
R1(config-ext-nacl)# permit tcp any host 192.168.100.10 eq 80
R1(config-ext-nacl)# permit tcp any host 192.168.100.10 eq 443
R1(config-ext-nacl)# deny ip 10.0.0.0 0.255.255.255 any
R1(config-ext-nacl)# deny ip 127.0.0.0 0.255.255.255 any
R1(config-ext-nacl)# permit icmp any any echo-reply
R1(config-ext-nacl)# exit
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# ip access-group BORDA-IN in
```

As duas linhas de negação no meio descartam endereço privado e de laço chegando de fora,
que só existe em pacote forjado. Filtrar isso na borda é higiene básica.

### Retorno sem estado

```text
R1(config)# ip access-list extended RETORNO
R1(config-ext-nacl)# permit tcp any 10.10.0.0 0.0.255.255 established
```

Mostre isso no laboratório e depois substitua, porque o próximo bloco resolve melhor.

### Firewall por zonas

Declare as zonas e o que inspecionar:

```text
R1(config)# zone security INTERNA
R1(config)# zone security EXTERNA
R1(config)# zone security DMZ
R1(config)# class-map type inspect match-any CM-SAIDA
R1(config-cmap)# match protocol tcp
R1(config-cmap)# match protocol udp
R1(config-cmap)# match protocol icmp
R1(config-cmap)# exit
R1(config)# policy-map type inspect PM-INTERNA-EXTERNA
R1(config-pmap)# class type inspect CM-SAIDA
R1(config-pmap-c)# inspect
R1(config-pmap-c)# exit
R1(config-pmap)# class class-default
R1(config-pmap-c)# drop log
```

Amarre as interfaces e o par de zonas:

```text
R1(config)# interface gigabitEthernet 0/0
R1(config-if)# zone-member security INTERNA
R1(config-if)# exit
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# zone-member security EXTERNA
R1(config-if)# exit
R1(config)# interface gigabitEthernet 0/2
R1(config-if)# zone-member security DMZ
R1(config-if)# exit
R1(config)# zone-pair security ZP-INT-EXT source INTERNA destination EXTERNA
R1(config-sec-zone-pair)# service-policy type inspect PM-INTERNA-EXTERNA
```

Para publicar o servidor da DMZ, um par de zonas próprio com política estreita:

```text
R1(config)# class-map type inspect match-any CM-WEB
R1(config-cmap)# match protocol http
R1(config-cmap)# match protocol https
R1(config-cmap)# exit
R1(config)# policy-map type inspect PM-EXT-DMZ
R1(config-pmap)# class type inspect CM-WEB
R1(config-pmap-c)# inspect
R1(config-pmap-c)# exit
R1(config)# zone-pair security ZP-EXT-DMZ source EXTERNA destination DMZ
R1(config-sec-zone-pair)# service-policy type inspect PM-EXT-DMZ
```

Repare no que não foi escrito: nenhum par de zonas com origem DMZ e destino INTERNA.
Essa ausência é a regra de segurança, já que o modelo nega por padrão.

### Conferir

```text
R1# show access-lists
R1# show ip access-lists BORDA-IN
R1# show zone security
R1# show zone-pair security
R1# show policy-map type inspect zone-pair sessions
```

O último comando mostra a tabela de sessões viva. Gere tráfego de dentro para fora e
observe as entradas aparecerem e expirarem, que é a demonstração mais direta da
diferença entre filtro sem estado e firewall com estado.

## Laboratórios do curso

O assunto aparece nos laboratórios 10.3.11 e 10.3.12 do módulo 10, e no 4.4.1.2 do
material antigo de CCNA Security.

O próximo capítulo trata do que fazer com o tráfego que passou pela filtragem e ainda
assim é malicioso.
