---
name: laravel-raptor-page-layouts
description: >
  Gera componentes reutilizáveis de página (ListPage, FormPage) e composables TypeScript
  para projetos Laravel + Inertia + Vue 3 + Tailwind, partindo obrigatoriamente de um
  tema base (pasta com design-system ou documento de tema). Não depende de pacotes
  callcocam — funciona com Laravel puro (versão 12 ou 13). Use esta skill SEMPRE que
  o usuário pedir para: criar componentes de página de listagem, criar componentes de
  formulário, extrair padrões de páginas Vue, criar layouts base de lista ou formulário,
  criar composables de filtros/seleção/paginação/dirty-check, ou quando mencionar
  "ListPage", "FormPage", "layout de lista", "layout de formulário", "componentes de
  página", "sublayout", "page layout". Também acione quando o usuário quiser
  "componentizar páginas", "criar base de listagem", "criar base de formulário",
  "extrair o que é comum nas páginas", "padrão de lista e form", "gerar page layouts",
  "criar estrutura de páginas", "partir do tema base", "construir do zero com tema",
  "design-system como base", "tema como fundação". Entrada obrigatória: pasta de
  design-system OU documento de tema colado. Entrada opcional: código Vue/HTML ou
  screenshot como referência visual de estilo. Saída: arquivos .vue, composables .ts e
  documentação markdown em docs/. Stack: Laravel 12/13 + Vue 3 + Composition API +
  TypeScript + TailwindCSS v4 + Inertia.js. Todos os textos hardcoded em PT-BR.
  Zero dependências de pacotes callcocam/*.
---

# Laravel Raptor — Vue Page Layouts

## Passo 0 — Verificar estado anterior

**ANTES de qualquer análise**, verificar se layouts de página já foram extraídos:

```bash
ls resources/js/components/page/ListPage.vue \
   resources/js/components/page/FormPage.vue 2>/dev/null \
  && echo "Layouts anteriores encontrados" || echo "Nenhum layout de página anterior"
```

| Resultado | Ação |
|-----------|------|
| `ListPage.vue` e/ou `FormPage.vue` encontrados | Perguntar ao usuário |
| Não encontrado | Prosseguir normalmente |

**Se encontrar layouts anteriores:**

> "Encontrei layouts de página já extraídos neste projeto. O que deseja fazer?"
> - **Atualizar** — analisar o novo input e apenas complementar/corrigir os layouts existentes
> - **Recriar** — extrair e recriar todos os layouts do zero (sobrescreve os existentes)
> - **Sem limitação** — gerar tudo novamente sem considerar o que já existe

---

## ⚠️ PRIMEIRA AÇÃO — Obrigatória antes de qualquer outra coisa

**NÃO analise, NÃO gere código, NÃO liste padrões** até ter respondido estas perguntas
com o usuário. Esta é a primeira mensagem que deve ser enviada ao receber a skill:

```
Antes de começar, preciso de algumas informações:

1. Qual é o caminho da pasta com o layout base de referência?
   (ex: resources/js/layouts/, resources/js/Pages/Auth/, etc.)
   Você pode colar o código aqui ou informar o caminho no projeto.

2. Onde devo salvar os componentes gerados?
   (sugestão: resources/js/components/page/)

3. Onde devo salvar os composables?
   (sugestão: resources/js/composables/)
```

Só prossiga para a **Fase 1** após o usuário responder as três perguntas acima.
Se o usuário já tiver colado o código na mesma mensagem, ainda assim confirme os
itens 2 e 3 antes de gerar qualquer arquivo.

---

## Objetivo

Esta skill analisa modelos/layouts fornecidos pelo usuário e produz um conjunto coeso de
componentes Vue reutilizáveis para **páginas de listagem** e **páginas de formulário**,
eliminando repetição e padronizando a estrutura visual e funcional do projeto.

---

## Stack obrigatório

- Vue 3 + Composition API (`<script setup lang="ts">`)
- TypeScript estrito
- TailwindCSS (classes utilitárias, sem CSS customizado salvo exceções)
- Inertia.js (`useForm`, `router`, `usePage`) para navegação e forms

### Componentes de UI — adaptar ao projeto

**Não existe uma biblioteca de UI obrigatória.** Antes de gerar código, identifique o que o
projeto já usa:

1. **Se o modelo fornecido usa shadcn-vue** → continue usando shadcn-vue
2. **Se usa outra biblioteca** (PrimeVue, Quasar, Headless UI, etc.) → siga o padrão existente
3. **Se não usa nenhuma** → gere HTML semântico puro com Tailwind

Quando não houver componentes de UI disponíveis, use esses equivalentes HTML+Tailwind:

| Necessidade | HTML+Tailwind equivalente |
|---|---|
| Botão primário | `<button class="px-4 py-2 bg-primary text-white rounded-md text-sm font-medium hover:bg-primary/90">` |
| Botão outline | `<button class="px-4 py-2 border border-input rounded-md text-sm font-medium hover:bg-accent">` |
| Input | `<input class="w-full border border-input rounded-md px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-ring">` |
| Card | `<div class="rounded-lg border bg-card text-card-foreground shadow-sm p-6">` |
| Badge | `<span class="inline-flex items-center rounded-full px-2.5 py-0.5 text-xs font-medium">` |

---

## Fase 1 — Análise do Modelo

### 1a. Coletar informações do projeto (OBRIGATÓRIO — fazer antes de qualquer análise)

Antes de analisar qualquer arquivo, **faça estas perguntas ao usuário** se ainda não foram
respondidas na conversa:

1. **Onde está a pasta com o layout base?**
   - Ex: `resources/js/layouts/`, `resources/js/Pages/`, `resources/js/components/`
   - Se o usuário ainda não colou o código nem forneceu o caminho, peça explicitamente:
     *"Qual é o caminho da pasta com o layout de referência? Você pode colar o código aqui
     ou informar o caminho no projeto."*

2. **Onde os componentes gerados devem ser salvos?**
   - Se não informado, sugira `resources/js/components/page/` e confirme com o usuário

3. **Onde os composables devem ser salvos?**
   - Se não informado, sugira `resources/js/composables/` e confirme com o usuário

Só avance para o passo **1b** após ter o layout de referência disponível (colado no chat
ou lido do filesystem via ferramenta bash).

### 1b. Identificar o tipo de entrada

O usuário pode fornecer **duas fontes diferentes** — trate cada uma adequadamente:

**Entrada: código Vue/HTML colado**
- Leia o código diretamente
- Identifique classes CSS, estrutura de tags, diretivas Vue, padrões de slot
- Extraia nomes de variáveis, props e emits já usados para manter consistência

**Entrada: print/screenshot de página**
- Analise visualmente os elementos presentes
- Identifique regiões: header, filtros, tabela, paginação, sidebar, footer
- Anote o que é claramente interativo (botões, inputs, selects, checkboxes)
- Anote o que parece dinâmico (badges de contagem, estados de loading, paginação)
- Se a imagem for ambígua, pergunte antes de assumir

### 1c. Garimpar padrões presentes

### Padrões de Lista a garimpar:
- [ ] Cabeçalho com título, subtítulo e breadcrumb
- [ ] Área de ações do header (criar, importar, exportar, outros)
- [ ] Barra de busca / filtros inline
- [ ] Filtros avançados (drawer, modal ou painel colapsável)
- [ ] Chips de filtros ativos com botão de limpar
- [ ] Tabela ou grid de dados
- [ ] Estado vazio (empty state com CTA)
- [ ] Estado de carregamento (skeleton)
- [ ] Seleção de linhas e ações em massa (bulk actions)
- [ ] Paginação com meta do Laravel
- [ ] Toggle de visualização (tabela / cards / kanban)

### Padrões de Formulário a garimpar:
- [ ] Cabeçalho: título dinâmico (Novo X / Editando X), breadcrumb, status badge
- [ ] Ações do header (salvar, cancelar, excluir, duplicar)
- [ ] Layout em colunas (main + sidebar)
- [ ] Seções colapsáveis / tabs dentro do form
- [ ] Sidebar de metadados (datas, autor, tags, status)
- [ ] Footer sticky com ações de salvar/cancelar
- [ ] Dirty check (aviso de saída sem salvar)
- [ ] Exibição de erros de validação (Inertia errors)
- [ ] Estado de loading durante submit

### Padrões transversais:
- [ ] Confirmação de exclusão (dialog)
- [ ] Toast / notificação de feedback
- [ ] Guardas de permissão em botões/seções (`can` / `v-if="$page.props.auth.can[...]"`)
- [ ] Navegação entre registros (anterior/próximo)

**Liste os padrões encontrados antes de gerar código.** Pergunte ao usuário se o diagnóstico
está correto antes de prosseguir.

---

## Fase 2 — Arquitetura de Saída

Gere os arquivos na seguinte estrutura (adapte ao projeto do usuário):

```
resources/js/
├── components/
│   └── page/
│       ├── PageShell.vue          # Container base (padding, max-width, scroll)
│       ├── PageHeader.vue         # Título, breadcrumb, slot header-actions
│       ├── PageFilters.vue        # Gerencia filtros, URL sync, chips ativos
│       ├── PagePagination.vue     # Paginação com meta do Laravel paginate()
│       ├── PageEmptyState.vue     # Estado vazio configurável
│       ├── PageBulkActionBar.vue  # Barra de ações em massa
│       ├── ListPage.vue           # Sublayout completo de listagem
│       └── FormPage.vue           # Sublayout completo de formulário
└── composables/
    ├── usePageFilters.ts          # Lê/escreve query params, debounce, reset
    ├── useSelection.ts            # Seleção de linhas, select-all, bulk
    ├── useFormDirty.ts            # Detecta mudanças não salvas, aviso de saída
    └── usePageActions.ts          # Helpers: loading states, confirmação, toast
```

---

## Fase 3 — Geração dos Componentes

### Regras de geração

1. **Slots sobre props** para conteúdo variável — nunca force o conteúdo dentro do componente
2. **Props com defaults** — todos os campos opcionais devem ter `default: undefined` ou `false`
3. **Emits tipados** — declare todos os emits com tipos TypeScript
4. **Composables separados** — lógica stateful vai em composables, não dentro do componente
5. **Sem lógica de negócio** — os componentes são estrutura, não regra; a página-pai decide o comportamento
6. **Adaptar UI ao projeto** — use os componentes de UI que o modelo fornecido já usa; se não houver, use HTML+Tailwind puro (ver tabela no Stack)
7. **Manter nomes do modelo** — se o modelo usa `header-actions`, mantenha esse nome de slot; se usa `actions`, mantenha `actions`

### Contrato de slots do `<ListPage>`

```vue
<ListPage
  title="Produtos"
  :breadcrumbs="[{ label: 'Dashboard', href: '/' }, { label: 'Produtos' }]"
  :total="meta.total"
  :loading="isLoading"
>
  <!-- Ações do cabeçalho (criar, importar, exportar) -->
  <template #header-actions>
    <Button @click="router.visit(route('products.create'))">Novo Produto</Button>
  </template>

  <!-- Filtros — passa o componente de filtros customizado da página -->
  <template #filters>
    <ProductFilters v-model="filters" />
  </template>

  <!-- Ações em massa — aparece só com seleção ativa -->
  <template #bulk-actions="{ selected, clear }">
    <Button variant="destructive" @click="deleteSelected(selected, clear)">
      Excluir {{ selected.length }} itens
    </Button>
  </template>

  <!-- Corpo da listagem (tabela, grid, kanban — livre) -->
  <template #default>
    <ProductTable :items="products" v-model:selection="selection" />
  </template>

  <!-- Paginação — opcional, automático se :meta for passado -->
  <template #pagination>
    <PagePagination :meta="meta" @change="changePage" />
  </template>
</ListPage>
```

### Contrato de slots do `<FormPage>`

```vue
<FormPage
  :title="form.id ? `Editando ${form.name}` : 'Novo Produto'"
  :breadcrumbs="[{ label: 'Produtos', href: route('products.index') }, { label: form.id ? 'Editar' : 'Novo' }]"
  :loading="form.processing"
  :dirty="form.isDirty"
  @submit="submit"
  @cancel="router.visit(route('products.index'))"
>
  <!-- Ações extras no header (excluir, duplicar) -->
  <template #header-actions>
    <Button variant="destructive" @click="confirmDelete">Excluir</Button>
  </template>

  <!-- Corpo principal do formulário -->
  <template #default>
    <FormField label="Nome" :error="form.errors.name">
      <Input v-model="form.name" />
    </FormField>
  </template>

  <!-- Sidebar de metadados -->
  <template #sidebar>
    <Card>
      <CardContent>
        <p>Criado em: {{ product.created_at }}</p>
        <StatusSelect v-model="form.status" />
      </CardContent>
    </Card>
  </template>
</FormPage>
```

---

## Fase 4 — Composables

### `usePageFilters(defaults)`

```ts
// Sincroniza filtros com a URL (query params) via Inertia router
// Debounce configurável para busca de texto
// Reset volta aos defaults e limpa a URL
const { filters, setFilter, resetFilters, activeFilterCount } = usePageFilters({
  search: '',
  status: null,
  category_id: null,
})
```

### `useSelection(items, key?)`

```ts
// Controle de seleção de linhas para bulk actions
// key padrão: 'id'
const { selected, toggle, toggleAll, clear, isSelected, hasSelection } = useSelection(products)
```

### `useFormDirty(form)`

```ts
// Detecta se o form foi modificado (Inertia form.isDirty)
// Registra beforeunload para avisar o usuário
// Integra com onBeforeRouteLeave do Vue Router / Inertia
const { isDirty, confirmLeave } = useFormDirty(form)
```

### `usePageActions()`

```ts
// Helpers padronizados para ações comuns
const { confirm, toast, withLoading } = usePageActions()

// Exemplo de uso:
await confirm({ title: 'Excluir produto?', description: 'Essa ação não pode ser desfeita.' })
await withLoading(() => router.delete(route('products.destroy', product.id)))
toast.success('Produto excluído com sucesso.')
```

---

## Fase 5 — Documentação dos Componentes

Para cada componente gerado, produza um bloco de documentação em markdown com:

```markdown
## NomeDoComponente

**Localização:** `resources/js/components/page/NomeDoComponente.vue`

**Propósito:** [uma frase descrevendo o que resolve]

### Props
| Prop | Tipo | Obrigatório | Default | Descrição |
|------|------|-------------|---------|-----------|
| title | string | ✅ | — | Título principal |
| loading | boolean | ❌ | false | Exibe skeleton |

### Slots
| Slot | Escopo | Descrição |
|------|--------|-----------|
| default | — | Corpo da listagem |
| header-actions | — | Botões do cabeçalho |
| filters | — | Componente de filtros |
| bulk-actions | `{ selected, clear }` | Ações em massa |

### Emits
| Evento | Payload | Descrição |
|--------|---------|-----------|
| page-change | `{ page: number }` | Mudança de página |

### Exemplo mínimo de uso
\`\`\`vue
<ListPage title="Produtos" :total="meta.total">
  <template #header-actions>
    <button @click="criar">Novo</button>
  </template>
  <ProductTable :items="products" />
</ListPage>
\`\`\`
```

---

## Fase 6 — Checklist final antes de entregar

- [ ] Todos os componentes têm `<script setup lang="ts">`
- [ ] Props com `defineProps<{...}>()` tipadas
- [ ] Emits com `defineEmits<{...}>()` tipados
- [ ] Slots documentados com comentários inline no `.vue` E na documentação markdown
- [ ] Nenhuma lógica de negócio nos componentes de layout
- [ ] Estados de loading, vazio e erro cobertos
- [ ] Classes Tailwind sem valores arbitrários desnecessários
- [ ] Componentes de UI adaptados ao que o projeto usa (não forçar shadcn)
- [ ] Composables exportam apenas o necessário (sem expor estado interno)
- [ ] Documentação markdown gerada para cada componente e composable

---

## Referências adicionais

- Leia `references/list-patterns.md` para exemplos de variações de listagem
- Leia `references/form-patterns.md` para exemplos de variações de formulário