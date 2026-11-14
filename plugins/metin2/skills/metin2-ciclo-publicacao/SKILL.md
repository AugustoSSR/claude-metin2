---
name: metin2-ciclo-publicacao
description: 'Use para gerar protos e packs de um projeto Metin2 (QuantumCore X) e colocar no ar: gerar protos, instalar conteúdo, copiar packs para o client, commit e push, deploy do servidor, patch do launcher e conferência. Ordem dos geradores, client fechado, git sem add -A, deploy que derruba jogador e verificação pós-deploy.'
license: PolyForm-Noncommercial-1.0.0
---

# Metin2: ciclo gerar → client → servidor → patch

Ciclo genérico. Os nomes de scripts e pastas são placeholders (`<...>`): troque pelos do seu projeto. O que **não** muda
é a ordem, os portões e a conferência.

Variáveis de exemplo (Git Bash): `R` = o repositório do servidor; `W` = pasta de trabalho temporária para saídas;
`C` = a pasta do client (cuidado com caminhos com espaço).

## Pré-condições (confira antes de qualquer passo que derrube ou publique)

1. **Deploy do servidor derruba quem está em jogo** (recria os containers de jogo e de autenticação). "Pode dar deploy"
   vale **para o deploy pedido**, não para os seguintes. Confirme com o responsável pelo projeto, **veja quantos
   jogadores estão online** e acumule mudança não urgente para a janela combinada. Se outra pessoa ou sessão mexe no
   mesmo servidor, avise e **espere a resposta** (avisar não basta: já saiu deploy duplo).
2. **Publicar o patch** (o publicador do launcher): combine antes com quem mantém site, launcher e criptografia dos
   packs; nunca publique ao mesmo tempo que essa pessoa.
3. O portão de produção em si (reiniciar serviço, mexer em infra) pertence a quem opera a infraestrutura; esta skill
   só descreve a ordem.

## Passos

1. **Gerar, em ordem.** A partir de `R`, rode só os geradores que o trabalho tocou, mas **respeite a ordem**. Regras:
   - Cada gerador documenta no cabeçalho quais arquivos regrava. Leia **antes** de rodar.
   - Se um gerador regrava um arquivo que outro também escreve, rode o dono do arquivo **primeiro** e o que só
     acrescenta **depois**.
   - Um gerador que redesenha ícones sobrescreve ícones escolhidos à mão: reaplique a escolha depois, ou restaure com
     `git checkout -- <ícones>`.
   - Se um gerador tem efeito colateral num arquivo compartilhado (por exemplo, a lista de drops), restaure-o:
     `git checkout -- <arquivo>`.
   - **O pack do `root` sai do gerador de protos, não do instalador de conteúdo.** O instalador de conteúdo vem
     **antes** (ele regrava arquivos e gera os ícones que o outro empacota). Se você mexeu em `root`, rode **os dois**,
     nesta ordem: instalador de conteúdo, depois gerador de protos. Uma versão já foi publicada sem a correção por
     causa disso.
2. **Client fechado.** Liste os processos pelo exe do client. `taskkill` costuma falhar com "Acesso negado" (o client
   roda como administrador): reconfira; se continuar aberto, **pare e peça para fechar**. Nunca copie pack com o client
   aberto.
3. **Copiar para o client.** Copie os packs gerados e o `Index` para a pasta `pack` do client. Se o `.exe` mudou (build
   C++), antes de sobrescrever guarde o anterior (`Metin2Distribute.exe.anterior`) e copie o novo de `binary\`. Se o
   client real não pode ser tocado, trabalhe numa cópia de sombra (scripts copiados e `pack` como *junction* criada
   pelo PowerShell).
4. **Git** (outra sessão pode editar o mesmo repositório): `git pull --rebase`; `git status --short`;
   `git add -- <seus caminhos>` — **nunca `git add -A`/`git add .`** (já levou trabalho alheio para produção); commit;
   `git push`. Se o build C# foi tocado, compile e **só commite com `0 Error(s)`**.
5. **Deploy do servidor** (só depois da pré-condição 1): rode o script de deploy do projeto com o alvo "servidor". Se
   criou um arquivo de dados novo, ele precisa estar na lista de arquivos enviados pelo script.
6. **Conferir o deploy** (não declare sucesso sem isto):
   - todos os containers esperados `Up` (jogo, autenticação, site, banco, cache, backup);
   - o endpoint de status responde com `online`, `auth`, `game` e `heartbeat` verdadeiros;
   - a porta do jogo responde;
   - leia o boot em ~10 s, sem ficar esperando mais.
7. **Publicar o patch** (pré-condição 2): rode o publicador **sem** as opções de launcher e de desenvolvimento
   (o launcher padrão já entra e os packs vão cifrados; a opção de desenvolvimento publica o formato cru; há uma opção
   "só gerar" para testar sem subir). Ele lê um arquivo de regras, ignora backups, troca o exe pelo de `binary\` e sobe
   só o que mudou. Mudou o texto de novidades? Edite o arquivo de novidades **antes**.
8. **Conferir o conteúdo publicado.** Liste o índice do pack gerado e o do pack cifrado e confira que o arquivo novo
   está lá e que o manifest tem a versão nova. Use `set -o pipefail` nos scripts: um `| tail -1` já escondeu o erro do
   gerador e saíram packs velhos.

## Armadilhas

- Heredoc do Git Bash perde `\\` e `\r\n`: script com barra invertida → ferramenta de escrita + `python arquivo.py`.
- Argumento que começa com `/` (comando de chat) vira caminho do Git: `MSYS_NO_PATHCONV=1`.
- Nunca apague nada na pasta do client; o original vira backup.
- Ícone novo: busque num banco de ícones antes de desenhar.
- Valide a saída de qualquer script que recebe vnum, pack ou caminho (contagem de linhas, conteúdo do pack) antes de
  declarar pronto.
