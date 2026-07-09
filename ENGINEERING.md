# Manual de Engenharia — Vertex Devs

Fonte da verdade de como se constrói software na Vertex. Vale para todos os repos da org.
Quando um repo precisar divergir, o desvio é documentado no `CLAUDE.md` do próprio repo.

## 1. Princípios

1. **Sob medida > genérico.** O sistema nasce do processo do cliente. Nada de dashboard-por-números.
2. **Entrega transferível.** Todo projeto deve poder ser entregue "as is": README que sobe o projeto do zero, migrations versionadas, zero dependência de conhecimento tribal.
3. **IA com propósito.** Determinístico onde dá (geometria, cálculo, regra de negócio); IA só onde há ambiguidade real (classificação, linguagem natural). IA nunca é a primeira ferramenta — é a última.
4. **Simplicidade que envelhece bem.** KISS/YAGNI. A pergunta antes de cada abstração: "isso resolve um problema que já existe?"
5. **Silêncio de maison.** Na interface e na comunicação: contenção, hierarquia, nada de efeito gratuito. Motion lento, pesado, scroll-driven; hover quase invisível.

## 2. Stack padrão

| Camada | Padrão | Quando divergir |
|---|---|---|
| Frontend | Vite + React + TypeScript + Tailwind + shadcn/ui | Projetos vindos do Lovable seguem o scaffold de origem (TanStack Start ⇒ `ssr: false` se hospedado como SPA) |
| Backend | Supabase (Postgres + RLS + Edge Functions) | APIs de terceiros pesadas ⇒ camada própria em edge function |
| Auth | Supabase Auth | Nunca rolar auth próprio |
| Deploy | Vercel (prebuilt) ou GitHub Pages | Ver playbooks internos de deploy |
| Runtime local | Bun | Node quando dependência exigir |
| Package manager | bun / npm — o que o lockfile do repo disser | Nunca misturar lockfiles |

## 3. Git

- **Branch padrão:** `main`.
- **Solo (interno):** commit direto na `main` é aceitável, desde que cada commit passe nos hooks e deixe a `main` deployável.
- **Projeto de cliente ou par:** feature branch curta + PR. Branches vivem dias, não semanas.
- **Commits convencionais:** `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`, `perf:`, `ci:`. Mensagem no imperativo, corpo explica o *porquê*.
- **E-mail de commit:** conta pessoal da Vertex (noreply do GitHub). Nunca e-mail de terceiros/empregador — o `includeIf` do gitconfig cuida disso para `~/code/vertex/` e `~/Vertex/`.
- **Hooks:** todo repo tem `lefthook.yml` com **gitleaks** no pre-commit. Sem exceção. (Atenção: `lefthook install` sobrescreve hook global — o lefthook.yml do repo deve incluir o gitleaks explicitamente.)

## 4. Segredos

Regra única: **segredo nunca encosta no repositório.**

- Chaves de app ⇒ Vault do Supabase (`app_secrets`) ou secrets do provedor de deploy.
- `.env` no `.gitignore` sempre; o repo versiona só `.env.example` com placeholders.
- Vazou? Rotacionar na hora, depois limpar histórico. Rotação vem primeiro.
- Detalhes e canal de reporte: [`SECURITY.md`](./SECURITY.md).

## 5. Qualidade

- **TypeScript estrito.** `strict: true`; `any` é dívida declarada em comentário.
- **Teste onde o erro custa caro:** lógica de negócio, transformação de dados, integrações. Cobertura é consequência, não meta de vaidade — mas módulos críticos (dinheiro, permissão, dados de cliente) não entram na `main` sem teste.
- **Arquivos ≤ ~400 linhas, funções ≤ ~50.** Passou disso, extrai módulo.
- **Imutabilidade por padrão.** Retorne cópias; mutação só com justificativa local.
- **Erros nunca são engolidos.** Toda falha ou é tratada, ou é logada com contexto e propagada.
- **Validação nas bordas:** toda entrada externa (usuário, API, webhook) é validada com schema (zod) antes de tocar a lógica.

## 6. Banco (Supabase)

- **RLS ligada em toda tabela** do schema `public`. Sem política = sem acesso, nunca o contrário.
- `SECURITY DEFINER` exige `REVOKE ... FROM PUBLIC` **e** revogação explícita de `anon`/`authenticated` (grant default do Supabase não é coberto pelo revoke de PUBLIC).
- Migrations versionadas no repo (`supabase/migrations/`). DDL manual em produção só em emergência — e vira migration retroativa no mesmo dia.
- PII nunca em tabela sem RLS; views que expõem dados sensíveis são tratadas como tabelas (RLS/grants auditados).

## 7. Fluxo com Lovable (projetos com gitsync)

1. Trabalhe no repo local: commit → push.
2. Aplique no Lovable via prompt/MCP **com `project_id` explícito** (o default do token quase sempre aponta pra outro projeto).
3. Edge functions e migrations: quem deploya é o ambiente Lovable/Supabase — confirme o deploy antes de considerar entregue.
4. Nunca editar pela UI do Lovable e pelo repo ao mesmo tempo sem reconciliar — o repo é a fonte da verdade.

## 8. Todo repo nasce com

Use o [`template-vertex`](https://github.com/vertexdevs-hq/template-vertex). O mínimo obrigatório:

- `README.md` — o que é, como rodar do zero, como deployar.
- `CLAUDE.md` — contexto para agentes de IA: stack, comandos, gotchas, o que **não** tocar.
- `lefthook.yml` — gitleaks no pre-commit.
- `.env.example` — todas as variáveis, valores placeholder.
- `.github/workflows/ci.yml` — typecheck + test + build (template `vertex-ci` da org).

## 9. Registro de pendências

Descobriu bug real, risco de segurança ou dívida no meio do trabalho? **Vira issue no repo na hora** — curta e acionável. Nada morre em contexto de sessão ou em cabeça de dev.

## 10. Fronteiras

- Esta org (`vertexdevs-hq`) é exclusivamente da **Vertex**. Trabalho de empregador/terceiros não entra aqui, e projetos Vertex não saem daqui (repos legados em `ghmbarboza` migram com o tempo).
- Apps de terceiros (Lovable gitsync etc.) são instalados **por repositório selecionado**, nunca "all repositories".
