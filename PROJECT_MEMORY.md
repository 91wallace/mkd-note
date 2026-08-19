# MEMÓRIA DO PROJETO

## 1. Visão Geral e Objetivo
* **Nome**: Bloco de Notas Markdown (`bloco-de-notas`)
* **Descrição**: Bloco de notas offline focado em escrita/edição em Markdown, com sincronização nativa para dispositivos móveis usando Capacitor.
* **Plataformas**: Web (PWA) e Mobile (Android/iOS via Capacitor).

## 2. Estrutura de Arquivos e Componentes
```text
meu-bloco-de-notas/
├── android/                   # Código nativo Android gerado pelo Capacitor
├── www/                       # Pasta de distribuição Web (arquivos estáticos)
│   ├── index.html             # Interface principal contendo HTML, CSS e Lógica JS
│   ├── marked.min.js          # Parser Markdown (local/offline)
│   ├── turndown.js            # Conversor HTML para Markdown (local/offline)
│   ├── sw.js                  # Service Worker para suporte PWA/Cache offline
│   ├── manifest.json          # Manifesto do PWA
│   ├── capacitor.js           # Script de ponte do Capacitor (gerado/injetado)
│   └── icon-192.png / icon-512.png # Ícones do app
├── capacitor.config.json      # Configurações do Capacitor (App ID, Nome, Pasta Web)
├── package.json               # Dependências, metadados e scripts npm
└── PROJECT_MEMORY.md          # Esta memória do projeto
```

## 3. Estado Atual da Aplicação (HTML/CSS/JS)
* **Banco de Dados local**: IndexedDB (`dbName: 'mkd_notepad_db_v3'`).
  * Tabela `preferences`: Armazena configurações simples (chave-valor).
  * Tabela `notes`: Armazena as notas estruturadas com a chave única sendo o `id`.
* **Sincronização Nativa (Android/Capacitor)**:
  * Sincroniza as notas entre IndexedDB e o sistema de arquivos local (`mkd-notes/*.md`).
* **Lógica do Cabeçalho de Nota (Metadados)**:
  * Armazena capa (`![capa](url)`), status de fixado (`<!--PINNED:true-->`), ícone personalizado (`<!--ICON:valor-->`), altura da capa (`<!--COVER_HEIGHT:valor-->`) e transformações da capa (`<!--COVER_TRANSFORM:translate(...) scale(...)-->`).
* **Sistema de Tags**:
  * Tags baseadas em hashtags `#nomedatag` no corpo do texto.
  * Renderização dinâmica de hashtags em spans com a classe `.rendered-tag` no modo de leitura.

## 4. Histórico de Modificações
* Inicialização da memória do projeto unificada em `PROJECT_MEMORY.md` com base nas especificações.
* Correção da funcionalidade de capa:
  * Ajuste do fluxo de abertura do modal (`openCoverModal`) para definir `display: flex` antes de chamar `updateCoverPreview`, evitando dimensões zeradas no cálculo de gestos e escalas.
  * Proteção contra `NaN` nos cálculos de escala e translação no preview e visualizadores.
  * Integração nativa do Capacitor FilePicker para seleção de imagens em dispositivos móveis.
  * Remoção segura de capas de texto muito longos (base64) no extrator de metadados para evitar erros de RegExp.
* Alteração do design do botão hambúrguer (`#menu-btn`):
  * Remoção de fundo, bordas e sombras para igualar ao design do botão de 3 pontinhos (`#note-menu-btn`).
  * Aumento do tamanho do botão e do ícone (para `26px`), herdando o comportamento de escala e transição no hover/active.
* Ajuste na hierarquia tipográfica:
  * Redução do tamanho dos cabeçalhos `h1` (de `2.2em` para `1.55em`), `h2` (de `1.65em` para `1.35em`) e `h3` (de `1.3em` para `1.2em`), e introdução do `h4` (`1.1em`) para atenuar o contraste de escala em relação ao texto de corpo mantendo a distinção.
* Correção na restauração da configuração da capa ao recarregar a aplicação:
  * Ajuste de `extractCoverAndContent` para inicializar propriedades como `null` ao invés de valores padrão vazios/estáticos.
  * Correção de `loadNotesLS` para restaurar e preservar as propriedades `coverHeight`, `coverTransform` e `pinned` a partir do IndexedDB caso não estejam presentes como comentários no markdown bruto do banco.
