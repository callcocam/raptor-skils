# Padrões de Formulário — Referência

## Variações de Header do Form

### Criação
```vue
<PageHeader title="Novo Produto" :breadcrumbs="[{ label: 'Produtos', href: route('products.index') }, { label: 'Novo' }]">
  <template #actions>
    <Button variant="outline" @click="cancel">Cancelar</Button>
    <Button @click="submit" :loading="form.processing">Salvar</Button>
  </template>
</PageHeader>
```

### Edição com ações extras
```vue
<PageHeader
  :title="`Editando ${product.name}`"
  :breadcrumbs="breadcrumbs"
  :status="product.status"
>
  <template #actions>
    <Button variant="ghost" @click="duplicate"><Copy class="w-4 h-4" />Duplicar</Button>
    <Button variant="destructive" @click="confirmDelete"><Trash class="w-4 h-4" />Excluir</Button>
    <Button variant="outline" @click="cancel">Cancelar</Button>
    <Button @click="submit" :loading="form.processing">Salvar Alterações</Button>
  </template>
</PageHeader>
```

---

## Layouts de Formulário

### Coluna única (forms simples)
```vue
<FormPage :title="..." layout="single">
  <template #default>
    <!-- fields em coluna única -->
  </template>
</FormPage>
```

### Main + Sidebar (padrão)
```vue
<FormPage :title="..." layout="sidebar">
  <template #default>
    <!-- 8 colunas: campos principais -->
  </template>
  <template #sidebar>
    <!-- 4 colunas: status, datas, autor, tags -->
  </template>
</FormPage>
```

### Com seções / tabs
```vue
<FormPage :title="..." layout="tabs" :tabs="['Geral', 'Estoque', 'SEO', 'Imagens']">
  <template #tab-geral>...</template>
  <template #tab-estoque>...</template>
  <template #tab-seo>...</template>
  <template #tab-imagens>...</template>
</FormPage>
```

---

## Footer Sticky de Ações

Sempre presente no FormPage, com os botões Cancelar e Salvar.
Fica fixo no rodapé enquanto o usuário rola o formulário.

```vue
<!-- Gerado automaticamente pelo FormPage -->
<!-- Pode ser substituído via slot: -->
<template #footer>
  <div class="flex items-center justify-between">
    <span v-if="form.isDirty" class="text-sm text-muted-foreground">
      Você tem alterações não salvas
    </span>
    <div class="flex gap-2 ml-auto">
      <Button variant="outline" @click="cancel">Cancelar</Button>
      <Button @click="submit" :disabled="!form.isDirty || form.processing">
        <Loader2 v-if="form.processing" class="w-4 h-4 mr-2 animate-spin" />
        Salvar
      </Button>
    </div>
  </div>
</template>
```

---

## Dirty Check — Aviso de Saída

O `useFormDirty` integra com Inertia automaticamente:

```ts
// Em qualquer FormPage ou página de edição
const form = useForm({ name: product.name, status: product.status })
const { isDirty, confirmLeave } = useFormDirty(form)

// O composable registra automaticamente:
// - window.beforeunload (aba fechada)
// - router.on('before') do Inertia (navegação interna)
// Ambos exibem o Dialog de confirmação se form.isDirty
```

---

## Exibição de Erros de Validação (Inertia)

O `<FormField>` encapsula o padrão de exibição:

```vue
<FormField label="Nome do Produto" :error="form.errors.name" required>
  <Input v-model="form.name" :class="{ 'border-destructive': form.errors.name }" />
</FormField>

<!-- Renderiza: label, input, e abaixo o erro em vermelho se existir -->
```

---

## Sidebar de Metadados — Padrões Comuns

```vue
<template #sidebar>
  <!-- Status -->
  <div class="rounded-lg border bg-card p-4 space-y-3">
    <p class="text-sm font-medium">Status</p>
    <select v-model="form.status" class="w-full border rounded-md px-3 py-2 text-sm">
      <option value="draft">Rascunho</option>
      <option value="active">Ativo</option>
      <option value="inactive">Inativo</option>
    </select>
  </div>

  <!-- Datas (somente leitura) -->
  <div v-if="product.id" class="rounded-lg border bg-card p-4 text-sm text-muted-foreground space-y-1">
    <p>Criado em: {{ formatDate(product.created_at) }}</p>
    <p>Atualizado em: {{ formatDate(product.updated_at) }}</p>
    <p v-if="product.created_by">Por: {{ product.created_by.name }}</p>
  </div>
</template>
```

---

## Navegação entre Registros

```vue
<FormPage
  :title="..."
  :prev-url="product.prev_url"
  :next-url="product.next_url"
>
```

O backend precisa injetar `prev_url` e `next_url` no resource.
O FormPage renderiza setas de navegação no header quando presentes.

