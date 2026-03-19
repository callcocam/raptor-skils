---
name: laravel-raptor-patterns
description: >
  Define e aplica padrões de criação de código para projetos Laravel: Models, Controllers, Services,
  Repositories, Form Requests, e componentes Vue/TypeScript frontend. Use esta skill SEMPRE que o
  usuário pedir para: criar um model, controller, service, repository, componente Vue, página Inertia,
  ou qualquer artefato de código num projeto Laravel. Também acione quando mencionar "criar CRUD",
  "gerar recurso", "novo módulo", "padrão de código", "seguir o tema", "seguir o mock", "pasta tema",
  "configurar layout", "criar sidebar", "telas de auth", "bootstrap do projeto", "dashboard",
  "componentes base", ou qualquer pedido de scaffolding. Em projetos novos, SEMPRE executar o
  Passo 0 (bootstrap visual) antes de qualquer tela de negócio: verificar /tema, definir estratégia
  de componentes, adaptar auth e criar layout principal. Nunca criar telas de negócio sem a base
  visual estabelecida.
---

# Laravel Patterns Skill

## Visão Geral

Este skill define os padrões de criação de código para projetos Laravel. Antes de gerar qualquer arquivo, é preciso identificar:

1. **O padrão de arquitetura backend** do projeto
2. **O padrão visual/tema frontend** — lendo a pasta `/tema` na raiz do projeto

Todo projeto novo passa por um **bootstrap visual obrigatório** antes de qualquer tela de negócio. Ver Passo 0.

---

## Passo 0 — Bootstrap Visual (projeto novo ou sem base visual)

Antes de criar qualquer tela de negócio, verificar se o projeto já tem a base visual estabelecida:

```bash
# Verificar se auth já foi personalizado
ls resources/js/pages/auth/

# Verificar se já existe sistema de componentes próprio
ls resources/js/components/

# Verificar se layout principal já existe
ls resources/js/layouts/
```

Se qualquer um desses ainda estiver no padrão Laravel/Breeze virgem → **executar o bootstrap visual primeiro**.

### O bootstrap visual tem 3 etapas, nesta ordem:

---

### 0.1 — Definir a estratégia de componentes

Ler a pasta `/tema` (ver Passo 2) e decidir a estratégia com base no que encontrar:

| Situação nos mocks | Estratégia |
|---|---|
| Mocks usam classes CSS próprias / design system custom | Criar `resources/js/components/` do zero, sem shadcn |
| Mocks usam padrão próximo ao shadcn (cards, botões, inputs parecidos) | Instalar shadcn-vue e sobrescrever o tema via CSS variables |
| Pasta `/tema` não existe | Perguntar ao usuário qual abordagem prefere |

**Nunca misturar as duas abordagens no mesmo projeto.** Definir uma e manter.

#### Estrutura da pasta de componentes personalizada (quando criar do zero):

```
resources/js/components/
├── ui/                  ← primitivos base (Button, Input, Card, Badge, etc.)
│   ├── Button.vue
│   ├── Input.vue
│   ├── Card.vue
│   ├── Badge.vue
│   ├── Select.vue
│   ├── Checkbox.vue
│   ├── Modal.vue
│   └── index.ts         ← exporta todos
├── layout/              ← estrutura de layout
│   ├── Sidebar.vue
│   ├── Topbar.vue
│   ├── MobileNav.vue
│   └── AppLayout.vue
├── shared/              ← componentes de domínio reutilizáveis
│   ├── DataTable.vue
│   ├── Pagination.vue
│   ├── EmptyState.vue
│   └── ConfirmDialog.vue
└── index.ts             ← exporta tudo
```

> Criar os primitivos em `ui/` **antes** de qualquer página. Eles são a base de tudo.

---

### 0.2 — Auth: adaptar ou recriar as telas de autenticação

Verificar o que existe em `resources/js/pages/auth/`:

```bash
ls resources/js/pages/auth/
```

