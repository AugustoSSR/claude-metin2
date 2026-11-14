# claude-metin2

Plugin do Claude Code com o que aprendemos mantendo um servidor Metin2 em QuantumCore X (C#) e o client (Python 2,
C++): dois agentes de trabalho (servidor e client), armadilhas de ferramenta e de deploy, convenções para editar o
client, o padrão de um sistema novo que toca servidor e client, e o ciclo de publicação.

**Estado: privado, em revisão.** Só vira público depois de uma revisão independente.

## O que tem

| Item | Para que serve |
|---|---|
| `agents/server` | Servidor QuantumCore X: sistemas, itens, drops, protos, balanceamento, migrações, deploy com portão |
| `agents/client` | Client: Python 2 do root, uiscript, C++ do exe, packs, troca de exe e patch |
| `skills/metin2-gotchas` | Armadilhas transversais: Windows + Git Bash, git compartilhado, deploy, teto int32, Redis/BigInteger, Latin-1 |
| `skills/metin2-client-convencoes` | Regras para editar o client: encoding, handler de comando, limites de vnum e nome |
| `skills/metin2-sistema-novo` | Padrão de sistema novo (gerador de tabela, regra em C#, comando servidor-client, UI) |
| `skills/metin2-ciclo-publicacao` | Ordem dos geradores, client fechado, git sem `add -A`, portão de deploy, conferência e patch |
| `docs/quantumcore-servidor-de-teste.md` | Topologia, escolha da base e bugs do upstream de um servidor de teste |
| `docs/redis-biginteger-json.md` | Por que um `BigInteger` virou 0 no cache Redis e como evitar |
| `docs/build-do-client.md` | Compilar o exe do client e o que costuma ser alterado |

## Instalação

```
claude plugin marketplace add AugustoSSR/claude-metin2
claude plugin install metin2@claude-metin2 --scope user
```

Os agentes ficam como `metin2:server` e `metin2:client`. As skills disparam pela descrição.

Mantenedor, antes de cada commit: `claude plugin validate .`

## Licença

[PolyForm Noncommercial 1.0.0](LICENSE): uso não comercial. Veja também [NOTICE.md](NOTICE.md).
