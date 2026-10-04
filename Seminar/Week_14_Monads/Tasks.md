# Седмица 14 - Задачи за в час

## Задача 1: `do` ↔ `>>=`

Пренапишете с `>>=` и ламбди:

```haskell
f1 :: Maybe Int
f1 = do
  x <- Just 5
  y <- Just (x + 1)
  return (x * y)
```

Пренапишете с `do`:

```haskell
f2 :: [(Int, Char)]
f2 = [1, 2] >>= \n -> "ab" >>= \c -> return (n, c)

f3 :: IO ()
f3 = getLine >>= \s -> putStrLn (reverse s) >> putStrLn "готово"
```

---

## Задача 2: Верига от търсения

Дадени са три асоциативни списъка:

```haskell
people :: [(String, String)]     -- човек → град
people = [("ana", "Sofia"), ("ivan", "Plovdiv"), ("maria", "Ruse")]

cities :: [(String, Int)]        -- град → пощенски код
cities = [("Sofia", 1000), ("Plovdiv", 4000)]

zones :: [(Int, String)]         -- код → зона
zones = [(1000, "A"), (4000, "B")]
```

Напишете `zoneOf :: String -> Maybe String` веднъж с `>>=` и веднъж с `do`.

```haskell
>>> zoneOf "ivan"
Just "B"
>>> zoneOf "maria"
Nothing
>>> zoneOf "petar"
Nothing
```

---

## Задача 3: Зарове

Чрез списъчната монада напишете `dice :: Int -> Int -> [[Int]]` - всички начини да хвърлим `k` зара така, че сумата да е `target`.

```haskell
>>> dice 2 3
[[1,2],[2,1]]
>>> length (dice 2 7)
6
>>> length (dice 3 10)
27
```

> 💡 `replicateM k [1..6]` дава всички комбинации - защо?

---

## Задача 4: Изрази с деление

```haskell
data Expr = Lit Int | Add Expr Expr | Div Expr Expr

eval :: Expr -> Either String Int
```

`eval` връща `Left "деление на нула"` при деление на 0. Използвайте `do`.

```haskell
>>> eval (Div (Lit 10) (Add (Lit 3) (Lit 2)))
Right 2
>>> either putStrLn print (eval (Div (Lit 1) (Add (Lit 1) (Lit (-1)))))
деление на нула
```

---

## Задача 5: Обратен полски запис

Чрез `foldM` в монадата `Maybe` напишете `rpn :: String -> Maybe Int`, която пресмята израз в обратен полски запис. При невалиден израз (недостатъчно операнди, непознат символ) връща `Nothing`.

```haskell
>>> rpn "3 4 + 2 *"
Just 14
>>> rpn "1 +"
Nothing
```

---

## Задача 6: Собствена монада

```haskell
data Option a = None | Some a deriving Show
```

Напишете инстанции `Functor`, `Applicative` и `Monad` за `Option` (без да използвате `Maybe`). Проверете законите за няколко примера.
