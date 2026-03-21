---
name: laravel-raptor-i18n
description: >
  Use when fazer varredura minuciosa e corrigir textos hardcoded em inglês em projetos
  Laravel Raptor: varre Vue (useT composable), Blade (__()),  PHP controllers/services (__()),
  e Enum labels — tanto no app como nos pacotes callcocam/laravel-raptor-*. Fluxo:
  varredura completa → relatório por arquivo → corrige com confirmação. Garante
  que todas as chaves existam em lang/pt_BR/ antes de aplicar.
---

# Laravel Raptor — Internacionalização (i18n)

## Passo 0 — Verificar estado anterior

**ANTES de qualquer ação**, verificar se uma varredura já foi feita:

```bash
ls .raptor/i18n-report.md 2>/dev/null \
  && head -8 .raptor/i18n-report.md \
  || echo "Nenhuma varredura anterior encontrada"
```

| Resultado | Ação |
|-----------|------|
| `.raptor/i18n-report.md` encontrado | Perguntar ao usuário |
| Não encontrado | Prosseguir normalmente |

**Se encontrar varredura anterior:**

> "Encontrei uma varredura i18n anterior. O que deseja fazer?"
> - **Refazer** — nova varredura completa (substitui o relatório anterior)
> - **Continuar pendentes** — usar relatório existente e corrigir apenas os itens ainda não traduzidos
> - **Sem limitação** — varrer e tratar tudo novamente sem considerar o que já foi corrigido

Ao final de cada varredura, salvar relatório em `.raptor/i18n-report.md`.

---

## Fluxo obrigatório

```
Fase 1: Descoberta → mapear caminho do projeto e pacotes
Fase 2: Varredura  → ler todos os arquivos, listar textos hardcoded
Fase 3: Relatório  → mostrar tudo antes de tocar qualquer arquivo
Fase 4: Correção   → arquivo a arquivo, com confirmação por arquivo
```

**NÃO corrigir nada durante a varredura.** Primeiro mapear tudo, depois corrigir.

---

## Fase 1 — Descoberta do projeto

### 1a. Localizar o projeto principal

```bash
# Verificar estrutura base
ls resources/js/
ls resources/views/
ls app/
ls lang/

# Verificar locale atual
grep -r "'locale'" config/app.php
```

### 1b. Localizar os pacotes Raptor

```bash
# Pacotes locais em /packages
find packages -name "*.vue" -path "*/resources/js/*" 2>/dev/null | head -20
find packages -name "*.php" -path "*/src/*" 2>/dev/null | head -20

# Pacotes em vendor (modo development)
find vendor/callcocam -name "*.vue" 2>/dev/null | head -20
find vendor/callcocam -name "*.php" -path "*/src/*" 2>/dev/null | head -20
```

Perguntar ao usuário se os pacotes estão em `packages/`, `vendor/callcocam/` ou outro caminho.

### 1c. Verificar traduções existentes

```bash
# Estrutura atual de traduções
find lang -name "*.php" | sort
find lang -name "*.json" | sort

# Conteúdo do app.php principal
cat lang/pt_BR/app.php 2>/dev/null || cat lang/pt-BR/app.php 2>/dev/null || echo "AUSENTE"
```

> Referência de chaves padrão disponíveis: `laravel-raptor/references/i18n.md`

---

## Fase 2 — Varredura de arquivos

### Escopo completo da varredura

| Tipo | Caminhos |
|------|---------|
| Vue (app) | `resources/js/**/*.vue` |
| Blade (app) | `resources/views/**/*.blade.php` |
| PHP — Controllers | `app/Http/Controllers/**/*.php` |
| PHP — Services | `app/Services/**/*.php` |
| PHP — Enums | `app/Enums/**/*.php` |
| PHP — Models (labels) | `app/Models/**/*.php` |
| Vue (pacotes) | `packages/callcocam/*/resources/js/**/*.vue` |
| Blade (pacotes) | `packages/callcocam/*/resources/views/**/*.blade.php` |
| PHP (pacotes) | `packages/callcocam/*/src/**/*.php` |

### O que buscar em cada tipo

**Vue (`*.vue`):**
- Strings literais em inglês no `<template>` (exceto atributos técnicos como `type`, `name`, `id`)
- Strings hardcoded em `defineProps`, `ref()`, `computed()` que viram texto visível
- Mensagens de toast/alert hardcoded em inglês
- Textos em `aria-label`, `placeholder`, `title` em inglês
- Labels em componentes de formulário

