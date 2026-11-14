# QuantumCore X: tipo novo e o cache Redis (caso do BigInteger)

## O que aconteceu

No QuantumCore X o personagem passa por um cache Redis (formatador JSON do cliente Redis usado pelo projeto, banco 1,
chave `player:<id>`). Ao migrar o ouro do jogador para `BigInteger`, o valor era gravado no cache como um objeto
(`{"IsPowerOfTwo":..,"IsZero":true,..}`) e lido de volta como **0**: o ouro não subia nem salvava, e depois de um
reinício o banco ficou com 0.

O banco havia sido testado (a coluna virou `varchar(41)` com `HasConversion`), mas o **cache não**: o defeito só
apareceu em jogo.

## Regra

Ao mudar o tipo de **qualquer** campo de `PlayerData` ou de `ItemInstance`, confira o cache depois de logar:

```
docker exec <cache> redis-cli -n 1 get player:<id>
```

Para tipos não primitivos, **serialize como string**: uma propriedade de texto persistida (por exemplo `GoldTexto`) mais
a propriedade tipada marcada com `[JsonIgnore]`, para os **dois** serializadores em jogo (System.Text.Json e
Newtonsoft). Depois de ajustar, logue, ganhe ouro, deslogue, logue de novo e confira o banco e o cache.

## Por que enganou

- O teste de banco passou, e o formatador do cache é outro caminho de código.
- O sintoma (ouro que "não sobe") parecia bug de lógica de ganho, não de serialização.
- Um restart gravava o 0 do cache no banco, apagando o valor real.

## Teto de 32 bits relacionado

HP, EXP e ouro de mob e de proto ficam em ~2,1 bilhões (`int32`). Valores acima disso devem vir de uma tabela própria
carregada pelo servidor, não do proto; scripts de balanceamento que gravam o proto podem cortar no teto.
