# Padrões de Substituição i18n — Referência Completa

## Vue Components (`*.vue`)

### Texto simples no template

```vue
<!-- ❌ Antes -->
<h1>Products</h1>
<p>No results found</p>
<span>Loading...</span>

<!-- ✅ Depois -->
<script setup lang="ts">
import { useT } from '@/composables/useT'
const { t } = useT()
</script>

<h1>{{ t('app.resources.products') }}</h1>
<p>{{ t('app.no_results') }}</p>
<span>{{ t('app.loading') }}</span>
```

---

### Botões e ações

```vue
<!-- ❌ Antes -->
<Button>Create Product</Button>
<Button variant="outline">Cancel</Button>
<Button variant="destructive">Delete</Button>
<Button>Save</Button>
<Button>Edit</Button>
<Button>View</Button>

<!-- ✅ Depois -->
<Button>{{ t('app.create', { resource: t('app.resources.product_singular') }) }}</Button>
<Button variant="outline">{{ t('app.cancel') }}</Button>
<Button variant="destructive">{{ t('app.delete') }}</Button>
<Button>{{ t('app.save') }}</Button>
<Button>{{ t('app.edit') }}</Button>
<Button>{{ t('app.view') }}</Button>
```

---

### Inputs — placeholder, label, aria

```vue
<!-- ❌ Antes -->
<input placeholder="Search products..." type="search" />
<label for="name">Product name</label>
<input aria-label="Search" />

<!-- ✅ Depois -->
<input :placeholder="t('app.search')" type="search" />
<label for="name">{{ t('resources/products.fields.name') }}</label>
<input :aria-label="t('app.search')" />
```

---

### Estado vazio (empty state)

```vue
<!-- ❌ Antes -->
<div v-if="!items.length">
  <h3>No products found</h3>
  <p>Create your first product to get started.</p>
  <Button>Create Product</Button>
</div>

<!-- ✅ Depois -->
<div v-if="!items.length">
  <h3>{{ t('app.no_results') }}</h3>
  <p>{{ t('app.empty_description', { resource: t('app.resources.product_singular') }) }}</p>
  <Button>{{ t('app.create', { resource: t('app.resources.product_singular') }) }}</Button>
</div>
```

---

### Modal de confirmação destrutiva

```vue
<!-- ❌ Antes -->
<ConfirmModal
  title="Delete Product"
  description="Are you sure you want to delete this product? This action cannot be undone."
  confirm-text="Delete"
  cancel-text="Cancel"
/>

<!-- ✅ Depois -->
<ConfirmModal
  :title="t('app.confirm_delete_title', { resource: t('app.resources.product_singular') })"
  :description="t('app.confirm_delete_description')"
  :confirm-text="t('app.delete')"
  :cancel-text="t('app.cancel')"
/>
```

---

### Toast / Notificações

```vue
<!-- ❌ Antes (em script setup) -->
toast.success('Product created successfully!')
toast.error('Failed to delete product.')

<!-- ✅ Depois -->
toast.success(t('app.created_success', { resource: t('app.resources.product_singular') }))
toast.error(t('app.error_message'))
```

---

### Títulos de página (Vue Head / useHead)

```vue
<!-- ❌ Antes -->
useHead({ title: 'Products' })
useHead({ title: 'Create Product' })
useHead({ title: `Edit ${product.name}` })

<!-- ✅ Depois -->
useHead({ title: t('app.resources.products') })
useHead({ title: t('app.create', { resource: t('app.resources.product_singular') }) })
useHead({ title: t('app.edit_item', { item: product.name }) })
```

---

### Paginação e metadados

```vue
<!-- ❌ Antes -->
<p>Showing {{ from }} to {{ to }} of {{ total }} results</p>
<p>10 per page</p>

<!-- ✅ Depois -->
<p>{{ t('app.pagination.showing', { from, to, total }) }}</p>
<p>{{ t('app.pagination.per_page', { count: 10 }) }}</p>
```

---

### Tabs e navegação interna

```vue
<!-- ❌ Antes -->
<TabsList>
  <TabsTrigger value="details">Details</TabsTrigger>
  <TabsTrigger value="history">History</TabsTrigger>
  <TabsTrigger value="settings">Settings</TabsTrigger>
</TabsList>

<!-- ✅ Depois -->
<TabsList>
  <TabsTrigger value="details">{{ t('app.tabs.details') }}</TabsTrigger>
  <TabsTrigger value="history">{{ t('app.tabs.history') }}</TabsTrigger>
  <TabsTrigger value="settings">{{ t('app.tabs.settings') }}</TabsTrigger>
</TabsList>
```

---

### Status badges

```vue
<!-- ❌ Antes -->
<Badge>Draft</Badge>
<Badge variant="success">Published</Badge>
<Badge variant="destructive">Inactive</Badge>

<!-- ✅ Depois -->
<Badge>{{ t('app.status.draft') }}</Badge>
<Badge variant="success">{{ t('app.status.published') }}</Badge>
<Badge variant="destructive">{{ t('app.status.inactive') }}</Badge>
```

---

## PHP — Controllers

### Inertia render com título

```php
// ❌ Antes
return Inertia::render('Products/Index', [
    'title' => 'Products',
    'subtitle' => 'Manage your products',
]);

// ✅ Depois
return Inertia::render('Products/Index', [
    'title' => __('app.resources.products'),
    'subtitle' => __('app.resources.products_description'),
]);
```

