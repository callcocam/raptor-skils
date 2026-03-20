# Referência — Inputs

## Anatomia universal de um input

```
┌─────────────────────────────────────────┐
│ Label                          (opcional)│
│ ┌─────────────────────────────────────┐ │
│ │ [prefix] [valor/placeholder] [clear]│ │  ← wrapper com borda
│ │          [suffix]                   │ │
│ └─────────────────────────────────────┘ │
│ Hint text ou Error message    (opcional) │
└─────────────────────────────────────────┘
```

Slots: `prefix`, `suffix`, `label` (override), `hint` (override), `error-icon`

---

## InputText

**Props específicas:**
- `type`: `'text' | 'email' | 'password' | 'search' | 'url' | 'tel'` — default `'text'`
- `maxlength`: number — exibe contador de caracteres no suffix quando definido
- `clearable`: boolean — exibe botão X quando há valor

**Detalhe de qualidade — campo de senha:**
- Botão de toggle visibilidade (olho aberto/fechado)
- Nunca usar `autocomplete="off"` — deixar o browser gerenciar
- `type` alterna entre `'password'` e `'text'` via estado interno

**Detalhe de qualidade — campo de busca:**
- Ícone de lupa no prefix por padrão
- Debounce de 300ms no emit quando `search-debounce` prop ativa
- Botão clear quando há valor

---

## InputNumber

**Props específicas:**
- `min`, `max`, `step`: number
- `precision`: number — casas decimais (default 0)
- `prefix-text`: string — ex: `'R$'`, `'US$'`
- `suffix-text`: string — ex: `'kg'`, `'%'`, `'un'`
- `controls`: boolean — botões +/- nas laterais (default false)
- `format`: `'decimal' | 'currency' | 'percent'`

**Detalhe de qualidade:**
- Formatar exibição com `Intl.NumberFormat` baseado no `format` prop
- Aceitar apenas dígitos + separador decimal no input
- Scroll do mouse incrementa/decrementa quando o campo está focado
- Botões +/- com repeat automático quando mantidos pressionados

---

## DatePicker

**Props específicas:**
- `modelValue`: `string | Date | null` — formato ISO 8601
- `mode`: `'date' | 'datetime' | 'month' | 'year' | 'range'`
- `min-date`, `max-date`: `Date | string`
- `disabled-dates`: `Date[] | ((date: Date) => boolean)`
- `locale`: string — default `'pt-BR'`
- `format`: string — ex: `'DD/MM/YYYY'`
- `placeholder`: string — default `'Selecione uma data'`

**Detalhe de qualidade:**
- Exibir sempre no formato local (`DD/MM/YYYY` para pt-BR)
- Armazenar/emitir sempre em ISO 8601
- Navegação por teclado: setas movem dias, PgUp/PgDown mudam mês
- Fechar ao selecionar (mode=date) ou ao clicar fora
- Em mobile: usar `<input type="date">` nativo como fallback

**Estrutura interna sugerida:**
```
InputText (display) → abre → Popover → DatePickerCalendar
```

---

## SelectInput

**Props específicas:**
- `options`: `Array<{ label: string, value: any, disabled?: boolean, group?: string }>`
- `multiple`: boolean — seleção múltipla com chips
- `searchable`: boolean — filtro de opções por texto
- `clearable`: boolean
- `placeholder`: string
- `empty-text`: string — texto quando não há opções (default: 'Nenhuma opção encontrada')
- `loading`: boolean — exibe spinner enquanto carrega opções remotas

**Detalhe de qualidade:**
- Grouping: se `group` presente nas options, exibe separadores de grupo
- Multiple: chips removíveis no campo, badge de contagem quando há muitos selecionados
- Searchable: input de busca dentro do dropdown, highlight do texto encontrado
- Virtual scroll se lista > 100 itens
- Navegação por teclado: setas, Enter para selecionar, Escape para fechar

---

## CheckboxInput

**Props específicas:**
- `modelValue`: `boolean | any[]` — array quando em grupo
- `value`: any — valor do item no array (para grupos)
- `indeterminate`: boolean — estado parcial (ex: "selecionar todos" com alguns marcados)
- `label`: string
- `description`: string — texto auxiliar abaixo do label

**Detalhe de qualidade:**
- Estado `indeterminate` via `el.indeterminate = true` na ref do input
- Animação de check ao marcar
- Área de clique cobre label + description, não só o quadrado

---

## RadioGroup

**Props específicas:**
- `modelValue`: any
- `options`: `Array<{ label: string, value: any, description?: string, disabled?: boolean }>`
- `orientation`: `'vertical' | 'horizontal'` — default `'vertical'`
- `name`: string — gerado automaticamente se não informado

---

## TextareaInput

**Props específicas:**
- `rows`: number — default 3
- `max-rows`: number — auto-resize até este limite
- `auto-resize`: boolean — cresce com o conteúdo
- `maxlength`: number — contador de caracteres
- `resize`: `'none' | 'vertical' | 'both'` — default `'none'` quando auto-resize ativo

**Detalhe de qualidade:**
- Auto-resize via `scrollHeight`: ajustar `height` no input event
- Contador de caracteres no canto inferior direito: `atual / máximo`
- Animação suave no resize

---

## FileUpload

**Props específicas:**
- `accept`: string — ex: `'image/*'`, `'.pdf,.docx'`
- `multiple`: boolean
- `max-size`: number — em bytes
- `max-files`: number
- `preview`: boolean — preview de imagens selecionadas
- `drag-drop`: boolean — área de drag and drop (default true)

**Detalhe de qualidade:**
- Drag over: highlight visual da área
- Validação imediata de tipo e tamanho com mensagem de erro por arquivo
- Preview com thumbnail para imagens, ícone de tipo para outros
- Botão de remoção individual em cada arquivo
- Barra de progresso de upload (via emit de progresso externo)
- Estado de loading por arquivo individual