# alcantarajulio.github.io

Blog em Hugo (tema terminal, vendorizado em `themes/terminal/`), bilíngue PT-BR/EN,
publicado no GitHub Pages pelo workflow em `.github/workflows/`. O CI fixa o Hugo em
0.151.0; a versão local pode ser mais nova e emitir avisos de depreciação que o CI não
emite.

## Escrita

Todo texto novo ou editado em `content/` passa pelas skills `stop-slop` e `humanizer`
antes de ser commitado. As duas se complementam: stop-slop corta o excesso mecânico
(travessões, negrito decorativo, aberturas de encheção), humanizer pega os padrões de
prosa de IA (importância inflada, gerúndio de análise rasa, fontes vagas, tríades
forçadas, finais otimistas genéricos).

Regras que valem para os dois idiomas:

- Sem travessão (— ou –), sem aspas curvas, sem emoji.
- Voz ativa, sujeito explícito. `is`/`é` em vez de `serves as`/`serve como`.
- Cabeçalhos em caixa de sentença.
- Nada de afirmação factual inventada. Comando de IOS, número de laboratório e nome de
  framework vêm do material do curso ou não entram.
- Toda contagem declarada no texto ("quatro objetivos", "cinco passos") tem que bater
  com a enumeração que vem depois, e com a versão no outro idioma.

## Conteúdo

- Cada página tem par nos dois idiomas, ligado por `translationKey`. Os capítulos usam
  `chapter`, `weight` e `date` no front matter.
- Material da Cisco Networking Academy e da Coventry University é licenciado e não pode
  ser reproduzido aqui. Os guias apontam para os laboratórios pelo número, sem copiar o
  arquivo.

## Hugo

Depois de criar diretório novo em `content/`, reinicie o servidor com
`hugo server --disableFastRender`. O Fast Render Mode não adiciona diretórios criados
depois da inicialização à lista de observação e serve conteúdo velho sem avisar.
