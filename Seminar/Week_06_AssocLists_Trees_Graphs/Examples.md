# Седмица 6 - Примери

## Пример 1: Телефонен указател

```scheme
#lang racket

(define phone-book
  '((ivan . "0888") (maria . "0877") (petar . "0899")))

(define (lookup key alist)
  (let ((p (assoc key alist)))
    (if p (cdr p) #f)))

(define (alist-set key value alist)
  (cond ((null? alist) (list (cons key value)))
        ((equal? (caar alist) key) (cons (cons key value) (cdr alist)))
        (else (cons (car alist) (alist-set key value (cdr alist))))))

(define (alist-delete key alist)
  (filter (lambda (p) (not (equal? (car p) key))) alist))
```

```scheme
> (lookup 'petar phone-book)
"0899"
> (alist-set 'maria "0811" phone-book)
'((ivan . "0888") (maria . "0811") (petar . "0899"))
> (alist-set 'gosho "0822" phone-book)
'((ivan . "0888") (maria . "0877") (petar . "0899") (gosho . "0822"))
> (alist-delete 'ivan phone-book)
'((maria . "0877") (petar . "0899"))
> phone-book                     ; оригиналът е непроменен!
'((ivan . "0888") (maria . "0877") (petar . "0899"))
```

---

## Пример 2: Хистограма

Броим срещанията на всеки елемент - асоциативен списък `елемент → брой`:

```scheme
(define (histogram lst)
  (foldl (lambda (x acc)
           (alist-set x (+ 1 (or (lookup x acc) 0)) acc))
         '()
         lst))
```

```scheme
> (histogram '(a b a c b a))
'((a . 3) (b . 2) (c . 1))
```

> 💡 `(or (lookup x acc) 0)` - ако ключът липсва, `lookup` връща `#f` и `or` дава `0`. Удобен идиом за стойност по подразбиране.

---

## Пример 3: Двоично дърво за търсене

```scheme
(define empty-tree '())
(define (empty-tree? t) (null? t))
(define (make-tree root left right) (list root left right))
(define (root t) (car t))
(define (left t) (cadr t))
(define (right t) (caddr t))

(define (bst-insert x t)
  (cond ((empty-tree? t) (make-tree x empty-tree empty-tree))
        ((< x (root t)) (make-tree (root t) (bst-insert x (left t)) (right t)))
        ((> x (root t)) (make-tree (root t) (left t) (bst-insert x (right t))))
        (else t)))

(define (bst-member? x t)
  (cond ((empty-tree? t) #f)
        ((= x (root t)) #t)
        ((< x (root t)) (bst-member? x (left t)))
        (else (bst-member? x (right t)))))

(define (list->bst lst) (foldl bst-insert empty-tree lst))

(define (inorder t)
  (if (empty-tree? t)
      '()
      (append (inorder (left t)) (list (root t)) (inorder (right t)))))

(define (tree-sort lst) (inorder (list->bst lst)))
```

```scheme
> (list->bst '(5 3 8 1 4 9))
'(5 (3 (1 () ()) (4 () ())) (8 () (9 () ())))
> (bst-member? 4 (list->bst '(5 3 8 1 4 9)))
#t
> (bst-member? 7 (list->bst '(5 3 8 1 4 9)))
#f
> (tree-sort '(5 3 8 1 4 9 2))
'(1 2 3 4 5 8 9)
```

> ⚠️ `foldl` извиква `(bst-insert x acc)` - редът на аргументите на `bst-insert` съвпада с реда, който `foldl` в Racket очаква. Удобно съвпадение!

---

## Пример 4: Дърво с произволна разклоненост

```scheme
(define t '(1 (2 (5) (6)) (3) (4 (7 (8)))))

(define (tree-root t) (car t))
(define (tree-children t) (cdr t))

(define (count-nodes t)
  (+ 1 (apply + (map count-nodes (tree-children t)))))

(define (height t)
  (+ 1 (apply max 0 (map height (tree-children t)))))

(define (leaves t)
  (if (null? (tree-children t))
      (list (tree-root t))
      (append-map leaves (tree-children t))))

(define (tree->list t)          ; preorder
  (cons (tree-root t) (append-map tree->list (tree-children t))))
```

```scheme
> (count-nodes t)
8
> (height t)
4
> (leaves t)
'(5 6 3 8)
> (tree->list t)
'(1 2 5 6 3 4 7 8)
```

---

## Пример 5: Граф - основни операции

```scheme
(define g '((a b c) (b c d) (c e) (d e) (e) (f a)))

(define (vertices g) (map car g))

(define (successors v g)
  (let ((p (assoc v g)))
    (if p (cdr p) '())))

(define (edge? u v g) (and (member v (successors u g)) #t))

(define (predecessors v g)
  (filter (lambda (u) (edge? u v g)) (vertices g)))

(define (edges g)
  (append-map (lambda (u) (map (lambda (v) (cons u v)) (successors u g)))
              (vertices g)))
```

```scheme
> (vertices g)
'(a b c d e f)
> (successors 'b g)
'(c d)
> (edge? 'a 'c g)
#t
> (edge? 'c 'a g)
#f
> (predecessors 'e g)
'(c d)
> (edges g)
'((a . b) (a . c) (b . c) (b . d) (c . e) (d . e) (f . a))
```

---

## Пример 6: DFS и BFS

```scheme
(define (dfs start g)
  (define (visit v visited)
    (if (member v visited)
        visited
        (foldl visit (cons v visited) (successors v g))))
  (reverse (visit start '())))

(define (bfs start g)
  (define (loop queue visited)
    (if (null? queue)
        (reverse visited)
        (let* ((v (car queue))
               (new (filter (lambda (u) (not (member u visited)))
                            (successors v g))))
          (loop (append (cdr queue) new)
                (append (reverse new) visited)))))
  (loop (list start) (list start)))
```

```scheme
> (dfs 'a g)
'(a b c e d)          ; от b отиваме в c, после в e, чак тогава в d
> (bfs 'a g)
'(a b c d e)          ; първо ниво 1 (b, c), после ниво 2 (d, e)
> (dfs 'f g)
'(f a b c e d)
```

Проследяване на `dfs` от `a`:

| Посещаваме | visited (обърнат) | Следващи                |
| ---------- | ----------------- | ----------------------- |
| `a`        | `(a)`             | `b`, `c`                |
| `b`        | `(b a)`           | `c`, `d`                |
| `c`        | `(c b a)`         | `e`                     |
| `e`        | `(e c b a)`       | -                       |
| `d`        | `(d e c b a)`     | `e` вече е посетен      |
| `c` (от a) | -                 | вече е посетен          |

---

## Пример 7: Всички пътища в ацикличен граф

```scheme
(define (all-paths u v g)
  (if (eq? u v)
      (list (list v))
      (map (lambda (path) (cons u path))
           (append-map (lambda (w) (all-paths w v g))
                       (successors u g)))))
```

```scheme
> (all-paths 'a 'e g)
'((a b c e) (a b d e) (a c e))
```

> ⚠️ Тази функция **зацикля** при граф с цикли! Защо? Как да я поправите? (вижте домашното)
