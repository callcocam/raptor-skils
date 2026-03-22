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

## Fase 0 — Verificar estado anterior

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
> - **Atualizar** — ler o novo tema e apenas complementar/corrigir os layouts existentes
> - **Recriar** — gerar todos os componentes do zero (sobrescreve os existentes)
> - **Sem limitação** — gerar tudo novamente sem considerar o que já existe

---

## ⚠️ PRIMEIRA AÇÃO — Obrigatória antes de qualquer outra coisa

**NÃO analise, NÃO gere código, NÃO liste padrões** até ter todas as respostas abaixo.
Esta é a **primeira mensagem** que deve ser enviada ao receber a skill:

```
Antes de começar, preciso de algumas informações:

1. Qual é a base de tema do projeto? (obrigatório — escolha UMA opção)
   a) Caminho para a pasta do projeto
      (ex: resources/js/, resources/, .)
      Vou ler o app.css para encontrar os tokens de tema e listar os
      componentes de UI disponíveis.
   b) Cole aqui um documento com as instruções de tema
      (tokens de cor, tipografia, espaçamentos, convenções visuais)

2. Onde devo salvar os componentes gerados?
   (sugestão: resources/js/components/page/)

3. Onde devo salvar os composables?
   (sugestão: resources/js/composables/)

4. (Opcional) Você tem algum código Vue/HTML ou screenshot de referência
   visual para guiar o estilo dos componentes? Cole o código ou informe
   o caminho da imagem.
```

**Regras:**
- Perguntas 1, 2 e 3 são obrigatórias. Não prossiga sem elas.
- Se o usuário já forneceu a base de tema na mensagem de invocação (caminho de pasta
  ou documento colado), trate como resposta da pergunta 1 automaticamente.
- Pergunta 4 é opcional. Se o usuário já colou código ou screenshot na mesma
  mensagem, trate como resposta da pergunta 4 automaticamente.
- Se o usuário responder 2 e 3 mas pular a 1, peça especificamente a base de
  tema novamente — ela é a fundação de tudo.
- **NÃO confunda** o código da pergunta 4 (referência visual) com a pasta/documento
  da pergunta 1 (tema base). São entradas distintas.

---

## Fase 1 — Ler o Ambiente

Execute todas as sub-etapas antes de passar para a Fase 2.

### 1a — Detectar estrutura do projeto

```bash
# Layouts existentes
ls resources/js/Layouts/ 2>/dev/null

# Componentes de UI disponíveis para reuso
ls resources/js/components/ui/ 2>/dev/null
ls resources/js/Components/ 2>/dev/null
ls resources/js/components/shared/ 2>/dev/null

# CSS global com tokens de tema
ls resources/css/app.css resources/css/app.pcss 2>/dev/null
```

**Registrar:**
- Quais subpastas de componentes existem
- Quais arquivos CSS globais existem
- **NÃO assumir** nomes Breeze (PrimaryButton, TextInput, etc.) — registrar apenas o que for encontrado

### 1b — Ler a base de tema

**APENAS se a resposta da pergunta 1 foi uma PASTA:**

```bash
# Ler tokens de tema no CSS global — tentar app.css, fallback para app.pcss
cat resources/css/app.css 2>/dev/null || cat resources/css/app.pcss 2>/dev/null
```

Buscar no CSS:
- `@theme { }` — tokens Tailwind v4 (ex: `--color-primary: oklch(...)`)
- `:root { }` — CSS custom properties (ex: `--primary: 222 47% 11%`)
- `@layer base { }` — overrides de base

**Se nenhum arquivo CSS for encontrado** (`app.css` e `app.pcss` ausentes): acionar o fallback de tokens ausentes (ver abaixo).

**Se `components/ui/` existir:** para cada `.vue` encontrado, ler o arquivo para identificar nome e props aceitas, depois classificar por papel funcional usando a tabela de mapeamento (seção "Mapeamento de componentes UI").

**Se `components/ui/` não existir ou estiver vazio:** registrar "Sem componentes UI encontrados — usar fallback HTML+Tailwind" e prosseguir.

**Se o CSS lido não contiver `@theme {}` nem `:root {}`:** perguntar ao usuário:
> "Não encontrei tokens de tema no CSS. Quer colar um documento com os tokens
> ou devo usar classes Tailwind utilitárias padrão (ex: `bg-blue-600`, `text-gray-*`)?"
> - Se colar tokens → processar como documento (abaixo)
> - Se pedir padrão → usar classes utilitárias Tailwind sem `[]`

**APENAS se a resposta da pergunta 1 foi um DOCUMENTO:**

Extrair do documento:
- Tokens de cor nomeados e seus valores (hex, CSS var ou classe Tailwind)
- Tipografia: famílias de fonte, escala de tamanhos, pesos
- Espaçamentos e convenções de padding/gap
- Convenções visuais: border-radius, shadow, estilo de card, estilo de input
- Quaisquer regras documentadas de componentes

### 1c — Analisar referência visual (opcional)

