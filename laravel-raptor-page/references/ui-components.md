# Templates de Componentes ui/

Estes são os templates base para criação de componentes primitivos em `resources/js/components/ui/`.
Cada template deve ser adaptado ao estilo visual extraído do mock antes de ser criado.

---

## Button.vue

```vue
<script setup lang="ts">
interface Props {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost' | 'destructive'
  size?: 'sm' | 'md' | 'lg'
  disabled?: boolean
  loading?: boolean
  type?: 'button' | 'submit' | 'reset'
}

withDefaults(defineProps<Props>(), {
  variant: 'primary',
  size: 'md',
  type: 'button',
})
</script>

<template>
  <button
    :type="type"
    :disabled="disabled || loading"
    :class="[
      'inline-flex items-center justify-center font-medium transition-colors',
      'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2',
      'disabled:pointer-events-none disabled:opacity-50',
      // classes de variante e tamanho extraídas do mock
    ]"
  >
    <slot />
  </button>
</template>
```

---

## Input.vue

```vue
<script setup lang="ts">
interface Props {
  modelValue?: string | number
  type?: string
  placeholder?: string
  disabled?: boolean
  error?: string
  label?: string
  required?: boolean
}

withDefaults(defineProps<Props>(), { type: 'text' })
defineEmits<{ 'update:modelValue': [value: string] }>()
</script>

<template>
  <div class="flex flex-col gap-1">
    <label v-if="label" class="text-sm font-medium">
      {{ label }}
      <span v-if="required" class="text-destructive">*</span>
    </label>
    <input
      :value="modelValue"
      :type="type"
      :placeholder="placeholder"
      :disabled="disabled"
      @input="$emit('update:modelValue', ($event.target as HTMLInputElement).value)"
      :class="['w-full rounded-md border px-3 py-2 text-sm', error ? 'border-destructive' : '']"
    />
    <span v-if="error" class="text-xs text-destructive">{{ error }}</span>
  </div>
</template>
```

---

## Select.vue

```vue
<script setup lang="ts">
interface Option {
  value: string | number
  label: string
}

interface Props {
  modelValue?: string | number
  options: Option[]
  placeholder?: string
  disabled?: boolean
  error?: string
  label?: string
  required?: boolean
}

defineProps<Props>()
defineEmits<{ 'update:modelValue': [value: string | number] }>()
</script>

<template>
  <div class="flex flex-col gap-1">
    <label v-if="label" class="text-sm font-medium">
      {{ label }}
      <span v-if="required" class="text-destructive">*</span>
    </label>
    <select
      :value="modelValue"
      :disabled="disabled"
      @change="$emit('update:modelValue', ($event.target as HTMLSelectElement).value)"
      :class="['w-full rounded-md border px-3 py-2 text-sm', error ? 'border-destructive' : '']"
    >
      <option v-if="placeholder" value="" disabled>{{ placeholder }}</option>
      <option v-for="opt in options" :key="opt.value" :value="opt.value">
        {{ opt.label }}
      </option>
    </select>
    <span v-if="error" class="text-xs text-destructive">{{ error }}</span>
  </div>
</template>
```

---

## Textarea.vue

```vue
<script setup lang="ts">
interface Props {
  modelValue?: string
  placeholder?: string
  rows?: number
  disabled?: boolean
  error?: string
  label?: string
  required?: boolean
}

withDefaults(defineProps<Props>(), { rows: 4 })
defineEmits<{ 'update:modelValue': [value: string] }>()
</script>

<template>
  <div class="flex flex-col gap-1">
    <label v-if="label" class="text-sm font-medium">
      {{ label }}
      <span v-if="required" class="text-destructive">*</span>
    </label>
    <textarea
      :value="modelValue"
      :rows="rows"
      :placeholder="placeholder"
      :disabled="disabled"
      @input="$emit('update:modelValue', ($event.target as HTMLTextAreaElement).value)"
      :class="['w-full rounded-md border px-3 py-2 text-sm resize-y', error ? 'border-destructive' : '']"
    />
    <span v-if="error" class="text-xs text-destructive">{{ error }}</span>
  </div>
</template>
```

---

## Checkbox.vue

```vue
<script setup lang="ts">
interface Props {
  modelValue?: boolean
  label?: string
  disabled?: boolean
  error?: string
}

defineProps<Props>()
defineEmits<{ 'update:modelValue': [value: boolean] }>()
</script>

<template>
  <div class="flex flex-col gap-1">
    <label class="flex items-center gap-2 cursor-pointer">
      <input
        type="checkbox"
        :checked="modelValue"
        :disabled="disabled"
        @change="$emit('update:modelValue', ($event.target as HTMLInputElement).checked)"
        class="rounded border"
      />
      <span v-if="label" class="text-sm">{{ label }}</span>
    </label>
    <span v-if="error" class="text-xs text-destructive">{{ error }}</span>
  </div>
</template>
```

