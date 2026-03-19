---
name: laravel-raptor-setup
description: >
  Configura projetos Laravel novos ou existentes com Laravel Sail como ambiente de desenvolvimento.
  Use esta skill SEMPRE que o usuário pedir para: criar um novo projeto Laravel, configurar o
  ambiente de desenvolvimento, configurar o docker-compose do Sail, adicionar serviços ao Sail
  (banco de dados, Redis, filas, e-mail, busca, WebSocket), rodar migrations, instalar dependências,
  ou executar qualquer comando artisan/composer/npm no contexto de um projeto Laravel. Também
  acione quando o usuário mencionar "novo projeto", "setup Laravel", "configurar Sail", "subir
  ambiente", "instalar pacotes Laravel", "configurar Reverb", "WebSocket", "broadcasting" ou
  qualquer tarefa de bootstrap de projeto. O primeiro comando após criar o projeto é sempre
  `php artisan sail:install`. Nunca execute comandos sem o prefixo Sail após o ambiente subir,
  a menos que o usuário explicitamente diga que não está usando Sail.
---

# Laravel Raptor Setup Skill

## Visão Geral

Este skill orienta a criação e configuração de novos projetos Laravel usando **sempre a versão mais recente** do framework. O ambiente padrão é **Laravel Sail** (Docker). Se o usuário confirmar que **não está usando Sail**, os comandos devem ser adaptados para o ambiente local.

Stack padrão do projeto:
- **Backend:** Laravel (latest) + Sail
- **Frontend:** Inertia.js + Vue 3 + TypeScript + TailwindCSS
- **WebSocket:** Laravel Reverb (sempre incluído)
- **Filas/Cache:** Redis (recomendado)

---

## Passo 1 — Detectar se vai usar Sail

Antes de qualquer comando, confirmar:

> "Você vai usar o **Laravel Sail** para o ambiente de desenvolvimento deste projeto?"

- **Sim (padrão)** → seguir este skill com `./vendor/bin/sail` em todos os comandos
- **Não** → usar `php artisan`, `composer`, `npm` diretamente. Ignorar instruções de Sail.

---

## Passo 2 — Coletar configuração dos serviços

Perguntar quais serviços o projeto vai precisar:

### Banco de Dados (obrigatório — escolher um)
- `mysql` — MySQL (padrão mais comum)
- `pgsql` — PostgreSQL
- `mariadb` — MariaDB
- `sqlite` — SQLite (sem container, para projetos simples)

### Cache / Filas / Session
- `redis` — Redis (**recomendado** — necessário para Horizon e filas)
- `valkey` — Valkey (alternativa open-source ao Redis)

### Busca Full-Text (opcional)
- `meilisearch` — integra com Laravel Scout
- `typesense` — alternativa ao Meilisearch

### Email (desenvolvimento)
- `mailpit` — **recomendado** — captura e-mails localmente com interface web

### Outros (raramente necessário)
- `selenium` — testes de browser com Laravel Dusk
- `minio` — armazenamento S3-compatible local

> **Reverb não entra no `--with` do sail:install** — é instalado separadamente no Passo 4.

---

## Passo 3 — Criar o projeto e instalar o Sail

```bash
# 1. Criar o projeto Laravel (versão mais recente)
composer create-project laravel/laravel nome-do-projeto
cd nome-do-projeto

# 2. Instalar o Sail
composer require laravel/sail --dev

# 3. ⚡ PRIMEIRO COMANDO OBRIGATÓRIO — publicar docker-compose com os serviços escolhidos
php artisan sail:install --with=mysql,redis,mailpit
# Ajustar os serviços conforme escolha do Passo 2

# 4. Subir os containers
./vendor/bin/sail up -d
```

> ⚠️ O `php artisan sail:install` é o **único momento** em que se usa `php artisan` diretamente.
> Após o `sail up -d`, **todos os comandos usam `./vendor/bin/sail`**.

---

## Passo 4 — Instalar Laravel Reverb (WebSocket)

O Reverb é instalado **sempre**, dentro do container após o Sail subir:

```bash
# Instalar o Reverb (broadcasting)
./vendor/bin/sail artisan install:broadcasting
```

O `reverb:install` adiciona ao `.env`:

```env
REVERB_APP_ID=seu-app-id
REVERB_APP_KEY=sua-chave
REVERB_APP_SECRET=seu-secret
REVERB_HOST=localhost
REVERB_PORT=8080
REVERB_SCHEME=http

VITE_REVERB_APP_KEY="${REVERB_APP_KEY}"
VITE_REVERB_HOST="${REVERB_HOST}"
VITE_REVERB_PORT="${REVERB_PORT}"
VITE_REVERB_SCHEME="${REVERB_SCHEME}"
```

### Iniciar o servidor Reverb em desenvolvimento

```bash
# Em terminal separado (ou via Supervisor em produção)
./vendor/bin/sail artisan reverb:start --debug
```

---

## Passo 5 — Configurar ULID como padrão de ID

O projeto usa **ULID** em vez de auto-increment em todos os models. Aplicar imediatamente após o Sail subir.

### 5.1 — Adaptar a migration de `users`

Abrir `database/migrations/*_create_users_table.php` e substituir o `id()` por `ulid()`:

```php
// ❌ Padrão Laravel (remover)
$table->id();

// ✅ Substituir por
$table->ulid('id')->primary();
```

Fazer o mesmo para as outras migrations padrão do Laravel que usam `id()`:
- `create_password_reset_tokens_table` — coluna `email` já é PK, não precisa alterar
- `create_sessions_table` — substituir `$table->id()` por `$table->ulid('id')->primary()`
- `create_cache_table` — não usa `id()`, ignorar
- `create_jobs_table` — não precisa alterar (jobs usam int por performance)

