# Седмица 10 - Задачи за в час

## Задача 1: Дни от седмицата

```haskell
data Day = Mon | Tue | Wed | Thu | Fri | Sat | Sun
  deriving (Show, Eq, Ord, Enum, Bounded)
```

Напишете:

```haskell
nextDay :: Day -> Day          -- след Sun идва Mon
isWorkday :: Day -> Bool
daysUntil :: Day -> Day -> Int -- колко дни напред е вторият ден
```

```haskell
>>> nextDay Sun
Mon
>>> filter isWorkday [minBound .. maxBound]
[Mon,Tue,Wed,Thu,Fri]
>>> daysUntil Fri Tue
4
```

> 💡 Използвайте `fromEnum`, `toEnum`, `maxBound`.

---

## Задача 2: Операции над дървета

С типа `Tree a` от теорията напишете:

```haskell
size :: Tree a -> Int
sumTree :: Num a => Tree a -> a
treeMap :: (a -> b) -> Tree a -> Tree b
leaves :: Tree a -> [a]
```

```haskell
>>> let t = Node 5 (Node 3 Empty Empty) (Node 8 Empty (Node 9 Empty Empty))
>>> size t
4
>>> sumTree t
25
>>> leaves t
[3,9]
>>> toList (treeMap (* 10) t)
[30,50,80,90]
```

---

## Задача 3: Безопасни функции

```haskell
safeLast :: [a] -> Maybe a
safeIndex :: [a] -> Int -> Maybe a
lookupAll :: Eq k => [k] -> [(k, v)] -> [v]    -- стойностите на намерените ключове
```

```haskell
>>> safeLast []
Nothing
>>> safeIndex "abc" 1
Just 'b'
>>> safeIndex "abc" 5
Nothing
>>> lookupAll [1, 3, 5] [(1, "a"), (2, "b"), (3, "c")]
["a","c"]
```

> 💡 За `lookupAll` - `mapMaybe` от `Data.Maybe`.

---

## Задача 4: Вектори

```haskell
data V2 = V2 Double Double
```

1. Напишете инстанция на `Show`, която показва вектора като `<x, y>`.
2. Напишете инстанция на `Eq`.
3. Напишете инстанция на `Num`, където `+`, `-` са покоординатни, `*` е покоординатно умножение, а `fromInteger n` е `V2 n n`.

```haskell
>>> V2 1 2 + V2 3 4
<4.0, 6.0>
>>> V2 1 2 * 3
<3.0, 6.0>
>>> sum [V2 1 1, V2 2 2, V2 3 3]
<6.0, 6.0>
```

---

## Задача 5: Валидация с `Either`

```haskell
data Person = Person { personName :: String, age :: Int } deriving Show

mkPerson :: String -> Int -> Either String Person
```

`mkPerson` връща грешка, ако името е празно или възрастта не е в $[0, 150]$.

```haskell
>>> mkPerson "Ana" 30
Right (Person {personName = "Ana", age = 30})
>>> mkPerson "" 30
Left "empty name"
>>> mkPerson "Ana" 200
Left "invalid age: 200"
```
