# Седмица 3 - Примери

## Пример 1: Факториел - два процеса

```scheme
#lang racket

; Рекурсивен процес
(define (fact n)
  (if (= n 0)
      1
      (* n (fact (- n 1)))))

; Итеративен процес
(define (fact-iter n)
  (define (iter i result)
    (if (> i n)
        result
        (iter (+ i 1) (* result i))))
  (iter 1 1))
```

```scheme
> (fact 5)
120
> (fact-iter 20)
2432902008176640000
```

> 💡 Шаблон за итеративен процес: помощна функция с **акумулатор** + брояч; началните стойности са „празният резултат“ (`1` за произведение, `0` за сума).

---

## Пример 2: Фибоначи

```scheme
(define (fib n)
  (cond ((= n 0) 0)
        ((= n 1) 1)
        (else (+ (fib (- n 1)) (fib (- n 2))))))

(define (fib-iter n)
  (define (iter a b count)
    (if (= count 0)
        a
        (iter b (+ a b) (- count 1))))
  (iter 0 1 n))
```

```scheme
> (fib 10)
55
> (fib-iter 50)
12586269025
> (time (fib 30))
cpu time: ~50 real time: ~50 gc time: 0
832040
```

> ⚠️ `time` е удобен начин да усетите разликата между експоненциален и линеен алгоритъм. Опитайте `(fib 35)` и `(fib-iter 35)`.

---

## Пример 3: Броене на начините за размяна на пари

По колко начина можем да разменим 1 евро (100 цента) с монети от 1, 2, 5, 10, 20 и 50 цента?

Идея: начините да разменим сумата `a` с `k` вида монети =

- начините да разменим `a` **без** първия вид монета, **плюс**
- начините да разменим `a - d` с всички `k` вида, където `d` е стойността на първия вид.

```scheme
(define (count-change amount)
  (define (first-denomination kinds)
    (cond ((= kinds 1) 1)
          ((= kinds 2) 2)
          ((= kinds 3) 5)
          ((= kinds 4) 10)
          ((= kinds 5) 20)
          ((= kinds 6) 50)))
  (define (cc amount kinds)
    (cond ((= amount 0) 1)
          ((or (< amount 0) (= kinds 0)) 0)
          (else (+ (cc amount (- kinds 1))
                   (cc (- amount (first-denomination kinds)) kinds)))))
  (cc amount 6))
```

```scheme
> (count-change 100)
4562
```

> 💡 Естествено дървовидно-рекурсивно решение. Итеративен вариант не е очевиден!

---

## Пример 4: Обобщено сумиране

```scheme
(define (inc x) (+ x 1))
(define (id x) x)
(define (cube x) (* x x x))

(define (sum term a next b)
  (if (> a b)
      0
      (+ (term a) (sum term (next a) next b))))
```

```scheme
> (sum id 1 inc 100)                  ; 1 + 2 + ... + 100
5050
> (sum cube 1 inc 10)                 ; 1³ + ... + 10³
3025
> (* 8 (sum (lambda (x) (/ 1.0 (* x (+ x 2))))
            1
            (lambda (x) (+ x 4))
            1000))                    ; ≈ π
3.139592655589783
```

### Приближено интегриране

$$\int_a^b f \approx \left[ f\left(a + \tfrac{dx}{2}\right) + f\left(a + dx + \tfrac{dx}{2}\right) + \cdots \right] dx$$

```scheme
(define (integral f a b dx)
  (* (sum f (+ a (/ dx 2.0)) (lambda (x) (+ x dx)) b)
     dx))
```

```scheme
> (integral cube 0 1 0.01)
0.24998750000000042
> (integral cube 0 1 0.001)
0.249999875000001
```

---

## Пример 5: `accumulate` и асоциативност

```scheme
(define (accumulate op nv term a next b)
  (if (> a b)
      nv
      (op (term a) (accumulate op nv term (next a) next b))))

(define (accumulate-i op nv term a next b)
  (if (> a b)
      nv
      (accumulate-i op (op nv (term a)) term (next a) next b)))
```

```scheme
> (accumulate * 1 id 1 inc 5)
120
> (accumulate - 0 id 1 inc 4)      ; 1 - (2 - (3 - (4 - 0)))
-2
> (accumulate-i - 0 id 1 inc 4)    ; (((0 - 1) - 2) - 3) - 4
-10
```

> ⚠️ `accumulate` групира **отдясно**, `accumulate-i` - **отляво**. Ще срещнем същата разлика при `foldr` и `foldl` (Седмица 4).

---

## Пример 6: Предикат чрез `accumulate`

```scheme
(define (count-divisors n)
  (accumulate + 0
              (lambda (i) (if (= (remainder n i) 0) 1 0))
              1 inc n))

(define (prime? n)
  (and (> n 1) (= (count-divisors n) 2)))
```

```scheme
> (count-divisors 12)
6
> (prime? 17)
#t
> (prime? 1)
#f
```

---

## Пример 7: `let` и `let*`

```scheme
> (let ((x 3) (y 4)) (+ x y))
7
> (let* ((x 3) (y (* x 2))) (+ x y))
9
> (let ((x 2))
    (let ((x 3)
          (y x))       ; y вижда външното x = 2
      (* x y)))
6
```

---

## Пример 8: Функции, връщащи функции

```scheme
(define (compose f g) (lambda (x) (f (g x))))

(define (repeated f n)
  (if (= n 0)
      id
      (compose f (repeated f (- n 1)))))

(define dx 0.00001)
(define (deriv f)
  (lambda (x) (/ (- (f (+ x dx)) (f x)) dx)))
```

```scheme
> ((compose square inc) 6)        ; (6 + 1)²
49
> ((compose inc square) 6)        ; 6² + 1
37
> ((repeated inc 10) 0)
10
> ((deriv cube) 5)                ; 3 · 5² = 75
75.00014999664018
```

### Неподвижна точка

$x$ е неподвижна точка на $f$, ако $f(x) = x$. За някои функции я намираме, като прилагаме $f$ многократно, докато стойността спре да се променя:

```scheme
(define (fixed-point f guess)
  (let ((next (f guess)))
    (if (< (abs (- next guess)) 0.00001)
        next
        (fixed-point f next))))
```

```scheme
> (fixed-point cos 1.0)
0.7390822985224024
> (fixed-point (lambda (y) (+ 1 (/ 1 y))) 1.0)   ; златното сечение φ
1.6180327868852458
```
