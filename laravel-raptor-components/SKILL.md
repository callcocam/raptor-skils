---
name: laravel-raptor-components
description: >
  Cria componentes Vue 3 profissionais e reutilizáveis a partir do nome do componente desejado
  ou de uma referência (imagem, HTML, código). Cobre todos os tipos: inputs (text, number,
  date, calendar, select, checkbox, radio, textarea), feedback (modal, dialog, toast, alert,
  badge), layout (card, panel, accordion, tabs), navegação (breadcrumb, menu, pagination,
  steps) e data display (table, list, avatar, tag, tooltip). Use esta skill SEMPRE que o
  usuário pedir para: "criar um componente", "fazer um input de", "preciso de um select",
  "cria um modal", "faz um card", "componente de calendario", "componente de tabela", ou
  qualquer pedido de criação de componente Vue isolado. Também acione quando o usuário
  mencionar nomes de componentes de UI como "DatePicker", "MultiSelect", "DataTable",
  "Combobox", "Drawer", "Stepper", "FileUpload", "RichTextEditor", "ColorPicker", etc.
  A skill sempre pergunta por uma referência visual ou de código antes de gerar, e sempre
  busca componentes existentes no projeto para manter consistência de padrão.
  Stack: Vue 3 + Composition API + TypeScript + TailwindCSS + Inertia.js.
---

# Laravel Raptor — Criação de Componentes Profissionais

## ⚠️ PRIMEIRA AÇÃO — Obrigatória antes de qualquer geração

**NÃO gere nenhum código** até executar este roteiro de perguntas na primeira mensagem:

```
Para criar um componente profissional que se encaixe perfeitamente no seu sistema, preciso de
algumas informações:

1. Você tem alguma referência para esse componente?
   Pode ser uma imagem/print, um trecho de HTML, um código Vue de outro projeto,
   ou um link de biblioteca (ex: shadcn, PrimeVue, Headless UI).
   Se não tiver nenhuma referência, tudo bem — vou criar com base nas melhores práticas.

2. Onde ficam os componentes existentes do projeto?
   (ex: resources/js/components/ui/, resources/js/components/)
   Vou analisar o padrão dos seus componentes atuais antes de criar o novo.

3. Onde devo salvar o componente gerado?
   (sugestão: mesma pasta dos componentes existentes)
```

Após receber as respostas:
- Se o usuário fornecer referência → analise na **Fase 1b**
- Se houver pasta de componentes → leia 2-3 arquivos existentes na **Fase 1c** para capturar o padrão
- Só então vá para a **Fase 2**

---

## Objetivo

Gerar componentes Vue 3 **prontos para produção**: com todos os estados, slots flexíveis,
variantes visuais, acessibilidade e TypeScript estrito — que pareçam ter sido feitos por
um time de design system experiente, não por um gerador genérico.

---

## Stack

- Vue 3 + Composition API (`<script setup lang="ts">`)
- TypeScript estrito — sem `any`, sem tipos implícitos
- TailwindCSS — classes utilitárias, zero CSS customizado salvo animações complexas
- Inertia.js quando relevante (formulários, navegação)
- Sem biblioteca de UI obrigatória — adaptar ao que o projeto já usa

---

## Fase 1 — Coleta e análise

### 1a. Identificar o componente solicitado

Mapeie o pedido do usuário para o tipo canônico:

| O usuário pediu | Tipo canônico | Referência |
|---|---|---|
| input texto, campo de texto | `InputText` | `references/inputs.md` |
| input número, campo numérico | `InputNumber` | `references/inputs.md` |
| calendário, date picker, data | `DatePicker` | `references/inputs.md` |
| select, dropdown, combobox | `SelectInput` | `references/inputs.md` |
| checkbox, toggle, switch | `CheckboxInput` | `references/inputs.md` |
| radio, opções | `RadioGroup` | `references/inputs.md` |
| textarea, área de texto | `TextareaInput` | `references/inputs.md` |
| upload, arquivo, imagem | `FileUpload` | `references/inputs.md` |
| modal, dialog, popup | `Modal` | `references/feedback.md` |
| toast, notificação, snackbar | `Toast` | `references/feedback.md` |
| alert, aviso, banner | `Alert` | `references/feedback.md` |
| badge, chip, tag | `Badge` | `references/feedback.md` |
| card, painel, container | `Card` | `references/layout.md` |
| accordion, colapsável | `Accordion` | `references/layout.md` |
| tabs, abas | `Tabs` | `references/layout.md` |
| tabela, lista de dados | `DataTable` | `references/display.md` |
| avatar, foto de perfil | `Avatar` | `references/display.md` |
| tooltip, dica | `Tooltip` | `references/display.md` |
| breadcrumb, caminho | `Breadcrumb` | `references/navigation.md` |
| paginação | `Pagination` | `references/navigation.md` |
| steps, stepper, etapas | `Stepper` | `references/navigation.md` |

