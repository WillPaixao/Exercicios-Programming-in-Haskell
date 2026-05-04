# Exercício 1:

Podemos descrever o somatório $\sum_{x = 1}^{100} x^2$ de forma enxuta em Haskell pela expressão `sum [x^2 | x <- [1..100]]`.

# Exercício 2:

A função `grid`, que dadas dimensões `m` e `n` retorna a grade coordenada correspondente, pode ser implementada como:

```haskell
grid :: Int -> Int -> [(Int,Int)]
grid m n = [(x,y) | x <- [0..m], y <- [0..n]]
```

# Exercício 3:

Podemos escrever a função `square`, que retorna uma grade coordenada quadrada com uma certa dimensão excluindo a diagonal principal, como:

```haskell
square :: Int -> [(Int,Int)]
square n = [(x,y) | (x,y) <- grid n n, x /= y]
```

# Exercício 4:

`replicate`, função do Prelude que cria uma lista com uma certa quantidade de repetições de um elemento, pode ser definida usano _list comprehension_:

```haskell
replicate :: Int -> a -> [a]
replicate n x = [x | _ <- [1..n]]
```

# Exercício 5:

A função `pyths`, que gera uma lista de triplas pitagóricas com elementos menores ou iguais a um certo valor, pode ser descrita como:

```haskell
pyths :: Int -> [(Int,Int,Int)]
pyths n = 
  [(x,y,z) | x <- [1..n], 
             y <- [1..n], 
			 z <- [1..n], 
			 x^2 + y^2 == z^2]
```

# Exercício 6:

A função `perfects`, que retorna uma lista dos números perfeitos menores ou iguais a um valor, pode ser implementada assim:

```haskell
perfects :: Int -> [Int]
perfects n = 
  [x | x <- [1..n], perfect x]
  where
    perfect x = (sum (factors x)) == 2*x
```

# Exercício 7:

A expressão

```haskell
[(x,y) | x <- [1,2], y <- [3,4]]
```

é avaliada iterando por cada `y` possível para cada `x` possível. Desse jeito, ela pode ser reescrita como concatenação de duas _list comprehensions_ aninhadas

```haskell
concat [[(x,y) | y <- [3,4]] | x <- [1,2]]
```

# Exercício 8:

A função `positions` pode ser redefinida usando `find` dessa maneira:

```haskell
positions :: Eq a => a -> [a] -> [Int]
positions x xs = find x (zip xs [0..])
```

# Exercício 9:

Definindo a função `scalarproduct`, que calcula o produto escalar entre duas listas de inteiros:

```haskell
scalarproduct :: [Int] -> [Int] -> Int
scalarproduct xs ys = sum [x*y | (x,y) <- zip xs ys]
```

# Exercício 10:

As únicas alterações necessárias foram nas representações codificadas das letras, para realizar o processo de `shift`amento das letras mantendo a sua caixa, e na função de contagem de letras para o cálculo das frequências, que deve ser insensível à caixa.

```haskell
-- Na codificação...
let2int :: Char -> (Int,Int)
let2int c | isAsciiLower c = (ord 'a', ord c - ord 'a')
          | isAsciiUpper c = (ord 'A', ord c - ord 'A')
          | otherwise      = (0, ord c)

int2let :: (Int,Int) -> Char
int2let (base, offset) = chr (base + offset)

shift :: Int -> Char -> Char
shift n c = int2let (base, offset')
  where
    (base,offset) = let2int c
    offset' = 
      if base == 0 then 
        offset 
      else 
        (offset + n) `mod` 26

-- Na montagem da tabela de frequências...
letters :: String -> Int
letters cs = sum [1 | c <- cs, isAsciiLower c || isAsciiUpper c]

count :: Char -> String -> Int
count c cs = sum [1 | c' <- cs, toLower c' == c]

freqs :: String -> [Float]
freqs cs = [percent (count c cs) total | c <- ['a'..'z']]
  where
    total = letters cs
```