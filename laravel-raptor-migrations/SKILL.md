---
name: laravel-raptor-migrations
description: >
  Use when auditar migrations existentes ou criar novas em projeto Laravel multi-tenant Raptor:
  verifica ULID, foreignUlid, unique composto com tenant_id (ex: [email, tenant_id]),
  softDeletes, indexes em FK, e campos únicos por tenant. Gera relatório com severidade
  e corrige arquivo a arquivo com confirmação. Use em qualquer fase do projeto.
---

# Laravel Raptor — Migrations

## Passo 0 — Verificar estado anterior

**ANTES de qualquer ação**, verificar se uma auditoria já foi feita:

```bash
ls .raptor/migrations-audit.md 2>/dev/null \
  && head -5 .raptor/migrations-audit.md \
  || echo "Nenhuma auditoria anterior encontrada"
```

| Resultado | Ação |
|-----------|------|
| `.raptor/migrations-audit.md` encontrado | Perguntar ao usuário |
| Não encontrado | Prosseguir normalmente |

**Se encontrar auditoria anterior:**

> "Encontrei uma auditoria anterior de migrations. O que deseja fazer?"
> - **Refazer** — nova varredura completa (substitui o relatório anterior)
> - **Continuar pendentes** — usar relatório existente e tratar apenas os itens ainda não corrigidos
> - **Sem limitação** — varrer tudo novamente sem considerar o que já foi tratado

Ao final de cada varredura, salvar relatório em `.raptor/migrations-audit.md`.

---

## Dois modos de uso

```
Projeto existente? → Modo Auditoria → Varredura → Relatório → Corrigir por arquivo
Nova migration?    → Modo Criação   → Template  → Checklist → Gerar código
```

---

## Modo 1 — Auditoria de Migrations Existentes

### Passo 1 — Localizar as migrations

```bash
# Listar todas as migrations do projeto
find database/migrations -name "*.php" | sort

# Se projeto tiver pacotes locais, incluir também
find packages/callcocam/*/database/migrations -name "*.php" 2>/dev/null | sort
```

Se o projeto usar Sail: prefixar com `./vendor/bin/sail exec laravel` ou simplesmente ler os arquivos diretamente (não precisa executar).

### Passo 2 — Varredura automática

Para cada migration encontrada, verificar os itens abaixo. **Ler o arquivo completamente antes de classificar.**

#### Checklist de auditoria

| # | Problema | Severidade | Verificação |
|---|---------|-----------|------------|
| 1 | `$table->id()` sem ULID | 🔴 Crítico | Buscar `->id()` sem `ulid` |
| 2 | `$table->foreignId()` | 🔴 Crítico | Buscar `foreignId(` |
| 3 | `unique('campo')` sem `tenant_id` | 🔴 Crítico | Buscar `unique(` com string simples em projeto multi-tenant |
| 4 | `$table->unique(['campo'])` com array mas sem `tenant_id` | 🔴 Crítico | Unique em campo sensível sem tenant |
| 5 | Ausência de `softDeletes()` em tabela de recurso | 🟡 Aviso | Tabelas que não são pivô devem ter softDeletes |
| 6 | `foreignUlid` sem `->index()` ou `constrained()` | 🟡 Aviso | FK sem index impacta performance |
| 7 | Campo único sem tenant: `email`, `slug`, `cpf`, `cnpj`, `codigo` | 🟡 Aviso | Unique individual em campo sensível |
| 8 | Enum declarado sem Enum PHP 8.1 no `$casts` do Model | 🔵 Sugestão | Verificar se Model usa `BackedEnum` |
| 9 | `$table->string('status')` sem enum | 🔵 Sugestão | Preferir `$table->enum()` ou cast para BackedEnum |

#### Campos tipicamente únicos por tenant

Sempre verificar se estes campos têm `unique` composto com `tenant_id`:

- `email` (usuários, contatos, fornecedores)
- `slug` (qualquer recurso com URL)
- `cpf`, `cnpj` (pessoas físicas/jurídicas)
- `codigo`, `code` (identificadores de negócio)
- `matricula`, `registration`

### Passo 3 — Gerar relatório

Formato de saída do relatório:

```
=== RELATÓRIO DE MIGRATIONS ===
Total: X arquivos analisados | Y problemas encontrados

📄 2024_01_01_000000_create_users_table.php
  🔴 linha 8:  $table->id() → deve ser $table->ulid('id')->primary()
  🔴 linha 12: $table->unique('email') → deve ser $table->unique(['tenant_id', 'email'])
  🟡 linha 20: softDeletes() ausente

📄 2024_01_02_000000_create_products_table.php
  🔴 linha 9:  $table->foreignId('tenant_id') → deve ser $table->foreignUlid('tenant_id')
  🟡 linha 14: $table->foreignUlid('category_id') sem ->index()
  🔵 linha 22: status como string sem BackedEnum

📄 2024_01_03_000000_create_orders_table.php
  ✅ Sem problemas encontrados
```

