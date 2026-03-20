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
- **Antes de qualquer geração**, verificar se o componente e o composable já existem:
  ```
  # Verificar componente
  find {pasta_informada} -name "NomeDoComponente.vue" 2>/dev/null

  # Verificar composable
  find {pasta_composables} -name "useNomeDoComponente.ts" 2>/dev/null

  # Verificar useFieldWatcher especificamente
  find {pasta_composables} -name "useFieldWatcher.ts" 2>/dev/null
  ```
  - **Se já existe** → leia o arquivo, mostre ao usuário o que foi encontrado e pergunte:
    *"Encontrei `NomeDoComponente.vue` em `{caminho}`. Deseja que eu atualize o existente
    ou crie uma nova versão?"*
  - **Se não existe** → prossiga normalmente para a geração
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

## Anatomia Universal — Aplicável a TODOS os componentes

**Todo componente gerado por esta skill, independente do tipo, deve suportar estas capacidades.**
Não são opcionais — são o padrão mínimo que eleva o componente ao nível profissional.
Implemente todas que fizerem sentido para o tipo em questão; documente explicitamente as que
foram omitidas e o motivo.

---

### 1. Prefix e Suffix — com suporte a ações

Todo componente que tem uma área de input, trigger ou display deve ter slots `prefix` e `suffix`.
Esses slots **não são decorativos** — eles podem conter elementos interativos: botões, ícones
clicáveis, selects, spinners, contadores.

```vue
<!-- Slots obrigatórios em todo componente com campo/trigger -->
<slot name="prefix" />   <!-- antes do conteúdo principal -->
<slot name="suffix" />   <!-- depois do conteúdo principal -->
```

**Exemplos de uso real:**
```vue
<!-- Prefix com ícone estático -->
<InputText v-model="search">
  <template #prefix><SearchIcon class="h-4 w-4 text-muted-foreground" /></template>
</InputText>

<!-- Suffix com botão de ação -->
<InputText v-model="url">
  <template #suffix>
    <button @click="copyToClipboard(url)" class="hover:text-primary">
      <CopyIcon class="h-4 w-4" />
    </button>
  </template>
</InputText>

<!-- Prefix com select (ex: DDI no telefone) -->
<InputText v-model="phone" type="tel">
  <template #prefix>
    <select v-model="ddi" class="border-0 bg-transparent text-sm pr-1">
      <option value="+55">🇧🇷 +55</option>
      <option value="+1">🇺🇸 +1</option>
    </select>
  </template>
</InputText>

<!-- Suffix com múltiplos ícones de ação -->
<InputText v-model="password" type="password">
  <template #suffix>
    <button @click="toggleVisible" :aria-label="visible ? 'Ocultar senha' : 'Mostrar senha'">
      <EyeIcon v-if="!visible" class="h-4 w-4" />
      <EyeOffIcon v-else class="h-4 w-4" />
    </button>
  </template>
</InputText>
```

**Regras de implementação:**
- O `prefix` e `suffix` ficam **dentro** da borda do componente, alinhados verticalmente ao centro
- Nunca aplicar padding no wrapper quando o slot estiver preenchido — o slot é responsável pelo seu próprio espaçamento
- Ícones dentro dos slots devem herdar a cor via `currentColor` (não hardcode)
- Botões dentro dos slots devem ter `type="button"` para não submeter forms

---

### 2. Botão de Ação (Action Button)

Além do prefix/suffix, todo componente pode ter um **botão de ação primário** associado —
separado visualmente do campo, mas parte do componente. É diferente do suffix: o action button
fica **fora** da borda, ao lado direito do componente.

```vue
<!-- Slot action — fora da borda, ao lado do componente -->
<slot name="action" />
```

**Prop auxiliar para casos simples:**
```ts
interface Props {
  /** Texto do botão de ação — renderiza botão padrão sem precisar de slot */
  actionLabel?: string
  /** Variante do botão de ação */
  actionVariant?: 'default' | 'outline' | 'ghost'
  /** Loading state do botão de ação */
  actionLoading?: boolean
}
```

**Emit correspondente:**
```ts
const emit = defineEmits<{
  'action': []  // disparado ao clicar no botão de ação
}>()
```

