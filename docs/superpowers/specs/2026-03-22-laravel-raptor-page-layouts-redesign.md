# Spec: Reestruturação do skill `laravel-raptor-page-layouts`

**Data:** 2026-03-22
**Status:** Aprovado

---

## Contexto e motivação

O skill atual (`laravel-raptor-page-layouts`) parte de código Vue/HTML colado ou screenshots para extrair padrões de lista e formulário. O objetivo desta reestruturação é torná-lo independente do ecossistema `callcocam/laravel-raptor-*` e compatível com projetos Laravel puros (Laravel 12/13 + Inertia + Vue 3 + Tailwind v4), partindo obrigatoriamente de um **tema base** (pasta de design-system ou documento de tema) em vez de código existente.

---

## Decisões

| Decisão | Escolha |
|---------|---------|
| Entrada primária | Tema base: pasta de design-system OU documento colado |
| Entrada secundária | Código Vue/HTML ou screenshot como referência visual opcional |
| Independência de pacotes | Funciona sem `callcocam/*` — só Laravel + Inertia + Vue + Tailwind |
| Tokens de tema | Lidos de `app.css` (`@theme {}` Tailwind v4 ou `:root {}`) — sem `tailwind.config.js` |
| Componentes de UI | Descobertos dinamicamente em `resources/js/components/ui/` (shadcn, custom, etc.) — sem assumir nomes Breeze |
| Textos | Hardcoded PT-BR — sem i18n |
| Footer do FormPage | Nativo (built-in) — "Salvar" e "Cancelar" embutidos via props/emits, não slot |

---

## Fluxo do skill (novo)

```
Fase 0 — Verificar estado anterior
  └─ ListPage.vue / FormPage.vue já existem?
     → Atualizar / Recriar / Sem limitação

PRIMEIRA AÇÃO — Coletar contexto base (3 obrigatórias + 1 opcional)
  ├─ 1. Tema base: caminho da pasta OU cole o documento
  ├─ 2. Onde salvar componentes? (sugestão: resources/js/components/page/)
  ├─ 3. Onde salvar composables? (sugestão: resources/js/composables/)
  └─ 4. (Opcional) Código Vue/HTML ou screenshot de referência visual

Fase 1 — Ler o Ambiente
  ├─ 1a: Detectar estrutura do projeto
  │       Listar resources/js/components/ e subpastas
  │       Verificar resources/js/Layouts/
  │       NÃO assumir nomes Breeze — registrar o que for encontrado
  ├─ 1b: Ler tema base
  │       Se pasta → ler app.css (@theme {} ou :root {}) + listar components/ui/
  │       Se documento → extrair tokens de cor, tipografia, espaçamento, convenções visuais
  ├─ 1c: Analisar referência visual (opcional — só se fornecida na questão 4)
  │       Se código → identificar classes, componentes, padrões de slot, nomes de variáveis
  │       Se screenshot → identificar regiões, botões, inputs, badges, estados
  └─ 1d: Consolidar internamente
          tokens identificados + componentes UI disponíveis + convenções visuais + padrões da referência

Fase 2 — Diagnóstico e Confirmação
  └─ Apresentar ao usuário:
     - Tokens de tema encontrados
     - Componentes UI disponíveis para reuso
     - O que será gerado (ListPage, FormPage, componentes de suporte)
     - Como os tokens serão aplicados
     → Aguardar confirmação antes de gerar qualquer coisa

Fase 3 — Arquitetura de Saída
  └─ Definir estrutura de arquivos exata com caminhos confirmados

Fase 4 — Geração dos Componentes
  └─ 8 componentes .vue com slot contracts

Fase 5 — Composables
  └─ 4 composables .ts

Fase 6 — Documentação
  └─ Markdown por componente e composable

Fase 7 — Checklist Final
```

---

## Arquitetura de saída

