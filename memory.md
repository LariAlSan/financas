# Soldo — Memória do Projeto

## Sobre o Projeto
App de gestão financeira pessoal com nome oficial **Soldo**, hospedado no **GitHub Pages** (`https://larialsan.github.io/financas/`).
Arquivo fonte: `C:\Persona\Financas\lari4.0.html` → publicado como `index.html` no repo `LariAlSan/financas`.
Sem servidor, sem dependências locais. Dados persistem via **IndexedDB** (database `lari4db`).

## Stack
- HTML5 + CSS3 + JavaScript Vanilla puro — arquivo único monolítico
- Chart.js 4.4.0 (CDN)
- PDF.js 3.11.174 (CDN)
- Google Identity Services (CDN) — OAuth2 Google Drive
- Google Fonts: Playfair Display (títulos) + Nunito (corpo)
- PWA: `manifest.json` + `sw.js` (cache `soldo-v2`) + ícones 192/512px

## Seções (13 no total)
Dashboard · Salário · Despesas Fixas · Parceladas · Investimentos · Variáveis · Combustível · Cartões · Orçamento · Relatório · Desejos · Notas · Calendário

## Módulos JavaScript
| Módulo | Responsabilidade |
|--------|-----------------|
| `ST` | Estado global |
| `STORAGE` | IndexedDB load/save, export/import, File System API sync |
| `GDRIVE` | Sync Google Drive bidirecional (OAuth2, token IDB); arquivo `soldo_sync.json` |
| `NOTIF` | Notificações de vencimento (Browser Notifications) |
| `DARKMODE` | Toggle claro/escuro; preferência em `localStorage('soldo-theme')` |
| `C` | Todos os cálculos financeiros |
| `R` | Render de cada seção + calendário + exportPDF |
| `CHART` | Gráficos Chart.js + relatório anual |
| `MODAL` | Formulários CRUD |
| `IMPORT` | Parser CSV/PDF (Nubank, C6) |

## Modelo de Dados (IndexedDB `lari4db`, store `data`, key `main`)
```json
{
  "meta": { "version": "4.0", "lastUpdated": "..." },
  "config": { "taxaCDI": 13.75, "brandName": "Soldo", "brandIcon": "R$", "theme": "rosa" },
  "salarios": { "YYYY-MM": { "base": 0, "ticket": 0, "extras": [] } },
  "despesasFixas": [{ "id","nome","valor","categoria","vencimento","pagamentos":{},"formaPagamento","cartaoId","historico":[] }],
  "parceladas": [{ "id","descricao","valorTotal","numParcelas","mesInicio","valorParcela","formaPagamento","cartaoId","categoria" }],
  "investimentos": [{ "id","nome","tipo","instituicao","indexador","percentual","autoCalc","dataAplicacao","dataVencimento","liquidez","valorInvestido","valorAtual" }],
  "despesasVariaveis": [{ "id","data","descricao","categoria","valor","formaPagamento","cartaoId" }],
  "combustivel": [{ "id","data","tipo","litros","valorTotal","kmRodados","posto","formaPagamento","cartaoId","variavelId" }],
  "cartoes": [{ "id","nome","bandeira","limite","diaFechamento","diaVencimento" }],
  "faturas": { "YYYY-MM": { "<cartaoId>": { "gastos":[], "paga":false } } },
  "orcamentos": { "<categoria>": 0 },
  "desejos": [{ "id","nome","custo","valorGuardado","prioridade","categoria","status","dataMeta","notas","investimentoId" }],
  "notas": { "YYYY-MM": "" },
  "receitasRecorrentes": [{ "id","descricao","valor" }],
  "metasEconomia": { "YYYY-MM": 0 },
  "contas": [{ "id","nome","saldo","atualizadoEm" }]
}
```

---

## Auth — Módulo de Autenticação

Adicionado em 2026-05-10. Overlay fixo (`#auth-screen`, z-index 99999) com dois fluxos.