### Passo 4 — Corrigir arquivo a arquivo

Para cada arquivo com problemas:
1. Mostrar o arquivo completo atual
2. Propor a versão corrigida completa
3. Aguardar confirmação do usuário (`s` para aplicar, `n` para pular)
4. Aplicar a correção cirurgicamente (usar str_replace, não reescrever tudo)
5. Avançar para o próximo arquivo

> ⚠️ **Atenção:** Alterações em migrations já executadas podem exigir nova migration de alteração (`php artisan make:migration alter_*`) em vez de modificar a migration original. Verificar se a migration já foi rodada antes de editar.

#### Identificar se migration já foi rodada

```bash
# Verificar migrations executadas
php artisan migrate:status | grep "Ran"
# ou via Sail:
./vendor/bin/sail artisan migrate:status | grep "Ran"
```

Se a migration já foi executada → criar migration de `ALTER TABLE` em vez de editar o arquivo original.

---

## Modo 2 — Criar Nova Migration

### Passo 1 — Checklist antes de gerar código

Antes de escrever qualquer linha, responder:

- [ ] O projeto é **multi-tenant**? (tem `tenant_id` nas tabelas?)
- [ ] Quais campos precisam ser **únicos por tenant**?
- [ ] Quais campos são **FKs** (referência a outras tabelas)?
- [ ] O recurso tem **slug** (URL amigável)?
- [ ] O recurso tem **status** (enum)?
- [ ] A tabela é de recurso (softDeletes) ou pivô (sem softDeletes)?

### Passo 2 — Template padrão Raptor

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('{tabela}', function (Blueprint $table) {
            // ✅ SEMPRE: ULID como PK
            $table->ulid('id')->primary();

            // ✅ SEMPRE em multi-tenant: FK para tenant
            $table->foreignUlid('tenant_id')->constrained()->cascadeOnDelete();

            // Campos do recurso
            $table->string('name');
            $table->string('slug');

            // FKs — sempre foreignUlid + constrained
            // $table->foreignUlid('category_id')->constrained()->cascadeOnDelete();

            // Status — BackedEnum PHP 8.1
            $table->string('status')->default('draft');

            // Outros campos

            // ✅ SEMPRE em multi-tenant: unique composto com tenant_id
            $table->unique(['tenant_id', 'slug']);
            // $table->unique(['tenant_id', 'email']); // se aplicável

            // Indexes em campos de busca frequente (não em booleanos)
            // $table->index('category_id'); // se não usar constrained()

            // ✅ SEMPRE em tabelas de recurso (não em pivô)
            $table->softDeletes();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('{tabela}');
    }
};
```

### Passo 3 — Regras de index

| Situação | Ação |
|---------|------|
| FK via `foreignUlid()->constrained()` | Index criado automaticamente |
| FK via `foreignUlid()` sem `constrained()` | Adicionar `->index()` manualmente |
| Campo usado em `where()` frequente | Adicionar `->index()` |
| Campo único por tenant | `$table->unique(['tenant_id', 'campo'])` |
| Campo único global | `$table->unique('campo')` (raro em multi-tenant) |
| Campo booleano | **Não** indexar (baixa cardinalidade) |
| Campo `status` (enum com poucos valores) | **Não** indexar isolado; indexar composto se necessário |

### Passo 4 — Validação correspondente (Form Request)

Para cada `unique` criado na migration, criar a regra correspondente no Form Request:

```php
// Store (criação)
Rule::unique('{tabela}', 'email')
    ->where(fn ($q) => $q->where('tenant_id', tenant('id'))),

// Update (edição) — ignorar o registro atual
Rule::unique('{tabela}', 'email')
    ->where(fn ($q) => $q->where('tenant_id', tenant('id')))
    ->ignore($this->route('{recurso}')?->id),
```

### Passo 5 — Model correspondente

Verificar que o Model:
- Estende `AbstractModel` (herda HasUlids, SoftDeletes, HasSlug)
- Tem `$casts` para o BackedEnum de status
- Tem `$fillable` com todos os campos da migration
- Tem as relações `belongsTo` para as FKs

---

## Regras Gerais

- ❌ Nunca `$table->id()` — sempre `$table->ulid('id')->primary()`
- ❌ Nunca `$table->foreignId()` — sempre `$table->foreignUlid()`
- ❌ Nunca `unique('email')` em projeto multi-tenant — sempre `unique(['tenant_id', 'email'])`
- ❌ Nunca editar migration já executada — criar migration de `ALTER TABLE`
- ✅ Sempre `softDeletes()` em tabelas de recurso (exceto tabelas pivô)
- ✅ Sempre verificar campos únicos: email, slug, cpf, cnpj, codigo
- ✅ Sempre criar regra `Rule::unique` correspondente no Form Request
- ✅ Sempre indexar FKs (via `constrained()` ou `->index()` manual)
- ✅ Unique composto `['tenant_id', 'campo']` para qualquer campo único em contexto multi-tenant