---

## Design Tokens — Formulário (extraídos do tema/)

### Separador de seção (ícone + título)
```html
<div class="flex items-center gap-2 pb-2 border-b border-outline-variant">
  <span class="material-symbols-outlined text-primary text-xl">person</span>
  <h3 class="font-mono font-bold text-sm tracking-widest uppercase text-on-surface-variant">
    Dados Pessoais
  </h3>
</div>
```

### Input estilo underline (padrão do tema)
```html
<div class="space-y-2">
  <label class="text-xs font-mono font-medium text-on-surface-variant uppercase tracking-wider">
    Nome Completo
  </label>
  <input class="w-full bg-surface-container-low border-b-2 border-outline-variant
                focus:border-primary focus:ring-0 px-0 py-2 transition-all outline-none
                placeholder:text-outline/50" />
  <p class="text-xs text-error">{{ error }}</p>
</div>
```

### Input estilo caixa (padrão dos componentes existentes)
```html
<input class="w-full rounded-lg border border-border bg-card px-3 py-2.5 text-sm text-foreground
              focus:outline-none focus:ring-1 focus:ring-primary" />
```

### Checkbox item selecionável (roles/permissões)
```html
<!-- Desmarcado -->
<label class="flex items-center justify-between p-4 rounded-xl border-2 border-outline-variant
              hover:border-primary/50 transition-all cursor-pointer bg-surface-container-lowest">
  <div class="flex flex-col">
    <span class="font-bold text-sm">Administrator</span>
    <span class="text-[10px] font-mono text-on-surface-variant">FULL_SYSTEM_ACCESS</span>
  </div>
  <input type="checkbox" class="rounded text-primary focus:ring-primary h-5 w-5" />
</label>

<!-- Marcado -->
<label class="flex items-center justify-between p-4 rounded-xl border-2 border-primary
              bg-primary/5 transition-all cursor-pointer">
  <div class="flex flex-col">
    <span class="font-bold text-sm text-primary">Inventory Manager</span>
    <span class="text-[10px] font-mono text-primary/70">STOCK_CONTROL_WRITE</span>
  </div>
  <input checked type="checkbox" class="rounded text-primary focus:ring-primary h-5 w-5 border-primary" />
</label>
```

### Cards de metadados (abaixo ou na sidebar do form)
```html
<div class="grid grid-cols-3 gap-4">
  <div class="p-4 bg-surface rounded-xl border border-outline-variant flex flex-col gap-1">
    <span class="text-[10px] font-mono text-on-surface-variant uppercase tracking-widest">Status</span>
    <div class="flex items-center gap-2">
      <span class="w-1.5 h-1.5 bg-primary rounded-full"></span>
      <span class="font-mono text-sm font-bold text-on-surface">PROVISIONING</span>
    </div>
  </div>
  <div class="p-4 bg-surface rounded-xl border border-outline-variant flex flex-col gap-1">
    <span class="text-[10px] font-mono text-on-surface-variant uppercase tracking-widest">Criado em</span>
    <span class="font-mono text-sm font-bold text-on-surface">{{ created_at }}</span>
  </div>
</div>
```

### Modal de confirmação de exclusão (visual do tema)
```html
<div class="bg-surface-container-lowest border border-outline-variant shadow-2xl rounded-xl max-w-md w-full">
  <div class="p-6">
    <div class="flex items-center gap-4 mb-4">
      <div class="w-12 h-12 bg-error-container text-error rounded-full flex items-center justify-center">
        <span class="material-symbols-outlined">delete_forever</span>
      </div>
      <h3 class="text-xl font-bold font-headline">Excluir registro?</h3>
    </div>
    <p class="text-on-surface-variant text-sm leading-relaxed mb-6">
      Esta ação é permanente e não pode ser desfeita.
    </p>
    <div class="flex gap-3 justify-end">
      <button class="px-4 py-2 text-sm font-medium text-on-surface-variant hover:bg-surface-container-high rounded-lg">
        Cancelar
      </button>
      <button class="px-4 py-2 text-sm font-bold bg-error text-white rounded-lg hover:bg-error-dim shadow-sm">
        Excluir
      </button>
    </div>
  </div>
</div>
```

---

## Confirmação de Exclusão — Padrão

```ts
// Usando usePageActions
const { confirm } = usePageActions()

async function confirmDelete() {
  const ok = await confirm({
    title: 'Excluir produto?',
    description: `"${product.name}" será excluído permanentemente. Esta ação não pode ser desfeita.`,
    confirmLabel: 'Sim, excluir',
    variant: 'destructive',
  })
  if (ok) {
    router.delete(route('products.destroy', product.id), {
      onSuccess: () => router.visit(route('products.index')),
    })
  }
}
```