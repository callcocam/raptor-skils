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
| Componentes de UI | Descobertos dinamicamente em `resources/js/components/ui/` — sem assumir nomes Breeze |
| Textos | Hardcoded PT-BR — sem i18n |
| Footer do FormPage | Nativo (built-in) — "Salvar" e "Cancelar" embutidos via props/emits, não slot |
| Nomes de exports dos composables | Inglês (padrão de código interno) — textos de interface em PT-BR |
| Layout de tabs | Removido desta versão — apenas `'single'` e `'sidebar'` |

---

## Fluxo do skill (novo)

```
Fase 0 — Verificar estado anterior
  └─ ListPage.vue / FormPage.vue já existem?
     → Atualizar / Recriar / Sem limitação

PRIMEIRA AÇÃO — Coletar contexto base (3 obrigatórias + 1 opcional)
  ├─ 1. Tema base: caminho de PASTA (não código) OU cole DOCUMENTO de tema
  ├─ 2. Onde salvar componentes? (sugestão: resources/js/components/page/)
  ├─ 3. Onde salvar composables? (sugestão: resources/js/composables/)
  └─ 4. (Opcional) Código Vue/HTML ou screenshot de referência visual para estilo

Fase 1 — Ler o Ambiente
  ├─ 1a: Detectar estrutura do projeto
  │       Listar resources/js/components/ e subpastas (ui/, shared/, layout/)
  │       Verificar resources/js/Layouts/
  │       NÃO assumir nomes Breeze — registrar o que for encontrado
  ├─ 1b: Ler tema base
  │       APENAS se questão 1 foi uma PASTA:
  │         → Ler app.css: buscar @theme {} (Tailwind v4) ou :root {} (CSS vars)
  │         → Se nenhum encontrado: usar fallback (ver seção "Fallbacks")
  │         → Se components/ui/ existir: listar componentes e classificar por papel
  │         → Se components/ui/ não existir: registrar "sem componentes UI — usar fallback HTML"
  │       APENAS se questão 1 foi um DOCUMENTO:
  │         → Extrair tokens de cor, tipografia, espaçamento, convenções visuais
  │         → Ignorar qualquer detecção de pasta
  ├─ 1c: Analisar referência visual (opcional — só se fornecida na questão 4)
  │       Se código Vue/HTML → identificar classes, componentes, padrões de slot, nomes
  │       Se screenshot → identificar regiões, botões, inputs, badges, estados
  │       NUNCA confundir código da questão 4 com tema base da questão 1
  └─ 1d: Consolidar internamente
          tokens identificados + componentes UI disponíveis + convenções visuais + padrões da referência

Fase 2 — Diagnóstico e Confirmação
  └─ Apresentar: tokens encontrados + componentes UI + o que será gerado + como tokens serão aplicados
     → Aguardar confirmação antes de gerar qualquer coisa

Fase 3 — Arquitetura de Saída
  └─ Definir estrutura de arquivos exata com caminhos confirmados

Fase 4 — Geração dos Componentes
  └─ 8 componentes .vue (contratos completos neste doc)

Fase 5 — Composables
  └─ 4 composables .ts

Fase 6 — Documentação
  └─ Markdown salvo em <caminho-componentes>/docs/ (ex: resources/js/components/page/docs/)
     Um arquivo .md por componente + um por composable

Fase 7 — Checklist Final
```

---

## Fallbacks

### Sem tokens CSS em `app.css`

Se `app.css` não contiver `@theme {}` nem `:root {}`:
1. Perguntar ao usuário: "Não encontrei tokens de tema no `app.css`. Quer colar um documento com os tokens ou devo usar classes Tailwind utilitárias padrão (ex: `bg-blue-600`, `gray-*`)?"
2. Se o usuário colar tokens → processar como documento (Fase 1b, branch documento)
3. Se o usuário pedir para usar padrão → usar classes utilitárias Tailwind seguras sem `[]`

### Sem `components/ui/`

