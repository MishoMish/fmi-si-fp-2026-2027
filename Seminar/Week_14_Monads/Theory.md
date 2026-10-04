# Монади

## Въведение

В Седмица 12 използвахме `do` за подреждане на `IO` действия. В Седмица 13 видяхме, че `Applicative` комбинира **независими** изчисления в контекст: `f <$> x <*> y`. Но какво, ако втората стъпка **зависи** от резултата на първата?

**Монадата** е шаблон за **верижно свързване** на изчисления, всяко от които връща резултат в контекст (възможен неуспех, грешка, много резултати, странични ефекти, състояние...).

---

## 1. Проблемът: верижни `Maybe` изчисления

```haskell
safeDiv :: Int -> Int -> Maybe Int
safeDiv _ 0 = Nothing
safeDiv x y = Just (x `div` y)

-- (a / b) / c  - всяко деление може да се провали
calc :: Int -> Int -> Int -> Maybe Int
calc a b c = case safeDiv a b of
  Nothing -> Nothing
  Just x  -> case safeDiv x c of
    Nothing -> Nothing
    Just y  -> Just y
```

Повтарящ се шаблон: „ако е `Nothing` - спри, ако е `Just x` - продължи с `x`“. С `<*>` не става - второто деление използва **резултата** `x` от първото.

---

## 2. Операторът `>>=` (bind)

```haskell
(>>=) :: Maybe a -> (a -> Maybe b) -> Maybe b
Nothing >>= _ = Nothing
Just x  >>= f = f x
```

„Вземи резултата отляво и го подай на функцията отдясно, която продължава изчислението.“

```haskell
calc :: Int -> Int -> Int -> Maybe Int
calc a b c = safeDiv a b >>= \x -> safeDiv x c

-- или още по-кратко
calc a b c = safeDiv a b >>= (`safeDiv` c)
```

| Комбинатор | Тип                                  | Втората стъпка зависи от първата? |
| ---------- | ------------------------------------ | --------------------------------- |
| `fmap`     | `(a -> b) -> m a -> m b`             | Няма втора стъпка с контекст      |
| `<*>`      | `m (a -> b) -> m a -> m b`           | Не - структурата е фиксирана      |
| `>>=`      | `m a -> (a -> m b) -> m b`           | **Да** - `f` вижда резултата      |

---

## 3. Класът `Monad`

```haskell
class Applicative m => Monad m where
  return :: a -> m a              -- същото като pure
  (>>=)  :: m a -> (a -> m b) -> m b
  (>>)   :: m a -> m b -> m b     -- игнорира резултата отляво
  m >> k = m >>= \_ -> k
```

```haskell
instance Monad Maybe where
  Nothing >>= _ = Nothing
  Just x  >>= f = f x

instance Monad (Either e) where
  Left e  >>= _ = Left e
  Right x >>= f = f x

instance Monad [] where
  xs >>= f = concatMap f xs
```

> ⚠️ `return` **не** прекратява изчислението! То само „опакова“ стойност: `return x = Just x` за `Maybe`, `[x]` за списъци.

### Йерархията

```
Functor  ⊂  Applicative  ⊂  Monad
 fmap        pure, <*>       >>=
```

Всяка монада е апликативен функтор, а всеки апликативен - функтор:

```haskell
fmap f m  = m >>= \x -> return (f x)
mf <*> mx = mf >>= \f -> mx >>= \x -> return (f x)
```

---

## 4. `do`-нотация

`do` е **синтактична захар** за вериги от `>>=` - за **всяка** монада, не само за `IO`:

```haskell
calc a b c = do
  x <- safeDiv a b
  y <- safeDiv x c
  return y
```

| `do` запис                  | Превежда се до                       |
| --------------------------- | ------------------------------------ |
| `do { x <- m; rest }`       | `m >>= \x -> do { rest }`            |
| `do { m; rest }`            | `m >> do { rest }`                   |
| `do { let x = e; rest }`    | `let x = e in do { rest }`           |
| `do { m }`                  | `m`                                  |

