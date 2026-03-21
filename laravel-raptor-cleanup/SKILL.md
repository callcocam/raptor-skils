---
name: laravel-raptor-cleanup
description: >
  Use when auditar e corrigir qualidade de código em projetos Laravel Raptor Vue/TS:
  detecta componentes com nomes mistos PT-BR/inglês (convenção: inglês), arquivos obsoletos
  não importados, componentes duplicados/similares que podem compartilhar base ou composable,
  e tipos TypeScript inline ou duplicados para consolidar em resources/js/types/. Sempre
  cria branch separada antes de qualquer mudança. Mostra antes/depois de cada rename com
  opção de editar a sugestão.
---

# Laravel Raptor — Cleanup & Code Quality

## Visão Geral

Esta skill executa 5 fases de diagnóstico e correção de dívida técnica em projetos Vue/TS:

```
Passo 0: Branch Safety     → garantir branch de trabalho separada
Fase 1:  Naming Audit      → nomes mistos PT-BR/inglês → padronizar em inglês
Fase 2:  Obsolete Files    → arquivos não importados em lugar nenhum
Fase 3:  Duplicate/Similar → componentes quase idênticos → base/composable
Fase 4:  TypeScript Types  → tipos inline ou duplicados → resources/js/types/
Fase 5:  Apply Fixes       → aplicar correções aprovadas + relatório final
```

**Regra de ouro:** cada fase gera um relatório completo **antes** de tocar qualquer arquivo. O usuário aprova item a item.

---

## Passo 0 — Branch Safety

**PRIMEIRO PASSO — sem exceção. Nenhuma edição antes daqui.**

```bash
git branch --show-current
git status --short
git branch -a | grep -v "HEAD\|remotes/origin/HEAD" | head -20
```

### Branches protegidas (nunca editar diretamente)

`main` · `master` · `dev` · `develop` · `staging` · `production` · `release/*` · `hotfix/*`

### Decisão por situação

| Situação | Ação |
|----------|------|
| Em branch protegida | Bloquear — exigir nova branch antes de continuar |
| Em branch de trabalho existente (não protegida) | Perguntar: usar esta ou criar nova? |
| Repositório sem commits | Criar branch antes de qualquer mudança |

### Se em branch protegida, apresentar opções

> "Você está na branch `{branch}` que é protegida. Preciso criar uma branch de trabalho antes de qualquer mudança."
>
> - **`raptor/cleanup-{YYYY-MM-DD}`** ← criar esta (recomendado)
> - **Informar nome** — você digita o nome da branch
> - **Usar branch existente** — listar branches disponíveis para escolher

```bash
# Criar a branch
git checkout -b raptor/cleanup-$(date +%Y-%m-%d)
```

Só prosseguir para a Fase 1 após confirmação da branch de trabalho.

---

## Fase 1 — Naming Audit

### Convenção obrigatória

**Inglês puro, PascalCase.** Exemplos corretos: `SaveButton.vue`, `ConfirmModal.vue`, `ProductForm.vue`, `UserTable.vue`.

### O que detectar

Arquivos `.vue` e `.ts` em `resources/js/` com palavras em PT-BR no nome:

```bash
find resources/js -type f \( -name "*.vue" -o -name "*.ts" \) | \
  grep -iE "(Botao|Botoes|Campo|Campos|Lista|Listagem|Formulario|Pagina|\
Listagem|Novo|Nova|Editar|Excluir|Salvar|Busca|Filtro|Tabela|Linha|\
Cartao|Painel|Topo|Barra|Menu|Perfil|Senha|Usuario|Usuarios|\
Configuracao|Notificacao|Confirmacao|Cadastro|Relatorio|\
Importacao|Exportacao|Criacao|Edicao|Visualizacao|Detalhes|\
Produto|Produtos|Loja|Lojas|Pedido|Pedidos|Cliente|Clientes|\
Fornecedor|Categoria|Categorias|Estoque|Venda|Vendas)"
```

Também detectar:
- `snake_case.vue` ou `kebab-case.vue` (deve ser PascalCase)
- Abreviações em PT-BR: `Btn`, `Frm`, `Lst`, `Tbl`, `Mnu`

### Regra de sugestão de nome

1. Identificar o **tipo** do componente pelo sufixo: `Form`, `Modal`, `List`, `Table`, `Button`, `Page`, `Layout`, `Card`, `Badge`, `Panel`, `Sidebar`, `Header`, `Footer`
2. Identificar o **recurso/domínio**: `Product`, `User`, `Order`, `Category`, `Store`
3. Montar em inglês: `{Resource}{Type}.vue`

| PT-BR original | Sugestão EN |
|----------------|-------------|
| `FormularioProduto.vue` | `ProductForm.vue` |
| `ListagemLojas.vue` | `StoreList.vue` |
| `BotaoSalvar.vue` | `SaveButton.vue` |
| `ModalConfirmacao.vue` | `ConfirmModal.vue` |
| `TabelaUsuarios.vue` | `UserTable.vue` |
| `FiltrosBusca.vue` | `SearchFilters.vue` |
| `PaginaDetalhe.vue` | `DetailPage.vue` |

