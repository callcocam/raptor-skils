---
name: laravel-raptor-page
description: >
  Analisa mock de design (screen.png + HTML/CSS + opcional DESIGN.md) e gera páginas internas e
  CRUDs completos em Laravel + Inertia + Vue 3 + TypeScript. Use para módulos de negócio e telas de
  recurso (listagem, formulário, detalhe). Pressupõe base global pronta via laravel-raptor-patterns.
---

# Laravel Raptor Page Skill

## Visão Geral

Esta skill transforma um mock de design em páginas internas e CRUDs completos (backend + frontend), mantendo o estilo visual do projeto e reaproveitando componentes existentes.

### Ambiente de execução

- O padrão do projeto é **Laravel Sail**
- Antes de sugerir ou executar comandos, confirmar se o projeto usa Sail
- Se o usuário não disser o contrário, assumir **Sail como padrão**
- Só usar `php artisan`, `composer` e `npm` diretamente se o usuário disser explicitamente que **não usa Sail**

### Escopo desta skill

- Páginas internas de módulos de negócio
- CRUD completo de recursos (backend + frontend)
- Conversão de mock para componentes e páginas de recurso
- Listagens, formulários, filtros, paginação e ações de recurso

### Fora de escopo desta skill

- Bootstrap visual inicial do projeto
- Auth, layouts globais, páginas de erro e perfil
- Estrutura global de tenant

Para itens fora de escopo, usar `laravel-raptor-patterns`.

### Pré-condição obrigatória

Antes de usar esta skill, a base global deve estar pronta via `laravel-raptor-patterns`:

- Auth adaptado ao tema
- Layout global (`AppLayout`/`AuthLayout`) pronto
- Páginas de erro (`403`, `404`, `500`) prontas
- Perfil de usuário pronto
- Gerenciamento de tenant pronto (se multi-tenant)

Se a base global não existir, interromper o CRUD e executar primeiro `laravel-raptor-patterns`.

### Skills relacionadas

- **`laravel-raptor-migrations`** — use para auditar as migrations geradas por esta skill (unique composto com tenant_id, ULID, softDeletes, indexes)
- **`laravel-raptor-i18n`** — use após criar o CRUD para garantir que labels, mensagens e títulos estão em PT-BR via `__()` e `t()`

### Handoff

Se durante a implementação do CRUD surgir demanda de base global (auth/layout/erro/perfil/tenant), pausar esta skill e acionar `laravel-raptor-patterns`.

### Decisão rápida

| Pedido do usuário | Skill correta |
|---|---|
| "Criar CRUD", "criar módulo", "criar tela interna" | `laravel-raptor-page` |
| "Gerar backend + frontend de recurso" | `laravel-raptor-page` |
| "Ajustar login", "criar AppLayout", "criar páginas de erro" | `laravel-raptor-patterns` |
| "Configurar perfil" ou "gerenciamento global de tenant" | `laravel-raptor-patterns` |

Se a base global não estiver pronta, retornar o fluxo para `laravel-raptor-patterns` antes de continuar no CRUD.

---

## Passo 0 — Verificar estado anterior do recurso

**ANTES de iniciar**, verificar se o recurso solicitado já foi gerado anteriormente.

Primeiro identificar o nome do recurso (pelo mock ou pelo que o usuário informou), depois verificar:

```bash
# Substituir {Recurso} pelo nome identificado (ex: Loja, Produto, Pedido)
ls app/Models/{Recurso}.php \
   app/Http/Controllers/{Recurso}Controller.php \
   resources/js/pages/{Recurso}/Index.vue 2>/dev/null \
  && echo "Recurso anterior encontrado" || echo "Recurso novo"
```

| Resultado | Ação |
|-----------|------|
| Model + Controller + Page encontrados | Perguntar ao usuário |
| Não encontrado | Prosseguir normalmente |

**Se encontrar recurso anterior:**

> "Encontrei arquivos do recurso `{Recurso}` já existentes. O que deseja fazer?"
> - **Atualizar** — analisar o que mudou no mock e aplicar apenas as diferenças
> - **Recriar** — recriar backend e frontend do zero (sobrescreve os arquivos existentes)
> - **Sem limitação** — gerar todos os arquivos sem verificar o que já existe

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
- Mapear ícones do mock para Lucide (`lucide-vue-next`)

