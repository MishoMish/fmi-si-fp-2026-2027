# Типове, класове и полиморфизъм

## Въведение

В Scheme (Седмица 5) строяхме съставни данни от двойки и ги скривахме зад конструктори и селектори. В Haskell това се прави на ниво **типове**:

- `data` дефинира **нов тип** с неговите **конструктори**
- **образците** заместват селекторите и предикатите
- **класовете** позволяват една функция да работи по различен начин за различни типове

---

## 1. Видове полиморфизъм

**Полиморфизъм** - една и съща функция работи с **различни типове**.

| Вид              | Идея                                              | Пример в Haskell                       |
| ---------------- | ------------------------------------------------- | -------------------------------------- |
| **Параметричен** | Една реализация за **всички** типове              | `length :: [a] -> Int`                 |
| **Ad-hoc**       | **Различна** реализация за всеки тип              | `(==) :: Eq a => a -> a -> Bool`       |
| Подтипов         | Обект от подтип може да замести обект от надтип   | Няма в Haskell (има в Java, C++)       |

```haskell
length :: [a] -> Int         -- не „знае“ нищо за a - работи еднакво за всички
show   :: Show a => a -> String  -- различна реализация за Int, Bool, [Char]...
```

> 💡 В Scheme всяка функция е „полиморфна“, защото няма статични типове - но грешките се откриват по време на изпълнение.

---

## 2. Синоними на типове: `type`

`type` дава **ново име** на съществуващ тип (не създава нов тип):

```haskell
type Name  = String
type Point = (Double, Double)
type AssocList k v = [(k, v)]
```

> ⚠️ `Name` и `String` са напълно взаимозаменяеми - компилаторът не ги различава.

---

## 3. Алгебрични типове данни: `data`

### Изброени типове (суми от константи)

```haskell
data Color = Red | Green | Blue
data Day = Mon | Tue | Wed | Thu | Fri | Sat | Sun
```

`Red`, `Green`, `Blue` са **конструктори на данни** - стойности от тип `Color`. Обработваме ги с образци:

```haskell
isWeekend :: Day -> Bool
isWeekend Sat = True
isWeekend Sun = True
isWeekend _   = False
```

### Конструктори с параметри

```haskell
data Shape = Circle Double                 -- радиус
           | Rectangle Double Double       -- ширина, височина
```

```haskell
area :: Shape -> Double
area (Circle r)      = pi * r ^ 2
area (Rectangle w h) = w * h
```

Конструкторът е **функция**: `Circle :: Double -> Shape`, `Rectangle :: Double -> Double -> Shape`.

| Scheme (Седмица 5)                    | Haskell                                  |
| ------------------------------------- | ---------------------------------------- |
| `(define (make-circle r) (list 'circle r))` | конструктор `Circle`               |
| `(circle? s)`                         | образец `(Circle _)`                     |
| `(circle-radius s)`                   | образец `(Circle r)`                     |
| грешен тип → грешка при изпълнение    | грешен тип → грешка при компилация       |

### Записи (records)

Именувани полета - автоматично генерирани селектори:

```haskell
data Student = Student
  { name  :: String
  , fn    :: Int
  , grade :: Double
  }

ivan :: Student
ivan = Student { name = "Иван", fn = 12345, grade = 5.5 }
```

```haskell
ghci> grade ivan
5.5
ghci> grade (ivan { grade = 6.0 })   -- „обновяване“ - създава НОВ запис
6.0
```

### `newtype`

Нов тип с **точно един** конструктор с **едно** поле - без разходи по време на изпълнение:

```haskell
newtype Meters  = Meters Double
newtype Seconds = Seconds Double

speed :: Meters -> Seconds -> Double
speed (Meters m) (Seconds s) = m / s
```

> 💡 За разлика от `type`, `Meters` и `Seconds` са **различни** типове - не можем случайно да ги объркаме.

---

## 4. Параметризирани и рекурсивни типове

Типовете могат да имат **типови параметри**:

```haskell
data Maybe a    = Nothing | Just a         -- вграден
data Either a b = Left a  | Right b        -- вграден
```

И да са **рекурсивни** - да се отнасят към себе си:

```haskell
data List a = Nil | Cons a (List a)        -- като вградения [a]

data Tree a = Empty | Node a (Tree a) (Tree a)
```

Двоичното дърво за търсене от Седмица 6 - вече **типизирано**:

