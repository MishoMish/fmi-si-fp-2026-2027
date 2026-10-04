# Седмица 14 - Задачи за домашна работа

## Задача 1: Доказателство на законите

Докажете трите закона за монадите за:

1. `Maybe`
2. списъците (използвайте `xs >>= f = concatMap f xs`; за асоциативността може да ползвате без доказателство `concatMap g (concatMap f xs) = concatMap (concatMap g . f) xs`)

---

## Задача 2: Монада за журнал

```haskell
newtype Logger a = Logger (a, [String])

runLogger :: Logger a -> (a, [String])
tell :: String -> Logger ()
```

1. Напишете `Functor`, `Applicative` и `Monad` за `Logger` - при свързване журналите се слепват.
2. Напишете `factLog :: Integer -> Logger Integer`, която записва всяка стъпка.

```haskell
>>> runLogger (factLog 3)
(6,["fact 0 = 1","fact 1 = 1","fact 2 = 2","fact 3 = 6"])
```

> 💡 Тази монада е известна като `Writer`. Какво изискване трябва да има за типа на журнала, за да обобщим `[String]` до произволен тип `w`? (Седмица 13!)

---

## Задача 3: Конят на шахматната дъска

Като използвате `moves` от примерите:

1. Напишете `reachableIn :: Int -> Pos -> [Pos]` - позициите (без повторения), достижими с точно `n` хода.
2. Напишете `minMoves :: Pos -> Pos -> Int` - минималния брой ходове между две позиции.

```haskell
>>> minMoves (1, 1) (8, 8)
6
```

---

## Задача 4: Стекова машина със `State`

С `State` от теорията напишете операциите над стек:

```haskell
push :: Int -> State [Int] ()
pop  :: State [Int] (Maybe Int)
```

и функция `evalRPN :: String -> State [Int] (Maybe Int)`, която пресмята израз в обратен полски запис (Задача 5 от упражнението).

```haskell
>>> fst (runState (evalRPN "3 4 + 2 *") [])
Just 14
```

---

## Задача 5: Случайни числа със `State`

Генераторът на случайни числа от Седмица 12 трябваше да се предава ръчно: `(a, g1) = randomR (1, 6) g0`. Това е точно `State StdGen`!

1. Напишете `rollDie :: State StdGen Int`.
2. Чрез `replicateM` напишете `rollDice :: Int -> State StdGen [Int]`.
3. Сравнете с `threeDice` от теорията на Седмица 12.

---

## Задача 6: Парсер

Парсер е функция, която опитва да прочете стойност от началото на низ и връща стойността и остатъка:

```haskell
newtype Parser a = Parser { runParser :: String -> Maybe (a, String) }
```

1. Напишете `Functor`, `Applicative` и `Monad` за `Parser`.
2. Напишете основни парсери: `item :: Parser Char` (един символ), `sat :: (Char -> Bool) -> Parser Char`, `char :: Char -> Parser Char`, `many1 :: Parser a -> Parser [a]`.
3. Чрез тях напишете `number :: Parser Int` и `sumP :: Parser Int`, която разпознава суми като `"1+22+3"`.

```haskell
>>> runParser sumP "1+22+3abc"
Just (26,"abc")
>>> runParser sumP "x"
Nothing
```

> 💡 Това е основата на библиотеки като `parsec` и `megaparsec` - добра идея за учебен проект!
