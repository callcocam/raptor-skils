# Padrões de Listagem — Referência

## Variações de Header

### Simples (só título + criar)
```vue
<PageHeader title="Produtos">
  <template #actions>
    <button class="px-4 py-2 bg-primary text-white rounded-md text-sm">Novo Produto</button>
  </template>
</PageHeader>
```

### Completo (criar + importar + exportar)
```vue
<PageHeader title="Produtos" :total="meta.total">
  <template #actions>
    <!-- Adaptar ao componente de dropdown do projeto -->
    <button class="px-3 py-2 border rounded-md text-sm" @click="showImportMenu = !showImportMenu">
      ↑ Importar
    </button>
    <button class="px-3 py-2 border rounded-md text-sm" @click="exportData">
      ↓ Exportar
    </button>
    <button class="px-4 py-2 bg-primary text-white rounded-md text-sm" @click="router.visit(route('products.create'))">
      + Novo
    </button>
  </template>
</PageHeader>
```

---

## Variações de Filtros

### Busca simples inline
```vue
<PageFilters v-model="filters" :defaults="filterDefaults">
  <template #default="{ filters, set }">
    <Input
      :model-value="filters.search"
      placeholder="Buscar..."
      @update:model-value="set('search', $event)"
    />
  </template>
</PageFilters>
```

### Filtros com chips ativos
O `<PageFilters>` automaticamente exibe chips para cada filtro ativo
com botão X individual e botão "Limpar todos".

### Filtros avançados em Drawer
```vue
<PageFilters v-model="filters" :defaults="filterDefaults" advanced>
  <template #default="{ filters, set }">
    <Input ... /> <!-- filtros rápidos sempre visíveis -->
  </template>
  <template #advanced="{ filters, set }">
    <!-- aparece dentro do Drawer lateral -->
    <Select ... />
    <DateRangePicker ... />
  </template>
</PageFilters>
```

---

## Variações de Empty State

### Sem dados com CTA
```vue
<PageEmptyState
  title="Nenhum produto encontrado"
  description="Comece criando seu primeiro produto."
  icon="Package"
>
  <template #action>
    <Button @click="router.visit(route('products.create'))">Criar Produto</Button>
  </template>
</PageEmptyState>
```

### Sem dados por filtro ativo
```vue
<PageEmptyState
  title="Nenhum resultado"
  description="Tente ajustar os filtros aplicados."
  icon="Search"
>
  <template #action>
    <Button variant="outline" @click="resetFilters">Limpar Filtros</Button>
  </template>
</PageEmptyState>
```

---

## Variações de Bulk Actions

### Barra flutuante (aparece com seleção)
```vue
<PageBulkActionBar :count="selected.length" @clear="clear">
  <button class="px-3 py-1.5 border rounded text-sm" @click="changeStatus(selected, 'active')">Ativar</button>
  <button class="px-3 py-1.5 border rounded text-sm" @click="changeStatus(selected, 'inactive')">Inativar</button>
  <button class="px-3 py-1.5 bg-destructive text-white rounded text-sm" @click="deleteSelected(selected)">Excluir</button>
</PageBulkActionBar>
```

---

## Paginação com meta do Laravel

O Laravel retorna `meta` no formato:
```json
{
  "current_page": 1,
  "from": 1,
  "last_page": 10,
  "per_page": 15,
  "to": 15,
  "total": 143
}
```

```vue
<PagePagination
  :meta="meta"
  :page-sizes="[15, 25, 50, 100]"
  @change="({ page, perPage }) => router.get(route('products.index'), { page, per_page: perPage })"
/>
```

---

## Toggle de Visualização

```vue
<ViewToggle v-model="viewMode" :options="['table', 'grid', 'kanban']" />
<!-- No slot default do ListPage, renderiza condicionalmente -->
<ProductTable v-if="viewMode === 'table'" ... />
<ProductGrid v-else-if="viewMode === 'grid'" ... />
```