# Седмица 10 - Задачи за домашна работа

## Задача 1: Аритметични изрази

```haskell
data Expr = Lit Int
          | Var String
          | Add Expr Expr
          | Mul Expr Expr
          | Neg Expr
```

Напишете:

```haskell
eval :: [(String, Int)] -> Expr -> Maybe Int    -- Nothing, ако има недефинирана променлива
pretty :: Expr -> String                        -- с минимално нужните скоби
simplify :: Expr -> Expr                        -- x + 0 = x, x * 1 = x, x * 0 = 0, -(-x) = x
```

```haskell
>>> eval [("x", 5)] (Add (Lit 1) (Mul (Var "x") (Lit 2)))
Just 11
>>> eval [] (Var "y")
Nothing
>>> pretty (Mul (Add (Lit 1) (Var "x")) (Lit 2))
"(1 + x) * 2"
>>> pretty (simplify (Add (Mul (Var "x") (Lit 1)) (Lit 0)))
"x"
```

> 💡 Сравнете със символното диференциране от Седмица 5. Добавете и `deriv :: String -> Expr -> Expr`.

---

## Задача 2: Естествени числа на Пеано

```haskell
data Nat = Zero | Succ Nat
```

1. Напишете `toInt :: Nat -> Int` и `fromInt :: Int -> Nat`.
2. Напишете `add` и `mul` **без** да преобразувате към `Int`.
3. Напишете инстанции на `Eq`, `Ord`, `Show` (показва числото) и `Num`.

```haskell
>>> Succ (Succ Zero) + Succ Zero
3
>>> fromInt 3 * fromInt 4
12
>>> Succ Zero < Succ (Succ Zero)
True
```

---

## Задача 3: Дърво с произволна разклоненост

```haskell
data Rose a = Rose a [Rose a]
```

Напишете (сравнете с Пример 4 от Седмица 6):

```haskell
roseSize :: Rose a -> Int
roseDepth :: Rose a -> Int
roseFlatten :: Rose a -> [a]          -- preorder
roseLeaves :: Rose a -> [a]
```

```haskell
>>> let r = Rose 1 [Rose 2 [Rose 5 [], Rose 6 []], Rose 3 [], Rose 4 [Rose 7 [Rose 8 []]]]
>>> roseSize r
8
>>> roseDepth r
4
>>> roseFlatten r
[1,2,5,6,3,4,7,8]
>>> roseLeaves r
[5,6,3,8]
```

---

## Задача 4: Абстрактен тип „множество“

Напишете модул `IntSet`, който експортира **само**:

```haskell
module IntSet (IntSet, empty, insert, member, delete, toList, fromList) where
```

Представете множеството като двоично дърво за търсене. Напишете кратка програма, която използва модула. Може ли тя да построи невалидно дърво? Защо?

---

## Задача 5: Клас `Container`

```haskell
class Container f where
  empty  :: f a
  insert :: a -> f a -> f a
  toL    :: f a -> [a]
```

Напишете инстанции за:

- `newtype Stack a = Stack [a]` - `toL` връща елементите в обратен ред на добавяне
- `newtype Queue a = Queue ([a], [a])` - `toL` връща елементите в реда на добавяне

```haskell
>>> toL (insert 3 (insert 2 (insert 1 empty)) :: Stack Int)
[3,2,1]
>>> toL (insert 3 (insert 2 (insert 1 empty)) :: Queue Int)
[1,2,3]
```

> 💡 Тук класът е над **конструктор на типове** `f` (а не над тип `a`). Това е подготовка за `Functor` в Седмица 13.

---

## Задача 6: Алгебра на типовете

1. Колко стойности има всеки от типовете: `Maybe (Maybe Bool)`, `Either Bool (Bool, Bool)`, `Bool -> Bool`, `Maybe Bool -> Bool`?
2. Напишете функциите, които доказват изоморфизма:

```haskell
distribute :: (a, Either b c) -> Either (a, b) (a, c)
factor     :: Either (a, b) (a, c) -> (a, Either b c)
```

и проверете, че `factor . distribute = id` и `distribute . factor = id`.

3. Напишете изоморфизма между `Either () a` и `Maybe a`.
