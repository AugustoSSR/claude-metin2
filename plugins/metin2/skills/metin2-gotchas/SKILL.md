---
name: metin2-gotchas
description: 'Use no início de qualquer trabalho em servidor Metin2 baseado em QuantumCore X (C#) ou em client TMP4, e antes de commitar, rodar gerador, trocar pack ou fazer deploy. Armadilhas transversais: ferramentas no Windows + Git Bash, git compartilhado, client aberto, deploy que derruba jogador, teto de 32 bits, Redis/BigInteger, encoding Latin-1.'
license: PolyForm-Noncommercial-1.0.0
---

# Metin2 (QuantumCore X + client TMP4): gotchas transversais

Só o que custou retrabalho real. Para as regras do client, veja `metin2-client-convencoes`.

## Ferramentas (Windows + Git Bash)

1. **Heredoc/string do Bash perde `\\` (vira `\`) e `\r\n`.** Já virou `ã` real dentro de `.py` do client e byte
   nulo em script. Script com barra invertida ou CRLF: ferramenta de escrita de arquivo + `python arquivo.py`, ou `chr(13)`.
2. **Argumento que começa com `/`** (comando de chat `/calendario`, mensagem de commit) vira `C:/Program Files/Git/...`:
   usar `MSYS_NO_PATHCONV=1`.
3. **`sed`/`awk` removem CRLF** de arquivos do client e do servidor: usar Python ou um editor de arquivo; detectar CRLF/LF por arquivo.
4. **Gravar arquivo:** calcular os bytes ANTES de abrir em `'wb'` (abrir trunca antes do `encode` falhar).
5. **Python no Windows** não entende `/c/...` (usar `C:\...`); `/tmp` vira `C:\tmp`. Script que importa módulo da própria
   pasta não roda com `python -I`.
6. **`mklink` no Git Bash falha:** junction via PowerShell `New-Item -ItemType Junction`.
7. **Antes de criar arquivo "novo", procurar pelo nome exato.** Se a gravação diz "updated", sobrescreveu algo: parar e
   recuperar do git. Nunca concluir "não existe" por `grep | head`.
8. **`set -o pipefail`** em script de publicação: um `| tail -1` escondeu erro do gerador de protos e saíram packs velhos.
9. **UPDATE direto no banco de produção** não se faz por automação: entregar o `.sql` e o comando para quem tem o acesso.
   Migração EF: gerar num host com banco temporário e `DOTNET_ROLL_FORWARD=Major` se o runtime for mais novo.

## Git compartilhado (várias sessões no mesmo repositório)

10. **Nunca `git add -A` / `git add .`:** já levou trabalho de outra sessão para produção. `git status --short` e
    `git add -- <meus caminhos>`. Arquivo sujo que não é meu fica quieto.
11. **Commit só no fim da sessão (ou na janela de deploy), nunca a cada ajuste.** Se o deploy compila o C# a partir do
    git, a ordem é: `build 0 Error(s)` -> commit -> `git pull --rebase --autostash` -> push -> conferir
    `git log origin/main -1` -> só então o script de deploy. Deploy antes do commit sobe código velho.
12. Memória de agente e arquivos de sessão **nunca** dentro do repositório do servidor.

## Client e deploy

13. **Client aberto = nada de trocar pack/exe.** `taskkill` pode falhar com "Acesso negado" (processo como admin):
    reconferir `tasklist`, senão pedir para fechar. Nunca apagar arquivo do client original: guardar como `.original-tmp4`.
14. **Deploy derruba quem está em jogo.** "Pode dar deploy" vale para o deploy pedido, não para os seguintes; conferir
    quantos jogadores estão online antes e acumular mudança não urgente para a janela combinada.
15. Separe os deploys (servidor, site, dados, bot): só o do servidor recria game/auth e derruba jogador. Confirme no
    script de deploy antes de supor.

## Servidor (C#, QuantumCore X)

16. **Teto int32:** HP/EXP/gold de mob e proto ficam em ~2,1 bi; acima disso vai por tabela de valores própria, não pelo proto.
17. **Tipo novo em `PlayerData`/`ItemInstance` (ex.: `BigInteger`) quebra o cache Redis** (o formatador grava
    `{"IsZero":true}` e lê 0): serializar como string (propriedade-texto + `[JsonIgnore]` para System.Text.Json e
    Newtonsoft) e conferir `redis-cli get player:<id>` depois de logar. Banco testado não prova o cache.
18. DbContext curto por operação (o do personagem descartado no logout não salva); `IServiceScopeFactory`, nunca guardar
    `IServiceProvider` do pacote; o mapa faz Dispose da entidade ao sair: gravar ANTES de `DespawnEntity`.
19. Esperar resposta do client dentro de handler trava a conexão (pacotes em sequência): menus fire-and-forget. Nunca
    mandar chat fora da fase GAME (na LOADING, antes do 0x71, o client trava).
20. Textos de pacote/proto em **Latin-1 (cp1252)**, nome de item/mob <= 23 bytes; mensagem servidor->client < ~1000 bytes.
21. Gerador que regrava arquivo de outro gerador duplica/zera linhas: ler o cabeçalho, rodar na ordem e restaurar com
    `git checkout -- <arquivo>` o que não era para mudar.

## Conteúdo

22. Ícone: ferramenta de banco de ícones primeiro; PIL só como último recurso.
23. Pillow grava DDS com linear size/mipmap errados -> textura preta: recolorir direto nos blocos DXT.

## Pedido ambíguo

24. **"Buff" = buff de habilidade** (aura da skill), não efeito de traje. Em pedido ambíguo de sistema (buff, efeito,
    aura) que vai mexer no exe, confirmar o alvo em uma linha ANTES de implementar e publicar.
25. Sistema "deployado, não testado": tratar como não validado e pedir o teste antes de construir em cima. Bot de teste
    (conta dedicada) acha bug que build verde não acha.
26. Script que recebe vnum, pack ou caminho: validar a saída (contagem de linhas, listar o conteúdo do pack) antes de
    declarar pronto.
