###### Exercício 1:

Indicando os _redexes_ de cada uma das expressões, com as relações de continência representadas pela estrutura de tópicos (o _redex_ mais externo é o tópico no nível mais alto, enquanto o(s) mais interno(s) encontra(m)-se nos subtópicos do _redex_ subjacente):

- `1 + (2*3)`
   - `2*3`
 
- `(1+2) * (2+3)`
   - `1+2`
   - `2+3`
 
- `fst (1+2, 2+3)`
   - `(1+2, 2+3)`
      - `1+2`
	  - `2+3`
 
- `(\x -> 1 + x) (2*3)`
   - `\x -> 1 + x`
      - `1 + x`
   - `2*3`
 
Vale ressaltar que, no último item, o _redex_ `1 + x` do interior da expressão _lambda_ é abarcado pela definição do _redex_ mais interno, no sentido de ser um _redex_ — uma aplicação de função a argumentos —, e não possuir um _redex_ em seu interior.

###### Exercício 2:

Uma das vantagens a favor da avaliação mais interna é que garantidamente cada expressão passada como argumento é avaliada apenas uma vez, enquanto no modelo mais externo pode haver duplicação de expressões e, consequentemente, repetição de cálculos. Entretanto, neste último, ainda existe a possibilidade de o argumento nem sequer ser avaliado, o que representa um ganho não só no caso de expressões autoreferenciais, mas também em expressões que não dependem do conteúdo explícito dos argumentos. Um exemplo disso é a computação de `fst (1+2,2+3)`. No estilo _call-by-value_, são necessários 3 passos de execução:

```haskell
  fst (1+2,2+3)
= {aplicação de + a 1 e 2}
  fst (3,2+3)
= {aplicação de + a 2 e 3}
  fst (3,5)
= {aplicação de fst a 3 e 5}
  3
```

Enquanto isso, em _call-by-name_, foram precisos só 2 passos:

```haskell
  fst (1+2,2+3)
= {aplicação de fst a 1+2 e 2+3}
  1+2
= {aplicação de + a 1 e 2}
  3
```

###### Exercício 3:

Dada a definição _curried_ da multiplicação de dois números, dada por `mult = \x -> (\y -> x * y)`, a avaliação de `mult 3 4`, utilizando _call-by-name_, se dá por:

```haskell
  mult 3 4
= {definição de mult}
  (\x -> (\y -> x * y)) 3 4
= {aplicação de (\x -> (\y -> x * y)) a 3}
  (\y -> 3 * y) 4
= {aplicação de (\y -> 3 * y) a 4}
  3 * 4
= {aplicação de * a 3 e 4}
  12
```

###### Exercício 4:

A estrutura infinita dos números de Fibonacci, que denotaremos por `fibs`, pode ser definida por:

```haskell
fibs :: [Integer]
fibs = 0 : 1 : [m+n | (m,n) <- zip fibs (tail fibs)]
```

Aqui seguimos fielmente a descrição recursiva da sequência, que é a de que os dois primeiros números são 0 e 1, e os seguintes são ditos como a soma do número atual mais o próximo (com "atual" se referindo, na primeira expansão da _list comprehension_, ao 0 na cabeça de `fibs`).

###### Exercício 5:

As funções `repeat`, `take` e `replicate` do _Prelude_ padrão podem ser adaptadas, para o tipo de dados

```haskell
data Tree a = Leaf | Node (Tree a) a (Tree a)
  deriving Show
```

com funções auxiliares (parecidas com as implementadas nos **Exercícios 3 e 4** do **Capítulo 8**)

```haskell
-- Conta os valores nos nós de uma árvore binária
countValues :: Tree a -> Int
countValues Leaf           = 0
countValues (Node ls _ rs) = countValues ls + countValues rs

-- Transforma uma lista em uma árvore binária balanceada
balance :: [a] -> Tree a
balance [] = Leaf
balance xs = Node (balance ls) r (balance rs')
  where
    (ls,rs) = splitAt (length xs `div` 2) xs
	(r:rs') = rs
```

como:

```haskell
-- Gera uma árvore binária potencialmente infinita com repetições de um elemento
repeatTree :: a -> Tree a
repeatTree x = Node (repeatTree x) x (repeatTree x)

-- Pega n elementos mais altos de uma árvore binária preservando as ligações entre os nós
takeTree :: Int -> Tree a -> Tree a
takeTree 0 _    = Leaf
takeTree _ Leaf = Leaf
takeTree n (Node ls x rs) = Node (takeTree (n-n'-1) ls) x (takeTree n' rs)
  where
    n' = (n-1) `div` 2

-- Gera uma árvore binária balanceada com n elementos iguais a um valor dado
replicateTree :: Int -> a -> Tree a
replicateTree n x = takeTree n (repeatTree x)
```

###### Exercício 6:

A função `sqroot`, que utiliza o método de Newton para obtenção da raiz quadrada aproximada de um `Double`, pode ser definida em termos de uma busca por uma aproximação boa o suficiente de uma sequência de aproximações cada vez melhores. Lançaremos mão da função `iterate` do _Prelude_ padrão, que gera uma lista infinita com aplicações sucessivas de uma função a um elemento.

```haskell
sqroot :: Double -> Double
sqroot n = loop (iterate next 1)
  where
    loop (x:xs) =
	  if abs (x - (next x)) < 0.00001 then
	    next x
	  else
	    loop xs
	next a = (a + n/a) / 2
```