---
title: "IPS, endpoint e camada 2"
description: "Detecção de intrusão, defesa do host, e os ataques que acontecem no andar de baixo da pilha."
date: 2026-08-19
chapter: 5
translationKey: "psf-522-05"
weight: 50
tags: ["ips", "ids", "endpoint", "layer2", "switch"]
---

O firewall decide com base em endereço, porta e estado da conexão. Isso deixa passar
tudo que é malicioso e ao mesmo tempo bem formado: a requisição HTTP com injeção de SQL
chega na porta 443 de uma sessão legítima. Este capítulo trata das três camadas de
defesa que continuam depois da filtragem.

## Detectar e prevenir são posições diferentes

Um sistema de detecção de intrusão recebe uma cópia do tráfego, analisa fora do caminho
e emite alerta. Ele não atrasa nada e não bloqueia nada, então um ataque bem-sucedido
gera um alerta sobre um estrago que já aconteceu.

Um sistema de prevenção fica no caminho. Todo pacote atravessa a análise antes de
seguir, o que permite descartar a sessão maliciosa antes que ela chegue ao destino. O
preço aparece em dois lugares: a latência que a análise acrescenta, e o falso positivo,
que no modo em linha derruba tráfego legítimo em vez de gerar um alerta que alguém
ignora.

A escolha entre os dois é uma escolha sobre o custo do erro. Em rede de laboratório, o
falso positivo custa um chamado. Em sistema de pagamento, custa receita por minuto.

## Os quatro resultados possíveis

Toda decisão de detecção cai em uma de quatro caixas, e a prova cobra os nomes.

O verdadeiro positivo alerta sobre um ataque real. O verdadeiro negativo fica calado
diante de tráfego legítimo. O falso positivo alerta sobre tráfego normal. O falso
negativo fica calado diante de um ataque.

Os dois erros não custam o mesmo. O falso negativo é a invasão que ninguém viu. O falso
positivo é barulho, e barulho suficiente produz um efeito pior que o próprio erro: a
equipe para de olhar os alertas. Ajustar um IPS consiste em quase todo o tempo reduzir
falso positivo sem criar falso negativo, e esse ajuste nunca termina.

## Como o sistema decide

A detecção por assinatura compara o tráfego com padrões conhecidos. Ela é precisa e cega
para o que ainda não tem assinatura, o que inclui todo ataque novo.

A detecção por anomalia aprende o comportamento normal da rede e alerta sobre o desvio.
Ela pega o que não tem assinatura e gera falso positivo sempre que o normal muda, o que
acontece toda vez que alguém sobe um serviço novo.

A detecção por política bloqueia o que a organização declarou proibido, sem julgar se é
ataque. E o honeypot atrai o atacante para um alvo falso, o que serve para estudar
técnica e para desviar atenção.

Quando o sistema decide agir, as opções vão de registrar o evento a descartar o pacote,
reiniciar a conexão ou bloquear o endereço de origem por um tempo.

## Onde a defesa mora

Um sensor de rede vê o tráfego de muitos hosts e não vê nada do que acontece dentro
deles. Ele também fica cego diante de tráfego cifrado, que hoje é a maior parte.

Um agente no host vê o oposto: chamada de sistema, alteração de arquivo, processo que
nasce, e o conteúdo depois da decifragem. Ele custa instalação e manutenção em cada
máquina, e o atacante que ganha privilégio no host desliga o agente.

As duas visões se complementam, e a defesa do endpoint hoje soma antimalware, controle
de aplicação, cifra de disco, e verificação de postura antes de liberar acesso à rede.
Essa última fecha o círculo com o capítulo anterior: a máquina que não tem correção
aplicada entra em uma rede restrita até se ajustar.

## O andar de baixo

Tudo que os capítulos anteriores construíram assume que a camada 2 funciona. Ela é a
fundação, e quem a compromete não precisa lutar contra o que está acima.

A inundação da tabela de endereços enche o switch com endereços falsos até ele parar de
aprender e passar a inundar todo quadro por todas as portas, o que transforma o switch
em hub e entrega o tráfego a quem estiver escutando.

O salto de VLAN atravessa a separação que você criou. Em uma variante, o atacante
negocia um enlace tronco com o switch e passa a receber todas as VLANs. Na outra, ele
insere duas etiquetas no quadro, o primeiro switch remove uma e encaminha, e o segundo
entrega o quadro na VLAN de destino.

Contra o DHCP existem dois ataques que se combinam. O primeiro esgota o conjunto de
endereços pedindo concessões com endereços de origem falsos. O segundo sobe um servidor
falso que responde antes do verdadeiro e distribui a si mesmo como gateway, o que coloca
o atacante no meio de toda conversa.

