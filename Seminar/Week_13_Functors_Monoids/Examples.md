# Седмица 13 - Примери

## Пример 1: `Functor` и `Foldable` за дърво

```haskell
data Tree a = Empty | Node a (Tree a) (Tree a)
  deriving (Show)

instance Functor Tree where
  fmap _ Empty        = Empty
  fmap f (Node v l r) = Node (f v) (fmap f l) (fmap f r)

instance Foldable Tree where
  foldMap _ Empty        = mempty
  foldMap f (Node v l r) = foldMap f l <> f v <> foldMap f r

fromList :: Ord a => [a] -> Tree a
fromList = foldr insert Empty
  where
    insert x Empty = Node x Empty Empty
    insert x t@(Node v l r)
      | x < v     = Node v (insert x l) r
      | x > v     = Node v l (insert x r)
      | otherwise = t

tree :: Tree Int
tree = fromList [5, 3, 8, 1, 4, 9]
```

```haskell
ghci> sum tree
30
ghci> length tree
6
ghci> maximum tree
9
ghci> elem 4 tree
True
ghci> foldr (:) [] tree
[1,3,4,5,8,9]
ghci> foldr (:) [] (fmap (* 10) tree)
[10,30,40,50,80,90]
ghci> foldr (:) [] (show <$> tree)
["1","3","4","5","8","9"]
```

> 💡 Написахме **една** функция (`foldMap`) и получихме `sum`, `length`, `maximum`, `elem`, `foldr`, `null`, `product`... Сравнете с Пример 3 от Седмица 10, където всяка функция беше отделна рекурсия.

---

## Пример 2: Проверка на законите за функтора

```haskell
-- fmap id = id
lawIdentity :: Bool
lawIdentity = foldr (:) [] (fmap id tree) == foldr (:) [] tree

-- fmap (f . g) = fmap f . fmap g
lawComposition :: Bool
lawComposition =
  foldr (:) [] (fmap ((+ 1) . (* 2)) tree)
    == foldr (:) [] ((fmap (+ 1) . fmap (* 2)) tree)
```

```haskell
ghci> (lawIdentity, lawComposition)
(True,True)
```

> ⚠️ Проверката за **едно** дърво не е доказателство! Доказателството за всички дървета е по структурна индукция - вижте домашното.

---

## Пример 3: Валидация с `Applicative`

```haskell
import Text.Read (readMaybe)

data User = User { userName :: String, userAge :: Int, userEmail :: String }
  deriving (Show)

validateName :: String -> Either String String
validateName n
  | null n    = Left "празно име"
  | otherwise = Right n

validateAge :: String -> Either String Int
validateAge s = case readMaybe s of
  Nothing -> Left ("невалидна възраст: " ++ s)
  Just a
    | a < 0 || a > 150 -> Left ("възраст извън граници: " ++ show a)
    | otherwise        -> Right a

validateEmail :: String -> Either String String
validateEmail e
  | '@' `elem` e = Right e
  | otherwise    = Left ("невалиден имейл: " ++ e)

mkUser :: String -> String -> String -> Either String User
mkUser n a e = User <$> validateName n <*> validateAge a <*> validateEmail e
```

```haskell
ghci> mkUser "ana" "25" "ana@fmi.bg"
Right (User {userName = "ana", userAge = 25, userEmail = "ana@fmi.bg"})
ghci> either putStrLn print (mkUser "ana" "abc" "ana@fmi.bg")
невалидна възраст: abc
ghci> either putStrLn print (mkUser "" "abc" "ana")
празно име
```

> 💡 `User <$> ... <*> ... <*> ...` - конструкторът `User` е обикновена функция на три аргумента, приложена към три стойности в `Either String`. При **първата** грешка изчислението спира.

---

## Пример 4: Апликативни функтори - всички комбинации

```haskell
-- Всички суми от зар и монета
outcomes :: [Int]
outcomes = (+) <$> [1 .. 6] <*> [0, 10]

-- Всички думи с дължина 2 над азбука
words2 :: [String]
words2 = (\a b -> [a, b]) <$> "ab" <*> "xyz"

-- Безопасно събиране на три стойности от речник
lookupSum :: [(String, Int)] -> Maybe Int
lookupSum env = (\x y z -> x + y + z)
  <$> lookup "x" env <*> lookup "y" env <*> lookup "z" env
```

```haskell
ghci> outcomes
[1,11,2,12,3,13,4,14,5,15,6,16]
ghci> words2
["ax","ay","az","bx","by","bz"]
ghci> lookupSum [("x", 1), ("y", 2), ("z", 3)]
Just 6
ghci> lookupSum [("x", 1), ("z", 3)]
Nothing
```

---

## Пример 5: Моноиди за статистика с едно обхождане

```haskell
import Data.Semigroup

data Stats = Stats { count :: Int, total :: Int, smallest :: Min Int, largest :: Max Int }
  deriving (Show)

instance Semigroup Stats where
  Stats c1 t1 mn1 mx1 <> Stats c2 t2 mn2 mx2 =
    Stats (c1 + c2) (t1 + t2) (mn1 <> mn2) (mx1 <> mx2)

single :: Int -> Stats
single x = Stats 1 x (Min x) (Max x)

stats :: [Int] -> Maybe Stats
stats []     = Nothing
stats (x:xs) = Just (foldr ((<>) . single) (single x) xs)

average :: Stats -> Double
average s = fromIntegral (total s) / fromIntegral (count s)
```

```haskell
ghci> fmap average (stats [3, 1, 4, 1, 5, 9, 2, 6])
Just 3.875
ghci> fmap (getMin . smallest) (stats [3, 1, 4])
Just 1
ghci> stats []
Nothing
```

> 💡 `Stats` е полугрупа, защото всеки компонент е полугрупа. Тъй като `<>` е асоциативна, можем да разделим списъка на части, да пресметнем статистиката на всяка поотделно (например паралелно) и да ги комбинираме.

---

## Пример 6: Сортиране по няколко критерия

```haskell
import Data.List (sortBy)
import Data.Ord (comparing, Down (..))

students :: [(String, Int, Double)]     -- (име, курс, успех)
students = [("Ivan", 2, 5.5), ("Maria", 1, 6.0), ("Petar", 2, 4.5), ("Ana", 1, 5.5)]

-- По курс (нарастващо), после по успех (намаляващо), после по име
ranked :: [(String, Int, Double)]
ranked = sortBy (comparing course <> comparing (Down . grade) <> comparing name) students
  where name (n, _, _)   = n
        course (_, c, _) = c
        grade (_, _, g)  = g
```

```haskell
ghci> ranked
[("Maria",1,6.0),("Ana",1,5.5),("Ivan",2,5.5),("Petar",2,4.5)]
```
