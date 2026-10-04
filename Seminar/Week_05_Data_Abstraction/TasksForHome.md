# Седмица 5 - Задачи за домашна работа

## Задача 1: Интервална аритметика

Интервал $[a, b]$ представя неточна стойност (например съпротивление $6.8\,\Omega \pm 10\%$). Реализирайте:

- `(make-interval a b)`, `(lower-bound i)`, `(upper-bound i)`
- `(add-interval x y)` - $[a_1 + a_2, b_1 + b_2]$
- `(mul-interval x y)` - минимумът и максимумът на четирите произведения на краищата
- `(div-interval x y)` - умножение по $[1/b_2, 1/a_2]$; съобщете за грешка, ако $y$ съдържа 0
- `(width i)` - половината от дължината на интервала

```scheme
> (add-interval (make-interval 1 2) (make-interval 3 5))
'(4 . 7)
> (mul-interval (make-interval -1 2) (make-interval 3 5))
'(-5 . 10)
> (div-interval (make-interval 1 2) (make-interval -1 1))
; div-interval: division by an interval spanning zero
```

---

## Задача 2: Множества като неподредени списъци

Множество се представя като списък **без повторения**. Напишете:

- `(element-of-set? x s)`
- `(adjoin-set x s)`
- `(union-set a b)`
- `(intersection-set a b)`

```scheme
> (union-set '(1 2 3) '(2 3 4))
'(1 2 3 4)
> (intersection-set '(1 2 3) '(2 3 4))
'(2 3)
```

Каква е сложността на `intersection-set`?

---

## Задача 3: Множества като подредени списъци

Сега множествата са **сортирани** списъци от числа. Пренапишете `element-of-set?` и `intersection-set`, като използвате наредбата. Постигнете $\Theta(n)$ за `intersection-set`.

> 💡 Сравнете първите елементи на двата списъка - кой може да бъде отхвърлен?

---

## Задача 4: Комплексни числа

Реализирайте комплексни числа в **алгебричен** вид: `(make-complex re im)`, `(real-part z)`, `(imag-part z)`. После напишете `(add-complex z w)`, `(mul-complex z w)` и `(magnitude* z)`.

```scheme
> (magnitude* (make-complex 3 4))
5
> (mul-complex (make-complex 0 1) (make-complex 0 1))     ; i · i
'(-1 . 0)
```

Добавете **второ** представяне - тригонометричен вид `(make-from-mag-ang r φ)`. Кои функции зависят от представянето и кои не?

---

## Задача 5: Двойки от числа

Покажете, че двойки от неотрицателни цели числа могат да се представят само с числа: двойката $(a, b)$ се представя като $2^a 3^b$. Напишете `num-cons`, `num-car`, `num-cdr`.

```scheme
> (num-cons 3 2)
72
> (num-car 72)
3
> (num-cdr 72)
2
```

---

## Задача 6: Пътища в дърво

Напишете `(tree-paths t)`, която връща списък от всички пътища от корена до листата.

```scheme
> (tree-paths t)
'((5 3 1) (5 3 4) (5 8 9))
```

---

## Задача 7: Балансирано дърво

Дърво е **балансирано**, ако за всеки връх височините на лявото и дясното поддърво се различават с най-много 1. Напишете `(balanced? t)`.

```scheme
> (balanced? t)
#t
> (balanced? (make-tree 1 (make-tree 2 (leaf 3) empty-tree) empty-tree))
#f
```
