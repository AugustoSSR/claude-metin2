# Exemplo trabalhado: aba "Equipamento" com submenus

Exemplo genérico de como ficam a ideia, as medidas e a lista de peças de uma aba com submenus. Nomes, tamanhos e
descrições são ilustrativos: troque pelos do seu projeto. O estilo do **bloco 1** não aparece aqui; ele vem do
`SKILL.md` e é colado antes de cada lote.

## 1. A ideia

Uma janela de personagem com uma coluna de **abas redondas** à esquerda. Cada aba abre um submenu com um painel de
tamanho fixo e slots:

| Submenu | Painel | Slots |
|---|---|---|
| Equipamento | silhueta do personagem | cabeça, arma, roupa e uma coluna de 4 extras à direita |
| Anéis | grade | 6 nichos (2 colunas x 3 linhas) |
| Amuletos | grade | 9 nichos (3x3) |
| Relíquias | moldura de pedra | 6 nichos alongados em forma de gema |
| Livros | estante | 9 nichos redondos (3x3) e uma faixa de título no topo |

Texto (nome do submenu, contadores, "+N") **não** entra na arte: o código desenha.

## 2. As medidas (do layout da janela)

Antes de pedir a arte, meça no layout: o painel de cada submenu tem **156x188 px** (fixo), as abas **24x24**, os slots
**36x36** (interior de 32x32 para o ícone), o cadeado **14x16**, o ícone do olho **14x10**. Os 9 nichos da grade
3x3 ficam alinhados ao grid do código: peça ao gerador a posição aproximada e **ajuste o código** à arte entregue, ou o
contrário, mas registre as posições finais na lista.

## 3. A lista de peças (colada depois do bloco 1)

### 3.1 Abas (ícones em coluna, à esquerda do painel)

| Arquivo | Tamanho | Estados | Descrição |
|---|---|---|---|
| equip/aba_equipamento_{n,h,s}.png | 24x24 | n h s | botão redondo com uma espada e um escudo dourados |
| equip/aba_aneis_{n,h,s}.png | 24x24 | n h s | dois anéis entrelaçados |
| equip/aba_amuletos_{n,h,s,d}.png | 24x24 | n h s d | amuleto pendurado com gema vermelha |
| equip/aba_reliquias_{n,h,s,d}.png | 24x24 | n h s d | cálice de pedra com runas |
| equip/aba_livros_{n,h,s,d}.png | 24x24 | n h s d | livro fechado com selo dourado |
| equip/cadeado.png | 14x16 | - | cadeado dourado pequeno, vai por cima de uma aba bloqueada |

### 3.2 Fundos dos painéis (156x188, tamanho fixo)

| Arquivo | Descrição |
|---|---|
| equip/fundo_equipamento.png | painel de couro escuro com a silhueta clara de um guerreiro no centro; espaços planos para cabeça (topo centro), arma (esquerda), roupa (centro) e uma coluna de 4 extras à direita |
| equip/fundo_aneis.png | painel de couro com 6 nichos quadrados em 2 colunas x 3 linhas |
| equip/fundo_amuletos.png | painel de couro com fios dourados ligando 9 nichos em grade 3x3 |
| equip/fundo_reliquias.png | moldura de pedra escura com runas e 6 nichos alongados |
| equip/fundo_livros.png | estante ornamentada com fio azul-turquesa, 9 nichos redondos maiores (3x3) e uma faixa vazia no topo para o título |

### 3.3 Peças comuns

| Arquivo | Tamanho | Estados | Descrição |
|---|---|---|---|
| equip/slot_quadrado.png | 36x36 | - | moldura de slot quadrada dourada, interior escuro, **área central 32x32 lisa** |
| equip/slot_redondo.png | 36x36 | - | moldura de slot redonda dourada, interior escuro, **área central 32x32 lisa** |
| equip/slot_gema.png | 30x40 | - | nicho em forma de gema alongada |
| equip/slot_bloqueado.png | 36x36 | - | slot escurecido com uma corrente cruzada |
| equip/olho_{n,h}.png | 14x10 | n h | olho aberto pequeno (mostrar) |
| equip/olho_fechado_{n,h}.png | 14x10 | n h | olho riscado (ocultar) |
| equip/barra_fundo.png | 120x8 | - | barra vazia |
| equip/barra_cheia.png | 120x8 | - | preenchimento azul-turquesa brilhante |
| equip/raridade_comum.png | 34x34 | - | borda fina cinza (vai por cima do ícone) |
| equip/raridade_rara.png | 34x34 | - | borda fina azul |
| equip/raridade_epica.png | 34x34 | - | borda fina roxa |
| equip/raridade_lendaria.png | 34x34 | - | borda fina laranja-dourada |

### 3.4 Ícones de item (bloco 2; 32x32; pasta `icones/`)

| Arquivo | Descrição |
|---|---|
| icones/anel_prata.png | anel de prata simples com pequena gema azul |
| icones/anel_ouro.png | o mesmo anel em ouro com gema vermelha (mesma família, só muda o material) |
| icones/amuleto_runico.png | amuleto de pedra com uma runa |
| icones/livro_chamas.png | livro com uma chama na capa |
| icones/livro_gelo.png | o mesmo livro com um cristal de gelo na capa |

## 4. Como pedir em lotes

1. Cole o **bloco 1**. Peça só a seção 3.1 (abas). Confira (checklist do `SKILL.md`).
2. Mesma conversa: peça a 3.2 (fundos). Confira o tamanho exato 156x188 e o alfa.
3. Peça a 3.3 (peças comuns). Atenção às molduras de slot (centro liso) e aos pares de estados.
4. Cole o **bloco 2** e peça a 3.4 (ícones).
5. Se um lote destoar, regere só esse lote, apontando uma peça boa do anterior como referência de estilo.

## 5. Depois de receber os PNG

- Pasta de assets: `ui2/equip/<nome>.png` e `icones/<nome>.png`.
- O empacotador converte para TGA e grava em `<client>/ui/ui2/equip/<nome>.tga`.
- Na janela, cada aba é um botão com os estados `_n`/`_h`/`_s`/`_d`; cada submenu mostra o `fundo_*` correspondente e
  posiciona os slots nas coordenadas medidas no passo 2. O cadeado e as bordas de raridade são imagens por cima do
  slot; as barras usam duas imagens (fundo e cheia) com largura ajustada pelo código.
