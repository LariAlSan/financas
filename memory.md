# lari4.0 — Memória do Projeto

## Sobre o Projeto
Gestor financeiro pessoal hospedado no **GitHub Pages** (`https://larialsan.github.io/financas/`).
Arquivo local: `C:\Persona\Financas\lari4.0.html` → publicado como `index.html` no repo `LariAlSan/financas`.
Sem servidor, sem dependências locais. Dados persistem via **IndexedDB** (database `lari4db`).

## Stack
- HTML5 + CSS3 + JavaScript Vanilla puro — arquivo único monolítico
- Chart.js 4.4.0 (CDN)
- PDF.js 3.11.174 (CDN)
- Google Identity Services (CDN) — OAuth2 Google Drive
- Google Fonts: Playfair Display (títulos) + Nunito (corpo)
- PWA: manifest.json + sw.js + ícones 192/512px

## Seções (13 no total)
Dashboard · Salário · Despesas Fixas · Parceladas · Investimentos · Variáveis · Combustível · Cartões · Orçamento · Relatório · Desejos · Notas · **Calendário** (novo)

## Módulos JavaScript
| Módulo | Responsabilidade |
|--------|-----------------|
| `ST` | Estado global |
| `STORAGE` | IndexedDB load/save, export/import, File System API sync |
| `GDRIVE` | Sync Google Drive bidirecional (OAuth2, token IDB) |
| `NOTIF` | Notificações de vencimento (Browser Notifications) |
| `DARKMODE` | Toggle claro/escuro (preferência em localStorage) |
| `C` | Todos os cálculos financeiros |
| `R` | Render de cada seção + calendário + exportPDF |
| `CHART` | Gráficos Chart.js + relatório anual |
| `MODAL` | Formulários CRUD |
| `IMPORT` | Parser CSV/PDF (Nubank, C6) |

## Modelo de Dados (IndexedDB `lari4db`, store `data`, key `main`)
```json
{
  "meta": { "version": "4.0", "lastUpdated": "..." },
  "config": { "taxaCDI": 13.75 },
  "salarios": { "YYYY-MM": { "base": 0, "ticket": 0, "extras": [] } },
  "despesasFixas": [{ "id","nome","valor","categoria","vencimento","pagamentos":{},"formaPagamento","cartaoId","historico":[] }],
  "parceladas": [{ "id","descricao","valorTotal","numParcelas","mesInicio","valorParcela","formaPagamento","cartaoId","categoria" }],
  "investimentos": [{ "id","nome","tipo","instituicao","indexador","percentual","autoCalc","dataAplicacao","dataVencimento","liquidez","valorInvestido","valorAtual" }],
  "despesasVariaveis": [{ "id","data","descricao","categoria","valor","formaPagamento","cartaoId" }],
  "combustivel": [{ "id","data","tipo","litros","valorTotal","kmRodados","posto","formaPagamento","cartaoId","variavelId" }],
  "cartoes": [{ "id","nome","bandeira","limite","diaFechamento","diaVencimento" }],
  "faturas": { "YYYY-MM": { "<cartaoId>": { "gastos":[], "paga":false } } },
  "orcamentos": { "<categoria>": 0 },
  "desejos": [{ "id","nome","custo","valorGuardado","prioridade","categoria","status","dataMeta","notas" }],
  "notas": { "YYYY-MM": "" },
  "receitasRecorrentes": [{ "id","descricao","valor" }],
  "metasEconomia": { "YYYY-MM": 0 }
}
```

---

## Histórico de Atualizações

### 2026-04-24 — v4.1.7 (fixes mobile: barra fixa + zoom inputs)
- **Barra de módulos fixa**: adicionado `position:fixed` explicitamente na regra `#sidebar` do media query `@media(max-width:768px)`. Antes era herdado do desktop mas iOS Safari podia perder o comportamento fixo durante transições de viewport (barra de URL).
- **Zoom em inputs**: `font-size` dos campos de formulário em mobile alterado de `1rem` (= 14px com base 14px) para `16px`. iOS Safari faz auto-zoom em qualquer `<input>/<select>` com `font-size < 16px`; ao recarregar a página o zoom persistia.
- **Viewport**: adicionado `viewport-fit=cover` na meta viewport — melhora comportamento de `position:fixed` em PWA instalado no iOS e suporte a notch.
- Aplicado em: `lari4.0.html`, `index.html`, `Jamilly.html`, `LYD.html`.