---

## Passo 3 — Auditar o projeto (o que já existe)

Antes de criar qualquer arquivo, verificar o que já existe:

```bash
# Componentes reutilizáveis por domínio (primeira busca obrigatória)
ls resources/js/components/ui/domain/ 2>/dev/null
ls resources/js/components/ui/domain/NotificationCenter.vue 2>/dev/null
ls resources/js/components/ui/domain/ConfirmActionModal.vue 2>/dev/null

# Componentes ui/ disponíveis
ls resources/js/components/ui/ 2>/dev/null

# Componentes shared/ disponíveis
ls resources/js/components/shared/ 2>/dev/null

# Wrappers base de formulário/listagem
ls resources/js/components/shared/ResourceFormShell.vue 2>/dev/null
ls resources/js/components/shared/ResourceFormModal.vue 2>/dev/null
ls resources/js/components/shared/ResourceListShell.vue 2>/dev/null

# Layouts disponíveis
ls resources/js/layouts/ 2>/dev/null

# Base global pronta? (auth, erro, perfil)
ls resources/js/pages/auth/ 2>/dev/null
ls resources/js/pages/errors/ 2>/dev/null
ls resources/js/pages/profile/ 2>/dev/null

# Model já existe?
ls app/Models/{NomeModel}.php 2>/dev/null

# Controller já existe?
ls app/Http/Controllers/{NomeModel}Controller.php 2>/dev/null

# Service já existe?
ls app/Services/{NomeModel}Service.php 2>/dev/null

# Policy já existe?
ls app/Policies/{NomeModel}Policy.php 2>/dev/null

# Páginas Vue já existem?
ls resources/js/pages/{NomeModel}/ 2>/dev/null

# Componentes específicos do recurso já existem?
ls resources/js/components/{NomeModel}/ 2>/dev/null
```

### Ordem obrigatória de busca de componentes reutilizáveis

Antes de criar qualquer componente novo, seguir esta ordem:

1. Procurar em `resources/js/components/ui/domain/` (primeira busca obrigatória)
2. Procurar em `resources/js/components/ui/`
3. Procurar em `resources/js/components/shared/`
4. Procurar em `resources/js/components/{NomeModel}/`
5. Só então criar novo componente

Critério de criação:

- Se o componente tende a ser usado por mais de um recurso/tela: criar reutilizável em `resources/js/components/ui/domain/` (ou `shared/` se for layout/comportamento global)
- Se o componente for específico do recurso: criar em `resources/js/components/{NomeModel}/`

### Relatório de auditoria

Antes de gerar qualquer código, apresentar ao usuário um relatório resumido:

```
📋 Análise do mock: {nome_da_pasta}
   Recurso identificado: {NomeModel}

✅ Já existe no projeto:
   - app/Models/Loja.php
   - resources/js/components/ui/Button.vue
   - resources/js/components/ui/Input.vue

🔐 Autorização (Policy):
  - Policy existente? sim/não
  - Será criada? sim/não
  - Estratégia: usar policy para ações index/view/create/update/delete

🔒 Estratégia de unicidade por campo:
  - slug: global / por tenant
  - código: global / por tenant
  - email: global / por tenant
  - Regra: se multi-tenant, preferir unicidade composta com `tenant_id`
  - Validar em dois pontos: migration (índice) + Form Request (`Rule::unique`)

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
  - app/Policies/LojaPolicy.php

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

> ⚠️ Se a base global não estiver pronta, parar aqui e orientar execução da skill `laravel-raptor-patterns`.

---

## Passo 4 — Criar componentes ui/ ausentes (se confirmado)

Para cada componente `ui/` ausente identificado na auditoria, criar baseado no estilo do mock.

Ver `references/ui-components.md` para os templates de cada componente primitivo.

**Regra:** cada componente `ui/` deve ser genérico e reutilizável — sem lógica de negócio, sem referência ao recurso atual.

**Regra adicional obrigatória:** sempre tentar reaproveitar componente existente em `resources/js/components/ui/domain/` antes de criar novo.

**Internacionalização obrigatória:** textos de interface, labels, placeholders, mensagens de sucesso e erro devem estar em pt-BR.

```
resources/js/components/ui/
├── Select.vue       ← props: options, modelValue, placeholder, disabled, error
├── DatePicker.vue   ← props: modelValue, min, max, disabled, error
├── Badge.vue        ← props: variant ('success'|'warning'|'danger'|'info'), label
└── ...
```

---

## Passo 5 — Gerar o Backend

Antes de qualquer comando, validar o ambiente:

> "Este projeto está usando Laravel Sail? Se sim, vou usar `./vendor/bin/sail ...` em todos os comandos."

Por padrão, assumir que **sim**.

Comandos usando Sail:

```bash
# Model + Migration
./vendor/bin/sail artisan make:model {NomeModel} -m

