---
title: "Criptografia aplicada"
description: "Confidencialidade, integridade e autenticidade: cifras simétricas, chave pública, resumo, HMAC, assinatura e PKI."
date: 2026-08-19
chapter: 6
translationKey: "psf-522-06"
weight: 60
tags: ["criptografia", "openssl", "pki", "hmac", "aes"]
---

Os capítulos anteriores usaram criptografia sem explicá-la. O SSH cifra a sessão, o OSPF
autentica a vizinhança com um resumo, o SNMPv3 protege o conteúdo. Este capítulo abre a
caixa, porque o próximo constrói uma VPN inteira em cima dela.

## Três objetivos que costumam ser confundidos

A confidencialidade impede que alguém leia. A integridade permite detectar que alguém
alterou. A autenticidade prova quem enviou. O não repúdio impede que o remetente negue
depois que enviou.

Nem toda ferramenta entrega os quatro, e escolher a errada produz sistema que parece
seguro. Cifrar sem verificar integridade deixa o atacante alterar o criptograma às cegas
e provocar efeito no destinatário. Verificar integridade sem autenticar permite que o
atacante altere a mensagem e recalcule o resumo. Cada objetivo tem sua ferramenta.

## O princípio que define o campo

Kerckhoffs, no século XIX, estabeleceu que um sistema deve permanecer seguro mesmo que
tudo sobre ele seja conhecido, exceto a chave. Isso parece óbvio hoje e continua sendo
violado toda vez que alguém confia em um algoritmo secreto.

A razão é prática. Algoritmo vaza: por engenharia reversa, por funcionário que sai, por
publicação acidental. Chave se troca em minutos. Um sistema que depende do segredo do
algoritmo perde tudo de uma vez, e não tem como se recuperar.

## Cifras simétricas

Uma cifra simétrica usa a mesma chave para cifrar e decifrar. Ela é rápida e resolve
volume, e por isso protege o corpo de toda sessão cifrada que você usa.

O DES, com chave de 56 bits, caiu para força bruta em 1998 e não deve aparecer em
projeto novo. O 3DES aplicou a mesma cifra três vezes para ganhar sobrevida, custa caro
em desempenho e está sendo aposentado. O AES é o padrão atual, com chave de 128, 192 ou
256 bits.

O modo de operação importa tanto quanto o algoritmo. O modo mais simples cifra cada
bloco de forma independente, o que faz blocos idênticos de texto claro gerarem blocos
idênticos de criptograma, e a imagem cifrada continua reconhecível. O encadeamento de
blocos resolve isso misturando cada bloco com o anterior. E o modo autenticado moderno
entrega cifra e verificação de integridade na mesma operação, que é a resposta correta
para quase todo caso novo.

O problema que a criptografia simétrica não resolve sozinha é o da distribuição da
chave. Duas partes precisam combinar um segredo antes de conversar, e combinar um
segredo por um canal inseguro é o problema original.

## Chave pública

A criptografia assimétrica usa um par de chaves ligadas por matemática. O que uma cifra,
só a outra decifra. Uma delas você publica, a outra guarda.

Isso resolve a distribuição, e resolve mais que isso. Cifrar com a chave pública do
destinatário dá confidencialidade, já que só ele decifra. Cifrar com a sua chave privada
dá autenticidade, já que qualquer um verifica com a sua pública e só você poderia ter
produzido aquilo.

O custo é desempenho. Operação assimétrica é ordens de magnitude mais lenta que
simétrica, o que inviabiliza cifrar um fluxo inteiro com ela. Daí o desenho híbrido que
todo protocolo moderno usa: a assimétrica negocia uma chave de sessão, e a simétrica
cifra os dados com essa chave.

A troca de chaves de Diffie-Hellman merece destaque porque faz algo contraintuitivo:
duas partes chegam a um segredo compartilhado trocando mensagens em público, sem que
quem escuta consiga derivar o mesmo segredo. O IPsec do próximo capítulo depende disso.

Entre as famílias, o RSA baseia a segurança na dificuldade de fatorar números grandes e
exige chave de 2048 bits ou mais. A criptografia de curva elíptica alcança segurança
equivalente com chave muito menor, o que importa em dispositivo com pouca energia.

## Resumo criptográfico

Uma função de resumo transforma entrada de qualquer tamanho em saída de tamanho fixo, de
forma que a mesma entrada sempre produz a mesma saída e mudar um bit muda a saída
inteira.

Ela não é criptografia, porque não existe operação inversa, e essa confusão aparece em
prova. Resumo serve para detectar alteração, não para esconder conteúdo.

