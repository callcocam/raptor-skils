---
name: laravel-raptor-patterns
description: >
  Use when implementar a estrutura global de um projeto Laravel Raptor: estratégia de componentes
  (shadcn vs. custom), páginas de auth, layouts globais (AppLayout, AuthLayout, Sidebar, Topbar),
  páginas de erro (403, 404, 500), perfil do usuário, gerenciamento de tenant, e padrões de
  autorização via policies. Usar APÓS laravel-raptor (identidade visual já definida). Não usar
  para CRUDs ou módulos de negócio — usar laravel-raptor-page.
---

# Laravel Patterns Skill

## Posição no Workflow

```
laravel-raptor-setup → laravel-raptor → laravel-raptor-patterns → laravel-raptor-components → laravel-raptor-page
```

**Pré-condição:** `laravel-raptor` concluída (tema, cores, CSS vars, sidebar/header visuais já definidos).
**Esta skill:** estrutura e páginas globais (quais páginas criar, como organizar componentes).
**Depois dela:** `laravel-raptor-components` (componentes individuais) ou `laravel-raptor-page` (CRUDs).

---

## Visão Geral

Este skill cobre somente a base global do projeto — estrutura de páginas e organização, não identidade visual.

A identidade visual (tema, cores, dark mode, tailwind tokens) é responsabilidade da skill `laravel-raptor`.
Esta skill usa a identidade visual já estabelecida e constrói a estrutura de páginas sobre ela.

Todo projeto novo passa por um **bootstrap de estrutura obrigatório** antes de qualquer tela de negócio (ver Passo 0).

### Ambiente de execução

- O padrão do projeto é **Laravel Sail**
- Antes de sugerir ou executar comandos, confirmar se o projeto usa Sail
- Se o usuário não disser o contrário, assumir **Sail como padrão**
- Só usar `php artisan`, `composer` e `npm` diretamente se o usuário disser explicitamente que **não usa Sail**

### Escopo desta skill

- Estratégia de componentes (shadcn-vue vs. custom — baseada nos mocks)
- Organização da pasta de componentes (`ui/`, `shared/`, `layout/`)
- Layouts globais (`AppLayout`, `AuthLayout`, navegação)
- Telas de autenticação
- Páginas de erro (`403`, `404`, `500`)
- Perfil de usuário (dados básicos, senha, preferências)
- Gerenciamento de tenant (somente contexto global de tenant)

### Fora de escopo desta skill

- CRUDs de domínio (Lojas, Produtos, Pedidos, etc.)
- Módulos internos de negócio
- Fluxos específicos de um recurso

Para itens fora de escopo, usar `laravel-raptor-page`.

### Handoff

Quando o usuário pedir criação de telas internas ou CRUDs de recurso, encerrar esta skill e seguir com `laravel-raptor-page`.

### Decisão rápida

| Pedido do usuário | Skill correta |
|---|---|
| "Configurar auth", "ajustar layout", "criar página 403/404/500" | `laravel-raptor-patterns` |
| "Criar perfil do usuário" ou "organizar tenant ativo" | `laravel-raptor-patterns` |
| "Criar CRUD de lojas/produtos/pedidos" | `laravel-raptor-page` |
| "Gerar listagem + formulário de recurso" | `laravel-raptor-page` |
| "Novo módulo interno seguindo mock" | `laravel-raptor-page` |

Se houver dúvida: primeiro preparar base global com `laravel-raptor-patterns`, depois construir módulo interno com `laravel-raptor-page`.

---

## Passo 0a — Verificar estado anterior

**ANTES de qualquer ação**, verificar se a estrutura global já foi criada:

```bash
ls resources/js/layouts/AppLayout.vue \
   resources/js/pages/auth/Login.vue \
   resources/js/pages/errors/404.vue 2>/dev/null \
  && echo "Estrutura anterior encontrada" || echo "Projeto sem estrutura global"
```

| Resultado | Ação |
|-----------|------|
| AppLayout + Auth + Erros encontrados | Perguntar ao usuário |
| Não encontrado | Prosseguir normalmente |

**Se encontrar estado anterior:**

> "Encontrei estrutura global Raptor já criada neste projeto. O que deseja fazer?"
> - **Refazer** — recriar toda a estrutura global do zero
> - **Atualizar** — verificar apenas o que está faltando ou divergindo e completar pontualmente
> - **Sem limitação** — revisar e reaplicar tudo sem filtrar pelo que já existe

---

## Passo 0 — Bootstrap de Estrutura (projeto novo ou sem estrutura global)

> **Atenção:** O tema visual (cores, CSS vars, dark mode, tailwind tokens) deve estar definido antes deste passo — use `laravel-raptor` primeiro se ainda não estiver feito.

Antes de criar qualquer tela de negócio, verificar se o projeto já tem a estrutura global estabelecida:

```bash
# Verificar se auth já foi personalizado
ls resources/js/pages/auth/

# Verificar se já existe sistema de componentes próprio
ls resources/js/components/

# Verificar se layout principal já existe
ls resources/js/layouts/
```

Se qualquer um desses ainda estiver no padrão Laravel/Breeze virgem → **executar o bootstrap de estrutura primeiro**.

