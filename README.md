# WealthHerHer

> **Wealth literacy + on-chain savings for women 40+.**
> Educação financeira gamificada com cofrinho de reserva de emergência em USDC na Solana.

[![Solana](https://img.shields.io/badge/Solana-Devnet-9945FF?logo=solana&logoColor=white)](https://explorer.solana.com/?cluster=devnet)
[![Lovable Cloud](https://img.shields.io/badge/Backend-Lovable_Cloud-FF6B6B)](https://lovable.dev)
[![React](https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=black)](https://react.dev)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Languages](https://img.shields.io/badge/i18n-pt%20%7C%20en%20%7C%20es-blue)]()

🌎 **Live demo:** https://wealthherher.lovable.app
🛠 **Submetido para:** Hackanation 2026 — Trilha Solana (Real World Web3 Applications)
🎯 **Categoria principal:** Payments, RWAs & Tokenização

---

## 📖 English summary (for judges)

WealthHerHer is a financial literacy platform for women 40+ in Latin America. We combine a gamified education flow (lessons, XP, streaks, quests, AI companion) with an **on-chain emergency-fund vault** built on Solana devnet using SPL USDC + the Memo program. Every deposit is a **self-custodial, on-chain, verifiable** commitment. The platform is fully internationalized (PT/EN/ES) and ships with a Vitest-based RLS regression suite that blocks CI when permission boundaries break.

---

## 🩷 O problema

> **75% das mulheres brasileiras a partir dos 40 não têm reserva de emergência.** (SPC, 2024)
> A diferença de aposentadoria entre homens e mulheres no Brasil chega a **30%**. (OCDE, 2023)

Educação financeira tradicional não engaja. Bancos não falam com essa persona. Cripto, do jeito que é apresentado hoje, intimida.

## 💡 A solução

**WealthHerHer** une três camadas que normalmente vivem separadas:

1. **Educação** — trilhas curtas, quiz de clareza, lições com checkpoints e XP.
2. **Ferramentas** — calculadoras de reserva de emergência, patrimônio líquido e gap de aposentadoria.
3. **Ação on-chain** — cofrinho USDC na Solana que transforma a meta calculada num compromisso público e verificável, **sem a usuária jamais perder a custódia do dinheiro dela**.

---

## ⛓ Como a Solana entra (Trilha 1 — Payments / RWAs / Tokenização)

```
┌──────────────────────────────────────────────────────────────────┐
│                     WealthHerHer Web App                         │
│                                                                  │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────────────┐  │
│  │  Calculadora │──▶│  Meta (BRL)  │──▶│   Cofrinho USDC      │  │
│  │  Reserva     │   │  → USDC      │   │   (auto-custódia)    │  │
│  └──────────────┘   └──────────────┘   └──────────┬───────────┘  │
│                                                    │              │
│                                          assina tx │              │
└────────────────────────────────────────────────────┼──────────────┘
                                                     ▼
                       ┌─────────────────────────────────────────┐
                       │       Solana Devnet                     │
                       │  • SPL Token (USDC) transferChecked     │
                       │  • Memo Program (WHH:goal:<uuid>)       │
                       └────────────┬────────────────────────────┘
                                    │ signature
                                    ▼
                       ┌─────────────────────────────────────────┐
                       │  Edge Function verify-solana-deposit    │
                       │  • Re-fetches tx via RPC                │
                       │  • Validates mint, owner, memo          │
                       │  • Persists deposit + recomputes total  │
                       │  • Unlocks achievement on completion    │
                       └─────────────────────────────────────────┘
```

### Decisões técnicas

| Decisão | Por quê |
|---|---|
| **Self-transfer de USDC + memo** | Preserva auto-custódia total. Phantom mostra uma tx USDC real, on-chain, verificável. O dinheiro **nunca sai da carteira da usuária**. Memo `WHH:goal:<uuid>` amarra a tx à meta. |
| **Verificação server-side via RPC** | Cliente nunca grava em `solana_deposits` (sem policy de INSERT). Edge function re-fetches a tx, valida `mint == USDC_DEVNET`, `authority == wallet`, memo pattern, e insere via service-role. **Cliente não pode mentir.** |
| **Idempotência por `tx_signature` UNIQUE** | Tentativas duplicadas retornam 200 sem corromper o saldo. |
| **USDC devnet oficial Circle** (`4zMMC9sr...DLGmf`) | Padrão da indústria; faucet pública. |
| **PDA-style state via tabela espelho** | Sem programa Anchor custom (escopo de hackathon), o estado on-chain é a **soma de todas as txs com memo válido**. A tabela `solana_savings_goals` é só um cache para leitura rápida — pode ser reconstruída do zero a partir da chain. |
| **Achievement on completion** | Ao bater 100%, o RPC `unlock_achievement('emergency-fund-onchain')` concede +200 XP e desbloqueia o badge. Roadmap: substituir por **cNFT (Metaplex Bubblegum)** mintado direto na carteira (~US$ 0.00025/mint). |

---

## ✨ Features completas

| Área | O que tem |
|---|---|
| 🎓 **Aprender** | 5 lições com checkpoints, multi-idioma, XP por conclusão |
| 🧮 **Calculadoras** | Reserva de Emergência (com vault on-chain), Patrimônio Líquido, Gap de Aposentadoria |
| 💰 **Cofrinho Solana** | Phantom/Solflare, USDC devnet, memo on-chain, histórico explorer-linked |
| 🏆 **Gamificação** | XP, níveis (1-4), streaks com freezes, weekly quests, 14+ achievements |
| 💬 **Sofia AI Companion** | Tutor financeiro via Lovable AI Gateway (Gemini 2.5 Flash) — modos Ask / Plan / Reflect |
| 👥 **Comunidade** | Fórum por categoria, threads anônimas, upvotes |
| 👩‍🏫 **Coaches** | Diretório + sistema de booking |
| 🛟 **Feedback** | Widget flutuante visível só para admin no backend |
| 🌐 **i18n** | Português, English, Español — detecção automática |
| 🔒 **Segurança** | RLS em todas as tabelas, GRANTs explícitos, **30 testes de regressão** rodando em CI |

---

## 🏗 Arquitetura

```
Frontend (Vite + React 18 + Tailwind + shadcn/ui)
  ├── @solana/web3.js, @solana/spl-token, wallet-adapter (Phantom/Solflare)
  ├── i18next (pt/en/es)
  └── React Query + Supabase JS

Backend (Lovable Cloud / Supabase)
  ├── Postgres + RLS em 18 tabelas
  ├── SECURITY DEFINER RPCs: award_xp, unlock_achievement, has_role
  ├── Edge functions (Deno):
  │   ├── companion-chat        — streaming AI via Lovable AI Gateway
  │   └── verify-solana-deposit — tx validation server-side
  └── Realtime: per-user XP/achievement channels

Solana
  ├── Devnet RPC (https://api.devnet.solana.com)
  ├── USDC mint: 4zMMC9srt5Ri5X14GAgXhaHii3GnPAEERYPJgZJDncDU
  └── Memo Program: MemoSq4gqABAXKb96qnH8TysNcWxMyWCqXgDLGmfcHr
```

### Stack completa

`React 18` · `TypeScript 5` · `Vite 5` · `Tailwind v3` · `shadcn/ui` · `Radix UI` · `react-router-dom` · `react-i18next` · `Zod` · `React Query` · `Supabase (Postgres + Edge Functions + Realtime)` · `Lovable AI Gateway (Gemini 2.5 Flash)` · `@solana/web3.js` · `@solana/spl-token` · `@solana/wallet-adapter-react` · `Vitest`

---

## 🎬 Demo flow para os jurados (2 minutos)

1. Cadastre-se em `/auth` (PT é default).
2. Faça o quiz em `/quiz` — recebe trilha personalizada.
3. Vá para `/plan/emergency-fund`:
   - Aba **Calcular**: defina gastos mensais + meses → meta gerada (ex.: R$ 18.000).
   - Aba **Cofrinho on-chain**: clique em **Connect Wallet** → escolha Phantom (devnet).
   - Pegue USDC grátis em https://faucet.circle.com/ (selecione Solana Devnet).
   - Clique **Criar meta** → registra goal no Postgres.
   - Clique **Depositar** → 10 USDC → Phantom abre, você assina → **tx aparece no Solana Explorer**.
   - Edge function valida server-side, soma o depósito, atualiza progresso.
   - Repita até 100% → achievement on-chain desbloqueado (+200 XP).
4. Veja o histórico com links clicáveis para cada tx no explorer.

---

## 🚀 Como rodar localmente

```bash
git clone <este-repo>
cd wealthherher
bun install
bun dev
```

O arquivo `.env` é auto-gerado pelo Lovable Cloud com:

```
VITE_SUPABASE_URL=...
VITE_SUPABASE_PUBLISHABLE_KEY=...
VITE_SUPABASE_PROJECT_ID=...
```

**Para testar o cofrinho:** instale a extensão [Phantom](https://phantom.app) e mude para Devnet em Settings → Developer Settings.

### Rodar os testes de segurança

```bash
bun run test:security
```

Saída esperada: `Tests  30 passed (30)`. Se algum falhar, **uma policy de RLS regrediu** e a PR não deve ser aceita.

---

## 🔒 Modelo de segurança

- Todas as 18 tabelas têm RLS habilitado.
- Cada migração inclui `GRANT` explícito por role (`anon`, `authenticated`, `service_role`).
- Tabelas per-user usam `auth.uid() = user_id`.
- `user_achievements` e `solana_deposits` **não têm policy de INSERT** — só edge functions / RPCs SECURITY DEFINER podem gravar, garantindo que XP e depósitos verificados não sejam forjáveis.
- `forum_threads.author_id` revogado do `anon` (column-level) para não vazar UUIDs.
- `feedback`: anon pode INSERT, mas SELECT/UPDATE/DELETE são restritos a `has_role(auth.uid(),'admin')`.
- Companion edge function: cap de 20 mensagens, content truncado em 2000 chars, erros sanitizados.
- Verificação server-side de toda tx Solana antes de persistir.

---

## 🗺 Roadmap pós-hackathon

- [ ] Migrar para **Solana mainnet** com USDC real (Circle CCTP para bridge BRL via PIX→USDC)
- [ ] **cNFT certificate** (Bubblegum) ao atingir 100% — portabilidade total da conquista
- [ ] Opt-in yield via **Kamino/MarginFi** lending (educacional, com aviso de risco)
- [ ] **Programa Anchor custom** com PDA-vault próprio (substituir o memo pattern)
- [ ] Integração **PIX → USDC** via parceria com on/off-ramp brasileiro
- [ ] B2B: white-label para ONGs e cooperativas de educação financeira feminina

---

## 👩‍💻 Equipe

**Lannara Silva** — Product + Design + Build
Hackanation 2026 — TokenNation

---

## 📄 Licença

MIT — veja [LICENSE](LICENSE).