Após identificar o tipo, leia o arquivo de referência correspondente antes de gerar.

### 1b. Analisar a referência fornecida (se houver)

**Referência em imagem/print:**
- Identifique todos os estados visuais presentes (normal, hover, focus, erro, disabled)
- Mapeie as regiões do componente (label, input, prefix, suffix, hint, error message)
- Observe espaçamentos, bordas, raio, cores — anote as classes Tailwind equivalentes
- Identifique animações ou transições presentes

**Referência em HTML/Vue:**
- Extraia a estrutura de tags e classes
- Identifique props, emits e slots já definidos
- Mantenha nomes consistentes com o que o usuário já usa
- Observe padrões de acessibilidade presentes (aria, role, tabindex)

**Referência em link de biblioteca:**
- Use a API pública da biblioteca como inspiração de contrato (props/emits/slots)
- NÃO copie código — reimplemente com o stack do projeto

### 1c. Capturar o padrão do sistema (se houver componentes existentes)

Leia 2-3 componentes existentes na pasta informada e extraia:
- Estrutura do `<script setup>`: como definem props, emits, computed
- Padrão de classes Tailwind: usa `cn()`, template literals, objeto de classes?
- Convenção de nomes: PascalCase para componentes, camelCase para props?
- Como tratam estados: classes condicionais, `v-bind:class`, `cva()`?
- Importações comuns: têm um `useToast`, `useModal`, composable próprio?

**Documente o padrão encontrado antes de gerar** e siga-o à risca no novo componente.

---

## Fase 2 — O que torna um componente profissional

Antes de gerar qualquer linha de código, planeje estes aspectos para o componente em questão:

### 2a. Estados completos

Todo componente deve ter implementação visual distinta para:

```
default → hover → focus → active → disabled → loading → error → success
```

Não deixe nenhum estado sem estilo. Use classes Tailwind condicionais:

```ts
// Padrão recomendado com objeto de classes
const inputClasses = computed(() => ({
  'border-input bg-background': !props.error && !props.disabled,
  'border-destructive bg-destructive/5': !!props.error,
  'opacity-50 cursor-not-allowed pointer-events-none': props.disabled,
  'ring-2 ring-ring ring-offset-2': isFocused.value,
}))
```

### 2b. Slots flexíveis — anatomia completa

Pense no componente como uma anatomia de regiões, cada uma com um slot:

**Para inputs:**
```
[label]
[prefix-icon] [input-content] [suffix-icon] [clear-button]
[hint-text] ou [error-message]
```

**Para cards:**
```
[header-icon] [title] [subtitle] [header-actions]
[content / default]
[footer-left] [footer-right]
```

**Para modais:**
```
[icon] [title] [close-button]
[content / default]
[footer-left] [footer-right / actions]
```

Regra: **se é uma região visual distinta, é um slot**. Nunca force conteúdo via prop string
quando um slot seria mais flexível.

### 2c. Variantes visuais

Defina sempre pelo menos:

```ts
// Size
type Size = 'xs' | 'sm' | 'md' | 'lg' | 'xl'

// Variant (visual style)
type Variant = 'default' | 'outline' | 'ghost' | 'filled'

// Color / intent (quando aplicável)
type Intent = 'default' | 'primary' | 'success' | 'warning' | 'danger'
```

Implemente as variantes com um mapa de classes — nunca com if/else encadeado:

```ts
const sizeClasses: Record<Size, string> = {
  xs: 'h-6 px-2 text-xs',
  sm: 'h-8 px-3 text-sm',
  md: 'h-10 px-4 text-sm',
  lg: 'h-11 px-5 text-base',
  xl: 'h-12 px-6 text-base',
}
```

### 2d. Acessibilidade — checklist mínimo

- [ ] `id` único gerado automaticamente se não passado via prop (`useId()` ou `crypto.randomUUID()`)
- [ ] `<label>` sempre associado ao input via `for` / `id`
- [ ] `aria-describedby` apontando para o hint e/ou error message
- [ ] `aria-invalid="true"` quando em estado de erro
- [ ] `aria-disabled="true"` + `tabindex="-1"` quando disabled
- [ ] `aria-label` ou `aria-labelledby` em ícones e botões sem texto
- [ ] Navegação por teclado: `Tab`, `Enter`, `Escape`, setas — onde aplicável
- [ ] `role` correto: `dialog` para modais, `alert` para erros, `status` para loading
- [ ] Cores não são o único indicador de estado (usar ícone ou texto junto)

