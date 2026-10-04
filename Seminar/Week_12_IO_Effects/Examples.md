# Седмица 12 - Примери

## Пример 1: Първа интерактивна програма

```haskell
main :: IO ()
main = do
  putStrLn "Как се казваш?"
  name <- getLine
  putStrLn "На колко години си?"
  age <- readLn :: IO Int
  putStrLn ("Здравей, " ++ name ++ "! След 10 години ще си на " ++ show (age + 10) ++ ".")
```

```
$ runghc Hello.hs
Как се казваш?
Мария
На колко години си?
20
Здравей, Мария! След 10 години ще си на 30.
```

---

## Пример 2: Поточна обработка с `interact`

```haskell
-- Номерира редовете на входа
main :: IO ()
main = interact (unlines . zipWith number [1 :: Int ..] . lines)
  where number i line = show i ++ ": " ++ line
```

```
$ printf "първи\nвтори\nтрети\n" | runghc Number.hs
1: първи
2: втори
3: трети
```

```haskell
-- Статистика за текст: редове, думи, символи (като wc)
main :: IO ()
main = do
  text <- getContents
  let ls = length (lines text)
      ws = length (words text)
      cs = length text
  putStrLn (show ls ++ " " ++ show ws ++ " " ++ show cs)
```

> 💡 Цялата логика е в **чисти** функции (`lines`, `words`, `length`); `IO` е само тънък слой за вход и изход.

---

## Пример 3: Файлове

```haskell
import Data.Char (toUpper)

main :: IO ()
main = do
  writeFile "notes.txt" "първи ред\nвтори ред\n"
  appendFile "notes.txt" "трети ред\n"
  content <- readFile "notes.txt"
  putStrLn ("Редове: " ++ show (length (lines content)))
  writeFile "NOTES.txt" (map toUpper content)
```

---

## Пример 4: Безопасен калкулатор

```haskell
import Text.Read (readMaybe)
import System.IO

calculate :: Double -> String -> Double -> Either String Double
calculate x "+" y = Right (x + y)
calculate x "-" y = Right (x - y)
calculate x "*" y = Right (x * y)
calculate _ "/" 0 = Left "деление на нула"
calculate x "/" y = Right (x / y)
calculate _ op  _ = Left ("непозната операция: " ++ op)

evalLine :: String -> Either String Double
evalLine line = case words line of
  [a, op, b] -> case (readMaybe a, readMaybe b) of
    (Just x, Just y) -> calculate x op y
    _                -> Left "невалидно число"
  _ -> Left "очаквам: <число> <операция> <число>"

loop :: IO ()
loop = do
  putStr "> "
  line <- getLine
  if line == "quit"
    then putStrLn "Довиждане!"
    else do
      putStrLn (either ("Грешка: " ++) show (evalLine line))
      loop

main :: IO ()
main = do
  hSetBuffering stdout NoBuffering
  loop
```

```
> 3 + 4
7.0
> 10 / 0
Грешка: деление на нула
> 2 ^ 3
Грешка: непозната операция: ^
> abc
Грешка: очаквам: <число> <операция> <число>
> quit
Довиждане!
```

> 💡 `evalLine` е **чиста** - лесно се тества в GHCi без вход/изход. `loop` е рекурсивно действие - така правим „цикъл“ в `IO`.

---

## Пример 5: Случайни числа - чисто и в `IO`

```haskell
import System.Random
import Data.List (sort, group)

-- Чисто: 6000 хвърляния на зар със зададено семе
rolls :: Int -> [Int]
rolls seed = take 6000 (randomRs (1, 6) (mkStdGen seed))

histogram :: [Int] -> [(Int, Int)]
histogram = map (\g -> (head g, length g)) . group . sort

main :: IO ()
main = do
  mapM_ print (histogram (rolls 42))
  seed <- randomRIO (1, 1000000)        -- различно семе всеки път
  print (take 10 (rolls seed))
```

```
(1,995)
(2,1005)
(3,1007)
(4,1031)
(5,1010)
(6,952)
[...]          ← 10 случайни числа, различни при всяко изпълнение
```

> 💡 `rolls 42` дава **винаги** същия резултат - удобно за възпроизводими експерименти. Случайността идва само от `randomRIO` в `IO`.

---

## Пример 6: Игра „Познай числото“

```haskell
import System.Random (randomRIO)
import Text.Read (readMaybe)
import System.IO

play :: Int -> Int -> IO ()
play secret attempts = do
  putStr "Твоето предположение: "
  input <- getLine
  case readMaybe input of
    Nothing -> do
      putStrLn "Моля, въведи число."
      play secret attempts
    Just guess
      | guess < secret -> putStrLn "По-голямо!" >> play secret (attempts + 1)
      | guess > secret -> putStrLn "По-малко!"  >> play secret (attempts + 1)
      | otherwise      -> putStrLn ("Позна от " ++ show attempts ++ " опита!")

main :: IO ()
main = do
  hSetBuffering stdout NoBuffering
  secret <- randomRIO (1, 100)
  putStrLn "Намислих число от 1 до 100."
  play secret 1
```

> 💡 `a >> b` изпълнява `a`, после `b` - същото като `do { a; b }` на един ред.

---

## Пример 7: Изключения

```haskell
import Control.Exception
import System.IO

main :: IO ()
main = do
  -- Липсващ файл
  r1 <- try (readFile "missing.txt") :: IO (Either IOException String)
  case r1 of
    Left e  -> putStrLn ("Не мога да прочета файла: " ++ show e)
    Right s -> putStr s

  -- Деление на нула - evaluate принуждава оценяването в try
  r2 <- try (evaluate (1 `div` 0 :: Int)) :: IO (Either ArithException Int)
  print r2

  -- Без evaluate изключението „изтича“ извън try
  r3 <- try (return (1 `div` 0 :: Int)) :: IO (Either ArithException Int)
  putStrLn (either (const "хванато") (const "НЕ е хванато") r3)

  -- catch с обработчик
  n <- evaluate (length [1 .. 10]) `catch` \e -> do
         putStrLn ("Грешка: " ++ show (e :: SomeException))
         return 0
  print n
```

```
Не мога да прочета файла: missing.txt: openFile: does not exist (No such file or directory)
Left divide by zero
НЕ е хванато
10
```