O Laravel/Breeze gera por padrão: `Login.vue`, `Register.vue`, `ForgotPassword.vue`, `ResetPassword.vue`, `ConfirmPassword.vue`, `VerifyEmail.vue`.

**Fluxo:**

1. Verificar quais mocks de auth existem em `tema/` (ex: `login_desktop_light`, `register_mobile_dark`)
2. Se existirem mocks → reescrever cada arquivo de auth seguindo o mock (ver Passo 2)
3. Se não existirem mocks → adaptar as telas existentes para usar os componentes `ui/` do projeto
4. **Sempre** garantir responsividade mobile — testar o mock `_mobile` se existir, senão adaptar manualmente

> ⚠️ Nunca apagar arquivos de auth sem confirmar com o usuário quais rotas/guards estão em uso.

---

### 0.3 — Layout principal: Sidebar + Topbar + Dashboard

Verificar o layout padrão do projeto nos mocks:

```bash
# Procurar mocks de layout/dashboard
ls tema/ | grep -E "dashboard|layout|sidebar|home"
```

**Checklist do layout principal:**

- [ ] Identificar nos mocks se a sidebar é fixa ou colapsável
- [ ] Identificar se existe topbar separada ou integrada à sidebar
- [ ] Verificar como o layout se comporta no mobile (drawer? bottom nav? menu hamburger?)
- [ ] Criar `resources/js/layouts/AppLayout.vue` com sidebar + topbar integrados
- [ ] Criar `resources/js/layouts/AuthLayout.vue` para as telas de auth (sem sidebar)
- [ ] Criar `resources/js/pages/Dashboard.vue` seguindo o mock correspondente

**Estrutura mínima do AppLayout:**

```vue
<script setup lang="ts">
import Sidebar from '@/components/layout/Sidebar.vue'
import Topbar from '@/components/layout/Topbar.vue'
import MobileNav from '@/components/layout/MobileNav.vue'
</script>

<template>
  <div class="min-h-screen bg-background">
    <!-- Sidebar: oculta no mobile, visível em lg+ -->
    <Sidebar class="hidden lg:flex" />

    <!-- MobileNav: visível só no mobile -->
    <MobileNav class="lg:hidden" />

    <div class="lg:pl-[var(--sidebar-width)]">
      <Topbar />
      <main class="p-4 lg:p-6">
        <slot />
      </main>
    </div>
  </div>
</template>
```

> O `--sidebar-width` deve ser definido como CSS variable no tema, extraído do mock.

---

### Ordem de entrega do bootstrap

```
1. components/ui/*        ← primitivos base
2. components/layout/*    ← sidebar, topbar, mobile nav
3. layouts/AppLayout.vue  ← layout principal
4. layouts/AuthLayout.vue ← layout de auth
5. pages/auth/*           ← telas de login, register, etc.
6. pages/Dashboard.vue    ← dashboard principal
```

Só depois do bootstrap concluído → iniciar telas de negócio.

---

## Passo 1 — Identificar o padrão de arquitetura

Se não foi informado anteriormente, perguntar:

> "Qual padrão de camadas você quer usar neste projeto?"

- **A) Model + Controller** — Simples, sem camadas extras. Bom para CRUDs pequenos.
- **B) Model + Service + Controller** — Service encapsula a lógica de negócio. Recomendado para a maioria dos projetos.
- **C) Model + Service + Repository + Controller** — Desacopla banco da lógica. Bom para projetos grandes ou com múltiplas fontes de dados.

Guardar a escolha como o **padrão do projeto** e aplicar consistentemente.

---

## Passo 2 — Ler a pasta `/tema` e extrair o padrão visual

A pasta `/tema` na raiz do projeto contém os mocks de design. Cada subpasta segue o padrão:

```
{nome_da_tela}_{device}_{modo}
```

