# Exercício 1:

A função `choices`, que retorna todos os arranjos possíveis de elementos de uma lista, pode ser descrita por uma _list comprehension_:

```haskell
choices :: [a] -> [[a]]
choices xs = [c | s <- subs xs, c <- perms s]
```

# Exercício 2:

A função `isChoice`, que testa se uma lista pode ser configurada como um arranjo de elementos de outra, sem necessidade de gerar todos os arranjos possíveis, pode ser dada por:

```haskell
-- Função auxiliar que remove a primeira ocorrência de um item de uma lista, se possível
remElem :: Eq a => a -> [a] -> [a]
remElem x [] = error "Element not found in list!"
remElem x (y:ys) | x == y    = ys
                 | otherwise = y : remElem x ys

isChoice :: Eq a => [a] -> [a] -> Bool
isChoice []     ys = True
isChoice (x:xs) ys = 
  elem x ys && isChoice xs (remElem x ys)
```

# Exercício 3:

A função `split`, que gera todas as bipartições não-vazias ordenadas de uma lista, foi desenvolvida com o intuito de gerar todas as subdivisões de uma dada escolha de inteiros para composição de expressões do tipo `App` na função `exprs`, que serão avaliadas e comparadas com o alvo em `solutions`. Nesse sentido, se `split` compreendesse listas vazias, tanto na partição esquerda quanto na direita, a função `exprs`, que é chamada recursivamente em cada partição, poderia ter como argumento uma lista vazia, que, por sua vez, retorna uma lista vazia.

No caso de a partição esquerda ser vazia, o gerador `l <- exprs ls` será avaliado na ordem dentro da _list comprehension_, e como `exprs ls` é vazia, não haverá elemento algum a ser coletado e, portanto, pula-se para a próxima partição. Se a partição direita é vazia, então a lista original completa está do lado esquerdo, e, como os geradores são avaliados na ordem em que aparecem, a lista `exprs ls` será requisitada recursivamente em `l <- exprs ls`. Como `ls` é exatamente a lista de entrada de `exprs`, a função tentará determinar desde o início as expressões da lista original, causando um loop infinito. Dessa forma, `exprs`, e, consequentemente, `solutions`, rodarão para sempre.

Uma maneira de evitar o problema de recursão infinita é proibindo que a partição direita seja vazia. Mesmo assim, a bipartição com a lista esquerda vazia sabidamente nunca fornecerá uma expressão, fazendo com que sua inclusão só sobrecarregue desnecessariamente o programa.

# Exercício 4:

Para encontrarmos o número total de expressões possíveis, válidas ou não, com valores escolhidos de uma certa lista de inteiros, podemos calcular o tamanho da seguinte lista obtida por _list comprehension_, cujo procedimento para obtenção foi encapsulado na função `generateAllExprs`:

```haskell
generateAllExprs :: [Int] -> [Expr]
generateAllExprs ns = [e | ns' <- choices ns,
                           e <- exprs ns']
```

Computando `length (generateAllExprs [1,3,7,10,25,50])`, temos como resultado o número 33.665.406. Para contabilizarmos quantas são avaliáveis dessas, basta que filtremos a lista das expressões geradas naquelas cuja aplicação de `eval` gera algum resultado (i.e., não é nula):

```haskell
generateValidExprs :: [Int] -> [Expr]
generateValidExprs = 
  filter (not . null . eval) . generateAllExprs
```

Ao chamarmos `length (generateValidExprs [1,3,7,10,25,50])`, o número 4.672.540 é retornado. Veja como as expressões válidas, segundo as regras usuais do jogo, correspondem a pouco mais de 13,87% de todas as expressões possíveis com os inteiros daquela lista.

# Exercício 5:

Se afrouxássemos um pouco as regras, permitindo que qualquer inteiro pudesse ser resultado intermediário ou final da expressão, certamente teríamos uma gama consideravelmente maior de possibilidades. Modificando a função `valid` na cláusula da operação de subtração, tornando-a sempre verdadeira, e adicionando a verificação de denominador não-nulo na cláusula da divisão, ao executarmos `length (generateValidExprs [1,3,7,10,25,50])` obtemos 10.839.369 expressões válidas no domínio dos inteiros. Isso representa quase 32,2% do total, e são aproximadamente 2,32 vezes mais expressões frente às com as restrições primordiais.