# Controller Resource
./vendor/bin/sail artisan make:controller {NomeModel}Controller --resource --model={NomeModel}

# Form Requests
./vendor/bin/sail artisan make:request {NomeModel}/Store{NomeModel}Request
./vendor/bin/sail artisan make:request {NomeModel}/Update{NomeModel}Request

# Policy (autorização/permissões)
./vendor/bin/sail artisan make:policy {NomeModel}Policy --model={NomeModel}
```

Service e Repository criados manualmente conforme padrão do `laravel-raptor-patterns`.

Se houver dúvida sobre necessidade de policy para o recurso, perguntar antes de seguir:

> "Vamos precisar criar policy para o model {NomeModel} (autorização/permissões)?"

---

### Model — padrão obrigatório

> `AbstractModel` é criado no bootstrap do projeto (`laravel-raptor-setup`, Passo 5.3). Todo model deve herdar dele — `HasFactory`, `HasUlids`, `SoftDeletes`, `HasSlug` (e `BelongsToTenant` se multi-tenant) já vêm incluídos. O pacote `callcocam/tall-sluggable` deve estar instalado.

Todo model gerado segue esta estrutura:

```php
<?php

namespace App\Models;

use App\Enums\NomeModelStatus;
use Callcocam\Tall\Sluggable\SlugOptions;

class NomeModel extends AbstractModel
{
    // HasFactory, HasUlids, SoftDeletes e HasSlug já herdados do AbstractModel

    protected $fillable = [
        // 'tenant_id', ← somente se multi-tenant e BelongsToTenant não gerencia automaticamente
        'name',
        'slug',
        'status',
        // demais campos inferidos do mock
    ];

    protected $casts = [
        'status' => NomeModelStatus::class,
    ];

    // Sobrescrever somente se o slug não vem do campo 'name':
    // public function getSlugOptions(): SlugOptions { ... }

    // Relationships abaixo
}
```

---

### Enum de Status — padrão obrigatório

> Padrão completo definido em `laravel-raptor-patterns`. Criar sempre em `app/Enums/`:

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
  $table->string('slug');                           // sempre incluir
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

    // Unicidade:
    // Multi-tenant  → $table->unique(['tenant_id', 'slug'])
    // Single-tenant → $table->unique('slug')

    $table->softDeletes();                            // sempre incluir
    $table->timestamps();
});
```

**Regras da migration:**
- ❌ Nunca usar `$table->id()` — sempre `$table->ulid('id')->primary()`
- ❌ Nunca usar `$table->foreignId()` — sempre `$table->foreignUlid()`
- ✅ `slug` sempre presente
- ✅ `status` sempre presente com `default('draft')`
- ✅ `softDeletes()` sempre presente
- ✅ `tenant_id` somente se o projeto for multi-tenant
- ✅ Em multi-tenant, campos únicos devem usar índice composto com `tenant_id`

**Regra de validação (Form Requests) para campos únicos:**

Em projetos multi-tenant, `unique` deve considerar `tenant_id`.

```php
use Illuminate\Validation\Rule;

'slug' => [
  'required',
  Rule::unique('nome_models', 'slug')
    ->where(fn ($q) => $q->where('tenant_id', tenant('id'))),
],
```