---

### Flash messages

```php
// ❌ Antes — vários formatos encontrados no wild
session()->flash('success', 'Product created successfully.');
session()->flash('error', 'Something went wrong.');
back()->with('message', 'Product updated.');
return redirect()->route('products.index')->with('success', 'Deleted!');

// ✅ Depois
session()->flash('success', __('app.created_success', ['resource' => __('resources/products.singular')]));
session()->flash('error', __('app.error_message'));
back()->with('message', __('app.updated_success', ['resource' => __('resources/products.singular')]));
return redirect()->route('products.index')
    ->with('success', __('app.deleted_success', ['resource' => __('resources/products.singular')]));
```

---

### Abort com mensagem

```php
// ❌ Antes
abort(403, 'You are not authorized to do this.');
abort(404, 'Product not found.');

// ✅ Depois
abort(403, __('app.unauthorized'));
abort(404, __('app.not_found'));
```

---

### Validação de Form Request — attribute names

```php
// ❌ Antes
public function attributes(): array
{
    return [
        'name'  => 'name',
        'email' => 'email',
        'price' => 'price',
    ];
}

// ✅ Depois
public function attributes(): array
{
    return [
        'name'  => __('resources/products.fields.name'),
        'email' => __('validation.attributes.email'),
        'price' => __('resources/products.fields.price'),
    ];
}
```

---

## PHP — Enums

### label() e color()

```php
// ❌ Antes
public function label(): string
{
    return match($this) {
        self::Draft     => 'Draft',
        self::Published => 'Published',
        self::Archived  => 'Archived',
        self::Inactive  => 'Inactive',
    };
}

// ✅ Depois
public function label(): string
{
    return match($this) {
        self::Draft     => __('app.status.draft'),
        self::Published => __('app.status.published'),
        self::Archived  => __('app.status.archived'),
        self::Inactive  => __('app.status.inactive'),
    };
}
```

---

### Enum com descrições

```php
// ❌ Antes
public function description(): string
{
    return match($this) {
        self::Draft     => 'Not visible to users',
        self::Published => 'Visible to all users',
    };
}

// ✅ Depois
public function description(): string
{
    return match($this) {
        self::Draft     => __('app.status.draft_description'),
        self::Published => __('app.status.published_description'),
    };
}
```

---

## PHP — Services (mensagens de log/exception)

```php
// ❌ Antes — mensagens de exception que chegam ao usuário
throw new \Exception('Product not found in external system.');
throw new \RuntimeException('Failed to sync product: ' . $e->getMessage());

// ✅ Depois — apenas se a mensagem chega ao usuário final
throw new \Exception(__('app.not_found'));
// Mensagens técnicas de log NÃO precisam de tradução:
Log::error('Product sync failed: ' . $e->getMessage()); // ← manter em inglês
```

---

## Blade Templates

### Títulos e metadados

```blade
{{-- ❌ Antes --}}
<title>Products — My App</title>
<meta name="description" content="Manage your products">

{{-- ✅ Depois --}}
<title>{{ __('app.resources.products') }} — {{ config('app.name') }}</title>
<meta name="description" content="{{ __('app.resources.products_description') }}">
```

---

### Textos inline

```blade
{{-- ❌ Antes --}}
<h1>Dashboard</h1>
<p>Welcome back, {{ $user->name }}!</p>
<a href="{{ route('products.create') }}">Create Product</a>

{{-- ✅ Depois --}}
<h1>{{ __('app.dashboard') }}</h1>
<p>{{ __('app.welcome', ['name' => $user->name]) }}</p>
<a href="{{ route('products.create') }}">{{ __('app.create', ['resource' => __('resources/products.singular')]) }}</a>
```

---

### Botões em forms Blade

```blade
{{-- ❌ Antes --}}
<button type="submit">Save</button>
<a href="{{ url()->previous() }}">Cancel</a>
<button type="button" onclick="confirm('Are you sure?')">Delete</button>

{{-- ✅ Depois --}}
<button type="submit">{{ __('app.save') }}</button>
<a href="{{ url()->previous() }}">{{ __('app.cancel') }}</a>
<button type="button" onclick="confirm('{{ __('app.confirm_delete_description') }}')">{{ __('app.delete') }}</button>
```

---

## Estrutura de arquivos lang/ recomendada

```
lang/
└── pt_BR/
    ├── app.php              ← strings comuns de UI
    ├── auth.php             ← autenticação
    ├── validation.php       ← mensagens de validação
    ├── passwords.php        ← reset de senha
    └── resources/
        ├── products.php     ← strings específicas de produto
        ├── categories.php
        └── {recurso}.php    ← um arquivo por recurso de negócio
```

### Estrutura de resources/{recurso}.php

```php
<?php
// lang/pt_BR/resources/products.php

return [
    'singular'    => 'Produto',
    'plural'      => 'Produtos',
    'description' => 'Gerencie seus produtos',

    'fields' => [
        'name'        => 'Nome',
        'description' => 'Descrição',
        'price'       => 'Preço',
        'status'      => 'Status',
        'category'    => 'Categoria',
        'slug'        => 'Slug',
    ],
];
```
