# Exercício 1:

A lista formada pela _list comprehension_ `[f x | x <- xs, p x]`, pode ser interpretada pelo seu gerador e guarda, além da expressão que descreve seus elementos. O gerador `x <- xs` indica que cada elemento `x` de uma lista originária `xs` é tomado. Em cima desse elemento `x`, é aplicada uma guarda `p x`, que verifica se a propriedade descrita pelo predicado `p` é satisfeita por `x`. Isso, em termos de funções de alta ordem, equivale a uma filtragem de `xs` sob a função de teste `p`. Por fim, os valores de `x` que satisfizerem `p` serão transformados por `f`, através da aplicação `f x`, e comporão a lista resultante. A aplicação de `f` a cada um dos elementos devidamente filtrados corresponde a um `map`eamento. Destarte, temos a seguinte igualdade:

```haskell
[f x | x <- xs, p x] = map f (filter p xs)
```

# Exercício 2:

As seguintes funções de alta ordem do _Prelude_ padrão podem ser definidas como:

```haskell
-- Checa se todos os elementos de uma lista respeitam um predicado
all :: (a -> Bool) -> [a] -> Bool
all p = foldr (\x b -> p x && b) True

-- Checa se algum dos elementos de uma lista respeita um predicado
any :: (a -> Bool) -> [a] -> Bool
any p = foldr (\x b -> p x || b) False

-- Seleciona items de uma lista enquanto satisfazem um predicado
takeWhile :: (a -> Bool) -> [a] -> [a]
takeWhile p []                 = []
takeWhile p (x:xs) | p x       = x : takeWhile p xs
                   | otherwise = []

-- Remove items de uma lista enquanto satisfazem um predicado
dropWhile :: (a -> Bool) -> [a] -> [a]
dropWhile p []                 = []
dropWhile p (x:xs) | p x       = dropWhile p xs
                   | otherwise = x:xs
```

# Exercício 3:

Redefinindo `map` e `filter` usando `foldr`:

```haskell
map :: (a -> b) -> [a] -> [b]
map f = foldr (\x xs -> f x : xs) []

filter :: (a -> Bool) -> [a] -> [a]
filter p = foldr condCons []
  where
    condCons x xs | p x       = x:xs
	              | otherwise = xs
```

# Exercício 4:

A função `dec2int`, que transforma uma lista de dígitos decimais em um inteiro, pode ser escrita, utilizando `foldl`, assim:

```haskell
dec2int :: [Int] -> Int
dec2int = foldl (\n d -> 10*n + d) 0
```

# Exercício 5:

As funções `curry` e `uncurry`, que tratam da conversão de versões de funções que recebem pares para sua _curried_, e vice-versa, podem ser definidas como:

```haskell
curry :: ((a,b) -> c) -> (a -> b -> c)
curry f = \x y -> f (x,y)

uncurry :: (a -> b -> c) -> ((a,b) -> c)
uncurry f = \(x,y) -> f x y
```

# Exercício 6:

É dada uma função `unfold`, que recebe um critério de parada `p`, uma transformação `h`, uma função de redução `t`, e um valor inicial `x`, e retorna uma lista formada por concatenações sucessivas do valor transformado `h x`, seguidas da redução `t x`, enquanto o critério de parada não for atendido. Sua definição é:

```haskell
unfold :: (a -> Bool) -> (a -> b) -> (a -> a) -> a -> [b]
unfold p h t x | p x       = []
               | otherwise = h x : unfold p h t (t x)
```

Podemos definir algumas funções de maneira mais concisa em termos dela:

```haskell
chop8 :: [Bit] -> [[Bit]]
chop8 = unfold null (take 8) (drop 8)

map :: (a -> b) -> [a] -> [b]
map f = unfold null (f.head) tail

iterate :: (a -> a) -> a -> [a]
iterate f = unfold (const False) id f
```

# Exercício 7:

Para implementar a verificação de erros de transmissão nos bits da mensagem, definimos três funções auxiliares: 

- `getCheckBit`: calcula o bit de paridade para uma dada representação 8-bits de um caractere;
- `putCheckBit`: adiciona o bit de paridade no início da representação;
- `dropCheckBit`: retira o bit de paridade de uma sequência de 9 bits se não for detectado erro, e lança uma exceção caso contrário.

```haskell
getCheckBit :: [Bit] -> Bit
getCheckBit bits = sum bits `mod` 2

putCheckBit :: [Bit] -> [Bit]
putCheckBit bits = getCheckBit bits : bits

dropCheckBit :: [Bit] -> [Bit]
dropCheckBit (cb:bits) | allRight  = bits
                       | otherwise = error "Error detected in message!"
  where
    allRight = cb == getCheckBit bits
```

Esses procedimentos podem ser incorporados nas funções de codificação e decodificação (a função `chop9` é análoga à `chop8`, partindo o fluxo de bits em sequências de tamanho 9):

```haskell
encode :: String -> [Bit]
encode = concat . map (putCheckBit . make8 . int2bin . ord)

decode :: [Bit] -> String
decode = map (chr . bin2int) . (map dropCheckBit) . chop9
```

Perceba que as funções `putCheckBit` e `dropCheckBit` calculam indiretamente o bit de checagem por meio da interface `getCheckBit`. Dessa maneira, a tecnologia de determinação do bit foi isolada das funções básicas de adição e remoção do bit em uma sequência, de modo que qualquer algoritmo de cômputo do bit seja aplicável pela simples modificação de `getCheckBit`. Se quiséssemos usar uma verificação por paridade da representação numérica dos bits, por exemplo, bastaria mudarmos `getCheckBit`:

```haskell
getCheckBit :: [Bit] -> Bit
getCheckBit bits = bin2int bits `mod` 2
```

Todo o resto permanece intacto e funcional.

# Exercício 8:

Ao simular um canal de comunicação ruidoso, definindo `channel = tail`, a função de decodificação conseguiu detectar, corretamente, o erro na mensagem. O serviço de verificação foi provido pela função `dropCheckBit`, que é mapeada no fluxo de representações de 9 bits recebido do outro lado do canal. Quando o primeiro bit do fluxo é perdido, é bem provável que a paridade do bit de checagem não seja obedecida em alguma das sequências, provocando o lançamento da exceção.

# Exercício 9:

A função `altMap`, que mapeia duas funções aos elementos de uma lista alternadamente, pode ser dada por:

```haskell
altMap :: (a -> b) -> (a -> b) -> [a] -> [b]
altMap f g = applyF
  where
    applyF [] = []
	applyF (x:xs) = f x : applyG xs
	applyG [] = []
	applyG (x:xs) = g x : applyF xs
```

# Exercício 10:

O algoritmo de Luhn, para validação de números de cartão de crédito, pode ser implementado com `altMap`:

```haskell
luhn :: [Int] -> Bool
luhn = (divisibleBy 10) . sum . (altMap id luhnDouble) . reverse
  where
    divisibleBy n = \x -> (x `mod` n) == 0
```

OBS.: Funcionou com o número do meu cartão :-)