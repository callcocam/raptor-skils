---
name: laravel-raptor-page-layouts
description: >
  Gera componentes reutilizáveis de página (ListPage, FormPage) e composables TypeScript
  para projetos Laravel + Inertia + Vue 3 + Tailwind, partindo obrigatoriamente de um
  tema base (pasta com design-system ou documento de tema). Não depende de pacotes
  callcocam — funciona com Laravel puro (versão 12 ou 13). Use esta skill SEMPRE que
  o usuário pedir para: criar componentes de página de listagem, criar componentes de
  formulário, extrair padrões de páginas Vue, criar layouts base de lista ou formulário,
  criar composables de filtros/seleção/paginação/dirty-check, ou quando mencionar
  "ListPage", "FormPage", "layout de lista", "layout de formulário", "componentes de
  página", "sublayout", "page layout". Também acione quando o usuário quiser
  "componentizar páginas", "criar base de listagem", "criar base de formulário",
  "extrair o que é comum nas páginas", "padrão de lista e form", "gerar page layouts",
  "criar estrutura de páginas", "partir do tema base", "construir do zero com tema",
  "design-system como base", "tema como fundação". Entrada obrigatória: pasta de
  design-system OU documento de tema colado. Entrada opcional: código Vue/HTML ou
  screenshot como referência visual de estilo. Saída: arquivos .vue, composables .ts e
  documentação markdown em docs/. Stack: Laravel 12/13 + Vue 3 + Composition API +
  TypeScript + TailwindCSS v4 + Inertia.js. Todos os textos hardcoded em PT-BR.
  Zero dependências de pacotes callcocam/*. O layout padrão do Laravel deve ser
  substituído por completo pela estrutura e tokens do tema indicado (sem manter shell
  visual padrão anterior). Quando solicitado, também define padrão de backend
  (rotas + controllers + convenção de páginas Inertia) para acesso organizado.
---

# Laravel Raptor — Vue Page Layouts

## Fase 0 — Verificar estado anterior

**ANTES de qualquer análise**, verificar se layouts de página já foram extraídos:

```bash
ls resources/js/components/page/ListPage.vue \
   resources/js/components/page/FormPage.vue 2>/dev/null \
  && echo "Layouts anteriores encontrados" || echo "Nenhum layout de página anterior"
```

| Resultado | Ação |
|-----------|------|
| `ListPage.vue` e/ou `FormPage.vue` encontrados | Perguntar ao usuário |
| Não encontrado | Prosseguir normalmente |

**Se encontrar layouts anteriores:**

> "Encontrei layouts de página já extraídos neste projeto. O que deseja fazer?"
> - **Atualizar** — ler o novo tema e apenas complementar/corrigir os layouts existentes
> - **Recriar** — gerar todos os componentes do zero (sobrescreve os existentes)
> - **Sem limitação** — gerar tudo novamente sem considerar o que já existe

---

## ⚠️ PRIMEIRA AÇÃO — Obrigatória antes de qualquer outra coisa

**NÃO analise, NÃO gere código, NÃO liste padrões** até ter todas as respostas abaixo.
Esta é a **primeira mensagem** que deve ser enviada ao receber a skill:

```
Antes de começar, preciso de algumas informações:

1. Qual é a base de tema do projeto? (obrigatório — escolha UMA opção)
   a) Caminho para a pasta do projeto
      (ex: resources/js/, resources/, .)
      Vou ler o app.css para encontrar os tokens de tema e listar os
      componentes de UI disponíveis.
   b) Cole aqui um documento com as instruções de tema
      (tokens de cor, tipografia, espaçamentos, convenções visuais)

2. Onde devo salvar os componentes gerados?
   (sugestão: resources/js/components/page/)

3. Onde devo salvar os composables?
   (sugestão: resources/js/composables/)

4. (Opcional) Você tem algum código Vue/HTML ou screenshot de referência
   visual para guiar o estilo dos componentes? Cole o código ou informe
   o caminho da imagem.
```

**Regras:**
- Perguntas 1, 2 e 3 são obrigatórias. Não prossiga sem elas.
- Se o usuário já forneceu a base de tema na mensagem de invocação (caminho de pasta
  ou documento colado), trate como resposta da pergunta 1 automaticamente.
- Pergunta 4 é opcional. Se o usuário já colou código ou screenshot na mesma
  mensagem, trate como resposta da pergunta 4 automaticamente.
- Se o usuário responder 2 e 3 mas pular a 1, peça especificamente a base de
  tema novamente — ela é a fundação de tudo.
- **NÃO confunda** o código da pergunta 4 (referência visual) com a pasta/documento
  da pergunta 1 (tema base). São entradas distintas.
- Objetivo obrigatório desta skill: **substituir completamente** o layout padrão Laravel
  (shell visual, wrappers e blocos estruturais de página) pelo tema base indicado.

---

## Fase 1 — Ler o Ambiente

Execute todas as sub-etapas antes de passar para a Fase 2.

### 1a — Detectar estrutura do projeto

```bash
# Layouts existentes
ls resources/js/Layouts/ 2>/dev/null

# Componentes de UI disponíveis para reuso
ls resources/js/components/ui/ 2>/dev/null
ls resources/js/Components/ 2>/dev/null
ls resources/js/components/shared/ 2>/dev/null

# CSS global com tokens de tema
ls resources/css/app.css resources/css/app.pcss 2>/dev/null
```

**Registrar:**
- Quais subpastas de componentes existem
- Quais arquivos CSS globais existem
- **NÃO assumir** nomes Breeze (PrimaryButton, TextInput, etc.) — registrar apenas o que for encontrado

### 1a.1 — Mapear layout padrão Laravel para substituição completa

Antes de gerar arquivos, identificar quais shells/layouts atualmente estruturam as páginas
para garantir substituição total do padrão anterior.

```bash
# Layout principal e variantes comuns
ls resources/js/Layouts 2>/dev/null
ls resources/js/layouts 2>/dev/null

# Procurar usos dos layouts atuais
rg "AppLayout|AuthenticatedLayout|GuestLayout|MainLayout|DefaultLayout|Layout" resources/js/Pages resources/js/pages 2>/dev/null
```

**Registrar:**
- Quais layouts base estão em uso real nas páginas
- Quais wrappers/containers devem ser removidos, substituídos ou reencaixados
- Se houver conflito entre layout antigo e tema novo, o tema novo sempre vence

### 1b — Ler a base de tema

**APENAS se a resposta da pergunta 1 foi uma PASTA:**

```bash
# Ler tokens de tema no CSS global — tentar app.css, fallback para app.pcss
cat resources/css/app.css 2>/dev/null || cat resources/css/app.pcss 2>/dev/null
```

Buscar no CSS:
- `@theme { }` — tokens Tailwind v4 (ex: `--color-primary: oklch(...)`)
- `:root { }` — CSS custom properties (ex: `--primary: 222 47% 11%`)
- `@layer base { }` — overrides de base

**Se nenhum arquivo CSS for encontrado** (`app.css` e `app.pcss` ausentes): acionar o fallback de tokens ausentes (ver abaixo).

**Se `components/ui/` existir:** para cada `.vue` encontrado, ler o arquivo para identificar nome e props aceitas, depois classificar por papel funcional usando a tabela de mapeamento (seção "Mapeamento de componentes UI").

**Se `components/ui/` não existir ou estiver vazio:** registrar "Sem componentes UI encontrados — usar fallback HTML+Tailwind" e prosseguir.

**Se o CSS lido não contiver `@theme {}` nem `:root {}`:** perguntar ao usuário:
> "Não encontrei tokens de tema no CSS. Quer colar um documento com os tokens
> ou devo usar classes Tailwind utilitárias padrão (ex: `bg-blue-600`, `text-gray-*`)?"
> - Se colar tokens → processar como documento (abaixo)
> - Se pedir padrão → usar classes utilitárias Tailwind sem `[]`

**APENAS se a resposta da pergunta 1 foi um DOCUMENTO:**

Extrair do documento:
- Tokens de cor nomeados e seus valores (hex, CSS var ou classe Tailwind)
- Tipografia: famílias de fonte, escala de tamanhos, pesos
- Espaçamentos e convenções de padding/gap
- Convenções visuais: border-radius, shadow, estilo de card, estilo de input
- Quaisquer regras documentadas de componentes

### 1c — Analisar referência visual (opcional)

**Só executar se o usuário forneceu código ou screenshot na pergunta 4.**

**Se código Vue/HTML:**
- Ler o código diretamente
- Identificar classes CSS, diretivas Vue, nomes de componentes, padrões de slot
- Extrair nomes de variáveis/props/emits já em uso (preservar consistência de nomenclatura)
- Verificar quais componentes de UI já estão sendo usados no código

**Se screenshot:**
- Identificar regiões: header, filtros, tabela/grid, paginação, sidebar, footer
- Anotar elementos interativos: botões, inputs, selects, checkboxes
- Anotar elementos dinâmicos: badges de contagem, estados de loading, empty state
- Se regiões forem ambíguas, perguntar antes de assumir

### 1d — Consolidar descobertas

Compilar internamente (não exibir ainda — isso acontece na Fase 2):

- **Tokens identificados:** lista de tokens de cor, CSS vars ou classes Tailwind do tema
- **Componentes UI disponíveis:** lista de componentes mapeados por papel funcional
- **Convenções visuais:** estilo de border-radius, padrão de card, estilo de input
- **Padrões da referência (se fornecida):** padrões de lista/form encontrados

---

## Mapeamento de componentes UI

Quando componentes são encontrados em `components/ui/`, mapear para papéis funcionais:

| Papel | Como identificar |
|-------|-----------------|
| Botão primário | Nome contendo "Button" ou "Btn" com `variant="default"` ou `variant="primary"`. Se houver ambiguidade entre múltiplos arquivos de botão, classificar o primeiro alfabeticamente como primário e registrar a ambiguidade em 1d. |
| Botão secundário | Mesmo componente com `variant="secondary"` ou `variant="outline"` |
| Input de texto | Nome contendo "Input" com prop `modelValue` |
| Label de campo | Nome contendo "Label" |
| Erro de campo | Nome contendo "Error" ou "Message" |
| Dialog/Confirm | Nome contendo "Dialog", "Modal" ou "Alert" |
| Toast/Notificação | Nome contendo "Toast", "Notification" ou "Sonner" |

Se um papel não for mapeável com confiança → usar HTML+Tailwind puro para aquele papel.

**Equivalentes HTML+Tailwind (quando não há componente mapeável):**

| Papel | Fallback |
|-------|---------|
| Botão primário | `<button class="px-4 py-2 bg-primary text-primary-foreground rounded-md text-sm font-medium hover:bg-primary/90 disabled:opacity-50">` |
| Botão secundário | `<button class="px-4 py-2 border border-input rounded-md text-sm font-medium hover:bg-accent disabled:opacity-50">` |
| Input | `<input class="w-full border border-input rounded-md px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-ring">` |
| Label | `<label class="text-sm font-medium text-foreground">` |
| Erro | `<p class="text-sm text-destructive mt-1">` |
| Dialog/Confirm | `window.confirm('mensagem')` — retorna boolean |
| Toast | Elemento `<div>` fixo no canto inferior direito, removido após 3s |

---

## Fase 2 — Diagnóstico e Confirmação

Após concluir a Fase 1, apresentar ao usuário o diagnóstico abaixo e **aguardar confirmação** antes de gerar qualquer arquivo.

```
## Diagnóstico do Ambiente

### Tokens de Tema Identificados
[Listar tokens extraídos, ex:]
- Cor primária: `--color-primary` / `bg-primary`
- Cor de fundo: `--color-background` / `bg-background`
- Borda: `--color-border`
- Destructive: `--color-destructive`
- Radius padrão: `rounded-md`

### Componentes de UI Disponíveis para Reuso
[Listar componentes encontrados com ✅ ou ❌:]
- ✅ Button.vue → papel: botão primário (variant="default") e secundário (variant="outline")
- ✅ Input.vue → papel: input de texto
- ❌ Dialog → não encontrado, usará `window.confirm()` como fallback
- ❌ Toast → não encontrado, usará implementação inline

### O Que Será Gerado
- ListPage: header + filtros + tabela + paginação + bulk actions + empty state
- FormPage: header + conteúdo principal + sidebar + footer sticky nativo (Salvar/Cancelar)
- 6 componentes de suporte: PageShell, PageHeader, PageFilters, PagePagination,
  PageEmptyState, PageBulkActionBar
- 4 composables: usePageFilters, useSelection, useFormDirty, usePageActions
- Documentação markdown em `<caminho-componentes>/docs/`
- Plano de substituição do layout padrão atual pelas novas bases de página

### Como os Tokens Serão Aplicados
- Botão primário → [componente ou HTML+Tailwind] com classes `bg-primary text-primary-foreground`
- Input → [componente ou HTML+Tailwind]
- Card/container → `rounded-lg border bg-card`
- Footer → `bg-white border-t` (ou equivalente com tokens do tema)

### Política de Substituição de Layout
- O shell visual padrão anterior do Laravel **não será preservado**
- O tema informado será a fonte única de estrutura visual e tokens
- Qualquer bloco legado incompatível será removido ou reescrito

O diagnóstico está correto? Posso prosseguir com a geração?
```

**Aguardar confirmação do usuário.** Se corrigir algo (token errado, componente diferente), atualizar o diagnóstico interno e reconfirmar.

---

## Fase 3 — Arquitetura de Saída

Gere os arquivos na seguinte estrutura (adapte aos caminhos confirmados na PRIMEIRA AÇÃO):

```
resources/js/
├── components/
│   └── page/                          ← caminho confirmado na PRIMEIRA AÇÃO
│       ├── PageShell.vue              # Container base (padding, max-width, scroll)
│       ├── PageHeader.vue             # Título, breadcrumb, slot #header-actions
│       ├── PageFilters.vue            # Filtros com chips ativos, "Limpar filtros"
│       ├── PagePagination.vue         # Paginação com meta do Laravel paginate()
│       ├── PageEmptyState.vue         # Estado vazio configurável
│       ├── PageBulkActionBar.vue      # Barra de ações em massa
│       ├── ListPage.vue               # Sublayout completo de listagem
│       ├── FormPage.vue               # Sublayout completo de formulário
│       └── docs/                      # Documentação gerada (um .md por artefato)
└── composables/                       ← caminho confirmado na PRIMEIRA AÇÃO
    ├── usePageFilters.ts
    ├── useSelection.ts
    ├── useFormDirty.ts
    └── usePageActions.ts
```

**Decisões arquiteturais obrigatórias:**

- `FormPage.vue` tem footer sticky **nativo** (built-in) com "Salvar" e "Cancelar".
  Não é um slot — é comportamento nativo controlado por `@submit`, `@cancel` e `:loading`.
  O slot `#footer` existe apenas para override completo do layout (uso raro).
- `ListPage.vue` não tem botões de ação built-in — header actions, bulk actions e
  paginação são todos slots.
- Botões e inputs: usar componentes mapeados na Fase 1 quando disponíveis;
  caso contrário, HTML+Tailwind puro com os tokens do tema.
- **Zero imports de `callcocam/*`** em qualquer arquivo gerado.
- Substituir completamente o layout padrão Laravel usado atualmente nas páginas alvo;
  não manter wrappers visuais antigos por compatibilidade estética.

### 3.1 — Instrução de backend para acesso padronizado (quando solicitado)

Quando o usuário pedir para "criar páginas no backend" ou "organizar acesso", aplicar
este padrão junto da geração dos layouts/composables.

**Objetivo:** garantir acesso previsível e organizado às páginas Inertia que usam os
layouts gerados.

**Estrutura recomendada (Laravel 12/13):**

```text
app/Http/Controllers/Admin/<Recurso>Controller.php
routes/web.php
resources/js/Pages/Admin/<Recurso>/Index.vue
resources/js/Pages/Admin/<Recurso>/Create.vue
resources/js/Pages/Admin/<Recurso>/Edit.vue
```

**Convenções obrigatórias:**
- Agrupar rotas por domínio com prefixo e name prefix:
  `Route::prefix('admin')->name('admin.')->middleware(['auth', 'verified'])->group(...)`
- Seguir naming REST para páginas de CRUD:
  `admin.<recurso>.index`, `admin.<recurso>.create`, `admin.<recurso>.store`,
  `admin.<recurso>.edit`, `admin.<recurso>.update`, `admin.<recurso>.destroy`
- Controller retorna Inertia com caminhos padronizados:
  `Inertia::render('Admin/<Recurso>/Index', ...)`
- Página `Index` usa `ListPage`; páginas `Create/Edit` usam `FormPage`
- Não misturar convenções antigas de pasta/namespace fora do padrão `Admin/<Recurso>`

**Template de rotas (base):**

```php
Route::prefix('admin')
    ->name('admin.')
    ->middleware(['auth', 'verified'])
    ->group(function () {
        Route::resource('produtos', ProdutoController::class)
            ->except(['show']);
    });
```

**Template de controller (base):**

```php
public function index(Request $request): Response
{
    return Inertia::render('Admin/Produtos/Index', [
        // dados paginados + filtros
    ]);
}

public function create(): Response
{
    return Inertia::render('Admin/Produtos/Create');
}

public function edit(Produto $produto): Response
{
    return Inertia::render('Admin/Produtos/Edit', [
        'produto' => $produto,
    ]);
}
```

### 3.2 — Instrução de autorização backend (Policy + authorize)

Quando o usuário pedir padronização completa de backend, aplicar também autorização
por Policy para cada ação de CRUD.

**Objetivo:** garantir acesso seguro e previsível, sem regras soltas em controller.

**Padrão obrigatório:**
- Criar/usar `app/Policies/<Recurso>Policy.php`
- Registrar Policy no projeto conforme convenção da versão Laravel em uso
- Em controllers, usar `authorizeResource(<Model>::class, '<parametro>')` no construtor
  ou chamadas explícitas `$this->authorize('ability', $model)` por ação
- Nunca hardcodar regra de permissão em Vue; frontend apenas reage ao que backend autoriza

**Abilities mínimas por recurso:**
- `viewAny` para index
- `view` para detalhe/edição quando aplicável
- `create` para create/store
- `update` para edit/update
- `delete` para destroy

**Exemplo de controller com authorizeResource:**

```php
public function __construct()
{
  $this->authorizeResource(Produto::class, 'produto');
}
```

**Exemplo de proteção adicional por ação (quando necessário):**

```php
public function index(Request $request): Response
{
  $this->authorize('viewAny', Produto::class);

  return Inertia::render('Admin/Produtos/Index', [
    // dados paginados + filtros
  ]);
}
```

**Integração com frontend Inertia (recomendado):**
- Compartilhar abilities permitidas no payload da página para controlar visibilidade de
  botões (`Novo`, `Editar`, `Excluir`) sem substituir a validação backend.
- Se não autorizado: backend responde 403 e frontend apenas exibe estado de erro padrão.

---

## Fase 4 — Geração dos Componentes

### Regras de geração

1. `<script setup lang="ts">` em todos os arquivos — sem Options API
2. `defineProps<{}>()` com tipos TypeScript completos — sem objetos de props em runtime
3. `defineEmits<{}>()` com tipos TypeScript completos
4. **Slots sobre props** para conteúdo variável — nunca hardcodar conteúdo de negócio
5. Props opcionais com defaults explícitos (`false`, `undefined`, `[]`, `'sidebar'`)
6. Nenhuma lógica de negócio nos componentes de layout — a página-pai decide o comportamento
7. **Todos os textos de interface hardcoded em PT-BR** (ver lista abaixo)
8. Tokens do tema detectados na Fase 1 aplicados consistentemente
9. Componentes UI descobertos na Fase 1b reutilizados por papel funcional
10. **Zero imports de `callcocam/*`** — em nenhum arquivo gerado
11. Fallback HTML+Tailwind puro para papéis não mapeáveis (ver tabela na seção Mapeamento)
12. Compatível com Laravel 12/13 + Tailwind v4 (sem `tailwind.config.js`)

### Textos PT-BR obrigatórios

`Salvar` · `Cancelar` · `Excluir` · `Novo` · `Buscar` · `Limpar filtros` ·
`Nenhum resultado encontrado` · `Você tem alterações não salvas` ·
`Sim, excluir` · `Esta ação não pode ser desfeita`

---

### Contratos dos componentes de suporte

#### `PageShell.vue`

Props:

| Prop | Tipo | Default | Descrição |
|------|------|---------|-----------|
| `maxWidth` | `'sm'\|'md'\|'lg'\|'xl'\|'2xl'\|'full'` | `'2xl'` | Largura máxima do conteúdo |
| `padding` | `boolean` | `true` | Aplica padding lateral padrão |
| `scrollable` | `boolean` | `true` | Permite scroll vertical interno |

Slots: `#default` — conteúdo livre. Emits: nenhum.

#### `PageHeader.vue`

Props:

| Prop | Tipo | Obrigatório | Default | Descrição |
|------|------|-------------|---------|-----------|
| `title` | `string` | Sim | — | Título principal |
| `breadcrumbs` | `{ label: string, href?: string }[]` | Não | `[]` | Trilha de navegação |
| `total` | `number` | Não | `undefined` | Exibido junto ao título: "Produtos (42)" |

Slots: `#header-actions` — botões posicionados à direita do título. Emits: nenhum.

#### `PageFilters.vue`

Props:

| Prop | Tipo | Default | Descrição |
|------|------|---------|-----------|
| `activeCount` | `number` | `0` | Filtros ativos — exibe badge e botão "Limpar filtros" |

Slots: `#default` — conteúdo dos filtros (inputs, selects, etc.).

Emits: `clear` — clique em "Limpar filtros".

> `PageFilters` é estrutural — quem sincroniza com a URL é o composable `usePageFilters`.

#### `PagePagination.vue`

Props:

| Prop | Tipo | Obrigatório | Descrição |
|------|------|-------------|-----------|
| `meta` | `LaravelPaginatorMeta` | Sim | Meta do Laravel `paginate()` |

```ts
interface LaravelPaginatorMeta {
  current_page: number
  last_page: number
  per_page: number
  total: number
  from: number | null
  to: number | null
}
```

Emits: `change` com payload `{ page: number }`. Slots: nenhum — autocontido.

#### `PageEmptyState.vue`

Props:

| Prop | Tipo | Default | Descrição |
|------|------|---------|-----------|
| `title` | `string` | `'Nenhum resultado encontrado'` | Título |
| `description` | `string` | `''` | Texto explicativo |
| `icon` | `string` | `''` | Nome do ícone lucide-vue-next (opcional) |

Slots: `#action` — CTA (ex: "Criar primeiro item"). Emits: nenhum.

#### `PageBulkActionBar.vue`

Props:

| Prop | Tipo | Obrigatório | Descrição |
|------|------|-------------|-----------|
| `count` | `number` | Sim | Quantidade de itens selecionados |

Slots: `#default` — botões de ação em massa.

Emits: `clear` — limpar seleção.

> `ListPage` passa `count` e escuta `@clear`. `PageBulkActionBar` não consome `useSelection` diretamente.

---

### Contrato do `<ListPage>`

```vue
<ListPage
  title="Produtos"
  :breadcrumbs="[{ label: 'Painel', href: '/' }, { label: 'Produtos' }]"
  :total="meta.total"
  :loading="carregando"
>
  <!-- Ações do cabeçalho (Novo, Importar, Exportar) -->
  <template #header-actions>
    <button @click="criar">Novo Produto</button>
  </template>

  <!-- Filtros — componente de filtros customizado da página -->
  <template #filters>
    <FiltrosProdutos v-model="filtros" />
  </template>

  <!-- Ações em massa — aparece somente com seleção ativa -->
  <template #bulk-actions="{ selected, clear }">
    <button @click="excluirSelecionados(selected, clear)">
      Excluir {{ selected.length }} itens
    </button>
  </template>

  <!-- Corpo da listagem (tabela, grid, kanban — livre) -->
  <template #default>
    <TabelaProdutos :itens="produtos" v-model:selecao="selecao" />
  </template>

  <!-- Paginação -->
  <template #pagination>
    <PagePagination :meta="meta" @change="mudarPagina" />
  </template>
</ListPage>
```

**Props de `ListPage`:**

| Prop | Tipo | Obrigatório | Default | Descrição |
|------|------|-------------|---------|-----------|
| `title` | `string` | Sim | — | Título da página |
| `breadcrumbs` | `{ label: string, href?: string }[]` | Não | `[]` | Trilha |
| `total` | `number` | Não | `undefined` | Total de registros |
| `loading` | `boolean` | Não | `false` | Estado de carregamento |

**Slots de `ListPage`:**

| Slot | Escopo | Descrição |
|------|--------|-----------|
| `#header-actions` | — | Botões do cabeçalho |
| `#filters` | — | Barra de filtros |
| `#bulk-actions` | `{ selected: any[], clear: () => void }` | Ações em massa |
| `#default` | — | Corpo da listagem |
| `#pagination` | — | Componente de paginação |
| `#empty` | — | Estado vazio customizado |

**Emits de `ListPage`:** `page-change` com `{ page: number }`.

---

### Contrato do `<FormPage>`

```vue
<FormPage
  :title="form.id ? `Editando ${form.nome}` : 'Novo Produto'"
  :breadcrumbs="[{ label: 'Produtos', href: route('produtos.index') }, { label: 'Formulário' }]"
  :loading="form.processing"
  :dirty="form.isDirty"
  @submit="salvar"
  @cancel="router.visit(route('produtos.index'))"
>
  <!-- Ações extras no cabeçalho (Excluir, Duplicar) — opcional -->
  <template #header-actions>
    <button @click="confirmarExclusao">Excluir</button>
  </template>

  <!-- Campos do formulário -->
  <template #default>
    <div class="space-y-4">
      <!-- campos aqui -->
    </div>
  </template>

  <!-- Sidebar de metadados — opcional, só com layout="sidebar" -->
  <template #sidebar>
    <div class="rounded-lg border p-4 space-y-3">
      <p class="text-sm font-medium">Status</p>
      <select v-model="form.status">
        <option value="rascunho">Rascunho</option>
        <option value="ativo">Ativo</option>
      </select>
    </div>
  </template>

  <!-- NÃO use #footer a menos que precise substituir completamente o footer padrão -->
  <!-- O footer padrão já inclui "Salvar" e "Cancelar" automaticamente -->
</FormPage>
```

**Footer nativo (built-in — não é slot):**

```vue
<!-- sticky bottom — gerado automaticamente pelo FormPage -->
<!-- SecondaryButton/PrimaryButton = componente descoberto ou HTML+Tailwind puro -->
<footer class="sticky bottom-0 border-t bg-white px-6 py-3">
  <div class="flex items-center justify-between">
    <span v-if="dirty" class="text-sm text-muted-foreground">
      Você tem alterações não salvas
    </span>
    <div class="flex gap-2 ml-auto">
      <SecondaryButton @click="$emit('cancel')">{{ cancelLabel }}</SecondaryButton>
      <PrimaryButton :disabled="loading" @click="$emit('submit')">
        <Loader2 v-if="loading" class="mr-2 h-4 w-4 animate-spin" />
        {{ submitLabel }}
      </PrimaryButton>
    </div>
  </div>
</footer>
```

**Props de `FormPage`:**

| Prop | Tipo | Obrigatório | Default | Descrição |
|------|------|-------------|---------|-----------|
| `title` | `string` | Sim | — | Título dinâmico |
| `breadcrumbs` | `{ label: string, href?: string }[]` | Não | `[]` | Trilha |
| `loading` | `boolean` | Não | `false` | Processando — desabilita "Salvar" |
| `dirty` | `boolean` | Não | `false` | Exibe "Você tem alterações não salvas" |
| `layout` | `'single' \| 'sidebar'` | Não | `'sidebar'` | Disposição |
| `submitLabel` | `string` | Não | `'Salvar'` | Texto do botão de envio |
| `cancelLabel` | `string` | Não | `'Cancelar'` | Texto do botão cancelar |

**Slots de `FormPage`:**

| Slot | Escopo | Descrição |
|------|--------|-----------|
| `#header-actions` | — | Ações extras (Excluir, Duplicar) |
| `#default` | — | Campos do formulário |
| `#sidebar` | — | Metadados — só com `layout="sidebar"` |
| `#footer` | — | Override completo do footer (uso raro) |

**Emits de `FormPage`:** `submit`, `cancel`.

---

## Fase 5 — Composables

> **Convenção:** nomes de export em inglês (padrão de código interno).
> Textos de interface nos componentes gerados em PT-BR.

### `usePageFilters(defaults)`

Sincroniza filtros com a URL (query params) via Inertia router.
Debounce configurável para busca de texto.
Reset volta aos defaults e limpa a URL.

```ts
const { filters, setFilter, resetFilters, activeFilterCount } = usePageFilters({
  busca: '',
  status: null as string | null,
  categoria_id: null as string | null,
})
```

Exports: `filters` (Ref\<T\>), `setFilter`, `resetFilters`, `activeFilterCount` (ComputedRef\<number\>)

---

### `useSelection(items, key?)`

Controle de seleção de linhas para bulk actions. Key padrão: `'id'`.

```ts
const { selected, toggle, toggleAll, clear, isSelected, hasSelection } = useSelection(produtos)
```

Exports: `selected` (Ref\<T[keyof T][]\>), `toggle`, `toggleAll`, `clear`, `isSelected`, `hasSelection` (ComputedRef\<boolean\>)

---

### `useFormDirty(form)`

Detecta se o form foi modificado (Inertia `form.isDirty`).
Registra `beforeunload` para avisar o usuário se tentar fechar a aba.
Integra com navegação Inertia para interceptar saída sem salvar.

```ts
const { isDirty, confirmLeave } = useFormDirty(form)
```

Exports: `isDirty` (ComputedRef\<boolean\>), `confirmLeave` (() => Promise\<boolean\>)

---

### `usePageActions()`

Helpers padronizados para ações comuns: confirmação, toast, loading state.

**`confirm`:** se componente Dialog/Modal for encontrado em `components/ui/` na Fase 1b → usar esse componente; caso contrário → `window.confirm()` nativo.

**`toast`:** se componente Toast/Sonner for encontrado em `components/ui/` na Fase 1b → usar esse componente; caso contrário → elemento `<div>` fixo no canto inferior direito, removido após 3s.

```ts
const { confirm, toast, withLoading } = usePageActions()

const confirmado = await confirm({
  title: 'Excluir produto?',
  description: 'Esta ação não pode ser desfeita.',
  confirmLabel: 'Sim, excluir',
  variant: 'destructive',
})

if (confirmado) {
  await withLoading(async () => {
    await router.delete(route('produtos.destroy', produto.id))
  })
  toast.success('Produto excluído com sucesso.')
}
```

Exports: `confirm`, `toast` (`{ success, error, warning }`), `withLoading`

---

## Fase 6 — Documentação dos Componentes

Para cada componente e composable gerado, salvar um arquivo markdown em
`<caminho-componentes>/docs/` (ex: `resources/js/components/page/docs/`).

Formato de cada arquivo:

```markdown
## NomeDoComponente

**Localização:** `resources/js/components/page/NomeDoComponente.vue`

**Propósito:** [uma frase descrevendo o que resolve]

### Props
| Prop | Tipo | Obrigatório | Default | Descrição |
|------|------|-------------|---------|-----------|

### Slots
| Slot | Escopo | Descrição |
|------|--------|-----------|

### Emits
| Evento | Payload | Descrição |
|--------|---------|-----------|

### Exemplo mínimo de uso
\`\`\`vue
<!-- exemplo em PT-BR aqui -->
\`\`\`
```

**Exemplo mínimo para ListPage:**
```vue
<ListPage title="Produtos" :total="meta.total">
  <template #header-actions>
    <button @click="criar">Novo Produto</button>
  </template>
  <template #default>
    <TabelaProdutos :itens="produtos" />
  </template>
</ListPage>
```

**Exemplo mínimo para FormPage:**
```vue
<FormPage
  :title="form.id ? 'Editando Produto' : 'Novo Produto'"
  :loading="form.processing"
  :dirty="form.isDirty"
  @submit="salvar"
  @cancel="voltar"
>
  <template #default>
    <!-- campos -->
  </template>
</FormPage>
```

---

## Fase 7 — Checklist final antes de entregar

### Estrutura e TypeScript
- [ ] Todos os componentes têm `<script setup lang="ts">`
- [ ] Props com `defineProps<{}>()` totalmente tipadas
- [ ] Emits com `defineEmits<{}>()` totalmente tipados
- [ ] Slots documentados com comentários inline no `.vue` e na documentação markdown
- [ ] Composables exportam apenas o necessário (sem expor estado interno)

### Lógica e comportamento
- [ ] Nenhuma lógica de negócio nos componentes de layout
- [ ] Estados de carregamento, vazio e erro cobertos em cada componente
- [ ] FormPage com footer sticky nativo funcionando (Salvar/Cancelar)
- [ ] `useFormDirty` integrado ao FormPage (aviso "Você tem alterações não salvas")
- [ ] Bulk actions só aparecem com seleção ativa
- [ ] `usePageActions.confirm` usa Dialog descoberto ou `window.confirm` como fallback
- [ ] `usePageActions.toast` usa Toast descoberto ou implementação inline como fallback
- [ ] Layout padrão Laravel anterior removido/substituído nas páginas alvo
- [ ] Páginas alvo renderizando com o novo shell baseado no tema (sem mistura visual legado)

### Tema e visual
- [ ] Tokens do tema aplicados consistentemente (CSS vars `@theme {}` ou `:root {}`)
- [ ] Sem valores Tailwind arbitrários que contradizem o tema
- [ ] Dark mode respeitado se o tema tiver tokens de dark mode

### Dependências e stack
- [ ] **Zero imports de `callcocam/*`** em qualquer arquivo gerado
- [ ] Compatível com Laravel 12/13 (sem assumir Breeze ou qualquer starter kit)
- [ ] Compatível com Tailwind v4 (sem `tailwind.config.js`)
- [ ] Componentes UI mapeados por papel funcional — sem assumir nomes fixos
- [ ] Fallback HTML+Tailwind para papéis não mapeáveis
- [ ] Quando solicitado, rotas e controllers backend criados em padrão único
- [ ] Nomes de rotas e caminhos Inertia padronizados (`admin.<recurso>.*` + `Admin/<Recurso>/*`)
- [ ] Quando solicitado, Policy do recurso implementada e registrada
- [ ] Controller protegido com `authorizeResource` ou `authorize` por ação
- [ ] Ações críticas (create/update/delete) sempre validadas no backend

### Textos e idioma
- [ ] Todos os textos de interface hardcoded em PT-BR
- [ ] Labels padrão: "Salvar", "Cancelar", "Excluir", "Novo", "Buscar", "Limpar filtros"
- [ ] Mensagens de estado vazio em PT-BR
- [ ] Mensagens de confirmação em PT-BR
- [ ] Exports de composables em inglês (padrão de código interno)

### Documentação
- [ ] Documentação markdown gerada em `<caminho-componentes>/docs/`
- [ ] Um arquivo .md por componente (8 total) e por composable (4 total)
- [ ] Exemplo mínimo PT-BR em cada bloco de documentação

---

## Referências adicionais

- Leia `references/list-patterns.md` para variações de listagem
- Leia `references/form-patterns.md` para variações de formulário