---
name: laravel-raptor-page
description: >
  Analisa um mock de design (pasta com screen.png + HTML/CSS + opcional DESIGN.md) e gera um
  CRUD completo para um projeto Laravel com Inertia + Vue 3 + TypeScript. Use esta skill SEMPRE
  que o usuário pedir para: "criar uma página", "criar um CRUD", "gerar tela de X", "implementar
  módulo de X", "seguir o mock para X", "criar formulário de X", "criar listagem de X", passar
  uma pasta de mock para análise, ou mencionar qualquer combinação de mock + geração de código.
  A skill analisa o mock visualmente e em código, verifica o que já existe no projeto, avisa
  sobre dependências faltando, e gera tudo na ordem correta: backend → componentes ui/ →
  componentes base → páginas Vue.
---

# Laravel Raptor Page Skill

## Visão Geral

Esta skill transforma um mock de design (pasta com `screen.png` + arquivo `code` HTML/CSS) em um CRUD completo, cobrindo backend e frontend, respeitando ao máximo o estilo visual do mock e reaproveitando componentes já existentes no projeto.

---

## Passo 1 — Localizar o mock

### Se o usuário passou um caminho explícito:
Usar o caminho informado diretamente.

### Se não passou caminho:
Listar as pastas disponíveis em `/tema` e perguntar qual usar:

```bash
ls tema/
```

Exibir a lista e pedir confirmação:
> "Qual pasta de mock você quer usar? Encontrei estas opções em `/tema`:"

### Nomear o recurso
Extrair o nome do recurso a partir do nome da pasta do mock.
Exemplo: `gerenciamento_de_lojas_desktop_light` → recurso `Loja` / `lojas`.

Se ambíguo, perguntar ao usuário:
> "Qual será o nome do Model Laravel para este recurso? (ex: `Loja`, `Produto`, `Pedido`)"

### Verificar contexto multi-tenant

Se ainda não foi definido no projeto, perguntar **uma única vez** e guardar para todos os models:

> "Este projeto é multi-tenant? O model precisa do campo `tenant_id`?"

- **Sim** → incluir `tenant_id` na migration e no model de todos os recursos criados
- **Não** → ignorar `tenant_id` em todos os resources

> Se já foi respondido antes nesta conversa, não perguntar novamente — usar a resposta anterior.

---

## Passo 2 — Analisar o mock completamente

Ler **todos** os arquivos da pasta antes de gerar qualquer código:

```bash
# 1. Ler o HTML/CSS do mock
cat {pasta}/code

# 2. Ler anotações de design (se existir)
cat {pasta}/DESIGN.md 2>/dev/null

# 3. Carregar o screenshot para referência visual
# screen.png — analisar layout, hierarquia, espaçamentos, cores
```

Se existirem variantes do mesmo mock (ex: `_dark`, `_mobile`, `_tablet`), ler também:
```bash
ls tema/ | grep "^{nome_base}"
```

### O que extrair do mock:

**Estrutura de dados (campos):**
- Identificar todos os campos visíveis no formulário e na listagem
- Inferir tipo de cada campo: texto, número, data, select, checkbox, textarea, upload
- Identificar campos obrigatórios (marcação visual, asterisco, etc.)
- Identificar relacionamentos (selects com dados de outras entidades)

**Estilo visual:**
- Paleta de cores (variáveis CSS, classes Tailwind, ou valores hex)
- Tipografia (tamanhos, pesos, famílias)
- Espaçamentos recorrentes (padding, gap, margin)
- Padrão de cards, tabelas, formulários, botões, badges
- Comportamento dark/light se existirem variantes
- Breakpoints mobile/tablet se existirem variantes

**Componentes identificados no mock:**
- Listar cada componente visual distinto (ex: campo de busca, dropdown de filtro, botão de ação, badge de status)
- Anotar quais parecem reutilizáveis vs. específicos desta tela

---

## Passo 3 — Auditar o projeto (o que já existe)

Antes de criar qualquer arquivo, verificar o que já existe:

```bash
# Componentes ui/ disponíveis
ls resources/js/components/ui/ 2>/dev/null

# Componentes shared/ disponíveis
ls resources/js/components/shared/ 2>/dev/null

# Layouts disponíveis
ls resources/js/layouts/ 2>/dev/null

# Model já existe?
ls app/Models/{NomeModel}.php 2>/dev/null

# Controller já existe?
ls app/Http/Controllers/{NomeModel}Controller.php 2>/dev/null

# Service já existe?
ls app/Services/{NomeModel}Service.php 2>/dev/null

# Páginas Vue já existem?
ls resources/js/pages/{NomeModel}/ 2>/dev/null

# Componentes específicos do recurso já existem?
ls resources/js/components/{NomeModel}/ 2>/dev/null
```

