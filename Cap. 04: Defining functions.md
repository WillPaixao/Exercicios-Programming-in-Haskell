###### Exercício 1:

Escrevendo a função `halve`, que quebra uma lista de entrada em um par com suas duas metades:

```haskell
halve :: [a] -> ([a],[a])
halve xs = 
  (take n xs, drop n xs)
  where
    n = length xs `div` 2
```

###### Exercício 2:

Definindo a função `third`, que retorna o terceiro elemento de uma lista de tamanho mínimo 3, de três maneiras diferentes:

```haskell
-- Usando head e tail
third' :: [a] -> a
third' xs = head (tail (tail xs))

-- Usando indexação por !!
third'' :: [a] -> a
third'' xs = xs !! 2

-- Usando pattern matching
third''' :: [a] -> a
third''' (_:(_:(x:_))) = x
```

###### Exercício 3:

Descrevendo `safetail`, que se comporta como `tail` para listas não-nulas e como a identidade para listas nulas, de três formas distintas:

```haskell
-- Usando expressão condicional
safetail' :: [a] -> [a]
safetail' xs =
  if null xs then
    xs
  else
    tail xs

-- Usando equações de guarda
safetail'' :: [a] -> [a]
safetail'' xs | null xs   = xs
              | otherwise = tail xs

-- Usando pattern matching
safetail''' :: [a] -> [a]
safetail''' [] = []
safetail''' xs = tail xs
```

###### Exercício 4:

Com _pattern matching_, a disjunção de dois valores booleanos pode ser definida de alguns jeitos:

```haskell
(||) :: Bool -> Bool -> Bool

-- Versão totalmente expandida
True  || True  = True
True  || False = True
False || True  = True
False || False = False

-- Versão aproveitando da ordem das definições
False || False = False
_     || _     = True

-- Versão determinada pelo primeiro operando
True  || _ = True
False || x = x

-- Versão repetindo o nome da variável (vedada)
b || b = b
_ || _ = True
```

###### Exercício 5:

Usando apenas expressões condicionais, temos a seguinte descrição de `&&`:

```haskell
a && b =
  if a then
    if b then
	  True
	else
	  False
  else
    False
```

###### Exercício 6:

Repetindo o estilo de descrição por condicionais, mas para a versão do `&&` determinada pelo primeiro operando:

```haskell
a && b =
  if a then
    b
  else
    False
```

###### Exercício 7:

Expressando `mult`, definido como

```haskell
mult :: Int -> Int -> Int -> Int
mult x y z = x*y*z
```

formalmente por meio de expressões _lambda_:

```haskell
mult :: Int -> (Int -> (Int -> Int))
mult = \x -> (\y -> (\z -> x*y*z))
```

###### Exercício 8:

Definindo as funções `luhnDouble` e `luhn`, que simulam a operação de verificação de dígitos de um cartão de crédito:

```haskell
luhnDouble :: Int -> Int
luhnDouble d | dbl > 9   = dbl - 9
             | otherwise = dbl
			 where
			   dbl = 2*d

luhn :: Int -> Int -> Int -> Int -> Bool
luhn a b c d =
  (sum [a',b,c',d] `mod` 10) == 0
  where
    a' = luhnDouble a
	c' = luhnDouble c
```