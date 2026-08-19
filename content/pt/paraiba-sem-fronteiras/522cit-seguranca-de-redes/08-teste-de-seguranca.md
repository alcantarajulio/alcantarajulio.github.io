---
title: "Teste de segurança de redes"
description: "Verificar na rede o que você acredita ter configurado, com autorização por escrito e método repetível."
date: 2026-08-19
chapter: 8
translationKey: "psf-522-08"
weight: 80
tags: ["pentest", "auditoria", "nmap", "wireshark"]
---

Os sete capítulos anteriores produziram uma configuração. Este trata da distância entre
a configuração que você acredita ter aplicado e o comportamento que a rede apresenta de
verdade, que é onde moram os incidentes.

## Por que a verificação é uma disciplina separada

Você aplicou a lista de acesso na interface errada. Você configurou o firewall por zonas
e esqueceu uma interface fora de zona. Você desligou o Telnet em três roteadores e o
quarto ficou de fora porque a sessão caiu no meio. Nenhum desses erros aparece lendo a
configuração com atenção, porque quem lê é a mesma pessoa que escreveu e lê o que
pretendia escrever.

O teste inverte a posição. Em vez de perguntar o que a configuração diz, ele pergunta o
que a rede responde.

## Autorização vem antes de tudo

Varrer uma rede sem autorização é crime. No Brasil a Lei 12.737 tipifica a invasão de
dispositivo informático, e no Reino Unido o Computer Misuse Act cobre o mesmo terreno
com penas maiores. Nenhuma das duas abre exceção para curiosidade acadêmica ou para boa
intenção.

Antes de qualquer varredura, existe um documento. Ele define o escopo por endereço e por
sistema, a janela de tempo, quais técnicas ficam permitidas, quem é o contato dos dois
lados, e o que fazer se o teste derrubar um serviço. Sem isso, você não está fazendo
teste de segurança, está cometendo um crime com boa intenção.

Na oficina isso tem consequência prática: monte o laboratório em máquinas suas ou em
ambiente virtual isolado, e nunca aponte ferramenta para a rede da instituição.

## Avaliação e teste de invasão

Os dois termos se misturam e descrevem trabalhos diferentes.

A avaliação de vulnerabilidade busca amplitude. Ela varre tudo que está no escopo,
compara com bases de falhas conhecidas e produz uma lista de problemas com severidade. A
saída típica é um relatório longo com muitos itens, e ela não confirma se a falha é
explorável no seu contexto.

O teste de invasão busca profundidade. Ele parte de um objetivo, como alcançar o banco
de dados a partir da rede de visitantes, e persegue esse caminho até conseguir ou até
esgotar as opções. A saída é curta e concreta: este caminho funciona, e aqui está a
prova.

A escolha depende da pergunta. Para saber o que precisa de correção, avaliação. Para
saber se a sua defesa aguenta, teste de invasão.

Sobre o nível de informação entregue ao testador, há três arranjos. Sem informação
alguma, o teste simula o atacante externo e gasta tempo em reconhecimento. Com
informação completa, ele encontra mais em menos tempo e simula pior o mundo real. O
meio-termo, com credencial de usuário comum e diagrama de rede, costuma dar o melhor
retorno por hora.

## As fases e o que o defensor aprende com elas

Um teste segue reconhecimento, varredura, obtenção de acesso, manutenção do acesso e
remoção de rastro. Essa sequência descreve também o que um atacante real faz, e é por
isso que ela interessa a quem defende.

Cada fase deixa sinal. O reconhecimento aparece como consulta de DNS e coleta pública. A
varredura aparece como conexão a portas fechadas em sequência. A obtenção de acesso
aparece como falha de autenticação repetida. A manutenção aparece como conta nova ou
tarefa agendada. A remoção de rastro aparece como buraco no registro.

Ler a lista de fases como um roteiro de detecção é mais útil que decorá-la. Se você
configurou o registro remoto do capítulo 2, tem material para procurar cada um desses
sinais. Se não configurou, a fase cinco funciona.

## As categorias de teste

O material do curso separa por tipo, e vale conhecer os nomes. A varredura de rede
descobre o que existe e o que responde. A varredura de vulnerabilidade compara o que
respondeu com falhas conhecidas. A quebra de senha mede a resistência das credenciais. A
verificação de integridade compara arquivos com uma referência conhecida. A revisão de
registro procura no que já aconteceu. E o teste de integridade de configuração compara o
que está rodando com a linha de base aprovada.

Essa última merece ênfase, porque é a que fecha o ciclo do guia. Uma linha de base é a
configuração que você declarou correta. Comparar cada equipamento contra ela transforma
auditoria em rotina automatizável, e transforma desvio em alerta.

## Prática

Monte um laboratório isolado com o roteador dos capítulos anteriores, um switch e uma
máquina Linux. Nada aqui deve tocar rede de terceiros.

### Descobrir o que existe

```text
$ nmap -sn 10.20.0.0/24
```

A varredura sem porta apenas descobre hosts ativos. Ela é o primeiro passo e já revela
equipamento que ninguém sabia que estava ligado.

### Ver o que responde

```text
$ nmap -sS -p- 10.20.0.1
$ nmap -sV -O 10.20.0.1
```

O resultado é a sua conferência do capítulo 2. Se a porta 23 aparecer aberta, o Telnet
sobreviveu em algum lugar. Se a 80 aparecer, o servidor HTTP embutido continua ligado.
Se aparecer serviço que você não reconhece, você acabou de encontrar o motivo do
capítulo.

### Confirmar o que a filtragem faz

```text
$ nmap -sS -p 22,23,80,443 10.30.0.10
$ nmap -Pn --reason -p 80 10.30.0.10
```

O `--reason` mostra por que o nmap classificou a porta daquele jeito, e a diferença
entre resposta de recusa e ausência de resposta distingue porta fechada de porta
filtrada. Essa distinção é o teste real da sua lista de acesso.

### Demonstrar o capítulo 1 com captura

Abra o Wireshark, faça uma sessão Telnet contra um equipamento de teste sem
configuração, e siga o fluxo TCP. A senha aparece caractere a caractere. Repita com SSH
no equipamento configurado e mostre o mesmo fluxo ilegível.

Essa é a demonstração mais eficaz da oficina inteira, e não custa nada além de dois
minutos.

### Medir a resistência da credencial

Extraia de um laboratório seu o resumo de uma senha tipo 5 e o de uma tipo 9 e submeta
os dois à mesma tentativa de quebra com dicionário. A diferença de tempo entre eles é o
argumento do capítulo 1 transformado em número, e é o tipo de evidência que convence
quem decide orçamento.

### Auditar a configuração

```text
R1# show running-config | include ^line|transport|login|password|username
R1# show ip interface brief
R1# show archive config differences system:running-config nvram:startup-config
R1# show ip access-lists
R1# show crypto session
```

Compare o resultado com a sua linha de base. Divergência entre configuração em execução
e configuração salva costuma indicar mudança feita sem registro, e essa é uma das
primeiras coisas a procurar depois de um incidente.

## Laboratórios do curso

O módulo 22 não traz laboratório de Packet Tracer, e boa parte da prática acima exige
ferramenta real em máquina Linux, que o simulador não substitui. A avaliação prática do
módulo, o Skills Assessment, cobre a configuração dos capítulos anteriores em conjunto.

Este é o último capítulo do guia. Os oito juntos cobrem o ciclo completo: proteger o
equipamento, proteger o plano de controle, controlar quem administra, filtrar o tráfego,
detectar o que passou, entender a criptografia, aplicá-la em um túnel, e voltar ao
começo para verificar se tudo isso funciona de verdade.