# Exercício 6:

## Letra a.

A parte do programa que diz respeito à implementação das operações é na declaração do tipo enumerado `Op`, junto com as regras de avaliação e validez das operações em argumentos inteiros, na forma das funções `apply` e `valid`, respectivamente. A última peça dependente é a lista de operações disponíveis para combinação `ops`. Já que todo o restante do programa, especialmente os procedimentos de geração e avaliação de expressões, é interfaceado por esses três componentes (`apply`, `valid` e `ops`), nada mais deverá ser alterado.

Poucas linhas de código são precisas para incorporar a operação de exponenciação no sistema:

```haskell
-- Novo construtor para a operação de exponenciação
data Op =
    ...
  | Exp

-- A lista de operações é incrementada
ops :: [Op]
ops = [..., Exp]

-- Uma regra para exibição do operador é estabelecida
instance Show Op where
  ...
  show Exp = "^"

-- Um método de exponenciação em inteiros é providenciado
apply :: Op -> Int -> Int -> Int
...
apply Exp m n = m ^ n

-- Um método para validação da exponenciação é adicionado
-- Note que esta versão leva em conta a otimização para exclusão de expressões equivalentes
valid :: Op -> Int -> Int -> Bool
...
valid Exp m n = m /= 1 && n /= 1
```

## Letra b.

A partir de `solutions'`, que produz as expressões que avaliam exatamente no valor de alvo, podemos fazer uma busca em largura no domínio dos alvos atrás de uma lista de resultados que não seja vazia. Isso resolveria o problema de encontrar as soluções mais próximas de um dado alvo.

A implementação da busca em largura pode ser feita em duas etapas. Primeiro definiremos uma função que avalia as soluções dentro de um intervalo predefinido nos valores de alvo:

```haskell
solutionsInRange :: [Int] -> Int -> Int -> [Result]
solutionsInRange ns t r = [(e,t') | t' <- ts, e <- solutions' ns t']
  where
    lb = max (t-r) 1
    ts = [lb..(t+r)]
```

De posse de `solutionsInRange`, podemos testar cada tamanho de intervalo iterativamente, começando por 0 (que representa a solução exata no alvo), até encontrarmos uma lista de resultados não-vazia.

```haskell
nearestSolutions :: [Int] -> Int -> [Result]
nearestSolutions ns t = tryRange 0
  where
    tryRange :: Int -> [Result]
    tryRange r | not (null ss) = ss
               | otherwise     = tryRange (r+1)
      where
        ss = solutionsInRange ns t r
```

Por exemplo, ao executarmos `nearestSolutions [1,3,7,10,25,50] 831` na versão padrão do sistema (sem a operação de exponenciação), são retornados os resultados para os alvos 830 e 832 contendo as expressões que os produzem.

Vale notar que a função `solutionsInRange` testa todos os alvos no intervalo `[max (t-r) 1, t+r]`, o que inclui os valores intermediários. No entanto, pela característica da busca em largura feita por `nearestSolutions`, temos que não pode haver solução exata para eles, pois essa é a condição para a manutenção da iteração pelo espaço de busca. Logo, podemos realizar uma pequena otimização em `solutionsInRange` fazendo-a avaliar os resultados somente nos extremos do intervalo. Em termos de código, ela seria implementada simplesmente substituindo `ss = [lb..(t-r)]` por  `ss = [lb,t-r]`. Isso pouparia a repetição de `solutions'` em alvos já observados.

## Letra c.

Podemos nos aproveitar do sistema de classes de tipos do Haskell para satisfazermos a tarefa de ordenar, por um critério de complexidade, as expressões retornadas pelas nossas funções. Caso estabeleçamos regras que tornem `Expr`essões dados `Eq`uiparáveis e `Ord`enáveis, teríamos ao nosso dispor todo o arcabouço de funcionalidades da linguagem para realizar as mais variadas comparações entre expressões e, principalmente, acesso à função `sort` do _Prelude_, que ordena uma lista com itens ordenáveis.

