# Memória do Projeto: Bloco de Notas Markdown

Este documento serve como a fonte de verdade para o estado do projeto, arquitetura, dependências e fluxos. Ele deve ser atualizado e consultado a cada modificação ou leitura realizada no código.

---

## 1. Visão Geral
* **Nome**: Bloco de Notas Markdown (`bloco-de-notas`)
* **Descrição**: Bloco de notas offline focado em escrita/edição em Markdown, com sincronização nativa para dispositivos móveis usando Capacitor.
* **Plataformas**: Web (PWA) e Mobile (Android/iOS via Capacitor).

---

## 2. Tecnologias e Dependências
* **Core**: HTML5, Vanilla CSS3 (Custom Properties/Variaveis), Vanilla JavaScript.
* **Bibliotecas Externas (Incluídas Localmente em `www/`)**:
  * `marked.min.js`: Parser de Markdown para renderizar visualizações HTML a partir de texto cru.
  * `turndown.js`: Conversor de HTML para Markdown, permitindo edição fluida e conversão reversa.
* **Framework/Runtime Nativo**: Capacitor.
* **Dependências de Produção (`package.json`)**:
  * `@capacitor/android`: Suporte nativo ao Android.
  * `@capacitor/core`: Runtime core do Capacitor.
  * `@capacitor/filesystem`: Acesso ao sistema de arquivos nativo.
  * `@capawesome/capacitor-file-picker`: Seleção de arquivos nativos.
* **Dependências de Desenvolvimento**:
  * `@capacitor/cli`: Interface de linha de comando do Capacitor.
  * `live-server` / `http-server`: Servidores de desenvolvimento local.

---

## 3. Estrutura do Projeto
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
└── project_memory.md          # Esta memória do projeto
```

---

## 4. Arquitetura de Dados e Lógica (Baseada em `www/index.html`)

### 4.1 Armazenamento
* **Banco de Dados local**: IndexedDB (`dbName: 'mkd_notepad_db_v3'`).
  * Tabela `preferences`: Armazena configurações simples (chave-valor).
  * Tabela `notes`: Armazena as notas estruturadas com a chave única sendo o `id`.
* **Sincronização Nativa (Android/Capacitor)**:
  * Quando executado via Capacitor com o plugin Filesystem, ele sincroniza as notas entre a tabela `notes` do IndexedDB e o sistema de arquivos local (`mkd-notes/*.md`).

### 4.2 Lógica do Cabeçalho de Nota (Metadados Incorporados)
As notas armazenam metadados (como capa, ícone personalizado, status de fixado e altura da capa) em formato de comentários Markdown no topo do conteúdo cru:
* `<!--PINNED:true-->`: Nota fixada.
* `<!--ICON:valor-->`: Ícone personalizado (emoji, SVG ou URL de imagem).
* `<!--COVER_HEIGHT:valor-->`: Altura personalizada da capa.
* `![capa](url)`: Capa da nota.

A função `extractCoverAndContent()` divide o conteúdo cru nesses metadados, enquanto `packCoverAndContent()` remonta a string da nota para salvar.

### 4.3 Fluxos de Visualização
* **Dashboard View**: Exibe notas recentes em formato de cartões (cards), com barra de rolagem horizontal, além de atalhos e visualizações gerais.
* **Editor View**:
  * **Modo Read (Leitura)**: Renderiza o HTML com o `marked`.
  * **Modo Edit (Edição)**:
    * **Visual**: Edição interativa (se implementada/suportada).
    * **Raw/Markdown**: Edição do texto cru (com suporte a autocomplete e fontes mono).
* **Limpeza Automática**: A função `cleanupEmptyNotes()` remove notas sem título/conteúdo criado acidentalmente ao navegar fora delas.

### 4.4 Sistema de Tags (Hashtags)
* **Estrutura**: Tags são definidas diretamente no texto da nota utilizando a sintaxe `#nomedatag`. A detecção é baseada em regex que valida `#` seguido de caracteres alfanuméricos e descarta cabeçalhos Markdown normais (ex: `# Cabeçalho`).
* **Funções Globais de Tags (Pré-prontas)**:
  * `window.extractNoteTags(content)`: Varre o conteúdo textual de uma nota e retorna um array de tags exclusivas (em minúsculas).
  * `window.getAllUniqueTags()`: Analisa todas as notas em `NoteStorage.notes` e retorna uma lista consolidada de tags do banco.
  * `window.getNotesByTag(tag)`: Filtra e retorna a coleção de notas que contêm a tag indicada.
* **Renderização Dinâmica**: A função `processHashtagsInDOM(container)` varre o DOM do container do modo leitura (`readPane`), identificando nós de texto com hashtags e transformando-os em spans com a classe `.rendered-tag` para exibição como pílulas visuais. Elementos como links, tags de código (`<code>`, `<pre>`) e textareas são ignorados no mapeamento para preservar o código-fonte original.

---

## 5. Regras de Desenvolvimento (Restrições Importantes)
1. **Consistência de Contexto**: Sempre use esta memória (`project_memory.md`) para obter o contexto do projeto antes de propor mudanças.
2. **Escopo Estrito**: Nunca realize alterações não solicitadas ou fora do escopo do pedido do usuário.
3. **Comunicação de Mudanças**: Antes de executar qualquer alteração, envie uma descrição detalhada das mudanças propostas.
4. **Verificação de Segurança**: Teste a sintaxe e a lógica da aplicação antes de salvar/aplicar alterações. Se quebrar a lógica, corrija imediatamente.
5. **Atualização da Memória**: Sempre atualize a memória do projeto após validações e testes bem-sucedidos.
