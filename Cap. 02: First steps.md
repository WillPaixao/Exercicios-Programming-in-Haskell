# Exercício 1:

Apenas experimentar os exemplos no GHCi.

# Exercício 2:

Explicitando os parênteses nas expressões:

```haskell
> 2^3*4   -- (2^3)*4
> 2*3+4*5 -- (2*3)+(4*5)
> 2+3*4^5 -- 2+(3*(4^5))
```

# Exercício 3:

O script em questão é:

```haskell
N = a 'div' length xs
    where
	   a = 10
	  xs = [1,2,3,4,5]
```

Os três erros sintáticos presentes no trecho são:

- O nome da variável `N` começa com letra maiúscula;
- `div` está envolto de aspas simples (não aspas reversas);
- `xs` não está no mesmo nível de indentação que `a`.

Corrigindo esses problemas, ficamos com o _script_:

```haskell
n = a `div` length xs
    where
	  a  = 10
	  xs = [1,2,3,4,5]
```

# Exercício 4:

A função `last`, da biblioteca `Prelude` padrão, retorna o último elemento de uma lista não-vazia. Poderíamos defini-la das seguintes maneiras:

```haskell
-- Opção raiz:
last' (x:[]) = x
last' (x:(x':xs)) = last' (x':xs)

-- Opção nutella:
last'' xs = head (reverse xs)

-- Opção nutella 2:
last''' xs = 
  drop (length xs - 1) xs !! 0
```

# Exercício 5:

A função `init`, do `Prelude`, recebe uma lista não-vazia e retorna uma cópia sem o último elemento. Eis algumas formas de defini-la:

```haskell
-- Opção raiz:
init' (x:[]) = []
init' (x:(x':xs)) = 
  x : init' (x':xs)

-- Opção nutella:
init'' xs = 
  take (length xs - 1) xs

-- Opção nutella 2:
init''' xs =
  reverse (tail (reverse xs))
```