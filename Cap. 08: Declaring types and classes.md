# Exercício 1:

Seguindo a mesma lógica da soma recursiva em números `Nat`urais, a função `mult` pode ser escrita assim:

```haskell
mult :: Nat -> Nat -> Nat
mult Zero     _ = Zero
mult (Succ m) n = add n (mult m n)
```

# Exercício 2:

A operação `compare` é conveniente no sentido de que demanda, possivelmente, apenas uma comparação entre seus dois operandos para determinação de sua ordenação relativa. Isso dispensa a necessidade de duas chamadas comparativas explícitas de igualdade `==` e menoridade `<`, potencialmente economizando alguns passos de execução. 

Um tipo de dados `Ord`inal em que isso representa um ganho de eficiência é o de listas contendo itens comparáveis entre si. Em uma comparação de igualdade entre duas listas ocorrem na ordem de `n` operações, sendo `n` o tamanho da menor lista. Para a checagem de uma lista ser menor que outra lexicograficamente, também são precisos até `n` passos, sendo `n` o comprimento da menor lista. Portanto, para chegar no caso em que a lista `x` é maior que a lista `y`, na versão original de `occurs`, são realizadas por volta de `2n` comparações item-a-item. Lançando mão de `compare`, dá para determinar a ordenação em uma única passagem pelas listas, em até `n` passos. Dessa maneira, diminuímos pela metade, em média, a quantidade de comparações feitas para determinação da próxima chamada recursiva.

A função `occurs`, que procura um elemento em uma árvore binária de busca, pode ser reimplementada com `compare`:

```haskell
occurs :: Ord a => a -> Tree a -> Bool
occurs x (Leaf y)     = x == y
occurs x (Node l y r) =
  case compare x y of
    LT -> occurs x l
	EQ -> True
	GT -> occurs x r
```

# Exercício 3:

Para detectar o balanceamento de árvores cujos valores atômicos encontram-se apenas em suas folhas, é de grande valia a definição de uma função auxiliar que calcula o número de folhas em uma árvore desse tipo:

```haskell
countLeaves :: Tree a -> Int
countLeaves (Leaf _)   = 1
countLeaves (Node l r) = countLeaves l + countLeaves r
```

Para uma árvore ser balanceada, ou ela deve ser uma folha (balanceada trivialmente), ou um nó cuja diferença absoluta entre o número de folhas na subárvore esquerda e o na subárvore direita não excede 1. O predicado `balanced` que faz essa testagem é:

```haskell
balanced :: Tree a -> Bool
balanced (Leaf _) = True
balanced (Node l r) = 
  (abs (countLeaves l - countLeaves r)) <= 1
```

# Exercício 4:

Uma função que, dada uma lista não vazia de itens, gera uma árvore balanceada contendo-os, pode ser dada por:

```haskell
balance :: [a] -> Tree a
balance []  = error "Cannot balance empty list!"
balance [x] = Leaf x
balance xs  = Node (balance l) (balance r)
  where
    (l,r) = splitAt (length xs `div` 2) xs
```

# Exercício 5:

Vamos assumir um tipo de dado recursivo `Expr` que corresponde a expressões de dois tipos: uma aplicação de operação unária a um inteiro `UN Int`; e uma aplicação de operação binária a duas subexpressões `BIN Expr Expr`. Os nomes foram alterados, com relação aos apontados no enunciado, para levar em conta a generalização dos significados de `Val` e `Add`.

```haskell
data Expr = UN Int | BIN Expr Expr
```

A função `folde`, que avalia uma expressão aplicando operadores `u` e `b`, pode ser descrita por:

```haskell
folde :: (Int -> a) -> (a -> a -> a) -> Expr -> a
folde u b (UN n)      = u n
folde u b (BIN e1 e2) = b (folde u b e1) (folde u b e2)
```

Para além do seu significado recursivo, `folde u b` pode ser entendida pela mudança que promove na estrutura de uma expressão, substituindo aparições de `UN` por `u` e aparições de `BIN` por `b`:

```haskell
  folde u b (BIN (UN x) (BIN (UN y) (UN z)))
= {aplicação de `folde u b` a 
   (BIN (UN x) (BIN (UN y) (UN z)))}
  b (folde u b (UN x)) (folde u b (BIN (UN y) (UN z)))
= {aplicação de `folde u b` a (UN x)}
  b (u x) (folde u b (BIN (UN y) (UN z)))
= {aplicação de `folde u b` a (BIN (UN y) (UN z))}
  b (u x) (b (folde u b (UN y)) (folde u b (UN z)))
= {aplicação de `folde u b` a (UN y)}
  b (u x) (b (u y) (folde u b (UN z)))
= {aplicação de `folde u b` a (UN z)}
  b (u x) (b (u y) (u z))
```

# Exercício 6:

Note que podemos interpretar as operações descritas por `UN` e `BIN` como quisermos, em particular na forma de operações de identidade e soma de inteiros, respectivamente. Logo, temos um mecanismo para avaliação de expressões no contexto original, de expressões compostas de somas de expressões com valores inteiros:

```haskell
eval :: Expr -> Int
eval = folde id (+)
```

Se preferíssemos tratar tais expressões como multiplicações de quadrados de inteiros, por exemplo, bastaria externarmos esse desejo nos argumentos passados para `folde`:

```haskell
eval' :: Expr -> Int
eval' = folde (^2) (*)
```

Um exemplo prático dessa generalização é para a contagem de valores em uma expressão. Visualizando cada inteiro como 1 unidade de valor e a operação de soma como a agregadora de duas expressões, temos um meio muito simples de contar quantos inteiros existem em uma expressão:

```haskell
size :: Expr -> Int
size = folde (const 1) (+)
```

# Exercício 7:

As declarações de instâncias da classe `Eq` devem, no mínimo, delimitar um método para checagem de igualdade entre dois elementos daquele tipo. Nesse sentido, as instanciações dos tipos `Maybe a` e `[a]` como `Eq`uiparáveis podem ser feitas assim:

```haskell
instance Eq a => Eq (Maybe a) where
  Nothing == Nothing = True
  Just x  == Just y  = x == y
  _       == _       = False

instance Eq a => Eq [a] where
  []     == []     = True
  (_:_)  == []     = False
  []     == (_:_)  = False
  (x:xs) == (y:ys) = (x == y) && (xs == ys)
```

# Exercício 8:

Para incorporar as operações de disjunção e equivalência lógicas em nosso sistema simbólico, é suficiente que definamos três características de cada uma das expressões correspondentes:

- Um construtor, na definição do tipo `Prop`;
- Um método de avaliação, no corpo da função `eval`;
- Um método para obtenção das variáveis, no corpo da função `vars`.

De posse dessas três, o restante do programa se encarregará de determinar as tabelas de substituição e processar todos os valores-verdade das proposições estendidas.

As atualizações necessárias no código são as seguintes:

```haskell
-- Adicionar os dois construtores
data Prop =
    ...
  | Or Prop Prop
  | Iff Prop Prop

-- Adicionar cláusulas para avaliação
eval :: Subst -> Prop -> Bool
...
eval s (Or p q)    = eval s p || eval s q
eval s (Iff p q)   = eval s p == eval s q

-- Adicionar cláusulas para contagem de variáveis
vars :: Prop -> [Char]
...
vars (Or p q)    = vars p ++ vars q
vars (Iff p q)   = vars p ++ vars q
```

# Exercício 9:

A máquina abstrata, apresentada na seção 8.7, é limitada à avaliação de expressões aritméticas compostas de valores inteiros e adições. O coração do sistema reside em três definições: a das operações feitas pela máquina, enumeradas no tipo de dado `Op`; a descrição da avaliação de cada tipo de expressão, nos casos cobertos pelas equações de `eval`; e a descrição da execução da máquina sobre cada operação retirada da pilha de controle, em `exec`.

A avaliação de uma expressão através da máquina abstrata se dá por recursão mútua entre as funções `eval` e `exec`. A função `eval` tem o papel de "agendar" a avaliação de expressões mais à direita na pilha de controle enquanto continua avaliando as mais à esquerda, até que uma expressão atômica seja alcançada. A partir daí, `exec` é chamada com o objetivo de ir desempilhando as operações agendadas, aplicando-as sucessivamente no inteiro determinado. Caso uma operação aritmética envolvendo um valor inteiro conhecido seja a operação requisitada na vez, a máquina atualiza seu estado aplicando a operação no inteiro em sua memória e chamando a próxima tarefa na pilha. Por outro lado, caso a operação atual seja uma avaliação de expressão previamente agendada, sob o contexto de um operador binário, então agenda-se a operação em si com o número na memória corrente da máquina, e aplica-se `eval` à expressão.

Portanto, tendo em vista a necessidade de agendamento da avaliação de uma expressão sob uma operação e de agendamento da operação em si, além, é claro, da declaração do construtor associado à expressão que descreve a operação, três novos construtores devem ser inclusos no código:

- `Mult Expr Expr`, para introdução da `Expr`essão de multiplicação;
- `MULT Int`, que representa a `Op`eração de multiplicação computada pela máquina;
- `EVAL_MULT Expr`, que representa a `Op`eração agendada de avaliação de uma expressão do lado direito de uma multiplicação.

Ademais, cláusulas devem ser adicionadas às funções `eval` e `exec` explicitando como lidar com esses novos dados. A saber, `eval` deve ser capaz de agendar a avaliação da expressão do lado direito para posterior multiplicação, e `exec` deve saber requisitá-la no momento em que for tirada da pilha de controle, postergando o produto pelo número em memória. Também é preciso que a máquina saiba atualizar seu estado, quando se deparar com uma operação de controle de multiplicação.

As mudanças no programa são exibidas a seguir:

```haskell
-- Novo construtor para expressão Mult
data Expr =
    Val  Int
  | Add  Expr Expr
  | Mult Expr Expr

-- Novos construtores para operações de máquina EVAL_MULT e MULT
data Op = 
    EVAL_ADD Expr 
  | ADD  Int
  | EVAL_MULT Expr
  | MULT Int

-- Nova cláusula para avaliação de uma multiplicação
eval :: Expr -> Cont -> Int
eval (Val n)      c = exec c n
eval (Add e1 e2)  c = eval e1 (EVAL_ADD e2 : c)
eval (Mult e1 e2) c = eval e1 (EVAL_MULT e2 : c)

-- Novas cláusulas para execução de uma avaliação de expressão sob multiplicação e de operação de multiplicação na memória
exec :: Cont -> Int -> Int
exec []                n = n
exec (EVAL_ADD e : c)  n = eval e (ADD n : c)
exec (EVAL_MULT e : c) n = eval e (MULT n : c)
exec (ADD m : c)       n = exec c (m+n)
exec (MULT m : c)      n = exec c (m*n)
```

Veja que o construtor `EVAL` foi renomeado para `EVAL_ADD`, para ficar consistente com o critério usado na nomeação de `EVAL_MULT`.