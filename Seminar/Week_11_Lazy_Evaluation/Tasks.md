# Седмица 11 - Задачи за в час

## Задача 1: Какво ще върне?

Определете резултата от всеки израз - стойност, грешка или зацикляне. После проверете в GHCi.

```haskell
length [1, undefined, 3]
sum [1, undefined, 3]
head (tail [undefined, 2])
take 0 undefined
fst (snd (undefined, (1, 2)))
let xs = 1 : xs in take 3 xs
null (undefined : undefined)
foldr (\x _ -> x) 0 [1..]
foldl (\_ x -> x) 0 [1..]
```

---

## Задача 2: Безкрайни списъци

Дефинирайте:

```haskell
alternating :: [Int]        -- 1, -1, 1, -1, ...
multiplesOf :: Int -> [Int] -- кратните на n: n, 2n, 3n, ...
tribonacci :: [Integer]     -- 0, 0, 1, 1, 2, 4, 7, 13, ...  (всеки е сума на предишните три)
```

```haskell
>>> take 4 alternating
[1,-1,1,-1]
>>> take 5 (multiplesOf 3)
[3,6,9,12,15]
>>> take 10 tribonacci
[0,0,1,1,2,4,7,13,24,44]
```

> 💡 За `tribonacci` - неявна дефиниция със `zipWith3`.

---

## Задача 3: FizzBuzz без `mod`

Дефинирайте безкрайния списък `fizzbuzz :: [String]`: числата от 1 нататък, като кратните на 3 са заменени с `"Fizz"`, на 5 - с `"Buzz"`, на 15 - с `"FizzBuzz"`. **Не** използвайте `mod` - използвайте `cycle`.

```haskell
>>> take 15 fizzbuzz
["1","2","Fizz","4","Buzz","Fizz","7","8","Fizz","Buzz","11","Fizz","13","14","FizzBuzz"]
```

---

## Задача 4: Ранно спиране с `foldr`

Чрез `foldr` напишете `myAny`, `myElem` и `myTakeWhile`, които работят и с **безкрайни** списъци.

```haskell
>>> myAny (> 100) [1..]
True
>>> myElem 42 [1..]
True
>>> myTakeWhile (< 5) [1..]
[1,2,3,4]
```

Защо не може да ги напишете с `foldl`?

---

## Задача 5: Безточков стил

Пренапишете в безточков стил:

```haskell
f1 xs = sum (map (\x -> x * x) (filter (\x -> x > 0) xs))
f2 n = take n (iterate (\x -> x * 2) 1)
f3 xs = map (\(a, b) -> a + b) (zip xs (tail xs))
f4 s = length (filter (\w -> length w > 3) (words s))
```

> 💡 `f2` може да стане `flip take (iterate (* 2) 1)`. За `f3` - `zipWith`.

---

## Задача 6: Изтичане на памет

Обяснете защо `foldl (+) 0 [1..10^8]` използва много памет, а `foldl' (+) 0 [1..10^8]` - не. Нарисувайте как изглежда акумулаторът след първите 4 стъпки в двата случая.
