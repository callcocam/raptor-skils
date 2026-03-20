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