### Formato do relatório

```
=== FASE 1 — NOMES DE ARQUIVOS ===
X arquivos com nomenclatura inconsistente

📄 resources/js/components/ui/BotaoSalvar.vue
   Antes:  BotaoSalvar.vue
   Depois: SaveButton.vue  ← sugestão (editável)
   Opções: [ Aplicar ] [ Editar sugestão ] [ Pular ]

📄 resources/js/pages/Produtos/FormularioProduto.vue
   Antes:  FormularioProduto.vue
   Depois: ProductForm.vue  ← sugestão (editável)
   Opções: [ Aplicar ] [ Editar sugestão ] [ Pular ]
```

Aguardar decisão por arquivo. Ao aplicar um rename:
1. Renomear o arquivo fisicamente
2. Buscar e atualizar **todos os imports** que referenciam o nome antigo:
   ```bash
   grep -rn "BotaoSalvar" resources/js/ --include="*.vue" --include="*.ts"
   # Substituir cada ocorrência pelo novo nome
   ```

---

## Fase 2 — Arquivos Obsoletos

### O que detectar

Arquivos `.vue` e `.ts` que não são importados em lugar nenhum.

```bash
# Para cada .vue, contar quantas vezes é importado
find resources/js -name "*.vue" | while read file; do
  name=$(basename "$file" .vue)
  # Buscar import estático
  static=$(grep -rl "import.*['\"].*${name}['\"]" resources/js/ 2>/dev/null | grep -v "^${file}$" | wc -l)
  # Buscar uso como component em template
  template=$(grep -rl "<${name}[> /]" resources/js/ 2>/dev/null | grep -v "^${file}$" | wc -l)
  total=$((static + template))
  if [ "$total" -eq 0 ]; then
    mtime=$(stat -c "%y" "$file" | cut -d' ' -f1)
    echo "ZERO_IMPORTS | $mtime | $file"
  fi
done
```

**Também verificar imports em:**
- `resources/js/app.ts` — componentes globais
- `resources/views/app.blade.php` — Vite entrypoints

### ⚠️ Exceções: não marcar como obsoleto

- Arquivos em `resources/js/pages/` — Inertia os carrega pelo nome da rota, não por import estático
- Arquivos em `resources/js/types/` — são tipos, não componentes
- `index.ts` de barris de exportação
- Qualquer arquivo com `() => import(` em algum lugar (import dinâmico)

### Formato do relatório

```
=== FASE 2 — ARQUIVOS POSSIVELMENTE OBSOLETOS ===
⚠️  Atenção: imports dinâmicos não aparecem na busca estática.
    Arquivos em pages/ são carregados pelo Inertia e podem parecer não importados.

📄 resources/js/components/ui/OldCard.vue
   Última modificação: 45 dias atrás
   Referências encontradas: 0
   Opções: [ Ver conteúdo ] [ Excluir ] [ Manter ]

📄 resources/js/components/shared/DeprecatedTable.vue
   Última modificação: 120 dias atrás
   Referências encontradas: 0
   Opções: [ Ver conteúdo ] [ Excluir ] [ Manter ]
```

Nunca excluir sem confirmação explícita por arquivo.

---

## Fase 3 — Duplicatas e Componentes Similares

### O que detectar

**3a. Props quase idênticas entre componentes:**

```bash
# Extrair os defineProps de cada .vue para comparar
grep -rn "defineProps" resources/js/components --include="*.vue" -A 10 | head -100
```

Comparar componentes no mesmo domínio que tenham:
- Mesmo conjunto de props (ou subset de 80%+)
- Mesmo padrão de template (lista de itens, card com imagem+título+status, etc.)

**3b. Lógica duplicada em script setup:**

```bash
# Buscar padrões de código repetidos (fetch, paginação, filtros)
grep -rn "const.*ref\|watch\|onMounted\|router\." resources/js/pages --include="*.vue" | \
  awk -F: '{print $3}' | sort | uniq -d | head -20
```

**3c. Composables com funções de mesmo nome:**

```bash
grep -rn "^export function\|^export const\|^export async function" \
  resources/js/composables --include="*.ts" | \
  awk -F: '{print $3}' | sort | uniq -d
```

### Tipos de refatoração sugerida

| Situação | Solução sugerida |
|---------|-----------------|
| 2 componentes com mesmas props + estrutura similar | `Base{Name}.vue` com slots para variações |
| Lógica de fetch/loading repetida em N páginas | `useResource{Name}()` composable |
| Lógica de filtros + URL params duplicada | `useFilters()` composable (verificar se já existe) |
| Paginação implementada manualmente em vários lugares | `usePagination()` composable |
| Mesmas permissões verificadas em vários lugares | `usePermissions()` ou `can()` helper |

### Formato do relatório