### Relatório de auditoria

Antes de gerar qualquer código, apresentar ao usuário um relatório resumido:

```
📋 Análise do mock: {nome_da_pasta}
   Recurso identificado: {NomeModel}

✅ Já existe no projeto:
   - app/Models/Loja.php
   - resources/js/components/ui/Button.vue
   - resources/js/components/ui/Input.vue

⚠️  Componentes ui/ necessários mas AUSENTES:
   - Select.vue        ← usado no campo "categoria" do formulário
   - DatePicker.vue    ← usado no campo "data_abertura"
   - Badge.vue         ← usado no status da listagem

🔨 Será criado:
   Backend:
   - app/Models/Loja.php (migration incluída)
   - app/Http/Controllers/LojaController.php
   - app/Services/LojaService.php
   - app/Http/Requests/Loja/StoreLojaRequest.php
   - app/Http/Requests/Loja/UpdateLojaRequest.php

   Frontend:
   - resources/js/components/Loja/LojaForm.vue       ← FormBase
   - resources/js/components/Loja/LojaList.vue       ← ListBase
   - resources/js/components/Loja/LojaFilterBar.vue  ← FilterBar
   - resources/js/pages/Loja/Index.vue
   - resources/js/pages/Loja/Form.vue
   - resources/js/pages/Loja/Show.vue (se mock tiver tela de detalhe)

Deseja prosseguir? Quer criar os componentes ui/ ausentes antes de continuar?
```

> ⚠️ **Aguardar confirmação do usuário antes de gerar qualquer arquivo.**

---

## Passo 4 — Criar componentes ui/ ausentes (se confirmado)

Para cada componente `ui/` ausente identificado na auditoria, criar baseado no estilo do mock.

Ver `references/ui-components.md` para os templates de cada componente primitivo.

**Regra:** cada componente `ui/` deve ser genérico e reutilizável — sem lógica de negócio, sem referência ao recurso atual.

```
resources/js/components/ui/
├── Select.vue       ← props: options, modelValue, placeholder, disabled, error
├── DatePicker.vue   ← props: modelValue, min, max, disabled, error
├── Badge.vue        ← props: variant ('success'|'warning'|'danger'|'info'), label
└── ...
```

---

## Passo 5 — Gerar o Backend

Sempre usando Sail (se ativo):

```bash
# Model + Migration
./vendor/bin/sail artisan make:model {NomeModel} -m

# Controller Resource
./vendor/bin/sail artisan make:controller {NomeModel}Controller --resource --model={NomeModel}

# Form Requests
./vendor/bin/sail artisan make:request {NomeModel}/Store{NomeModel}Request
./vendor/bin/sail artisan make:request {NomeModel}/Update{NomeModel}Request
```

Service e Repository criados manualmente conforme padrão do `laravel-raptor-patterns`.

---

### Model — padrão obrigatório

Todo model gerado segue esta estrutura base:

```php
<?php

namespace App\Models;

use Callcocam\Tall\Sluggable\HasSlug;           // slug automático
use Callcocam\Tall\Sluggable\SlugOptions;
use Illuminate\Database\Eloquent\Concerns\HasUlids;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class NomeModel extends Model
{
    use HasFactory, HasUlids, SoftDeletes, HasSlug;

    protected $fillable = [
        // tenant_id (se multi-tenant)
        'name',
        'slug',
        'status',
        // demais campos inferidos do mock
    ];

    protected $casts = [
        'status' => NomeModelStatus::class, // enum
    ];

    // Configuração do slug — gerado a partir do campo 'name' por padrão
    public function getSlugOptions(): SlugOptions
    {
        return SlugOptions::create()
            ->generateSlugsFrom('name')
            ->saveSlugsTo('slug');
    }

    // Relationships abaixo
}
```

**Instalar o pacote de slug (se ainda não estiver instalado):**
```bash
./vendor/bin/sail composer require callcocam/tall-sluggable
```

---

### Enum de Status — padrão obrigatório

Criar sempre em `app/Enums/`:

```php
<?php

namespace App\Enums;

enum NomeModelStatus: string
{
    case Draft     = 'draft';
    case Published = 'published';

    public function label(): string
    {
        return match($this) {
            self::Draft     => 'Rascunho',
            self::Published => 'Publicado',
        };
    }

    public function color(): string
    {
        return match($this) {
            self::Draft     => 'warning',
            self::Published => 'success',
        };
    }
}
```

> Se o mock mostrar outros status (ex: `inactive`, `archived`, `pending`) além de draft/published,
> adicionar os casos adicionais ao enum conforme os badges/labels visíveis no mock.

---

### Migration — padrão obrigatório

```php
Schema::create('nome_models', function (Blueprint $table) {
    $table->ulid('id')->primary();                    // sempre ulid, nunca id()

    // Multi-tenant (incluir somente se o projeto for multi-tenant)
    $table->foreignUlid('tenant_id')
          ->constrained('tenants')
          ->cascadeOnDelete();

    $table->string('name');
    $table->string('slug')->unique();                 // sempre incluir
    $table->enum('status', ['draft', 'published'])    // sempre incluir
          ->default('draft');

    // Demais campos inferidos do mock:
    // Input texto curto   → $table->string('campo')
    // Textarea            → $table->text('campo')
    // Input número        → $table->integer / decimal('campo')
    // Input data          → $table->date / datetime('campo')
    // Select de enum      → $table->enum('campo', ['a', 'b'])
    // Select FK           → $table->foreignUlid('model_id')->constrained()
    // Checkbox            → $table->boolean('campo')->default(false)
    // Upload              → $table->string('campo')->nullable()

    $table->softDeletes();                            // sempre incluir
    $table->timestamps();
});
```

**Regras da migration:**
- ❌ Nunca usar `$table->id()` — sempre `$table->ulid('id')->primary()`
- ❌ Nunca usar `$table->foreignId()` — sempre `$table->foreignUlid()`
- ✅ `slug` sempre presente e `unique()`
- ✅ `status` sempre presente com `default('draft')`
- ✅ `softDeletes()` sempre presente
- ✅ `tenant_id` somente se o projeto for multi-tenant

---

### Controller
Sempre retornar Inertia responses com os dados necessários para as páginas:

```php
public function index(): Response
{
    return Inertia::render('{NomeModel}/Index', [
        'items'   => $this->service->list(request()->all()),
        'filters' => request()->only(['search', 'status', ...]),
    ]);
}
```

---

## Passo 6 — Gerar Componentes Base do Recurso

Criar na pasta `resources/js/components/{NomeModel}/`:

### 6.1 — `{NomeModel}Form.vue` (FormBase)

Componente com **todos os campos do formulário**, reutilizado tanto na página Create quanto Edit:

```vue
<script setup lang="ts">
import type { {NomeModel}Form } from '@/types/{nome-model}'

interface Props {
  form: {NomeModel}Form        // useForm() do Inertia — passado pela página pai
  errors?: Record<string, string>
  // dados de relacionamentos (ex: categorias para um select)
  categorias?: Categoria[]
}

defineProps<Props>()
</script>

<template>
  <!-- Estrutura fiel ao mock do formulário -->
  <!-- Usar apenas componentes ui/ que existem no projeto -->
</template>
```

**Regras do FormBase:**
- Não contém lógica de submit — só renderiza campos
- Recebe `form` (useForm do Inertia) como prop
- Exibe `form.errors.campo` em cada campo
- Reflete exatamente a estrutura visual do mock

### 6.2 — `{NomeModel}List.vue` (ListBase)

Componente de listagem com tabela/cards + paginação, fiel ao mock:

```vue
<script setup lang="ts">
import type { {NomeModel}, PaginatedResponse } from '@/types/{nome-model}'

interface Props {
  items: PaginatedResponse<{NomeModel}>
}

defineProps<Props>()
defineEmits<{
  edit: [item: {NomeModel}]
  delete: [item: {NomeModel}]
}>()
</script>
```

**Regras do ListBase:**
- Paginação sempre incluída (links do Laravel)
- Colunas inferidas dos campos visíveis no mock da listagem
- Ações (editar, excluir, visualizar) conforme o mock
- Responsivo — adaptar ao mock mobile se existir

### 6.3 — `{NomeModel}FilterBar.vue` (FilterBar)

Componente de filtros da listagem, fiel ao mock:

```vue
<script setup lang="ts">
interface Props {
  filters: {
    search?: string
    status?: string
    // outros filtros visíveis no mock
  }
}

defineProps<Props>()
defineEmits<{ filter: [filters: Props['filters']] }>()
</script>
```

**Regras do FilterBar:**
- Debounce no campo de busca (300ms)
- Emite evento `filter` a cada mudança
- Filtros inferidos dos controles visíveis no mock

---

## Passo 7 — Gerar as Páginas Vue

### `Index.vue` — Listagem completa

```vue
<script setup lang="ts">
import { router } from '@inertiajs/vue3'
import {NomeModel}List from '@/components/{NomeModel}/{NomeModel}List.vue'
import {NomeModel}FilterBar from '@/components/{NomeModel}/{NomeModel}FilterBar.vue'

// props vindas do Controller
const props = defineProps<{
  items: PaginatedResponse<{NomeModel}>
  filters: Record<string, string>
}>()

function applyFilters(filters: Record<string, string>) {
  router.get(route('{nome-model}.index'), filters, {
    preserveState: true,
    replace: true,
  })
}
</script>
```

### `Form.vue` — Create + Edit unificado

```vue
<script setup lang="ts">
import { useForm } from '@inertiajs/vue3'
import {NomeModel}Form from '@/components/{NomeModel}/{NomeModel}Form.vue'

const props = defineProps<{ {nomeModel}?: {NomeModel} }>()
const isEditing = computed(() => !!props.{nomeModel}?.id)

const form = useForm({
  // campos inferidos do mock
})

function submit() {
  isEditing.value
    ? form.put(route('{nome-model}.update', props.{nomeModel}!.id))
    : form.post(route('{nome-model}.store'))
}
</script>
```

### `Show.vue` — Somente se o mock tiver tela de detalhe

---

## Passo 8 — Registrar a Rota

```php
// routes/web.php
Route::resource('{nome-model}', {NomeModel}Controller::class);
```

---

## Ordem de entrega

```
1. Relatório de auditoria → aguardar confirmação
2. Componentes ui/ ausentes (se aprovados)
3. app/Enums/{NomeModel}Status.php
4. Migration + Model (com HasUlids, HasSlug, SoftDeletes, enum cast, tenant_id se multi-tenant)
5. Service (+ Repository se padrão C)
6. Controller + Form Requests
7. Tipos TypeScript (resources/js/types/{nome-model}.ts)
8. {NomeModel}Form.vue    ← inclui campo status com Select + campo slug (readonly/auto)
9. {NomeModel}List.vue    ← inclui coluna status com Badge usando enum.color()
10. {NomeModel}FilterBar.vue ← inclui filtro por status
11. Pages/Index.vue
12. Pages/Form.vue
13. Pages/Show.vue (se aplicável)
14. Rota no web.php
```

---

## Regras Gerais

- ❌ Nunca gerar código sem antes apresentar o relatório de auditoria
- ❌ Nunca criar componentes `ui/` sem confirmar com o usuário
- ❌ Nunca recriar componentes que já existem — sempre reaproveitar
- ❌ Nunca usar `$table->id()` — sempre `$table->ulid('id')->primary()`
- ❌ Nunca usar `$table->foreignId()` — sempre `$table->foreignUlid()`
- ❌ Nunca criar model sem `HasUlids`, `SoftDeletes` e `HasSlug`
- ❌ Nunca criar migration sem `slug`, `status` e `softDeletes()`
- ❌ Nunca hardcodar labels/cores de status — sempre usar o enum (`NomeModelStatus::label()`, `::color()`)
- ✅ Perguntar sobre multi-tenant **uma única vez** por conversa — reusar a resposta
- ✅ Incluir `tenant_id` somente se o projeto for multi-tenant
- ✅ `status` sempre começa como `draft` por padrão
- ✅ `slug` sempre gerado automaticamente via `callcocam/tall-sluggable`
- ✅ Fidelidade visual ao mock tem prioridade sobre convenções genéricas
- ✅ Sempre TypeScript — nunca `.vue` sem `lang="ts"`
- ✅ Sempre `<script setup>` — nunca Options API
- ✅ Formulários sempre com `useForm` do Inertia
- ✅ Comandos Artisan sempre com `./vendor/bin/sail` (se Sail ativo)

---

## Referências

- `references/ui-components.md` — Templates dos componentes primitivos ui/
- `references/type-inference.md` — Como inferir tipos TypeScript a partir do mock