```
resources/js/
├── components/
│   └── page/                          ← caminho confirmado na PRIMEIRA AÇÃO
│       ├── PageShell.vue              # Container base (padding, max-width, scroll)
│       ├── PageHeader.vue             # Título, breadcrumb, slot header-actions
│       ├── PageFilters.vue            # Filtros com chips ativos, "Limpar filtros"
│       ├── PagePagination.vue         # Paginação com meta do Laravel paginate()
│       ├── PageEmptyState.vue         # Estado vazio configurável
│       ├── PageBulkActionBar.vue      # Barra de ações em massa
│       ├── ListPage.vue               # Sublayout completo de listagem
│       └── FormPage.vue               # Sublayout completo de formulário
└── composables/                       ← caminho confirmado na PRIMEIRA AÇÃO
    ├── usePageFilters.ts              # Sincroniza filtros com URL (Inertia)
    ├── useSelection.ts                # Seleção de linhas, select-all, bulk actions
    ├── useFormDirty.ts                # Detecta mudanças não salvas, aviso de saída
    └── usePageActions.ts              # confirm, toast, withLoading
```

---

## Contratos de slot

### `ListPage.vue`

**Props:**

| Prop | Tipo | Obrigatório | Default | Descrição |
|------|------|-------------|---------|-----------|
| `title` | `string` | Sim | — | Título da página |
| `breadcrumbs` | `{ label: string, href?: string }[]` | Não | `[]` | Trilha de navegação |
| `total` | `number` | Não | `undefined` | Total de registros |
| `loading` | `boolean` | Não | `false` | Estado de carregamento |

**Slots:**

| Slot | Escopo | Descrição |
|------|--------|-----------|
| `#header-actions` | — | Botões do cabeçalho (Novo, Importar, Exportar) |
| `#filters` | — | Barra de filtros da página |
| `#bulk-actions` | `{ selecionados: any[], limpar: () => void }` | Ações em massa — só visível com seleção ativa |
| `#default` | — | Corpo da listagem (tabela, grid, kanban) |
| `#pagination` | — | Componente de paginação |
| `#empty` | — | Estado vazio customizado |

**Emits:**

| Evento | Payload | Descrição |
|--------|---------|-----------|
| `page-change` | `{ page: number }` | Mudança de página (se paginação automática) |

**Exemplo mínimo:**

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

---

### `FormPage.vue`

**Props:**

| Prop | Tipo | Obrigatório | Default | Descrição |
|------|------|-------------|---------|-----------|
| `title` | `string` | Sim | — | Título dinâmico |
| `breadcrumbs` | `{ label: string, href?: string }[]` | Não | `[]` | Trilha de navegação |
| `loading` | `boolean` | Não | `false` | Processando (desabilita "Salvar") |
| `dirty` | `boolean` | Não | `false` | Alterações não salvas (exibe aviso) |
| `layout` | `'single' \| 'sidebar' \| 'tabs'` | Não | `'sidebar'` | Disposição do formulário |
| `submitLabel` | `string` | Não | `'Salvar'` | Texto do botão de envio |
| `cancelLabel` | `string` | Não | `'Cancelar'` | Texto do botão cancelar |

**Slots:**

| Slot | Escopo | Descrição |
|------|--------|-----------|
| `#header-actions` | — | Ações extras (Excluir, Duplicar) |
| `#default` | — | Campos do formulário |
| `#sidebar` | — | Metadados (status, datas, tags) |
| `#footer` | — | Override completo do footer padrão (uso raro) |

**Emits:**

| Evento | Payload | Descrição |
|--------|---------|-----------|
| `submit` | — | Clique em "Salvar" |
| `cancel` | — | Clique em "Cancelar" |

**Footer nativo (built-in):**

