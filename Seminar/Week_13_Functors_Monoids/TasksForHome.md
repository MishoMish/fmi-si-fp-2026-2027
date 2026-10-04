# Седмица 13 - Задачи за домашна работа

## Задача 1: Доказателство на законите

Докажете законите за функтора за инстанциите на `Maybe` и `Tree`:

1. `fmap id = id`
2. `fmap (f . g) = fmap f . fmap g`

За `Tree` използвайте структурна индукция.

---

## Задача 2: Невалиден функтор

Коя от следните инстанции нарушава законите? Посочете конкретен контрапример.

```haskell
data Counter a = Counter Int a deriving Show

instance Functor Counter where
  fmap f (Counter n x) = Counter (n + 1) (f x)
```

```haskell
newtype Rev a = Rev [a] deriving Show

instance Functor Rev where
  fmap f (Rev xs) = Rev (map f xs)
```

---

## Задача 3: Валидация с натрупване на грешките

`Either` спира при **първата** грешка. Понякога искаме **всички** грешки наведнъж (например при попълване на форма).

```haskell
data Validation e a = Failure e | Success a deriving Show
```

Напишете инстанции `Functor (Validation e)` и `Semigroup e => Applicative (Validation e)`, при които две грешки се **комбинират** с `<>`.

```haskell
data User = User String Int deriving Show

vName :: String -> Validation [String] String
vName n = if null n then Failure ["empty name"] else Success n

vAge :: Int -> Validation [String] Int
vAge a = if a < 0 then Failure ["negative age"] else Success a
```

```haskell
>>> User <$> vName "" <*> vAge (-1)
Failure ["empty name","negative age"]
>>> User <$> vName "ana" <*> vAge 20
Success (User "ana" 20)
```

> 💡 Защо `Validation` **не може** да има законна инстанция на `Monad` (Седмица 14), съвместима с тази?

---

## Задача 4: Бърз Фибоначи с моноид

Матриците $2 \times 2$ с умножение образуват моноид с неутрален елемент единичната матрица. Тъй като

$$\begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix}^n = \begin{pmatrix} F_{n+1} & F_n \\ F_n & F_{n-1} \end{pmatrix}$$

можем да пресметнем $F_n$ за $O(\log n)$ умножения.

```haskell
data M2 = M2 Integer Integer Integer Integer deriving Show
```

1. Напишете инстанции `Semigroup M2` и `Monoid M2`.
2. Напишете `fib :: Integer -> Integer` чрез `stimes` от `Data.Semigroup` (той използва бързо степенуване - Седмица 3!).

```haskell
>>> fib 10
55
>>> fib 100
354224848179261915075
```

---

## Задача 5: Собствен `ZipList`

```haskell
newtype ZL a = ZL [a] deriving Show
```

Напишете `Functor` и `Applicative` за `ZL`, при които `<*>` прилага функциите **покомпонентно**. Какво трябва да е `pure`, за да е изпълнен законът за идентитета `pure id <*> v = v`?

```haskell
>>> (+) <$> ZL [1, 2, 3] <*> ZL [10, 20, 30]
ZL [11,22,33]
```

---

## Задача 6: Функции като апликативен функтор

Функциите `(->) r` са апликативен функтор: `pure x = \_ -> x` и `(f <*> g) x = f x (g x)`.

1. Определете стойността на `((+) <$> (* 2) <*> (+ 10)) 3`.
2. Чрез `<$>` и `<*>` напишете безточково `isPalindrome :: String -> Bool`.
3. Обяснете защо `((==) <*> reverse) . show` (Седмица 11, Пример 7) работи.

---

## Задача 7: Свободният моноид

Докажете, че `foldMap f` е **хомоморфизъм на моноиди**, т.е. за всеки моноид `m` и `f :: a -> m`:

```haskell
foldMap f []         = mempty
foldMap f (xs ++ ys) = foldMap f xs <> foldMap f ys
```

където `foldMap f = foldr (\x acc -> f x <> acc) mempty`.
