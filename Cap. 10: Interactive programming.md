###### Exercício 1:

A ação `putStr cs`, que imprime a string `cs` na saída padrão, pode ser descrita em função de `putChar` e `sequence_`:

```haskell
putStr :: String -> IO ()
putStr cs = sequence_ [putChar c | c <- cs]
```

###### Exercício 2:

A ação `putBoard b`, para exibição de um tabuleiro de Nim dado por uma lista contendo as quantidades de estrelas em cada linha, pode ser definida recursivamente, para tabuleiros de qualquer tamanho:

```haskell
putBoard :: Board -> IO ()
putBoard = loop 1
  where
    loop _ []     = return ()
	loop i (r:rs) = do
	  putRow i r
	  loop (i+1) rs
```

###### Exercício 3:

A generalização de `putBoard` pode ser escrita de maneira mais enxuta através da função `sequence_`:

```haskell
putBoard :: Board -> IO ()
putBoard b = sequence_ [putRow i r | (i,r) <- zip [1..] b]
```

###### Exercício 4:

Podemos desenhar a ação `adder`, que corresponde a um programa que lê uma certa quantidade de números da entrada padrão e apresenta seu somatório ao final, da seguinte forma:

```haskell
-- Ação auxiliar que lê um inteiro da entrada padrão
getInt :: IO Int
getInt = do
  n <- getLine
  return (read n)

-- Ação auxiliar que acumula os números de entrada até a quantidade de iterações requerida
accum :: Int -> IO Int
accum remIter = 
  if remIter <= 0 then
    return 0
  else do
    n <- getInt
    rest <- accum (remIter-1)
    return (n + rest)

adder :: IO ()
adder = do
  putStr "How many numbers? "
  nIter <- getInt
  res <- accum nIter
  putStr "The total is "
  putStrLn (show res)
```

Vale ressaltar que existem alguns pontos passíveis de falha, em especial no que diz respeito à validação do tipo dos dados de entrada em `getInt`. Poderíamos ter incorporado essa defesa na ação, solicitando o usuário repetidas vezes até que a condição da entrada ser `Int`eira fosse obedecida. Porém, não há tanto motivo de preocupação, já que a função `read`, usada para converter a string do usuário em um dado do tipo `Int`, faz o papel de lançar uma exceção no caso de formato inválido, prevenindo comportamentos imprevisíveis.

###### Exercício 5:

Valendo-se da função `sequence`, que combina uma sequência de ações do tipo `[IO a]` em uma única ação do tipo `IO [a]`, a ação `adder` fica muito mais enxuta, sem nem precisarmos da definição auxiliar `accum`, do **Exercício 4**:

```haskell
adder :: IO ()
adder = do
  putStr "How many numbers? "
  nIter <- getInt
  vals <- sequence (replicate nIter getInt)
  putStr "The total is "
  putStrLn (show (sum vals))
```

###### Exercício 6:

A ação `readLine`, que implementa `getLine` com a funcionalidade adicional de permitir que a tecla de _Backspace_ seja utilizada para apagar caracteres digitados, pode ser escrita assim:

```haskell
readLine :: IO String
readLine = loop []
  where
    loop buf = do
      hSetEcho stdin False
      c <- getChar
      case c of
        '\n' -> do
          putChar '\n'
          hSetEcho stdin True
          return (reverse buf)
        '\DEL' -> 
          if null buf then
            loop buf
          else do
            putStr "\b \b"
            loop (tail buf)
        _ -> do
          putChar c
          loop (c : buf)
```

Percebe-se que a estrutura de `readLine` difere bastante da de `getLine`, como apresentado no livro, por conta de dois fatores:

- Armazenamento do conteúdo da linha em um _buffer_ que pode ter elementos removidos;
- Exibição do apagamento, demandando uma tecnologia de eco dos caracteres própria.

Dessa maneira, `readLine` faz uso de uma função auxiliar `loop`, que opera como um _loop while_ capturando caracteres individuais da entrada padrão até a quebra de linha, momento em que o estado do _buffer_ é retornado. Se um _Backspace_ é detectado, por meio da cláusula `'\DEL'` na expressão `case`, então, se o _buffer_ não estiver vazio, retira-se o primeiro caractere nele, o último que foi digitado. Se qualquer outro caractere for lido, ele é inserido na cabeça do _buffer_, analogamente a uma estrutura de pilha. Quando da saída do `loop`, o buffer é retornado revertido para manter a ordem de digitação.

O usuário final não percebe a existência do _buffer_, então torna-se imprescindível a construção de um mecanismo de exibição das operações de escrita e apagamento de caracteres. O método usual de ecoamento de caracteres é pouco conveniente para nossa aplicação, pois ele trata de ecoar até mesmo as teclas de controle e, em particular, o _Backspace_. Portanto, a ação `loop` desativa esse comportamento padrão inicialmente, e imprime os caracteres distintos de _Backspace_ conforme eles são digitados. Caso um _Backspace_ seja lido, imprime-se `"\b \b"` ao invés, que corresponde a mover o cursor para trás (atrás do último caractere inserido), botar um espaço em branco (apagando o caractere e movendo o cursor para frente), e mover novamente o cursor para trás (à frente do agora último caractere inserido). O ecoamento é religado ao retornar da ação, para minimizar o efeito colateral causado.