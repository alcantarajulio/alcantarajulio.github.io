---
title: "Acesso seguro a dispositivos de rede"
description: "Por que o plano de gerência é o primeiro alvo, o que uma senha armazenada vale, e como o privilégio deveria ser distribuído."
date: 2026-08-19
chapter: 1
translationKey: "psf-522-01"
weight: 10
tags: ["cisco", "ios", "ssh", "hardening"]
---

Um roteador sai de fábrica pronto para ser administrado por quem chegar primeiro. Aceita
Telnet, guarda a senha em texto legível dentro do arquivo de configuração e concede
privilégio total a quem souber uma única palavra. No laboratório da faculdade, essa
palavra costuma sobreviver a três semestres.

Este capítulo trata do que precisa mudar nesse estado inicial e da razão de cada
mudança. Firewall, IPS e VPN não protegem um equipamento que aceita login remoto sem
criptografia, e é por isso que a trilha começa aqui.

## Os três planos e onde dói mais

Um equipamento de rede opera em três planos. O plano de dados encaminha os pacotes dos
usuários. O plano de controle troca rotas, mantém a topologia e decide para onde o
tráfego vai. O plano de gerência é por onde você configura os outros dois.

A diferença de impacto entre eles não é de grau. Quem compromete o plano de dados lê o
tráfego que passa naquele ponto. Quem compromete o plano de gerência reescreve as listas
de acesso, desliga o registro de eventos, levanta um túnel para fora e apaga o rastro da
própria visita. Você defende os dois primeiros planos usando o terceiro, o que torna o
terceiro o alvo de maior retorno para o atacante.

## O que uma senha armazenada vale

O IOS guarda senha em formatos diferentes, e a escolha entre eles decide o que acontece
quando o arquivo de configuração vaza. E ele vaza: vai para backup, entra em anexo de
chamado, aparece em repositório git, viaja por e-mail até o suporte do fabricante.

O tipo 0 grava o texto puro. O tipo 7 aplica uma cifra de Vigenère com chave conhecida,
que qualquer página na internet reverte em um segundo. Esse formato existe para
atrapalhar quem lê por cima do seu ombro, e cumpre só isso. O tipo 5 usa MD5, adequado
em 1995 e quebrado para esse uso desde os anos 2000, porque uma GPU testa bilhões de
candidatos por segundo. Os tipos 8 e 9, PBKDF2 e scrypt, foram desenhados para serem
lentos de propósito, e o scrypt ainda exige memória, o que encarece o ataque em hardware
dedicado.

Daí vem uma armadilha comum. O comando que "criptografa" as senhas do arquivo aplica
tipo 7, reversível. Ele muda a aparência da configuração e não muda a segurança dela.
Quem vê o arquivo cifrado e conclui que pode compartilhá-lo entendeu o contrário do que
o comando faz.

## Credencial compartilhada custa a responsabilização

Uma senha de linha autentica o equipamento, não a pessoa. Quando três alunos usam a
mesma credencial, o registro guarda três sessões idênticas, e a pergunta de quem apagou
a configuração fica sem resposta.

Contas nominais existem para separar autenticação de responsabilização. A primeira
decide se você entra. A segunda decide se, meses depois, alguém consegue reconstruir o
que aconteceu. Auditoria sem identidade individual é uma lista de eventos sem sujeito.

## Por que o Telnet saiu de cena

O Telnet transporta usuário, senha e todo o resto da sessão em texto claro. Basta um
ponto de observação no caminho: uma porta espelhada no switch, um envenenamento de ARP
no segmento, um roteador intermediário comprometido. O atacante não precisa quebrar
nada, só ler.

O SSH resolve dois problemas de uma vez. Ele cifra a sessão, o que derruba a captura
passiva, e autentica o servidor por chave de host, o que dificulta a um impostor se
passar pelo roteador. A primeira conexão ainda exige confiança no que você vê, e por
isso vale comparar a impressão digital da chave por um canal separado.