> 💡 Вече разбираме какво правеше `do` в Седмица 12: `IO` е монада, `<-` е `>>=`, а редовете без `<-` се свързват с `>>`.

---

## 5. Примери за монади

### `Maybe` - изчисление, което може да се провали

```haskell
lookupAge :: String -> Maybe Int
lookupAge name = do
  idx <- lookup name [("ana", 1), ("ivan", 2)]
  lookup idx [(1, 25), (2, 30)]
```

### `Either e` - изчисление с описание на грешката

```haskell
parseAge :: String -> Either String Int
parseAge s = do
  n <- maybe (Left ("не е число: " ++ s)) Right (readMaybe s)
  if n < 0 then Left "отрицателна възраст" else return n
```

### Списък - недетерминирано изчисление

`>>=` пробва **всички** възможности - `do` блокът е list comprehension:

```haskell
pairs :: [(Int, Char)]
pairs = do
  n <- [1, 2]
  c <- "ab"
  return (n, c)
-- [(1,'a'),(1,'b'),(2,'a'),(2,'b')]  ≡  [(n, c) | n <- [1,2], c <- "ab"]
```

Филтриране - с `guard` от `Control.Monad`:

```haskell
import Control.Monad (guard)

pythagorean :: Int -> [(Int, Int, Int)]
pythagorean n = do
  c <- [1 .. n]
  b <- [1 .. c]
  a <- [1 .. b]
  guard (a * a + b * b == c * c)
  return (a, b, c)
```

### `IO` - странични ефекти

```haskell
main :: IO ()
main = getLine >>= \name -> putStrLn ("Здравей, " ++ name)
```

---

## 6. Полезни функции за монади

| Функция      | Тип                                         | Описание                                   |
| ------------ | ------------------------------------------- | ------------------------------------------ |
| `mapM`       | `Monad m => (a -> m b) -> [a] -> m [b]`     | `map` + свързване (= `traverse`)           |
| `mapM_`      | `Monad m => (a -> m b) -> [a] -> m ()`      | Само ефектите                              |
| `sequence`   | `Monad m => [m a] -> m [a]`                 | Свързва списък от изчисления               |
| `forM`, `forM_` | `mapM`, `mapM_` с обърнати аргументи     |                                            |
| `when`, `unless` | `Bool -> m () -> m ()`                  | Условно изпълнение                         |
| `replicateM` | `Int -> m a -> m [a]`                       | Повторение                                 |
| `foldM`      | `(b -> a -> m b) -> b -> [a] -> m b`        | `foldl` с контекст                         |
| `join`       | `m (m a) -> m a`                            | „Изравнява“ два слоя                       |
| `(>=>)`      | `(a -> m b) -> (b -> m c) -> (a -> m c)`    | Композиция на монадни функции (Клайсли)    |

```haskell
ghci> mapM (\x -> if x > 0 then Just x else Nothing) [1, 2, 3]
Just [1,2,3]
ghci> sequence [[1, 2], [3, 4]]
[[1,3],[1,4],[2,3],[2,4]]
ghci> foldM safeDiv 1000 [2, 5, 10]
Just 10
ghci> foldM safeDiv 1000 [2, 0, 10]
Nothing
ghci> join [[1, 2], [3]]
[1,2,3]
ghci> join (Just (Just 5))
Just 5
```

---

## 7. Приложение: монадата `State`

Как да моделираме **състояние** (като `set!` в Scheme) в чист език? Функция, която приема старото състояние и връща резултат и **новото** състояние:

```haskell
newtype State s a = State { runState :: s -> (a, s) }
```

Свързването предава състоянието от една стъпка на следващата - автоматично:

```haskell
instance Functor (State s) where
  fmap f (State g) = State $ \s -> let (a, s') = g s in (f a, s')

instance Applicative (State s) where
  pure a = State $ \s -> (a, s)
  State f <*> State g = State $ \s ->
    let (h, s1) = f s
        (a, s2) = g s1
    in (h a, s2)

instance Monad (State s) where
  State g >>= f = State $ \s ->
    let (a, s1) = g s
    in runState (f a) s1

get :: State s s
get = State $ \s -> (s, s)

put :: s -> State s ()
put s = State $ \_ -> ((), s)

modify :: (s -> s) -> State s ()
modify f = State $ \s -> ((), f s)
```

