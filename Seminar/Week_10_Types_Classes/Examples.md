# Седмица 10 - Примери

## Пример 1: Геометрични фигури

```haskell
data Shape = Circle Double
           | Rectangle Double Double
           | Triangle Double Double Double
  deriving (Show, Eq)

area :: Shape -> Double
area (Circle r)       = pi * r ^ 2
area (Rectangle w h)  = w * h
area (Triangle a b c) = sqrt (s * (s - a) * (s - b) * (s - c))
  where s = (a + b + c) / 2

perimeter :: Shape -> Double
perimeter (Circle r)       = 2 * pi * r
perimeter (Rectangle w h)  = 2 * (w + h)
perimeter (Triangle a b c) = a + b + c

isRound :: Shape -> Bool
isRound (Circle _) = True
isRound _          = False
```

```haskell
ghci> area (Circle 1)
3.141592653589793
ghci> map area [Rectangle 3 4, Triangle 3 4 5]
[12.0,6.0]
ghci> filter isRound [Circle 1, Rectangle 1 2, Circle 3]
[Circle 1.0,Circle 3.0]
ghci> :t Rectangle
Rectangle :: Double -> Double -> Shape
ghci> map (Rectangle 2) [1, 2, 3]       -- конструкторът е функция - може частично!
[Rectangle 2.0 1.0,Rectangle 2.0 2.0,Rectangle 2.0 3.0]
```

---

## Пример 2: Записи

```haskell
data Student = Student
  { name   :: String
  , fn     :: Int
  , grades :: [Int]
  } deriving (Show)

average :: Student -> Double
average s = fromIntegral (sum (grades s)) / fromIntegral (length (grades s))

addGrade :: Int -> Student -> Student
addGrade g s = s { grades = g : grades s }

students :: [Student]
students =
  [ Student "Ivan" 101 [5, 6, 4]
  , Student "Maria" 102 [6, 6, 5]
  , Student { name = "Petar", fn = 103, grades = [3, 4] }
  ]
```

```haskell
ghci> map name (filter ((>= 5) . average) students)
["Ivan","Maria"]
ghci> grades (addGrade 6 (head students))
[6,5,6,4]
```

---

## Пример 3: Двоично дърво за търсене

```haskell
data Tree a = Empty | Node a (Tree a) (Tree a)
  deriving (Show)

insert :: Ord a => a -> Tree a -> Tree a
insert x Empty = Node x Empty Empty
insert x t@(Node v l r)
  | x < v     = Node v (insert x l) r
  | x > v     = Node v l (insert x r)
  | otherwise = t

member :: Ord a => a -> Tree a -> Bool
member _ Empty = False
member x (Node v l r)
  | x == v    = True
  | x < v     = member x l
  | otherwise = member x r

fromList :: Ord a => [a] -> Tree a
fromList = foldr insert Empty

toList :: Tree a -> [a]
toList Empty        = []
toList (Node v l r) = toList l ++ [v] ++ toList r

height :: Tree a -> Int
height Empty        = 0
height (Node _ l r) = 1 + max (height l) (height r)
```

```haskell
ghci> let t = fromList [9, 4, 1, 8, 3, 5]
ghci> toList t
[1,3,4,5,8,9]
ghci> member 8 t
True
ghci> height t
3
ghci> toList (fromList "haskell")
"aehkls"
```

> 💡 Сравнете с Пример 3 от Седмица 6 - алгоритъмът е същият, но типът `Ord a => a -> Tree a -> Tree a` гарантира, че **не можем** да вмъкнем низ в дърво от числа.

---

## Пример 4: `Maybe` и `Either`

```haskell
safeDiv :: Int -> Int -> Maybe Int
safeDiv _ 0 = Nothing
safeDiv x y = Just (x `div` y)

safeHead :: [a] -> Maybe a
safeHead []    = Nothing
safeHead (x:_) = Just x

-- Търсене на студент по факултетен номер
findStudent :: Int -> [Student] -> Maybe Student
findStudent _ [] = Nothing
findStudent n (s:ss)
  | fn s == n = Just s
  | otherwise = findStudent n ss

-- Валидация с описание на грешката
parseGrade :: Int -> Either String Int
parseGrade g
  | g < 2     = Left ("твърде ниска оценка: " ++ show g)
  | g > 6     = Left ("твърде висока оценка: " ++ show g)
  | otherwise = Right g
```

```haskell
ghci> safeDiv 10 2
Just 5
ghci> safeDiv 10 0
Nothing
ghci> fmap name (findStudent 102 students)
Just "Maria"
ghci> parseGrade 5
Right 5
ghci> mapM_ (putStrLn . either ("Грешка: " ++) show . parseGrade) [5, 7, 1]
5
Грешка: твърде висока оценка: 7
Грешка: твърде ниска оценка: 1
```

> ⚠️ `show` на низ с кирилица дава кодове (`"\1090\1074..."`). За четим изход използвайте `putStrLn`. `mapM_` прилага `IO` действие за всеки елемент - подробно в Седмица 12.

> 💡 `fmap` прилага функция „вътре“ в `Maybe` - ще го разгледаме подробно в Седмица 13.

---

## Пример 5: Инстанции на класове

```haskell
data Color = Red | Green | Blue
  deriving (Eq, Ord, Enum, Bounded)

instance Show Color where
  show Red   = "червено"
  show Green = "зелено"
  show Blue  = "синьо"

-- Рационални числа - собствена инстанция на Num
data Rat = Rat Integer Integer

makeRat :: Integer -> Integer -> Rat
makeRat n d = Rat (s * n `div` g) (s * abs d `div` g)
  where g = gcd n d
        s = signum d

instance Show Rat where
  show (Rat n d) = show n ++ "/" ++ show d

instance Eq Rat where
  Rat a b == Rat c d = a * d == b * c

instance Num Rat where
  Rat a b + Rat c d = makeRat (a * d + b * c) (b * d)
  Rat a b * Rat c d = makeRat (a * c) (b * d)
  negate (Rat a b)  = Rat (negate a) b
  abs (Rat a b)     = Rat (abs a) b
  signum (Rat a _)  = Rat (signum a) 1
  fromInteger n     = Rat n 1
```

```haskell
ghci> [minBound .. maxBound] :: [Color]
[червено,зелено,синьо]
ghci> makeRat 1 2 + makeRat 1 3
5/6
ghci> sum [makeRat 1 2, makeRat 1 3, makeRat 1 6]
1/1
ghci> makeRat 1 2 * 4
2/1
```

> 💡 Щом `Rat` е инстанция на `Num`, работят и `sum`, `product`, и литералите (`4` става `fromInteger 4`). Сравнете с рационалните числа в Scheme (Седмица 5).

---

## Пример 6: Собствен клас

```haskell
class Describable a where
  describe :: a -> String
  describe _ = "нещо"              -- реализация по подразбиране

  name' :: a -> String

instance Describable Bool where
  describe True  = "истина"
  describe False = "лъжа"
  name' _ = "булева стойност"

instance Describable Shape where
  name' _ = "фигура"               -- describe остава по подразбиране
```

```haskell
ghci> putStrLn (describe True)
истина
ghci> putStrLn (describe (Circle 1))
нещо
ghci> putStrLn (name' (Circle 1))
фигура
```
