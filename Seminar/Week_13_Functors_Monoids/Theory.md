# Функтори, апликативни функтори и моноиди

## Въведение

Досега видяхме много „контейнери“: списъци, `Maybe`, `Either e`, дървета, `IO`. За всеки от тях писахме сходни функции - `map` за списъци, `treeMap` за дървета, обработка на `Just`/`Nothing`... Тази седмица извличаме общите им структури в **класове**:

- **`Functor`** - прилагане на функция „вътре“ в контейнер
- **`Applicative`** - комбиниране на няколко контейнера
- **`Semigroup`** и **`Monoid`** - комбиниране на стойности с асоциативна операция

Това е абстракция от същия вид като `accumulate` в Седмица 3 - но над **типове**, не над функции.

---

## 1. Видове (kinds)

Както стойностите имат **типове**, така типовете имат **видове**:

```haskell
ghci> :k Int
Int :: *                      -- „конкретен“ тип - има стойности
ghci> :k Maybe
Maybe :: * -> *               -- конструктор на типове: по тип дава тип
ghci> :k Either
Either :: * -> * -> *
ghci> :k Either String
Either String :: * -> *       -- частично приложен конструктор
```

Класовете от тази седмица са над конструктори от вид `* -> *` - точно като `Container f` от домашното на Седмица 10.

---

## 2. `Functor`

```haskell
class Functor f where
  fmap :: (a -> b) -> f a -> f b
```

„Ако знам как да превърна `a` в `b`, мога да превърна `f a` в `f b`“ - без да променям **структурата** на `f`.

```haskell
instance Functor [] where
  fmap = map

instance Functor Maybe where
  fmap _ Nothing  = Nothing
  fmap f (Just x) = Just (f x)

instance Functor (Either e) where
  fmap _ (Left e)  = Left e        -- грешката се запазва
  fmap f (Right x) = Right (f x)
```

```haskell
ghci> fmap (+ 1) [1, 2, 3]
[2,3,4]
ghci> fmap (+ 1) (Just 3)
Just 4
ghci> fmap (+ 1) Nothing
Nothing
ghci> fmap (+ 1) (Left "err" :: Either String Int)
Left "err"
ghci> fmap (+ 1) ("label", 3)          -- функтор по втория компонент
("label",4)
```

`<$>` е инфиксен синоним на `fmap`:

```haskell
ghci> (+ 1) <$> Just 3
Just 4
```

### Функциите като функтор

`(->) r` - функциите с аргумент от тип `r` - също са функтор. `fmap` е **композиция**:

```haskell
instance Functor ((->) r) where
  fmap f g = f . g

ghci> fmap (* 2) (+ 1) 5        -- (5 + 1) * 2
12
```

### `IO` е функтор

```haskell
ghci> fmap length getLine       -- действие, което дава дължината на въведения ред
hello
5
```

### Законите за функторите

Всяка инстанция трябва да удовлетворява:

1. **Идентитет:** `fmap id = id`
2. **Композиция:** `fmap (f . g) = fmap f . fmap g`

Законите гарантират, че `fmap` **само** прилага функцията и не променя структурата. Компилаторът **не** ги проверява - отговорност на програмиста е.

> ⚠️ Пример за невалидна инстанция: `fmap f xs = reverse (map f xs)` - типът е правилен, но `fmap id [1, 2] = [2, 1] ≠ [1, 2]`.

---

## 3. `Applicative`

Какво става, ако функцията също е „в контейнера“?

```haskell
ghci> :t fmap (+) (Just 3)
fmap (+) (Just 3) :: Num a => Maybe (a -> a)
```

Получихме `Maybe (a -> a)` - функция в `Maybe`. Как да я приложим към `Just 4`? `fmap` не стига.

```haskell
class Functor f => Applicative f where
  pure  :: a -> f a                       -- „опакова“ стойност
  (<*>) :: f (a -> b) -> f a -> f b       -- прилага опакована функция
```

```haskell
instance Applicative Maybe where
  pure = Just
  Just f  <*> mx = fmap f mx
  Nothing <*> _  = Nothing
```

