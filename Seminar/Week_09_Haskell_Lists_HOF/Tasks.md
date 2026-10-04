# Седмица 9 - Задачи за в час

## Задача 1: `isSorted` и `compress`

```haskell
isSorted :: Ord a => [a] -> Bool          -- дали списъкът е сортиран (ненамаляващо)
compress :: Eq a => [a] -> [a]            -- премахва последователните повторения
```

```haskell
>>> isSorted [1, 2, 2, 5]
True
>>> isSorted [3, 1]
False
>>> compress "aaabccaa"
"abca"
```

> 💡 Използвайте образеца `(x:y:rest)`.

---

## Задача 2: `countBy` и `partition'`

```haskell
countBy :: (a -> Bool) -> [a] -> Int
partition' :: (a -> Bool) -> [a] -> ([a], [a])
```

```haskell
>>> countBy even [1..10]
5
>>> partition' even [1..10]
([2,4,6,8,10],[1,3,5,7,9])
```

Напишете `partition'` с **едно** обхождане чрез `foldr`.

---

## Задача 3: `applyAll`

Напишете `applyAll :: [a -> a] -> a -> a`, която прилага списък от функции, като **последната** се прилага първа (като `compose-all` от Седмица 4). Използвайте `foldr` и `(.)`.

```haskell
>>> applyAll [(+ 1), (* 2), (+ 3)] 5        -- ((5 + 3) * 2) + 1
17
```

---

## Задача 4: `merge` и `quickSort`

```haskell
merge :: Ord a => [a] -> [a] -> [a]       -- слива два сортирани списъка
quickSort :: Ord a => [a] -> [a]          -- с list comprehensions
```

```haskell
>>> merge [1, 4, 7] [2, 3, 9]
[1,2,3,4,7,9]
>>> quickSort [3, 1, 4, 1, 5, 9, 2, 6]
[1,1,2,3,4,5,6,9]
```

---

## Задача 5: List comprehensions

С list comprehensions напишете:

1. `perfectNumbers :: Int -> [Int]` - съвършените числа до `n`
2. `sumPairs :: Int -> [(Int, Int)]` - всички двойки `(a, b)`, $1 \le a \le b$, с $a + b = n$
3. `triangles :: Int -> [(Int, Int, Int)]` - всички правоъгълни триъгълници с цели страни и периметър точно `p`

```haskell
>>> perfectNumbers 500
[6,28,496]
>>> sumPairs 6
[(1,5),(2,4),(3,3)]
>>> triangles 24
[(6,8,10)]
```

---

## Задача 6: Безточков стил

Пренапишете в безточков стил, като използвате `(.)`:

```haskell
f1 xs = length (filter (> 10) (map (* 3) xs))
f2 xs = sum (map fst xs)
f3 s = reverse (words s)
f4 x = not (even x)
```