Сега „императивен“ код е чиста функция:

```haskell
-- Брояч - сравнете с make-counter от Scheme
tick :: State Int Int
tick = do
  n <- get
  put (n + 1)
  return n

threeTicks :: State Int [Int]
threeTicks = do
  a <- tick
  b <- tick
  c <- tick
  return [a, b, c]
```

```haskell
ghci> runState threeTicks 10
([10,11,12],13)
```

> 💡 Това е моделът `World -> (a, World)` от Седмица 12 - `IO` е „`State` над целия свят“. Готова `State` има в пакета `mtl` (`Control.Monad.State`).

---

## 8. Законите за монадите

| Закон              | Формулировка                                  |
| ------------------ | --------------------------------------------- |
| Ляв идентитет      | `return a >>= f  =  f a`                      |
| Десен идентитет    | `m >>= return  =  m`                          |
| Асоциативност      | `(m >>= f) >>= g  =  m >>= (\x -> f x >>= g)` |

Чрез Клайсли композицията `(f >=> g) x = f x >>= g` законите стават особено ясни:

$$\texttt{return} \mathbin{>=>} f = f \qquad f \mathbin{>=>} \texttt{return} = f \qquad (f \mathbin{>=>} g) \mathbin{>=>} h = f \mathbin{>=>} (g \mathbin{>=>} h)$$

т.е. функциите `a -> m b` с `>=>` и `return` образуват **моноид** (по-точно категория - **категорията на Клайсли**).

Законите гарантират, че `do` блоковете се държат „разумно“:

- ляв идентитет: `do { x <- return a; f x }` = `f a`
- десен идентитет: `do { x <- m; return x }` = `m`
- асоциативност: можем да **изнасяме** части от `do` блок в отделни функции, без да променим смисъла

---

## 9. Формално: алтернативна дефиниция

Монадата може да се дефинира и чрез `fmap`, `return` и **`join`**:

$$\texttt{join} :: m\ (m\ a) \to m\ a \qquad m \mathbin{>>=} f = \texttt{join}\ (\texttt{fmap}\ f\ m)$$

| Монада     | `join`                               |
| ---------- | ------------------------------------ |
| `Maybe`    | `Just (Just x) ↦ Just x`, иначе `Nothing` |
| `[]`       | `concat`                             |
| `Either e` | `Right (Right x) ↦ Right x`          |

В теорията на категориите монада е тройка $(T, \eta, \mu)$ - функтор $T$ и естествени трансформации $\eta : \mathrm{Id} \to T$ (`return`) и $\mu : T \circ T \to T$ (`join`), удовлетворяващи законите за асоциативност и единица. Структурата е същата като на **моноид** ($\mu$ - умножение, $\eta$ - единица), но „над функтори“ - оттук и знаменитата фраза „монадата е моноид в категорията на ендофункторите“.

---

## Обобщение

| Концепция         | Описание                                                    |
| ----------------- | ----------------------------------------------------------- |
| `>>=` (bind)      | Свързва изчисление с функция, която зависи от резултата му  |
| `return`          | Опакова стойност; **не** прекратява изчислението            |
| `>>`              | Свързване без използване на резултата                       |
| `do`-нотация      | Захар за вериги от `>>=` - за всяка монада                  |
| `Maybe`           | Възможен неуспех                                            |
| `Either e`        | Грешка с описание                                           |
| `[]`              | Недетерминизъм - всички възможности; `guard`                |
| `IO`              | Странични ефекти                                            |
| `State s`         | Чисто моделирано състояние                                  |
| `mapM`, `sequence`, `foldM` | Функции от по-висок ред за монади                 |
| `>=>`, `join`     | Клайсли композиция, изравняване                             |
| Закони            | Ляв/десен идентитет, асоциативност                          |
| Functor ⊂ Applicative ⊂ Monad | Йерархия на абстракциите                        |