Duas escolhas dentro do SSH importam. A versão 1 tem falhas de integridade e não deve
ser aceita, então só a versão 2 conta. E o tamanho da chave RSA precisa de 2048 bits no
mínimo, porque 1024 saiu da faixa considerada segura.

## Onde a gerência deveria morar

Criptografia protege o conteúdo da sessão. Não reduz quem consegue bater na porta.

O ideal é uma rede de gerência fora de banda, fisicamente separada do tráfego dos
usuários, alcançável só de onde os administradores trabalham. Quando o orçamento ou a
topologia não permitem, a alternativa é filtrar por endereço de origem nas linhas de
acesso remoto, aceitando conexão apenas da faixa administrativa.

As duas medidas resolvem coisas distintas e se somam. A cifra impede que alguém leia a
sua sessão. A restrição de origem impede que a maioria sequer chegue a tentar.

## O custo de tentar

Um ataque de dicionário depende de tentativas baratas. Encarecê-las muda o cálculo: o
equipamento fecha o login por um período depois de um número de falhas dentro de uma
janela de tempo, e registra cada falha.

Esse controle tem um efeito colateral que aparece na prova e no plantão. Se o bloqueio
vale para todo mundo, um atacante derruba o seu acesso administrativo de propósito,
errando senha até o equipamento fechar. Por isso o bloqueio precisa de uma exceção para
a faixa de gerência, ou você troca um risco de invasão por uma negação de serviço contra
si mesmo.

## O banner é peça jurídica

O aviso de acesso não serve de enfeite. Em processo por acesso não autorizado, a defesa
argumenta que o sistema estava aberto ao público e que não havia como saber. Um texto
explícito de restrição, registro e consequência legal derruba esse argumento. A palavra
"bem-vindo" o sustenta, e por isso não entra em banner de equipamento.

## Privilégio proporcional ao papel

O IOS oferece 16 níveis de privilégio. O nível 1 abre o modo de usuário, o 15 abre tudo,
e os 14 do meio ficam vazios até alguém preenchê-los. O monitor da oficina que só precisa
olhar o estado das interfaces não tem uso para o nível 15.

Esse modelo numérico tem um limite estrutural: ele herda para cima. Quem está no nível 5
carrega tudo que o nível 1 carrega, e você não consegue descrever dois papéis paralelos
com conjuntos de comandos que se cruzam sem se conter. As views baseadas em função
existem para isso. Cada view lista os comandos que aquele papel executa, sem hierarquia
entre elas, e uma superview agrupa views quando um técnico acumula funções.

O princípio por trás dos dois mecanismos é o mesmo, e é o do menor privilégio: cada conta
recebe o necessário para a função e nada além, porque toda permissão extra é uma
permissão que o atacante herda ao comprometer aquela conta.

## Verificar faz parte do controle

Configurar sem conferir produz sensação de segurança, que atrapalha mais que a
insegurança assumida, porque interrompe a investigação.

Quatro perguntas fecham o capítulo na prática. Quais protocolos as linhas de acesso
remoto aceitam de fato, e o Telnet continua entre eles? Qual o tamanho da chave em uso?
O bloqueio por tentativa está armado, e quantas falhas o equipamento contou até agora?
Quais contas existem, e alguma delas ficou com privilégio maior do que a função pede?

## Prática

A configuração abaixo aplica o capítulo inteiro em um roteador. Monte no Packet Tracer
um roteador, um switch e dois PCs, um deles na faixa de gerência 10.20.0.0/24 e o outro
fora dela.

### Senhas e contas

Defina o comprimento mínimo antes de criar qualquer credencial, senão o IOS aceita as
que você já criou e só cobra das próximas:

```text
Router(config)# security passwords min-length 10
Router(config)# enable algorithm-type scrypt secret Ufcg#2026Redes
Router(config)# no enable password
Router(config)# service password-encryption
```

O `no enable password` importa. Com as duas linhas presentes, o IOS autentica pelo
`enable secret` e deixa a outra senha legível no `running-config`.

Crie contas nominais em vez de usar senha de linha:

```text
Router(config)# username julio privilege 15 algorithm-type scrypt secret Intercambio#25
Router(config)# username operador privilege 5 algorithm-type scrypt secret Oficina#25
```

