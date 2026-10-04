# Седмица 8 - Примери

## Пример 1: Аритметика в GHCi

```haskell
ghci> 2 + 3 * 4       -- 14 (приоритет на *)
ghci> (2 + 3) * 4     -- 20
ghci> 7 `div` 2       -- 3
ghci> 7 `mod` 2       -- 1
ghci> (-7) `mod` 3    -- 2   (като modulo в Racket)
ghci> (-7) `rem` 3    -- -1  (като remainder в Racket)
ghci> sqrt 16         -- 4.0
ghci> 2 ^ 100
1267650600228229401496703205376
```

> ⚠️ `-7 `mod` 3` без скоби се чете като `-(7 `mod` 3)` = `-1`. Отрицателните числа като аргументи слагайте в скоби: `abs (-5)`.

---

## Пример 2: От Scheme към Haskell

Функциите от първата част на курса, пренаписани на Haskell:

```haskell
-- (define (sum-of-squares x y) (+ (square x) (square y)))
sumOfSquares :: Int -> Int -> Int
sumOfSquares x y = square x + square y
  where square z = z * z

-- Факториел - рекурсивен процес
factorial :: Integer -> Integer
factorial 0 = 1
factorial n = n * factorial (n - 1)

-- Фибоначи - итеративен процес
fib :: Int -> Integer
fib n = helper 0 1 n
  where
    helper a _ 0 = a
    helper a b k = helper b (a + b) (k - 1)

-- Бързо степенуване с гардове
fastExpt :: Integer -> Int -> Integer
fastExpt _ 0 = 1
fastExpt b n
  | even n    = half * half
  | otherwise = b * fastExpt b (n - 1)
  where half = fastExpt b (n `div` 2)
```

```haskell
ghci> sumOfSquares 3 4
25
ghci> factorial 20
2432902008176640000
ghci> fib 50
12586269025
ghci> fastExpt 2 100
1267650600228229401496703205376
```

> 💡 Сравнете с Седмица 3. Образците (`factorial 0 = 1`) заместват `(if (= n 0) 1 ...)`, а гардовете - `cond`.

---

## Пример 3: Метод на Нютон с `where`

```haskell
mySqrt :: Double -> Double
mySqrt x = sqrtIter 1.0
  where
    sqrtIter guess
      | goodEnough guess = guess
      | otherwise        = sqrtIter (improve guess)
    goodEnough guess = abs (guess * guess - x) < 0.001
    improve guess = (guess + x / guess) / 2
```

```haskell
ghci> mySqrt 9
3.00009155413138
ghci> mySqrt 2
1.4142156862745097
```

> 💡 Абсолютно същата структура като блоковата структура в Седмица 2 - помощните функции „виждат“ `x`.

---

## Пример 4: Гардове и `where`

```haskell
-- Тип на триъгълник
triangleType :: Int -> Int -> Int -> String
triangleType a b c
  | not valid            = "невалиден"
  | a == b && b == c     = "равностранен"
  | a == b || b == c || a == c = "равнобедрен"
  | otherwise            = "разностранен"
  where
    valid = a + b > c && a + c > b && b + c > a

-- Високосна година
isLeapYear :: Int -> Bool
isLeapYear y = (y `mod` 4 == 0 && y `mod` 100 /= 0) || y `mod` 400 == 0
```

```haskell
ghci> triangleType 3 4 5
"разностранен"
ghci> triangleType 2 2 3
"равнобедрен"
ghci> triangleType 1 2 10
"невалиден"
ghci> isLeapYear 1900
False
```

---

## Пример 5: Кортежи

```haskell
-- Корени на квадратно уравнение (приемаме D ≥ 0)
quadraticRoots :: Double -> Double -> Double -> (Double, Double)
quadraticRoots a b c = (x1, x2)
  where
    d  = sqrt (b * b - 4 * a * c)
    x1 = (-b + d) / (2 * a)
    x2 = (-b - d) / (2 * a)

-- Цяло деление с остатък
divMod' :: Int -> Int -> (Int, Int)
divMod' a b = (a `div` b, a `mod` b)

-- Разстояние между точки
distance :: (Double, Double) -> (Double, Double) -> Double
distance (x1, y1) (x2, y2) = sqrt ((x2 - x1) ^ 2 + (y2 - y1) ^ 2)
```

```haskell
ghci> quadraticRoots 1 (-3) 2
(2.0,1.0)
ghci> divMod' 17 5
(3,2)
ghci> distance (0, 0) (3, 4)
5.0
```

> ⚠️ `quadraticRoots` не проверява дали дискриминантата е отрицателна. Как да обработваме такива случаи безопасно с `Maybe` ще видим в Седмица 10.

---

## Пример 6: Currying и частично прилагане

```haskell
add :: Int -> Int -> Int
add x y = x + y

add5 :: Int -> Int
add5 = add 5            -- частично прилагане - без да пишем аргумента!
```

```haskell
ghci> add5 10
15
ghci> :t add
add :: Int -> Int -> Int
ghci> :t add 5
add 5 :: Int -> Int
ghci> :t max
max :: Ord a => a -> a -> a
ghci> :t max 3
max 3 :: (Ord a, Num a) => a -> a
```

> 💡 `Num a =>`, `Ord a =>` са **ограничения** (constraints) - „за всеки тип `a`, който е число / е наредим“. Ще ги разгледаме в Седмица 10.

---

## Пример 7: Типови грешки

```haskell
ghci> length [1, 2, 3] / 2
-- error: No instance for (Fractional Int)
-- length връща Int, а / изисква дробен тип

ghci> fromIntegral (length [1, 2, 3]) / 2
1.5

ghci> 'a' + 1
-- error: No instance for (Num Char)

ghci> if True then 1 else "no"
-- error: No instance for (Num String)
-- двата клона трябва да са от един тип
```

> 💡 Тези грешки се откриват **преди** изпълнението. В Racket `(+ #\a 1)` дава грешка едва когато изразът се оцени.
