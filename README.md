# Dashboard de Captura de Leads · Bruno Corano — Funil Black Friday 2026 (BC-BF26)

Dashboard **100% na nuvem** de controle de tráfego pago do **Funil Black Friday
2026** de **Bruno Corano** (sigla de campanha `BC-BF26`). Cruza os leads do
formulário de captura com o investimento do **Meta Ads** e mostra o funil
**Gasto → Impressões → Cliques → Page Views → Leads**, com CPM, CTR, CPC,
Connect Rate, ConvLP e **CPL** por campanha, conjunto e anúncio. É publicada no
**GitHub Pages** e reconstruída a cada ~30 min pelo **cron-job.org**, sem
depender de nenhum PC ligado.

**URL pública:** `https://scale-ag.github.io/dash-corano-bf26/`

## Páginas

- **Visão Geral:** todos os leads (Meta + orgânicos), funil, evolução diária,
  tabela diária com heatmap, KPIs secundários e distribuição por origem,
  campanha, plataforma e anúncio.
- **Funil de Leads (Meta Ads):** só leads atribuídos à Meta. Funil, evolução
  diária, leads por anúncio e conjunto, compilado por CPL e hierarquia
  Campanha → Conjunto → Anúncio com filtro cruzado.
- **Relatório:** espelho da Visão Geral, painel de metas editável (Meta CPL,
  volume mínimo de leads) e ranking de anúncios por CPL.

Neste funil **não há** classificação de leads (MQL), planilha de compradores
nem CRM. Essas métricas foram removidas.

## Fontes de dados (somente leitura)

Buscadas por **nome de aba** (`gviz/tq?tqx=out:csv&sheet=...`), sem gid:

| Planilha | ID | Aba | Colunas |
|---|---|---|---|
| Captura (leads) | `1c9VDkBO6UAq52IQEM8YQHa3f4yhtYkKHn0sXqAjpl9w` | `Página1` | Data · Hora · Nome · Email · Telefone · utm_campaign · utm_source · utm_content · utm_term · utm_medium · fbclid · id · sck |
| Gerenciador Meta Ads | `14nKN2tWHwpSXEDzbnCtuRgd3xhT7rVrEVjYvBPH26S0` | `Página1` | Day · Campaign Name · Ad Set Name · Ad Name · Amount Spent · Impressions · Link Clicks · Landing Page Views |

**Nada é escrito de volta** nas planilhas.

## Arquitetura

```
cron-job.org  ──(POST workflow_dispatch a cada 30 min)──▶  GitHub Actions
                                                              │
                          build/build.py  lê os CSVs ◀────────┘
                                 │  cruza leads × Meta Ads por UTM
                                 ▼
                          dist/index.html  ──▶  deploy  ──▶  GitHub Pages
```

**Cache-bust:** metatags `no-cache`, horário do último build na tela, botão
**Atualizar** e recarga automática (`?t=timestamp`) ~30 min após aberta.

## Rodar localmente (opcional)

```bash
python build/build.py --leads-file leads.csv --meta-file meta.csv --out dist/index.html
```

## cron-job.org

Veja **`SETUP-CRON.md`**.

> ⚠️ Nunca comite tokens. Use um token *fine-grained* só com **Actions:
> read/write** neste repositório, cadastrado apenas no cron-job.org.