### SSH no lugar do Telnet

A chave RSA deriva do nome do equipamento e do domínio, então configure os dois antes
de gerar. Gerar fora de ordem produz chave com nome errado e o SSH não sobe:

```text
Router(config)# hostname R1
R1(config)# ip domain-name redes.ufcg.br
R1(config)# crypto key generate rsa general-keys modulus 2048
R1(config)# ip ssh version 2
R1(config)# ip ssh time-out 60
R1(config)# ip ssh authentication-retries 2
```

Em IOS-XE a partir da versão 16 o comando perdeu o hífen e virou `ip domain name`.

Agora feche as linhas de acesso remoto:

```text
R1(config)# line vty 0 4
R1(config-line)# transport input ssh
R1(config-line)# login local
R1(config-line)# exec-timeout 5 0
R1(config-line)# logging synchronous
```

Sem `login local`, a linha continua pedindo senha de linha e as contas nominais não
servem de nada. Deixar `transport input all` mantém o Telnet vivo ao lado do SSH e anula
o trabalho.

A porta auxiliar merece o mesmo tratamento, já que quase ninguém a usa e por isso ninguém
a vigia:

```text
R1(config)# line aux 0
R1(config-line)# no exec
R1(config-line)# transport input none
```

### Restringir a origem e conter a tentativa

```text
R1(config)# ip access-list standard GERENCIA
R1(config-std-nacl)# permit 10.20.0.0 0.0.0.255
R1(config-std-nacl)# exit
R1(config)# line vty 0 4
R1(config-line)# access-class GERENCIA in
R1(config-line)# exit
R1(config)# login block-for 120 attempts 3 within 60
R1(config)# login quiet-mode access-class GERENCIA
R1(config)# login on-failure log
R1(config)# login on-success log
```

O `login quiet-mode access-class` é a exceção que a teoria pediu: durante o bloqueio, o
roteador continua aceitando conexão vinda da faixa de gerência, e você não fica de fora
do próprio equipamento.

### Banner

```text
R1(config)# banner motd ^
Acesso restrito a pessoal autorizado.
Toda atividade nesta sessao e registrada e auditada.
A conexao sem autorizacao configura crime previsto em lei.
^
```

### Privilégio

Níveis numéricos resolvem o caso simples:

```text
R1(config)# privilege exec level 5 show ip interface brief
R1(config)# privilege exec level 5 show version
R1(config)# privilege exec level 5 ping
```

Papéis paralelos exigem views, que por sua vez exigem AAA ligado:

```text
R1(config)# aaa new-model
R1# enable view
R1(config)# parser view MONITOR
R1(config-view)# secret Oficina#Ver25
R1(config-view)# commands exec include show ip interface brief
R1(config-view)# commands exec include show ip route
R1(config-view)# commands exec include ping
```

### Conferir

```text
R1# show ip ssh
R1# show ssh
R1# show login
R1# show users
R1# show parser view all
R1# show running-config | include username|enable
```

O `show ip ssh` confirma a versão 2 e o tamanho da chave. O `show login` mostra se o
bloqueio está armado e quantas falhas o roteador contou.

Para fechar, teste dos dois lados: abra SSH do PC na faixa de gerência e confirme o
acesso, tente do PC de fora e falhe, erre a senha quatro vezes e veja o contador subir
no `show login`.

Um aviso antes de sair do modo de configuração: `login local` sem nenhum usuário criado
tranca você para fora, e a recuperação passa por reinicialização com quebra de senha.

## Laboratórios do curso

Este guia não reproduz os laboratórios. Quem tiver conta no Networking Academy e o
Packet Tracer instalado encontra o assunto deste capítulo nos laboratórios 4.4.7, 4.4.8
e 4.4.9 do módulo 4, e no 5.2.5 do módulo 5.

No próximo capítulo, o alvo passa a ser o resto do equipamento: serviços que ninguém usa
e continuam ligados, protocolos de roteamento que aceitam qualquer vizinho e registros
que não chegam a lugar nenhum.
