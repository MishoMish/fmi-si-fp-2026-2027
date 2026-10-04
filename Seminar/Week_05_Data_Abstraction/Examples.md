# Седмица 5 - Примери

## Пример 1: Рационални числа - пълна реализация

```scheme
#lang racket

;;; Ниво 1: представяне
(define (make-rat n d)
  (let* ((g (gcd n d))
         (sign (if (< d 0) -1 1)))      ; знаменателят винаги е положителен
    (cons (* sign (/ n g)) (* sign (/ d g)))))

(define (numer r) (car r))
(define (denom r) (cdr r))

;;; Ниво 2: операции
(define (add-rat x y)
  (make-rat (+ (* (numer x) (denom y)) (* (numer y) (denom x)))
            (* (denom x) (denom y))))

(define (mul-rat x y)
  (make-rat (* (numer x) (numer y))
            (* (denom x) (denom y))))

(define (equal-rat? x y)
  (= (* (numer x) (denom y)) (* (numer y) (denom x))))

(define (rat->string r)
  (format "~a/~a" (numer r) (denom r)))
```

```scheme
> (define one-half (make-rat 1 2))
> (define one-third (make-rat 1 3))
> (rat->string (add-rat one-half one-third))
"5/6"
> (rat->string (mul-rat one-half one-third))
"1/6"
> (rat->string (add-rat one-third one-third))
"2/3"                                    ; съкратено!
> (rat->string (make-rat 2 -4))
"-1/2"                                   ; нормализиран знак
> (equal-rat? (make-rat 1 2) (make-rat 2 4))
#t
```

> 💡 `format` замества всяко `~a` със следващия аргумент - удобно за отпечатване.

---

## Пример 2: Точки и отсечки - нива на абстракция

```scheme
;;; Точки
(define (make-point x y) (cons x y))
(define (x-point p) (car p))
(define (y-point p) (cdr p))

;;; Отсечки - изградени върху точки
(define (make-segment a b) (cons a b))
(define (start-segment s) (car s))
(define (end-segment s) (cdr s))

(define (midpoint s)
  (let ((a (start-segment s))
        (b (end-segment s)))
    (make-point (/ (+ (x-point a) (x-point b)) 2)
                (/ (+ (y-point a) (y-point b)) 2))))

(define (segment-length s)
  (let ((a (start-segment s))
        (b (end-segment s)))
    (sqrt (+ (sqr (- (x-point b) (x-point a)))
             (sqr (- (y-point b) (y-point a)))))))
```

```scheme
> (define s (make-segment (make-point 0 0) (make-point 6 8)))
> (midpoint s)
'(3 . 4)
> (segment-length s)
10
```

> 💡 `midpoint` не знае, че точките са двойки - тя използва `x-point` и `y-point`. `sqr` е вграденото повдигане на квадрат в Racket.

---

## Пример 3: Двойки без двойки

```scheme
; Чрез съобщения
(define (my-cons x y)
  (lambda (m)
    (cond ((= m 0) x)
          ((= m 1) y)
          (else (error "my-cons: invalid selector" m)))))
(define (my-car p) (p 0))
(define (my-cdr p) (p 1))

; Още по-кратко: двойката приема функция и ѝ подава x и y
(define (cons2 x y) (lambda (m) (m x y)))
(define (car2 z) (z (lambda (p q) p)))
(define (cdr2 z) (z (lambda (p q) q)))
```

```scheme
> (my-car (my-cons 1 2))
1
> (car2 (cons2 'a 'b))
'a
> (cdr2 (cons2 'a 'b))
'b
```

Проследяване на `(car2 (cons2 'a 'b))`:

```scheme
(car2 (cons2 'a 'b))
(car2 (lambda (m) (m 'a 'b)))
((lambda (m) (m 'a 'b)) (lambda (p q) p))
((lambda (p q) p) 'a 'b)
'a
```

---

## Пример 4: Двоично дърво

```scheme
(define empty-tree '())
(define (empty-tree? t) (null? t))
(define (make-tree root left right) (list root left right))
(define (root t) (car t))
(define (left t) (cadr t))
(define (right t) (caddr t))
(define (leaf x) (make-tree x empty-tree empty-tree))
(define (leaf? t)
  (and (not (empty-tree? t))
       (empty-tree? (left t))
       (empty-tree? (right t))))

(define t
  (make-tree 5
             (make-tree 3 (leaf 1) (leaf 4))
             (make-tree 8 empty-tree (leaf 9))))

(define (tree-sum t)
  (if (empty-tree? t)
      0
      (+ (root t) (tree-sum (left t)) (tree-sum (right t)))))

(define (count-leaves t)
  (cond ((empty-tree? t) 0)
        ((leaf? t) 1)
        (else (+ (count-leaves (left t)) (count-leaves (right t))))))

(define (tree-map f t)
  (if (empty-tree? t)
      empty-tree
      (make-tree (f (root t))
                 (tree-map f (left t))
                 (tree-map f (right t)))))
```

```scheme
> (tree-sum t)
30
> (count-leaves t)
3
> (tree-map (lambda (x) (* 10 x)) t)
'(50 (30 (10 () ()) (40 () ())) (80 () (90 () ())))
```

---

## Пример 5: Символно диференциране

```scheme
(define (variable? x) (symbol? x))
(define (same-variable? a b) (and (variable? a) (variable? b) (eq? a b)))
(define (=number? e n) (and (number? e) (= e n)))

; Конструктори с опростяване
(define (make-sum a b)
  (cond ((=number? a 0) b)
        ((=number? b 0) a)
        ((and (number? a) (number? b)) (+ a b))
        (else (list '+ a b))))

(define (make-product a b)
  (cond ((or (=number? a 0) (=number? b 0)) 0)
        ((=number? a 1) b)
        ((=number? b 1) a)
        ((and (number? a) (number? b)) (* a b))
        (else (list '* a b))))

(define (sum? e) (and (pair? e) (eq? (car e) '+)))
(define (addend e) (cadr e))
(define (augend e) (caddr e))
(define (product? e) (and (pair? e) (eq? (car e) '*)))
(define (multiplier e) (cadr e))
(define (multiplicand e) (caddr e))

(define (deriv expr var)
  (cond ((number? expr) 0)
        ((variable? expr) (if (same-variable? expr var) 1 0))
        ((sum? expr)
         (make-sum (deriv (addend expr) var)
                   (deriv (augend expr) var)))
        ((product? expr)
         (make-sum (make-product (multiplier expr) (deriv (multiplicand expr) var))
                   (make-product (deriv (multiplier expr) var) (multiplicand expr))))
        (else (error "deriv: unknown expression" expr))))
```

```scheme
> (deriv '(+ x 3) 'x)
1
> (deriv '(* x y) 'x)
'y
> (deriv '(+ (* 3 x) (* x x)) 'x)
'(+ 3 (+ x x))
> (deriv '(* (* x y) (+ x 3)) 'x)
'(+ (* x y) (* y (+ x 3)))
```

> 💡 Без опростяването в `make-sum` и `make-product`, `(deriv '(* x y) 'x)` би върнало `'(+ (* x 0) (* 1 y))`. Опростяването е изцяло в **конструкторите** - самият `deriv` не се промени.

---

## Пример 6: `struct`

```scheme
(struct point (x y) #:transparent)

(define (distance p q)
  (sqrt (+ (sqr (- (point-x p) (point-x q)))
           (sqr (- (point-y p) (point-y q))))))
```

```scheme
> (distance (point 0 0) (point 3 4))
5
> (point 1 2)
(point 1 2)
> (equal? (point 1 2) (point 1 2))
#t
```