Se o diretório `resources/js/components/ui/` não existir (ou existir mas vazio):
- Registrar: "Sem componentes UI encontrados"
- Gerar todos os elementos de botão, input e label como HTML+Tailwind puro
- Não tentar importar nenhum componente externo

### Mapeamento de componentes UI descobertos

Quando componentes são encontrados em `components/ui/`, mapear para papéis funcionais assim:

| Papel | Como identificar |
|-------|-----------------|
| Botão primário (ação principal) | Componente com nome contendo "Button", "Btn" + aceita `variant="default"` ou `variant="primary"` **OU** é o único botão sem variante |
| Botão secundário (ação secundária) | Mesmo componente com `variant="secondary"` ou `variant="outline"` |
| Input de texto | Componente com nome contendo "Input" aceita `modelValue` |
| Label de campo | Componente com nome contendo "Label" |
| Erro de campo | Componente com nome contendo "Error" ou "Message" |
| Dialog/Confirm | Componente com nome contendo "Dialog", "Modal", "Alert" |
| Toast/Notificação | Componente com nome contendo "Toast", "Notification", "Sonner" |

Se um papel não for mapeável com confiança → usar HTML+Tailwind puro para aquele papel específico.

---

## Arquitetura de saída

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
│       └── docs/                      # Documentação gerada
│           ├── ListPage.md
│           ├── FormPage.md
│           ├── PageShell.md
│           ├── PageHeader.md
│           ├── PageFilters.md
│           ├── PagePagination.md
│           ├── PageEmptyState.md
│           ├── PageBulkActionBar.md
│           ├── usePageFilters.md
│           ├── useSelection.md
│           ├── useFormDirty.md
│           └── usePageActions.md
└── composables/                       ← caminho confirmado na PRIMEIRA AÇÃO
    ├── usePageFilters.ts
    ├── useSelection.ts
    ├── useFormDirty.ts
    └── usePageActions.ts
```

---

## Contratos dos componentes de suporte

### `PageShell.vue`

**Props:**

| Prop | Tipo | Obrigatório | Default | Descrição |
|------|------|-------------|---------|-----------|
| `maxWidth` | `'sm' \| 'md' \| 'lg' \| 'xl' \| '2xl' \| 'full'` | Não | `'2xl'` | Largura máxima do conteúdo |
| `padding` | `boolean` | Não | `true` | Aplica padding lateral padrão |
| `scrollable` | `boolean` | Não | `true` | Permite scroll vertical interno |

**Slots:** `#default` apenas — conteúdo livre.

**Emits:** nenhum.

---

### `PageHeader.vue`

**Props:**

| Prop | Tipo | Obrigatório | Default | Descrição |
|------|------|-------------|---------|-----------|
| `title` | `string` | Sim | — | Título principal da página |
| `breadcrumbs` | `{ label: string, href?: string }[]` | Não | `[]` | Trilha de navegação |
| `total` | `number` | Não | `undefined` | Exibido junto ao título (ex: "Produtos (42)") |

**Slots:**

| Slot | Escopo | Descrição |
|------|--------|-----------|
| `#header-actions` | — | Botões do cabeçalho (posicionados à direita do título) |

**Emits:** nenhum.

---

### `PageFilters.vue`

**Props:**

| Prop | Tipo | Obrigatório | Default | Descrição |
|------|------|-------------|---------|-----------|
| `activeCount` | `number` | Não | `0` | Número de filtros ativos (exibe badge e botão "Limpar filtros") |

**Slots:**

| Slot | Escopo | Descrição |
|------|--------|-----------|
| `#default` | — | Conteúdo dos filtros (inputs, selects, etc.) |

**Emits:**

| Evento | Payload | Descrição |
|--------|---------|-----------|
| `clear` | — | Clique em "Limpar filtros" |

> **Nota:** `PageFilters` é estrutural — não sincroniza com URL. Quem faz a sincronização é `usePageFilters` no composable.

