# Contribuindo — Vertex Devs

O manual completo está em [`ENGINEERING.md`](./ENGINEERING.md). Este é o resumo operacional.

## Fluxo

1. **Issue primeiro** para qualquer trabalho não-trivial — curta e acionável.
2. **Branch:** `feat/nome-curto`, `fix/nome-curto` (solo em repo interno pode ir direto na `main`, desde que deployável).
3. **Commits convencionais:**

   ```
   feat: adiciona filtro de período no funil
   fix: corrige fuso na agregação diária
   refactor: extrai cálculo de margem para lib/
   ```

   Imperativo, minúsculo, sem ponto final. Corpo explica o *porquê* quando não for óbvio.

4. **Antes do push:** os hooks do repo (lefthook + gitleaks) precisam passar. Não contornar com `--no-verify`.
5. **PR:** preencha o template. PR pequeno revisa rápido; acima de ~400 linhas, quebre.

## Checklist de qualidade (antes de marcar pronto)

- [ ] TypeScript sem erro (`tsc --noEmit` ou o check do repo)
- [ ] Testes do módulo tocado passam; lógica crítica nova tem teste
- [ ] Nenhum segredo, chave ou URL interna no diff
- [ ] Migrations versionadas (se tocou banco) e RLS revisada
- [ ] README/CLAUDE.md atualizados se o setup mudou

## O que não entra

- Código com segredo hardcoded (o CI e o gitleaks bloqueiam — mas não deixe chegar neles)
- Dependência nova sem justificativa no PR
- `any` sem comentário de dívida
- Tabela nova sem RLS