### 5.2 — Adaptar o Model `User`

Abrir `app/Models/User.php` e adicionar a trait `HasUlids`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Concerns\HasUlids; // ← adicionar
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use HasUlids, Notifiable; // ← adicionar HasUlids

    // resto do model sem alteração
}
```

### 5.3 — Padrão para todos os novos Models

**Todo model criado no projeto deve seguir este padrão:**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Concerns\HasUlids;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class NomeModel extends Model
{
    use HasFactory, HasUlids, SoftDeletes;

    protected $fillable = [
        // campos explícitos
    ];
}
```

**Migration correspondente:**

```php
Schema::create('nome_models', function (Blueprint $table) {
    $table->ulid('id')->primary(); // ← sempre ulid, nunca id()
    // ... campos
    $table->softDeletes();
    $table->timestamps();
});
```

**FKs para models com ULID:**

```php
// Em vez de foreignId (que assume bigint)
$table->foreignUlid('user_id')->constrained()->cascadeOnDelete();
$table->foreignUlid('nome_model_id')->constrained('nome_models')->cascadeOnDelete();
```

### 5.4 — Rodar as migrations

```bash
./vendor/bin/sail artisan migrate
```

> ⚠️ Nunca usar `$table->id()` em nenhuma migration do projeto. Sempre `$table->ulid('id')->primary()`.

---

## Passo 6 — Configurar o .env

Revisar os valores principais após o `reverb:install`:

```env
APP_NAME=NomeDoProjeto
APP_URL=http://localhost

# Banco de dados
DB_CONNECTION=mysql        # ou pgsql, sqlite
DB_HOST=mysql              # nome do serviço no docker-compose
DB_PORT=3306
DB_DATABASE=nome_do_banco
DB_USERNAME=sail
DB_PASSWORD=password

# Redis — habilitar todas as drivers
REDIS_HOST=redis
REDIS_PORT=6379
CACHE_STORE=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis
BROADCAST_CONNECTION=reverb   # ← broadcasting via Reverb

# Mail
MAIL_HOST=mailpit
MAIL_PORT=1025

# Meilisearch (se escolhido)
MEILISEARCH_HOST=http://meilisearch:7700

# Typesense (se escolhido)
TYPESENSE_HOST=typesense
TYPESENSE_PORT=8108
```

---

## Passo 7 — Frontend (Inertia + Vue 3 + TypeScript + TailwindCSS)

```bash
# Inertia server-side
./vendor/bin/sail composer require inertiajs/inertia-laravel
./vendor/bin/sail artisan inertia:middleware

# Vue 3 + TypeScript + Inertia client
./vendor/bin/sail npm install vue @inertiajs/vue3 @vitejs/plugin-vue
./vendor/bin/sail npm install -D typescript vue-tsc

# TailwindCSS
./vendor/bin/sail npm install -D tailwindcss @tailwindcss/vite

# Echo + Pusher (cliente para o Reverb)
./vendor/bin/sail npm install laravel-echo pusher-js

# Subir dev server
./vendor/bin/sail npm run dev
```

### Configurar o Echo no frontend (`resources/js/bootstrap.ts`)

```typescript
import Echo from 'laravel-echo'
import Pusher from 'pusher-js'

window.Pusher = Pusher

window.Echo = new Echo({
  broadcaster: 'reverb',
  key: import.meta.env.VITE_REVERB_APP_KEY,
  wsHost: import.meta.env.VITE_REVERB_HOST,
  wsPort: import.meta.env.VITE_REVERB_PORT ?? 8080,
  wssPort: import.meta.env.VITE_REVERB_PORT ?? 443,
  forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'http') === 'https',
  enabledTransports: ['ws', 'wss'],
})
```

---

## Regras Absolutas de Comandos

| Contexto | Comando correto |
|---|---|
| **Exceção única** — instalar Sail | `php artisan sail:install` |
| Artisan | `./vendor/bin/sail artisan <cmd>` |
| Composer | `./vendor/bin/sail composer <cmd>` |
| NPM | `./vendor/bin/sail npm <cmd>` |
| Testes | `./vendor/bin/sail artisan test` |
| Tinker | `./vendor/bin/sail artisan tinker` |
| Build frontend | `./vendor/bin/sail npm run build` |
| Reverb | `./vendor/bin/sail artisan reverb:start` |

> ❌ **Nunca** usar `php artisan`, `composer`, ou `npm` diretamente após o Sail subir.

---

## Checklist Final de Setup

- [ ] Projeto criado com a versão mais recente do Laravel
- [ ] `php artisan sail:install --with=...` executado com os serviços corretos
- [ ] Containers rodando (`sail up -d`)
- [ ] Reverb instalado (`sail artisan install:broadcasting`)
- [ ] Migration de `users` e `sessions` adaptadas para `ulid('id')->primary()`
- [ ] Model `User` com trait `HasUlids`
- [ ] `.env` configurado — incluindo variáveis do Reverb e `BROADCAST_CONNECTION=reverb`
- [ ] `sail artisan key:generate` executado
- [ ] Migrations rodadas (`sail artisan migrate`)
- [ ] Dependências JS instaladas (`sail npm install`)
- [ ] `laravel-echo` e `pusher-js` instalados
- [ ] Echo configurado em `resources/js/bootstrap.ts`
- [ ] Frontend compilando (`sail npm run dev`)
- [ ] Reverb rodando em terminal separado (`sail artisan reverb:start --debug`)

---

## Referências

- `references/shadcn-setup.md` — Instalação e configuração do shadcn-vue
- `references/sail-services.md` — Detalhes de cada serviço disponível no Sail