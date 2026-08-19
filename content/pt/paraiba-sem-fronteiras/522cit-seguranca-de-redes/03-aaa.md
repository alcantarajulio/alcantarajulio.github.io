---
title: "AAA: autenticação, autorização e contabilização"
description: "Tirar as credenciais de dentro do equipamento, escolher entre RADIUS e TACACS+, e não se trancar para fora no processo."
date: 2026-08-19
chapter: 3
translationKey: "psf-522-03"
weight: 30
tags: ["aaa", "radius", "tacacs", "cisco"]
---

Contas locais funcionam bem em três roteadores. Em trinta, elas viram um problema de
sincronização: alguém entra na equipe e você configura trinta vezes, alguém sai e você
esquece de um, e a auditoria descobre seis meses depois que o estagiário do semestre
passado ainda entra no roteador da borda.

## Três perguntas diferentes

A sigla junta três funções que resolvem problemas distintos e que costumam ser
confundidas em prova.

A autenticação responde quem é você. A autorização responde o que você pode fazer depois
de entrar. A contabilização responde o que você fez, e é a que sustenta a auditoria.

Um sistema pode autenticar sem autorizar de forma granular, e é o que acontece quando
todo administrador entra no nível 15. Pode também autenticar e autorizar sem registrar,
e aí você descobre o que aconteceu perguntando às pessoas.

## Local e centralizado

O AAA local guarda a base de usuários no próprio equipamento. Ele já é melhor que senha
de linha, porque traz identidade individual, e continua sofrendo do problema de escala.

O AAA centralizado põe a base em um servidor. A revogação passa a acontecer em um lugar
só, a política de senha se aplica de forma uniforme, e o registro de acesso a toda a
rede cai em um repositório único. O custo é uma dependência nova: se o servidor cai e
você não previu alternativa, ninguém entra em lugar nenhum.

Essa alternativa se chama lista de métodos, e é a parte que derruba aluno em laboratório.
Você declara a ordem de tentativa: primeiro o servidor, depois a base local. O
equipamento só passa para o segundo método quando o primeiro não responde. Servidor que
responde negando não aciona a alternativa, e essa distinção entre "não respondeu" e
"respondeu não" é a diferença entre entrar e ficar de fora.

## RADIUS e TACACS+

Os dois protocolos fazem AAA e servem propósitos diferentes.

O RADIUS roda sobre UDP, cifra apenas o campo de senha e deixa o resto do pacote legível,
inclusive o nome do usuário. Ele combina autenticação e autorização em uma troca só, o
que reduz a granularidade. É padrão aberto, tem implementação em todo lugar, e domina o
acesso de usuário final: rede sem fio, 802.1X, VPN.

O TACACS+ roda sobre TCP, cifra o corpo inteiro do pacote e separa as três funções em
trocas independentes. Essa separação permite autorização comando a comando: o servidor
decide, a cada linha digitada, se aquele administrador pode executá-la. O protocolo é da
Cisco, e o uso natural dele é a administração de dispositivo de rede.

A escolha prática costuma ser a combinação dos dois. TACACS+ para quem administra o
equipamento, RADIUS para quem apenas usa a rede.

## Contabilização é o que sobra depois

O registro de AAA responde perguntas que o syslog do equipamento responde mal. Quem
abriu sessão, a que horas, de onde, por quanto tempo, e quais comandos executou. Em
ambiente com TACACS+ e autorização por comando, o registro guarda a linha exata.

Isso tem um efeito colateral que vale dizer em voz alta na oficina: contabilização
completa registra o comportamento das pessoas, e existe uma conversa sobre proporção e
transparência que não é técnica. Registrar comando de administrador em equipamento
crítico é prática comum e defensável. Registrar tudo de todos sem avisar ninguém é outra
coisa.

## Prática

O cenário acrescenta um servidor AAA em 10.20.0.20. O Packet Tracer traz um servidor
com RADIUS e TACACS+ embutidos, o que basta para o laboratório.

### Preparar a alternativa local antes de qualquer coisa

Esta é a ordem que evita o desastre. Crie a conta local primeiro, depois ligue o AAA:

```text
R1(config)# username julio privilege 15 algorithm-type scrypt secret Intercambio#25
R1(config)# aaa new-model
```

### Apontar para os servidores

```text
R1(config)# tacacs server AAA-TAC
R1(config-server-tacacs)# address ipv4 10.20.0.20
R1(config-server-tacacs)# key Tac#Redes2026
R1(config-server-tacacs)# exit
R1(config)# aaa group server tacacs+ GRUPO-TAC
R1(config-sg-tacacs+)# server name AAA-TAC
```

Para RADIUS, o equivalente:

```text
R1(config)# radius server AAA-RAD
R1(config-radius-server)# address ipv4 10.20.0.20 auth-port 1812 acct-port 1813
R1(config-radius-server)# key Rad#Redes2026
```

### Listas de métodos

A lista padrão vale para tudo que não tiver lista própria. A lista nomeada para o
console é o seu seguro contra perda do servidor:

```text
R1(config)# aaa authentication login default group GRUPO-TAC local
R1(config)# aaa authentication login CONSOLE local
R1(config)# aaa authentication enable default group GRUPO-TAC enable
```

Aplique a lista do console na linha correspondente:

```text
R1(config)# line console 0
R1(config-line)# login authentication CONSOLE
```

### Autorização e contabilização

```text
R1(config)# aaa authorization exec default group GRUPO-TAC local
R1(config)# aaa authorization commands 15 default group GRUPO-TAC local
R1(config)# aaa accounting exec default start-stop group GRUPO-TAC
R1(config)# aaa accounting commands 15 default start-stop group GRUPO-TAC
```

O `local` no fim das linhas de autorização importa tanto quanto no da autenticação.
Sem ele, servidor indisponível significa administrador autenticado que não consegue
executar nada.

### Conferir antes de fechar a sessão

Nunca feche a sessão atual sem testar em outra. Abra um segundo terminal e verifique:

```text
R1# test aaa group GRUPO-TAC julio Intercambio#25 legacy
R1# show aaa servers
R1# show aaa sessions
R1# debug aaa authentication
```

O `test aaa` valida a comunicação com o servidor sem depender de uma nova sessão de
login. O `debug aaa authentication` mostra qual método a lista tentou e em que ordem,
que é a informação que falta quando o login falha sem mensagem clara.

### O erro clássico

Ligar `aaa new-model` sem criar usuário local e sem servidor alcançável tranca o
equipamento. O `aaa new-model` muda o comportamento padrão das linhas na hora, e a senha
de linha que funcionava para de valer. A recuperação passa por acesso físico ao console
e quebra de senha na inicialização.

## Laboratórios do curso

O assunto aparece nos laboratórios 7.2.5 e 7.2.6 do módulo 7.

O próximo capítulo sai do controle sobre quem administra o equipamento e passa ao
controle sobre o tráfego que atravessa a rede.
