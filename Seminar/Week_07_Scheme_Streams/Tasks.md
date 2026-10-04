# Седмица 7 - Задачи за в час

Във всички задачи може да използвате:

```scheme
(define (take-list s n) (stream->list (stream-take s n)))
(define (integers-from n) (stream-cons n (integers-from (+ n 1))))
```

---

## Задача 1: `repeat` и `cycle`

Напишете:

- `(repeat x)` - безкраен поток `x, x, x, ...`
- `(cycle lst)` - безкраен поток, повтарящ елементите на непразния списък `lst`

```scheme
> (take-list (repeat 'a) 3)
'(a a a)
> (take-list (cycle '(a b c)) 7)
'(a b c a b c a)
```

---

## Задача 2: `stream-map2`

Вграденият `stream-map` работи само с един поток. Напишете `(stream-map2 f s1 s2)`, която прилага двуаргументна функция върху съответните елементи на два потока. Чрез нея дефинирайте `(stream-zip s1 s2)`.

```scheme
> (take-list (stream-map2 + (integers-from 1) (integers-from 10)) 3)
'(11 13 15)
> (take-list (stream-zip (integers-from 0) (stream-map sqr (integers-from 0))) 4)
'((0 . 0) (1 . 1) (2 . 4) (3 . 9))
```

---

## Задача 3: Факториели - неявно

Като използвате `stream-map2`, дефинирайте **неявно** потока `facts` от факториелите: $0!, 1!, 2!, \ldots$

```scheme
> (take-list facts 7)
'(1 1 2 6 24 120 720)
```

> 💡 $(n+1)! = n! \cdot (n + 1)$ - кой поток трябва да умножим по `facts`?

---

## Задача 4: `take-while`

Напишете `(take-while p? s)`, която връща **списък** от началните елементи на потока, докато `p?` е изпълнен.

```scheme
> (take-while (lambda (x) (< x 100)) fibs)
'(0 1 1 2 3 5 8 13 21 34 55 89)
```

Без `take-while`, намерете първото число на Фибоначи, по-голямо от 1000.

```scheme
1597
```

---

## Задача 5: Число $e$

$$e = \sum_{k=0}^{\infty} \frac{1}{k!}$$

Като използвате `facts` и `partial-sums` от примерите, дефинирайте потока `e-stream` от частичните суми.

```scheme
> (take-list e-stream 6)
'(1.0 2.0 2.5 2.6666666666666665 2.708333333333333 2.7166666666666663)
> (stream-ref e-stream 15)
2.718281828458995
```

---

## Задача 6: Конвейер от потоци

Без явна рекурсия (само с `stream-map`, `stream-filter`) дефинирайте:

1. Поток от кратните на 3: `3, 6, 9, ...`
2. Поток от нечетните квадрати: `1, 9, 25, 49, ...`

```scheme
> (take-list multiples-of-3 5)
'(3 6 9 12 15)
> (take-list odd-squares 5)
'(1 9 25 49 81)
```