### 2e. Animações e transições

Use `<Transition>` do Vue com classes Tailwind para:
- Aparecer/desaparecer de modais, dropdowns, tooltips
- Mudança de estado (loading spinner, check de sucesso)
- Feedback de erro (shake suave no input)

```vue
<Transition
  enter-active-class="transition duration-200 ease-out"
  enter-from-class="opacity-0 translate-y-1"
  enter-to-class="opacity-100 translate-y-0"
  leave-active-class="transition duration-150 ease-in"
  leave-from-class="opacity-100 translate-y-0"
  leave-to-class="opacity-0 translate-y-1"
>
  <div v-if="isOpen">...</div>
</Transition>
```

### 2f. Composable dedicado (quando necessário)

Componentes com lógica stateful complexa devem ter um composable par:

| Componente | Composable |
|---|---|
| `Modal.vue` | `useModal.ts` |
| `Toast.vue` | `useToast.ts` |
| `DataTable.vue` | `useDataTable.ts` |
| `DatePicker.vue` | `useDatePicker.ts` |
| `FileUpload.vue` | `useFileUpload.ts` |

Componentes simples (Badge, Avatar, Alert) não precisam de composable.

---

## Fase 3 — Estrutura do arquivo gerado

### Template padrão de um componente profissional

```vue
<script setup lang="ts">
// 1. Imports externos
import { computed, ref, useId } from 'vue'

// 2. Types locais
type Size = 'sm' | 'md' | 'lg'
type Variant = 'default' | 'outline' | 'ghost'

// 3. Props com tipos explícitos e defaults documentados
interface Props {
  /** Valor atual do campo */
  modelValue?: string
  /** Label exibido acima do campo */
  label?: string
  /** Texto de placeholder */
  placeholder?: string
  /** Mensagem de erro — ativa o estado de erro quando presente */
  error?: string
  /** Texto auxiliar abaixo do campo */
  hint?: string
  /** Desabilita o campo completamente */
  disabled?: boolean
  /** Exibe spinner de carregamento */
  loading?: boolean
  /** Tamanho do componente */
  size?: Size
  /** Variante visual */
  variant?: Variant
}

const props = withDefaults(defineProps<Props>(), {
  modelValue: '',
  disabled: false,
  loading: false,
  size: 'md',
  variant: 'default',
})

// 4. Emits tipados
const emit = defineEmits<{
  'update:modelValue': [value: string]
  'focus': [event: FocusEvent]
  'blur': [event: FocusEvent]
  'clear': []
}>()

// 5. ID único para acessibilidade
const id = useId()
const hintId = computed(() => `${id}-hint`)
const errorId = computed(() => `${id}-error`)

// 6. Estado interno
const isFocused = ref(false)

// 7. Classes computadas
const rootClasses = computed(() => [
  // base
  'relative flex items-center w-full rounded-md border transition-colors',
  // size
  { 'h-8 px-3 text-sm': props.size === 'sm' },
  { 'h-10 px-4 text-sm': props.size === 'md' },
  { 'h-11 px-4 text-base': props.size === 'lg' },
  // state
  { 'border-destructive bg-destructive/5': !!props.error },
  { 'border-input bg-background hover:border-ring': !props.error && !props.disabled },
  { 'opacity-50 cursor-not-allowed': props.disabled },
  { 'ring-2 ring-ring ring-offset-background ring-offset-2': isFocused.value && !props.error },
])

// 8. Handlers
function onFocus(event: FocusEvent) {
  isFocused.value = true
  emit('focus', event)
}

function onBlur(event: FocusEvent) {
  isFocused.value = false
  emit('blur', event)
}
</script>

<template>
  <div class="flex flex-col gap-1.5">
    <!-- Label -->
    <label
      v-if="label"
      :for="id"
      class="text-sm font-medium leading-none"
      :class="{ 'text-destructive': error, 'text-muted-foreground': disabled }"
    >
      {{ label }}
      <span v-if="required" class="text-destructive ml-0.5" aria-hidden="true">*</span>
    </label>

    <!-- Input wrapper -->
    <div :class="rootClasses">
      <!-- Slot prefix (ícone, texto, moeda...) -->
      <slot name="prefix" />

      <input
        :id="id"
        v-bind="$attrs"
        :value="modelValue"
        :disabled="disabled || loading"
        :aria-invalid="!!error"
        :aria-describedby="[hint ? hintId : '', error ? errorId : ''].filter(Boolean).join(' ') || undefined"
        class="flex-1 bg-transparent outline-none placeholder:text-muted-foreground"
        @input="emit('update:modelValue', ($event.target as HTMLInputElement).value)"
        @focus="onFocus"
        @blur="onBlur"
      />

      <!-- Loading spinner -->
      <svg v-if="loading" class="h-4 w-4 animate-spin text-muted-foreground" ... />

      <!-- Clear button -->
      <button
        v-if="modelValue && !disabled && !loading"
        type="button"
        class="text-muted-foreground hover:text-foreground"
        aria-label="Limpar campo"
        @click="emit('update:modelValue', ''); emit('clear')"
      >
        ×
      </button>

      <!-- Slot suffix (ícone, unidade...) -->
      <slot name="suffix" />
    </div>

    <!-- Error message -->
    <p v-if="error" :id="errorId" class="text-sm text-destructive flex items-center gap-1" role="alert">
      <slot name="error-icon"><span aria-hidden="true">⚠</span></slot>
      {{ error }}
    </p>

    <!-- Hint text -->
    <p v-else-if="hint" :id="hintId" class="text-sm text-muted-foreground">
      {{ hint }}
    </p>
  </div>
</template>
```

