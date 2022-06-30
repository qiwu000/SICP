# SICP Lecture 3A
# Henderson Escher Example

*Harold Abelson*

## 关于向量表示的更多内容

```lisp
;向量相加函数
(define (+vect v1 v2)
  (make-vector
    (+ (xcor v1) (xcor v2))
    (+ (ycor v1) (ycor v2))))


;缩放向量v，大小为s倍
(define (scale s v)
  (make-vector
    (* s (xcor v))
    (* s (ycor v))))
```

Actually no need for formal parameters, procedures can be aliased

其实不需要形参，程序可以别名

```lisp
(define make-vector cons)
(define xcor car)
(define ycor cdr)

(define make-segment cons)
(define seg-start car)
(define seg-end cdr)
```

Closures allow to build complexity.

闭包性质允许构造复杂性

## Lists

A list is **convention** to reprensent a sequence using a **chain of pairs**.

    ; [1, 2, 3, 4]
    (cons 1 (cons 2 (cons 3 (cons 4 nil))))

`nil` marks the end of the list

`nil`表示list的结尾



Syntactic sugar:

语法糖：

    (define 1-to-4 (list 1 2 3 4))
    
    (car (cdr 1-to-4)) -> 2
    (car (cdr (cdr 1-to-4))) -> 3
    (cdr (cdr 1-to-4)) -> (3, 4)
    (cdr (cdr (cdr (cdr 1-to-4)))) -> ()
    
    
    ; Scaling all items of a list
    ; 缩放一个list的所有元素
    (define (scale-list s l)
      (if (null? l)
        nil
        (cons (* s (car l)) (scale-list s (cdr l)))))

### Map

    ; Generalization
    ; 概括
    ; 对l中的每个元素做p操作
    (define (map p l)
      (if (null? l)
        nil
        (cons (p (car l)) (map p (cdr l)))))
    
    ; Redefined using map
    ; 用map重新定义sacle
    (define (scale-list s l)
      (map (lambda (x) (* s x)) l))
    
    ; Other examples
    ; 其他例子
    (map square 1-to-4)
    (map (lambda (x) (+ x 10)) 1-to-4)

### For Each

Similar to map but does not return a new list

和map差不多，但是返回一个新列表

    (define (for-each proc list)
      (cond ((null? list) *done*)
            (else (proc (car list))
                  (for-each proc (cdr list)))))


## Henderson Escher Example（亨德森-埃舍尔的例子）

Primitive in the language: picture (image in a rectangle)

语言中的基本概念：图片（矩形中的图像）。Means of combination:

* rotate
* 旋转
* flip
* 翻转
* beside and above (fits 2 pictures in a rectangle)、
* 

A rectangle is specified by 3 vectors:

一个矩形由三个向量指定：

* an origin

* 原点

* the horizontal part of the rectangle

* 矩形的水平部分

* the vertical part of the rectangle

* 矩形的竖直部分

    (define (make-rect o h v) (...))
    
    创建矩形
    
    (define (horiz rect) (...))
    
    获取矩形的水平部分
    
    (define (vert rect) (...))
    
    获取矩形的竖直部分
    
    (define (origin rect) (...))
    
    获取矩形的原点

A rectangle defines transformation from a unit square.

矩形定义了从一个单位正方形的转变。

```lisp
(x,y) -> (x',y') = orig + x*horz.x + y*vert.y

; Returns a procedure which transform a point in the rectangle's referential
; 返回一个用一个点把正方形变换为矩形的过程
(define (coord-map rect)
  (lambda (point)
    (+vect
      (+vect
        (scale (xcor point) (horiz rect))
        (scale (ycor point) (vert rect)))
      (origin rect))))
```

Constructing primitive pictures from lists of segments:

从线段的列表中构建原始图片：

    (define (make-picture seglist)
      (lambda (rect)
        (for-each
          (lambda (seg)
            (drawline
              ((coord-map rect) (seg-start seg))
              ((coord-map rect) (seg-end seg))))
          seglist)))

How to use this:

    (define r (make-rect ...))
    (define g (make-picture ...))
    
    ; Draw picture g in rectangle r
    ; 在矩形r中画出图像g
    (g r)

With these abstractions in place, the combination possibilities fall out:

有了这些抽象，就可以组合了：

    (define (beside p1 p2 a)
      (lambda (rect)
        (p1 (make-rect
              (origin rect)
              (scale a (horiz rect))
              (vert rect)))
        (p2 (make-rect
          (+vect (origin rect) (scale a (horiz rect)))
          (scale (- 1 a) (horiz rect))
          (vert rect))))))
    
    (define p1-next-to-p1 (beside p1 p2 golden-ratio))
    (p1-next-to-p2 rect)



    (define (rotate90 pict)
      (lambda (rect)
        (pict (make-rect
          (+vect (origin rect) (horiz rect))
          (vert rect)
          (scale -1 (horiz rect))))))

We use the representation of pictures as procedure to benefit from the closure property.

The language is **embedded in Lisp** and we can thus benefit from all its constructs.

Example: build a procedure which puts 4 pictures in rectangle:

我们使用图片的表示方法作为程序，以受益于封闭属性。

这种语言是**嵌入Lisp的，因此我们可以从它的所有结构中获益。

例如：建立一个过程，将4张图片放在矩形中。

    (define (make-4-picture p1 p2 p3 p4)
      (lambda (rect)
        (beside
          (above p1 p3 0.5)
          (above p2 p4 0.5)
        0.5))))

Example of recursive operation

    (define (right-push p n a)
      (if (= n 0)
        p
        (beside p (right-push p (-1 n) a))))
    
    ; Generalization using a provided combinator
    (define (push comb)
      (lambda (pict n a)
        ((repeated
          (lambda (i) (comb pict i a))
          n)
          pict)))
    
    (define right-push (push beside))


There is no difference between data and procedure, the system is both and neither.

A sequence of layers of language:

  * language of schemes of combinations: (push, ...)
  * language of geometric positions
  * language of primitive (picts)

Design as **levels of languages** vs hierarchy of tasks