```bash
# Exemplo de busca — strings em inglês em Vue
grep -rn '"[A-Z][a-z]' resources/js/ --include="*.vue" | grep -v "//.*" | head -50
grep -rn "'[A-Z][a-z]" resources/js/ --include="*.vue" | grep -v "//.*" | head -50
```

**PHP — Controllers/Services:**
- Strings de retorno Inertia (`'title' => 'Products'`)
- Mensagens de flash/session (`'message' => 'Created successfully'`)
- Textos em `abort()` e `throw new`

```bash
# Exemplo — strings em inglês em PHP
grep -rn "'[A-Z][a-z]\+'" app/ --include="*.php" | grep -v "//\|*\|class\|use " | head -50
```

**PHP — Enums:**
- Método `label()` retornando string literal em inglês

```bash
grep -rn "return '[A-Z]\|return \"[A-Z]" app/Enums/ --include="*.php"
```

**Blade templates:**
- Texto direto fora de tags PHP que não usa `{{ __() }}`

### O que NÃO marcar como problema

- Nomes de rotas: `route('products.index')`
- Nomes de classes, métodos, variáveis: `ProductController`, `$productName`
- Chaves de array PHP: `['key' => 'value']` onde key é técnico
- Comentários: `// This creates a product`
- Strings técnicas: SQL, regex, hash, UUIDs
- Valores internos de enum PHP: `case Draft = 'draft'` (só o `label()` importa)
- Atributos HTML técnicos: `type="submit"`, `name="email"`, `id="search"`
- Nomes de pacotes/configuração: `'driver' => 'mysql'`

---

## Fase 3 — Relatório

Apresentar o relatório completo antes de corrigir qualquer coisa.

### Formato do relatório

```
=== RELATÓRIO DE i18n ===
Projeto: /home/user/project
Pacotes: packages/callcocam/laravel-raptor-core, packages/callcocam/laravel-raptor-table
Total: X arquivos | Y ocorrências

━━━ APP PRINCIPAL ━━━

📄 resources/js/pages/Products/Index.vue (5 ocorrências)
  linha 12:  "Products"          → t('app.resources.products')
  linha 45:  "Create Product"    → t('app.create', { resource: t('app.resources.product_singular') })
  linha 67:  "No products found" → t('app.no_results')
  linha 89:  "Search..."         → t('app.search') [placeholder]
  linha 102: "Delete"            → t('app.delete')

📄 app/Http/Controllers/ProductController.php (2 ocorrências)
  linha 34: 'Products'                → __('app.resources.products')  [título Inertia]
  linha 61: 'Product created.'        → __('app.created_success', ['resource' => __('resources/products.singular')])

📄 app/Enums/ProductStatus.php (2 ocorrências)
  linha 12: return 'Draft';     → return __('app.status.draft');
  linha 16: return 'Published'; → return __('app.status.published');

━━━ PACOTES ━━━

📄 packages/callcocam/laravel-raptor-table/resources/js/components/DataTable.vue (3 ocorrências)
  ...

━━━━━━━━━━━━━━━━━━━━
Chaves novas a criar em lang/pt_BR/: X chaves
Chaves reutilizadas de app.php:       Y chaves
```

---

## Fase 4 — Correção arquivo a arquivo

### Fluxo por arquivo

1. Mostrar o arquivo com as ocorrências destacadas
2. Propor a versão corrigida
3. Aguardar confirmação (`s` = aplicar, `n` = pular arquivo)
4. Se confirmado:
   a. Verificar se as chaves usadas existem em `lang/pt_BR/`
   b. Se não existirem → criar as entradas primeiro
   c. Aplicar a correção no arquivo
5. Confirmar o próximo

### Regra de criação de chave nova

Antes de criar uma chave nova, verificar na ordem:

1. **Existe chave genérica com placeholder?**
   - `t('app.create', { resource: '...' })` em vez de criar `t('app.create_product')`
2. **Existe chave similar?**
   - Verificar se `app.no_results`, `app.empty`, `app.loading` já cobrem o caso
3. **Onde criar a chave nova:**
   - String comum de UI → `lang/pt_BR/app.php`
   - String de recurso específico → `lang/pt_BR/resources/{recurso}.php`
   - Nunca criar chave duplicada com mesmo significado

### Seções do app.php para referência

