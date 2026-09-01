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
└── PROJECT_MEMORY.md          # Memória única do projeto
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
* Consolidação da memória do projeto em arquivo único:
  * Fusão do conteúdo de `project_memory.md` no arquivo primário `PROJECT_MEMORY.md` e remoção do arquivo duplicado.
* Retorno da funcionalidade "Fixar Nota" para o menu de 3 pontinhos (`#note-menu-dropdown`):
  * Reintegração do botão `#menu-pin-note-btn` com o texto dinâmico "Fixar Nota" / "Desafixar Nota" no menu de opções da nota.
  * Remoção do botão inline `.pin-note-btn` do container de ações de tags (`.tags-actions-wrapper`).
  * Atualização da lógica de clique e sincronização do estado `pinned` com o armazenamento local e re-renderização do Dashboard e lista de notas.
* Remoção da barra de tags mantendo funcionalidade interna:
  * Remoção dos contêineres visuais da barra de tags (`.note-tags-wrapper`) dos painéis de visualização/edição do HTML.
  * Preservação total dos metadados de tags (`<!--TAGS:...-->`), estrutura de dados (`note.tags`) e métodos globais de busca/extração (`window.extractNoteTags`, `window.getAllUniqueTags`, `window.getNotesByTag`).
* Implementação do gesto de deslize (swipe) para a barra lateral:
  * Suporte ao gesto de arrastar/deslizar o dedo da esquerda para a direita (`deltaX > 60px`) em qualquer ponto da tela para abrir a barra lateral (`#sidebar`).
  * Suporte ao gesto de deslizar da direita para a esquerda quando a barra lateral estiver aberta para fechá-la.
  * Proteção contra ativação acidental durante rolagens verticais ou com modais ativos.
* Acionamento da edição por clique único na área de texto:
  * Configuração do container de leitura (`#read-pane-wrapper`) para abrir o editor no modo preferido (`raw` ou `visual`) ao dar um clique simples em qualquer lugar do campo de texto da nota.
  * Preservação da navegação ao clicar em links internos/externos no texto.
* Posicionamento preciso do cursor ao clicar:
  * Adição das funções `getCharOffsetFromPoint` e `setCaretAtCharOffset` para calcular o deslocamento exato do caractere no ponto de clique (`caretRangeFromPoint` / `caretPositionFromPoint`).
  * No modo visual (`#edit-visual-pane`), o cursor de edição é definido no nó e offset exatos. No modo raw (`#raw-pane`), `setSelectionRange` posiciona a seleção na mesma posição de caractere.
* Finalização automática da edição ao recolher o teclado virtual:
  * Monitoramento de eventos de desfoque (`blur` / `focusout`) nos elementos editáveis (`rawPane`, `editVisualPane`, `noteTitleInput`).
  * Monitoramento de redimensionamento do `window.visualViewport` para detectar o recolhimento do teclado em dispositivos móveis/PWAs, salvando a nota e retornando ao modo leitura (`setMode('read')`).
* Salvamento automático ao pressionar o botão Voltar (Voltar Físico/Gesto/Escape):
  * Integração com o histórico de navegação (`history.pushState` ao entrar na edição) e com eventos de popstate (`window.addEventListener('popstate')`), teclado (`keydown` Escape/GoBack) e plugin nativo do Capacitor (`Capacitor.Plugins.App.addListener('backButton')`).
  * Ao pressionar Voltar enquanto o teclado se oculta, a função `finishEditing()` salva imediatamente as alterações da nota e retorna a interface ao modo de leitura.
* Bloqueio do gesto de barra lateral durante o dimensionamento da capa e remoção total do cursor:
  * O gesto de deslize para abrir a sidebar é desativado durante a interação com a capa (`.note-cover-box`), sliders (`input[type="range"]`), modais ou gestos com 2 ou mais dedos.
  * A função `finishEditing()` executa `blur()` em todos os elementos e limpa todas as seleções de texto (`window.getSelection().removeAllRanges()`), garantindo que o cursor de escrita desapareça por completo ao esconder o teclado por qualquer ação.
