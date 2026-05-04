# Exercício 1:

A versão fornecida de `fac` desconsidera o tratamento de entradas negativas tendo em vista que não há algum teste para tal. Um inteiro negativo se enquadraria, dessa maneira, no caso recursivo `fac n = n * fac (n-1)`, que chama recursivamente o fatorial com uma unidade a menos. Como a entrada negativa, quando decrementada de 1, torna-se menor ainda, e os inteiros negativos são, em teoria, infinitos, então `fac` seria expandido indefinidamente.

# Exercício 2:

Poderíamos definir `sumdown`, que recebe um inteiro não-negativo `n` como entrada e retorna o somatório $\sum_{i=0}^{n}i$ da seguinte maneira:

```haskell
sumdown :: Int -> Int
sumdown 0 = 0
sumdown n = n + sumdown (n-1)
```

# Exercício 3:

A implementação da exponenciação (para inteiros não-negativos) pode ser dada recursivamente da seguinte forma:

```haskell
(^) :: Int -> Int -> Int
b ^ 0 = 1
b ^ n = b * (b ^ (n-1))
```

cuja avaliação de `2 ^ 3` se dá em 4 passos de derivação (contando apenas expansões de `^`):

```haskell
  2 ^ 3
= {aplicação de ^ a 2 e 3}
  2 * (2 ^ 2)
= {aplicação de ^ a 2 e 2}
  2 * (2 * (2 ^ 1))
= {aplicação de ^ a 2 e 1}
  2 * (2 * (2 * (2 ^ 0)))
= {aplicação de ^ a 2 e 0}
  2 * (2 * (2 * 1))
= {aplicação de *}
  8
```

Uma maneira menos convencional de descrever a exponenciação de inteiros, conhecida pelo nome de **square-multiply**, é:

```haskell
(^) :: Int -> Int -> Int
b ^ 0 = 1
b ^ n | even n    = (square b) ^ (n `div` 2)
      | otherwise = b * b ^ (n-1)
  where
    square x = x * x
```

Nessa nova definição, a avaliação de `2 ^ 3` também leva 4 passos de derivação (contando apenas expansões de `^`):

```haskell
  2 ^ 3
= {aplicação de ^ a 2 e 3}
  2 * (2 ^ 2)
= {aplicação de ^ a 2 e 2}
  2 * ((2 * 2) ^ 1)
= {aplicação de ^ a (2 * 2) e 1}
  2 * ((2 * 2) * ((2 * 2) ^ 0))
= {aplicação de ^ a (2 * 2) e 0}
  2 * ((2 * 2) * 1)
= {aplicação de *}
  8
```

Nesse caso, com expoentes muito baixos, o segundo algoritmo atua de forma ruim. Porém, para altos expoentes pares, ele reduz o problema pela metade, apenas introduzindo um quadrado na base. Essa otimização para o caso par, quando repetida várias vezes no cálculo da exponenciação, pode reduzir bastante o tempo de execução final. Vejamos o caso `2 ^ 100` (a avaliação é feita em ordem aplicativa para economizar espaço):

```haskell
  2 ^ 100
= {aplicação de ^ a 2 e 100}
  4 ^ 50
= {aplicação de ^ a 4 e 50}
  16 ^ 25
= {aplicação de ^ a 16 e 25}
  16 * (16 ^ 24)
= {aplicação de ^ a 16 e 24}
  16 * (256 ^ 12)
= {aplicação de ^ a 256 e 12}
  16 * (65536 ^ 6)
= {aplicação de ^ a 65536 e 6}
  16 * (4294967296 ^ 3)
= {aplicação de ^ a 4294967296 e 3}
  16 * (4294967296 * (4294967296 ^ 2))
= {aplicação de ^ a 4294967296 e 2}
  16 * (4294967296 * (18446744073709551616 ^ 1))
= {aplicação de ^ a 18446744073709551616 e 1}
  16 * (4294967296 * (18446744073709551616 *
  (18446744073709551616 ^ 0))
= {aplicação de ^ a 18446744073709551616 e 0}
  16 * (4294967296 * (18446744073709551616 * 1)
= {aplicação de *}
  1267650600228229401496703205376
```

Veja que, em apenas 10 passos de derivação (contando apenas expansões de `^`), chegou-se ao resultado. Em contraste, a versão ingênua demandaria 101 reduções, já que o expoente é reduzido de 1 em 1 até chegar a 0. Empiricamente, a diferença de desempenho fica gritante quando tentamos calcular exponenciações com expoentes maiores ainda, como 1 milhão.