```haskell
insert :: Ord a => a -> Tree a -> Tree a
insert x Empty = Node x Empty Empty
insert x t@(Node v l r)
  | x < v     = Node v (insert x l) r
  | x > v     = Node v l (insert x r)
  | otherwise = t

toList :: Tree a -> [a]
toList Empty        = []
toList (Node v l r) = toList l ++ [v] ++ toList r
```

`Tree` е **конструктор на типове**: `Tree Int`, `Tree String` са типове, а `Tree` сам по себе си - не.

---

## 5. `Maybe` и `Either` - безопасна обработка на грешки

### `Maybe a` - стойност, която може да липсва

```haskell
safeDiv :: Int -> Int -> Maybe Int
safeDiv _ 0 = Nothing
safeDiv x y = Just (x `div` y)

safeHead :: [a] -> Maybe a
safeHead []    = Nothing
safeHead (x:_) = Just x
```

> 💡 Вместо `#f` като „няма резултат“ (`assoc` в Scheme), типът **казва** явно, че резултатът може да липсва - и компилаторът ни **задължава** да обработим и двата случая.

### `Either e a` - резултат или грешка с описание

```haskell
safeDiv' :: Int -> Int -> Either String Int
safeDiv' _ 0 = Left "деление на нула"
safeDiv' x y = Right (x `div` y)
```

По конвенция `Left` е грешката, `Right` - успешният („правилен“) резултат.

### `case` израз

Образци вътре в израз:

```haskell
describe :: Maybe Int -> String
describe m = case m of
  Nothing -> "няма стойност"
  Just 0  -> "нула"
  Just n  -> "числото " ++ show n
```

### Полезни функции

```haskell
import Data.Maybe (fromMaybe, mapMaybe, catMaybes)

fromMaybe 0 (Just 5)     -- 5
fromMaybe 0 Nothing      -- 0
maybe "?" show (Just 3)  -- "3"
either length (* 2) (Left "abc" :: Either String Int)   -- 3
```

---

## 6. Класове

**Клас** (type class) описва множество от типове, които поддържат определени операции:

```haskell
class Eq a where
  (==) :: a -> a -> Bool
  (/=) :: a -> a -> Bool
  x /= y = not (x == y)      -- реализация по подразбиране
```

### Инстанции

```haskell
instance Eq Color where
  Red   == Red   = True
  Green == Green = True
  Blue  == Blue  = True
  _     == _     = False

instance Show Shape where
  show (Circle r)      = "Кръг с радиус " ++ show r
  show (Rectangle w h) = "Правоъгълник " ++ show w ++ "×" ++ show h
```

### `deriving` - автоматични инстанции

```haskell
data Color = Red | Green | Blue
  deriving (Show, Eq, Ord, Enum, Bounded)
```

```haskell
ghci> Red < Blue          -- по реда на деклариране
True
ghci> [Red ..]
[Red,Green,Blue]
ghci> maxBound :: Color
Blue
```

### Инстанции за параметризирани типове

```haskell
instance Show a => Show (Tree a) where
  show Empty        = "."
  show (Node v l r) = "(" ++ show l ++ " " ++ show v ++ " " ++ show r ++ ")"
```

Контекстът `Show a =>` казва: „можем да покажем `Tree a`, ако можем да покажем `a`“.

### Собствен клас

```haskell
class Shape2D a where
  area2D      :: a -> Double
  perimeter2D :: a -> Double

data Square = Square Double

instance Shape2D Square where
  area2D (Square s)      = s * s
  perimeter2D (Square s) = 4 * s
```

### Стандартни класове

| Клас         | Основни операции                   | Суперкласове     |
| ------------ | ---------------------------------- | ---------------- |
| `Eq`         | `==`, `/=`                         | -                |
| `Ord`        | `compare`, `<`, `max`, `min`       | `Eq`             |
| `Show`       | `show`                             | -                |
| `Read`       | `read`                             | -                |
| `Enum`       | `succ`, `pred`, `[a..b]`           | -                |
| `Bounded`    | `minBound`, `maxBound`             | -                |
| `Num`        | `+`, `-`, `*`, `abs`, `fromInteger`| -                |
| `Integral`   | `div`, `mod`                       | `Real`, `Enum`   |
| `Fractional` | `/`, `fromRational`                | `Num`            |
| `Floating`   | `sqrt`, `exp`, `sin`, `pi`         | `Fractional`     |

