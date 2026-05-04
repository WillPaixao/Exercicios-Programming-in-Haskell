###### Exercício 1:

Decifrando os tipos das expressões:

```haskell
> ['a','b','c'] :: [Char]
> ('a','b','c') :: (Char,Char,Char)
> [(False,'0'),(True,'1')] :: [(Bool,Char)]
> ([False,True],['0','1']) :: ([Bool],[Char])
> [tail, init, reverse] :: [[a] -> [a]]
```

###### Exercício 2:

Instanciando expressões que pertencem aos tipos correspondentes:

```haskell
> bools = [True, False] :: [Bool]
> nums = [[1,2],[3],[]] :: [[Int]]
> add x y z = x+y+z :: Int -> Int -> Int -> Int
> copy x = (x,x) :: a -> (a,a)
> apply f x = f x :: (a -> b) -> a -> b
```

###### Exercício 3:

Enunciando os tipos das funções:

```haskell
> second xs = head (tail xs) :: [a] -> a
> swap (x,y) = (y,x) :: (a,b) -> (b,a)
> pair x y = (x,y) :: a -> b -> (a,b)
> double x = x*2 :: Num a => a -> a
> palindrome xs = reverse xs == xs :: Eq a => [a] -> Bool
> twice f x = f (f x) :: (a -> a) -> a -> a
```

###### Exercício 4:

As respostas para os exercícios anteriores foram verificadas no GHCi.

###### Exercício 5:

Em geral, não conseguimos equiparar funções porque isso demandaria a garantia de que:

- Os domínios das funções são equivalentes;
- As imagens das funções são equivalentes;
- Os mapeamentos das funções são equivalentes, para todo valor no domínio de ambas.

Na verdade, o último ponto abrange os dois primeiros. A questão de determinar todos os pares de entrada e saída de uma função qualquer já é um problema por si só, pois isso potencialmente envolveria decidir retornos não-decidíveis.

Por exemplo, poderíamos ter uma função identidade para todo `Int` de entrada à exceção do `0`, caso em que a função entra em recursão infinita. Como identificaríamos a recursão infinita nesse caso? Como descobriríamos, computacionalmente, a diferença dessa função para uma identidade que, no caso de receber `0`, itera por chamadas recursivas zilhões de vezes antes de retornar `0`?

Uma solução para isso seria comparar os corpos das funções, ou ainda uma versão "reduzida" deles. Mesmo assim, isso deixaria de considerar valores _iguais_ no sentido apresentado acima. Qual seria o resultado da equiparação das funções `f` e `g` a seguir, por exemplo?

```haskell
f :: Int -> Int
f x = x

g :: Int -> Int
g x =
  if f == g then
     (f x) + 1
  else
     f x
```

Um caso bastante restrito que permitiria equiparação de funções seria para aquelas compostas somente de operações aritméticas, que poderiam ser simplificadas por um sistema algébrico simbólico e comparadas como _strings_.