Exemplos:
```
tema/
├── 403_sem_permissao_desktop_light/
│   ├── screen.png    ← screenshot da tela
│   ├── code          ← HTML/CSS do mock (sem extensão ou .html)
│   └── DESIGN.md     ← notas de design (pode não existir)
├── 403_sem_permissao_desktop_dark/
├── gerenciamento_de_lojas_desktop_light/
├── gerenciamento_de_lojas_mobile_dark/
└── ...
```

### Como ler os mocks

1. **Listar as pastas disponíveis:**
   ```bash
   ls tema/
   ```

2. **Para cada tela relevante, ler:**
   - O arquivo `code` (HTML/CSS) — extrair estrutura, classes, variáveis CSS, padrão de grid/flex
   - O `screen.png` — referenciar visualmente para entender layout, espaçamentos e hierarquia
   - O `DESIGN.md` se existir — tem anotações importantes sobre decisões de design

3. **Extrair e documentar internamente:**
   - Paleta de cores (variáveis CSS ou classes Tailwind usadas)
   - Padrão tipográfico (tamanhos, pesos, famílias)
   - Espaçamentos recorrentes (padding, gap, margin)
   - Padrão de cards, tabelas, formulários, botões
   - Comportamento em dark/light mode
   - Diferenças entre desktop/mobile/tablet

### Prioridade dos mocks
- Sempre preferir o mock `desktop_light` como referência principal
- Usar `dark` para entender variáveis de tema
- Usar `mobile` para entender responsividade

### Sem pasta `/tema`
Se a pasta não existir, perguntar ao usuário:
- Quer criar do zero com **shadcn-vue**?
- Tem algum design para referenciar?

---

## Passo 3 — Gerar os arquivos

### Comandos Artisan (sempre com Sail quando aplicável)

```bash
# Model + Migration
./vendor/bin/sail artisan make:model NomeModel -m

# Controller (Resource)
./vendor/bin/sail artisan make:controller NomeController --resource --model=NomeModel

# Form Request (validação)
./vendor/bin/sail artisan make:request StoreNomeRequest
./vendor/bin/sail artisan make:request UpdateNomeRequest

# Policy
./vendor/bin/sail artisan make:policy NomePolicy --model=NomeModel
```

> Services e Repositories **não têm comando Artisan nativo** — criar manualmente conforme padrões abaixo.

---

## Padrões de Código por Camada

### Model

```php
// app/Models/NomeModel.php
<?php

namespace App\Models;

use App\Enums\NomeModelStatus;
use Callcocam\Tall\Sluggable\HasSlug;
use Callcocam\Tall\Sluggable\SlugOptions;
use Illuminate\Database\Eloquent\Concerns\HasUlids;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class NomeModel extends Model
{
    use HasFactory, HasUlids, SoftDeletes, HasSlug;

    protected $fillable = [
        // 'tenant_id',  ← incluir somente se projeto multi-tenant
        'name',
        'slug',
        'status',
        // demais campos
    ];

    protected $casts = [
        'status' => NomeModelStatus::class,
    ];

    public function getSlugOptions(): SlugOptions
    {
        return SlugOptions::create()
            ->generateSlugsFrom('name')
            ->saveSlugsTo('slug');
    }

    // Relationships abaixo
}
```

**Enum de status — criar sempre em `app/Enums/`:**

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

**Migration correspondente:**

```php
Schema::create('nome_models', function (Blueprint $table) {
    $table->ulid('id')->primary();

    // $table->foreignUlid('tenant_id')->constrained('tenants')->cascadeOnDelete();
    // ↑ incluir somente se projeto multi-tenant

    $table->string('name');
    $table->string('slug')->unique();
    $table->enum('status', ['draft', 'published'])->default('draft');

    // demais campos inferidos do contexto

    $table->softDeletes();
    $table->timestamps();
});
```

