# Седмица 13 - Задачи за в час

## Задача 1: Какво ще върне?

Определете резултата (после проверете в GHCi):

```haskell
fmap (fmap (+ 1)) [Just 1, Nothing]
fmap length (Just "abc")
(fmap . fmap) (* 2) (Just [1, 2])
fmap (+ 1) (3, 4)
(+) <$> [1, 2] <*> [10, 100]
pure 5 :: Maybe Int
Just (+ 1) <*> Nothing
(,) <$> Just 'a' <*> Just True
sequenceA [[1, 2], [3]]
mconcat ["ab", "", "cd"]
```

---

## Задача 2: Инстанции на `Functor`

Напишете инстанции на `Functor` за:

```haskell
data Pair a = Pair a a deriving Show
data Rose a = Rose a [Rose a] deriving Show
newtype Box a = Box a deriving Show
```

```haskell
>>> fmap (+ 1) (Pair 1 2)
Pair 2 3
>>> fmap show (Rose 1 [Rose 2 [], Rose 3 []])
Rose "1" [Rose "2" [],Rose "3" []]
```

Може ли да напишете инстанция на `Functor` за `data Pred a = Pred (a -> Bool)`? Защо?

---

## Задача 3: Апликативен стил

Пренапишете **без** `case` и образци, като използвате `<$>` и `<*>`:

```haskell
addMaybes :: Maybe Int -> Maybe Int -> Maybe Int
addMaybes (Just x) (Just y) = Just (x + y)
addMaybes _        _        = Nothing

area :: Maybe Double -> Maybe Double -> Maybe Double
area mw mh = case mw of
  Nothing -> Nothing
  Just w  -> case mh of
    Nothing -> Nothing
    Just h  -> Just (w * h)
```

---

## Задача 4: `traverse` за парсване

Чрез `traverse` и `readMaybe` напишете `parseAll :: [String] -> Maybe [Int]`, която връща `Nothing`, ако **някой** от низовете не е число.

```haskell
>>> parseAll ["1", "2", "3"]
Just [1,2,3]
>>> parseAll ["1", "x"]
Nothing
```

---

## Задача 5: Моноиди

1. Дефинирайте `newtype MaxInt = MaxInt Int` и направете го `Semigroup` и `Monoid` (неутралният елемент е `minBound`).
2. С **едно** извикване на `foldMap` пресметнете броя думи и дължината на най-дългата дума в текст.

```haskell
>>> foldMap (\w -> (Sum 1, Max (length w))) (words "the quick brown fox")
(Sum {getSum = 4},Max {getMax = 5})
```

> 💡 Двойка от моноиди е моноид.

---

## Задача 6: `Foldable` за `Rose`

Напишете инстанция `Foldable Rose` чрез `foldMap`. После без допълнителна рекурсия пресметнете сумата, броя върхове и максимума на дърво.

```haskell
>>> let r = Rose 1 [Rose 2 [Rose 5 []], Rose 3 [], Rose 4 [Rose 7 []]]
>>> (sum r, length r, maximum r)
(22,6,7)
```
