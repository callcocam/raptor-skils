# Frontend Patterns Reference

## Lendo a Pasta de Mocks

Quando a pasta `resources/js/components/_mocks/` (ou similar) existir, **sempre lê-la antes de gerar qualquer componente Vue**. Extrair:

1. **Naming convention** — PascalCase, prefixos, sufixos usados (`AppButton`, `BaseCard`, `UiModal`?)
2. **Estrutura de props** — como são tipadas (TypeScript interface? `defineProps<{}>()`? `withDefaults`?)
3. **Composição de layout** — qual componente de layout é usado como wrapper
4. **Componentes de UI base** — quais componentes são reutilizados (`Button`, `Input`, `Card`, etc.)
5. **Padrão de imports** — paths com `@/`, aliases customizados?
6. **Estilo** — classes Tailwind direto? componentes abstraídos? variantes com `cva`?

---

## Padrão shadcn-vue (padrão quando não há mocks)

### Estrutura de um componente de página (Inertia)

```vue
<script setup lang="ts">
import { Head } from '@inertiajs/vue3'
import AppLayout from '@/layouts/AppLayout.vue'
import { Button } from '@/components/ui/button'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'

interface Props {
  items: ItemType[]
}

const props = defineProps<Props>()
</script>

<template>
  <Head title="Título da Página" />

  <AppLayout>
    <div class="space-y-6">
      <Card>
        <CardHeader>
          <CardTitle>Título</CardTitle>
        </CardHeader>
        <CardContent>
          <!-- conteúdo -->
        </CardContent>
      </Card>
    </div>
  </AppLayout>
</template>
```

### Formulários (Create / Edit)

```vue
<script setup lang="ts">
import { useForm } from '@inertiajs/vue3'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'

const form = useForm({
  nome: '',
  // outros campos
})

const submit = () => {
  form.post(route('recurso.store'))
}
</script>

<template>
  <form @submit.prevent="submit" class="space-y-4">
    <div class="space-y-2">
      <Label for="nome">Nome</Label>
      <Input id="nome" v-model="form.nome" />
      <p v-if="form.errors.nome" class="text-sm text-destructive">
        {{ form.errors.nome }}
      </p>
    </div>

    <Button type="submit" :disabled="form.processing">
      Salvar
    </Button>
  </form>
</template>
```

### Tabela de listagem (Index)

```vue
<script setup lang="ts">
import {
  Table, TableBody, TableCell,
  TableHead, TableHeader, TableRow
} from '@/components/ui/table'
import { Button } from '@/components/ui/button'
import { Link } from '@inertiajs/vue3'

interface Props {
  items: ItemType[]
}

defineProps<Props>()
</script>

<template>
  <div class="flex justify-end mb-4">
    <Button as-child>
      <Link :href="route('recurso.create')">Novo</Link>
    </Button>
  </div>

  <Table>
    <TableHeader>
      <TableRow>
        <TableHead>Nome</TableHead>
        <TableHead class="text-right">Ações</TableHead>
      </TableRow>
    </TableHeader>
    <TableBody>
      <TableRow v-for="item in items" :key="item.id">
        <TableCell>{{ item.nome }}</TableCell>
        <TableCell class="text-right space-x-2">
          <Button variant="outline" size="sm" as-child>
            <Link :href="route('recurso.edit', item.id)">Editar</Link>
          </Button>
        </TableCell>
      </TableRow>
    </TableBody>
  </Table>
</template>
```

---

## Tipos TypeScript

Criar arquivo de tipos para cada recurso em `resources/js/types/`:

```ts
// resources/js/types/nome-model.ts
export interface NomeModel {
  id: number
  nome: string
  created_at: string
  updated_at: string
}
```

---

## Usando a pasta /tema como referência de estilo

A pasta `/tema` fica na **raiz do projeto** (não dentro de `resources/`). Cada subpasta segue o padrão `{tela}_{device}_{modo}`.

### Processo obrigatório antes de criar qualquer componente

```bash
# 1. Listar mocks disponíveis
ls tema/

# 2. Ler o HTML do mock relevante (arquivo sem extensão ou .html)
cat tema/nome_da_tela_desktop_light/code

# 3. Ler DESIGN.md se existir
cat tema/nome_da_tela_desktop_light/DESIGN.md 2>/dev/null
```

O `screen.png` é usado como referência visual — carregar na conversa para entender o layout.

### Conversão HTML/CSS → Vue 3 + Tailwind

| HTML/CSS original | Equivalente Vue/Tailwind |
|---|---|
| `display: flex; gap: 16px` | `flex gap-4` |
| `padding: 24px` | `p-6` |
| `var(--color-primary)` | token Tailwind (mapear em `tailwind.config`) |
| `class="card"` | `rounded-lg border bg-card p-6` |
| `class="btn btn-primary"` | `<Button>` do shadcn-vue |
| `<input type="text">` | `<Input>` do shadcn-vue |

### Dark/Light mode
- Sempre implementar ambos quando existir mock `_dark` e `_light`
- Usar variáveis CSS semânticas: `bg-background`, `text-foreground`, `bg-card`, `text-muted-foreground`
- Nunca hardcodar cores (`#fff`, `text-gray-900`) — usar tokens

### Responsividade
- Comparar `_desktop`, `_mobile` e `_tablet` para entender breakpoints
- Prefixos Tailwind: `sm:` (640px), `md:` (768px), `lg:` (1024px)

### Extração de componentes reutilizáveis
- Se o mesmo padrão aparece em múltiplos mocks → extrair para `resources/js/components/shared/`
- shadcn-vue fornece os primitivos; o estilo do mock sobrescreve via `class` ou tema CSS

---

## Regras de Estilo

- Sempre usar **TypeScript** com `<script setup lang="ts">`
- Sempre usar **`defineProps<Interface>()`** — nunca a syntax de array/objeto legada
- Usar `useForm` do Inertia para **todos os formulários** — nunca `ref` manual para campos de form
- Erros de validação sempre vindos de `form.errors.campo`
- Espaçamento de layout: `space-y-6` para seções, `space-y-4` para grupos de campos, `space-y-2` para label+input
- Botões destrutivos (delete): sempre `variant="destructive"` do shadcn-vue
- Nunca estilizar diretamente com `style=""` — sempre classes Tailwind
- **Fidelidade visual ao mock tem prioridade** sobre convenções genéricas de estilo