### O bootstrap de estrutura tem 3 etapas, nesta ordem:

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
│   ├── domain/           ← componentes reutilizáveis de domínio/transversais
│   │   ├── NotificationCenter.vue
│   │   ├── ConfirmActionModal.vue
│   │   └── index.ts
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
│   ├── ConfirmDialog.vue
│   └── FormErrorSummary.vue
└── index.ts             ← exporta tudo
```

> Criar os primitivos em `ui/` **antes** de qualquer página. Eles são a base de tudo.

#### Ordem obrigatória de busca/reuso de componentes

Antes de criar componente novo, sempre procurar nesta ordem:

1. `resources/js/components/ui/domain/`
2. `resources/js/components/ui/`
3. `resources/js/components/shared/`
4. Criar componente novo (somente se não houver equivalente)

Se o componente puder ser usado em múltiplas telas, criar em `ui/domain`.

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

## Passo 1 — Ler a pasta `/tema` e extrair o padrão visual global

A pasta `/tema` na raiz do projeto contém os mocks de design. Cada subpasta segue o padrão:

```
{nome_da_tela}_{device}_{modo}
```

### Prioridade de leitura

- Sempre preferir `desktop_light` como referência principal
- Usar `dark` para validar variáveis de tema
- Usar `mobile` para comportamento responsivo

### O que documentar internamente

- Paleta de cores e tokens
- Tipografia e escala
- Espaçamentos recorrentes
- Componentes visuais globais
- Padrões de navegação

---

## Passo 2 — Implementar páginas e fluxos globais

Depois da base visual pronta, implementar apenas o escopo global:

1. Auth: `Login`, `Register`, `ForgotPassword`, `ResetPassword`, `ConfirmPassword`, `VerifyEmail`
2. Layouts: `AppLayout`, `AuthLayout`, `Sidebar`, `Topbar`, navegação mobile
3. Páginas de erro: `403`, `404`, `500`
4. Perfil do usuário: dados básicos, atualização de senha e preferências
5. Gerenciamento de tenant (somente se multi-tenant): troca de tenant ativo e contexto atual
6. Feedbacks globais: mensagens de erro padronizadas por campo e resumo de erros por formulário
7. Notificações globais: componente de notificação reutilizável (`NotificationCenter.vue`)
8. Confirmações destrutivas: modal reutilizável (`ConfirmActionModal.vue`) com modo opcional de confirmação por digitação de texto aleatório/token
9. Autorização/permissões: padronizar uso de policies para recursos e ações sensíveis
10. Ícones: padronizar uso de Lucide (`lucide-vue-next`) para navegação e ações visuais
11. Multi-tenant: padronizar unicidade composta com `tenant_id` para campos únicos de recurso

> Não criar CRUDs de módulos de negócio neste skill.

---

## Passo 3 — Critério de pronto da base

Considerar a base global pronta quando todos os itens abaixo estiverem concluídos:

- [ ] Sistema de componentes base definido e consistente
- [ ] Auth adaptado ao tema
- [ ] Layout principal funcional em desktop e mobile
- [ ] Páginas de erro implementadas
- [ ] Perfil do usuário implementado
- [ ] Gerenciamento de tenant implementado (se aplicável)

---

## Handoff para a skill de páginas

Quando a solicitação envolver telas internas de negócio, módulos e CRUDs, usar a skill **laravel-raptor-page**.

Exemplos de handoff:

- Cadastro de lojas, produtos, pedidos, contratos
- Listagens internas com filtros e paginação
- Formulários de create/edit/show de recursos
- Backend de recurso (Model, Requests, Service, Controller, rotas)

---

## Regras Gerais

- ❌ Nunca criar CRUD de domínio nesta skill
- ❌ Nunca misturar bootstrap global com implementação de módulo interno no mesmo passo
- ❌ Nunca criar componente novo sem procurar antes em `resources/js/components/ui/domain/`
- ❌ Nunca usar `php artisan`, `composer` ou `npm` diretamente sem confirmar que o projeto não usa Sail
- ✅ Sempre estabelecer a base visual global antes de qualquer tela de negócio
- ✅ Sempre manter consistência de tema entre auth, layout, erros, perfil e tenant
- ✅ Assumir Sail como padrão até que o usuário informe o contrário
- ✅ Textos e mensagens de interface devem estar em pt-BR
- ✅ Toda ação com erro deve exibir feedback visível (campo + resumo quando aplicável)
- ✅ Toda ação destrutiva deve passar por confirmação; em operações críticas, exigir digitação de texto aleatório/token
- ✅ Notificações de sucesso/erro/aviso devem usar componente reutilizável global
- ✅ Adotar policy como padrão de autorização/permissões para recursos do sistema
- ✅ Se não estiver claro no escopo, perguntar se deve criar policy para o model antes de implementar
- ✅ Usar `lucide-vue-next` como biblioteca padrão de ícones
- ✅ Sempre verificar permissão antes de exibir itens de navegação, links, botões e ações da interface
- ✅ Em projetos multi-tenant, campos únicos devem usar unicidade composta com `tenant_id` (migration + validação)