### 2026-04-24 — v4.1.6 (versão do app na sidebar)
- **Versão do app**: constante `APP_VER='4.1.5'` adicionada no topo do JS. O elemento `#sb-upd` na sidebar exibe `v4.1.5 · 24/04/2026 16:33` (versão + data do último save de dados). Sem dados salvos, exibe só `v4.1.5`. Para futuros deploys, basta atualizar `APP_VER`.
- Aplicado em: `lari4.0.html`, `Jamilly.html`, `LYD.html`.

### 2026-04-24 — v4.1.5 (sidebar: botão ↻ limpar cache + grupo 💾 Arquivos)
- **Botão ↻ Atualizar app**: aparece discreto ao lado do toggle de tema (🌙) na sidebar. Limpa todo o SW cache via `caches.keys()` + `caches.delete()`, força update do service worker e recarrega o app — sem precisar bumpar versão manualmente. Função global `clearCache()`. Nos arquivos sem dark mode (Jamilly, LYD), o botão fica sozinho no canto da brand.
- **Grupo 💾 Arquivos**: rótulo `.sb-group-label` adicionado antes dos 4 botões de export/import na sidebar (Exportar JSON, Exportar CSV, Importar JSON, Importar Extrato), separando-os visualmente dos botões de sync.
- Aplicado em: `lari4.0.html`, `Jamilly.html`, `LYD.html`.

### 2026-04-24 — v4.1.3 (fix: card Vale Refeição padronizado + saldo acumulado automático)
- **Card Vale Refeição (Variáveis)**: padronizado com os cards "Pagamento Direto" e "Via Cartão" — exibe apenas o total gasto com ticket + contagem de lançamentos. Removidas as sub-métricas Recebido/Usado/Saldo e a borda laranja.
- **Saldo acumulado 100% automático**: removido campo "Saldo inicial" manual do dashboard. `C.saldoAcumulado(mo)` agora acumula automaticamente desde o mês mais antigo em `ST.data.salarios`; função `setSaldoInicial` removida.
- Aplicado em: `lari4.0.html`, `Jamilly.html`, `LYD.html`.

### 2026-04-24 — v4.1.1 (fix: card Saldo em Contas responsivo)
- **Layout flex-wrap**: card "Saldo em Contas" agora usa `grid-column:1/-1` (largura total) + `.contas-list{flex-wrap:wrap}` com `.conta-item{flex:1;min-width:200px}` — no desktop 3–4 contas por linha, no mobile empilha automaticamente sem overflow.
- CSS classes adicionadas: `.sum-card.contas`, `.contas-list`, `.conta-item` (substituem inline styles anteriores).
- Aplicado em: `lari4.0.html`, `Jamilly.html`, `LYD.html`.

### 2026-04-24 — v4.1 (saldo acumulado + saldo em contas + alertas reposicionados)
- **Saldo acumulado (carry-over)**: `C.saldoAcumulado(mo)` soma o saldo desde o mês mais antigo com salário cadastrado até o mês atual (automático, sem input manual).
- **Saldo em Contas**: card no Dashboard com CRUD de contas bancárias (`ST.data.contas[]` — `{id, nome, saldo, atualizadoEm}`). Botão "+ Adicionar" abre modal; ✏️/🗑️ por conta; total automático. Função global `delConta(id)`.
- **Alertas/insights movidos**: painéis `dash-alerts` e `dash-insights` relocados do topo do Dashboard para o final (após `dash-orcamentos`), mantendo os cards de resumo em destaque.
- **Service worker v6**: `CACHE = 'lari-v6'`; invalida cache anterior para forçar atualização nos navegadores.
- Aplicado em: `lari4.0.html`, `Jamilly.html`, `LYD.html`.