**Regras absolutas de model/migration:**
- ❌ Nunca `$table->id()` → sempre `$table->ulid('id')->primary()`
- ❌ Nunca `$table->foreignId()` → sempre `$table->foreignUlid()`
- ❌ Nunca model sem `HasUlids`, `SoftDeletes`, `HasSlug`
- ❌ Nunca migration sem `slug`, `status` e `softDeletes()`
- ✅ `tenant_id` somente se o projeto for multi-tenant (perguntar uma vez por conversa)
- ✅ Status sempre usa enum tipado, nunca string hardcoded

### Service

```php
// app/Services/NomeService.php
namespace App\Services;

class NomeService
{
    public function __construct(
        protected readonly NomeRepository $repository // só no padrão C
    ) {}

    public function listar(array $filtros = []): Collection { ... }
    public function criar(array $dados): NomeModel { ... }
    public function atualizar(NomeModel $model, array $dados): NomeModel { ... }
    public function deletar(NomeModel $model): void { ... }
}
```

### Repository (só padrão C)

```php
// app/Repositories/NomeRepository.php
namespace App\Repositories;

class NomeRepository
{
    public function todos(array $filtros = []): Collection { ... }
    public function encontrar(int $id): NomeModel { ... }
    public function criar(array $dados): NomeModel { ... }
    public function atualizar(NomeModel $model, array $dados): NomeModel { ... }
    public function deletar(NomeModel $model): void { ... }
}
```

### Controller

```php
// app/Http/Controllers/NomeController.php
class NomeController extends Controller
{
    public function __construct(
        protected readonly NomeService $service
    ) {}

    public function index(): Response { ... }
    public function store(StoreNomeRequest $request): RedirectResponse { ... }
    public function update(UpdateNomeRequest $request, NomeModel $model): RedirectResponse { ... }
    public function destroy(NomeModel $model): RedirectResponse { ... }
}
```

---

## Estrutura de Pastas

```
app/
├── Http/
│   ├── Controllers/
│   │   └── NomeController.php
│   └── Requests/
│       ├── StoreNomeRequest.php
│       └── UpdateNomeRequest.php
├── Models/
│   └── NomeModel.php
├── Services/          ← criar pasta manualmente se não existir
│   └── NomeService.php
└── Repositories/      ← só padrão C — criar pasta manualmente se não existir
    └── NomeRepository.php
```

---

## Inertia + Vue 3 (Frontend)

Para cada recurso, criar a página Vue correspondente:

```
resources/js/pages/
└── NomeModel/
    ├── Index.vue     ← listagem
    ├── Create.vue    ← formulário de criação
    ├── Edit.vue      ← formulário de edição
    └── Show.vue      ← visualização (se necessário)
```

### Fluxo obrigatório antes de criar qualquer componente Vue:

1. Verificar `tema/` na raiz — listar as pastas
2. Identificar qual mock corresponde à tela que está sendo criada (pelo nome da pasta)
3. Ler o arquivo `code` do mock relevante
4. Visualizar o `screen.png` para entender o layout
5. Ler o `DESIGN.md` se existir
6. Converter o HTML/CSS do mock para Vue 3 + TypeScript + Tailwind, preservando a identidade visual

Ver `references/frontend-patterns.md` para padrões de componentes Vue e conversão HTML → Vue.

---

## Rota Resource

```php
// routes/web.php
Route::resource('nome-models', NomeController::class);
```

---

## Regras Gerais

- ❌ **Nunca criar camadas que não fazem parte do padrão escolhido** para o projeto
- ❌ **Nunca rodar `php artisan`** diretamente — sempre `./vendor/bin/sail artisan` (se Sail estiver ativo)
- ✅ **Sempre usar injeção de dependência** no construtor do Controller e Service
- ✅ **Sempre criar Form Requests** separados para `store` e `update` (nunca validar no Controller)
- ✅ **Sempre seguir o padrão visual** definido pelos mocks ou pelo sistema de design do projeto
- ✅ **Manter consistência** — se o projeto já tem um padrão estabelecido, identificar e seguir ele