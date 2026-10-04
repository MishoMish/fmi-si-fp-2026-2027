# Седмица 9 - Примери

## Пример 1: Реимплементация на стандартни функции

```haskell
myLength :: [a] -> Int
myLength []     = 0
myLength (_:xs) = 1 + myLength xs

myAppend :: [a] -> [a] -> [a]
myAppend []     ys = ys
myAppend (x:xs) ys = x : myAppend xs ys

myReverse :: [a] -> [a]
myReverse xs = go xs []
  where
    go []     acc = acc
    go (y:ys) acc = go ys (y : acc)

myTake :: Int -> [a] -> [a]
myTake n _ | n <= 0 = []
myTake _ []         = []
myTake n (x:xs)     = x : myTake (n - 1) xs

myZip :: [a] -> [b] -> [(a, b)]
myZip (x:xs) (y:ys) = (x, y) : myZip xs ys
myZip _      _      = []
```

```haskell
ghci> myAppend [1, 2] [3, 4]
[1,2,3,4]
ghci> myReverse "hello"
"olleh"
ghci> myTake 2 "abc"
"ab"
ghci> myZip [1, 2, 3] "ab"
[(1,'a'),(2,'b')]
```

> 💡 Сравнете с Пример 2 от Седмица 4 - структурата е същата, но образците правят кода по-кратък и по-четим.

---

## Пример 2: Изразяване чрез `foldr` и `foldl`

```haskell
mySum :: Num a => [a] -> a
mySum = foldr (+) 0

myLength' :: [a] -> Int
myLength' = foldr (\_ acc -> acc + 1) 0

mapViaFoldr :: (a -> b) -> [a] -> [b]
mapViaFoldr f = foldr (\x acc -> f x : acc) []

filterViaFoldr :: (a -> Bool) -> [a] -> [a]
filterViaFoldr p = foldr (\x acc -> if p x then x : acc else acc) []

reverseViaFoldl :: [a] -> [a]
reverseViaFoldl = foldl (flip (:)) []

-- Число от цифри: [1,2,3] -> 123
fromDigits :: [Int] -> Int
fromDigits = foldl (\acc d -> acc * 10 + d) 0
```

```haskell
ghci> mySum [1..100]
5050
ghci> mapViaFoldr (* 2) [1, 2, 3]
[2,4,6]
ghci> reverseViaFoldl [1, 2, 3]
[3,2,1]
ghci> fromDigits [1, 2, 3]
123
```

> 💡 Сравнете `fromDigits` със схемата на Хорнер (`make-poly` от Тест 1) - същата идея.

---

## Пример 3: Обработка на данни

```haskell
type Student = (String, [Int])     -- (име, оценки); type дава синоним на тип

students :: [Student]
students =
  [ ("Иван",  [5, 6, 4, 5])
  , ("Мария", [6, 6, 6, 5])
  , ("Петър", [3, 4, 3, 2])
  , ("Елена", [5, 5, 6, 6])
  ]

average :: [Int] -> Double
average xs = fromIntegral (sum xs) / fromIntegral (length xs)

goodStudents :: [Student] -> [String]
goodStudents = map fst . filter (\(_, grades) -> average grades >= 5.0)

averages :: [Student] -> [(String, Double)]
averages ss = [(name, average gs) | (name, gs) <- ss]
```

```haskell
ghci> goodStudents students
["Иван","Мария","Елена"]
ghci> averages students
[("Иван",5.0),("Мария",5.75),("Петър",3.0),("Елена",5.5)]
ghci> maximum (map (average . snd) students)
5.75
```

> ⚠️ GHCi отпечатва кирилица чрез `show` като кодове: `"\1048\1074\1072\1085"`. За да видите буквите, използвайте `putStrLn` или `mapM_ putStrLn`. В примерите показваме четимата форма.

---

## Пример 4: List comprehensions

```haskell
-- Делители на число
divisors :: Int -> [Int]
divisors n = [d | d <- [1..n], n `mod` d == 0]

-- Прости числа до n
primesUpTo :: Int -> [Int]
primesUpTo n = [p | p <- [2..n], divisors p == [1, p]]

-- Питагорови тройки с хипотенуза до n
pythagorean :: Int -> [(Int, Int, Int)]
pythagorean n = [(a, b, c) | c <- [1..n], b <- [1..c], a <- [1..b], a^2 + b^2 == c^2]

-- Декартово произведение
cartesian :: [a] -> [b] -> [(a, b)]
cartesian xs ys = [(x, y) | x <- xs, y <- ys]
```

```haskell
ghci> divisors 12
[1,2,3,4,6,12]
ghci> primesUpTo 30
[2,3,5,7,11,13,17,19,23,29]
ghci> pythagorean 15
[(3,4,5),(6,8,10),(5,12,13),(9,12,15)]
ghci> cartesian [1, 2] "ab"
[(1,'a'),(1,'b'),(2,'a'),(2,'b')]
```

---

## Пример 5: Обработка на текст

```haskell
import Data.Char (toUpper, isAlpha, toLower)

-- Главна първа буква на всяка дума
capitalize :: String -> String
capitalize = unwords . map cap . words
  where
    cap []     = []
    cap (c:cs) = toUpper c : cs

-- Брой думи с дължина поне n
countLongWords :: Int -> String -> Int
countLongWords n = length . filter ((>= n) . length) . words

-- Палиндром (без интервали и главни букви)
isPalindrome :: String -> Bool
isPalindrome s = clean == reverse clean
  where clean = map toLower (filter isAlpha s)
```

```haskell
ghci> capitalize "hello big world"
"Hello Big World"
ghci> countLongWords 4 "the quick brown fox jumps"
3
ghci> isPalindrome "A man a plan a canal Panama"
True
```

> 💡 `import Data.Char (...)` се пише в **началото** на файла. В GHCi: `ghci> import Data.Char`.

---

## Пример 6: Безточков стил и конвейери

```haskell
-- С аргумент
sumOfOddSquares xs = sum (map (^ 2) (filter odd xs))

-- С $
sumOfOddSquares' xs = sum $ map (^ 2) $ filter odd xs

-- Безточков (point-free) - композиция на функции
sumOfOddSquares'' :: [Int] -> Int
sumOfOddSquares'' = sum . map (^ 2) . filter odd
```

```haskell
ghci> sumOfOddSquares'' [1..10]
165
```

Чете се **отдясно наляво**: филтрирай нечетните, повдигни на квадрат, сумирай. Аналогично на конвейера от Пример 5 в Седмица 4, но без вложени скоби.