### 2026-04-23 — v4.0 (5 novas features + ícone)
- **Dark mode**: toggle ☀️/🌙 na sidebar; CSS via `[data-theme="dark"]`; preferência em `localStorage('lari-theme')`
- **Insights/Tendências**: análise automática no Dashboard — variação por categoria vs média 3 meses (>30% ou <-25%), saldo negativo consecutivo, comparação ano anterior
- **Meta de Economia**: campo mensal no Dashboard com barra de progresso; persiste em `metasEconomia[YYYY-MM]`
- **Calendário de Gastos**: nova seção `s-calendario` — grade mensal com cores por intensidade de gasto, detalhe ao clicar no dia; inclui variáveis, faturas de cartão e combustível
- **Exportar PDF**: botão na seção Relatório — abre popup com layout imprimível, aciona `window.print()`
- **Ícone melhorado**: "R$" em dourado com Georgia Bold, fundo marrom arredondado, anel decorativo, sombra

### 2026-04-23 — v4.0.2 (fix: combustível mobile + melhorias LYD/Jamilly)
- **Combustível responsivo (lari4.0)**: tabela de 10 colunas agora colapsa com `tbl-comb`; oculta colunas não essenciais via `nth-child` em 768px (6 colunas) e 480px (4 colunas: Data, Preço/L, Total, Ações); `.card{overflow-x:auto}` garante scroll visível
- **LYD.html responsivo**: adicionado `.tbl,.comp-table{display:block;overflow-x:auto}`, `.card{overflow-x:auto}`, touch targets (44px), `.cart-grid{grid-template-columns:1fr}` em 480px
- **Jamilly.html responsivo**: mesmas melhorias mobile que LYD

### 2026-04-23 — v4.0.1 (redesign ícone PWA)
- **Ícone rosa vibrante**: gradiente `#b05480→#7a2d55`, "R$" branco Georgia Bold 54%, cantos arredondados 18%, sombra sutil; gerado com Pillow Python para 192px e 512px
- **Service worker v3**: `CACHE = 'lari-v3'`; novos ícones pré-cacheados — invalida cache anterior

### 2026-04-23 — v4.0 (5 novas features + Desejos↔Investimentos)
- **Dark mode**: toggle ☀️/🌙 na sidebar; CSS via `[data-theme="dark"]`; preferência em `localStorage('lari-theme')`
- **Insights/Tendências**: análise automática no Dashboard — variação por categoria vs média 3 meses (>30% ou <-25%), saldo negativo consecutivo, comparação ano anterior
- **Meta de Economia**: campo mensal no Dashboard com barra de progresso; persiste em `metasEconomia[YYYY-MM]`
- **Calendário de Gastos**: nova seção `s-calendario` — grade mensal com cores por intensidade de gasto, detalhe ao clicar no dia; inclui variáveis, faturas de cartão e combustível
- **Exportar PDF**: botão na seção Relatório — abre popup com layout imprimível, aciona `window.print()`
- **Desejos ↔ Investimentos**: campo `investimentoId` em desejos; `C.valorDesejo(d)` usa `C.valorCalc(inv)` se vinculado; badge de vínculo em ambas as seções; `delInv()` limpa referência

### 2026-04-23 — v3.5.2 (fix: ícone + cache)
- **Ícone melhorado**: "R$" dourado Georgia Bold, fundo marrom, anel decorativo
- **Service worker v2**: `CACHE = 'lari-v2'`; ícones adicionados ao pré-cache

### 2026-04-23 — v3.5.1 (fix: bugs CSS mobile)
- **Cascade fix**: `#mobile-month-bar{display:none}` movido para antes dos `@media` queries — regra global após media query causava override permanente
- **`position:fixed`**: barra de mês trocada de `sticky` para `fixed;top:0;left:0;right:0;height:48px;z-index:50` — `sticky` não funciona em container `display:flex` row (`#app`)
- **`margin-top:48px`** em `#content` dentro do media query 768px — compensar barra fixa fora do fluxo
- **Offline bar**: `top:48px!important` no mobile — evita sobreposição com a barra de mês

### 2026-04-23 — v3.5 (responsividade mobile)
- **Barra de mês mobile**: `#mobile-month-bar` `position:fixed` no topo com botões ‹ › (`mobileMonthNav()`)
- Tabelas com `display:block; overflow-x:auto` no mobile
- Touch targets ≥44px para botões e inputs
- `.cart-grid` min-width reduzido 300px → 260px; 1-col em 480px
- Modal de importação em tela cheia (≤600px)
- `.inv-sum` 1-col em 480px; `.sal-inline input` flexível
- Font-size nav labels: 0.6rem → 0.68rem
- Novos breakpoints: 600px e ajustes em 480px

