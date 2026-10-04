# Седмица 4 - Примери

## Пример 1: Двойки и списъци в REPL

```scheme
> (cons 1 2)
'(1 . 2)
> (cons 1 (cons 2 (cons 3 '())))
'(1 2 3)
> (cons 1 (cons 2 3))
'(1 2 . 3)                     ; не е списък - завършва с 3, а не с '()
> (cons (list 1 2) (list 3 4))
'((1 2) 3 4)                   ; първият елемент е списък
> (list (list 1 2) 3 4)
'((1 2) 3 4)                   ; същото!
> (cadr '(1 2 3))
2
> (cddr '(1 2 3))
'(3)
> (list? '(1 . 2))
#f
> (pair? '())
#f
```

> 💡 Внимавайте с `(cons '(1 2) '(3 4))` - това **не** е слепване на списъци, а нов списък с първи елемент `'(1 2)`. За слепване използвайте `append`.

---

## Пример 2: Реимплементация на основни функции

```scheme
#lang racket

(define (my-length lst)
  (if (null? lst) 0 (+ 1 (my-length (cdr lst)))))

(define (my-list-ref lst n)
  (if (= n 0)
      (car lst)
      (my-list-ref (cdr lst) (- n 1))))

(define (my-member x lst)
  (cond ((null? lst) #f)
        ((equal? x (car lst)) lst)
        (else (my-member x (cdr lst)))))

(define (my-append xs ys)
  (if (null? xs)
      ys
      (cons (car xs) (my-append (cdr xs) ys))))

; Обръщане - итеративно, с акумулатор
(define (my-reverse lst)
  (define (iter lst acc)
    (if (null? lst)
        acc
        (iter (cdr lst) (cons (car lst) acc))))
  (iter lst '()))
```

```scheme
> (my-list-ref '(a b c d) 2)
'c
> (my-member 3 '(1 2 3 4))
'(3 4)
> (my-append '(1 2) '(3 4))
'(1 2 3 4)
> (my-reverse '(1 2 3))
'(3 2 1)
```

> ⚠️ Наивното `(define (rev lst) (append (rev (cdr lst)) (list (car lst))))` е $\Theta(n^2)$, защото всяко `append` е линейно. Версията с акумулатор е $\Theta(n)$.

---

## Пример 3: `map`, `filter`, `foldr`

```scheme
> (map (lambda (x) (* x x)) '(1 2 3 4))
'(1 4 9 16)
> (map + '(1 2 3) '(10 20 30))
'(11 22 33)
> (filter even? '(1 2 3 4 5 6))
'(2 4 6)
> (foldr + 0 '(1 2 3 4))
10
> (foldr cons '() '(1 2 3))
'(1 2 3)
> (foldl cons '() '(1 2 3))
'(3 2 1)
```

Проследяване:

```scheme
(foldr cons '() '(1 2 3))  = (cons 1 (cons 2 (cons 3 '())))  = '(1 2 3)
(foldl cons '() '(1 2 3))  = (cons 3 (cons 2 (cons 1 '())))  = '(3 2 1)
```

---

## Пример 4: Функции чрез `foldr`

Много функции над списъци са частни случаи на `foldr`:

```scheme
(define (sum lst)      (foldr + 0 lst))
(define (product lst)  (foldr * 1 lst))
(define (len lst)      (foldr (lambda (x acc) (+ 1 acc)) 0 lst))
(define (map* f lst)   (foldr (lambda (x acc) (cons (f x) acc)) '() lst))
(define (filter* p? lst)
  (foldr (lambda (x acc) (if (p? x) (cons x acc) acc)) '() lst))
(define (append* xs ys) (foldr cons ys xs))
```

```scheme
> (len '(a b c))
3
> (map* add1 '(1 2 3))
'(2 3 4)
> (filter* odd? '(1 2 3 4 5))
'(1 3 5)
> (append* '(1 2) '(3 4))
'(1 2 3 4)
```

> 💡 `add1` и `sub1` са вградени в Racket: `(add1 x)` ≡ `(+ x 1)`.

---

## Пример 5: Обработка на данни

Оценки на студенти като списък от двойки `(име . оценка)`:

```scheme
(define grades '((ivan . 5) (maria . 6) (petar . 3) (elena . 4)))

; Имената на студентите с оценка ≥ 5
(map car (filter (lambda (p) (>= (cdr p) 5)) grades))
; → '(ivan maria)

; Средна оценка
(/ (apply + (map cdr grades)) (length grades))
; → 9/2

; Най-висока оценка
(apply max (map cdr grades))
; → 6
```

> 💡 Комбинацията `map` + `filter` + `foldr`/`apply` е **конвейер** (pipeline) - данните „текат“ през поредица от трансформации.

---

## Пример 6: Дълбоки списъци

```scheme
(define (count-atoms t)
  (cond ((null? t) 0)
        ((pair? t) (+ (count-atoms (car t)) (count-atoms (cdr t))))
        (else 1)))

(define (my-flatten t)
  (cond ((null? t) '())
        ((pair? t) (append (my-flatten (car t)) (my-flatten (cdr t))))
        (else (list t))))

(define (deep-map f t)
  (cond ((null? t) '())
        ((pair? t) (cons (deep-map f (car t)) (deep-map f (cdr t))))
        (else (f t))))

(define (deep-reverse t)
  (if (pair? t)
      (reverse (map deep-reverse t))
      t))
```

```scheme
> (count-atoms '(1 (2 3) ((4)) 5))
5
> (my-flatten '(1 (2 (3 4)) ((5))))
'(1 2 3 4 5)
> (deep-map (lambda (x) (* 10 x)) '(1 (2 (3)) 4))
'(10 (20 (30)) 40)
> (deep-reverse '(1 (2 3) (4 (5 6))))
'(((6 5) 4) (3 2) 1)
```

---

## Пример 7: Код като данни

```scheme
> (define expr '(+ 1 (* 2 3)))
> (car expr)
'+
> (cadr expr)
1
> (caddr expr)
'(* 2 3)
> (eval expr (make-base-namespace))
7
```

> 💡 Тъй като програмата е списък, можем да я анализираме и трансформираме с функциите за списъци. В Седмица 5 ще използваме това за **символно диференциране**.
