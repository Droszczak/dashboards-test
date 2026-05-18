# Dashboards Test — GitHub Actions + Databricks

Projeto de teste do pipeline CI/CD para deploy do dashboard **NYC Taxi Trip Analysis** no Databricks via GitHub Actions + Asset Bundles.

---

## Estrutura

```text
dashboards_test/
├── .github/
│   └── workflows/
│       └── deploy.yml                              ← GitHub Actions
├── databricks.yml                                  ← Bundle config
├── README.md
└── src/
    └── samples_nyctaxi_nyc_taxi_trip_analysis.lvdash.json
```

---

## Setup (passo a passo)

### 1. Descobrir o nome do SQL Warehouse

Acesse: **Databricks → SQL Warehouses**

Copie o nome exato do warehouse disponível (ex: `Starter Warehouse`, `Serverless Starter`).
Abra o `databricks.yml` e atualize:

```yaml
warehouse_name: "nome exato aqui"
```

### 2. Gerar o Personal Access Token (PAT)

Databricks → (ícone do usuário) → Settings → Developer → Access Tokens → Generate New Token

- Description: `github-actions-test`
- Lifetime: 90 dias (ou o que preferir)

Guarde o token — você só vê uma vez.

### 3. Criar o repositório no GitHub

Abra o terminal na pasta `dashboards_test` e execute:

```bash
git init
git add .
git commit -m "feat: add NYC taxi dashboard test pipeline"
gh repo create dashboards-test --public --push --source=.
```

> Precisa do GitHub CLI instalado. Se não tiver: `winget install GitHub.cli`

### 4. Configurar Secrets no GitHub

No repositório criado → **Settings → Secrets and Variables → Actions → New repository secret**:

| Secret | Valor |
| --- | --- |
| `DATABRICKS_HOST` | `https://dbc-1e4a8057-b00e.cloud.databricks.com` |
| `DATABRICKS_TOKEN` | PAT gerado no passo anterior |

### 5. Configurar os GitHub Environments (para aprovação manual em prod)

Settings → Environments → New environment

- Crie `dev` (sem proteção)
- Crie `production` → marque **Required reviewers** → adicione seu usuário

### 6. Testar o pipeline

```bash
git checkout -b feature/test-deploy
# Faça qualquer pequena alteração (ex: adicionar um espaço no README)
git add README.md
git commit -m "test: trigger pipeline"
git push origin feature/test-deploy
```

Abra um **Pull Request** no GitHub.

O pipeline vai rodar automaticamente:

1. `validate` — valida JSON e bundle config
2. `deploy-dev` — deploya o dashboard no Databricks

Após o deploy, verifique em: `https://dbc-1e4a8057-b00e.cloud.databricks.com/dashboards`

O dashboard vai aparecer como **[dev] NYC Taxi Trip Analysis**.

---

## Fluxo do pipeline

```text
Pull Request aberto
      ↓
[validate]   valida JSON + databricks bundle validate
      ↓
[deploy-dev] databricks bundle deploy --target dev  ← automático
      ↓  (merge na main)
[deploy-prod] requer aprovação manual               ← botão no GitHub
```

---

## Notas sobre este dashboard

- Usa o catálogo `samples.nyctaxi.trips` — built-in do Databricks, sem prefixo de ambiente
- Não requer Unity Catalog personalizado para funcionar
- Não há substituição de catálogo (sem `dev_` / `prd_`) — as queries já apontam para o catálogo correto
