---
title: "Gestão de risco em TI"
description: "Identificar, medir e responder a risco sem transformar o registro em um documento que ninguém lê."
date: 2026-08-19
chapter: 1
translationKey: "psf-603-01"
weight: 10
tags: ["risco", "governança", "ti"]
---

Toda decisão de TI é uma aposta sobre o futuro. Você escolhe um fornecedor sem saber se
ele existirá em cinco anos, migra um sistema sem saber quanto tempo a migração vai
levar, e adia uma atualização apostando que a falha não será explorada antes da próxima
janela. Gestão de risco é o método que transforma essas apostas implícitas em decisões
declaradas.

## Risco não é problema

A distinção parece formalidade e muda o vocabulário da reunião. Um problema já
aconteceu e exige resposta. Um risco pode acontecer, e exige decisão antes.

Um risco tem três componentes: o evento, a probabilidade de ele ocorrer, e o impacto se
ocorrer. Descrever risco sem os três produz frases inúteis do tipo "risco de segurança",
que ninguém consegue priorizar nem orçar.

A forma que funciona nomeia causa, evento e consequência. Em vez de "risco de queda do
servidor", escreva "por causa da fonte única de alimentação no rack, o servidor de
faturamento pode ficar indisponível, o que interromperia a emissão de notas por até seis
horas". A segunda versão já contém a resposta dentro dela.

## Medir para poder comparar

A matriz de probabilidade e impacto é o instrumento mais usado, e o mais mal usado.
Você posiciona cada risco em uma grade, o que permite comparar itens que não têm nada em
comum, e produz uma ordem de atenção.

A armadilha está na escala. Quando "alto", "médio" e "baixo" ficam sem definição, cada
pessoa da equipe usa a sua régua, e o resultado agrega opiniões incompatíveis. A correção
é ancorar cada nível em número: impacto alto significa perda acima de tanto, ou parada
acima de tantas horas. Essa ancoragem transforma discussão de sensibilidade em discussão
de critério.

A avaliação qualitativa serve para triagem rápida e para risco difícil de quantificar. A
quantitativa, que multiplica probabilidade por perda esperada, serve quando existe
histórico e quando a decisão precisa de justificativa financeira. As duas convivem: você
tria com a primeira e aprofunda a segunda só no topo da lista.

## As quatro respostas

Depois de medir, existem quatro caminhos, e escolher um deles é obrigatório. Não escolher
é aceitar por omissão.

Evitar significa não fazer a atividade que gera o risco. Cancelar a integração com o
fornecedor duvidoso elimina o risco e elimina também o benefício que ela traria.

Transferir move a consequência financeira para outra parte, por seguro ou por contrato
com cláusula de nível de serviço. Note que a transferência move o dinheiro e não move a
responsabilidade: se o sistema cai, seus clientes reclamam com você.

Mitigar reduz a probabilidade, o impacto, ou os dois. A maior parte do trabalho de
segurança vive aqui.

Aceitar é a decisão de conviver, e é legítima quando o custo do controle supera a perda
esperada. Aceitação precisa ser registrada com nome e data, porque aceitação anônima vira
negligência quando o evento acontece.

## O registro de risco e por que ele morre

O registro é a lista viva de riscos com dono, avaliação, resposta escolhida e prazo. Ele
falha de duas formas previsíveis.

A primeira é o registro sem dono. Risco atribuído à "equipe de TI" não é atribuído a
ninguém, e a revisão trimestral encontra o mesmo item aberto por dois anos.

A segunda é o registro que ninguém revisa. Risco tem validade: o que era improvável no
ano passado pode ter virado rotina, e o controle que você implantou pode ter deixado de
funcionar. Um registro sem cadência de revisão documenta o passado.

## O que os desastres ensinam

A literatura de gestão de risco em TI é construída sobre fracassos caros, e vale estudar
os casos porque eles repetem padrões.

O padrão mais comum é o do risco conhecido e não escalado. Alguém na equipe sabia, o
alerta subiu um nível e parou, e a decisão foi tomada por quem não tinha a informação. O
segundo é o do otimismo em cadeia: cada camada da hierarquia arredonda a estimativa para
baixo, e o prazo final não tem relação com o trabalho.

O terceiro aparece em projeto público de grande porte, e é o do escopo que cresce sem
que o orçamento cresça junto. Ele produz sistemas entregues anos depois, com uma fração
da função prometida, e é o motivo pelo qual essa disciplina existe como matéria separada.

## Na prática

Escolha um sistema que você conheça bem, de preferência um que você mesmo opere.

Primeiro, liste cinco riscos usando a forma de causa, evento e consequência. A restrição
do formato é o exercício, porque escrever assim obriga a saber do que você está falando.

Segundo, defina a sua escala antes de avaliar. Escreva o que significa impacto alto,
médio e baixo em número, para o seu contexto. Faça o mesmo com probabilidade.

Terceiro, posicione os cinco na matriz e ordene.

Quarto, escolha uma das quatro respostas para cada um, com nome do dono e prazo. Se você
escolher aceitar, escreva a justificativa: qual o custo do controle e qual a perda
esperada.

Quinto, e é o passo que separa exercício de prática, marque uma data de revisão e
cumpra. Um registro revisado duas vezes vale mais que um registro perfeito revisado
nunca.

## Para aprofundar

O material do módulo trabalha com casos de falha em projetos públicos de tecnologia e
com a análise de risco em engenharia de alta consequência. Os dois recortes servem ao
mesmo propósito: mostrar que o risco raramente é desconhecido, e que a falha mora na
comunicação entre quem sabe e quem decide.

O próximo capítulo sai da defesa e vai para a pergunta de onde a tecnologia gera
vantagem.
