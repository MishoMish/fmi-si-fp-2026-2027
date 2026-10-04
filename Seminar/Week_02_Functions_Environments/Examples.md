# Седмица 2 - Примери

## Пример 1: Функции с `cond`

```scheme
#lang racket

; Оценка по точки
(define (grade points)
  (cond ((>= points 90) 6)
        ((>= points 75) 5)
        ((>= points 60) 4)
        ((>= points 50) 3)
        (else 2)))

; Брой реални корени на ax² + bx + c = 0 (a ≠ 0)
(define (roots-count a b c)
  (define d (- (* b b) (* 4 a c)))
  (cond ((> d 0) 2)
        ((= d 0) 1)
        (else 0)))
```

```scheme
> (grade 82)
5
> (roots-count 1 -3 2)
2
> (roots-count 1 2 1)
1
> (roots-count 1 0 1)
0
```

> 💡 Вложеният `(define d ...)` ни спестява трикратното пресмятане на дискриминантата.

---

## Пример 2: `and` и `or` като условия

```scheme
(define (divides? d n) (= (remainder n d) 0))

(define (leap-year? y)
  (or (and (divides? 4 y) (not (divides? 100 y)))
      (divides? 400 y)))

; Безопасно деление: ако b е 0, (/ a b) не се оценява
(define (safe-div-positive? a b)
  (and (not (zero? b)) (positive? (/ a b))))
```

```scheme
> (leap-year? 2024)
#t
> (leap-year? 1900)
#f
> (leap-year? 2000)
#t
> (safe-div-positive? 5 0)
#f
```

---

## Пример 3: Апликативен срещу нормален ред

```scheme
(define (p) (p))          ; безкрайна рекурсия

(define (test x y)
  (if (= x 0) 0 y))

(test 0 (p))
```

- **Апликативен ред** (Scheme): преди да приложи `test`, интерпретаторът оценява `(p)` → **зацикля**.
- **Нормален ред**: заместваме `(test 0 (p))` → `(if (= 0 0) 0 (p))` → `0`. Вторият аргумент **никога** не се оценява.

> 💡 Така можем експериментално да установим кой ред използва даден интерпретатор. В Haskell (мързелив) аналогичната функция връща `0` - ще го видим в Седмица 11.

---

## Пример 4: Защо `if` е специална форма?

Да опитаме да дефинираме `if` като обикновена функция:

```scheme
(define (new-if predicate then-clause else-clause)
  (cond (predicate then-clause)
        (else else-clause)))

> (new-if (= 2 3) 0 5)
5
> (new-if (= 1 1) 0 5)
0
```

Изглежда работи! Но:

```scheme
(define (fact n)
  (new-if (= n 0)
          1
          (* n (fact (- n 1)))))

> (fact 3)
; зацикля!
```

Тъй като `new-if` е функция, **и трите** аргумента се оценяват преди прилагането - включително `(* n (fact (- n 1)))`, дори когато `n = 0`. Рекурсията никога не спира.

---

## Пример 5: Метод на Нютон с блокова структура

```scheme
(define (square x) (* x x))
(define (average a b) (/ (+ a b) 2))

(define (my-sqrt x)
  (define (good-enough? guess)
    (< (abs (- (square guess) x)) 0.001))
  (define (improve guess)
    (average guess (/ x guess)))
  (define (sqrt-iter guess)
    (if (good-enough? guess)
        guess
        (sqrt-iter (improve guess))))
  (sqrt-iter 1.0))
```

```scheme
> (my-sqrt 9)
3.00009155413138
> (my-sqrt 2)
1.4142156862745097
> (my-sqrt 137)
11.704699917758145
```

Проследяване на приближенията за `(my-sqrt 2)`:

| guess    | x/guess  | Ново приближение |
| -------- | -------- | ---------------- |
| 1.0      | 2.0      | 1.5              |
| 1.5      | 1.3333   | 1.4167           |
| 1.4167   | 1.4118   | 1.4142           |

---

## Пример 6: Диаграма на средите

```scheme
(define (square x) (* x x))
(define (sum-of-squares x y) (+ (square x) (square y)))

(sum-of-squares 3 4)
```

```
Глобална среда
┌────────────────────────────────────────┐
│ square: <функция (x) (* x x)>          │
│ sum-of-squares: <функция (x y) ...>    │
└────────────────────────────────────────┘
     ▲                 ▲               ▲
     │                 │               │
E1 ┌─┴──────┐    E2 ┌──┴───┐     E3 ┌──┴───┐
   │ x: 3   │       │ x: 3 │        │ x: 4 │
   │ y: 4   │       └──────┘        └──────┘
   └────────┘      (square x)      (square y)
 (sum-of-squares 3 4)
```

- `E1` - рамка за `sum-of-squares`; в нея се оценява `(+ (square x) (square y))`
- `E2`, `E3` - рамки за двете извиквания на `square`
- **Всички три** рамки сочат към **глобалната** среда, защото там са създадени функциите - а не `E2` и `E3` към `E1`!

> 💡 Затова `x` в `E2` и `x` в `E1` не си пречат - те са в различни рамки.

---

## Пример 7: Среди при блокова структура

```scheme
(my-sqrt 2)
```

```
Глобална среда ┌───────────────────────────┐
               │ my-sqrt, square, average  │
               └───────────────────────────┘
                            ▲
               E1 ┌─────────┴───────────────┐
                  │ x: 2                    │
                  │ good-enough?: <функция> │──┐ всички вложени
                  │ improve: <функция>      │  │ функции сочат
                  │ sqrt-iter: <функция>    │  │ към E1
                  └─────────────────────────┘◄─┘
                            ▲
               E2 ┌─────────┴───────┐
                  │ guess: 1.0      │   (sqrt-iter 1.0)
                  └─────────────────┘
```

Когато `good-enough?` търси `x`, тя го намира в `E1` - средата, в която е **дефинирана**.
