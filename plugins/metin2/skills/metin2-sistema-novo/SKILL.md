---
name: metin2-sistema-novo
description: 'Use para criar ou estender um sistema de Metin2 (QuantumCore X) que toca servidor E client: item, peça de equipamento, bônus, janela alimentada pelo servidor, dado por item ou por jogador. Padrão: gerador de tabela para protos, classe C# de regra, comando servidor-client em um token, handler e módulo de UI no client.'
license: PolyForm-Noncommercial-1.0.0
---

# Metin2: padrão de sistema novo (servidor + client)

Receita que funcionou para sistemas que tocam os dois lados (peças de equipamento, bônus, janelas). Os nomes de pastas
e de scripts abaixo são a **convenção do nosso fork**: adapte ao layout do seu projeto, mas mantenha a separação de
responsabilidades. Antes de escrever, **leia o plano da área** (um `PLANO-*.md` na raiz, se houver) e confira se já
existe sistema equivalente: estender é melhor que duplicar.

## Peças do padrão

| Peça | Onde (convenção) | Observação |
|---|---|---|
| Definição dos itens | `dados/protos/itens_<sistema>.tsv` (colunas vnum, base, nome, descrição, campos) | o gerador de protos lê o arquivo principal **e** todos os `itens_*.tsv`; vira o proto de item do servidor e do client |
| Definição do sistema (regras) | `dados/<sistema>.tsv` | gerado, com cabeçalho "não editar à mão"; o servidor lê de `data/` |
| Ícones | `dados/icones/<nome>.png` | use um banco de ícones; desenhar com PIL só sem alternativa |
| Textos e fórmulas do tooltip no client | `root/ui<sistema>.py` **gerado** pelo mesmo gerador | módulo novo exige o marcador de backup vazio (ver `metin2-client-convencoes`) |
| Regra no servidor | `PlayerUtils/<Sistema>.cs` (classe estática com a definição carregada do tsv) | |
| Dado por item | coluna de texto `chave=valor;chave=valor` (tamanho máximo fixo) | chaves curtas; um bônus pode ser `tipo*10000+valor` |
| Dado por jogador ou conta | tabela chave-valor (`escopo`, `chave`, `quantidade`) | chave e escopo com tamanho limitado; débito e crédito **atômicos** |
| Servidor → client | `player.SendChatCommand("Nome campo1\|campo2\|campo3")` | um token; ver regras 4 a 6 de `metin2-client-convencoes` |
| Client recebe | `def __Nome(self, dados)` no `game.py` + registro no dicionário de comandos | `dados.split("\|")`; valide com `isdigit()` |
| Arquivo de dados novo | tem de entrar na lista de arquivos enviados pelo script de deploy | senão o deploy sobe sem o tsv |

## Procedimento

1. **Classificar.** É item novo (só tsv e proto), regra de servidor (C#), janela ou tooltip (client), ou os três? Um item
   sem regra nova não precisa de C#.
2. **Escolher o vnum.** Leia a tabela de vnums do plano da área, reserve uma faixa livre e registre no plano. Visual de
   personagem: vnum abaixo de 65536 (16 bits). Nome do item com no máximo 23 bytes. Vnum de mob abaixo de 65536.
3. **Gerador** (`tools/gerar_<sistema>.py`): cabeçalho com autoria e modo de uso; define os dados em Python e **gera** os
   tsv de itens e do sistema, os ícones e o `ui<sistema>.py`. Substitua **só as linhas do próprio sistema**; não
   regrave um arquivo compartilhado inteiro (já apagou entradas de outro sistema). Se outro gerador regrava o mesmo
   arquivo, documente a ordem no cabeçalho dos dois.
4. **Servidor.** Classe de regra, handler de comando ou de pacote, texto de jogo no idioma do projeto. Estado
   persistente: prefira o dado por item ou por jogador a uma coluna nova; tabela nova exige migração EF. Grave **antes**
   de `DespawnEntity`. Nunca espere resposta do client dentro de um handler (trava a conexão): menus são
   fire-and-forget.
5. **Client.** `SendChatCommand` → handler no `game.py` → módulo `ui*.py`. Mensagem abaixo de ~1000 bytes (pagine).
6. **Build.** Compile o servidor e rode os testes; só commite com `0 Error(s)`.
7. **Documentar.** Atualize o plano da área (situação, o que ficou de fora) e o documento de visão geral se a decisão
   for estrutural.
8. **Publicar** com a skill `metin2-ciclo-publicacao` (ordem dos geradores, client fechado, portão de deploy).

## Pré-condição de segurança

Mudança de esquema de banco ou deploy afeta jogadores online: siga os portões de `metin2-ciclo-publicacao`. Esta skill
não autoriza nada em produção.

## Armadilhas

- O servidor e o client leem o **mesmo proto**: gere uma vez e distribua para os dois lados; protos divergentes dão
  item "fantasma" ou nome trocado.
- Tipo novo em estado de jogador (por exemplo `BigInteger`) quebra o cache Redis: ver `docs/redis-biginteger-json.md`.
- Sistema entregue sem teste em jogo está "deployado, não validado". Use um bot de teste e peça o teste humano antes de
  construir em cima.
