# claude-metin2

Plugin do Claude Code com o que aprendemos mantendo um servidor Metin2 em QuantumCore X (C#) e o client TMP4
(Python 2, C++): armadilhas de ferramenta e de deploy, e as convenções para editar o client.

**Estado: privado, em revisão.** Só vira público depois de uma revisão independente.

## O que tem

| Item | Para que serve |
|---|---|
| `skills/metin2-gotchas` | Armadilhas transversais: Windows + Git Bash, git compartilhado, deploy, teto int32, Redis/BigInteger, Latin-1 |
| `skills/metin2-client-convencoes` | Regras para editar o client: encoding, handler de comando, limites de vnum e nome |

## Instalação

```
claude plugin marketplace add AugustoSSR/claude-metin2
claude plugin install metin2@claude-metin2 --scope user
```

Mantenedor, antes de cada commit: `claude plugin validate .`

## Licença

[PolyForm Noncommercial 1.0.0](LICENSE): uso não comercial. Veja também [NOTICE.md](NOTICE.md).
