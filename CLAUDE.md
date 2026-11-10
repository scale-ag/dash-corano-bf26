# CLAUDE.md — Contexto do projeto (Dashboard Bruno Corano · Funil Black Friday 2026 · BC-BF26)

> Lido automaticamente pelo Claude Code ao abrir o repositório. Carrega todo o
> contexto para continuar o trabalho sem depender de mensagens anteriores.
> Mantenha-o atualizado. Este repo nasceu do template de dashboard da Scale
> (cópia do dash Lucas Nigro/RAL) e foi **reconfigurado em 10/11/2026** para
> este cliente; para replicar o modelo, veja `GUIA-REPLICACAO.md`.

---

## Status da configuração

- **Repositório:** `scale-ag/dash-corano-bf26` · **URL pública:** `https://scale-ag.github.io/dash-corano-bf26/`.
- **`build/build.py`:** `SPREADSHEET_ID_FUNIL` (planilha de captura), `SPREADSHEET_ID_META`
  (gerenciador), `CLIENT_NAME="Bruno Corano"`, `MAIN_PRODUCT="Funil Black Friday 2026"`,
  `MAIN_PRODUCT_PREFIX="BC-BF26"`, `TAX_FACTOR=1.1385`.
- **Funil só de captação:** `Gasto → Impressões → Cliques → Page Views → Leads`.
  **Sem critério de MQL** (leads não são classificados — `q` é sempre 0), **sem
  planilha de compradores** e **sem CRM** — essas métricas foram removidas da UI
  (funil, KPIs, tabelas, gráficos, Top Anúncios). Métrica de decisão: **CPL**.
- Abas do template que não existem aqui ficam desligadas em `build.py`
  (`SHEET_VENDAS`/`SHEET_COM_*`/`SHEET_SEG = None` → `load_rows` devolve `[]`,
  sem buscar — o gviz devolveria a 1ª aba para um nome inexistente).
- Páginas: **Visão Geral** · **Funil de Leads (Meta Ads)** · **Relatório**. As páginas
  Seguidores e ChatGPT Ads saíram do menu (o código `renderSeg`/`renderGpt` ficou
  em `app.js` como engine, sem uso).
- **Insights de Tráfego por IA: desligados** (bloco removido do Relatório,
  `briefing.yml` só manual). Para ligar: adaptar `GUIA-RELATORIOS.md`,
  `GUIA-INTERPRETACAO-METRICAS.md` e `coletar_dados_relatorio.py` ao funil sem
  MQL, religar o schedule e criar a Routine do Claude (`create_trigger`).
- `build/identidade-visual.css`: cores default do template.

## Fontes de dados (Google Sheets, somente leitura)

Buscadas **por nome de aba** (`gviz/tq?tqx=out:csv&headers=1&sheet=<nome>`).

**Planilha de captura** — `SPREADSHEET_ID_FUNIL = "1c9VDkBO6UAq52IQEM8YQHa3f4yhtYkKHn0sXqAjpl9w"`,
aba **`Página1`** (`SHEET_LEADS`, fonte única de leads):
`Data` · `Hora` · `Nome` · `Email` · `Telefone` · `utm_campaign` · `utm_source` ·
`utm_content` · `utm_term` · `utm_medium` · `fbclid` · `id` · `sck`.
`Data` (+`Hora`) já em BRT. `utm_term` = ID do anúncio (não usado).

**Planilha Meta Ads** — `SPREADSHEET_ID_META = "14nKN2tWHwpSXEDzbnCtuRgd3xhT7rVrEVjYvBPH26S0"`,
aba **`Página1`** (`SHEET_META`, sem espaço):
`Day` · `Campaign Name` · `Ad Set Name` · `Ad Name` · `Amount Spent` · `Impressions` ·
`Link Clicks` · `Landing Page Views`.

### Cruzamento por UTMs (Leads × Meta Ads)
`utm_campaign` = Campaign Name · `utm_medium` = Ad Set Name · `utm_content` = Ad Name.
Sigla do funil: **`BC-BF26`** (ex.: `BC-BF26 | E2-CAP | P1-QUENTE | LEAD | CBO | 2026-11-09 | All in one`).

### Classificação da origem (`build.py::process`)
- `utm_source` começando com `meta`/`facebook`/`fb`/`instagram`/`ig` (ou vazio) **com
  campanha** → `src="meta"` (página Meta Ads e Top Anúncios usam só esses).
- `utm_source` orgânico (`organico`, `manychat`…; ex.: campanha `mctv`) ou sem
  campanha → `src="org"` (entra só na Visão Geral).
- **Leads de teste descartados** (`is_test_row`): nome começando com "teste",
  e-mail `teste…@` ou qualquer UTM contendo "teste".