Para update, usar `->ignore($model->id)`.

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

Criar em duas camadas: wrappers compartilhados + componentes do recurso.

### 6.1 — Wrappers compartilhados (obrigatórios)

Criar/reaproveitar em `resources/js/components/shared/`:

- `ResourceFormShell.vue`
- `ResourceFormModal.vue`
- `ResourceListShell.vue`

### 6.1-A — Componentes transversais obrigatórios

Criar/reaproveitar em `resources/js/components/ui/domain/`:

- `NotificationCenter.vue` (toasts/alerts globais)
- `ConfirmActionModal.vue` (confirmação de ações destrutivas)
- `PermissionGate.vue` (controle de renderização por permissão)

`ConfirmActionModal.vue` deve suportar dois modos:

- Confirmação simples (botão confirmar)
- Confirmação por digitação (usuário precisa digitar um texto aleatório/token para habilitar a ação)

Exemplos de uso do modo por digitação:

- Exclusão permanente
- Reset irreversível
- Ações de alto impacto operacional

Esses wrappers centralizam comportamento repetido (submit, estados de processamento, ações, validações e paginação/filtros), para evitar duplicação nas páginas.

#### `ResourceFormShell.vue`

Wrapper para páginas de create/edit usando Inertia (`useForm` ou `<Form>`).

- Deve receber `form`, `submit`, `isEditing`, `processing`, `errors`
- Deve expor slots: `header`, `content`, `actions`
- Deve encapsular botões padrão (Salvar/Cancelar), estado loading e bloco de erros gerais

#### `ResourceFormModal.vue`

Mesmo contrato do `ResourceFormShell`, mas em modal (create/edit rápido).

- Mesmo conjunto de slots: `header`, `content`, `actions`
- Deve suportar abrir/fechar e reset opcional do form no fechamento

#### `ResourceListShell.vue`

Wrapper de listagem para tabela **ou** cards.

- Deve receber `items`, `filters`, `pagination`, `loading`
- Deve centralizar filtros e paginação
- Deve expor slots: `header`, `filters`, `content`, `pagination`, `empty`, `actions`
- O slot `content` decide se renderiza tabela ou lista de cards

> Se o projeto já possuir wrappers equivalentes, reaproveitar e apenas adaptar o contrato de props/slots.

### 6.1.1 — Contrato obrigatório dos wrappers

`ResourceFormShell.vue` (page)

- Props mínimas: `form`, `submit`, `isEditing`, `processing`, `errors?`, `cancelHref?`, `submitLabel?`
- Emits mínimos: `submit`, `cancel`
- Slots obrigatórios: `header`, `content`, `actions`
- Deve renderizar feedback de erro por campo e resumo de erros do formulário

`ResourceFormModal.vue` (modal)

- Props mínimas: `form`, `open`, `submit`, `isEditing`, `processing`, `errors?`, `closeOnSuccess?`
- Emits mínimos: `update:open`, `submit`, `cancel`, `closed`
- Slots obrigatórios: `header`, `content`, `actions`
- Deve manter o mesmo padrão de feedback de erro do wrapper de página

`ResourceListShell.vue` (index/listagem)

- Props mínimas: `items`, `filters`, `loading?`, `meta?`, `links?`
- Emits mínimos: `filter`, `paginate`, `create`, `refresh`
- Slots obrigatórios: `header`, `filters`, `content`, `pagination`, `empty`, `actions`
- Deve exibir estado vazio e estado de erro de carregamento com mensagens em pt-BR
- Deve permitir gate de permissão para ações (ex: criar, editar, excluir)

### 6.2 — Componente de campos do recurso

Criar na pasta `resources/js/components/{NomeModel}/`:

`{NomeModel}FormFields.vue`

Componente com **todos os campos** do recurso (sem submit e sem botões globais).

```vue
<script setup lang="ts">
import type { {NomeModel}Form } from '@/types/{nome-model}'

interface Props {
  form: {NomeModel}Form
  errors?: Record<string, string>
  categorias?: Categoria[]
}

defineProps<Props>()
</script>

<template>
  <!-- Apenas campos do recurso -->
</template>
```

