# Седмица 11 - Примери

## Пример 1: „Генерирай и избери“

Мързеливостта позволява да опишем **безкрайно** пространство от кандидати и да вземем само нужните:

```haskell
-- Най-малкото число, кратно на 7, 11 и 13
smallest :: Int
smallest = head [x | x <- [1..], x `mod` 7 == 0, x `mod` 11 == 0, x `mod` 13 == 0]

-- Първите три съвършени числа
perfects :: [Int]
perfects = take 3 [n | n <- [1..], perfect n]
  where perfect n = sum [d | d <- [1..n-1], n `mod` d == 0] == n

-- Питагорови тройки - без горна граница!
triples :: [(Int, Int, Int)]
triples = [(a, b, c) | c <- [1..], b <- [1..c], a <- [1..b], a^2 + b^2 == c^2]
```

```haskell
ghci> smallest
1001
ghci> perfects
[6,28,496]
ghci> take 5 triples
[(3,4,5),(6,8,10),(5,12,13),(9,12,15),(8,15,17)]
```

> ⚠️ Внимавайте с реда на генераторите: `[(a, b, c) | a <- [1..], b <- [1..], c <- [1..], ...]` никога няма да намери нищо - `c` расте безкрайно при `a = 1, b = 1`. Безкрайният генератор трябва да е **най-външният**, а вътрешните - крайни.

---

## Пример 2: Безкрайни редици с `iterate` и `scanl`

```haskell
-- Триъгълникът на Паскал: всеки ред се получава от предишния
pascal :: [[Integer]]
pascal = iterate (\row -> zipWith (+) (0 : row) (row ++ [0])) [1]

-- Триъгълни числа: частични суми на 1, 2, 3, ...
triangulars :: [Integer]
triangulars = scanl1 (+) [1..]

-- Степени на 2
powersOf2 :: [Integer]
powersOf2 = iterate (* 2) 1
```

```haskell
ghci> take 6 pascal
[[1],[1,1],[1,2,1],[1,3,3,1],[1,4,6,4,1],[1,5,10,10,5,1]]
ghci> pascal !! 10
[1,10,45,120,210,252,210,120,45,10,1]
ghci> take 8 triangulars
[1,3,6,10,15,21,28,36]
ghci> length (takeWhile (< 10^100) powersOf2)
333
```

> 💡 `scanl f z xs` е като `foldl`, но връща **всички** междинни акумулатори: `scanl (+) 0 [1,2,3]` → `[0,1,3,6]`. Това е `partial-sums` от Седмица 7.

---

## Пример 3: Прости числа - по-ефективно

Решетото от теорията проверява делимост с **всички** предишни прости числа. По-добре е да проверяваме само до $\sqrt{n}$ - като използваме **самия** списък `primes`:

```haskell
primes :: [Int]
primes = 2 : filter isPrime [3, 5 ..]
  where
    isPrime n = all (\p -> n `mod` p /= 0)
                    (takeWhile (\p -> p * p <= n) primes)
```

```haskell
ghci> primes !! 999
7919
```

> 💡 `primes` е дефиниран чрез `isPrime`, която използва `primes`. Това работи, защото за проверка на `n` са нужни само прости числа до $\sqrt{n}$ - те вече са пресметнати.

---

## Пример 4: Числа на Хаминг

Директен превод на Пример 7 от Седмица 7:

```haskell
hamming :: [Integer]
hamming = 1 : merge (map (* 2) hamming) (merge (map (* 3) hamming) (map (* 5) hamming))
  where
    merge xx@(x:xs) yy@(y:ys)
      | x < y     = x : merge xs yy
      | x > y     = y : merge xx ys
      | otherwise = x : merge xs ys
```

```haskell
ghci> take 15 hamming
[1,2,3,4,5,6,8,9,10,12,15,16,18,20,24]
```

Сравнете с версията на Racket - кодът е почти същият, но без `stream-cons`, `stream-first` и `stream-rest`.

---

## Пример 5: Всички положителни рационални числа

Можем да **изброим** всички двойки $(p, q)$ с $\gcd(p, q) = 1$ - по диагонали $p + q = s$:

```haskell
rationals :: [(Int, Int)]
rationals = [(p, q) | s <- [2..], p <- [1 .. s - 1], let q = s - p, gcd p q == 1]
```

```haskell
ghci> take 10 rationals
[(1,1),(1,2),(2,1),(1,3),(3,1),(1,4),(2,3),(3,2),(4,1),(1,5)]
```

> 💡 Това е конструктивно доказателство, че $\mathbb{Q}^+$ е изброимо: всяко рационално число се появява на **крайна** позиция в списъка.

---

## Пример 6: Изтичане на памет

```haskell
import Data.List (foldl')

mean :: [Double] -> Double
mean xs = s / fromIntegral n
  where (s, n) = foldl' step (0, 0 :: Int) xs
        step (accS, accN) x = (accS + x, accN + 1)
```

> ⚠️ Дори с `foldl'` тази функция натрупва thunks! `foldl'` оценява акумулатора до **WHNF** - а двойката `(_, _)` вече е в WHNF. Компонентите ѝ остават неоценени.

Решение - принудително оценяване на компонентите:

```haskell
mean' :: [Double] -> Double
mean' xs = s / fromIntegral n
  where (s, n) = foldl' step (0, 0 :: Int) xs
        step (accS, accN) x = let s' = accS + x
                                  n' = accN + 1
                              in s' `seq` n' `seq` (s', n')
```

---

## Пример 7: Безточков стил

```haskell
-- Преди
squaresOfEvens xs = map (\x -> x ^ 2) (filter (\x -> even x) xs)

-- Сечения и η-редукция в ламбдите
squaresOfEvens xs = map (^ 2) (filter even xs)

-- Композиция и η-редукция
squaresOfEvens = map (^ 2) . filter even
```

```haskell
-- Сума на цифрите на число
digitSum :: Integer -> Int
digitSum = sum . map (read . (: [])) . show

-- Брой думи
wordCount :: String -> Int
wordCount = length . words

-- Квадрати-палиндроми
palindromicSquares :: [Integer]
palindromicSquares = filter isPal (map (^ 2) [1..])
  where isPal = ((==) <*> reverse) . show    -- прекалено „умно“ - вижте бележката
```

```haskell
ghci> digitSum (2 ^ 1000)
1366
ghci> wordCount "the quick brown fox"
4
ghci> take 10 palindromicSquares
[1,4,9,121,484,676,10201,12321,14641,40804]
```

> ⚠️ `((==) <*> reverse) . show` е безточкова версия на `\n -> show n == reverse (show n)`, но е трудна за четене. Предпочитайте втория вариант. (`<*>` за функции ще разберем в Седмица 13.)
