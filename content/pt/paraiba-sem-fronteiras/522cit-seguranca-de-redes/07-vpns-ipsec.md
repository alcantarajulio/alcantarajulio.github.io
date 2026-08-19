---
title: "VPNs IPsec site-to-site"
description: "O que o IPsec é, o que o IKE negocia em cada fase, e como montar um túnel entre dois roteadores pela linha de comando."
date: 2026-08-19
chapter: 7
translationKey: "psf-522-07"
weight: 70
tags: ["vpn", "ipsec", "ike", "isakmp"]
---

Duas unidades da mesma organização, uma em Campina Grande e outra em Coventry, precisam
trocar tráfego interno. Alugar um circuito dedicado entre elas resolve e custa caro. A
VPN usa a internet como transporte e reconstrói em software a privacidade que o circuito
dedicado dava por construção física.

## Duas formas de uso

A VPN site-to-site liga duas redes. Os roteadores de borda fazem todo o trabalho, os
computadores dos dois lados não sabem que existe túnel, e o tráfego entre as sub-redes
atravessa cifrado sem que ninguém configure nada nas máquinas.

A VPN de acesso remoto liga uma pessoa a uma rede. O programa cliente na máquina do
usuário cria o túnel, e o desenho muda porque o endereço de origem varia a cada
conexão.

Este capítulo trata da primeira, que é a que o curso configura na linha de comando.

## IPsec é um arcabouço, não um algoritmo

Essa distinção derruba gente em prova. O IPsec define quais serviços de segurança
existem e como negociá-los, e deixa a escolha do algoritmo aberta. Isso permite trocar
uma cifra quebrada sem trocar o protocolo, o que já aconteceu mais de uma vez.

Ele oferece quatro serviços. Confidencialidade pela cifra do conteúdo. Integridade por
resumo com chave. Autenticação da origem. E proteção contra repetição, que descarta o
pacote válido capturado e reenviado por um atacante.

Dois cabeçalhos implementam isso. O primeiro autentica e não cifra, o que o torna pouco
útil hoje e incompatível com tradução de endereço, já que ele protege campos que o NAT
altera. O segundo cifra e autentica, e é o que se usa na prática.

Existem também dois modos. No modo transporte, o cabeçalho IP original permanece e só o
conteúdo é protegido, o que serve para comunicação entre dois hosts. No modo túnel, o
pacote inteiro entra dentro de um pacote novo, o que esconde os endereços internos e é o
que a VPN site-to-site usa.

## O que o IKE negocia

Os dois roteadores precisam combinar algoritmos, autenticar-se e derivar chaves antes de
proteger qualquer dado. O protocolo que faz isso trabalha em duas fases, e entender a
divisão explica quase todo defeito de túnel que não sobe.

A primeira fase estabelece um canal seguro para a própria negociação. Os pares acertam
a cifra, o algoritmo de resumo, o método de autenticação, o grupo Diffie-Hellman e o
tempo de vida, executam a troca de chaves e provam identidade um ao outro, por segredo
compartilhado ou por certificado. O resultado é uma associação de segurança
bidirecional.

A segunda fase usa esse canal para negociar a proteção dos dados. Ali entram o conjunto
de transformação e o tráfego que o túnel vai cobrir. O resultado são duas associações
unidirecionais, uma para cada sentido. É possível ativar sigilo futuro perfeito, que
executa uma nova troca Diffie-Hellman na segunda fase, de modo que comprometer a chave
de longo prazo não decifra sessões antigas.

Sobre versões, a primeira geração do IKE é a que o curso configura e a que ainda aparece
em equipamento antigo. A segunda gera menos mensagens, lida melhor com NAT e mobilidade,
e é a escolha correta em projeto novo.

## Os cinco passos

O laboratório fica mais claro quando você enxerga a sequência.

Primeiro, definir o tráfego interessante, que é a lista de acesso descrevendo o que deve
entrar no túnel. Segundo, negociar a primeira fase. Terceiro, negociar a segunda.
Quarto, transferir os dados protegidos. Quinto, encerrar quando o tempo de vida expira
ou o tráfego cessa.