| Item | Detalhe |
|------|---------|
| Módulo JS | `AUTH` (IIFE) inserido antes do `DOMContentLoaded` |
| Storage usuários | `soldo-auth` (localStorage) — array de contas |
| Storage sessão | `soldo-session` (localStorage se "lembrar de mim", sessionStorage se não) |
| Segurança de senha | SHA-256 + salt aleatório via Web Crypto API (SubtleCrypto) |
| Google OAuth | Reutiliza `GDRIVE_CLIENT_ID`; em sucesso popula `GDRIVE._tok` e `GDRIVE._tokExp` |
| Fluxo Login | Email + senha + lembrar-me + esqueci senha |
| Fluxo Cadastro | Nome → apelido (auto-sugerido) → email → senha (barra de força 4 níveis) → confirmar → avatar emoji → termos |
| Identidade visual | Gradiente dark rose `#1a0d16→#3d1f2e`, card branco, primário `#c9607e`, Nunito 800 |
| SVG background | Curvas de gráfico financeiro + pontos de dados + grid |
| Mobile | Bottom sheet (border-radius top-only) em < 480px |
| Dark mode | `[data-theme=dark] #auth-screen` CSS integrado |

---

## Histórico de Atualizações

### 2026-05-11 — Fix: bump SW cache soldo-v4 → soldo-v5
- Service Worker bumped para forçar invalidação de cache e entrega da versão atualizada aos clientes

### 2026-05-11 — Fix sidebar: marca preservada + nome dinâmico
- `sb-footer-name` ("Soldo") voltou a ser estático — `data-auth-name` movido para `sb-footer-phrase`
- Resultado: "Soldo" fixo no topo; nome do usuário logado aparece onde estava o slogan "financeiro mas poético"

### 2026-05-10 — Botão de logout na sidebar
- Botão `⏻` adicionado em `sb-icon-area` no rodapé da sidebar, cor `#c9607e` → chama `AUTH.logout()`
- Três botões independentes: 🌙 tema / ↻ atualizar app / ⏻ sair da conta
- GDRIVE tokens (IndexedDB store `sync`) preservados no logout; reconexão automática garantida

### 2026-05-10 — Favicon + meta tags iOS PWA
- `href="/financas/icon-192.png"` → `href="icon-192.png"` (path relativo — funciona local e no GitHub Pages)
- Adicionadas: `apple-touch-icon`, `apple-mobile-web-app-capable`, `apple-mobile-web-app-title` ("Soldo"), `apple-mobile-web-app-status-bar-style`

### 2026-05-10 — Tela de Auth (login/cadastro)
- Módulo `AUTH` (IIFE) adicionado — overlay `#auth-screen` cobre o app até autenticação
- Dois fluxos: **Entrar** (email/senha) e **Criar conta** (nome, apelido, email, senha, avatar, termos)
- Senhas: SHA-256 + salt via Web Crypto API sem dependências externas
- Sessão em `localStorage` (lembrar de mim ON) ou `sessionStorage` (lembrar de mim OFF)
- Google OAuth reusa `GDRIVE_CLIENT_ID` existente; cria perfil local automaticamente
- Apelido auto-sugerido (primeiro token do nome, lowercase); avatar emoji selecionável (20 opções)
- Barra de força de senha em tempo real (4 níveis), validação de match, shake animation em erro
- Dark mode e mobile bottom sheet integrados

### 2026-04-26 — v4.3.6 (fix: espelhamento do ícone PWA)
- **Ícone corrigido**: `icon-192.png` e `icon-512.png` estavam espelhados horizontalmente após o resize com Pillow. Aplicado `FLIP_LEFT_RIGHT` para restaurar orientação original (ponto no canto superior esquerdo).
- **SW cache**: `soldo-v1` → `soldo-v2` para forçar refresh dos ícones nos dispositivos.

