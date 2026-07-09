# Política de Segurança — Vertex Devs

## Reportando uma vulnerabilidade

Encontrou uma falha em qualquer projeto desta organização?

- **E-mail:** guilherme.barboza@vertexdevs.org
- Ou abra um **Security Advisory** privado no próprio repositório (aba Security → Report a vulnerability).

Não abra issue pública com detalhes exploráveis. Resposta em até 48h úteis.

## Política de segredos

- Nenhum segredo (API key, token, senha, connection string) é versionado — em código, config ou histórico.
- Segredos de aplicação vivem no Vault do Supabase (`app_secrets`) ou nos secrets do provedor de deploy.
- Todo repo roda **gitleaks** no pre-commit via lefthook.
- Segredo exposto = **rotacionar primeiro**, limpar histórico depois, registrar o incidente em issue privada.

## Padrões mínimos dos projetos

- Supabase: RLS ligada em toda tabela; `SECURITY DEFINER` com grants explicitamente revogados de `anon`/`authenticated`.
- Toda entrada externa validada com schema antes da lógica.
- Edge functions públicas (`verify_jwt = false`) só com verificação própria de assinatura/token.
- Dependências auditadas quando o lockfile muda (`bun audit` / `npm audit`).
