# instatic-lp

Skills para agentes de IA (Claude Code, Codex, Cursor…) operarem um [Instatic](https://github.com/CoreBunch/Instatic) — CMS visual self-hosted — via MCP: conectar a instância e construir landing pages com design tokens, verificação visual e publicação controlada.

| Skill | O que faz |
|---|---|
| `instatic-setup` | Conecta o cliente ao MCP do Instatic (token pessoal ou OAuth), explica capabilities e resolve problemas comuns ("0 capabilities", "open the workspace", 401). |
| `instatic-lp` | Cria e edita landing pages: a partir de uma URL de referência, entrevista você (grill) sobre o que construir, oferece estilos (minimalist, soft, brutalist, redesign), monta a página em HTML semântico + CSS sobre o framework do site, verifica por breakpoint e publica só quando pedido. |

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

### Clientes com `mcpServers` em JSON (Claude Desktop, Cursor, Windsurf…)

Via ponte `mcp-remote` (precisa de Node/`npx`):

```json
"mcpServers": {
  "instatic": {
    "command": "npx",
    "args": [
      "-y",
      "mcp-remote@latest",
      "https://lp.suaempresa.com/_instatic/mcp",
      "--transport",
      "http-only",
      "--header",
      "Authorization:${INSTATIC_MCP_AUTH}"
    ],
    "env": {
      "INSTATIC_MCP_AUTH": "Bearer imcp_pat_..."
    }
  }
}
```

O `--header` fica sem espaço de propósito (alguns clientes quebram args em espaços); o `Bearer ` vai na variável de ambiente.

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

## Ferramentas disponíveis

Com todas as capabilities no token, o cliente lista **53 ferramentas**: contexto e mídia (2), Content — coleções e documentos (15) e Site — páginas, nós, CSS, tokens, code assets, inspeção visual e publicação (36). Menos que isso = token com menos capabilities. O mapa ferramenta → capability está em [`skills/instatic-lp/references/tools.md`](skills/instatic-lp/references/tools.md).

## Identidade visual por projeto

A skill `instatic-lp` usa sempre o **Framework** já configurado no Instatic (painel Framework → Colors, Type, Space), lido via `site_read_styles`: toda página nova sai com as mesmas cores, fontes e escalas de tipo/espaço, sem hex, `font-family` ou px soltos. Ela não altera o framework por conta própria (as ferramentas `site_set_*` mudam o site inteiro); só quando você pedir. Para fixar papéis (qual cor é o CTA, qual fonte é título) e tom de voz, copie [`skills/instatic-lp/references/brand-template.md`](skills/instatic-lp/references/brand-template.md) para a raiz do seu projeto como `instatic-brand.md`.

## Do link de referência à página

Mande uma URL (ou print) e peça a página. A skill:

1. **Lê a referência** e devolve uma leitura: seções, oferta, CTA, provas, linguagem visual, o que funciona e o que é fraco. Página do próprio Instatic é lida com `site_read_document`.
2. **Faz o grill**: uma pergunta por vez, sempre com a resposta recomendada, até fechar objetivo, público, oferta, plano de seções, copy, provas, imagens, formulário, slug e publicação. Se a referência for de terceiros, ela serve só de estrutura: a copy é reescrita e imagens/logos dela nunca entram.
3. **Oferece estilos** adaptados do [taste-skill](https://github.com/Leonxlnx/taste-skill), já traduzidos para o framework do site (cores e fontes continuam as do painel Framework):
   - **Minimalist**: editorial, monocromático, linhas finas, bento plano.
   - **Soft**: acabamento de agência, cards aninhados, CTA pílula, muito respiro, movimento fluido.
   - **Brutalist**: grade rígida, linhas visíveis, raio zero, tipografia gigante (Swiss print ou terminal).
   - **Redesign**: auditoria e melhorias numa página que já é sua, combinável com um dos estilos acima.
4. **Fecha um page spec** para você aprovar e só então constrói, verifica (incluindo o pre-flight do estilo) e publica se você pedir.

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
  instatic-lp/references/{tools.md,brand-template.md,reference-intake.md}
  instatic-lp/references/styles/{router.md,minimalist.md,soft.md,brutalist.md,redesign.md}
```

O catálogo de ferramentas em `references/tools.md` foi extraído do código do Instatic (`main` @ `f92e8dc`, 2026-09-13) e conferido contra uma instância real (53 ferramentas com todas as capabilities). Ao atualizar para uma versão nova do Instatic, revise esse arquivo.

## Licença

MIT — veja [LICENSE](LICENSE). Partes das orientações foram adaptadas dos prompts de sistema do Instatic (MIT) e do taste-skill (MIT) — veja [NOTICE](NOTICE).
