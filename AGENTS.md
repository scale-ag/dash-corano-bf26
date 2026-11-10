# AGENTS.md — Dashboard de captura de leads · Bruno Corano (BC-BF26)

> Contexto completo em **`CLAUDE.md`** — leia antes de mexer no projeto.

- Repositório `scale-ag/dash-corano-bf26`, Pages em `https://scale-ag.github.io/dash-corano-bf26/`.
- `build/build.py`: `SPREADSHEET_ID_FUNIL` (captura, aba `Página1`), `SPREADSHEET_ID_META`
  (gerenciador, aba `Página1`), `CLIENT_NAME="Bruno Corano"`, `MAIN_PRODUCT="Funil Black Friday 2026"`,
  `MAIN_PRODUCT_PREFIX="BC-BF26"`, `TAX_FACTOR=1.1385` (toggle Imposto Meta ligado por padrão).
- Funil só de captação: sem MQL, sem Vendas, sem CRM, sem Seguidores/ChatGPT (abas `SHEET_*=None`).
- Cruzamento por UTMs: `utm_campaign`/`utm_medium`/`utm_content` = Campanha/Conjunto/Anúncio.
- Insights por IA (`briefing.yml` + Routine) **desligados** neste cliente.
- cron-job.org: `SETUP-CRON.md`.
