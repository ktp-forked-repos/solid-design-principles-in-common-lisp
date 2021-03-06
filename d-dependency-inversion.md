# D: Dependency Inversion

* ##### High level modules should not depend upon low level modules. Both should depend upon abstractions.
* ##### Abstractions should not depend upon details. Details should depend upon asbtractions.

Dependency Inversion Principle encourages us to create higher level modules with its complex logic in such a way to be reusable and unaffected by any change from the lower level modules in our application. To achieve this kind of behavior in our apps, we introduce abstraction which decouples higher from lower level modules.

### Wait! What are these *low-level* and *high-level* modules again?

Well, low-level modules are more specific to the individual components focusing os smaller details of the application. This *low-level* module should be used in *high-level* modules in the application.

While *high-level* modules are more abstract and general in nature. They handle *low-level* modules and decide the logic for what goes where.

Think about a computer CPU (high-level) handling bunch of hardware inputs (low-level) including keyboard and mouse inputs.

See DI violation below.

```lisp
(defclass printer ()
  ((data-type
    :initarg :data-type
    :reader get-data-type)))

(defmethod print-epub ((printer printer))
  (let ((e (make-instance 'epub-formatter)))
    (process e (get-data-type printer))))

(defmethod print-mobi ((printer printer))
  (let ((m (make-instance 'mobi-formatter)))
    (process m (get-data-type printer))))

(defclass epub-formatter ()
  nil)

(defmethod process ((epub-formatter epub-formatter) data-type)
  (format t "~a~%data-type: ~a~%"
          "epub formatter's process logic goes here"
          data-type))

(defclass mobi-formatter ()
  nil)

(defmethod process ((mobi-formatter mobi-formatter) data-type)
  (format t "~a~%data-type: ~a~%"
          "mobi formatter's process logic goes here"
          data-type))

(defparameter epub-book (make-instance 'printer :data-type "epubs"))
(defparameter mobi-book (make-instance 'printer :data-type "mobis"))

(print-epub epub-book)
;; epub formatter's process logic goes here
;; data-type: epubs

(print-mobi mobi-book)
;; mobi formatter's process logic goes here
;; data-type: mobis
```

`printer` (high-level) class had to depend on `print-epub` and `print-mobi` which are both low-level modules. This breaks the " *High level modules should not depend upon low level modules. Both should depend upon abstractions.*" keypoint.

Let's fix it below.

```lisp
(defclass printer ()
  ((data-type
    :initarg :data-type
    :reader get-data-type)))

(defmethod prints ((printer printer) formatter)
  (let ((f (make-instance formatter)))
    (process f (get-data-type printer))))

(defclass epub-formatter ()
  nil)

(defmethod process ((epub-formatter epub-formatter) data-type)
  (format t "~a~%data-type: ~a~%"
          "epub formatter's process logic goes here"
          data-type))

(defclass mobi-formatter ()
  nil)

(defmethod process ((mobi-formatter mobi-formatter) data-type)
  (format t "~a~%data-type: ~a~%"
          "mobi formatter's process logic goes here"
          data-type))


(defparameter epub-book (make-instance 'printer :data-type "epubs"))
(defparameter mobi-book (make-instance 'printer :data-type "mobis"))

(prints epub-book 'epub-formatter)
;; epub formatter's process logic goes here
;; data-type: epubs

(prints mobi-book 'mobi-formatter)
;; mobi formatter's process logic goes here
;; data-type: mobis
```

We have moved both `print-epub` and `print-mobi` into separate classes. Now they can do their own things, and whenever we need to print an ebook, we are going to pass either `epub-formatter` or `mobi-formatter` to `printer` constructor, then call `prints` to execute the method based on the class we passed.

Now, our classes are not tightly coupled with the lower-tier objects and we can easily reuse the logic from the high-tier modules.

```lisp
(defclass printer ()
  ((amount
    :initarg :amount
    :reader amount
    :type integer)))

(defclass epub (printer) nil)
(defclass mobi (printer) nil)

(defgeneric print-book (book-format)
  (:documentation "print a book given the book format class"))

(defmethod print-book ((epub epub))
  (format t
          "~a: ~a~%"
          "epub formatter logic goes here"
          (amount epub)))

(defmethod print-book ((mobi mobi))
  (format t
          "~a: ~a~%"
          "mobi formatter logic goes here"
          (amount mobi)))

(defparameter *gatsby-in-epub*
  (make-instance 'epub
                 :amount 100))

(defparameter *dracula-in-mobi*
  (make-instance 'mobi
                 :amount 900))

(print-book *gatsby-in-epub*)
(print-book *dracula-in-mobi*)
```