```haskell
ghci> Just (+ 3) <*> Just 4
Just 7
ghci> (+) <$> Just 3 <*> Just 4
Just 7
ghci> (+) <$> Just 3 <*> Nothing
Nothing
```

Шаблонът `f <$> x <*> y <*> z` прилага **обикновена** функция на няколко аргумента към **опаковани** стойности. Ако някой аргумент е `Nothing` - резултатът е `Nothing`.

### Списъците: всички комбинации

```haskell
instance Applicative [] where
  pure x    = [x]
  fs <*> xs = [f x | f <- fs, x <- xs]
```

```haskell
ghci> [(+ 1), (* 2)] <*> [10, 20]
[11,21,20,40]
ghci> (,) <$> [1, 2] <*> "ab"
[(1,'a'),(1,'b'),(2,'a'),(2,'b')]
```

> 💡 Списъкът моделира **недетерминирано** изчисление - всички възможни резултати. За покомпонентно комбиниране има `ZipList`: `getZipList ((+) <$> ZipList [1,2,3] <*> ZipList [10,20,30])` → `[11,22,33]`.

### `IO`: последователни действия

```haskell
greeting :: IO String
greeting = (++) <$> getLine <*> getLine     -- прочита два реда и ги слепва
```

### Полезни функции

```haskell
liftA2   :: Applicative f => (a -> b -> c) -> f a -> f b -> f c
sequenceA :: Applicative f => [f a] -> f [a]
traverse  :: Applicative f => (a -> f b) -> [a] -> f [b]
```

```haskell
ghci> sequenceA [Just 1, Just 2]
Just [1,2]
ghci> sequenceA [Just 1, Nothing]
Nothing
ghci> traverse (\x -> if x > 0 then Just x else Nothing) [1, 2, 3]
Just [1,2,3]
```

### Законите за апликативните функтори

| Закон          | Формулировка                                       |
| -------------- | -------------------------------------------------- |
| Идентитет      | `pure id <*> v = v`                                |
| Хомоморфизъм   | `pure f <*> pure x = pure (f x)`                   |
| Размяна        | `u <*> pure y = pure ($ y) <*> u`                  |
| Композиция     | `pure (.) <*> u <*> v <*> w = u <*> (v <*> w)`     |

Следствие: `fmap f x = pure f <*> x`.

---

## 4. `Semigroup` и `Monoid`

**Полугрупа** - тип с **асоциативна** бинарна операция:

```haskell
class Semigroup a where
  (<>) :: a -> a -> a
-- закон: (x <> y) <> z = x <> (y <> z)
```

**Моноид** - полугрупа с **неутрален елемент**:

```haskell
class Semigroup a => Monoid a where
  mempty  :: a
  mconcat :: [a] -> a
  mconcat = foldr (<>) mempty
-- закони: mempty <> x = x,  x <> mempty = x
```

| Тип            | `<>`                    | `mempty`   |
| -------------- | ----------------------- | ---------- |
| `[a]`          | `++`                    | `[]`       |
| `Sum a`        | `+`                     | `0`        |
| `Product a`    | `*`                     | `1`        |
| `Min a`, `Max a` | `min`, `max`          | (само полугрупи) |
| `All`          | `&&`                    | `True`     |
| `Any`          | `\|\|`                  | `False`    |
| `First a`      | първият `Just`          | `Nothing`  |
| `Ordering`     | първото ≠ `EQ`          | `EQ`       |
| `Maybe a`      | комбинира съдържанието  | `Nothing`  |

### Защо `Sum` и `Product`?

Числата са моноид по **два** начина - с `+` и `0`, и с `*` и `1`. Един тип може да има само **една** инстанция на клас, затова използваме **`newtype`** обвивки (Седмица 10) от `Data.Monoid`:

```haskell
newtype Sum a = Sum { getSum :: a }

ghci> getSum (foldMap Sum [1..10])
55
ghci> getProduct (foldMap Product [1..5])
120
ghci> getAll (foldMap (All . even) [2, 4, 6])
True
```

### `Ordering` като моноид

`compare x y <> compare a b` - сравни по първия критерий, при равенство - по втория:

```haskell
ghci> import Data.List (sortBy)
ghci> import Data.Ord (comparing)
ghci> sortBy (comparing length <> compare) ["bb", "a", "ccc", "ab", "b"]
["a","b","ab","bb","ccc"]
```

