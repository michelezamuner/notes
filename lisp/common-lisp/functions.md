### Syntax

**Data mode**
```lisp
; use ' to enter data mode, like '(1 2 3)
```

**Quasiquoting**
```lisp
; use "`" to enter data mode, "," to embed code mode, like `(Some data ,variable other data ,(func call))
```

**Lambda**
```lisp
; use #' to tag a lambda, like #'princ
; this is equivalent to using the function function, like (function princ)
```

**Characters**
```lisp
; use #\ to tag a character, like #\a
```

### Values and Variables

**Global variables definition**
```lisp
(defparameter *name* value)
; return: *name*
; can be redefined with a new value
```
```lisp
(defvar *name* value)
; return: *name*
; ignores redefinitions of the same variable
```

**Local variables context definition**
```lisp
(let ((name1 value1)
      (name2 value2)
      ...
      (namen valuen))
  (<expression>))
; return <expression value>
; defined variables are only visible within <expression>
```

**Variable setting**
```lisp
(setf *name1* value1
      *name2* value2
      ...
      *namen* valuen)
; return: valuen
; can set any kind of variable, however created
```

**Identical**
```lisp
(eq value1 value2)
; return t if value1 is the same instance as value2, nil otherwise
; this should be used with symbols
```

**Equal**
```lisp
(equal value1 value2)
; return t if value1 looks the same as value2, nil otherwise
; can be used with all datatypes
```

### Functions

**Global functions definition**
```lisp
(defun name (param1
             param2
             ...
             paramn)
  (<expression>))
; return: name
```

**Local functions context definition**
```lisp
(flet ((name1 (param1 param2 ... paramn)
         (<expression>))
       (name2 (param1 param2 ... paramn)
         (<expression>))
       ...
       (namen (param1 param2 ... paramn)
         (<expression>)))
  (<expression>))
; return <expression value>
; defined functions are only visible within <expression>
```
```lisp
(labels ((name1 (param1 param2 ... paramn)
         (<expression>))
       (name2 (param1 param2 ... paramn)
         (<expression>))
       ...
       (namen (param1 param2 ... paramn)
         (<expression>)))
  (<expression>))
; return <expression value>
; defined functions are visible in their definitions, and <expression> only
```

**Apply, or arguments spread**
```lisp
(apply #'func '(value1 value2 ... valuen))
; return the value of func when called with the elements of the given list as arguments
```

### Expressions

**Sequential execution**
```lisp
(progn expr1 expr2 ... exprn)
; return: value of exprn
; execute all given expressions sequentially
```

### Logic and Arithmetic

**And**
```lisp
(and cond1 cond2 ... condn)
; return: t if all conditions are true, nil otherwise
; short-circuit to nil at the first false condition found
```

**Or**
```lisp
(or cond1 cond2 ... condn)
; return: t if at least one condition is true, nil otherwise
; short-circuit to t at the first true condition found
```

**Equal**
```lisp
(= value1 value2)
; return: t if value1 equals value2, nil otherwise
; automatically converts integers to floats, so for example 1 equals 1.0
```

**Add 1**
```lisp
(1+ value)
; return: <value + 1>
```

**Subtract 1**
```lisp
(1- value)
; return: <value - 1>
```

**Addition**
```lisp
(+ value1 value2 ... valuen)
; return: <value1 + value2 + ... + valuen>
```

**Multiplication**
```lisp
(* value1 value2 ... valuen)
; return: <value1 * value2 * ... * valuen>
```

**Bit shift**
```lisp
(ash value1 value2)
; return: <shift value2 amount of bits in value1>
; shift left if value2 is positive, shift right if value2 is negative
```

**Is odd**
```lisp
(oddp value)
; return: t if value is odd, nil otherwise
```

### Conditionals

**If**
```lisp
(if condition value1 value2)
; return: value1 if condition is evaluated true, value2 otherwise
; any non-empty list represents true, the empty list, represents false
; conventionally t represents true, nil represents false
```

**When**
```lisp
(when condition expr1 expr2 ... exprn)
; return: value of exprn, if condition is evaluated true, nil otherwise
; execute all given expressions sequentially, if condition is evaluated true
```

**Until**
```lisp
(until condition expr1 expr2 ... exprn)
; return: value of exprn, if condition is evaluated false, nil otherwise
; execute all given expressions sequentially, if condition is evaluated false
```

**Generic conditional construct**
```lisp
(cond (cond1 expr1 expr2 ... exprn)
      (cond2 expr1 expr2 ... exprn)
      ...
      (condn expr1 expr2 ... exprn))
; return: value of exprn of the first match found
; execute only expressions of the first match found
```

**Case**
```lisp
(case variable
      ((match11 match12 ... match1n) expr1 expr2 ... exprn)
      ((match21 match22 ... match2n) expr1 expr2 ... exprn)
      ...
      ((matchm1 matchm2 ... matchmn) expr1 expr2 ... exprn))
; return: value of exprn of the first match found
; execute only expressions of the first match found
; can use otherwise value to match all remaining cases
```

### Lists manipulation

**Create lists**
```lisp
(list value1 value2 ... valuen)
; return '(value1 value2 ... valuen)
```

**Push to front**
```lisp
(cons value '(value1 value2 ... valuen))
; return '(value value1 value2 ... valuen)
```
```lisp
(push value variable)
; variable is a variable containing a list
; variable is modified so that the given value is pushed to its front
; the new value of the variable is returned
```

**Join**
```lisp
(append '(value1 ... valuen) ... '(valuem ... valuez))
; return the list obtained joining all given lists in the given order
```

**Retrieve elements**
```lisp
(car '(value1 value2 ... valuen))
; return value1
```
```lisp
(cdr '(value1 value2 ... valuen))
; return '(value2 ... valuen)
```
```lisp
(cadr '(value1 value2 ... valuen))
; return value2
```
...

**Retrieve remainder from searched value**
```lisp
(member value '(value1 value2 ... value ... valuen)
; return '(value ... valuen) if value exists in the list, nil otherwise
```

**Find**
```lisp
(find value '(value1 value2 ... value))
; return value if it's found among the items of the given list, nil otherwise
```
```lisp
(find value '((value1 value2 ... value)
              (value1 value2 ... value)
              ...
              (value1 value2 ... value))
              :key #'func)
; to each inner list of the given list, the key function is applied, and the returning value is tested against the
;   given value. As soon as one inner list is found where the match succeeds, that inner list is returned. If no such
;   inner list is found, nil is returned.
```

**Find with lambda**
```lisp
(find-if #'func '(value1 value2 ... valuen))
; return the first value of the list that applied to func returns t, nil if func returns nil for every value
```

**Find list given the first element**
```lisp
(assoc value ((value1 value2 ... valuen)
              (value1 value2 ... valuen)
              ...
              (value1 value2 ... valuen))
; return, among the given lists, the one having value as first element, or nil if no such list can be found
```

**Map**
```lisp
(mapcar '#func '(value1 value2 ... valuen))
; return the list obtained applying func to each value of the given list
```

**Filter**
```lisp
(remove-if-not '#func '(value1 value2 ... valuen))
; return the list obtained filtering the given list by keeping only the values that applied to the given function return true
```

### I/O

**Print**
```lisp
(print value)
; return: value
; print a newline first, followed by value
```
```lisp
(princ value)
; return: value
; print only value, without any additional newlines
```

### System

**Quit**
```lisp
(quit)
; return: nil
```