Para "ensinar" o programa a comparar expressões, é importante que ele antes saiba comparar os elementos que as compõem. Os dados do tipo `Expr` são parametrizados por valores do tipo `Int`, no caso `Val Int`, e `Op`, em `App Op Expr Expr`. Já é estabelecido pela linguagem como comparar valores inteiros, apenas nos restando implantar essas noções no código para objetos do tipo `Op`. Faremos isso instanciando `Op` como pertencente às classes `Eq` e `Ord`:

```haskell
instance Eq Op where
  Add == Add = True
  Sub == Sub = True
  Mul == Mul = True
  Div == Div = True
  _   == _   = False

instance Ord Op where
  Add <= _   = True
  Sub <= Add = False
  Sub <= _   = True
  Mul <= Add = False
  Mul <= Sub = False
  Mul <= _   = True
  Div <= Div = True
  Div <= _   = False
```

Aqui usamos a ordenação intuitiva de operações `Add < Sub < Mul < Div`, tentando refletir o escalonamento do grau de complexidade. Essa ordenação é precisamente a mesma que teríamos caso incluíssemos `deriving (Eq, Ord)` na declaração de `Op`, mas a instanciação foi feita explicitamente para eliminar dependências estruturais (a saber, na ordem de declaração dos construtores) inesperadas.

Agora, partimos para a instanciação de `Expr`essões como coisas ordenáveis. Uma forma bastante intuitiva de determinar a complexidade de uma expressão aritmética perante a outra é analisando a discrepância entre seus comprimentos. Uma expressão muito extensa tende a ser mais complexa do que uma curta. Essa ideia de comprimento pode ser concretizada por meio da definição de uma função `exprLength` que o calcula. Diremos que o tamanho de uma expressão é o número de inteiros presentes nela.

```haskell
exprLength :: Expr -> Int
exprLength (Val _)     = 1
exprLength (App _ l r) = exprLength l + exprLength r
```

Valendo-se dessa "régua" para medição de complexidade de uma expressão, conseguimos instanciar o tipo `Expr` como desejávamos:

```haskell
instance Eq Expr where
  (Val m)     == (Val n)        = m == n
  (App o l r) == (App o' l' r') = (o,l,r) == (o',l',r')
  _           == _              = False

instance Ord Expr where
  (Val m)     <= (Val n)        = m <= n
  (App o l r) <= (App o' l' r') = (exprLength (App o l r) < exprLength (App o' l' r')) ||
                                  ((l,o,r) <= (l',o',r'))
  e1          <= e2             = exprLength e1 <= exprLength e2
```

Pouco há a ser discutido sobre as regras de igualdade. Uma expressão é igual à outra se são idênticas em seus parâmetros. Para a relação de `<=` foi aplicada uma cascata de comparações sobre características das expressões, em ordem de prioridade:

1. Tamanho da expressão;
2. Valor numérico;
3. Complexidade das operações.

Primeiramente, compara-se os tamanhos das expressões. Se o tamanho da primeira for menor que o da segunda, então a primeira é mais simples (em outras palavras, menor). Se os tamanhos forem iguais, ou elas são atômicas, do tipo `Val n`, situação na qual a complexidade é medida pela magnitude dos inteiros envolvidos; ou elas são aplicações, caso no qual as subexpressões à esquerda são comparadas recursivamente até atingir a forma atômica. Apenas se as subexpressões à esquerda forem constatadas iguais, compara-se os operadores e, em última instância, as subexpressões à direita.

Feitas as instanciações, o trabalho é terminado. Para ordenarmos as soluções fornecidas por `solutions'`, ou pelo método aproximado `nearestSolutions`, é suficiente uma aplicação de `sort` à lista de saída. Executando `sort (solutions' [1,2,3,4] 4)` como exemplo, temos:

```haskell
[
  4,
  1+3,
  2+(3-1),
  2*(3-1),
  ...
  ((4-3)+1)+2,
  ((4-3)+1)*2,
  ((4/2)+3)-1,
  ((4/2)-1)+3
]
```

Ouso dizer que é uma ordenação razoável em termos de complexidade.