### 2026-04-23 — v3.4 (6 melhorias)
- **PWA**: `manifest.json`, `sw.js`, ícones 192/512px; instala como app no Android
- **Conflito sync Drive**: confirm() antes de sobrescrever dados locais com versão mais nova do Drive
- **Indicador offline**: barra vermelha fixa no topo quando sem conexão
- **Notificações de vencimento**: `NOTIF` module — pede permissão, notifica fixas não pagas vencendo em ≤3 dias
- **Semáforo de orçamentos**: Dashboard mostra progress bars por categoria (verde/amarelo/vermelho)
- **Relatório anual**: toggle Mensal/Anual; `CHART.renderAnual()` com barras 12 meses, donut categorias, linha saldo, tabela resumo

### 2026-04-23 — v3.3 (branding + melhorias)
- Brand alterado: `🎀 lari4.0` → `R$ Lari` (ícone removido, texto "R$" em destaque)
- Título da página: `Lari — Gestão Financeira`
- Persistência de seção via hash: `location.hash = sec`; ao recarregar retorna à seção ativa

### 2026-04-23 — v3.2 (Google Drive sync)
- Novo módulo `GDRIVE` com OAuth2 via Google Identity Services
- Client ID: `644762904112-65kfgliosm28n8j1m9862ejpn7j1lttn.apps.googleusercontent.com`
- Scope: `https://www.googleapis.com/auth/drive.file email profile`
- Auto-save (5s debounce) + auto-load (ao abrir e ao `visibilitychange`)
- Token salvo em IDB store `sync`, key `gdrive`: `{email, fileId}`
- Arquivo no Drive: `lari4_sync.json`
- Botões na sidebar: ☁️ Google Drive + 🔄 Sincronizar agora

### 2026-04-23 — v3.1 (GitHub Pages)
- App publicado em `https://larialsan.github.io/financas/`
- Repo: `LariAlSan/financas`, arquivo `index.html`
- Deploy via Python REST API (GET SHA → PUT content base64)

### 2026-04-22 — v2.4 (LYD.html — cópia para outro usuário)
- `C:\Persona\Financas\LYD.html`: paleta lilás, brand "💜 LYD", localStorage key `lyd_data`, módulo Combustível removido

### 2026-04-22 — v2.3 (Ticket Refeição)
- Nova forma de pagamento "Vale Refeição"; campo `ticket` em salarios; `C.ticketUsado(mo)`

### 2026-04-22 — v2.2 (Investimentos — reformulação)
- Campos novos: `instituicao`, `indexador`, `percentual`, `autoCalc`, `dataAplicacao`, `dataVencimento`, `liquidez`
- `C.valorCalc()`, `C.diasVenc()`, `C.totalLiquido()`; 2 gráficos de rosca; badge ⚡

### 2026-04-22 — v2.1 (Combustível — histórico completo)
- `R.combustivel()` sem filtro por mês; gráfico de tendência preço/L + km/L

### 2026-04-22 — v2.0 (Módulo Combustível + auditoria de cards)
- Módulo `s-combustivel` adicionado; cards separados Direto vs Via Cartão em Parceladas e Variáveis

### 2026-04-22 — v1.8 (histórico não-retroativo + vínculo cartão)
- `despesasFixas[].historico[]`; `C.valorFixa()`; `cartaoId` em fixas/variáveis/parceladas

### 2026-04-22 — v1.3–1.7
- Cartões de crédito, orçamentos, comparativo mensal, notas, importador CSV/PDF, formas de pagamento

### 2026-04-22 — v1.0–1.2
- Criação inicial, redesign visual pink, Playfair Display + Nunito, tema pastel suave

---

## Credenciais e configurações
- **GitHub Token**: salvo localmente (não versionado)
- **Google Drive Client ID**: `644762904112-65kfgliosm28n8j1m9862ejpn7j1lttn.apps.googleusercontent.com`
- **URL pública**: `https://larialsan.github.io/financas/`
- **Usuário GitHub**: `LariAlSan`

---
*Atualizado em 2026-04-24*
