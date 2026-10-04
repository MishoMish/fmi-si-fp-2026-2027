# Седмица 7 - Примери

## Пример 1: Кога се оценява?

```scheme
#lang racket

(define s
  (stream-cons (begin (displayln "глава") 1)
               (begin (displayln "опашка") empty-stream)))
```

```scheme
> (stream-first s)
глава
1
> (stream-first s)
1                       ; главата е запомнена - не се отпечатва отново
> (stream-empty? (stream-rest s))
опашка
#t
> (stream-empty? (stream-rest s))
#t
```

> 💡 Дефиницията на `s` **не** отпечатва нищо - нито главата, нито опашката се оценяват при `stream-cons`.

---

## Пример 2: Ефективност

```scheme
(define (prime? n)
  (and (> n 1)
       (for/and ((d (in-range 2 (add1 (integer-sqrt n)))))
         (not (= 0 (remainder n d))))))
```

```scheme
; Със списък - проверяваме ВСИЧКИ числа в интервала
> (time (second (filter prime? (range 10000 1000000))))
cpu time: ~200 ...
10009

; С поток - проверяваме само докато намерим второто просто
> (time (stream-ref (stream-filter prime? (in-range 10000 1000000)) 1))
cpu time: 0 ...
10009
```

> 💡 `in-range` и `in-naturals` създават **последователности**, които също са потоци. `(in-naturals)` е безкрайният поток `0, 1, 2, ...`.

---

## Пример 3: Безкрайни потоци - явни дефиниции

```scheme
(define (take-list s n) (stream->list (stream-take s n)))

(define (integers-from n)
  (stream-cons n (integers-from (+ n 1))))

(define nats (integers-from 0))

(define (fib-gen a b)
  (stream-cons a (fib-gen b (+ a b))))

(define fibs (fib-gen 0 1))
```

```scheme
> (take-list nats 10)
'(0 1 2 3 4 5 6 7 8 9)
> (take-list fibs 12)
'(0 1 1 2 3 5 8 13 21 34 55 89)
> (take-list (stream-map sqr nats) 5)
'(0 1 4 9 16)
> (take-list (stream-filter even? nats) 5)
'(0 2 4 6 8)
```

---

## Пример 4: Неявни дефиниции

```scheme
(define (add-streams s1 s2)
  (stream-cons (+ (stream-first s1) (stream-first s2))
               (add-streams (stream-rest s1) (stream-rest s2))))

(define (scale-stream s k)
  (stream-map (lambda (x) (* x k)) s))

(define ones (stream-cons 1 ones))
(define nats (stream-cons 0 (add-streams ones nats)))
(define powers-of-2 (stream-cons 1 (scale-stream powers-of-2 2)))
(define fibs
  (stream-cons 0 (stream-cons 1 (add-streams fibs (stream-rest fibs)))))
```

```scheme
> (take-list nats 10)
'(0 1 2 3 4 5 6 7 8 9)
> (take-list powers-of-2 11)
'(1 2 4 8 16 32 64 128 256 512 1024)
> (stream-ref fibs 50)
12586269025               ; моментално - благодарение на запомнянето
```

---

## Пример 5: Решето на Ератостен

```scheme
(define (sieve s)
  (stream-cons
   (stream-first s)
   (sieve (stream-filter
           (lambda (x) (not (= 0 (remainder x (stream-first s)))))
           (stream-rest s)))))

(define primes (sieve (integers-from 2)))
```

```scheme
> (take-list primes 10)
'(2 3 5 7 11 13 17 19 23 29)
> (stream-ref primes 99)
541
```

Как изглежда потокът след няколко стъпки:

```
(integers-from 2)       2  3  4  5  6  7  8  9 10 11 12 13 ...
след филтър за 2:          3     5     7     9    11    13 ...
след филтър за 3:                5     7          11    13 ...
след филтър за 5:                      7          11    13 ...
```

---

## Пример 6: Частични суми и приближаване на π

$$\frac{\pi}{4} = 1 - \frac{1}{3} + \frac{1}{5} - \frac{1}{7} + \cdots$$

```scheme
(define (partial-sums s)
  (define result
    (stream-cons (stream-first s)
                 (add-streams result (stream-rest s))))
  result)

(define (pi-summands n)
  (stream-cons (/ 1.0 n)
               (stream-map - (pi-summands (+ n 2)))))

(define pi-stream
  (scale-stream (partial-sums (pi-summands 1)) 4))
```

```scheme
> (take-list (partial-sums (integers-from 1)) 6)
'(1 3 6 10 15 21)
> (take-list pi-stream 5)
'(4.0 2.666666666666667 3.466666666666667 2.8952380952380956 3.3396825396825403)
```

Редът се сходи **бавно**. Можем да го **ускорим** с трансформацията на Ойлер, която по три последователни члена $S_{n-1}, S_n, S_{n+1}$ дава по-добро приближение:

$$S_{n+1} - \frac{(S_{n+1} - S_n)^2}{S_{n-1} - 2S_n + S_{n+1}}$$

```scheme
(define (euler-transform s)
  (let ((s0 (stream-ref s 0))
        (s1 (stream-ref s 1))
        (s2 (stream-ref s 2)))
    (stream-cons (- s2 (/ (sqr (- s2 s1))
                          (+ s0 (* -2 s1) s2)))
                 (euler-transform (stream-rest s)))))
```

```scheme
> (take-list (euler-transform pi-stream) 5)
'(3.166666666666667 3.1333333333333337 3.1452380952380956 3.13968253968254 3.1427128427128435)
```

> 💡 Трансформацията е функция **поток → поток**. Можем да я прилагаме многократно - всяко прилагане ускорява сходимостта.

---

## Пример 7: Сливане на потоци - числа на Хаминг

Числата на Хаминг са тези, чиито единствени прости делители са 2, 3 и 5: `1, 2, 3, 4, 5, 6, 8, 9, 10, 12, ...`

Наблюдение: ако $h$ е число на Хаминг, то и $2h$, $3h$, $5h$ са такива.

```scheme
(define (merge s1 s2)
  (let ((a (stream-first s1))
        (b (stream-first s2)))
    (cond ((< a b) (stream-cons a (merge (stream-rest s1) s2)))
          ((> a b) (stream-cons b (merge s1 (stream-rest s2))))
          (else (stream-cons a (merge (stream-rest s1) (stream-rest s2)))))))

(define hamming
  (stream-cons 1 (merge (scale-stream hamming 2)
                        (merge (scale-stream hamming 3)
                               (scale-stream hamming 5)))))
```

```scheme
> (take-list hamming 15)
'(1 2 3 4 5 6 8 9 10 12 15 16 18 20 24)
```

---

## Пример 8: `for/stream`

Racket предлага и удобен синтаксис за създаване на потоци:

```scheme
> (take-list (for/stream ((i (in-naturals))) (* i 10)) 4)
'(0 10 20 30)
```
