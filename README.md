# KRAT — Linguagem Esotérica inspirada em *Lies of P*

**Extensão:** `.krat`  **Tradutor:** `krat2py.py` (Krat → Python 3)  **Testes:** `testes.py`

## 1. Propósito e ideia central

Krat é uma linguagem de propósito específico, criada para resolver os problemas do Projeto 2
(fatorial, Fibonacci, Torre de Hanói, ordenação e CRUD em JSON). O programa é a jornada de Pinóquio,
que **ACORDA** no Hotel Krat com uma reserva de energia e **DORME** ao terminar.

A identidade da linguagem está em **três mecânicas que mudam a semântica de execução**:

| Mecânica | Ideia | Efeito em tempo de execução |
|---|---|---|
| **Ergo** | energia computacional | laços, artes e acessos ao Stargazer custam Ergo; sem Ergo o programa desmorona |
| **Stargazer** | memória persistente de Pinóquio | lista de itens gravada em um arquivo JSON (array) pelo próprio runtime |
| **Humanidade** | informações com estado e confiança | um **Fato** incerto ou pouco confiável não decide nada: o programa é interrompido |

## 2. Tipos

| Tipo | Descrição | Exemplo |
|---|---|---|
| **Ergo** | inteiro; também é a unidade de energia (a divisão `/` é inteira) | `42` |
| **Fable** | texto | `"Hotel Krat"` |
| **Legião** | lista dinâmica (índice a partir de 0) | `LEGIAO v` |
| **Estado** | `VERDADE`, `MENTIRA` ou `INCERTO` (três valores) | `INCERTO` |
| **Fato** | um Estado + uma confiança de 0 a 100 | `FATO(VERDADE, 90)` |

Comparações (`==`, `<`, …) produzem `VERDADE`/`MENTIRA` simples, equivalentes a um Fato com 100% de confiança.
Nomes de variáveis começam com minúscula ou `_`; palavras com inicial maiúscula são reservadas.
Variáveis criadas dentro de uma `ARTE` são locais (globais podem ser lidas); listas são passadas por referência.

## 3. Mecânica 1 — Ergo (energia)

Todo programa tem uma reserva de Ergo. Padrão: **10000**. Para mudar: `ACORDAR 500`.

| Operação | Custo |
|---|---|
| cada execução do corpo de `PERSISTIR`, `CICLO`, `RECUAR` | 1 |
| cada chamada de uma `ARTE` | 1 |
| `ACENDER`, `CONSULTAR(c)` | 1 |
| `REGISTRAR`, `ATUALIZAR`, `EXCLUIR` | 2 |
| `REFORCAR(f, n)` | n |
| demais comandos (`FORJAR`, `PROCLAMAR`, `DUELO`, …) | 0 |

- **Consulta:** `ERGO()` retorna o Ergo restante.
- **Esgotamento:** se uma operação custa mais do que resta, o programa **é encerrado** com
  `[Ergo esgotado] linha N: ...` (código de saída 3). A operação não é executada, e o que já estava no
  Stargazer permanece.
- Efeito colateral útil: laços infinitos sempre terminam.

## 4. Mecânica 2 — Stargazer (persistência)

Um Stargazer é **uma lista** de informações gravada em um arquivo JSON: o arquivo é sempre um array
na raiz, nunca um objeto. Cada informação ocupa uma **posição** (0, 1, 2, ...), como um item de uma
Legião. O valor guardado pode ser Ergo, Fable ou Legião. Formato do arquivo:

```json
["Espada", "Escudo"]
```

| Comando / função | Efeito |
|---|---|
| `ACENDER "arq.json"` | abre o Stargazer (cria o arquivo com `[]` se não existir) |
| `REGISTRAR valor` | **Cria**: acrescenta `valor` ao final da lista |
| `CONSULTAR(posicao)` | **Lê** o valor na posição (erro se não existe) |
| `ATUALIZAR posicao, valor` | **Atualiza** o valor naquela posição (erro se não existe) |
| `EXCLUIR posicao` | **Exclui** o item daquela posição (erro se não existe) |
| `EXISTE(posicao)` | `VERDADE`/`MENTIRA`, sem custo |
| `TUDO()` | Legião com uma cópia de todos os itens, na ordem da lista |

Posições são inteiros a partir de 0. Cada alteração é gravada em disco imediatamente. Violações de
regra (posição inexistente, Stargazer não aceso, tipo não registrável) encerram o programa com
`[Stargazer] linha N: ...` (código 5). Os programas evitam o erro consultando `EXISTE` antes.

## 5. Mecânica 3 — Humanidade (estados e confiabilidade)

Um **Fato** carrega um estado e uma confiança. Operações:

| Função | Significado |
|---|---|
| `FATO(estado, confianca)` | cria um Fato (confiança entre 0 e 100) |
| `ESTADO(f)`, `CONFIANCA(f)` | consulta as duas partes |
| `JUNTAR(a, b)` | "e" de três estados |
| `ALTERNAR(a, b)` | "ou" de três estados |
| `INVERTER(a)` | nega; `INCERTO` permanece `INCERTO` |
| `ASSUMIR(f, estado)` | aposta num Fato `INCERTO`: vira `VERDADE`/`MENTIRA` com confiança **no máximo 40** |
| `REFORCAR(f, n)` | gasta **n Ergo** e soma n à confiança (máximo 100) |

**Tabelas** (T = VERDADE, F = MENTIRA, I = INCERTO):

| JUNTAR | T | F | I |   | ALTERNAR | T | F | I |
|---|---|---|---|---|---|---|---|---|
| **T** | T | F | I | | **T** | T | T | T |
| **F** | F | F | F | | **F** | T | F | I |
| **I** | I | F | I | | **I** | T | I | I |