* Padronização de Placeholders Esmaecidos e Cursor na Posição 1 (Corpo e Título):
  * As mensagens "Comece a digitar sua primeira nota aqui..." e "Título da nota" foram configuradas como placeholders CSS nativos esmaecidos (`color: rgba(255, 255, 255, 0.35)`).
  * Ao clicar no título ou corpo da nota quando vazios, o cursor é posicionado automaticamente na Posição 1 (índice 0) sobre o texto esmaecido.
  * Assim que o usuário digita o primeiro caractere, a mensagem esmaecida desaparece imediatamente.
* Contenção de Overscroll e Espaçamento Inferior Padrão de Indústria:
  * Aplicação de `overscroll-behavior: none` / `contain` em `html`, `body`, `.view-pane`, `#edit-visual-pane` e `#raw-pane`, impedindo o arrasto/deslocamento elástico da nota para fora da tela durante a escrita.
  * Adição de `padding-bottom: 120px` nas áreas de escrita do editor, garantindo o espaçamento confortável padrão de mercado após o final do texto acima do teclado virtual e barras de ferramentas.
* Eliminação de Barras Duplas de Rolagem e Deslocamento Lateral ao Abrir o Teclado:
  * Aplicação de `overflow-x: hidden !important; max-width: 100% !important;` no editor para impossibilitar o deslocamento lateral do texto para fora da tela.
  * Reformulação da função `updateFormatToolbarPosition()` para manter o contêiner do editor (`editorPanesWrapper`) em altura total sem encolhimento forçado de margem, unificando a rolagem vertical exclusivamente no `.view-pane` e eliminando barras de rolagem internas indesejadas.
* Trava Rígida de Deslocamento Horizontal 0px (`scrollLeft = 0` e Strict Overscroll):
  * Adição de ouvinte JS em todos os `.view-pane` para resetar `scrollLeft = 0` instantaneamente a qualquer tentativa de deslize horizontal.
  * Aplicação de `overscroll-behavior-x: none !important` e `touch-action: pan-y !important` estritos em `html`, `body`, `.view-pane` e editores, erradicando qualquer oscilação ou deslocamento lateral do texto.
* Aba Lateral Direita e Remoção do Menu de 3 Pontinhos do Cabeçalho Superior:
  * Remoção do botão de 3 pontinhos (`#note-menu-btn`) da barra superior (`.app-header`) e substituição do antigo dropdown por uma nova Aba Lateral Direita (`#right-sidebar`).
  * A aba lateral direita é acionada deslizando o dedo da direita para a esquerda em qualquer lugar da tela (`deltaX < -60px`), exibindo o painel "Opções da Nota" com ações de Capa, Ícone, Fixar/Desafixar Nota e Excluir.
  * O fecho pode ser acionado deslizando da esquerda para a direita ou clicando no overlay / botão de fechar (×).
* Remoção dos Botões Editar e Salvar da Barra Superior:
  * Os botões `#edit-btn` e `#save-btn` foram removidos da barra superior (`.app-header`), deixando o topo totalmente limpo.
  * A navegação para edição continua operando com 1 clique direto na área de texto e o auto-salvamento funciona em tempo real ao ocultar o teclado ou navegar.
* Transferência do Título para Dentro do Campo de Texto (Estilo Notion / Obsidian):
  * O container `.header-title-container` foi removido da barra superior (`.app-header`).
  * O título foi posicionado diretamente no topo da área do documento (`.note-title-content-box`), localizado abaixo da capa e imediatamente acima do corpo do texto.
  * No modo de leitura, o título é exibido em destaque com tipografia de 28px bold. Nos modos de edição (Visual e Raw), o input do título (`#note-title-input` / `#note-title-input-raw`) fica integrado no topo do editor de texto com sincronização bidirecional em tempo real.