- Para os gráficos da Visão Geral: `bucket` = `utm_campaign` cru ("Leads por
  campanha") e `prof` = `utm_content` ("Leads por anúncio").

### Imposto da mídia paga
`TAX_FACTOR = 1.1385` (13,85%, padrão do template). Toggle "Imposto Meta" ligado
por padrão (`STATE.tax=true`); `taxf()` multiplica o gasto em Gasto/CPM/CPC/CPV/CPL.

### Aba Relatório
Espelha a Visão Geral + painel de metas editável (**Meta CPL**, volume mínimo
amostral em **leads**, N dias p/ corte; `localStorage['dm_metas']`; o campo
interno continua `METAS.cpmql` por compatibilidade) + **Top Anúncios**
ranqueados por **CPL** (menor primeiro). Amostra relevante = gasto ≥
`SAMPLE_MIN_SPEND` (R$ 100) **e** leads ≥ volume mínimo (default
`SAMPLE_MIN_MQLS = 10`, aqui em leads); senão "Em observação".

**Ordem das colunas nas tabelas diárias:** `Data · Dia · Gasto · CPM · CTR · CPC ·
Page Views · Connect Rate · ConvLP · Leads · CPL`.

## Arquitetura / arquivos

```
build/build.py            # lê os CSVs (read-only), emite REGISTROS BRUTOS (leads[]/meta[]/sales[]/ad_links); render() COSTURA os 4 arquivos abaixo
build/template.html       # esqueleto HTML. Placeholders __STYLES__, __APP_JS__, __DATA_JSON__, __BUILD_ID__, __GENERATED_BRT__
build/identidade-visual.css  # TODAS as cores (tema claro=padrão / escuro). Mexa AQUI p/ trocar só cor
build/estilos.css         # layout/componentes (sidebar, topbar, period-picker, funil, tabelas, gráficos, aba Relatório)
build/app.js              # lógica + renderização (KPIs, funil, tabelas, filtro cruzado, period-picker, heatmap, Relatório)
build/relatorios.json     # Insights de Tráfego do DIA (aba Relatório), 6 seções — VERSIONADO; lido no build, sem API. Vazio no template ({}).
build/relatorios_dados.json      # números brutos do fechamento do dia (insumo p/ a Routine escrever relatorios.json) — não lido pelo site. Vazio no template ({}).
build/relatorio_lib.py           # agregação (agg/derived/compare) compartilhada (gerar_relatorios.py + coletar_dados_relatorio.py)
build/coletar_dados_relatorio.py # gera relatorios_dados.json (só números, sem texto) — roda no briefing.yml, 1x/dia
build/gerar_relatorios.py        # gera relatorios.json determinístico (sem IA) — fallback MANUAL, não roda mais sozinho
build/GUIA-RELATORIOS.md            # formato/estrutura dos Insights (as 6 seções) — funil, fase de calibração, gargalo de dado
build/GUIA-INTERPRETACAO-METRICAS.md # base de conhecimento por métrica (CTR/Connect Rate/ConvLP/CPL/TxMQL/CPMQL/Agendamento/CAC) — leitura obrigatória p/ redigir
.github/workflows/deploy.yml    # roda build.py e publica no Pages (workflow_dispatch + schedule + push)
.github/workflows/briefing.yml  # roda coletar_dados_relatorio.py e commita relatorios_dados.json na main (cron 1x/dia)
dist/index.html           # saída gerada (gitignored; o Actions reconstrói)
GUIA-REPLICACAO.md        # como replicar este modelo para outros relatórios/clientes
SETUP-CRON.md             # valores exatos do cron-job.org (com marcadores a preencher)
```

## Rodar/testar local

```bash
python build/build.py --leads-file leads.csv --meta-file meta.csv --out dist/index.html
# (o sandbox do agente NÃO alcança docs.google.com; use CSVs locais para testar.
#  O runner do GitHub Actions tem internet e busca os CSVs ao vivo.)
```

**Regras obrigatórias das tabelas** (ver `GUIA-REPLICACAO.md`): cabeçalho sticky;
ordenação tri‑state; colunas redimensionáveis (persist localStorage); linha
"Total Geral" fixa; dimensão nunca truncada; seleção com toggle + Ctrl multi;
filtro cruzado bidirecional; tabela diária com último dia no topo; heatmap de cor
fixa por métrica.

## Lacunas de dados
- Sem classificação de lead (MQL), sem compradores, sem CRM — fora do escopo deste funil.
- Sem coluna de permalink do criativo no export da Meta → "Link" não aparece.

## Publicação — problemas conhecidos
1. **Push:** se a integração GitHub da sessão for somente‑leitura (403), o caminho
   é `git push` direto para `github.com` com o **PAT do usuário**. Nunca gravar o
   token no `.git/config` (usar URL efêmera `https://x-access-token:<TOKEN>@github.com/...`).
2. **cron-job.org só funciona na `main`:** `workflow_dispatch` só existe na branch
   padrão. Levar `build/` + `.github/workflows/deploy.yml` para a `main`.
3. **Pages liga sozinho:** `actions/configure-pages@v5` com `enablement: true`
   (precisa `permissions: pages: write, id-token: write`).
4. **Proxy do sandbox:** o ambiente do agente costuma NÃO alcançar `docs.google.com`,
   `*.github.io` nem a API REST de Actions/Pages — mas o runner do Actions alcança tudo.
5. **Token exposto:** se um token foi colado no chat, **revogar e gerar um novo**.

## Branch / git
- Desenvolvimento na branch designada da sessão; manter sincronizada com `main`.