**Exemplos de uso:**
```vue
<!-- Prop simples -->
<InputText v-model="coupon" label="Cupom" action-label="Aplicar" @action="applyCoupon" />

<!-- Slot para ação customizada -->
<InputText v-model="cep" label="CEP">
  <template #action>
    <button @click="searchCep" class="px-3 py-2 border rounded-md text-sm">
      <SearchIcon class="h-4 w-4" />
    </button>
  </template>
</InputText>

<!-- Card com ação no header -->
<Card title="Produtos">
  <template #header-actions>
    <button @click="addProduct">+ Adicionar</button>
  </template>
</Card>
```

---

### 3. Abertura de Modal

Todo componente que dispara uma ação, exibe um item ou precisa de confirmação deve suportar
abertura de modal de forma nativa, sem exigir que o pai gerencie isso manualmente.

**Padrão: prop `modal` + composable `useModal`**

```ts
interface Props {
  /** Quando true, o clique no componente abre um modal em vez de emitir evento direto */
  modal?: boolean
  /** Título do modal (quando modal=true) */
  modalTitle?: string
  /** Tamanho do modal */
  modalSize?: 'sm' | 'md' | 'lg' | 'xl'
}
```

**Slot para conteúdo do modal:**
```vue
<slot name="modal-content" />   <!-- conteúdo renderizado dentro do modal -->
<slot name="modal-footer" />    <!-- ações do modal (confirmar, cancelar) -->
```

**Exemplo — input que abre modal de busca avançada:**
```vue
<SelectInput v-model="product" modal modal-title="Selecionar Produto" modal-size="lg">
  <template #modal-content>
    <ProductSearchTable @select="onProductSelect" />
  </template>
</SelectInput>
```

**Exemplo — card que abre modal ao clicar:**
```vue
<Card title="Ver detalhes" modal modal-title="Detalhes do Pedido">
  <template #modal-content>
    <OrderDetails :order="order" />
  </template>
</Card>
```

**Implementação interna:**
```vue
<template>
  <!-- O componente principal -->
  <div @click="modal ? openModal() : emit('click')">...</div>

  <!-- Modal integrado via Teleport -->
  <Teleport to="body">
    <Modal v-model="isModalOpen" :title="modalTitle" :size="modalSize">
      <slot name="modal-content" />
      <template #footer>
        <slot name="modal-footer">
          <button @click="closeModal">Fechar</button>
        </slot>
      </template>
    </Modal>
  </Teleport>
</template>
```

---

### 4. Help Text (Texto de Ajuda)

Todo componente deve ter suporte a texto de ajuda contextual — exibido abaixo do componente,
distinto da mensagem de erro.

**Props:**
```ts
interface Props {
  /** Texto de ajuda exibido abaixo do componente */
  hint?: string
  /** Quando true, exibe ícone de interrogação clicável que mostra o hint em tooltip */
  hintTooltip?: boolean
}
```

**Slot para hint rico (com HTML, links, etc.):**
```vue
<slot name="hint" />
```

**Lógica de exibição — prioridade:**
```
error > hint
```
Nunca exibir hint e error ao mesmo tempo — o erro tem prioridade absoluta.

**Variantes de hint:**

```vue
<!-- Hint simples abaixo do campo -->
<InputText v-model="cnpj" label="CNPJ" hint="Somente números, sem pontuação" />

<!-- Hint como tooltip no label (ícone ?) -->
<InputText v-model="slug" label="Slug" hint="URL amigável gerada automaticamente" hint-tooltip />

<!-- Hint rico via slot -->
<InputText v-model="password" label="Senha">
  <template #hint>
    A senha deve ter pelo menos 8 caracteres,
    <a href="/politica" class="underline">veja nossa política</a>.
  </template>
</InputText>
```

**Implementação:**
```vue
<!-- Hierarquia de exibição -->
<p v-if="error" :id="errorId" role="alert" class="text-sm text-destructive flex gap-1 items-center">
  <slot name="error-icon"><TriangleAlertIcon class="h-3.5 w-3.5 shrink-0" /></slot>
  {{ error }}
</p>
<div v-else-if="$slots.hint || hint" :id="hintId" class="text-sm text-muted-foreground">
  <slot name="hint">{{ hint }}</slot>
</div>
```

