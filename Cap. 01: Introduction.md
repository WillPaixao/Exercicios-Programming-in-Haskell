###### Exercício 1:

Uma outra derivação possível de `double (double 2)` é:

```haskell
  double (double 2) 
= {aplicação de double a 2}
  double (2 + 2) 
= {aplicação de double a (2 + 2)}
  (2 + 2) + (2 + 2)
= {aplicação de +}
  8
```

###### Exercício 2:

Assumindo a seguinte definição da função `sum`:

```haskell
sum []     = 0
sum (n:ns) = n + sum ns
```

temos a seguinte sequência de passos ao avaliar `sum [x]`, com `x` sendo um número qualquer:

```haskell
  sum [x] 
= {aplicação de sum a [x]}
  x + sum [] 
= {aplicação de sum a []}
  x + 0 
= {aplicação de + a x, 0}
  x
```

###### Exercício 3:

Definindo `product` de maneira similar a `sum`:

```haskell
product []     = 1
product (n:ns) = n * product ns
```

Aqui definimos o valor no caso base como `1`, levando em conta que essa é a identidade da multiplicação (`x * 1 = x` para todo número `x`).
Verificando o resultado para a lista `[2,3,4]`:

```haskell
  product [2,3,4]
= {aplicação de product a [2,3,4]}
  2 * product[3,4]
= {aplicação de product a [3,4]}
  2 * (3 * product [4])
= {aplicação de product a [4]}
  2 * (3 * (4 * product []))
= {aplicação de product a []}
  2 * (3 * (4 * 1))
= {aplicação de *}
  24
```

###### Exercício 4:

Bastaria trocar `smaller` e `larger` de posição no caso recursivo. Pensando em termos da abstração gerada pela definição indutiva, a lista ordenada resultante do caso recursivo consistiria na ordenação da lista com valores maiores que o pivô `x`, seguida do `x`, seguida da ordenação dos elementos menores ou iguais a `x`.

###### Exercício 5:

Na etapa de partição dos conjuntos de valores maiores e menores que o pivô, o que ocorreria é que valores iguais ao pivô não seriam incluídos na lista `smaller`, descartando-os. Dessa maneira, `qsort`, além de ordenar a lista crescentemente, eliminaria elementos repetidos.

Avaliando o caso `qsort [2,2,3,1,1]`:

```haskell
  qsort [2,2,3,1,1]
= {aplicação de qsort a [2,2,3,1,1]}
  qsort [1,1] ++ [2] ++ qsort [3]
= {aplicação de qsort a [1,1]}
  (qsort [] ++ [1] ++ qsort []) ++ [2] ++ qsort [3]
= {aplicação de qsort a []}
  ([] ++ [1] ++ []) ++ [2] ++ qsort [3]
= {aplicação de ++}
  [1] ++ [2] ++ qsort [3]
= {aplicação de qsort a [3]}
  [1] ++ [2] ++ (qsort [] ++ [3] ++ qsort [])
= {aplicação de qsort a []}
  [1] ++ [2] ++ ([] ++ [3] ++ [])
= {aplicação de ++}
  [1] ++ [2] ++ [3]
= {aplicação de ++}
  [1,2,3]
```