```vue
<!-- Gerado automaticamente — sticky bottom -->
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

> **Nota:** `SecondaryButton` e `PrimaryButton` são substituídos pelos componentes UI descobertos na Fase 1b (ou HTML+Tailwind puro se nenhum for encontrado).

**Exemplo mínimo:**

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

## Composables

### `usePageFilters(defaults)`

Sincroniza filtros com URL via Inertia router. Debounce configurável para busca de texto.

```ts
const { filters, setFilter, resetFilters, activeFilterCount } = usePageFilters({
  busca: '',
  status: null as string | null,
})
```

### `useSelection(items, key?)`

Seleção de linhas para bulk actions. Key padrão: `'id'`.

```ts
const { selected, toggle, toggleAll, clear, isSelected, hasSelection } = useSelection(produtos)
```

### `useFormDirty(form)`

Detecta mudanças não salvas. Registra `beforeunload` e intercepta navegação Inertia.

```ts
const { isDirty, confirmLeave } = useFormDirty(form)
```

### `usePageActions()`

Helpers padronizados: confirmação, toast, loading state.

```ts
const { confirm, toast, withLoading } = usePageActions()

const ok = await confirm({
  title: 'Excluir produto?',
  description: 'Esta ação não pode ser desfeita.',
  confirmLabel: 'Sim, excluir',
  variant: 'destructive',
})
```

---

## Regras de geração

1. `<script setup lang="ts">` em todos os arquivos
2. `defineProps<{}>()` e `defineEmits<{}>()` totalmente tipados
3. Slots para conteúdo variável — nunca hardcodar conteúdo de negócio no layout
4. Tokens do tema (CSS vars `@theme {}` ou `:root {}`) aplicados consistentemente
5. Componentes UI descobertos na Fase 1b reutilizados — sem assumir nomes Breeze
6. **Zero imports de `callcocam/*`**
7. Fallback HTML + Tailwind puro se nenhum componente UI for encontrado
8. **Todos os textos hardcoded em PT-BR**
9. Nenhuma lógica de negócio nos componentes de layout
10. Compatível com Laravel 12/13 + Tailwind v4 (sem `tailwind.config.js`)

---

## Textos PT-BR obrigatórios

`Salvar` · `Cancelar` · `Excluir` · `Novo` · `Buscar` · `Limpar filtros` · `Nenhum resultado encontrado` · `Você tem alterações não salvas` · `Sim, excluir` · `Esta ação não pode ser desfeita`

---

## Checklist final (Fase 7)

### Estrutura e TypeScript
- [ ] Todos os componentes têm `<script setup lang="ts">`
- [ ] Props com `defineProps<{}>()` totalmente tipadas
- [ ] Emits com `defineEmits<{}>()` totalmente tipados
- [ ] Slots documentados com comentários inline no `.vue` e na documentação markdown
- [ ] Composables exportam apenas o necessário

### Lógica e comportamento
- [ ] Nenhuma lógica de negócio nos componentes de layout
- [ ] Estados de carregamento, vazio e erro cobertos
- [ ] FormPage com footer sticky nativo funcionando (Salvar/Cancelar)
- [ ] `useFormDirty` integrado ao FormPage (aviso de saída sem salvar)
- [ ] Bulk actions só aparecem com seleção ativa

### Tema e visual
- [ ] Tokens do tema aplicados consistentemente
- [ ] Sem valores Tailwind arbitrários que contradizem o tema
- [ ] Dark mode respeitado se o tema tiver tokens de dark mode

### Dependências e stack
- [ ] Zero imports de `callcocam/*`
- [ ] Compatível com Laravel 12/13 (sem assumir Breeze)
- [ ] Compatível com Tailwind v4 (sem `tailwind.config.js`)
- [ ] Componentes UI descobertos dinamicamente e reutilizados
- [ ] Fallback HTML+Tailwind para projetos sem componentes UI

### Textos e idioma
- [ ] Todos os textos de interface hardcoded em PT-BR
- [ ] Labels padrão: "Salvar", "Cancelar", "Excluir", "Novo", "Buscar", "Limpar filtros"
- [ ] Mensagens de estado vazio em PT-BR
- [ ] Mensagens de confirmação em PT-BR

### Documentação
- [ ] Documentação markdown gerada para cada componente (props, slots, emits)
- [ ] Documentação markdown gerada para cada composable
- [ ] Exemplo mínimo PT-BR em cada bloco de documentação