### 2026-04-26 — v4.3.5 (rename oficial: Lari → Soldo)
- **Nome do app**: "Lari" renomeado para "Soldo" em toda a interface — título, manifest, brand sidebar, welcome, mensagens de erro, config padrão.
- **manifest.json**: `name/short_name` → "Soldo — Gestão Financeira" / "Soldo".
- **localStorage**: `lari-theme` → `soldo-theme`; `lari-onboarding` → `soldo-onboarding`.
- **Drive sync**: arquivo renomeado `lari4_sync.json` → `soldo_sync.json` (re-vincular Drive uma vez).
- **Export filenames**: `lari4_backup_*` → `soldo_backup_*`; `lari4_*.csv` → `soldo_*.csv`.
- **SW cache**: `lari-v11` → `soldo-v1`.
- **Preservado**: `lari4db` (IndexedDB) e `lari4_data` (store key) — sem alteração para não perder dados.
- Aplicado em: `lari4.0.html`, `Jamilly.html`, `LYD.html`, `manifest.json`, `sw.js`.

### 2026-04-26 — v4.3.4 (Relatório: 3 donuts lado a lado)
- **Layout Relatório**: grid 2→3 colunas via `.charts-grid.g3`; `ch-inv` removido de `.cc full` — os 3 donuts (Fixas, Variáveis, Portfólio) ficam na mesma linha. `ch-6m` permanece abaixo como full-width. Mobile (≤768px) continua empilhado em 1 coluna.

### 2026-04-26 — v4.3.3 (gráficos Investimentos menores)
- **Canvas height**: `ch-inv-idx` e `ch-inv-liq` reduzidos de `height="160"` → `height="110"` — donuts da seção Investimentos ficaram mais compactos.

### 2026-04-26 — v4.3.2 (novo ícone PWA — gradiente rosa/roxo)
- **Ícone redesenhado**: gradiente rosa→roxo, cantos arredondados, fundo rosé. Gerado via Pillow (512×512 e 192×192).
- **manifest.json**: `purpose` alterado de `"any maskable"` para `"any"` — imagem já tem cantos próprios, máscara do SO recortaria as bordas.
- **SW cache**: `lari-v10` → `lari-v11`.

### 2026-04-24 — v4.1.8 (UX mobile: toast visível + gradiente na nav)
- **Toast acima da barra**: `#toast` sobrescrito no media query mobile para `bottom:calc(58px + 12px)` — antes ficava escondido atrás da barra de navegação de 58px.
- **Gradiente na nav**: removido `overflow:hidden` do `#sidebar` mobile; adicionado `#sidebar::after` com `linear-gradient(transparent → var(--sidebar-bg))` de 40px na borda direita.
- Aplicado em: `lari4.0.html`, `Jamilly.html`, `LYD.html`.

### 2026-04-24 — v4.1.7 (fixes mobile: barra fixa + zoom inputs)
- **Barra de módulos fixa**: `position:fixed` explícito na regra `#sidebar` do media query 768px.
- **Zoom em inputs**: `font-size` dos campos alterado para `16px` — iOS Safari faz auto-zoom em inputs com `font-size < 16px`.
- **Viewport**: adicionado `viewport-fit=cover` — melhora comportamento de `position:fixed` em PWA no iOS.
- Aplicado em: `lari4.0.html`, `Jamilly.html`, `LYD.html`.

### 2026-04-24 — v4.1.6 (versão do app na sidebar)
- **APP_VER**: constante no topo do JS. O elemento `#sb-upd` exibe versão + data do último save. Basta atualizar `APP_VER` a cada deploy.
- Aplicado em: `lari4.0.html`, `Jamilly.html`, `LYD.html`.

### 2026-04-24 — v4.1.5 (sidebar: botão ↻ limpar cache + grupo 💾 Arquivos)
- **Botão ↻ Atualizar app**: limpa SW cache via `caches.keys()` + `caches.delete()`, força update e recarrega. Função global `clearCache()`.
- **Grupo 💾 Arquivos**: rótulo `.sb-group-label` antes dos botões de export/import na sidebar.
- Aplicado em: `lari4.0.html`, `Jamilly.html`, `LYD.html`.

### 2026-04-24 — v4.1.3 (fix: card Vale Refeição + saldo acumulado automático)
- **Card Vale Refeição**: padronizado — exibe total gasto + contagem. Removidas sub-métricas e borda laranja.
- **Saldo acumulado automático**: `C.saldoAcumulado(mo)` acumula desde o mês mais antigo em `ST.data.salarios`. Função `setSaldoInicial` removida.
- Aplicado em: `lari4.0.html`, `Jamilly.html`, `LYD.html`.

