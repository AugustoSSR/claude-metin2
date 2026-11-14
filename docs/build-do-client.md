# Build do client Metin2 (C++ do exe, Python 2 e packs)

Resumo de como compilar e substituir o executável do client e o que costuma ser alterado no C++. Vale para quem tem o
código-fonte do client **por meios legítimos**; este repositório não contém nem indica onde obtê-lo.

## Compilar o exe

- Projeto: `client.sln`, **Visual Studio 2022 (toolset v143)**, plataforma `Win32`, configuração `Distribute`. A
  carga de trabalho "Desenvolvimento para desktop com C++" basta (a edição Community serve).
- Linha de comando:

  ```
  MSBuild.exe client.sln /m /p:Configuration=Distribute /p:Platform=Win32
  ```

  A saída é `binary\<exe>`. Copie para a pasta do client **com o client fechado**, depois de guardar o exe anterior como
  backup.
- O projeto do exe costuma ficar **fora** do git do servidor. Guarde as suas mudanças em outro repositório ou em backup,
  e mantenha uma cópia `.original` de cada arquivo alterado.

## Mudanças que valem registrar

- **Páginas de inventário**: aumentar `c_Inventory_Page_Count` (por exemplo, para 10) desloca as posições de
  equipamento (equipamento em `450 + slot`); revise tudo que usa o índice absoluto.
- **Posição de refino e `TQuickSlot.Position` em WORD**: o formato original não comporta posições maiores.
- **Efeito preso a arma** (aura ou brilho): registre o efeito na tabela de efeitos do módulo de configuração do
  jogador (`playersettingmodule`), e não só no arquivo `.msm`.
- **Brilho de armadura com traje**: a armadura real precisa vir na parte `HEAD` do pacote para que o brilho funcione
  com um traje por cima.
- Limites herdados do formato: vnum visual de personagem em 16 bits (< 65536), `c_Wear_Max` e tamanhos de buffer de 1 KB
  nas mensagens servidor→client.

## Ícones

- O **ícone do exe, da janela, da barra de tarefas e do Alt-Tab** é o `.ico` do projeto C++ (carregado em
  `PythonApplication.cpp` com `LoadIcon(GetInstance(), MAKEINTRESOURCE(IDI_METIN2))`; o `.rc` associa
  `IDI_METIN2 ICON "metin2.ico"`). Trocar a "logo do client" é trocar esse `.ico` (gere com Pillow, tamanhos
  `16, 24, 32, 48, 64, 128, 256`, 32 bits) **e recompilar o exe**.
- A **logo da tela de login** é outra coisa: uma textura (`.tga`) empacotada num pack de UI.

## Python 2 e packs

- O `root` Python 2.7 vai num pack gerado pelo seu pipeline; um módulo novo exige o marcador de backup vazio que o
  empacotador espera (ver `metin2-client-convencoes`).
- O client só lê objetos (`.prb`, `.prt`, `.pre`, `.pra`) do pack `property`; mapas e objetos indoor (blocos tipo 2)
  exigem um empacotador atualizado.
- A ordem entre instalar conteúdo e gerar protos importa: ver `metin2-ciclo-publicacao`.

## Armadilhas

- Heredoc do Git Bash come `\r\n` e barras invertidas: use ferramenta de escrita de arquivo ou `chr(13)`/`chr(92)`.
- Client aberto bloqueia a troca de exe e de packs: confirme pela lista de processos.
- Um build que compila não prova que o client abre: teste o login e leia o `syserr.txt`.