---

## Badge.vue

```vue
<script setup lang="ts">
interface Props {
  variant?: 'success' | 'warning' | 'danger' | 'info' | 'default'
  label?: string
}

withDefaults(defineProps<Props>(), { variant: 'default' })
</script>

<template>
  <span
    :class="[
      'inline-flex items-center rounded-full px-2.5 py-0.5 text-xs font-medium',
      {
        'bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-200': variant === 'success',
        'bg-yellow-100 text-yellow-800 dark:bg-yellow-900 dark:text-yellow-200': variant === 'warning',
        'bg-red-100 text-red-800 dark:bg-red-900 dark:text-red-200': variant === 'danger',
        'bg-blue-100 text-blue-800 dark:bg-blue-900 dark:text-blue-200': variant === 'info',
        'bg-gray-100 text-gray-800 dark:bg-gray-800 dark:text-gray-200': variant === 'default',
      }
    ]"
  >
    <slot>{{ label }}</slot>
  </span>
</template>
```

---

## Card.vue

```vue
<script setup lang="ts">
interface Props {
  title?: string
  description?: string
  noPadding?: boolean
}

defineProps<Props>()
</script>

<template>
  <div class="rounded-lg border bg-card text-card-foreground shadow-sm">
    <div v-if="title || description" class="flex flex-col space-y-1.5 p-6 pb-0">
      <h3 v-if="title" class="font-semibold leading-none tracking-tight">{{ title }}</h3>
      <p v-if="description" class="text-sm text-muted-foreground">{{ description }}</p>
    </div>
    <div :class="noPadding ? '' : 'p-6'">
      <slot />
    </div>
  </div>
</template>
```

---

## Modal.vue

```vue
<script setup lang="ts">
interface Props {
  open: boolean
  title?: string
  size?: 'sm' | 'md' | 'lg' | 'xl'
}

withDefaults(defineProps<Props>(), { size: 'md' })
defineEmits<{ close: [] }>()
</script>

<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="open" class="fixed inset-0 z-50 flex items-center justify-center">
        <div class="absolute inset-0 bg-black/50" @click="$emit('close')" />
        <div
          :class="[
            'relative z-10 w-full rounded-lg bg-background p-6 shadow-lg',
            { 'max-w-sm': size === 'sm', 'max-w-md': size === 'md',
              'max-w-lg': size === 'lg', 'max-w-2xl': size === 'xl' }
          ]"
        >
          <h2 v-if="title" class="mb-4 text-lg font-semibold">{{ title }}</h2>
          <slot />
        </div>
      </div>
    </Transition>
  </Teleport>
</template>
```

---

## Pagination.vue

```vue
<script setup lang="ts">
interface Link {
  url: string | null
  label: string
  active: boolean
}

interface Props {
  links: Link[]
  meta?: { from: number; to: number; total: number }
}

defineProps<Props>()
</script>

<template>
  <div class="flex items-center justify-between">
    <p v-if="meta" class="text-sm text-muted-foreground">
      Exibindo {{ meta.from }}–{{ meta.to }} de {{ meta.total }} registros
    </p>
    <div class="flex gap-1">
      <template v-for="link in links" :key="link.label">
        <Link
          v-if="link.url"
          :href="link.url"
          preserve-state
          :class="['px-3 py-1 rounded text-sm border', link.active ? 'bg-primary text-primary-foreground' : '']"
          v-html="link.label"
        />
        <span v-else class="px-3 py-1 rounded text-sm border opacity-40" v-html="link.label" />
      </template>
    </div>
  </div>
</template>
```

---

## index.ts (exportar tudo)

```typescript
// resources/js/components/ui/index.ts
export { default as Button } from './Button.vue'
export { default as Input } from './Input.vue'
export { default as Select } from './Select.vue'
export { default as Textarea } from './Textarea.vue'
export { default as Checkbox } from './Checkbox.vue'
export { default as Badge } from './Badge.vue'
export { default as Card } from './Card.vue'
export { default as Modal } from './Modal.vue'
export { default as Pagination } from './Pagination.vue'
```

---

## Notas de adaptação ao mock

Ao criar qualquer componente acima, **sempre substituir** os valores genéricos pelos extraídos do mock:

- Classes de cor hardcoded → variáveis CSS do tema do projeto
- Border radius → valor do mock
- Tamanhos de fonte e espaçamento → valores do mock
- Variantes de botão → nomes e estilos do mock (pode ter mais ou menos variantes)
- Sombras, transições → conforme o mock