```
=== FASE 3 — DUPLICATAS E SIMILARES ===

🔄 Componentes com props similares:

ProductCard.vue  ↔  ProductListItem.vue
   Props compartilhadas (5): id, name, status, price, image_url
   Props únicas em Card: description, featured
   Props únicas em ListItem: compact, showActions
   Sugestão: BaseProductItem.vue com slots `extra` e `actions`
   Opções: [ Ver diff ] [ Criar base ] [ Pular ]

🔄 Lógica duplicada:

pages/Products/Index.vue (linha 12-34)  ↔  pages/Categories/Index.vue (linha 8-29)
   Padrão: fetch + loading + error + pagination
   Sugestão: composables/useIndexPage.ts (verificar se já existe)
   Opções: [ Ver código ] [ Criar composable ] [ Pular ]
```

---

## Fase 4 — TypeScript Types

### O que detectar

**4a. Interfaces e types inline em `.vue`:**

```bash
grep -rn "^\(export \)\?\(interface\|type\) " resources/js \
  --include="*.vue" | grep -v "//\s"
```

**4b. Mesma interface declarada em 2+ arquivos:**

```bash
# Extrair nomes de interfaces de todos os arquivos e encontrar duplicatas
grep -rh "^\(export \)\?interface \|^\(export \)\?type " \
  resources/js --include="*.vue" --include="*.ts" | \
  grep -oP "(interface|type) \K\w+" | sort | uniq -d
```

**4c. `defineProps` sem TypeScript:**

```bash
grep -rn "defineProps(\[" resources/js --include="*.vue"
# defineProps(['name', 'email']) ← sem tipagem
```

### Destino dos tipos

Sempre `resources/js/types/{resource}.ts`. Se o arquivo não existir, criar:

```ts
// resources/js/types/products.ts

export interface Product {
  id: string
  name: string
  slug: string
  status: 'draft' | 'published'
  created_at: string
  updated_at: string
}

export interface ProductForm {
  name: string
  status: 'draft' | 'published'
}
```

**Após mover o tipo:** atualizar o `<script setup>` do arquivo de origem com o import correto:

```ts
// Antes (inline no .vue)
interface Product { ... }

// Depois
import type { Product } from '@/types/products'
```

### Formato do relatório

```
=== FASE 4 — TYPESCRIPT TYPES ===

📦 Tipos inline para extrair:

📄 resources/js/pages/Products/Index.vue (linhas 5–15)
   interface Product { id, name, status, price }
   → Mover para: resources/js/types/products.ts
   Opções: [ Mover ] [ Pular ]

📄 resources/js/components/UserCard.vue (linha 3)
   + resources/js/pages/Users/Index.vue (linha 8)
   DUPLICATA: interface User { id, name, email, role }
   → Consolidar em: resources/js/types/users.ts
   Opções: [ Consolidar ] [ Pular ]

⚠️  Props sem tipagem TypeScript:
📄 resources/js/components/Badge.vue
   defineProps(['label', 'color', 'size'])
   → defineProps<{ label: string; color?: string; size?: 'sm' | 'md' | 'lg' }>()
   Opções: [ Aplicar sugestão ] [ Editar ] [ Pular ]
```

---

## Fase 5 — Aplicar Correções

Aplicar na seguinte ordem (minimiza conflitos):

1. **Tipos** (Fase 4) — mover para `types/`, atualizar imports nas Vues
2. **Composables/bases** (Fase 3) — criar arquivos novos, refatorar consumidores
3. **Renomes** (Fase 1) — renomear arquivo + atualizar todos os imports
4. **Exclusões** (Fase 2) — remover arquivos obsoletos confirmados

### Antes de cada grupo de mudanças

```bash
git add -A && git stash
# ou
git add -A && git commit -m "chore: checkpoint before raptor-cleanup phase N"
```

### Relatório final

```
=== RESUMO — laravel-raptor-cleanup ===
Branch: raptor/cleanup-2026-03-21

✅ Fase 1 — Nomes corrigidos:     X arquivos renomeados
✅ Fase 2 — Obsoletos removidos:  Y arquivos excluídos
✅ Fase 3 — Refatorações:         Z bases/composables criados
✅ Fase 4 — Types consolidados:   W interfaces → resources/js/types/

Imports atualizados: N referências corrigidas no total

Próximos passos sugeridos:
- Rodar `npm run type-check` para verificar erros de TypeScript
- Rodar `npm run build` para confirmar que não quebrou o build
- Abrir PR da branch raptor/cleanup-{data} para revisão
```

---

## Regras Gerais

- ❌ Nunca editar arquivo antes de garantir branch de trabalho (Passo 0)
- ❌ Nunca excluir arquivo sem confirmação explícita
- ❌ Nunca renomear sem atualizar todos os imports que referenciam o nome antigo
- ❌ Nunca mover um tipo sem atualizar o import no arquivo de origem
- ✅ Sempre mostrar antes/depois antes de qualquer rename
- ✅ Sempre verificar se composable já existe antes de sugerir criar um novo
- ✅ Sempre mencionar o aviso sobre imports dinâmicos na Fase 2
- ✅ Arquivos em `pages/` → tratar com cautela na Fase 2 (Inertia os carrega por nome de rota)
- ✅ Ao final: sugerir `npm run type-check` e `npm run build`
