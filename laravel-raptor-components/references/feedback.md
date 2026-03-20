# Referência — Feedback

## Modal / Dialog

**Anatomia:**
```
┌─────────────────────────────────────┐
│ [icon] Título              [×close] │  ← header
├─────────────────────────────────────┤
│                                     │
│  Conteúdo (slot default)            │  ← body com scroll
│                                     │
├─────────────────────────────────────┤
│ [footer-left]      [cancel] [confirm]│  ← footer
└─────────────────────────────────────┘
```

**Props:**
- `modelValue` / `open`: boolean — controle externo
- `title`: string
- `description`: string — subtítulo no header
- `size`: `'sm' | 'md' | 'lg' | 'xl' | 'full'`
- `closable`: boolean — exibe botão X e fecha ao clicar fora (default true)
- `close-on-overlay`: boolean — fecha ao clicar no backdrop (default true)
- `close-on-escape`: boolean — fecha com Escape (default true)
- `loading`: boolean — estado de loading das ações

**Slots:** `default`, `header`, `footer`, `footer-left`, `icon`

**Emits:** `update:modelValue`, `close`, `confirm`

**Detalhe de qualidade:**
- Trap focus: Tab só circula entre elementos dentro do modal enquanto aberto
- `role="dialog"` + `aria-modal="true"` + `aria-labelledby` no título
- Scroll do body bloqueado quando modal aberto (`overflow: hidden` no `<html>`)
- Animação: backdrop fade-in, modal slide-up + fade-in
- Empilhamento: suporta múltiplos modais com z-index incremental
- Renderizado via `<Teleport to="body">`

**Composable `useModal`:**
```ts
const { open, close, isOpen } = useModal()
// Uso programático: sem precisar de v-model no template
```

---

## Toast / Notificação

**Props de cada toast:**
- `title`: string
- `description`: string
- `intent`: `'default' | 'success' | 'warning' | 'error' | 'info'`
- `duration`: number — ms até fechar automaticamente (default 4000, 0 = não fecha)
- `closable`: boolean (default true)
- `action`: `{ label: string, onClick: () => void }` — botão de ação no toast

**Posições possíveis:** `top-right`, `top-center`, `top-left`, `bottom-right`, `bottom-center`, `bottom-left`

**Composable `useToast`:**
```ts
const { toast } = useToast()

toast.success('Produto salvo com sucesso!')
toast.error('Erro ao salvar', { description: 'Verifique sua conexão.' })
toast.warning('Atenção', { duration: 0, action: { label: 'Desfazer', onClick: undo } })
toast({ title: 'Custom', intent: 'info', duration: 6000 })
```

**Detalhe de qualidade:**
- Animação: slide-in da borda + fade; ao remover, slide-out + collapse de altura
- Toasts se empilham verticalmente com gap, novos aparecem no topo (top-*) ou embaixo (bottom-*)
- Pause automático do timer quando mouse está sobre o toast
- Limite máximo de toasts simultâneos (ex: 5), o mais antigo some quando ultrapassar
- `role="alert"` para erros, `role="status"` para success/info

---

## Alert

**Props:**
- `intent`: `'info' | 'success' | 'warning' | 'error'`
- `title`: string
- `dismissible`: boolean — exibe botão de fechar
- `icon`: boolean — exibe ícone do intent (default true)

**Slots:** `default` (descrição), `icon` (override do ícone), `actions`

**Detalhe de qualidade:**
- Ícone padrão por intent: ℹ️ info, ✅ success, ⚠️ warning, ⛔ error
- Animação de dismiss: fade-out + collapse de altura
- `role="alert"` para warning/error, `role="status"` para info/success

---

## Badge

**Props:**
- `intent`: `'default' | 'primary' | 'success' | 'warning' | 'danger' | 'info'`
- `variant`: `'solid' | 'soft' | 'outline'`
- `size`: `'xs' | 'sm' | 'md'`
- `dot`: boolean — exibe ponto colorido antes do texto
- `removable`: boolean — exibe botão × para remover

**Emits:** `remove`

**Slots:** `default`, `prefix` (ícone antes do texto)

**Detalhe de qualidade:**
- Variant `soft`: cor de fundo com 10-15% de opacidade + texto na cor do intent
- Variant `outline`: borda colorida + texto colorido, fundo transparente
- Dot animado (pulse) para estado `live` ou `active`