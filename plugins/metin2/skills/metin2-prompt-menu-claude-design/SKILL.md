---
name: metin2-prompt-menu-claude-design
description: 'Use quando o usuário pedir um menu, janela, arte de UI ou ícones para o client Metin2: escreve o prompt de arte (estilo fixo + regras técnicas + lista de peças com nome, tamanho e estados) para colar no Claude Design, e guia a conferência dos PNG, o empacotamento e a montagem da janela.'
license: PolyForm-Noncommercial-1.0.0
---

# Metin2: do menu ao prompt do Claude Design

Fluxo que usamos para criar a arte de uma janela nova: a ideia vira uma **lista exata de peças**, a lista vira um
**prompt**, o usuário cola o prompt no Claude Design (claude.ai) em lotes, os PNG voltam numa pasta, são conferidos,
empacotados e usados pela janela em Python. Esta skill cobre a parte do prompt e da conferência; o código da janela
fica com `metin2-client-convencoes` e o agente `metin2:client`.

## Quando usar

- "Quero um menu de X", "monta a janela de Y", "preciso da arte da aba Z", "gera os ícones dos itens".
- Antes de escrever qualquer `uiscript`/`ui*.py` que precise de imagem nova.
- **Não use** para modelos 3D, efeitos, trajes ou mapas (não são PNG de UI), nem para o código da janela.

## Regra de ouro

**O estilo é do projeto do usuário, não da skill.** O exemplo de estilo abaixo (dark fantasy) é só um modelo. Na
primeira vez, fixe o estilo com o usuário e guarde o **bloco 1** num arquivo do projeto (por exemplo
`PROMPT-UI-<assunto>.md`). **Todo prompt seguinte reutiliza o mesmo bloco 1, palavra por palavra.** Estilo reescrito a
cada vez gera um kit que não combina.

## Passo a passo

1. **Levantar a ideia e as telas.** Pergunte (ou deduza e confirme em uma linha): o que o menu faz, quais abas ou
   submenus, quantos slots, quais botões e estados, o que aparece de texto (o jogo desenha texto, a arte não), se há
   ícones de item novos. Faça uma lista de telas.
2. **Definir as medidas a partir do layout da janela.** Meça no `uiscript`/layout (ou desenhe o croqui com o usuário):
   tamanho do painel, posição e tamanho de cada slot, botão, aba. A arte tem de ter o **tamanho exato** do que o código
   vai posicionar. Prefira painel de **tamanho fixo**; use **9-slice** só para janela que muda de tamanho.
3. **Escrever o prompt em blocos** (template abaixo): bloco 1 (estilo + regras técnicas), bloco 2 (instruções de ícones
   32x32, só se houver ícone de item) e a **lista de peças** em tabelas (nome de arquivo exato, tamanho, estados,
   descrição curta). Salve num arquivo do projeto.
4. **Orientar a geração em lotes no Claude Design.** Diga ao usuário: cole o bloco 1; peça **uma seção por vez**
   (painéis, depois botões, depois slots, depois ícones); peça sempre "o nome exato do arquivo, PNG com transparência, no
   tamanho exato". Se um lote sair fora do estilo, regere só ele, citando uma peça boa do lote anterior como referência.
5. **Conferir os PNG recebidos** (checklist abaixo). Rejeite e peça de novo o que falhar; não "conserte" à mão uma arte
   que pode ser regenerada.
6. **Nomear, organizar e empacotar.** Coloque os arquivos na pasta de assets do projeto, um subdiretório por janela
   (`ui2/<janela>/<nome>.png`). O empacotador do projeto converte para o formato do client (TGA) e grava no caminho que o
   client espera. Rode a conferência de arquivos faltando, se o seu pipeline tiver uma.
7. **Montar a janela.** No Python do client, use **exatamente** os nomes e tamanhos da lista (`metin2-client-convencoes`).
   Estados de botão por sufixo (`_n`, `_h`, `_p`, `_s`, `_d`). Texto, números e contadores são desenhados pelo código.
   Teste no client: arte certa no PNG ainda pode ficar errada na tela (escala, ordem de camadas).

## Template do prompt

Salve em arquivo, com a lista de peças embaixo. O texto abaixo vai **em inglês**: geradores de imagem seguem melhor
instruções em inglês. O estilo do exemplo é genérico; troque pelo do seu projeto **uma vez** e mantenha.

### Bloco 1: estilo fixo + regras técnicas (cole sempre antes)

