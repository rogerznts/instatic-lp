# instatic-lp

Skills para agentes de IA (Claude Code, Codex, Cursor…) operarem um [Instatic](https://github.com/CoreBunch/Instatic) — CMS visual self-hosted — via MCP: conectar a instância e construir landing pages com design tokens, verificação visual e publicação controlada.

| Skill | O que faz |
|---|---|
| `instatic-setup` | Conecta o cliente ao MCP do Instatic (token pessoal ou OAuth), explica capabilities e resolve problemas comuns ("0 capabilities", "open the workspace", 401). |
| `instatic-lp` | Cria e edita landing pages: orientação, design tokens, seções em HTML semântico + CSS, imagens, verificação por breakpoint e publicação só quando pedida. |

## Pré-requisitos

1. Uma instância Instatic acessível (ex.: `https://lp.suaempresa.com`).
2. Um token de acesso MCP: no Instatic, **AI → MCP → Create access token** (guarde o `imcp_pat_…`, ele aparece uma vez só).
3. Para editar páginas, o **editor do Instatic aberto no navegador**, logado com o mesmo usuário dono do token. Leituras e publicação funcionam sem ele.

## Instalação

### Claude Code (plugin)

```text
/plugin marketplace add rogerznts/instatic-lp
/plugin install instatic@instatic-lp
```

As skills ficam disponíveis como `/instatic:instatic-setup` e `/instatic:instatic-lp`, e também são acionadas automaticamente quando você pede algo como "cria uma LP no instatic".

Depois, registre o MCP da sua instância (uma vez, escopo de usuário):

```sh
claude mcp add instatic --transport http --scope user \
  https://lp.suaempresa.com/_instatic/mcp \
  --header "Authorization: Bearer imcp_pat_..."
```

### Qualquer agente compatível com Agent Skills (skills.sh)

```sh
npx skills add rogerznts/instatic-lp
```

### Manual (Codex e outros)

```sh
git clone https://github.com/rogerznts/instatic-lp
cp -r instatic-lp/skills/* ~/.codex/skills/     # Codex
cp -r instatic-lp/skills/* ~/.claude/skills/    # Claude Code sem plugin
```

## Identidade visual por projeto

A skill `instatic-lp` é genérica: ela lê os tokens que já existem no site. Para fixar a marca, copie [`skills/instatic-lp/references/brand-template.md`](skills/instatic-lp/references/brand-template.md) para a raiz do seu projeto como `instatic-brand.md` e preencha cores, fontes, classes e tom de voz.

## Desenvolvimento

```sh
claude plugin validate .          # valida manifests e skills
claude --plugin-dir .             # testa localmente sem instalar
```

Estrutura:

```text
.claude-plugin/
  plugin.json          # manifest do plugin "instatic"
  marketplace.json     # este repo como marketplace
skills/
  instatic-setup/SKILL.md
  instatic-lp/SKILL.md
  instatic-lp/references/{tools.md,brand-template.md}
```

O catálogo de ferramentas em `references/tools.md` foi extraído do código do Instatic (`main` @ `f92e8dc`, 2026-09-13). Ao atualizar para uma versão nova do Instatic, revise esse arquivo.

## Licença

MIT — veja [LICENSE](LICENSE). Partes das orientações foram adaptadas dos prompts de sistema do Instatic (MIT) — veja [NOTICE](NOTICE).