---

### 5. Feedback de Erros

Todo componente deve ter um sistema de feedback de erro completo — não apenas exibir uma
string vermelha, mas comunicar o problema de forma clara, acessível e acionável.

**Props:**
```ts
interface Props {
  /** Mensagem de erro — ativa estado de erro quando presente */
  error?: string
  /** Array de erros — quando há múltiplos problemas */
  errors?: string[]
  /** Quando true, exibe o erro como tooltip em vez de texto abaixo */
  errorTooltip?: boolean
}
```

**Estados visuais obrigatórios quando `error` presente:**
```
borda → destructive color
label → destructive color
ícone de erro → aparece no suffix (por padrão) ou prefix
texto de erro → abaixo do componente, em destructive color
fundo → leve tint de destructive (bg-destructive/5)
```

**Slot para ícone de erro customizado:**
```vue
<slot name="error-icon">
  <!-- fallback padrão -->
  <TriangleAlertIcon class="h-3.5 w-3.5 shrink-0" />
</slot>
```

**Múltiplos erros (ex: validação complexa):**
```vue
<!-- errors prop com array -->
<PasswordInput v-model="password" :errors="['Mínimo 8 caracteres', 'Requer número', 'Requer símbolo']" />

<!-- Renderização como lista -->
<ul v-if="errors?.length" role="alert" class="text-sm text-destructive space-y-0.5">
  <li v-for="err in errors" :key="err" class="flex gap-1 items-center">
    <TriangleAlertIcon class="h-3 w-3 shrink-0" />
    {{ err }}
  </li>
</ul>
```

**Integração com Inertia (formulários Laravel):**
```vue
<!-- O componente aceita diretamente o objeto de erros do Inertia -->
<InputText
  v-model="form.email"
  label="E-mail"
  :error="form.errors.email"
/>

<!-- Para múltiplos erros do backend -->
<InputText
  v-model="form.tags"
  label="Tags"
  :errors="form.errors['tags'] ? [form.errors['tags']] : []"
/>
```

**Acessibilidade obrigatória para erros:**
- `aria-invalid="true"` no elemento interativo quando `error` presente
- `aria-describedby` apontando para o `id` do elemento de erro
- `role="alert"` na mensagem de erro (anuncia para leitores de tela automaticamente)
- A mensagem de erro deve ser suficientemente descritiva — não apenas "Campo inválido"

---

### Checklist universal — verificar em TODO componente gerado

Antes de entregar qualquer componente, confirme:

- [ ] Slot `prefix` implementado (com suporte a ícone e botão de ação)
- [ ] Slot `suffix` implementado (com suporte a ícone e botão de ação)
- [ ] Slot `action` ou prop `action-label` + emit `action` implementados
- [ ] Prop `modal` + slot `modal-content` implementados via Teleport
- [ ] Prop `hint` + slot `hint` implementados
- [ ] Prop `hint-tooltip` implementado (ícone ? no label)
- [ ] Prop `error` implementada com todos os estados visuais (borda, label, fundo, ícone, texto)
- [ ] Prop `errors` (array) para múltiplos erros
- [ ] Slot `error-icon` para customização do ícone de erro
- [ ] `aria-invalid`, `aria-describedby`, `role="alert"` presentes
- [ ] `hint` e `error` nunca exibidos simultaneamente (error tem prioridade)

---

## Composable: `useFieldWatcher` — Campos Reativos

Todo componente gerado deve ser compatível com o sistema de escuta reativa entre campos.
Quando o usuário descrever um comportamento do tipo **"ao mudar X, atualiza Y"**, gere
o composable `useFieldWatcher` e demonstre como conectá-lo aos componentes.

### O problema que resolve

```
Usuário seleciona produto  →  price carrega automaticamente
price ou quantidade muda   →  total é recalculado
Em um repeater com N linhas, cada linha tem seu próprio contexto isolado
```

### Composable `useFieldWatcher`