Considerando o segundo algoritmo, no seu melhor caso todos os expoentes são divididos por 2 em sequência, sem necessidade de subtração por 1. Nesse caso, o número de passos de redução seria $\log_2(n) + 1$, sendo $n$ o valor inicial do expoente. No pior caso, será necessário subtrair por 1 toda vez que houver uma divisão do expoente, fazendo com que as aproximadas $\log_2(n)$ divisões sejam acompanhadas, cada uma, de um decremento. Assim seriam feitos na ordem de $2 \log_2(n) + 1$ passos no cálculo. Logo, podemos afirmar que a complexidade do algoritmo square-multiply, em seu caso médio, é $O(\log_2(n))$, frente aos $O(n)$ da abordagem inicial.

# Exercício 4:

Da maneira enunciada, a função `euclid` poderia ser escrita como:

```haskell
euclid :: Int -> Int -> Int
euclid a b | a == b    = a
           | a <  b    = euclid a (b-a)
		   | otherwise = euclid (a-b) b
```

Uma implementação que cobre o caso 0 é dada por:

```haskell
euclid :: Int -> Int -> Int
euclid a 0 = a
euclid a b = euclid b (a `rem` b)
```

# Exercício 5:

Levando em conta as definições dadas no capítulo:

```haskell
length :: [a] -> Int
length []     = 0
length (_:xs) = 1 + length xs

drop :: Int -> [a] -> [a]
drop 0 xs     = xs
drop _ []     = []
drop n (_:xs) = drop (n-1) xs

init :: [a] -> [a]
init [_]    = []
init (x:xs) = x : init xs
```

As expressões do enunciado são avaliadas desse jeito:

```haskell
-- length [1,2,3]
  length [1,2,3]
= {aplicação de length a [1,2,3]}
  1 + length [2,3]
= {aplicação de length a [2,3]}
  1 + (1 + length [3])
= {aplicação de length a [3]}
  1 + (1 + (1 + length []))
= {aplicação de length a []}
  1 + (1 + (1 + 0))
= {aplicação de +}
  3

-- drop 3 [1,2,3,4,5]
  drop 3 [1,2,3,4,5]
= {aplicação de drop a 3 e [1,2,3,4,5]}
  drop 2 [2,3,4,5]
= {aplicação de drop a 2 e [2,3,4,5]}
  drop 1 [3,4,5]
= {aplicação de drop a 1 e [3,4,5]}
  drop 0 [4,5]
= {aplicação de drop a 0 e [4,5]}
  [4,5]

-- init [1,2,3]
  init [1,2,3]
= {aplicação de init a [1,2,3]}
  1 : init [2,3]
= {aplicação de init a [2,3]}
  1 : (2 : init [3])
= {aplicação de init a [3]}
  1 : (2 : [])
= {aplicação de :}
  [1,2]
```

# Exercício 6:

Definindo as funções listadas de maneira recursiva:

```haskell
-- Aplica and a todos os booleanos de uma lista
and :: [Bool] -> Bool
and []     = True
and (b:bs) = b && and bs

-- Concatena todas as listas em uma lista
concat :: [[a]] -> [a]
concat [[]]     = []
concat (xs:xss) = xs ++ concat xss

-- Replica um elemento n vezes em uma lista
replicate :: Int -> a -> [a]
replicate 0 x = []
replicate n x = x : replicate (n-1) x

-- Seleciona um item de uma lista em um certo índice
-- Gera erro quando o índice está fora dos limites
(!!) :: [a] -> Int -> a
[]     !! _ = error "Index out of bounds!"
(x:_)  !! 0 = x
(_:xs) !! n = xs !! (n-1)

-- Checa se um valor é elemento de uma lista
elem :: Eq a => a -> [a] -> Bool
elem x [] = False
elem x (x':xs) | x == x'   = True
               | otherwise = elem x xs
```

# Exercício 7:

A função `merge`, que recebe duas listas ordenadas e as mescla de maneira ordenada, pode ser dada recursivamente por:

```haskell
merge :: Ord a => [a] -> [a] -> [a]
merge xs [] = xs
merge [] ys = ys
merge (x:xs) (y:ys) | x < y     = x : merge xs (y:ys)
                    | otherwise = y : merge (x:xs) ys
```

# Exercício 8:

O algoritmo **Merge Sort** pode ser implementado, usando a função `merge` anterior, dessa maneira:

```haskell
msort :: Ord a => [a] -> [a]
msort []  = []
msort [x] = [x]
msort xs  = merge (msort ls) (msort rs)
  where 
    (ls,rs) = splitAt (length xs `div` 2) xs
```

# Exercício 9:

## `sum`: Calcula o somatório de uma lista de números

1. **Definir o tipo**: Começamos definindo no domínio das listas de inteiros. A função deve receber uma lista, contendo os inteiros, e retornar seu somatório, que deve ser um número inteiro.

```haskell
sum :: [Int] -> Int
```

2. **Enumerar os casos**: Temos duas situações claras: ou a lista de entrada é vazia, ou ela possui pelo menos um número.

```haskell
sum :: [Int] -> Int
sum []     =
sum (n:ns) =
```