O MD5 e o SHA-1 sofrem ataque de colisão prático, o que significa que alguém produz dois
documentos diferentes com o mesmo resumo. Para verificação de integridade contra
adversário, use SHA-2 ou SHA-3.

Existe um limite que o resumo puro não vence. Se o atacante altera a mensagem, ele
recalcula o resumo e envia os dois. A defesa é misturar uma chave no cálculo, e o
resultado se chama HMAC: quem não tem a chave não produz um resumo válido. É isso que o
OSPF e o NTP dos capítulos anteriores usam.

## Assinatura e infraestrutura de chave pública

A assinatura digital combina resumo e chave privada. Você calcula o resumo da mensagem e
o cifra com a sua chave privada. Quem recebe decifra com a sua chave pública, recalcula
o resumo e compara. Isso entrega integridade, autenticidade e não repúdio de uma vez, e
custa pouco, já que a operação lenta acontece sobre o resumo e não sobre a mensagem.

Falta uma peça. Verificar uma assinatura com uma chave pública prova que quem assinou
detém a privada correspondente, e não prova quem é essa pessoa. A chave pública que você
recebeu pode ser do atacante.

A infraestrutura de chave pública resolve por delegação de confiança. Uma autoridade
certificadora verifica a identidade de alguém e emite um certificado, que é a chave
pública dessa pessoa assinada pela autoridade. Você confia na autoridade, então aceita
as chaves que ela atesta.

Isso transfere o problema em vez de eliminá-lo, e a transferência vale a pena porque o
número de autoridades é pequeno. O sistema tem custos conhecidos: a autoridade
comprometida derruba todos os certificados que emitiu, e a revogação depende de listas
ou de consulta em tempo real que nem todo cliente verifica.

## Prática

A prática aqui roda em Linux com OpenSSL, e não no Packet Tracer, porque o simulador não
implementa essas operações. Qualquer máquina com OpenSSL instalado serve.

### Resumo e o efeito avalanche

```text
$ echo -n "seguranca de redes" | openssl dgst -sha256
$ echo -n "seguranca de rede"  | openssl dgst -sha256
```

Compare as duas saídas. Um caractere de diferença muda o resumo inteiro, e é isso que
torna a verificação de integridade útil.

### HMAC contra resumo puro

```text
$ echo -n "transferir 100" | openssl dgst -sha256
$ echo -n "transferir 100" | openssl dgst -sha256 -hmac "chave-secreta"
```

Peça ao aluno que recalcule o primeiro sem nenhuma informação extra, o que ele consegue,
e depois o segundo, o que ele não consegue sem a chave.

### Cifra simétrica

```text
$ openssl enc -aes-256-cbc -pbkdf2 -salt -in mensagem.txt -out mensagem.enc
$ openssl enc -d -aes-256-cbc -pbkdf2 -in mensagem.enc -out recuperada.txt
$ diff mensagem.txt recuperada.txt
```

### Par de chaves, assinatura e verificação

```text
$ openssl genrsa -out privada.pem 2048
$ openssl rsa -in privada.pem -pubout -out publica.pem
$ openssl dgst -sha256 -sign privada.pem -out assinatura.bin mensagem.txt
$ openssl dgst -sha256 -verify publica.pem -signature assinatura.bin mensagem.txt
```

Altere um byte de `mensagem.txt` e verifique de novo. A verificação falha, e é essa
falha que demonstra a integridade.

### Certificado

```text
$ openssl req -new -key privada.pem -out pedido.csr
$ openssl x509 -req -days 365 -in pedido.csr -signkey privada.pem -out certificado.crt
$ openssl x509 -in certificado.crt -text -noout
```

O certificado gerado é autoassinado, então nenhum navegador confia nele. Isso é a
demonstração do capítulo: a matemática funciona e a confiança falta, e é exatamente essa
lacuna que a autoridade certificadora preenche.

### Do lado do roteador

```text
R1(config)# crypto key generate rsa general-keys modulus 2048 label CA-KEYS
R1(config)# crypto pki trustpoint CA-LOCAL
R1(ca-trustpoint)# enrollment terminal
R1(ca-trustpoint)# subject-name CN=R1.redes.ufcg.br
R1(ca-trustpoint)# revocation-check none
R1# show crypto key mypubkey rsa
R1# show crypto pki certificates
```

## Laboratórios do curso

O assunto aparece no laboratório 15.0.3 do módulo 15, e nos 16.3.10 e 16.3.12 do módulo
16. O 16.3.12 fecha o círculo com o capítulo 1: você captura uma sessão Telnet e uma
sessão SSH no Wireshark e compara o que aparece.

O próximo capítulo usa tudo isto para montar um túnel entre dois roteadores.