**Regra da confiança:** o resultado de `JUNTAR`/`ALTERNAR` tem a **menor** confiança entre os dois
operandos ("uma cadeia vale tanto quanto seu elo mais fraco"). `INVERTER` preserva a confiança.

**Consequência na execução:** em `DUELO`, `REVIDAR` e `PERSISTIR`, um Fato só decide o caminho se o
estado for `VERDADE`/`MENTIRA` **e** a confiança for **≥ 50**. Caso contrário o programa é
interrompido com `[Humanidade insuficiente] linha N: ...` (código 4). O caminho para decidir é:
`INCERTO → ASSUMIR → REFORCAR (gasta Ergo) → decidir`. Operadores `E`/`OU`/`NAO` não valem para Fatos
(erro código 4); use as funções acima.

## 6. Comandos

Revisão: os comandos abaixo são equivalentes temáticos de estruturas convencionais, mantidos por
simplicidade; a originalidade está nas seções 3 a 5 e nas regras da seção 7.

| Comando | Sintaxe | Equivale a |
|---|---|---|
| ACORDAR | `ACORDAR` ou `ACORDAR n` | início obrigatório; define o Ergo inicial |
| DORMIR | `DORMIR` | fim obrigatório |
| FORJAR | `FORJAR x = expr` / `FORJAR v[i] = expr` | atribuição |
| PROCLAMAR / SUSSURRAR | `PROCLAMAR a, b` | imprime com / sem quebra de linha |
| OUVIR_ERGO / OUVIR_FABLE | `OUVIR_ERGO x` | lê inteiro (inválido → -1) / texto |
| LEGIAO | `LEGIAO v` | cria lista vazia |
| ANEXAR / REMOVER_EM | `ANEXAR v, x` / `REMOVER_EM v, i` | altera lista |
| DUELO / REVIDAR / ESQUIVA | `DUELO cond` … `FIM` | if / elif / else (aceita Fatos, ver seção 5) |
| PERSISTIR | `PERSISTIR cond` … `FIM` | while |
| CICLO / RECUAR | `CICLO i DE a ATE b` | for crescente / decrescente, inclusivo |
| FUGIR | `FUGIR` | break |
| ARTE / DEVOLVER / INVOCAR | `ARTE nome(p)` … `FIM` | função / retorno / chamada sem usar o resultado |
| ACENDER, REGISTRAR, ATUALIZAR, EXCLUIR | seção 4 | **Stargazer** |

Comentário: linha começando com `~`. Todo bloco termina com `FIM`.

## 7. Gramática e regras de execução

```
programa   = "ACORDAR" [ERGO_LITERAL] { comando } "DORMIR"
bloco      = { comando } "FIM"
expressao  = ordem usual: ( ) [ ] chamadas > * / % > + - > comparações > NAO > E > OU
embutidas  = TAMANHO TEXTO INTEIRO | ERGO | EXISTE CONSULTAR TUDO
             | FATO ESTADO CONFIANCA JUNTAR ALTERNAR INVERTER REFORCAR ASSUMIR
```

1. Um comando por linha; `ACORDAR` é a primeira e `DORMIR` a última linha de código.
2. O tradutor aceita **apenas** símbolos e palavras de Krat. `import`, `print`, `open`, ponto
   (`a.b`), etc. são rejeitados. Chamar uma arte não definida também.
3. Tradução: erros de sintaxe → `[Mentira detectada] linha N` (o código não é gerado).
4. Execução: toda falha encerra o programa com uma mensagem apontando a **linha do .krat**.

| Código de saída | Causa |
|---|---|
| 0 | execução normal |
| 3 | Ergo esgotado |
| 4 | Humanidade insuficiente |
| 5 | erro no Stargazer |
| 6 | outro erro de execução (divisão por zero, índice fora da Legião, variável não forjada…) |

## 8. O tradutor

`krat2py.py` valida o `.krat` e gera um `.py` contendo o **runtime de Krat** (Ergo, Stargazer, Fatos,
mapa de linhas, tratamento de erros) mais o programa traduzido. O runtime usa recursos internos do
Python apenas para implementar a semântica dos comandos; os programas `.krat` não têm acesso a eles.
Nota: o antigo par `SELAR`/`DESSELAR` foi substituído pelo Stargazer.

## 9. Programas

| Arquivo | Conteúdo |
|---|---|
| `programas/hello_world.krat` | Hello World! |
| `programas/garrafas.krat` | 99 garrafas de cerveja |
| `programas/q1_fatorial.krat` | fatorial de N (0 a 15) |
| `programas/q2_fibonacci.krat` | N-ésimo Fibonacci (1 a 10) |
| `programas/q3_hanoi.krat` | Torre de Hanói, 6 discos, 63 movimentos |
| `programas/q4_ordenacao.krat` | ordena N < 100 inteiros (Bubble Sort) |
| `programas/q5_crud.krat` | CRUD no Stargazer (`stargazer.json`) |
| `programas/ergo_demo.krat` | demonstra consumo e esgotamento do Ergo |
| `programas/humanidade_demo.krat` | demonstra Fatos, ASSUMIR, REFORCAR e interrupção |

## 10. Limitações conhecidas

- Só há inteiros (sem decimais); strings sem escape de aspas.
- O Stargazer aceita Ergo, Fable e Legião; não guarda Fatos nem registros com campos nomeados (é uma lista, não um dicionário).
- Um Stargazer aceso por vez; posições são sempre inteiras, a partir de 0.
- O limiar de confiança (50) e os custos de Ergo são fixos.
- O Ergo mede passos de laço, chamadas e acessos, não o tempo real de CPU.
- A ordenação com 99 valores consome cerca de 5000 Ergo (o padrão é 10000).
