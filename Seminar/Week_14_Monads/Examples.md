# Седмица 14 - Примери

## Пример 1: От вложени `case` към `>>=` и `do`

```haskell
import Text.Read (readMaybe)

type Env = [(String, String)]

-- Вариант 1: вложени case
process1 :: Env -> Maybe Int
process1 env = case lookup "x" env of
  Nothing  -> Nothing
  Just str -> case readMaybe str of
    Nothing -> Nothing
    Just n  -> if n > 0 then Just n else Nothing

-- Вариант 2: >>=
process2 :: Env -> Maybe Int
process2 env = lookup "x" env >>= readMaybe >>= positive
  where positive n = if n > 0 then Just n else Nothing

-- Вариант 3: do
process3 :: Env -> Maybe Int
process3 env = do
  str <- lookup "x" env
  n   <- readMaybe str
  if n > 0 then return n else Nothing
```

```haskell
ghci> map process3 [[("x", "42")], [("x", "abc")], [("y", "1")], [("x", "-5")]]
[Just 42,Nothing,Nothing,Nothing]
```

---

## Пример 2: Безопасен достъп до матрица

```haskell
safeIndex :: [a] -> Int -> Maybe a
safeIndex xs i
  | i < 0 || i >= length xs = Nothing
  | otherwise               = Just (xs !! i)

matrixAt :: [[a]] -> Int -> Int -> Maybe a
matrixAt m row col = do
  r <- safeIndex m row
  safeIndex r col

-- Сума на диагонала, ако всички елементи съществуват
diagonalSum :: [[Int]] -> Maybe Int
diagonalSum m = sum <$> mapM (\i -> matrixAt m i i) [0 .. length m - 1]
```

```haskell
ghci> let m = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
ghci> matrixAt m 1 2
Just 6
ghci> matrixAt m 5 0
Nothing
ghci> diagonalSum m
Just 15
ghci> diagonalSum [[1, 2], [3]]
Nothing
```

---

## Пример 3: `Either` - банкова сметка

```haskell
type Balance = Int

withdraw :: Int -> Balance -> Either String Balance
withdraw amount balance
  | amount <= 0      = Left ("невалидна сума: " ++ show amount)
  | amount > balance = Left ("недостатъчна наличност: " ++ show balance)
  | otherwise        = Right (balance - amount)

deposit :: Int -> Balance -> Either String Balance
deposit amount balance
  | amount <= 0 = Left ("невалидна сума: " ++ show amount)
  | otherwise   = Right (balance + amount)

transactions :: Balance -> Either String Balance
transactions start = do
  b1 <- deposit 100 start
  b2 <- withdraw 30 b1
  withdraw 50 b2

-- Същото с Клайсли композиция
transactions' :: Balance -> Either String Balance
transactions' = deposit 100 >=> withdraw 30 >=> withdraw 50
```

```haskell
ghci> transactions 0
Right 20
ghci> either putStrLn print (withdraw 500 100)
недостатъчна наличност: 100
ghci> foldM (flip withdraw) 100 [10, 20, 30]
Right 40
```

> 💡 `>=>` е от `Control.Monad`. Сравнете `deposit 100 >=> withdraw 30` с композицията `(.)` - същата идея, но за функции `a -> m b`.

---

## Пример 4: Списъчната монада - ходове на коня

```haskell
import Control.Monad (guard)

type Pos = (Int, Int)

moves :: Pos -> [Pos]
moves (c, r) = do
  (dc, dr) <- [(1, 2), (2, 1), (2, -1), (1, -2), (-1, -2), (-2, -1), (-2, 1), (-1, 2)]
  let (c', r') = (c + dc, r + dr)
  guard (c' `elem` [1 .. 8] && r' `elem` [1 .. 8])
  return (c', r')

-- Всички позиции, достижими с точно 3 хода
in3 :: Pos -> [Pos]
in3 start = return start >>= moves >>= moves >>= moves

canReachIn3 :: Pos -> Pos -> Bool
canReachIn3 from to = to `elem` in3 from
```

```haskell
ghci> moves (1, 1)
[(2,3),(3,2)]
ghci> canReachIn3 (6, 2) (6, 1)
True
ghci> canReachIn3 (6, 2) (7, 3)
False
```

> 💡 `>>=` за списъци „разклонява“ изчислението по всички възможности, а `guard` отрязва невалидните клонове. Това е търсене с връщане (backtracking) - без нито един ред код за връщането!

---

## Пример 5: Разгъване на `do`

```haskell
example :: Maybe (Int, Int)
example = do
  x <- Just 3
  let y = x * 2
  z <- Just (y + 1)
  return (x, z)
```

Компилаторът го превежда до:

```haskell
example' :: Maybe (Int, Int)
example' =
  Just 3 >>= \x ->
    let y = x * 2 in
      Just (y + 1) >>= \z ->
        return (x, z)
```

```haskell
ghci> example
Just (3,7)
ghci> example == example'
True
```

---

## Пример 6: `State` - номериране на върховете на дърво

С `State` от теорията:

```haskell
data Tree a = Leaf | Node (Tree a) a (Tree a) deriving Show

fresh :: State Int Int
fresh = do
  n <- get
  put (n + 1)
  return n

-- Замества всеки елемент с двойка (пореден номер, елемент) - inorder
label :: Tree a -> State Int (Tree (Int, a))
label Leaf = return Leaf
label (Node l x r) = do
  l' <- label l
  n  <- fresh
  r' <- label r
  return (Node l' (n, x) r')
```

```haskell
ghci> let t = Node (Node Leaf 'a' Leaf) 'b' (Node Leaf 'c' Leaf)
ghci> fst (runState (label t) 1)
Node (Node Leaf (1,'a') Leaf) (2,'b') (Node Leaf (3,'c') Leaf)
```

> 💡 Без `State` трябваше ръчно да подаваме и връщаме брояча при всяко рекурсивно извикване: `label :: Int -> Tree a -> (Tree (Int, a), Int)`. Монадата „скрива“ това предаване.

---

## Пример 7: Проверка на законите

```haskell
f :: Int -> Maybe Int
f x = if x > 0 then Just (x * 2) else Nothing

g :: Int -> Maybe Int
g x = if even x then Just (x `div` 2) else Nothing

leftId, rightId, assoc :: Int -> Bool
leftId a  = (return a >>= f) == f a
rightId a = (Just a >>= return) == Just a
assoc a   = ((Just a >>= f) >>= g) == (Just a >>= (\x -> f x >>= g))
```

```haskell
ghci> all leftId [-5 .. 5] && all rightId [-5 .. 5] && all assoc [-5 .. 5]
True
```