---

### `PagePagination.vue`

**Props:**

| Prop | Tipo | Obrigatório | Default | Descrição |
|------|------|-------------|---------|-----------|
| `meta` | `LaravelPaginatorMeta` | Sim | — | Meta do Laravel `paginate()` |

**Tipo `LaravelPaginatorMeta`:**

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

**Emits:**

| Evento | Payload | Descrição |
|--------|---------|-----------|
| `change` | `{ page: number }` | Usuário clicou em uma página |

**Slots:** nenhum — componente autocontido.

---

### `PageEmptyState.vue`

**Props:**

| Prop | Tipo | Obrigatório | Default | Descrição |
|------|------|-------------|---------|-----------|
| `title` | `string` | Não | `'Nenhum resultado encontrado'` | Título do estado vazio |
| `description` | `string` | Não | `''` | Texto explicativo |
| `icon` | `string` | Não | `''` | Nome do ícone (lucide-vue-next) ou vazio |

**Slots:**

| Slot | Escopo | Descrição |
|------|--------|-----------|
| `#action` | — | Botão ou link de CTA (ex: "Criar primeiro item") |

**Emits:** nenhum.

---

### `PageBulkActionBar.vue`

**Props:**

| Prop | Tipo | Obrigatório | Default | Descrição |
|------|------|-------------|---------|-----------|
| `count` | `number` | Sim | — | Quantidade de itens selecionados |

**Slots:**

| Slot | Escopo | Descrição |
|------|--------|-----------|
| `#default` | — | Botões de ação em massa |

**Emits:**

| Evento | Payload | Descrição |
|--------|---------|-----------|
| `clear` | — | Limpar seleção |

> **Nota:** `ListPage` é responsável por passar `count` e escutar `@clear`. `PageBulkActionBar` não consome `useSelection` diretamente.

---

## Contratos de slot — componentes principais

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
| `#bulk-actions` | `{ selected: any[], clear: () => void }` | Ações em massa — só visível com seleção ativa |
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
| `layout` | `'single' \| 'sidebar'` | Não | `'sidebar'` | Disposição do formulário |
| `submitLabel` | `string` | Não | `'Salvar'` | Texto do botão de envio |
| `cancelLabel` | `string` | Não | `'Cancelar'` | Texto do botão cancelar |

**Slots:**

| Slot | Escopo | Descrição |
|------|--------|-----------|
| `#header-actions` | — | Ações extras (Excluir, Duplicar) |
| `#default` | — | Campos do formulário |
| `#sidebar` | — | Metadados (status, datas, tags) — só usado com `layout="sidebar"` |
| `#footer` | — | Override completo do footer padrão (uso raro) |

**Emits:**

| Evento | Payload | Descrição |
|--------|---------|-----------|
| `submit` | — | Clique em "Salvar" |
| `cancel` | — | Clique em "Cancelar" |

**Footer nativo (built-in):**

```vue
<!-- Gerado automaticamente — sticky bottom -->
<!-- PrimaryButton / SecondaryButton = componentes descobertos ou HTML+Tailwind puro -->
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

> **Convenção de nomes:** exports dos composables em inglês (padrão de código interno). Textos de interface nos componentes gerados em PT-BR.

### `usePageFilters(defaults)`

Sincroniza filtros com URL via Inertia router. Debounce configurável para busca de texto.

```ts
const { filters, setFilter, resetFilters, activeFilterCount } = usePageFilters({
  busca: '',
  status: null as string | null,
})
```

**Exports:** `filters` (Ref), `setFilter`, `resetFilters`, `activeFilterCount` (ComputedRef\<number\>)

---

### `useSelection(items, key?)`

Seleção de linhas para bulk actions. Key padrão: `'id'`.

```ts
const { selected, toggle, toggleAll, clear, isSelected, hasSelection } = useSelection(produtos)
```

**Exports:** `selected` (Ref), `toggle`, `toggleAll`, `clear`, `isSelected`, `hasSelection` (ComputedRef\<boolean\>)

---

### `useFormDirty(form)`

Detecta mudanças não salvas. Registra `beforeunload` e intercepta navegação Inertia.

```ts
const { isDirty, confirmLeave } = useFormDirty(form)
```

**Exports:** `isDirty` (ComputedRef\<boolean\>), `confirmLeave` (() => Promise\<boolean\>)

---

### `usePageActions()`

Helpers padronizados: confirmação, toast, loading state.

**Mecanismo de `confirm`:**
- Se componente Dialog/Modal for encontrado em `components/ui/` na Fase 1b → usar esse componente
- Caso contrário → `window.confirm()` nativo com a mensagem formatada

**Mecanismo de `toast`:**
- Se componente Toast/Sonner for encontrado em `components/ui/` na Fase 1b → usar esse componente
- Caso contrário → implementação inline mínima com elemento fixo no canto inferior direito

```ts
const { confirm, toast, withLoading } = usePageActions()