**Regras:**

- Não contém lógica de submit
- Não contém botões padrão de ação
- Usa `form.errors` por campo

### 6.3 — Conteúdo de listagem do recurso

Criar:

`{NomeModel}ListContent.vue`

Renderiza somente o conteúdo da listagem no slot `content` do `ResourceListShell`.

- Pode ser tabela ou cards conforme o mock
- Emite ações do item (`edit`, `delete`, `show`)

### 6.4 — Filtros específicos do recurso (opcional)

Se os filtros forem complexos, criar:

`{NomeModel}FilterFields.vue`

Usado dentro do slot `filters` do `ResourceListShell`.

---

## Passo 7 — Gerar as Páginas Vue

### `Index.vue` — Listagem completa

```vue
<script setup lang="ts">
import { router } from '@inertiajs/vue3'
import ResourceListShell from '@/components/shared/ResourceListShell.vue'
import {NomeModel}ListContent from '@/components/{NomeModel}/{NomeModel}ListContent.vue'
import {NomeModel}FilterFields from '@/components/{NomeModel}/{NomeModel}FilterFields.vue'

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

<template>
  <ResourceListShell
    :items="props.items"
    :filters="props.filters"
    @filter="applyFilters"
  >
    <template #filters="{ filters, update }">
      <{NomeModel}FilterFields :filters="filters" @change="update" />
    </template>

    <template #content>
      <{NomeModel}ListContent :items="props.items" />
    </template>
  </ResourceListShell>
</template>
```

### `Form.vue` — Create + Edit unificado

```vue
<script setup lang="ts">
import { useForm } from '@inertiajs/vue3'
import ResourceFormShell from '@/components/shared/ResourceFormShell.vue'
import {NomeModel}FormFields from '@/components/{NomeModel}/{NomeModel}FormFields.vue'

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

<template>
  <ResourceFormShell
    :form="form"
    :is-editing="isEditing"
    :submit="submit"
    :processing="form.processing"
  >
    <template #header>
      <!-- Titulo/Subtitulo -->
    </template>

    <template #content>
      <{NomeModel}FormFields :form="form" :errors="form.errors" />
    </template>

    <template #actions>
      <!-- Ações extras opcionais -->
    </template>
  </ResourceFormShell>
</template>
```

> Para fluxo em modal, usar `ResourceFormModal.vue` com os mesmos slots (`header`, `content`, `actions`).

> Para ações destrutivas (delete/archive/reset), usar `ConfirmActionModal.vue`; para operações críticas, habilitar modo de confirmação por digitação.