3. **Definir os casos base**: O caso base, cujo resultado conseguimos determinar diretamente, é o da lista vazia. Definiremos o somatório da lista vazia como sendo 0, pois 0 é elemento neutro da soma.

```haskell
sum :: [Int] -> Int
sum []     = 0
sum (n:ns) =
```

4. **Definir os casos recursivos**: No caso recursivo, temos uma lista com pelo menos um número. Se eu sei calcular o somatório de uma lista e quero encontrar o da mesma lista acrescida de um inteiro na cabeça, posso simplesmente somar todos os elementos da lista e adicionar o inteiro no resultado.

```haskell
sum :: [Int] -> Int
sum []     = 0
sum (n:ns) = n + sum ns
```

5. **Generalizar e simplificar**: Em momento nenhum foi necessária a restrição de que os elementos a serem somados na lista deveriam ser do tipo `Int`. Como todos os tipos numéricos implementam a operação de soma, podemos generalizar a função para processar listas de quaisquer valores numéricos. Além disso, o corpo da função segue o padrão estrutural do `foldr`, e, por isso, podemos usá-lo com função de agregação `+` e caso base `0`.

```haskell
sum :: Num a => [a] -> a
sum = foldr (+) 0
```

## `take`: Pega os `n` primeiros elementos de uma lista

1. **Definir o tipo**: Nosso objetivo é coletar uma certa quantidade, inteira não-negativa, de elementos de uma lista qualquer. Logo, a restrição de tipo fica somente no argumento do número de elementos a serem extraídos.

```haskell
take :: Int -> [a] -> [a]
```

2. **Enumerar os casos**: Temos três situações principais: ou o número de elementos a serem pegos é 0; ou a lista de entrada é vazia; ou nenhum dos dois ocorre.

```haskell
take :: Int -> [a] -> [a]
take 0 xs     =
take n []     =
take n (x:xs) =
```

3. **Definir os casos base**: Os dois primeiros casos podem ser especificados de antemão. Se quero pegar 0 itens de uma lista, então a lista resultante deve ser vazia (sem nenhum elemento). No caso de tentar extrair valores de uma lista vazia, não há opção a não ser retornar o vazio.

```haskell
take :: Int -> [a] -> [a]
take 0 xs     = []
take n []     = []
take n (x:xs) =
```

4. **Definir os casos recursivos**: No caso recursivo, podemos extrair apenas o primeiro elemento da lista e delegar a tarefa de pegar os `n-1` restantes da lista remanescente para a chamada recursiva. Como essa próxima chamada terá ambos os argumentos mais próximos dos definidos nos casos base (a quantidade de elementos a serem obtidos e a lista são menores), temos garantia de que o algoritmo terminará em algum momento.

```haskell
take :: Int -> [a] -> [a]
take 0 xs     = []
take n []     = []
take n (x:xs) = x : take (n-1) xs
```

5. **Generalizar e simplificar**: A generalização de tipo não é estritamente necessária, já que deve consistir em um valor `Integral` e não é esperado que listas de tamanhos arbitrários, maiores que o limite superior de `Int`, sejam manipuladas. Uma simplificação possível é a ocultação dos nomes dos parâmetros não utilizados nos casos.

```haskell
take :: Int -> [a] -> [a]
take 0 _      = []
take _ []     = []
take n (x:xs) = x : take (n-1) xs
```

## `last`: Acessa o último elemento de uma lista

1. **Definir o tipo**: O tipo da lista não importa, assim como o de seu último elemento. Assim, `last` deve receber uma lista de elementos de um tipo qualquer e retornar um elemento daquele tipo.

```haskell
last :: [a] -> a
```

2. **Enumerar os casos**: Temos dois casos: ou a lista é unitária, ou ela tem mais de um elemento. Não devemos abranger a lista vazia, já que ela não tem um último elemento.

```haskell
last :: [a] -> a
last [x]    =
last (x:xs) =
```

3. **Definir os casos base**: O caso base é o mais simples dentre os dois: o da lista unitária. O seu último elemento é meramente o valor que está presente nela.

```haskell
last :: [a] -> a
last [x]    = x
last (x:xs) =
```

4. **Definir os casos recursivos**: Para o caso recursivo, se eu sei o último elemento de uma lista e quero descobrir o último da mesma lista acrescentada de um item à sua frente, basta que eu retorne o último elemento da lista anterior. O elemento acrescentado não mudará a identidade do último item.

```haskell
last :: [a] -> a
last [x]    = x
last (x:xs) = last xs
```

5. **Generalizar e simplificar**: A função parece ser genérica o suficiente. Uma oportunidade de simplificação seria deixar de nomear o primeiro elemento da lista no caso recursivo, que pouco importa no resultado.

```haskell
last :: [a] -> a
last [x]    = x
last (_:xs) = last xs
```