* Remoção da Barra Superior e Transição para Navegação 100% via Gestos:
  * A barra superior (`.app-header`) e o botão hambúrguer (`#menu-btn`) foram completamente removidos do HTML/CSS, criando uma área de trabalho em tela cheia (full screen canvas).
  * O espaçamento superior do editor (`.view-pane`) foi ajustado para `calc(16px + env(safe-area-inset-top))` para aproveitar todo o topo da tela.
  * O acesso às barras laterais ocorre 100% via gestos: deslize da esquerda para a direita abre a lista de notas; deslize da direita para a esquerda abre o menu de opções da nota.
* Correção da Exibição Imediata do Título em Modo Leitura:
  * Adicionadas verificações nulas (`if (editBtn)`, `if (saveBtn)`) na função `setMode()`, eliminando exceções que travavam a renderização no modo leitura.
  * Alteração da classe CSS `.note-title-display-container` de `display: none;` para `display: block !important;`, garantindo a visibilidade imediata e permanente do título na área do documento durante a leitura.
  * Adicionado ouvinte de clique no título em modo leitura para alternar automaticamente para edição e focar o input.
* Correção da Exibição do Dashboard (Início) e Eliminação de IDs Duplicados:
  * Remoção do bloco de HTML duplicado de `.editor-container` presente no arquivo, eliminando conflitos de IDs no DOM (`#read-pane`, `#edit-visual-pane`, `#raw-pane`).
  * Simplificação de `openDashboardView()` e `openEditorView()` para alternar de forma limpa a visibilidade entre o container do editor e o painel do dashboard (`#dashboard-pane`).
  * Inserção do cabeçalho visual `.dashboard-header` no HTML do `#dashboard-pane`, com o título exibindo o nome do aplicativo **"Verbose"** (sem botões ou subtítulos adicionais para manter a interface limpa).
  * Ajuste de `padding-top: calc(24px + env(safe-area-inset-top, 0px))` no `#dashboard-pane` para exibição perfeita sem a antiga barra superior.
  * Aprimoramento de `renderDashboard()` para renderizar cartões horizontais de notas recentes e, na lista vertical, exibir as notas fixadas ou todas as notas caso não haja fixadas, garantindo que o Dashboard nunca fique vazio ou como tela preta.
* Novo Ícone do Aplicativo e Identidade Visual (Verbose):
  * Importação e processamento da imagem oficial `/sdcard/Download/Verbose.png`.
  * Geração com reamostragem de alta qualidade (Lanczos) de todos os ícones web (`www/icon-192.png`, `www/icon-512.png`) e ícones nativos Android (`ic_launcher`, `ic_launcher_round`, `ic_launcher_foreground` em `mipmap-mdpi`, `hdpi`, `xhdpi`, `xxhdpi`, `xxxhdpi`).
  * Atualização do nome da aplicação para **"Verbose"** no `manifest.json`, `index.html`, `capacitor.config.json`, banner de instalação PWA e `strings.xml`.
  * Incremento do Service Worker para a versão `mkd-notes-v11` para invalidação e renovação imediata de cache e manifesto PWA.
* Resposta Imediata em 1 Clique na Barra Lateral:
  * Invocação de `finishEditing()` síncrona e desfoque de todos os campos de texto/teclado virtual no exato momento da abertura das barras laterais esquerda e direita.
  * Aplicação de `touch-action: manipulation;` e `-webkit-tap-highlight-color: transparent;` nos botões da barra lateral (`.sidebar-action-btn`, `.note-item`), eliminando atrasos de toque mobile e garantindo acionamento instantâneo no 1º clique.

## 5. Regras de Desenvolvimento (Restrições Importantes)
1. **Consistência de Contexto**: Sempre use esta memória (`PROJECT_MEMORY.md`) para obter o contexto do projeto antes de propor mudanças.
2. **Escopo Estrito**: Nunca realize alterações não solicitadas ou fora do escopo do pedido do usuário.
3. **Comunicação de Mudanças**: Antes de executar qualquer alteração, envie uma descrição detalhada das mudanças propostas.
4. **Verificação de Segurança**: Teste a sintaxe e a lógica da aplicação antes de salvar/aplicar alterações. Se quebrar a lógica, corrija imediatamente.
5. **Atualização da Memória**: Sempre atualize a memória do projeto após validações e testes bem-sucedidos.