> Botões, ações e links de navegação interna devem ser renderizados com checagem de permissão (policy/ability), preferencialmente via `PermissionGate.vue`.

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
1. Validar se a base global já está pronta
2. Relatório de auditoria → aguardar confirmação
3. Componentes ui/ ausentes (se aprovados)
4. app/Enums/{NomeModel}Status.php
5. Migration + Model (com HasUlids, HasSlug, SoftDeletes, enum cast, tenant_id se multi-tenant)
6. Service (+ Repository se padrão C)
7. Controller + Form Requests
8. Tipos TypeScript (resources/js/types/{nome-model}.ts)
9. Buscar e reaproveitar componentes em `ui/domain` → `ui` → `shared` → `{NomeModel}`
10. shared/ResourceFormShell.vue e shared/ResourceListShell.vue (e modal, se necessário)
11. Validar contrato mínimo de props/emits/slots dos wrappers
12. Garantir `NotificationCenter.vue` e `ConfirmActionModal.vue` reutilizáveis (ou reaproveitar existentes)
13. {NomeModel}FormFields.vue    ← inclui campo status com Select + campo slug (readonly/auto)
14. {NomeModel}ListContent.vue    ← tabela ou cards com ações
15. {NomeModel}FilterFields.vue   ← filtros específicos (se necessário)
16. Pages/Index.vue (com ResourceListShell)
17. Pages/Form.vue (com ResourceFormShell)
18. Pages/Show.vue (se aplicável)
19. Rota no web.php
```

---

## Regras Gerais

- ❌ Nunca gerar código sem antes apresentar o relatório de auditoria
- ❌ Nunca iniciar CRUD se a base global ainda não estiver pronta
- ❌ Nunca criar componentes `ui/` sem confirmar com o usuário
- ❌ Nunca recriar componentes que já existem — sempre reaproveitar
- ❌ Nunca criar componente novo sem procurar antes em `resources/js/components/ui/domain/`
- ❌ Nunca duplicar lógica de submit/ações em cada página de create/edit
- ❌ Nunca implementar listagem sem wrapper com filtros e paginação centralizados
- ❌ Nunca deixar mensagens de UI em inglês quando o projeto está em pt-BR
- ❌ Nunca executar ação destrutiva sem confirmação explícita
- ❌ Nunca exibir botão/ação/link de navegação sem validar permissão
- ❌ Nunca usar `php artisan`, `composer` ou `npm` diretamente sem confirmar que o projeto não usa Sail
- ❌ Nunca usar `$table->id()` — sempre `$table->ulid('id')->primary()`
- ❌ Nunca usar `$table->foreignId()` — sempre `$table->foreignUlid()`
- ❌ Nunca criar model sem estender `AbstractModel` (que já inclui `HasUlids`, `SoftDeletes`, `HasSlug`)
- ❌ Nunca criar migration sem `slug`, `status` e `softDeletes()`
- ❌ Nunca criar unicidade global para campo de recurso multi-tenant sem compor com `tenant_id`
- ❌ Nunca hardcodar labels/cores de status — sempre usar o enum (`NomeModelStatus::label()`, `::color()`)
- ❌ Nunca implementar CRUD sem validar estratégia de autorização/permissões
- ✅ Perguntar sobre multi-tenant **uma única vez** por conversa — reusar a resposta
- ✅ Incluir `tenant_id` somente se o projeto for multi-tenant
- ✅ `BelongsToTenant` já incluído no `AbstractModel` se projeto for multi-tenant
- ✅ `status` sempre começa como `draft` por padrão
- ✅ `slug` sempre gerado automaticamente via `callcocam/tall-sluggable`
- ✅ Fidelidade visual ao mock tem prioridade sobre convenções genéricas
- ✅ Sempre TypeScript — nunca `.vue` sem `lang="ts"`
- ✅ Sempre `<script setup>` — nunca Options API
- ✅ Formulários sempre com Inertia (`useForm` ou `<Form>`) via wrapper compartilhado
- ✅ Formulário deve usar slots `header`, `content`, `actions` (page e modal)
- ✅ Listagem deve usar wrapper compartilhado com filtros + paginação e slot de conteúdo (tabela/cards)
- ✅ Se componente puder ser usado em mais de um recurso, criar como reutilizável em `resources/js/components/ui/domain/`
- ✅ Feedback de erro deve ser exibido no campo e em resumo de formulário quando aplicável
- ✅ Notificações de sucesso/erro/aviso devem usar `NotificationCenter.vue`
- ✅ Ações destrutivas devem usar `ConfirmActionModal.vue` com modo de digitação para casos críticos
- ✅ Ícones devem usar `lucide-vue-next` (não misturar múltiplas bibliotecas sem necessidade)
- ✅ Navegação, links e botões devem respeitar permissões (policy/ability) no frontend e no backend
- ✅ Em multi-tenant, validação e índice de campos únicos devem considerar `tenant_id`
- ✅ Todos os textos de interface devem estar em pt-BR
- ✅ Usar policy para autorização de recursos sempre que houver operações de leitura/escrita no model
- ✅ Se o escopo do recurso não deixar claro, perguntar se deve criar `{NomeModel}Policy`
- ✅ Assumir Sail como padrão até que o usuário informe o contrário
- ✅ Comandos Artisan sempre com `./vendor/bin/sail` (se Sail ativo)
- ✅ Esta skill é para páginas internas e CRUDs; auth/layout/erros/perfil/tenant ficam na `laravel-raptor-patterns`

---

## Referências

- `references/ui-components.md` — Templates dos componentes primitivos ui/
- `references/type-inference.md` — Como inferir tipos TypeScript a partir do mock