```ts
// composables/useFieldWatcher.ts

import { watch, type Ref, type WatchStopHandle } from 'vue'

type FieldPath = string  // 'product_id' | 'items.2.product_id' | 'items[2].product_id'
type WatcherCallback<T = any> = (newValue: T, oldValue: T, context: WatcherContext) => void | Promise<void>

interface WatcherContext {
  /** Caminho completo do campo que disparou */
  field: FieldPath
  /** Index da linha se estiver dentro de um repeater (null se não) */
  index: number | null
  /** Chave composta para acesso ao campo: items[2].product_id */
  key: string
  /** Função para setar outro campo no mesmo contexto */
  set: (field: FieldPath, value: any) => void
  /** Função para pegar o valor de outro campo no mesmo contexto */
  get: (field: FieldPath) => any
}

interface FieldWatcherOptions {
  /** O objeto reativo completo do formulário (Inertia useForm ou reactive/ref) */
  form: Record<string, any>
  /** Prefixo do contexto — para campos dentro de repeater: 'items' */
  scope?: string
  /** Index da linha — para campos dentro de repeater */
  index?: number | Ref<number>
}

export function useFieldWatcher(options: FieldWatcherOptions) {
  const { form, scope, index } = options
  const stopHandles: WatchStopHandle[] = []

  /** Resolve o caminho completo do campo considerando scope e index */
  function resolvePath(field: FieldPath): string {
    if (scope === undefined) return field
    const idx = typeof index === 'object' ? index.value : index
    return idx !== undefined ? `${scope}[${idx}].${field}` : `${scope}.${field}`
  }

  /** Acessa um valor aninhado via dot/bracket notation */
  function getNestedValue(obj: any, path: string): any {
    return path
      .replace(/\[(\d+)\]/g, '.$1')
      .split('.')
      .reduce((acc, key) => acc?.[key], obj)
  }

  /** Seta um valor aninhado via dot/bracket notation */
  function setNestedValue(obj: any, path: string, value: any): void {
    const keys = path.replace(/\[(\d+)\]/g, '.$1').split('.')
    const last = keys.pop()!
    const target = keys.reduce((acc, key) => acc?.[key], obj)
    if (target && last) target[last] = value
  }

  /** Cria o contexto passado para o callback */
  function buildContext(field: FieldPath): WatcherContext {
    const idx = typeof index === 'object' ? index.value : index
    const resolvedPath = resolvePath(field)
    return {
      field,
      index: idx ?? null,
      key: resolvedPath,
      set: (targetField, value) => setNestedValue(form, resolvePath(targetField), value),
      get: (targetField) => getNestedValue(form, resolvePath(targetField)),
    }
  }

  /**
   * Registra um watcher para um campo.
   * @param field  - nome do campo a observar (relativo ao scope)
   * @param cb     - callback executado quando o campo muda
   */
  function watch_field<T = any>(field: FieldPath, cb: WatcherCallback<T>): void {
    const resolvedPath = resolvePath(field)
    const stop = watch(
      () => getNestedValue(form, resolvedPath),
      (newVal, oldVal) => cb(newVal, oldVal, buildContext(field)),
      { deep: true }
    )
    stopHandles.push(stop)
  }

  /** Para todos os watchers registrados */
  function stop(): void {
    stopHandles.forEach(fn => fn())
    stopHandles.length = 0
  }

  return { watch: watch_field, stop }
}
```

---

### Uso em formulário simples (sem repeater)

```ts
// Pages/Orders/Create.vue
import { useForm } from '@inertiajs/vue3'
import { useFieldWatcher } from '@/composables/useFieldWatcher'
import { onUnmounted } from 'vue'

const form = useForm({
  product_id: null,
  price: 0,
  quantity: 1,
  total: 0,
})

const { watch, stop } = useFieldWatcher({ form })

// Ao selecionar produto → carrega o preço via API
watch('product_id', async (productId, _, ctx) => {
  if (!productId) return ctx.set('price', 0)
  const product = await fetch(`/api/products/${productId}`).then(r => r.json())
  ctx.set('price', product.price)
})

// Ao mudar price ou quantity → recalcula total
watch('price', (price, _, ctx) => {
  ctx.set('total', price * ctx.get('quantity'))
})

watch('quantity', (qty, _, ctx) => {
  ctx.set('total', ctx.get('price') * qty)
})

onUnmounted(stop)
```

---

### Uso em repeater (linhas com index)

