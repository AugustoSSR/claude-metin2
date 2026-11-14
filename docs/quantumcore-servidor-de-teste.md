# Servidor de teste com QuantumCore X

Notas de quem subiu um servidor Metin2 privado de teste com o **QuantumCore X** (reimplementação em C#/.NET, licença
MPL-2.0, [LICENSE do upstream](https://github.com/MeikelLP/quantum-core-x/blob/main/LICENSE), sem código da Ymir) e o evoluiu até um servidor com sistemas próprios. Registra decisões, a topologia e o que
quebrou.

## Escolha da base

- **QuantumCore X**: servidor em C#, MPL-2.0, sem código vazado. É a base usada aqui.
- **Não suba *serverfiles* de origem duvidosa** (pacotes vazados, sem licença). Além do risco legal, a base
  reimplementada é mais fácil de manter e documentar. Este repositório não indica onde obter nenhum client: use um ao
  qual você tem direito.
- Mantenha um **fork privado** do QuantumCore, com o upstream configurado como remoto, para poder puxar correções e
  abrir pull requests com os bugs que você achar.

## Topologia que funcionou

- Containers: **auth**, **game**, banco (MySQL 8.4) e cache (Valkey/Redis), cada um com os próprios volumes. As portas do
  jogo e da autenticação ficam acessíveis só para a rede que precisa. Banco e cache não têm porta publicada.
- O diretório de dados do jogo (`exp.csv`, lojas, `atlasinfo.txt`, tabelas) é um volume montado em `/app/data`. Como o
  `appsettings.json` também seria lido desse *content root*, configurações de mapa vão por variáveis de ambiente
  (`maps__N`).
- **IP anunciado ao client ≠ IP de bind.** No upstream, o client recebia o IP do container ao entrar no jogo. A solução
  foi uma opção de configuração do IP público anunciado (um projeto pode adicionar uma opção como `Hosting:PublicIpAddress`). Sem isso, o login
  "funciona" e a entrada no jogo falha.
- A imagem do Dockerfile do game precisa ser a do runtime **ASP.NET** (`aspnet`), não a `runtime`: com a segunda, o
  container caía com "No frameworks were found".
- Deploy a partir do git: configure uma *deploy key* somente leitura no repositório e desligue o CI do fork se não for
  usar.

## Bugs do upstream que aparecem cedo

Observados no upstream em out/2026, sem versão registrada e **não verificados contra o HEAD atual**; podem já estar corrigidos.

- **Personagem novo derruba a conexão**: o grupo de habilidade `UNKNOWN = 0` virava índice `-1`.
- **Classes femininas derrubam a conexão**: a classe com gênero era usada como índice direto.
- Arquivos ausentes no `data/` (`exp.csv`, lojas, `skilltable.txt`, `atlasinfo.txt`) geram *warnings* no log e não
  impedem o login, mas faltam funcionalidades.
- O gold e o drop dos monstros não saíam no estado inicial; o fator de chance de drop vinha com um valor de depuração
  enorme.
- O HP máximo só era calculado no login.

## Do servidor "vanilla aberto" a um servidor próprio

Padrões que se repetiram, em ordem de valor:

1. **Dados em tabelas de texto** (`tsv`/`csv`), com scripts geradores que produzem protos para servidor e client. Uma
   fonte única de verdade por sistema.
2. **Um plano escrito por área** (`PLANO-*.md`) e um documento de projeto com o estado real: decisões, infraestrutura,
   pendências.
3. **Gate de build**: só commitar com `0 Error(s)`.
4. **Bot de teste** (cliente sem tela): acha bug que um build verde não acha.
5. **Pacotes**: mapeie os pacotes do client contra o comportamento observado; todo pacote cliente→servidor leva um byte
   de sequência; fechar armazém ou loja de itens é feito por comando de chat; handlers que esperam resposta do client
   travam a conexão.
6. **Persistência**: DbContext curto por operação; grave antes de despawnar a entidade (o mapa faz `Dispose`); testes de
   banco **não** provam o cache (ver `redis-biginteger-json.md`).

## Perfil de CPU

Um servidor ocioso consumia ~80% de um núcleo. Era custo fixo do QuantumCore (temporizadores de 1 ms e 10 ms e *spin*
do threadpool), que não cresce com jogadores. Para perfilar: um container *sidecar* com o SDK (`sdk:10.0-alpine`),
`--pid container:<id>`, `TMPDIR=/proc/1/root/tmp/` e `dotnet-trace --format speedscope`.

## Lentidão por DbContext compartilhado

O repositório de itens usava o `DbContext` do personagem: um item novo ficava rastreado e a coleta automática
concorrente dava `MySqlConnection already in use`. A correção foi um `DbContext` curto por operação.

## Migrações EF sem banco local

Gere a migração num host com um banco temporário e `dotnet-ef`; se o runtime for mais novo que o SDK da ferramenta, use
`DOTNET_ROLL_FORWARD=Major`. `--output-dir` é relativo ao `--project`. Entregue o `.sql` e o comando a quem tem acesso
ao banco de produção; não faça `UPDATE` direto em produção por automação.
