+++
title = "Stack Safety and CPS In Common Lisp"
date = "2024-03-18"
tags = [
	"lisp",
]
authors = ["david"]
+++

**Editor's note**: this is a translation of a Common Lisp file I sent to my
sister one day a couple months ago when she was asking why her Emacs Lisp function 
was hitting a recursion limit. I was fresh off of CPS lab and proceeded to nerd 
out. Also, any hate of `setf` in here is a comedic exaggeration.

This is a very simple tour of using Common Lisp to write some simple factorial
and is-even functions. If you're new to functional programming or Lisp you might 
find it interesting, but this is mostly to test out this website.

---

You're likely already aware that when you call a function, a "stack frame"
containing the memory needed for its local variables and such is pushed to
a processes' stack and then popped when the function returns (this way it
preserves the state of the function that calls another function).

The consequence of this is that the deeper a recursion goes, the more memory
is (traditionally) needed! This sorta sucks since there's plenty of regular
recursive algorithms that we would like to work.

One observation we can make is that we don't need to preserve the state of
a parent function if the recursive call is the _last_ thing we do (there's
nothing to come back to!). This is called "tail recursion" and we can perform
"tail call optimization" here by replacing the current stack frame with the
new one when we recurse instead of pushing a new stack frame (since we don't
need to come back to the old function!)

Many languages sadly do not do this optimization (e.g. only some lisps like
SBCL do it). Let's see an example with a factorial function.

```lisp
(defun bad-factorial (x)
  (if (zerop x) 1
      (* x (bad-factorial (- x 1)))))
```

This is equivalent to `n * fact (n-1)` which is problematic because we need to
do something with the result of the recursion.

Calling this will murder SBCL!
```text
CL-USER> (bad-factorial 500004)
Error: Control stack exhausted (no more space for function call frames).
```
Anyways this is slow because it's not a tail call: we do multiplication
after the recursion and so we need to preserve the previous stack frames.

One way of fixing this would be to add an accumulator argument like so:
```lisp
(defun tail-factorial (x &optional (acc 1))
  (if (zerop x) acc
      (tail-factorial (- x 1) (* acc x))))
```

This won't murder SBCL because it can reuse stack frames!
```lisp
(tail-factorial 50004)
```

It isn't always easy to write tail recursive functions though. Let's think
about a slightly harder problem: a tail recursive is-even function! As usual
we'll start with a bad implementation that will murder your stack:

```lisp
(defun bad-is-even (x)
  (case x
    (0 t)
    (1 nil)
    (otherwise (not (bad-is-even (- x 1))))))
```

Except we can't exactly add a nice accumulator argument (at least I don't
think so) since without modulo (cheating here) we can't work backwards
(we could earlier because multiplication is commutative) and our computation
must be done when coming back out of all the recursions. This is explained
a little weirdly but try and think through on your own why you can't use the
same strategy as before here.

There are some clever tricks we can do though! The first is being clever about
some "mutual recursion" in which we define two recursive functions `even` and
`odd` that call each other. Sidenote: these are still tail calls because the
recursion is the last thing that happens!

```lisp
(defun odd (x)
  (if (= x 0) nil (even (- x 1))))

(defun even (x)
  (if (= x 0) t (odd (- x 1))))
```

By using two functions we've nicely dodged our earlier problems. This is not
a generally applicable solution though, and by now you might despair that most
functions are not expressible as a tail recursive function. Yet this is not the
case! Observe the glory of Continuation Passing Style (what I was working on in
my functional programming pset all of last week).

Some background: a continuation passing style function is a tail recursive function
that additionally takes a lambda function of "what to do after this" (and calls it
as a tail call). You can start to build up some sneakiness here by calling more CPS
functions and build up arbitrary logic. Let's see a simple example with a factorial
function again:

```lisp
 ;; k is "what to do next" - we call k with our result when done
(defun cps-fact (x k)
  (if (zerop x) (funcall k 0) 
      ;; Here we call ourselves again but change the continuation: 
	  ;; now what we do with the result of this recursive call is
	  ;; multiply it by x and then pass it to k.
      (cps-fact (- x 1) (lambda (y) (funcall k (* x y))))))
```

It'll help to see what this evaluates to for an example input. I use `=>` to indicate
"evaluates to" in the following trace (also `write-to-string` is basically just `str()`
from Python but in Lisp.

```text
(cps-fact 3 write-to-string)
=> (cps-fact 2 (lambda (y) (funcall write-to-string (* 3 y))))
=> (cps-fact 1 (lambda (z) (funcall (lambda (y) (funcall write-to-string (* 3 y))) (* 2 z))))
=> (funcall (lambda (z) (funcall (lambda (y) (funcall write-to-string (* 3 y))) (* 2 z))) 1)
=> (funcall (lambda (y) (funcall write-to-string (* 3 y))) (* 2 1))
=> (funcall (lambda (y) (funcall write-to-string (* 3 y))) 2)
=> (funcall write-to-string (* 3 2))
=> (write-to-string 6)
=> "6"
```

Along the way we used nothing but tail calls! Hopefully after some more inspection
you can sort of see how we've basically punted all state to these growing lambda
functions that encode what to do next.

We can also do this for our `is-even` function! Here's how it would work: 

```lisp
(defun cps-even (x k)
  (case x
    (0 (funcall k t))
    (1 (funcall k nil))
    (otherwise (cps-even (- x 1) (lambda (y) (funcall k (not y)))))))
```

Try writing a trace out for yourself. Also we can call the function on big inputs!

```text
(cps-even 100003 'write-to-string)
(cps-even 100006 'write-to-string)
```

The cooler thing about continuation passing style is that it can express *anything*!
Maybe I'll send you some more complicated examples like what I did in my homework to
show this, but CPS is a whole style of programming (I wrote a simple SAT solver in SML
exclusively doing things this way!).

Anyways, this has been your primer on avoiding stack overflows in functional
languages. We can also be evil and do this kind of stuff imperatively. We can use
`setf` to update variables from let clauses in our factorial function.

```lisp
(defun evil-fact (x)
  (let ((out 1))
    (dotimes (_ x)
      (setf out (* x out)
            x (- x 1)))
    out))
```

Thankfully, Lisp is actually a multi-paradigm language and so if functional ever starts 
being a chore you can always just not bother.

Alternatively we can channel the dark arts of the loop macro:

```lisp
(defun loop-fact (x)
  (loop with out = 1
        for i from 2 to x
        do (setf out (* out i))
        finally (return out)))
```

Or if we like loop (it's actually the only clean `range()` equivalent) and hate `setf`
we could do a more functional implementation with reduce:

```lisp
(defun other-loop-fact (x)
  (reduce '* (loop for i from 1 to x collect i)))
```