O primeiro passo tem uma armadilha. A lista de tráfego interessante precisa ser
espelhada entre os dois lados: o que um descreve como origem, o outro descreve como
destino. Listas que não se espelham fazem a segunda fase falhar com mensagem obscura, e
esse é o defeito mais comum do laboratório.

## NAT e a ordem das operações

O roteador aplica NAT antes de aplicar a criptografia na saída. Se a sua regra de NAT
traduzir o tráfego que deveria entrar no túnel, ele sai traduzido, deixa de casar com a
lista de tráfego interessante e o túnel nunca sobe. A correção é excluir do NAT o
tráfego destinado à outra ponta, e essa isenção é o segundo defeito mais comum.

## Prática

R1 em 200.1.1.1 e R3 em 200.3.3.3, cada um com uma rede interna: 10.10.0.0/24 do lado do
R1 e 10.30.0.0/24 do lado do R3.

### Primeira fase

```text
R1(config)# crypto isakmp policy 10
R1(config-isakmp)# encryption aes 256
R1(config-isakmp)# hash sha256
R1(config-isakmp)# authentication pre-share
R1(config-isakmp)# group 14
R1(config-isakmp)# lifetime 3600
R1(config-isakmp)# exit
R1(config)# crypto isakmp key Vpn#Ufcg2026 address 200.3.3.3
```

O grupo 14 usa 2048 bits. Grupos 1, 2 e 5 aparecem em material antigo e não devem entrar
em projeto novo.

### Tráfego interessante

```text
R1(config)# ip access-list extended VPN-TRAFEGO
R1(config-ext-nacl)# permit ip 10.10.0.0 0.0.0.255 10.30.0.0 0.0.0.255
```

No R3, a mesma lista com origem e destino invertidos.

### Segunda fase e o mapa

```text
R1(config)# crypto ipsec transform-set TS-AES esp-aes 256 esp-sha256-hmac
R1(cfg-crypto-trans)# mode tunnel
R1(cfg-crypto-trans)# exit
R1(config)# crypto map CMAP 10 ipsec-isakmp
R1(config-crypto-map)# set peer 200.3.3.3
R1(config-crypto-map)# set transform-set TS-AES
R1(config-crypto-map)# set pfs group14
R1(config-crypto-map)# set security-association lifetime seconds 1800
R1(config-crypto-map)# match address VPN-TRAFEGO
R1(config-crypto-map)# exit
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# crypto map CMAP
```

O mapa vai na interface de saída, a que enfrenta a internet. Aplicá-lo na interface
interna é o terceiro erro clássico.

### Isentar o túnel do NAT

```text
R1(config)# ip access-list extended NAT-SAIDA
R1(config-ext-nacl)# deny ip 10.10.0.0 0.0.0.255 10.30.0.0 0.0.0.255
R1(config-ext-nacl)# permit ip 10.10.0.0 0.0.0.255 any
R1(config-ext-nacl)# exit
R1(config)# ip nat inside source list NAT-SAIDA interface gigabitEthernet 0/1 overload
```

A linha de negação vem primeiro e é a isenção. Sem ela, o tráfego do túnel sai traduzido
e o túnel não sobe.

### Levantar e conferir

O túnel só nasce quando aparece tráfego que case com a lista. Gere-o:

```text
PC1> ping 10.30.0.10
```

Depois verifique nos dois lados:

```text
R1# show crypto isakmp sa
R1# show crypto ipsec sa
R1# show crypto map
R1# show crypto session detail
```

O `show crypto isakmp sa` precisa mostrar o estado ativo da primeira fase. Se ele estiver
vazio, o problema está na fase 1: política incompatível, chave diferente ou endereço de
par errado. Se ele mostrar ativo e o `show crypto ipsec sa` não contar pacotes cifrados,
o problema está na fase 2, e a suspeita principal é a lista de tráfego não espelhada.

Os contadores de `show crypto ipsec sa` são o teste final. Pacotes encapsulados subindo
de um lado e desencapsulados subindo do outro provam que o túnel transporta dados de
verdade. Contador parado com associação estabelecida significa túnel montado que ninguém
usa, o que costuma ser roteamento ou NAT.

## Laboratórios do curso

O assunto aparece nos laboratórios 19.5.5 e 19.5.6 do módulo 19.

O último capítulo fecha o guia testando tudo que os sete anteriores configuraram.
