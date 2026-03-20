# Inferência de Tipos TypeScript a partir do Mock

## Como mapear campos visuais → tipos TypeScript

| Campo visual no mock | Tipo TypeScript |
|---|---|
| Input texto | `string` |
| Input número | `number` |
| Input data | `string` (ISO 8601) |
| Input datetime | `string` (ISO 8601) |
| Textarea | `string` |
| Checkbox / toggle | `boolean` |
| Select simples | `string` |
| Select com opções fixas (status, etc.) | union type: `'ativo' \| 'inativo' \| 'pendente'` |
| Select de relacionamento | `number` (FK id) + objeto relacionado opcional |
| Upload de arquivo | `File \| null` (form) / `string` (model — path) |
| Badge de status | union type baseado nas variantes visuais do mock |
| Lista/array de itens | `ItemType[]` |
| Valor monetário | `number` (centavos) ou `string` (formatado) |

---

## Padrão de arquivo de tipos

Criar em `resources/js/types/{nome-model}.ts`:

```typescript
// Tipo principal do Model (dados vindos do backend)
export interface NomeModel {
  id: number
  // campos inferidos do mock
  nome: string
  status: 'ativo' | 'inativo'
  categoria_id: number
  categoria?: Categoria        // relacionamento (eager load)
  created_at: string
  updated_at: string
  deleted_at: string | null
}

// Tipo para o formulário (useForm do Inertia)
// Omite campos gerados pelo backend
export interface NomeModelForm {
  nome: string
  status: 'ativo' | 'inativo'
  categoria_id: number | ''   // '' para estado vazio do select
}

// Tipo para listagem com paginação
export interface NomeModelIndex {
  data: NomeModel[]
  links: PaginationLink[]
  meta: PaginationMeta
}

// Tipo para filtros da listagem
export interface NomeModelFilters {
  search?: string
  status?: string
  // outros filtros visíveis no mock
}
```

---

## Tipos globais compartilhados

Criar/atualizar em `resources/js/types/index.d.ts`:

```typescript
export interface PaginationLink {
  url: string | null
  label: string
  active: boolean
}

export interface PaginationMeta {
  current_page: number
  from: number
  last_page: number
  per_page: number
  to: number
  total: number
}

export interface PaginatedResponse<T> {
  data: T[]
  links: PaginationLink[]
  meta: PaginationMeta
}

// Flash messages do Laravel
export interface Flash {
  success?: string
  error?: string
  warning?: string
  info?: string
}
```

---

## Identificando relacionamentos no mock

**Sinais de relacionamento no mock:**
- Select com label de outra entidade (ex: "Selecione a categoria")
- Coluna na tabela mostrando nome de outra entidade (ex: "Loja: Shopping Center Norte")
- Badge ou chip com dados de outra entidade
- Seção aninhada com dados de outra tabela

**Para cada relacionamento identificado:**
1. Adicionar `{entidade}_id: number` no tipo form
2. Adicionar `{entidade}?: Entidade` no tipo model (opcional, para eager load)
3. Adicionar `{entidades}: Entidade[]` nas props da página Form (para popular o select)
4. Passar os dados de relacionamento pelo Controller:

```php
public function create(): Response
{
    return Inertia::render('NomeModel/Form', [
        'categorias' => Categoria::select('id', 'nome')->orderBy('nome')->get(),
    ]);
}
```

---

## Identificando enums / status no mock

Quando o mock mostra badges ou selects com opções fixas:

1. Listar todas as variantes visuais (cores diferentes de badge = valores diferentes)
2. Criar union type com os valores inferidos
3. Criar constante com labels para exibição:

```typescript
// No arquivo de tipos
export type LojaStatus = 'ativa' | 'inativa' | 'em_reforma'

// Em um arquivo de constantes (resources/js/constants/loja.ts)
export const LOJA_STATUS_LABELS: Record<LojaStatus, string> = {
  ativa: 'Ativa',
  inativa: 'Inativa',
  em_reforma: 'Em Reforma',
}

export const LOJA_STATUS_VARIANT: Record<LojaStatus, 'success' | 'danger' | 'warning'> = {
  ativa: 'success',
  inativa: 'danger',
  em_reforma: 'warning',
}
```