# claude-metin2

Plugin do Claude Code com o que aprendemos mantendo um servidor Metin2 em QuantumCore X (C#) e o client (Python 2,
C++): dois agentes de trabalho (servidor e client), armadilhas de ferramenta e de deploy, convenções para editar o
client, o padrão de um sistema novo que toca servidor e client, e o ciclo de publicação.

## O que tem

| Item | Para que serve |
|---|---|
| `agents/server` | Servidor QuantumCore X: sistemas, itens, drops, protos, balanceamento, migrações, deploy com portão |
| `agents/client` | Client: Python 2 do root, uiscript, C++ do exe, packs, troca de exe e patch |
| `skills/metin2-gotchas` | Armadilhas transversais: Windows + Git Bash, git compartilhado, deploy, teto int32, Redis/BigInteger, Latin-1 |
| `skills/metin2-client-convencoes` | Regras para editar o client: encoding, handler de comando, limites de vnum e nome |
| `skills/metin2-sistema-novo` | Padrão de sistema novo (gerador de tabela, regra em C#, comando servidor-client, UI) |
| `skills/metin2-prompt-menu-claude-design` | Da ideia de um menu ao prompt de arte para o Claude Design, conferência dos PNG e montagem |
| `skills/metin2-ciclo-publicacao` | Ordem dos geradores, client fechado, git sem `add -A`, portão de deploy, conferência e patch |
| `docs/quantumcore-servidor-de-teste.md` | Topologia, escolha da base e bugs do upstream de um servidor de teste |
| `docs/redis-biginteger-json.md` | Por que um `BigInteger` virou 0 no cache Redis e como evitar |
| `docs/build-do-client.md` | Compilar o exe do client e o que costuma ser alterado |

## Como criamos menus: da ideia ao Claude Design

Uma janela nova do client precisa de **arte** (painéis, botões, molduras de slot, ícones) e de **código** (o Python da
janela). O gargalo é a arte: se cada peça sai num estilo e num tamanho, a janela fica torta. Por isso a arte nasce de um
**prompt único e controlado**, e o código só usa o que a lista de peças prometeu. O fluxo, do começo ao fim:

1. **A ideia vira telas.** Descrevemos o menu em uma frase e depois em telas: quais abas ou submenus, quantos slots,
   quais botões e em que estados, quais ícones de item novos. Texto, números e contadores **não** entram na arte; o jogo
   os desenha.
2. **As medidas vêm do layout.** Medimos (ou desenhamos o croqui): tamanho de cada painel, de cada slot, de cada botão e
   aba. A arte precisa ter o **tamanho exato** que o código vai posicionar. Painel de tamanho fixo sempre que possível;
   9-slice só para janela que muda de tamanho.
3. **A IA escreve o prompt**, em três partes:
   - **Bloco 1, estilo fixo e regras técnicas:** paleta, materiais, direção da luz, mais as regras que o client exige
     (PNG RGBA, tamanho exato, nenhum texto, estados de botão `_n/_h/_p/_s/_d` com a **mesma silhueta**, sombra dentro do
     tamanho, moldura de slot com a área central 32x32 livre). É o **mesmo texto em todos os prompts** do projeto.
   - **Bloco 2, ícones de item 32x32:** vista, preenchimento, contorno, sem texto, famílias que parecem um conjunto.
   - **A lista de peças:** uma tabela com o **nome exato do arquivo**, o tamanho, os estados e uma descrição curta do que
     a imagem mostra.
4. **O usuário cola no Claude Design** (claude.ai), **em lotes**: primeiro o bloco 1, depois uma seção por vez (abas,
   fundos, peças comuns, ícones). Lotes pequenos mantêm o estilo consistente; se um destoar, regera-se só ele.
5. **Os PNG voltam numa pasta de assets.** Cada arquivo é conferido: nome, tamanho em pixels, transparência de verdade,
   ausência de texto, estados com a mesma silhueta, sombra dentro do tamanho. O que falha é pedido de novo.
6. **O empacotador converte** os PNG aprovados para o formato do client (TGA) e grava no caminho que o client espera.
7. **A janela é montada no Python do client** usando exatamente esses arquivos e as posições medidas no passo 2, e é
   testada no client.

A skill [`metin2-prompt-menu-claude-design`](plugins/metin2/skills/metin2-prompt-menu-claude-design/SKILL.md) conduz esse
fluxo, traz o template completo do prompt, o checklist de conferência e os erros mais comuns; um exemplo trabalhado está
em [`references/exemplo-aba-equipamento.md`](plugins/metin2/skills/metin2-prompt-menu-claude-design/references/exemplo-aba-equipamento.md).
O estilo do exemplo (dark fantasy) é só um modelo: **o estilo é do seu projeto** e deve ser o mesmo em todos os prompts.

## Instalação

```
claude plugin marketplace add AugustoSSR/claude-metin2
claude plugin install metin2@claude-metin2 --scope user
```

Os agentes ficam como `metin2:server` e `metin2:client`. As skills disparam pela descrição.

Mantenedor, antes de cada commit: `claude plugin validate .`

## Licença

[PolyForm Noncommercial 1.0.0](LICENSE): uso não comercial. Veja também [NOTICE.md](NOTICE.md).
