# Gerador de Fase — Puzzle Numérico
 
## Contexto
 
Dado um conjunto de parâmetros de configuração, o algoritmo deve gerar uma equação válida, preencher estruturas de dados (blocos) com valores que garantam pelo menos uma solução, e derivar metadados de resultado a partir dessa equação. O restante dos valores é preenchido aleatoriamente, respeitando as restrições definidas.
 
A equação segue avaliação **estritamente da esquerda para a direita**, sem precedência de operadores.
 
> Exemplo: `3 + 4 * 2 = 14`
 
Divisões que resultem em decimais seguem arredondamento padrão: `>= 0.5` arredonda para cima, `< 0.5` arredonda para baixo.
 
---
 
## Inputs
 
| Parâmetro | Descrição | Restrição |
|---|---|---|
| `N` | Número de blocos | `N >= 2` |
| `F` | Número de faces por bloco | `F >= 1` |
| `S` | Número de slots numéricos na equação | `S >= 2` |
| `O` | Número de slots de operadores na equação | `O >= 1` |
| `P` | Peso de dificuldade alvo da fase | `1.0 <= P <= 4.0` |
 
Operadores fixos e seus pesos:
 
| Operador | Peso |
|---|---|
| `+` | 1 |
| `-` | 2 |
| `*` | 3 |
| `/` | 4 |
 
---
 
## Estruturas
 
### Bloco
- Contém `F` faces, cada uma com um valor inteiro de `0` a `9`
- Valores não se repetem dentro do mesmo bloco
- Valores podem se repetir entre blocos distintos
### Slot Numérico
- Pode ser ocupado por **1 bloco** (valor simples, `0-9`)
- Ou por **2 blocos adjacentes** formando um composto de 2 dígitos (`10-99`)
### Equação Gerada
```
[slot_num_1] [op_1] [slot_num_2] [op_2] ... [slot_num_S] = R
```
 
---
 
## Processo
 
### 1. Geração da Equação
- O algoritmo seleciona operadores e valores compatíveis com `P` alvo
- Avalia a equação da esquerda para a direita produzindo `R`
- Calcula o `peso_real` da equação gerada
- Se `peso_real` estiver fora do range `[P - 0.5, P + 0.5]`, descarta e tenta novamente
### 2. Cálculo do peso_real
 
```
Rmax = maior R possível dado S, O e os valores disponíveis (0-99)
 
media_operadores = média dos pesos dos operadores usados na equação
 
peso_valores:
  todos os slots simples (0-9)      → 1
  mix de simples e composto         → 2
  todos os slots compostos (10-99)  → 4
 
faixa_R = 1 + (R / Rmax) * 3
 
peso_real = (media_operadores + peso_valores + faixa_R) / 3
```
 
### 3. Preenchimento dos Blocos
- Os valores da equação garantida são plantados nas faces dos `S` blocos correspondentes
- As faces restantes são preenchidas aleatoriamente com valores de `0` a `9`, sem repetição no mesmo bloco
### 4. Geração do RMe e RMa
- O algoritmo varre todas as combinações possíveis de faces e operadores disponíveis
- Encontra `Rmin` — menor resultado real alcançável
- Encontra `Rmax_real` — maior resultado real alcançável
- Sorteia `RMe` no intervalo `[Rmin, R / 3]`
- Sorteia `RMa` no intervalo `[R + (R / 3), Rmax_real]`
- Se o intervalo for vazio, o respectivo retorno não é gerado
---
 
## Outputs
 
| Saída | Descrição |
|---|---|
| `R` | Resultado da equação gerada |
| `P` | Lista de operadores disponíveis, derivados do peso alvo |
| `B` | Lista de `N` arrays, cada um com `F` faces geradas |
| `RMe` | Valor sorteado abaixo de `R / 3`, com solução alcançável garantida |
| `RMa` | Valor sorteado acima de `R + (R / 3)`, com solução alcançável garantida |
| `equação` | Solução garantida — uso interno, não exposta ao jogador |
| `peso_real` | Peso efetivo da equação gerada |
 
---
 
## Restrições
 
- `N >= S` — deve haver blocos suficientes para preencher todos os slots numéricos
- Compostos de 2 dígitos ocupam exatamente 2 blocos adjacentes, produzindo valores de `10` a `99`
- `peso_real` deve estar no range `[P - 0.5, P + 0.5]`, caso contrário a equação é descartada e regerada
- `RMe` nunca pode ser menor que `Rmin`
- `RMa` nunca pode ser maior que `Rmax_real`
---
 
## Exemplo
 
**Config:**
```
N = 3, F = 4, S = 2, O = 1, P = 2.5
```
 
**Equação gerada:** `7 * 4 = 28`
 
**Cálculo do peso_real:**
```
Rmax             = 9 * 9 = 81
media_operadores = 3   (* = peso 3)
peso_valores     = 1   (todos simples)
faixa_R          = 1 + (28 / 81) * 3 = 2.03
peso_real        = (3 + 1 + 2.03) / 3 = 2.01  ✓ dentro do range [2.0, 3.0]
```
 
**Blocos gerados:**
```
Bloco 1: [7, 3, 1, 5]   <- 7 plantado, resto aleatório
Bloco 2: [4, 9, 2, 6]   <- 4 plantado, resto aleatório
Bloco 3: [8, 0, 5, 3]   <- apenas aleatório
```
 
**RMe e RMa:**
```
Rmin       = 0   (ex: 0 * 4)
Rmax_real  = 63  (ex: 9 * 7, iterando combinações)
 
RMe sorteado em [0, 9]    -> ex: 5
RMa sorteado em [37, 63]  -> ex: 48
```