```
You are generating a complete UI asset kit for a Metin2-style MMORPG client (dark fantasy).
STYLE (all assets must match): aged dark leather and carved dark wood, thin antique gold trim with small engraved
corners, subtle deep-red accent gems, painted texture with soft inner shadow, warm torch-like lighting from the top-left.
PALETTE: wood #2a1a10 / #3b2414, leather #241710, gold #c9a25a / highlight #f0d27a / shadow #6b4a1f, red gem #b3202a,
teal accent #3fb7b0, text area dark #1a110b.
TECHNICAL RULES:
- PNG with transparent background (RGBA), EXACT pixel size given for each file, no padding outside the shape.
  Drop shadows and glows must stay INSIDE the stated size (never extend the canvas).
- NO text, NO letters, NO numbers anywhere (the game draws text on top), unless an item explicitly says otherwise.
- Crisp edges, readable at 100% size, no blur, no photo realism, no watermark.
- Button sets come in states with the SAME size and the SAME silhouette: _n (normal), _h (hover: brighter gold rim, soft
  glow), _p (pressed: darker, inset 1px), _s (selected: warm red inner glow + bright gold rim), _d (disabled:
  desaturated, dark). Only generate the states listed.
- Panel backgrounds are FIXED SIZE (not 9-slice) unless the item says "9-slice". For 9-slice: decorative corners,
  seamless repeatable edges, plain tileable center.
- Slot frames: a dark recessed square or circle where a 32x32 item icon sits, centered. The frame MUST leave the inner
  32x32 area plain and dark (no ornament inside it).
- Return every file with the EXACT file name listed (folder/name.png).
```

### Bloco 2: ícones de item 32x32 (cole depois do bloco 1, só se houver ícones)

```
Now generate ITEM ICONS for the same project, in the SAME art style as the UI kit above, as inventory icons:
- PNG RGBA, EXACTLY 32x32 pixels, transparent background.
- One clear object per icon, centered, filling about 80% of the square, strong silhouette readable at 32 px.
- Slightly top-down 3/4 view, rich but not noisy, 1 px dark outline.
- NO text, letters or numbers (the game draws "+N", counts and names).
- Items of the same family must look like a SET (same object; only color or tier changes).
- Return each file with the EXACT file name listed.
```

### Lista de peças (tabelas; em português, é para você e para o usuário)

| Arquivo | Tamanho | Estados | Descrição |
|---|---|---|---|
| `<janela>/fundo.png` | LxA | - | painel de tamanho fixo, descrição curta de como é e onde ficam os slots |
| `<janela>/aba_<nome>_{n,h,s}.png` | 24x24 | n h s | botão redondo com o símbolo X |
| `<janela>/slot_quadrado.png` | 36x36 | - | moldura de slot, interior 32x32 liso e escuro |

Regras da lista: um nome de arquivo por linha; `{n,h,p,s,d}` na linha significa **um arquivo por estado**; todo estado
listado tem o **mesmo tamanho e a mesma silhueta**; descreva o que o símbolo **é** ("espada cruzada com escudo"), nunca
texto. Agrupe em seções que cabem num lote (uma pasta ou um tipo de peça por vez). Um exemplo completo está em
`references/exemplo-aba-equipamento.md`.

## Checklist de conferência (para cada PNG recebido)

- [ ] Nome do arquivo **exatamente** o da lista.
- [ ] Tamanho em pixels **exato** (abra as propriedades ou leia com Pillow; "parecido" não serve).
- [ ] Fundo **transparente** de verdade (canal alfa; sem quadriculado desenhado, sem fundo branco ou preto).
- [ ] Nada de **texto, letra ou número** dentro da imagem.
- [ ] Estados do mesmo botão com **a mesma silhueta e o mesmo tamanho**; só cor e brilho mudam.
- [ ] Sombra e brilho **dentro** do tamanho (nada cortado nas bordas).
- [ ] Moldura de slot com a área central **32x32 lisa**.
- [ ] Painéis de tamanho fixo no tamanho do layout; 9-slice com centro liso e bordas repetíveis.
- [ ] Estilo igual ao do lote anterior (paleta, luz, material).
- [ ] Ícones 32x32: silhueta legível em tamanho real, sem texto, famílias parecendo um conjunto.

Verificação rápida de tamanho e alfa, para uma pasta (Python com Pillow):

```python
import sys, pathlib
from PIL import Image
for p in sorted(pathlib.Path(sys.argv[1]).rglob("*.png")):
    im = Image.open(p)
    print(p.name, im.size, im.mode, "alfa" if im.mode == "RGBA" else "SEM ALFA")
```

## Erros comuns

- **Texto dentro da imagem.** O gerador adora escrever o nome do botão. O jogo desenha o texto por cima: a arte com
  letra fica duplicada e não traduz. Reforce "NO text" no prompt e rejeite.
- **Tamanho diferente entre estados.** `_h` com 2 px a mais faz o botão "pular" ao passar o mouse. Rejeite o conjunto.
- **Sombra ou brilho fora do tamanho.** O canvas cresce e o botão fica deslocado; exija a sombra dentro do tamanho.
- **Fundo não transparente.** Fundo branco ou preto vira retângulo na tela. Confira o alfa.
- **Gerar tudo de uma vez.** Lotes grandes perdem a consistência de estilo. Um tipo de peça por lote, sempre com o
  bloco 1 colado antes.
- **Estilo reescrito a cada prompt.** Use sempre o mesmo bloco 1.
- **Inventar medida.** Se a arte não bate com o layout do código, a janela sai torta. Meça antes de pedir.
- **Moldura de slot com ornamento no centro.** O ícone do item some embaixo dele.
- **Misturar ícone de item na lista de interface.** Ícones 32x32 têm bloco 2 e pasta próprios.