---

## Fase 4 — Documentação do componente

Para cada componente gerado, produza documentação em markdown:

```markdown
## NomeDoComponente

**Localização:** `resources/js/components/ui/NomeDoComponente.vue`
**Composable par:** `resources/js/composables/useNomeDoComponente.ts` (se existir)

**Propósito:** [uma frase]

### Props
| Prop | Tipo | Default | Descrição |
|------|------|---------|-----------|
| modelValue | string | `''` | Valor controlado via v-model |
| label | string | — | Label do campo |
| error | string | — | Mensagem de erro; ativa estado de erro |
| disabled | boolean | `false` | Desabilita interação |
| size | 'sm' \| 'md' \| 'lg' | `'md'` | Tamanho visual |

### Slots
| Slot | Descrição |
|------|-----------|
| prefix | Conteúdo antes do input (ícone, moeda) |
| suffix | Conteúdo após o input (unidade, ícone) |
| error-icon | Ícone customizado na mensagem de erro |

### Emits
| Evento | Payload | Descrição |
|--------|---------|-----------|
| update:modelValue | `string` | Novo valor |
| clear | — | Campo foi limpo |
| focus | `FocusEvent` | Campo recebeu foco |
| blur | `FocusEvent` | Campo perdeu foco |

### Exemplos de uso

**Básico:**
\`\`\`vue
<InputText v-model="name" label="Nome" placeholder="Digite seu nome" />
\`\`\`

**Com erro:**
\`\`\`vue
<InputText v-model="email" label="E-mail" :error="form.errors.email" />
\`\`\`

**Com slots:**
\`\`\`vue
<InputText v-model="price" label="Preço">
  <template #prefix>R$</template>
  <template #suffix>
    <InfoIcon class="h-4 w-4 text-muted-foreground" />
  </template>
</InputText>
\`\`\`
```

---

## Fase 5 — Checklist de qualidade antes de entregar

### TypeScript
- [ ] Zero usos de `any`
- [ ] Todos os tipos de prop explícitos com JSDoc nos campos importantes
- [ ] Emits com tipos completos `emit<{ evento: [payload] }>`
- [ ] Computed types corretos (sem `as` desnecessário)

### Estados
- [ ] `default` estilizado
- [ ] `hover` com `hover:` classes
- [ ] `focus` com ring/outline visível
- [ ] `disabled` com opacidade + `cursor-not-allowed`
- [ ] `error` com cor destrutiva em borda, label e mensagem
- [ ] `loading` com spinner e interação bloqueada

### Acessibilidade
- [ ] `id` único no input, `for` no label
- [ ] `aria-invalid` quando em erro
- [ ] `aria-describedby` para hint e error
- [ ] Botões sem texto têm `aria-label`
- [ ] Modal tem `role="dialog"` e `aria-modal="true"`

### Slots e flexibilidade
- [ ] Todas as regiões visuais têm slot correspondente
- [ ] Nenhum conteúdo hardcoded que deveria ser slot
- [ ] `v-bind="$attrs"` no elemento raiz ou no input (não no wrapper)

### Tailwind
- [ ] Sem valores arbitrários desnecessários (`w-[347px]`)
- [ ] Responsividade considerada onde faz sentido
- [ ] Dark mode com `dark:` se o projeto usar

---

## Referências por categoria

- `references/inputs.md` — InputText, InputNumber, DatePicker, Select, Checkbox, Radio, Textarea, FileUpload
- `references/feedback.md` — Modal, Toast, Alert, Badge
- `references/layout.md` — Card, Accordion, Tabs, Divider
- `references/display-navigation.md` — DataTable, Avatar, Tooltip, Tag, Breadcrumb, Pagination, Stepper