* Implementação da herança de tamanho de capa em novas notas:
  * Ajuste do método `handleNewNote` para herdar as propriedades `coverHeight` e `coverTransform` da última nota ativa, mantendo cada nota já salva isolada com sua própria proporção/transformação original.
* Reformulação da funcionalidade de tags:
  * Remoção da extração automática de hashtags baseada no símbolo `#` no corpo do texto (desativação de `processHashtagsInDOM`).
  * Implementação de uma barra de tags (`.note-tags-bar`) horizontal com altura fixa de `38px` logo abaixo da capa ou do título, evitando deslocamento de componentes na tela.
  * Criação do modal de gerenciamento de tags (`#tag-manager-modal`) acionado por um botão com ícone de etiqueta ao final da barra, permitindo a criação de novas tags e seleção interativa das existentes.
  * Armazenamento e sincronização explícita das tags de forma estruturada nas notas como metadados (`<!--TAGS:...-->`).
* Ajuste visual na barra de tags:
  * Substituição do ícone emoji `🏷️` por um ícone SVG estruturado tracejado/outline (tipo bandeira/bookmark) no botão `.add-tag-btn`.
  * Remoção da linha separadora inferior (`border-bottom`) da barra de tags `.note-tags-bar`.
  * Redução no espaçamento entre capa, tags e o início do texto: alterado `margin-bottom` da capa `.note-cover-box` de `24px` para `8px` e `margin-bottom` da barra `.note-tags-bar` de `12px` para `6px`.
  * Padronização de todos os espaçamentos da barra de tags `.note-tags-bar`: `padding: 0` (alinhamento lateral perfeito com a escrita), `margin-top: 4px`, `margin-bottom: 8px` e `margin-left/right: 0`.