**Só executar se o usuário forneceu código ou screenshot na pergunta 4.**

**Se código Vue/HTML:**
- Ler o código diretamente
- Identificar classes CSS, diretivas Vue, nomes de componentes, padrões de slot
- Extrair nomes de variáveis/props/emits já em uso (preservar consistência de nomenclatura)
- Verificar quais componentes de UI já estão sendo usados no código

**Se screenshot:**
- Identificar regiões: header, filtros, tabela/grid, paginação, sidebar, footer
- Anotar elementos interativos: botões, inputs, selects, checkboxes
- Anotar elementos dinâmicos: badges de contagem, estados de loading, empty state
- Se regiões forem ambíguas, perguntar antes de assumir

### 1d — Consolidar descobertas

Compilar internamente (não exibir ainda — isso acontece na Fase 2):

- **Tokens identificados:** lista de tokens de cor, CSS vars ou classes Tailwind do tema
- **Componentes UI disponíveis:** lista de componentes mapeados por papel funcional
- **Convenções visuais:** estilo de border-radius, padrão de card, estilo de input
- **Padrões da referência (se fornecida):** padrões de lista/form encontrados

---

## Mapeamento de componentes UI

Quando componentes são encontrados em `components/ui/`, mapear para papéis funcionais:

| Papel | Como identificar |
|-------|-----------------|
| Botão primário | Nome contendo "Button" ou "Btn" com `variant="default"` ou `variant="primary"`. Se houver ambiguidade entre múltiplos arquivos de botão, classificar o primeiro alfabeticamente como primário e registrar a ambiguidade em 1d. |
| Botão secundário | Mesmo componente com `variant="secondary"` ou `variant="outline"` |
| Input de texto | Nome contendo "Input" com prop `modelValue` |
| Label de campo | Nome contendo "Label" |
| Erro de campo | Nome contendo "Error" ou "Message" |
| Dialog/Confirm | Nome contendo "Dialog", "Modal" ou "Alert" |
| Toast/Notificação | Nome contendo "Toast", "Notification" ou "Sonner" |

Se um papel não for mapeável com confiança → usar HTML+Tailwind puro para aquele papel.

**Equivalentes HTML+Tailwind (quando não há componente mapeável):**

| Papel | Fallback |
|-------|---------|
| Botão primário | `<button class="px-4 py-2 bg-primary text-primary-foreground rounded-md text-sm font-medium hover:bg-primary/90 disabled:opacity-50">` |
| Botão secundário | `<button class="px-4 py-2 border border-input rounded-md text-sm font-medium hover:bg-accent disabled:opacity-50">` |
| Input | `<input class="w-full border border-input rounded-md px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-ring">` |
| Label | `<label class="text-sm font-medium text-foreground">` |
| Erro | `<p class="text-sm text-destructive mt-1">` |
| Dialog/Confirm | `window.confirm('mensagem')` — retorna boolean |
| Toast | Elemento `<div>` fixo no canto inferior direito, removido após 3s |

---

## Fase 2 — Diagnóstico e Confirmação

Após concluir a Fase 1, apresentar ao usuário o diagnóstico abaixo e **aguardar confirmação** antes de gerar qualquer arquivo.

```
## Diagnóstico do Ambiente

### Tokens de Tema Identificados
[Listar tokens extraídos, ex:]
- Cor primária: `--color-primary` / `bg-primary`
- Cor de fundo: `--color-background` / `bg-background`
- Borda: `--color-border`
- Destructive: `--color-destructive`
- Radius padrão: `rounded-md`

### Componentes de UI Disponíveis para Reuso
[Listar componentes encontrados com ✅ ou ❌:]
- ✅ Button.vue → papel: botão primário (variant="default") e secundário (variant="outline")
- ✅ Input.vue → papel: input de texto
- ❌ Dialog → não encontrado, usará `window.confirm()` como fallback
- ❌ Toast → não encontrado, usará implementação inline

### O Que Será Gerado
- ListPage: header + filtros + tabela + paginação + bulk actions + empty state
- FormPage: header + conteúdo principal + sidebar + footer sticky nativo (Salvar/Cancelar)
- 6 componentes de suporte: PageShell, PageHeader, PageFilters, PagePagination,
  PageEmptyState, PageBulkActionBar
- 4 composables: usePageFilters, useSelection, useFormDirty, usePageActions
- Documentação markdown em `<caminho-componentes>/docs/`

### Como os Tokens Serão Aplicados
- Botão primário → [componente ou HTML+Tailwind] com classes `bg-primary text-primary-foreground`
- Input → [componente ou HTML+Tailwind]
- Card/container → `rounded-lg border bg-card`
- Footer → `bg-white border-t` (ou equivalente com tokens do tema)

O diagnóstico está correto? Posso prosseguir com a geração?
```

**Aguardar confirmação do usuário.** Se corrigir algo (token errado, componente diferente), atualizar o diagnóstico interno e reconfirmar.

---

## Fase 3 — Arquitetura de Saída

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