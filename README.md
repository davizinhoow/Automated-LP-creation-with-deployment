# 🚀 Automated Landing Page Creation with Auto-Deploy

> Fluxo n8n para criação, edição e publicação automatizada de landing pages em Next.js/TSX com deploy automático via GitHub.

---

## 📋 Visão Geral

Este fluxo automatiza todo o ciclo de vida de uma landing page — da geração via IA até o deploy em produção — passando por um ambiente de **staging** para validação antes da publicação oficial.

O sistema opera em **três fluxos independentes** acionados por webhooks:

| Fluxo | Descrição |
|-------|-----------|
| 🟢 **Criação** | Gera uma nova LP do zero a partir de um briefing |
| 🟡 **Alteração** | Edita uma LP existente com base em instruções |
| 🔵 **Publicação** | Promove a LP do staging para produção |

---

## 🏗️ Arquitetura do Fluxo

```
                          ┌─────────────────────────────────────────────┐
                          │             FLUXO DE CRIAÇÃO                │
                          │                                             │
  POST /create-landing-page                                             │
         │                                                             │
         ▼                                                             │
  ┌─────────────┐    ┌──────────────┐    ┌──────────────────────┐      │
  │   Webhook   │───▶│ Extract PDF  │───▶│  Aprimora Briefing   │      │
  │             │    │  (Briefing)  │    │  (Gemini 2.5 Pro)    │      │
  └─────────────┘    └──────────────┘    └──────────┬───────────┘      │
         │                                           │                 │
         │           ┌──────────────┐                ▼                 │
         └──────────▶│ Aprimora     │──▶  ┌──────────────────────┐    │
                     │  Prompt      │     │        Merge         │    │
                     │ (Gemini Pro) │     │ (Briefing + Prompt)  │    │
                     └──────────────┘     └──────────┬───────────┘    │
                                                      │               │
                                                      ▼               │
                                          ┌──────────────────────┐    │
                                          │  Cria arquivo React  │    │
                                          │   (Agente Gemini)    │    │
                                          └──────────┬───────────┘    │
                                                      │               │
                                                      ▼               │
                                          ┌──────────────────────┐    │
                                          │ Aprimora arquivo TSX │    │
                                          │  (SEO, Design, UX)   │    │
                                          └──────────┬───────────┘    │
                                                      │               │
                              ┌───────────────────────┘               │
                              ▼                                        │
                  ┌───────────────────────┐                            │
                  │   Commit → STAGING    │                            │
                  │  (branch: staging)    │                            │
                  └───────────┬───────────┘                            │
                              │                                        │
                              ▼                                        │
                  ┌───────────────────────┐                            │
                  │   Registra no BD      │                            │
                  │  (API Anchieta)       │                            │
                  └───────────────────────┘                            │
                                                                       │
                          └─────────────────────────────────────────┘


                          ┌─────────────────────────────────────────────┐
                          │             FLUXO DE ALTERAÇÃO              │
                          │                                             │
  POST /update-landing-page                                             │
         │                                                             │
         ▼                                                             │
  ┌─────────────┐    ┌──────────────────────┐                          │
  │   Webhook   │───▶│ Extrai slug da URL   │                          │
  │             │    │ (modalidade + curso) │                          │
  └─────────────┘    └──────────┬───────────┘                          │
                                │                                      │
                                ▼                                      │
                    ┌──────────────────────┐                           │
                    │ Busca arquivo .tsx   │                           │
                    │  existente (GitHub)  │                           │
                    └──────────┬───────────┘                           │
                               │                                       │
                               ▼                                       │
                    ┌──────────────────────┐                           │
                    │  Aprimora alterações │                           │
                    │   (Agente Gemini)    │                           │
                    └──────────┬───────────┘                           │
                               │                                       │
                               ▼                                       │
                    ┌──────────────────────┐                           │
                    │  Refaz o código TSX  │                           │
                    │   com as mudanças    │                           │
                    └──────────┬───────────┘                           │
                               │                                       │
                               ▼                                       │
                    ┌──────────────────────┐                           │
                    │  Commit → STAGING    │                           │
                    │  (branch: staging)   │                           │
                    └──────────────────────┘                           │
                                                                       │
                          └─────────────────────────────────────────┘


                          ┌─────────────────────────────────────────────┐
                          │             FLUXO DE PUBLICAÇÃO             │
                          │                                             │
  POST /update-landing-page-status                                      │
         │                                                             │
         ▼                                                             │
  ┌─────────────┐    ┌──────────────────────┐                          │
  │   Webhook   │───▶│ Extrai slug da URL   │                          │
  │             │    │ (modalidade + curso) │                          │
  └─────────────┘    └──────────┬───────────┘                          │
                                │                                      │
                                ▼                                      │
                    ┌──────────────────────┐                           │
                    │ Busca arquivo .tsx   │                           │
                    │  no branch staging   │                           │
                    └──────────┬───────────┘                           │
                               │                                       │
                               ▼                                       │
                    ┌──────────────────────┐                           │
                    │ Commit → MAIN/PROD   │                           │
                    │   (branch: main)     │                           │
                    └──────────┬───────────┘                           │
                               │                                       │
                               ▼                                       │
                    ┌──────────────────────┐                           │
                    │ Atualiza status BD   │                           │
                    │    → PUBLICADO       │                           │
                    └──────────────────────┘                           │
                                                                       │
                          └─────────────────────────────────────────┘
```

---