* Ajuste no dimensionamento da capa da nota:
  * Redução da altura padrão da capa de `180px` para `160px` no CSS ([`.note-cover-box`](file:///root/project/mkd-note/www/index.html#L263)) e atualização de todos os fallbacks do motor Javascript para usar `160` ao invés de `200` ao criar ou restaurar notas sem configuração explícita de altura.
* Implementação do colapso e ajustes na barra de tags:
  * Criação do container `.note-tags-wrapper` com suporte à classe `.collapsed` para recolher a barra com transição suave (`height: 0`, `opacity: 0`, `overflow: hidden`).
  * Adicionado botão de colapso/expansão `.tags-toggle-btn` com ícone de seta (cima/baixo) que persiste seu estado global via `localStore`.
  * Aumento da fonte do texto das tags `.rendered-tag` na barra para `14px`.
  * Redução adicional do espaçamento inferior da barra de tags para `2px` (quando expandida).
* Reformulação total da adição e remoção de tags para modo inline:
  * Remoção do modal de gerenciamento de tags anterior.
  * Inclusão de um input de texto inline [`.inline-tag-input`](file:///root/project/mkd-note/www/index.html#L1439) ativado ao clicar no botão de adicionar tag inline.
  * O botão de adicionar tag `.add-tag-btn` foi movido para o final da lista de tags (deixando apenas o ícone tracejado de bandeira, sem o texto "Tag") e o placeholder estático "+tag" foi removido. Clicar no botão oculta-o e exibe o input de texto.
  * O botão de adicionar tag agora possui a altura exata das tags em pixels (`height: 26px` e `box-sizing: border-box`), alinhando-se perfeitamente.
  * Aumento da fonte do texto das tags `.rendered-tag` para `16px` e ajuste fino do padding interno para manter a proporção na barra de `38px`.
  * Implementação de dropdown de autocompletar dinâmico [`.tag-inline-autocomplete`](file:///root/project/mkd-note/www/index.html#L1414) que exibe tags existentes filtradas logo abaixo do input enquanto o usuário digita.
  * Adição de suporte para 10 esquemas de cores HSL/RGB distintas atribuídas automaticamente com base no hash do nome de cada tag.
  * Inclusão de botão `×` de remoção imediata dentro de cada pílula de tag `.rendered-tag` para exclusão sem sair do editor.
* Relocalização do botão de fixar nota:
  * Remoção da opção "Fixar Nota" do menu dropdown de 3 pontinhos.
  * Adicionado botão de fixar nota [`.pin-note-btn`](file:///root/project/mkd-note/www/index.html#L1453) ao lado do botão colapsar, agrupados no container [`.tags-actions-wrapper`](file:///root/project/mkd-note/www/index.html#L1435).
  * O botão de fixar nota utiliza o ícone de alfinete diagonal outline (Bootstrap pin-angle), mantendo-se limpo, sem bordas e sem fundo. Quando a nota é fixada, a classe `.active` oculta a versão outline e ativa o preenchimento total do vetor (pin-angle-fill), além de mudar a cor para a cor de destaque.
  * O botão de fixar nota colapsa e some suavemente (`width: 0`, `opacity: 0`, `overflow: hidden`) junto com a barra de tags quando a função de colapso é acionada.
* Atualização do ícone de colapso de tags:
  * Substituição do ícone de seta anterior por um ícone de triângulo outline (Bootstrap caret-up) no botão `.tags-toggle-btn`. O ícone rotaciona 180 graus ao colapsar para apontar para baixo.
* Configuração de quebra de linha automática para tags:
  * Remoção do scroll horizontal e de sua barra oculta no container `.note-tags-list`.
  * Adicionado suporte a quebra automática de linha (`flex-wrap: wrap`) na lista de tags para que ocupem mais de uma linha se necessário.
  * Ajuste do container `.note-tags-bar` para altura flexível (`min-height: 38px`, `height: auto`) e correção das regras de colapso para recolher a barra flexível perfeitamente para `height: 0` e `padding: 0`.
* Ajuste de cores e tamanho dos botões de ação:
  * Aumentado o tamanho do ícone de triângulo no botão `.tags-toggle-btn` de `16px` para `20px` e substituído pela versão preenchida (Bootstrap caret-up-fill).
  * Ajustada a cor padrão dos botões `.tags-toggle-btn` e `.pin-note-btn` para `var(--text-primary)` (cor branca padrão do tema). Removidas quaisquer alterações de cor por estado (como hover ou ativo), mantendo-os permanentemente com a mesma tonalidade branca para uniformidade estética.
* Prevenção de sobreposição de texto com botão colapsado:
  * Refatorada a regra de colapso: ao invés de aplicar preenchimento lateral em todo o corpo de texto, configuramos a classe `.note-tags-wrapper.collapsed` para se comportar como um elemento flutuante à direita (`float: right; width: 26px; height: 26px; margin-left: 10px;`).
  * O botão de expansão passa a ser posicionado de forma estática dentro desse elemento flutuante, forçando a quebra de linha natural da primeira linha do texto ao redor dele (recuo à direita), enquanto todas as demais linhas fluem normalmente ocupando a largura total (mesmo alinhamento lateral esquerdo e direito).
  * No editor de código raw (`#raw-pane`), mantemos o recuo lateral à direita fixado de `32px` via CSS para a área do textarea.
  * O contêiner `.note-tags-wrapper` foi configurado com `max-width: 800px` e centralizado (`margin: 0 auto`), alinhando-se perfeitamente às margens do texto. Quando colapsado, o contêiner zera seu tamanho e o botão de expansão `.tags-actions-wrapper` é flutuado à direita (`float: right`), de modo que o recuo na quebra de linha do texto ocorra exatamente na margem direita do texto, sem margens excessivas.
  * Remoção do `overflow: hidden` nos contêineres `#read-pane` e `#edit-visual-pane` para evitar que criem um Novo Contexto de Formatação de Bloco (BFC), o que empurrava o bloco de texto inteiro para a esquerda em vez de apenas fazer a primeira linha fluir naturalmente ao redor do botão flutuante. Margens e alinhamentos de todas as linhas foram restaurados.
  * Remoção completa da regra CSS `.note-tags-wrapper.collapsed + #raw-pane` que adicionava um recuo lateral de `32px` à direita no editor de código raw. Toda e qualquer margem excessiva adicionada anteriormente ao colapsar foi eliminada.