> 💡 Тук `<>` комбинира **функции**: ако `b` е моноид, то и `a -> b` е моноид - `(f <> g) x = f x <> g x`.

---

## 5. `Foldable` и `foldMap`

`foldr` работи не само за списъци - и за `Maybe`, дървета, `Either e`... Класът `Foldable` обобщава това:

```haskell
class Foldable t where
  foldMap :: Monoid m => (a -> m) -> t a -> m
  foldr   :: (a -> b -> b) -> b -> t a -> b
  -- и още: sum, product, length, elem, maximum, toList, null, ...
```

Достатъчно е да дефинираме **`foldMap`** - всичко останало идва наготово:

```haskell
data Tree a = Empty | Node a (Tree a) (Tree a)

instance Foldable Tree where
  foldMap _ Empty        = mempty
  foldMap f (Node v l r) = foldMap f l <> f v <> foldMap f r
```

```haskell
ghci> sum tree
ghci> length tree
ghci> elem 5 tree
ghci> maximum tree
ghci> foldr (:) [] tree          -- inorder обхождане
```

> 💡 `foldMap f` „превежда“ всеки елемент в моноид с `f` и ги комбинира с `<>`. Асоциативността позволява комбинирането да става в **произволен** ред на групиране - например паралелно.

---

## 6. Формално: функтори и моноиди в математиката

### Моноид

**Моноид** е тройка $(M, \cdot, e)$, където $\cdot : M \times M \to M$ е асоциативна и $e$ е неутрален елемент. Примери: $(\mathbb{N}, +, 0)$, $(\mathbb{N}, \cdot, 1)$, $(A^*, \text{конкатенация}, \varepsilon)$ - думите над азбуката $A$.

Списъците `[a]` са **свободният моноид** над `a`: всеки моноид $M$ и функция $f : a \to M$ определят **единствен** хомоморфизъм $h : [a] \to M$ с $h([x]) = f(x)$. Това е точно `foldMap f`! Затова `foldMap` е толкова универсален.

### Функтор (в теорията на категориите)

**Категория** има обекти и стрелки (морфизми) между тях, с композиция и идентитет. В Haskell: обектите са **типовете**, стрелките - **функциите**.

**Функтор** $F$ съпоставя:

- на всеки обект $A$ - обект $F(A)$ (в Haskell: `Maybe` праща `Int` в `Maybe Int`)
- на всяка стрелка $f : A \to B$ - стрелка $F(f) : F(A) \to F(B)$ (в Haskell: `fmap f`)

и **запазва** структурата:

$$F(\mathrm{id}_A) = \mathrm{id}_{F(A)} \qquad F(g \circ f) = F(g) \circ F(f)$$

Това са точно законите за `Functor`. Апликативните функтори съответстват на т.нар. **моноидални функтори** - функтори, съвместими с произведението (`liftA2 (,) :: f a -> f b -> f (a, b)`).

> 💡 За повече: B. Pierce, *Basic Category Theory for Computer Scientists* (допълнителна литература).

---

## Обобщение

| Концепция      | Сигнатура / пример                                  |
| -------------- | --------------------------------------------------- |
| Вид (kind)     | `Maybe :: * -> *`                                   |
| `Functor`      | `fmap :: (a -> b) -> f a -> f b`, `<$>`             |
| Закони за `Functor` | `fmap id = id`, `fmap (f . g) = fmap f . fmap g` |
| `Applicative`  | `pure :: a -> f a`, `<*> :: f (a -> b) -> f a -> f b` |
| Шаблон         | `f <$> x <*> y`                                     |
| `sequenceA`, `traverse` | Обръщане на `[f a]` в `f [a]`              |
| `Semigroup`    | `<>` - асоциативна операция                         |
| `Monoid`       | `mempty` - неутрален елемент                        |
| `newtype` обвивки | `Sum`, `Product`, `All`, `Any`, `First`, `Max`   |
| `Foldable`     | `foldMap` → `sum`, `length`, `elem`, ... наготово   |
| Свободен моноид | Списъците; `foldMap` е хомоморфизмът               |