## ⚙️ Componentes e Tecnologias

### 🤖 IAs Utilizadas

| Modelo | Função |
|--------|--------|
| **Gemini 2.5 Pro** | Aprimoramento do briefing recebido |
| **Gemini 3.1 Pro Preview** | Criação do arquivo TSX, refinamento e edição de código |
| **Kimi K2.5** (Moonshot AI) | Geração inicial de estrutura — suporte adicional de prompt |

### 🔗 Integrações

| Serviço | Uso |
|---------|-----|
| **GitHub** | Versionamento e commit dos arquivos `.tsx` nos branches `staging` e `main` |
| **API Interna (Anchieta)** | Registro e atualização de status das landing pages no banco de dados |

### 🏗️ Stack do Projeto de Landing Pages

- **Framework:** Next.js com App Router
- **Linguagem:** TypeScript (TSX)
- **Build:** Necessário executar `npm run build` no servidor após cada deploy
- **Estrutura de rotas:** `app/(modalidades)/[tipo_slug]/[slug]/pages/[arquivo].tsx`

---

## 📂 Estrutura de Arquivos no Repositório

```
site-anchieta/
└── app/
    └── (modalidades)/
        └── [tipo_slug_modalidade]/
            └── [slug]/
                └── pages/
                    └── [slug_do_curso].tsx   ← Landing Page gerada
```

---

## 🔄 Ciclo de Vida de uma Landing Page

```
1. [INPUT]       Requisição via webhook com briefing (PDF) + dados do curso
        │
        ▼
2. [IA]          Aprimoramento de briefing e prompt com Gemini
        │
        ▼
3. [IA]          Geração do arquivo TSX (Next.js) com foco em conversão e SEO
        │
        ▼
4. [IA]          Refinamento de responsividade, design e SEO
        │
        ▼
5. [GITHUB]      Commit no branch `staging`
        │
        ▼
6. [SERVIDOR]    Build do projeto React no servidor de staging
        │         → npm run build
        ▼
7. [VALIDAÇÃO]   Revisão da página em: https://lp-staging.anchieta.br/...
        │
        ▼
8. [WEBHOOK]     Aprovação → POST /update-landing-page-status
        │
        ▼
9. [GITHUB]      Commit no branch `main` (produção)
        │
        ▼
10. [SERVIDOR]   Build do projeto React no servidor de produção
        │         → npm run build
        ▼
11. [BD]         Status atualizado para → PUBLICADO
        │
        ▼
12. [PRODUÇÃO]   Página live em: https://anchieta.br/...
```

---

## 🌐 Webhooks

| Endpoint | Método | Descrição |
|----------|--------|-----------|
| `/webhook/create-landing-page` | `POST` | Inicia a criação de uma nova landing page |
| `/webhook/update-landing-page` | `POST` | Solicita alterações em uma LP existente |
| `/webhook/update-landing-page-status` | `POST` | Publica a LP do staging para produção |

### Exemplo de Payload — Criação

```json
{
  "data": {
    "nome_curso": "Administração",
    "slug_curso": "administracao",
    "tipo_slug_modalidade": "graduacao",
    "briefing_pdf": "<arquivo PDF em base64>"
  }
}
```

### Exemplo de Payload — Publicação

```json
{
  "id": "123",
  "url": "https://lp-staging.anchieta.br/graduacao/administracao"
}
```

---

## 🖥️ Deploy e Build no Servidor

Após cada commit automático (staging ou produção), é necessário executar o build do projeto Next.js no servidor correspondente.

### Servidor de Staging

```bash
cd /var/www/site-anchieta
git pull origin staging
npm install
npm run build
pm2 restart site-staging   # ou o processo equivalente
```

### Servidor de Produção

```bash
cd /var/www/site-anchieta
git pull origin main
npm install
npm run build
pm2 restart site-prod   # ou o processo equivalente
```

> 💡 **Recomendação:** Configure um webhook no GitHub Actions ou um script de CI/CD para disparar o build automaticamente após cada push nos branches `staging` e `main`, eliminando a necessidade de intervenção manual.

---

## 🔐 Variáveis e Credenciais Necessárias

| Variável | Descrição |
|----------|-----------|
| `GOOGLE_GEMINI_API_KEY` | Chave da API Google Gemini (configurada no n8n) |
| `MOONSHOT_API_KEY` | Chave da API Kimi/Moonshot AI |
| `GITHUB_TOKEN` | Token com permissão de escrita no repositório |
| `API_BD_URL` | URL da API interna para registro de LPs |

> ⚠️ Todas as credenciais devem ser configuradas via **Credentials** do n8n. Nunca versione chaves de API no repositório.

---

## 📌 Observações

- O fluxo de **Criação** gera um arquivo TSX otimizado para **conversão**, **SEO** e **responsividade**, com formulário de matrícula integrado.
- O fluxo de **Alteração** preserva a estrutura base e a lógica de conversão já existente, aplicando apenas as mudanças solicitadas.
- O fluxo de **Publicação** copia o arquivo do branch `staging` para o branch `main`, garantindo que apenas páginas validadas vão para produção.
- O banco de dados interno rastreia o status de cada LP: `EM CRIAÇÃO` → `STAGING` → `PUBLICADO`.

---

## 🗂️ Fluxo n8n

O arquivo de exportação do fluxo está disponível em:

```
Marketing_-_Criação_de_LandingPages.json
```

Para importar no n8n: **Settings → Import from File → selecione o JSON**.