### 2026-04-24 — v4.1.1 (fix: card Saldo em Contas responsivo)
- **Layout flex-wrap**: card usa `grid-column:1/-1` + `.contas-list{flex-wrap:wrap}` com `.conta-item{flex:1;min-width:200px}`.
- Aplicado em: `lari4.0.html`, `Jamilly.html`, `LYD.html`.

### 2026-04-24 — v4.1 (saldo acumulado + saldo em contas + alertas)
- **Saldo acumulado (carry-over)**: `C.saldoAcumulado(mo)` automático, sem input manual.
- **Saldo em Contas**: card no Dashboard com CRUD de contas bancárias (`ST.data.contas[]`). Função global `delConta(id)`.
- **Alertas/insights**: relocados para o final do Dashboard.
- **SW cache**: `lari-v6`.
- Aplicado em: `lari4.0.html`, `Jamilly.html`, `LYD.html`.

### 2026-04-23 — v4.0 (5 novas features + Desejos↔Investimentos)
- **Dark mode**: toggle ☀️/🌙; CSS via `[data-theme="dark"]`; preferência em `localStorage`.
- **Insights/Tendências**: variação por categoria vs média 3 meses, saldo negativo consecutivo, comparação ano anterior.
- **Meta de Economia**: campo mensal no Dashboard com barra de progresso; persiste em `metasEconomia[YYYY-MM]`.
- **Calendário de Gastos**: seção `s-calendario` — grade mensal com intensidade de gasto, detalhe ao clicar.
- **Exportar PDF**: popup imprimível via `window.print()`.
- **Desejos ↔ Investimentos**: campo `investimentoId` em desejos; `C.valorDesejo(d)` usa `C.valorCalc(inv)` se vinculado.

### 2026-04-23 — v4.0.1 (redesign ícone PWA)
- **Ícone rosa**: gradiente `#b05480→#7a2d55`, "R$" branco Georgia Bold; gerado com Pillow.
- **SW cache**: `lari-v3`.

### 2026-04-23 — v3.4 (PWA + sync + notificações + relatório anual)
- **PWA**: `manifest.json`, `sw.js`, ícones 192/512px.
- **Conflito sync Drive**: `confirm()` antes de sobrescrever dados locais.
- **Indicador offline**: barra vermelha fixa no topo.
- **Notificações de vencimento**: módulo `NOTIF` — fixas não pagas vencendo em ≤3 dias.
- **Semáforo de orçamentos**: progress bars por categoria (verde/amarelo/vermelho).
- **Relatório anual**: toggle Mensal/Anual; `CHART.renderAnual()`.

### 2026-04-23 — v3.2 (Google Drive sync)
- Módulo `GDRIVE` com OAuth2 via Google Identity Services.
- Client ID: `644762904112-65kfgliosm28n8j1m9862ejpn7j1lttn.apps.googleusercontent.com`
- Auto-save (5s debounce) + auto-load ao abrir e ao `visibilitychange`.
- Token salvo em IDB store `sync`, key `gdrive`: `{email, fileId}`.
- Arquivo no Drive: `soldo_sync.json`.

### 2026-04-23 — v3.1 (GitHub Pages)
- App publicado em `https://larialsan.github.io/financas/`; deploy via git push.

### 2026-04-22 — v2.4 (LYD.html + Jamilly.html)
- Apps derivados para outros usuários com paletas e dados independentes.

### 2026-04-22 — v1.0–v2.3
- Criação inicial, módulos Salário/Fixas/Parceladas/Variáveis/Investimentos/Combustível/Cartões/Orçamento/Desejos/Notas, importador CSV/PDF, responsividade mobile.

---

## Credenciais e configurações
- **GitHub Token**: salvo localmente (não versionado)
- **Google Drive Client ID**: `644762904112-65kfgliosm28n8j1m9862ejpn7j1lttn.apps.googleusercontent.com`
- **URL pública**: `https://larialsan.github.io/financas/`
- **Usuário GitHub**: `LariAlSan`

---
*Atualizado em 2026-04-27*