```
  Eq          Num
   │         ╱   ╲
  Ord       ╱   Fractional
    ╲      ╱        │
     Real         Floating
      │
   Integral  (изисква и Enum)
```

> 💡 Точната йерархия: `Ord` изисква `Eq`; `Real` изисква `Num` и `Ord`; `Integral` изисква `Real` и `Enum`; `Fractional` изисква `Num`; `Floating` изисква `Fractional`. Вижте `:info Integral` в GHCi.

---

## 7. Абстрактни типове данни

**Абстрактен тип данни** (Седмица 5) - потребителят вижда само **операциите**, а не **представянето**. В Haskell това се постига с **модули** и **списък на експортите**:

```haskell
module Queue (Queue, empty, isEmpty, enqueue, dequeue) where
-- експортираме ТИПА Queue, но НЕ конструктора му!

data Queue a = Queue [a] [a]     -- предна част, обърната задна част

empty :: Queue a
empty = Queue [] []

isEmpty :: Queue a -> Bool
isEmpty (Queue [] []) = True
isEmpty _             = False

enqueue :: a -> Queue a -> Queue a
enqueue x (Queue front back) = Queue front (x : back)

dequeue :: Queue a -> Maybe (a, Queue a)
dequeue (Queue [] [])       = Nothing
dequeue (Queue [] back)     = dequeue (Queue (reverse back) [])
dequeue (Queue (x:xs) back) = Just (x, Queue xs back)
```

Извън модула **не можем** да напишем `Queue [1] [2]` или да използваме образец `(Queue f b)` - само `empty`, `enqueue`, `dequeue`. Затова можем да сменим представянето, без да счупим чужд код.

> 💡 Представянето с два списъка дава **амортизирано** $O(1)$ за `enqueue` и `dequeue` - класическа функционална структура от данни.

---

## 8. Формално: защо „алгебрични“?

Типовете образуват **алгебра**. Ако $|T|$ е броят на стойностите на типа $T$:

| Конструкция                | Тип                  | Брой стойности         | Алгебрично |
| -------------------------- | -------------------- | ---------------------- | ---------- |
| Празен тип                 | `data Void`          | $0$                    | $0$        |
| Единичен тип               | `()`                 | $1$                    | $1$        |
| **Сума** (алтернатива)     | `Either a b`         | $\lvert a\rvert + \lvert b\rvert$ | $a + b$    |
| **Произведение** (кортеж)  | `(a, b)`             | $\lvert a\rvert \cdot \lvert b\rvert$ | $a \cdot b$ |
| Функция                    | `a -> b`             | $\lvert b\rvert^{\lvert a\rvert}$ | $b^a$      |

Например: `Bool` $= 1 + 1 = 2$, `Maybe a` $= 1 + a$, `Maybe Bool` $= 3$, `(Bool, Bool)` $= 4$.

Законите на алгебрата важат за типовете (до изоморфизъм): $a \cdot (b + c) = a \cdot b + a \cdot c$ означава, че `(a, Either b c)` и `Either (a, b) (a, c)` носят една и съща информация. А $c^{a \cdot b} = (c^b)^a$ е точно **currying**: `(a, b) -> c` ≅ `a -> b -> c`.

Рекурсивните типове са **уравнения**: `List a` $= 1 + a \cdot \text{List } a$, чието „решение“ е $1 + a + a^2 + a^3 + \cdots$ - списъците с дължина 0, 1, 2, ...

---

## Обобщение

| Концепция          | Синтаксис / пример                              |
| ------------------ | ----------------------------------------------- |
| Параметричен полиморфизъм | `length :: [a] -> Int`                  |
| Ad-hoc полиморфизъм | Класове: `Eq a => ...`                         |
| `type`             | Синоним: `type Point = (Double, Double)`        |
| `data`             | Нов тип: `data Shape = Circle Double \| ...`    |
| Запис              | `data S = S { field :: T }`                     |
| `newtype`          | Нов тип с един конструктор с едно поле          |
| Рекурсивен тип     | `data Tree a = Empty \| Node a (Tree a) (Tree a)` |
| `Maybe` / `Either` | Липсваща стойност / грешка с описание           |
| `case`             | Образци в израз                                 |
| `class` / `instance` | Дефиниране на клас / реализация за тип        |
| `deriving`         | Автоматични инстанции                           |
| АТД                | Модул, който експортира типа без конструкторите |
| Алгебра на типовете | Суми, произведения, експоненти                 |
