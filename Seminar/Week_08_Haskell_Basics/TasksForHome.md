# Седмица 8 - Задачи за домашна работа

## Задача 1: `fibonacci` (итеративен процес)

Напишете версия на Фибоначи с акумулатори (опашкова рекурсия):

```haskell
fibonacci :: Int -> Integer
```

```haskell
>>> fibonacci 10
55
>>> fibonacci 90
2880067194370816120
```

---

## Задача 2: `isPalindrome`

Напишете функция `isPalindrome :: Int -> Bool`, която проверява дали число е палиндром.

```haskell
>>> isPalindrome 12321
True
>>> isPalindrome 123
False
```

> 💡 Подсказка: Напишете първо `reverseNumber :: Int -> Int` с помощна функция и акумулатор.

---

## Задача 3: `isValidDate`

Напишете функция `isValidDate :: Int -> Int -> Int -> Bool`, която проверява дали дата (ден, месец, година) е валидна. Отчетете високосните години.

```haskell
>>> isValidDate 29 2 2024
True
>>> isValidDate 29 2 2023
False
>>> isValidDate 31 4 2025
False
```

---

## Задача 4: `sumDivisors` и `isPerfect`

```haskell
sumDivisors :: Int -> Int   -- сума на делителите, по-малки от n
isPerfect :: Int -> Bool
```

```haskell
>>> sumDivisors 12
16
>>> isPerfect 28
True
```

---

## Задача 5: `(~=)`

Дефинирайте **оператор** `(~=) :: Double -> Double -> Bool`, който сравнява две числа с точност `1e-9`.

```haskell
>>> 0.1 + 0.2 ~= 0.3
True
>>> 0.1 + 0.2 == 0.3
False
```

> 💡 Операторите се дефинират като функции: `x ~= y = ...`. Ще трябва да зададете и приоритет, например `infix 4 ~=` (като `==`).

---

## Задача 6: `intSqrt`

Напишете `intSqrt :: Int -> Int`, която връща $\lfloor \sqrt{n} \rfloor$ чрез **двоично търсене** (без `sqrt`).

```haskell
>>> intSqrt 17
4
>>> intSqrt 1000000
1000
```

---

## Задача 7: Стъпки на Колац

Напишете `collatzSteps :: Int -> Int`, която връща броя стъпки, нужни за да се стигне от `n` до 1 (вижте Седмица 7, Задача 4 за домашно).

```haskell
>>> collatzSteps 6
8
>>> collatzSteps 27
111
```

---

## Задача 8: Currying

Дадена е функцията:

```haskell
f :: Int -> Int -> Int -> Int
f x y z = x * 100 + y * 10 + z
```

1. Какъв е типът на `f 1`? А на `f 1 2`?
2. Дефинирайте `g = f 1 2` и пресметнете `g 3`.
3. Напишете `f` като ламбда изрази: `f = \x -> \y -> \z -> ...`
4. Напишете функция `flip3 :: (a -> b -> c -> d) -> c -> b -> a -> d`, която обръща реда на аргументите.

```haskell
>>> flip3 f 1 2 3
321
```