O envenenamento de ARP explora um protocolo que acredita em qualquer resposta. O
atacante afirma ser o gateway para a vítima e a vítima para o gateway, e passa a ver as
duas direções.

E o ataque à raiz do spanning tree anuncia uma prioridade melhor, vence a eleição e
redesenha a topologia para atravessar o equipamento do atacante.

## O conjunto de contramedidas

Cada ataque acima tem uma resposta específica, e elas dependem umas das outras.

A segurança de porta limita quantos endereços cada porta aprende, o que resolve a
inundação da tabela. O rastreamento de DHCP classifica portas em confiáveis e não
confiáveis, descarta resposta de servidor vinda de porta não confiável, e constrói uma
tabela que associa endereço IP, endereço físico e porta. A inspeção dinâmica de ARP usa
essa tabela para descartar resposta de ARP que não bate com ela, e é por isso que ela
depende do rastreamento estar ligado. A proteção contra BPDU derruba a porta de acesso
que receber mensagem de spanning tree, já que nenhum computador deveria enviar uma.

Fora isso, três hábitos de configuração fecham o resto. Desligue a negociação automática
de tronco nas portas de acesso, para que ninguém negocie um tronco com você. Troque a
VLAN nativa por uma que não transporte dado de usuário, o que quebra o salto por dupla
etiqueta. E deixe as portas não usadas desligadas e associadas a uma VLAN sem saída.

## Prática

Um switch com PCs em `f0/1` a `f0/5`, o servidor DHCP legítimo em `f0/24` e o roteador
em `g0/1`.

### Segurança de porta

```text
SW1(config)# interface range fastEthernet 0/1 - 5
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport port-security
SW1(config-if-range)# switchport port-security maximum 2
SW1(config-if-range)# switchport port-security mac-address sticky
SW1(config-if-range)# switchport port-security violation restrict
SW1(config-if-range)# switchport port-security aging time 60
```

A ação em caso de violação tem três opções. Proteger descarta em silêncio, restringir
descarta e registra, e desligar derruba a porta até intervenção manual. Desligar é a mais
segura e a que mais gera chamado, porque um usuário que troca de mesa derruba a própria
porta.

### Rastreamento de DHCP

```text
SW1(config)# ip dhcp snooping
SW1(config)# ip dhcp snooping vlan 10
SW1(config)# no ip dhcp snooping information option
SW1(config)# interface fastEthernet 0/24
SW1(config-if)# ip dhcp snooping trust
SW1(config-if)# exit
SW1(config)# interface range fastEthernet 0/1 - 5
SW1(config-if-range)# ip dhcp snooping limit rate 6
```

### Inspeção dinâmica de ARP

```text
SW1(config)# ip arp inspection vlan 10
SW1(config)# interface fastEthernet 0/24
SW1(config-if)# ip arp inspection trust
SW1(config-if)# exit
SW1(config)# ip arp inspection validate src-mac dst-mac ip
```

### Spanning tree e troncos

```text
SW1(config)# spanning-tree portfast default
SW1(config)# spanning-tree portfast bpduguard default
SW1(config)# interface gigabitEthernet 0/1
SW1(config-if)# spanning-tree guard root
SW1(config-if)# exit
SW1(config)# interface range fastEthernet 0/1 - 5
SW1(config-if-range)# switchport nonegotiate
SW1(config-if-range)# exit
SW1(config)# interface gigabitEthernet 0/2
SW1(config-if)# switchport trunk native vlan 999
```

### Portas ociosas

```text
SW1(config)# interface range fastEthernet 0/6 - 23
SW1(config-if-range)# switchport access vlan 999
SW1(config-if-range)# shutdown
```

### Conferir

```text
SW1# show port-security
SW1# show port-security address
SW1# show ip dhcp snooping
SW1# show ip dhcp snooping binding
SW1# show ip arp inspection
SW1# show ip arp inspection statistics
SW1# show spanning-tree inconsistentports
SW1# show interfaces status err-disabled
```

A tabela de associações do rastreamento de DHCP é o artefato central. Se ela estiver
vazia, a inspeção de ARP vai descartar tráfego legítimo, e o sintoma é uma rede que para
logo depois de você achar que a protegeu.

## Laboratórios do curso

Os módulos 11 a 14 do curso não trazem laboratório de Packet Tracer. A prática acima é
montável com um switch e três PCs, e o teste do lado ofensivo pede uma máquina com
ferramentas de geração de tráfego, que o Packet Tracer não simula.

O próximo capítulo muda de assunto e vai para a base matemática que sustenta o que vem
depois: criptografia.