```
app.resources.{recurso}        → nome plural do recurso (ex: 'Produtos')
app.resources.{recurso}_singular → nome singular (ex: 'Produto')
app.create                     → 'Novo :resource'
app.edit                       → 'Editar :resource'
app.delete                     → 'Excluir'
app.save                       → 'Salvar'
app.cancel                     → 'Cancelar'
app.search                     → 'Buscar...'
app.filter                     → 'Filtrar'
app.no_results                 → 'Nenhum resultado encontrado'
app.loading                    → 'Carregando...'
app.status.draft               → 'Rascunho'
app.status.published           → 'Publicado'
app.status.active              → 'Ativo'
app.status.inactive            → 'Inativo'
app.created_success            → ':resource criado com sucesso!'
app.updated_success            → ':resource atualizado com sucesso!'
app.deleted_success            → ':resource excluído com sucesso!'
```

Veja o arquivo completo de referência: `laravel-raptor/references/i18n.md`

---

## Padrões de substituição por tipo

Veja `references/patterns.md` para exemplos completos de cada tipo de arquivo.

### Vue — useT composable

```ts
// setup
import { useT } from '@/composables/useT'
const { t } = useT()

// Template
// ❌ Antes: <h1>Products</h1>
// ✅ Depois: <h1>{{ t('app.resources.products') }}</h1>

// ❌ Antes: <Button>Create Product</Button>
// ✅ Depois: <Button>{{ t('app.create', { resource: t('app.resources.product_singular') }) }}</Button>

// ❌ Antes: placeholder="Search products..."
// ✅ Depois: :placeholder="t('app.search')"

// ❌ Antes: <p v-if="!items.length">No products found</p>
// ✅ Depois: <p v-if="!items.length">{{ t('app.no_results') }}</p>
```

### PHP — Controller (Inertia)

```php
// ❌ Antes
return Inertia::render('Products/Index', [
    'title' => 'Products',
]);

// ✅ Depois
return Inertia::render('Products/Index', [
    'title' => __('app.resources.products'),
]);
```

### PHP — Enum label()

```php
// ❌ Antes
public function label(): string
{
    return match($this) {
        self::Draft     => 'Draft',
        self::Published => 'Published',
    };
}

// ✅ Depois
public function label(): string
{
    return match($this) {
        self::Draft     => __('app.status.draft'),
        self::Published => __('app.status.published'),
    };
}
```

### PHP — Flash messages

```php
// ❌ Antes
session()->flash('success', 'Product created successfully.');

// ✅ Depois
session()->flash('success', __('app.created_success', ['resource' => __('resources/products.singular')]));
```

### Blade

```blade
{{-- ❌ Antes --}}
<title>Products</title>
<button>Create</button>

{{-- ✅ Depois --}}
<title>{{ __('app.resources.products') }}</title>
<button>{{ __('app.create') }}</button>
```

---

## Configuração mínima necessária

Antes de corrigir, garantir que o projeto tem:

### config/app.php
```php
'locale'          => 'pt_BR',
'fallback_locale' => 'en',
'faker_locale'    => 'pt_BR',
'timezone'        => 'America/Sao_Paulo',
```

### HandleInertiaRequests.php
```php
public function share(Request $request): array
{
    return array_merge(parent::share($request), [
        'translations' => fn () => [
            'app'        => trans('app'),
            'validation' => trans('validation'),
        ],
        'locale' => app()->getLocale(),
    ]);
}
```

### composables/useT.ts
```ts
import { usePage } from '@inertiajs/vue3'

export function useT() {
    const page = usePage()

    function t(key: string, replace: Record<string, string> = {}): string {
        const keys = key.split('.')
        let value: unknown = page.props.translations as Record<string, unknown>

        for (const k of keys) {
            if (value && typeof value === 'object' && k in value) {
                value = (value as Record<string, unknown>)[k]
            } else {
                return key // fallback: retorna a chave se não encontrar
            }
        }

        let result = String(value)
        for (const [k, v] of Object.entries(replace)) {
            result = result.replace(`:${k}`, v)
        }

        return result
    }

    return { t }
}
```

---

## Regras Gerais

- ❌ Nunca corrigir sem antes apresentar o relatório completo
- ❌ Nunca criar chave nova sem verificar se já existe equivalente em `app.php`
- ❌ Nunca usar `t()` em Vue sem importar o composable `useT`
- ❌ Nunca usar `__()` em migração ou config — apenas em camadas de apresentação
- ✅ Sempre verificar se `useT` composable existe antes de usar nos Vue components
- ✅ Sempre criar as chaves em `lang/pt_BR/` antes de aplicar a substituição
- ✅ Sempre usar placeholders `:resource`, `:name` em vez de criar chave específica por recurso
- ✅ Corrigir gramática PT-BR ao criar chaves (ver tabela de correções em `laravel-raptor/references/i18n.md`)
- ✅ Pacotes em modo development (`packages/callcocam/`) devem ser tratados junto com o app
