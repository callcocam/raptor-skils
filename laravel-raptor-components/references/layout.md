# Referência — Layout

## Card

**Anatomia:**
```
┌────────────────────────────────┐
│ [icon] Título   [header-action]│  ← header (opcional)
│ Subtítulo                      │
├────────────────────────────────┤
│                                │
│  default slot                  │  ← body
│                                │
├────────────────────────────────┤
│ [footer-left]   [footer-right] │  ← footer (opcional)
└────────────────────────────────┘
```

**Props:**
- `title`: string
- `description`: string — subtítulo no header
- `variant`: `'default' | 'outline' | 'ghost' | 'elevated'`
- `padding`: `'none' | 'sm' | 'md' | 'lg'` — default `'md'`
- `loading`: boolean — skeleton do conteúdo
- `collapsible`: boolean — header clicável para colapsar o body
- `collapsed`: boolean — estado inicial colapsado

**Slots:** `default`, `header`, `header-icon`, `header-actions`, `footer`, `footer-left`, `footer-right`

**Detalhe de qualidade:**
- Variant `elevated`: `shadow-md hover:shadow-lg transition-shadow`
- Loading: skeleton animado no body com linhas de altura proporcional
- Collapsible: animação de altura via `<Transition>` + rotação do chevron

---

## Accordion

**Props do grupo (AccordionGroup):**
- `modelValue`: `string | string[] | null` — item(s) aberto(s)
- `multiple`: boolean — permite vários abertos (default false)
- `variant`: `'default' | 'bordered' | 'separated'`

**Props de cada item (AccordionItem):**
- `value`: string — identificador único
- `title`: string
- `description`: string — subtítulo no trigger
- `disabled`: boolean
- `icon`: string — nome do ícone (opcional)

**Slots por item:** `trigger` (override do header), `default` (conteúdo)

**Detalhe de qualidade:**
- Animação de abertura/fechamento via `max-height` ou `grid-rows` em CSS:
  ```css
  /* Técnica grid-rows — sem JS de altura */
  grid-template-rows: 0fr → 1fr
  ```
- `role="button"` + `aria-expanded` no trigger
- `role="region"` + `aria-labelledby` no painel
- Variant `separated`: cada item é um Card independente com gap entre eles

---

## Tabs

**Props do grupo (Tabs):**
- `modelValue`: string — tab ativa
- `variant`: `'line' | 'pill' | 'card' | 'enclosed'`
- `orientation`: `'horizontal' | 'vertical'`
- `size`: `'sm' | 'md' | 'lg'`

**Props de cada tab (TabItem):**
- `value`: string
- `label`: string
- `disabled`: boolean
- `badge`: string | number — badge ao lado do label

**Slots:** `tab-{value}` para conteúdo de cada aba; `tab-label-{value}` para label customizado

**Detalhe de qualidade:**
- Animação de underline/indicator deslizando entre tabs (não pisca, desliza)
- Navegação por teclado: setas esquerda/direita entre tabs, Home/End para primeira/última
- `role="tablist"` no wrapper, `role="tab"` em cada trigger, `role="tabpanel"` no conteúdo
- Tab com badge: o badge não conta como texto do tab para acessibilidade

---

## Divider

**Props:**
- `orientation`: `'horizontal' | 'vertical'` — default `'horizontal'`
- `variant`: `'solid' | 'dashed' | 'dotted'`
- `label`: string — texto centralizado na linha
- `align`: `'left' | 'center' | 'right'` — alinhamento do label

**Uso típico:**
```vue
<Divider label="ou" />
<Divider orientation="vertical" class="h-4" />
```