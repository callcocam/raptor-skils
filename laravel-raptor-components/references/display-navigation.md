# Referência — Data Display

## DataTable

**Props:**
- `data`: `any[]`
- `columns`: `Column[]` — definição das colunas
- `loading`: boolean — skeleton das linhas
- `empty-text`: string
- `row-key`: string — default `'id'`
- `selectable`: boolean — checkbox por linha
- `expandable`: boolean — linha expansível

**Tipo Column:**
```ts
interface Column {
  key: string
  label: string
  sortable?: boolean
  width?: string
  align?: 'left' | 'center' | 'right'
  render?: (value: any, row: any) => string // para formatação simples
}
```

**Slots:** `column-{key}` para célula customizada, `expanded-{key}` para linha expandida, `empty`, `loading-row`

**Emits:** `sort`, `select`, `row-click`, `expand`

**Detalhe de qualidade:**
- Sticky header ao fazer scroll vertical
- Colunas fixas (left/right) via `position: sticky`
- Skeleton: N linhas com células de largura aleatória animadas
- Ordenação: ícone de seta que indica asc/desc, neutro quando não ordenado
- Linha expansível: animação de accordion no conteúdo expandido
- Hover highlight na linha inteira
- Empty state com slot customizável

---

## Avatar

**Props:**
- `src`: string — URL da imagem
- `alt`: string — texto alternativo + fallback de iniciais
- `size`: `'xs' | 'sm' | 'md' | 'lg' | 'xl' | '2xl'`
- `shape`: `'circle' | 'square'`
- `status`: `'online' | 'offline' | 'busy' | 'away'` — badge de status

**Detalhe de qualidade:**
- Fallback em cascata: imagem → iniciais (extraídas do `alt`) → ícone genérico
- Iniciais: máximo 2 letras, cor de fundo determinística pelo nome (hash da string → hue)
- Status badge: bolinha colorida posicionada no canto inferior direito
- AvatarGroup: empilhamento horizontal com overlap e badge de "+N"

---

## Tooltip

**Props:**
- `content`: string — texto do tooltip
- `placement`: `'top' | 'right' | 'bottom' | 'left'` + variantes `*-start`, `*-end`
- `delay`: number — ms de delay para aparecer (default 300)
- `disabled`: boolean

**Slots:** `default` (trigger), `content` (override para HTML rico)

**Detalhe de qualidade:**
- Posicionamento automático: se não cabe no `placement` pedido, inverte (flip)
- Arrow apontando para o elemento trigger
- Não fecha ao mover o mouse entre trigger e tooltip (safe triangle)
- `role="tooltip"` + `aria-describedby` no trigger

---

## Tag / Chip

Semelhante ao Badge mas com foco em input interativo:

**Props:**
- `label`: string
- `removable`: boolean
- `selected`: boolean — estado selecionado (para filtros)
- `disabled`: boolean
- `intent`: `'default' | 'primary' | 'success' | 'warning' | 'danger'`

**Emits:** `remove`, `click`

---

# Referência — Navigation

## Breadcrumb

**Props:**
- `items`: `Array<{ label: string, href?: string, icon?: string }>`
- `separator`: `'/' | '>' | '›' | Component`
- `max-items`: number — colapsa itens do meio com "..." quando muitos

**Detalhe de qualidade:**
- Último item não é link (apenas texto)
- Items colapsados: ao clicar "...", expande todos
- `aria-label="Breadcrumb"` no `<nav>`, `aria-current="page"` no último item

---

## Pagination

**Props:**
- `meta`: `{ current_page, last_page, per_page, total, from, to }` — formato Laravel
- `page-sizes`: `number[]` — default `[15, 25, 50, 100]`
- `show-info`: boolean — exibe "Mostrando X-Y de Z" (default true)
- `siblings`: number — páginas ao redor da atual (default 1)

**Emits:** `change` com `{ page: number, perPage: number }`

**Detalhe de qualidade:**
- Sempre exibe primeira e última página, com "..." quando há gap
- Botões anterior/próximo desabilitados nas extremidades
- Page size select à direita
- Info "Mostrando 1-15 de 143 registros" à esquerda
- Em mobile: simplifica para "Página X de Y" + prev/next

---

## Stepper

**Props:**
- `modelValue`: number — step atual (0-indexed)
- `steps`: `Array<{ label: string, description?: string, icon?: string }>`
- `orientation`: `'horizontal' | 'vertical'`
- `variant`: `'default' | 'simple' | 'circles'`
- `linear`: boolean — bloqueia pular steps (default true)

**Emits:** `update:modelValue`, `complete`

**Slots:** `step-{index}` para conteúdo de cada step, `step-icon-{index}`

**Detalhe de qualidade:**
- Steps anteriores: ícone de check + cor de sucesso
- Step atual: destaque visual, numerado ou com ícone
- Steps futuros: cor muted, desabilitados se `linear`
- Linha conectora entre steps: progresso preenchido proporcional ao step atual
- Navegação: botões Anterior/Próximo gerados pelo Stepper ou via slot `actions`