const ok = await confirm({
  title: 'Excluir produto?',
  description: 'Esta ação não pode ser desfeita.',
  confirmLabel: 'Sim, excluir',
  variant: 'destructive',
})

toast.success('Produto excluído com sucesso.')
toast.error('Erro ao excluir produto.')

await withLoading(async () => {
  await router.delete(route('produtos.destroy', produto.id))
})
```

**Exports:** `confirm`, `toast` (`{ success, error, warning }`), `withLoading`

---

## Regras de geração

1. `<script setup lang="ts">` em todos os arquivos
2. `defineProps<{}>()` e `defineEmits<{}>()` totalmente tipados
3. Slots para conteúdo variável — nunca hardcodar conteúdo de negócio no layout
4. Tokens do tema (CSS vars `@theme {}` ou `:root {}`) aplicados consistentemente
5. Componentes UI descobertos na Fase 1b reutilizados (mapeamento por papel funcional)
6. **Zero imports de `callcocam/*`**
7. Fallback HTML + Tailwind puro quando papel funcional não mapeável
8. **Todos os textos de interface hardcoded em PT-BR**
9. Nenhuma lógica de negócio nos componentes de layout
10. Compatível com Laravel 12/13 + Tailwind v4 (sem `tailwind.config.js`)
11. Exports de composables em inglês — textos visíveis ao usuário em PT-BR

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
- [ ] `useFormDirty` integrado ao FormPage (aviso "Você tem alterações não salvas")
- [ ] Bulk actions só aparecem com seleção ativa
- [ ] `usePageActions.confirm` usa Dialog descoberto ou `window.confirm` como fallback
- [ ] `usePageActions.toast` usa Toast descoberto ou implementação inline como fallback

### Tema e visual
- [ ] Tokens do tema aplicados consistentemente
- [ ] Sem valores Tailwind arbitrários que contradizem o tema
- [ ] Dark mode respeitado se o tema tiver tokens de dark mode

### Dependências e stack
- [ ] Zero imports de `callcocam/*`
- [ ] Compatível com Laravel 12/13 (sem assumir Breeze)
- [ ] Compatível com Tailwind v4 (sem `tailwind.config.js`)
- [ ] Componentes UI mapeados por papel funcional — sem assumir nomes fixos
- [ ] Fallback HTML+Tailwind para papéis não mapeáveis

### Textos e idioma
- [ ] Todos os textos de interface hardcoded em PT-BR
- [ ] Labels padrão: "Salvar", "Cancelar", "Excluir", "Novo", "Buscar", "Limpar filtros"
- [ ] Mensagens de estado vazio em PT-BR
- [ ] Mensagens de confirmação em PT-BR
- [ ] Exports de composables em inglês

### Documentação
- [ ] Documentação markdown gerada em `<caminho-componentes>/docs/`
- [ ] Um arquivo .md por componente e por composable
- [ ] Exemplo mínimo PT-BR em cada bloco de documentação
