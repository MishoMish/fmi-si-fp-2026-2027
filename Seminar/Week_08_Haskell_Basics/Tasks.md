# Седмица 8 - Задачи за в час

## Задача 1: `max3`

Напишете функция `max3 :: Int -> Int -> Int -> Int`, която връща най-голямото от три числа. Решете я веднъж с гардове и веднъж чрез `max`.

```haskell
>>> max3 3 7 5
7
>>> max3 (-1) (-5) (-3)
-1
```

---

## Задача 2: `countDigits` и `digitSum`

Напишете функции за неотрицателно цяло число:

```haskell
countDigits :: Int -> Int
digitSum :: Int -> Int
```

```haskell
>>> countDigits 12345
5
>>> countDigits 0
1
>>> digitSum 9999
36
```

---

## Задача 3: `isPrime`

Напишете функция `isPrime :: Int -> Bool` с помощна функция в `where`, която проверява делителите от 2 до $\sqrt{n}$.

```haskell
>>> isPrime 17
True
>>> isPrime 1
False
>>> isPrime 91
False
```

---

## Задача 4: Вектори като кортежи

Вектор в равнината е двойка `(Double, Double)`. Напишете:

```haskell
addVectors :: (Double, Double) -> (Double, Double) -> (Double, Double)
scaleVector :: Double -> (Double, Double) -> (Double, Double)
dotProduct :: (Double, Double) -> (Double, Double) -> Double
```

```haskell
>>> addVectors (1, 2) (3, 4)
(4.0,6.0)
>>> scaleVector 2 (1, 3)
(2.0,6.0)
>>> dotProduct (1, 2) (3, 4)
11.0
```

---

## Задача 5: `gcd'`

Напишете `gcd' :: Int -> Int -> Int` по алгоритъма на Евклид, като използвате образци.

```haskell
>>> gcd' 48 18
6
>>> gcd' 17 5
1
```

---

## Задача 6: Какъв е типът?

Без GHCi определете типовете (после проверете с `:t`):

```haskell
f1 x y = x && not y
f2 x = (x, x)
f3 (a, b) = (b, a)
f4 f x = f (f x)
f5 = max 'a'
```

> 💡 Ако функцията работи за произволен тип, използвайте типова променлива (`a`, `b`, ...).
