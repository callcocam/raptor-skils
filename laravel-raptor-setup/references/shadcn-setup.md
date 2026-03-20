# shadcn-vue Setup no Laravel + Inertia + Vue 3

## Instalação

```bash
./vendor/bin/sail npm install -D @shadcn-vue/cli
./vendor/bin/sail npx shadcn-vue@latest init
```

### Perguntas do CLI:
- **TypeScript?** → Sim (padrão do projeto)
- **Framework?** → Vite
- **Tailwind CSS config path** → `tailwind.config.js`
- **CSS variables?** → Sim
- **CSS file path** → `resources/css/app.css`
- **Components alias** → `@/components`
- **Utils alias** → `@/lib/utils`

---

## Adicionar componentes individualmente

```bash
./vendor/bin/sail npx shadcn-vue@latest add button
./vendor/bin/sail npx shadcn-vue@latest add input
./vendor/bin/sail npx shadcn-vue@latest add dialog
# etc.
```

---

## Estrutura esperada após init

```
resources/js/
├── components/
│   └── ui/          ← componentes shadcn gerados aqui
├── lib/
│   └── utils.ts     ← cn() utility
└── app.ts
```

---

## Configuração do vite.config.ts com alias

```typescript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './resources/js'),
    },
  },
})
```

---

## Integração com Tailwind

Confirmar que `tailwind.config.js` inclui:

```js
content: [
  './resources/**/*.blade.php',
  './resources/**/*.js',
  './resources/**/*.vue',
  './resources/**/*.ts',
],
```