```ts
// Cada linha do repeater cria seu próprio watcher com contexto isolado
// components/OrderItemRow.vue

import { useFieldWatcher } from '@/composables/useFieldWatcher'
import { onUnmounted, toRef } from 'vue'

const props = defineProps<{
  form: Record<string, any>  // o form completo do pai
  index: number              // índice da linha
}>()

// scope='items' + index=2 → resolve 'product_id' como 'items[2].product_id'
const { watch, stop } = useFieldWatcher({
  form: props.form,
  scope: 'items',
  index: toRef(props, 'index'),  // reativo — funciona mesmo se o index mudar
})

// Mesma lógica — mas isolada para esta linha
watch('product_id', async (productId, _, ctx) => {
  if (!productId) return ctx.set('price', 0)
  const product = await fetch(`/api/products/${productId}`).then(r => r.json())
  ctx.set('price', product.price)
  // ctx.key === 'items[2].product_id'
  // ctx.index === 2
})

watch('price', (price, _, ctx) => {
  ctx.set('total', price * ctx.get('quantity'))
})

watch('quantity', (qty, _, ctx) => {
  ctx.set('total', ctx.get('price') * qty)
})

onUnmounted(stop)
```

**Template do repeater no pai:**
```vue
<template>
  <div v-for="(item, index) in form.items" :key="item._key">
    <OrderItemRow :form="form" :index="index" />
  </div>
  <button type="button" @click="addItem">+ Adicionar item</button>
</template>

<script setup lang="ts">
function addItem() {
  form.items.push({
    _key: crypto.randomUUID(),  // key estável para o v-for
    product_id: null,
    price: 0,
    quantity: 1,
    total: 0,
  })
}
</script>
```

---

### Chaves compostas suportadas

O composable aceita qualquer notação de caminho:

```ts
// Dot notation
'items.product_id'         // → form.items.product_id
'items.2.product_id'       // → form.items[2].product_id

// Bracket notation
'items[2].product_id'      // → form.items[2].product_id
'items[2].details.price'   // → form.items[2].details.price (nested profundo)
```

O `ctx.set` e `ctx.get` no callback também aceitam caminhos relativos ao scope:
```ts
ctx.set('price', 99)       // seta items[2].price
ctx.get('quantity')        // lê  items[2].quantity
ctx.set('meta.note', 'x') // seta items[2].meta.note (nested)
```

---

### Integração nos componentes — emit `change` e `blur`

Todo componente gerado deve emitir `change` e `blur` para permitir conexão com o sistema:

```ts
// Em qualquer componente (InputText, SelectInput, DatePicker, etc.)
const emit = defineEmits<{
  'update:modelValue': [value: any]
  'change': [value: any]   // dispara após confirmação do valor (blur ou select)
  'blur': [event: FocusEvent]
}>()

// Nunca depender apenas do v-model para acionar watchers —
// o useFieldWatcher observa o form diretamente via Vue watch()
// mas os emits permitem conexão pontual via @change também:
```

**Conexão pontual via @change (alternativa ao useFieldWatcher):**
```vue
<SelectInput
  v-model="form.product_id"
  :options="products"
  @change="loadProductPrice"
/>

<InputNumber
  v-model="form.quantity"
  @change="recalculateTotal"
/>
```

---

### Quando gerar o `useFieldWatcher`

Gere o composable automaticamente quando o usuário descrever qualquer um destes padrões:

- "ao selecionar X, preenche Y"
- "quando muda X, recalcula Y"
- "campo Y depende de X"
- "ao escolher categoria, filtra subcategorias"
- "preço × quantidade = total"
- "CEP → preenche endereço"
- "produto → carrega estoque/preço/descrição"
- qualquer formulário com **repeater/linhas** que tenha campos dependentes

**Antes de gerar**, sempre verificar se já existe:
```bash
find {pasta_composables} -name "useFieldWatcher.ts" 2>/dev/null
```
- **Se já existe** → leia o arquivo e apenas gere o exemplo de uso específico para o caso,
  sem reescrever o composable
- **Se não existe** → gere o composable completo + o exemplo de uso

Sempre gere junto: o composable (se não existir) + o exemplo de uso específico para o caso descrito.

---


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