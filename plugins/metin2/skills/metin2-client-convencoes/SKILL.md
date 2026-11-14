---
name: metin2-client-convencoes
description: 'Use ANTES de editar o client Metin2 TMP4 (Python 2 do root, uiscript, locale, C++ do exe) ou criar comando servidor->client: encoding latin-1/CRLF, handler do game.py, limite de mensagem, .original-tmp4, limites de vnum/nome.'
license: PolyForm-Noncommercial-1.0.0
---

# Metin2 client TMP4: convenções e gotchas

Para criar ou diagnosticar uma janela (uiscript, classes `ui*.py`) use uma skill de UI dedicada, se houver; aqui só
o que vale para o client em geral. `<CLIENT_DIR>` é a raiz do seu client.

## Onde fica

- Python 2 do client: `<CLIENT_DIR>/<pacote>/root` e `<pacote>/uiscript/uiscript`. A pasta `locale/<idioma>/ui` tem
  **prioridade** sobre o uiscript (ex.: a barra de baixo costuma vir de `taskbar.py` no locale).
- C++ do exe: o projeto do client no Visual Studio; compilar com
  `MSBuild.exe client.sln /m /p:Configuration=Distribute /p:Platform=Win32`. Arquivo alterado leva `.original-tmp4`.
- Os `.py` entram no pack por um gerador de protos/packs do seu pipeline.

## Regras

1. **Nunca apagar arquivo do client original.** O original vira `<arquivo>.original-tmp4`.
2. **Módulo novo do root**: criar `<nome>.py.original-tmp4` **vazio** ao lado; sem isso o empacotador pode não incluir o
   módulo e o `import` derruba o client no login.
3. **Encoding**: o `.py` é ASCII; acento vira escape latin-1 (`\xe1`, `\xe3`...), nunca UTF-8 no arquivo. O `game.py` é
   ISO-8859 com **CRLF**. **Detectar CRLF/LF por arquivo** antes de editar e preservar (arquivo misto existe). Gravar
   com `bytes.decode('latin-1')`/`encode('latin-1')`; calcular os bytes **antes** de abrir o arquivo em `'wb'`.
4. **Handler de comando servidor->client no `game.py`**: `def __X(self, dados)`, **nunca `*dados`**. O
   `stringcommander.py` conta parâmetros por `co_argcount`; `*dados` cai no `__noarg_call__` e a janela recebe a
   chamada sem os dados. Registrar no dicionário de comandos (`"ItemDados" : self.__ItemDados`).
5. **O comando tem 1 token só** (split por espaço): o servidor manda `Nome a|b|c` (campos com `|`, itens com `;`).
   Espaço dentro do texto vira `_` e o client desfaz; **lista vazia manda `-`** (senão RuntimeError).
6. **Mensagem servidor->client < ~1000 bytes** (buffer de 1 KB). Paginar/limitar a lista.
7. **Limites de dados**: vnum de mob < 65536; partes do personagem (arma/corpo/cabelo) vão em **16 bits**, então vnum
   de item usado como visual < 65536; nome de item <= **23 bytes**; posições de refino/quickslot em WORD.
8. **Imagem >= 1000 px** vai como `.jpg`; fundos que o código escala apontam direto para o JPG.
9. **Antes de usar helper do `ui.py`, `grep "def X" ui.py`**: dependendo da base do client, `ui.WindowDestroy` e
   `SAFE_SetEvent` não existem; usá-los fecha o client ao logar (AttributeError no import). `SlotWindow.Set*Event` não
   aceita `*args`.
10. **Client aberto** bloqueia troca de pack/exe: fechar antes de publicar.

## Armadilhas do Bash no Windows (Git Bash)

- Heredoc perde barra invertida (`"\\xe3"` virou `ã` real dentro de `.py` do client) e `\r\n`: escrever o script com
  uma ferramenta de arquivo e rodar com `python`.
- Argumento começando com `/` (ex.: `/calendario`) é trocado por `C:/Program Files/Git/...`: usar `MSYS_NO_PATHCONV=1`.
- `sed`/`awk` removem CRLF de arquivos do client: usar Python ou um editor.

## Verificação rápida antes de entregar

`file <arquivo>` (latin-1/CRLF preservados); `grep -n "\*dados"` no `game.py` (deve ser vazio); `.original-tmp4`
presente para todo módulo novo; vnums e nomes dentro dos limites; boot do client sem `syserr.txt` novo.
