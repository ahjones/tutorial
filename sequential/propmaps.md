# Property Lists and Maps

## Property Lists

Property lists in Erlang and LFE are a simple way to create key-value pairs. They have a very simple structure: a list of tuples, where the key is the first element of each tuple and is an atom. Very often you will see "options" for functions provided as property lists (this is similar to how other programming languages use keywords in function arguments).

Property lists can be created with just the basic data structures of LFE:

```lisp
> (set options (list (tuple 'debug 'true)
                     (tuple 'default 42)))
(#(debug true) #(default 42))
```

Or, more commonly, using quoted literals:

```lisp
> (set options '(#(debug true) #(default 42)))
(#(debug true) #(default 42))
```

There are convenience functions provided in the ``proplists`` module. In the last example, we define a default value to be used in the event that the given key is not found in the proplist:

```lisp
> (proplists:get_value 'default options)
42
> (proplists:get_value 'poetry options)
undefined
> (proplists:get_value 'poetry options "Vogon")
"Vogon"
```

Be sure to read the [module documentation](http://www.erlang.org/doc/man/proplists.html) for more information. Here's an example of our options in action:

```lisp
(defun div (a b)
  (div a b '()))

(defun div (a b opts)
  (let ((debug (proplists:get_value 'debug opts 'false))
        (ratio? (proplists:get_value 'ratio opts 'false)))
    (if (and debug ratio?)
        (io:format "Returning as ratio ...~n"))
    (if ratio?
        (++ (integer_to_list 1) "/" (integer_to_list 2))
        (/ a b))))
```

Let's try our funtion without and then with various options:

```lisp
> (div 1 2)
0.5
> (div 1 2 '(#(ratio true)))
"1/2"
> (div 1 2 '(#(ratio true) #(debug true)))
Returning as ratio ...
"1/2"
```

## Maps

As with property lists, maps are a set of key to value associations. You may create an association from "key" to value 42 in one of two ways: using the LFE core form ``map`` or entering a map literal:

```lisp
> (map "key" 42)
#M("key" 42)
> #M("key" 42)
#M("key" 42)
```

We will jump straight into the deep end with an example using some interesting features. The following example shows how we calculate alpha blending using maps to reference color and alpha channels:

```lisp
> (defmacro channel? (val)
    `(andalso (is_float ,val)
              (>= ,val 0.0)
              (=< ,val 1.0)))
()
> (defmacro all-channels? (r g b a)
    `(andalso (channel? ,r)
              (channel? ,g)
              (channel? ,b)
              (channel? ,a)))
()
> (defun new
    ((r g b a) (when (all-channels? r g b a))
     (map 'red r 'green g 'blue b 'alpha a)))
new
> (defun blend (src dst)
    (blend src dst (alpha src dst)))
blend
> (defun blend
    ((src dst alpha) (when (> alpha 0.0))
     (map-update dst
                 'red (/ (red src dst) alpha)
                 'green (/ (green src dst) alpha)
                 'blue (/ (blue src dst) alpha)
                 'alpha alpha))
    ((_ dst _)
     (map-update 'red 0 'green 0 'blue 0 'alpha 0)))
blend
> (defun alpha
    (((map 'alpha src-alpha)
      (map 'alpha dst-alpha))
     (+ src-alpha (* dst-alpha (- 1.0 src-alpha)))))
alpha
> (defun red
    (((map 'red src-val 'alpha src-alpha)
      (map 'red dst-val 'alpha dst-alpha))
     (* src-val
        (+ src-alpha
           (* dst-val dst-alpha (- 1.0 src-alpha))))))
red
> (defun green
    (((map 'green src-val 'alpha src-alpha)
      (map 'green dst-val 'alpha dst-alpha))
     (* src-val
        (+ src-alpha
           (* dst-val dst-alpha (- 1.0 src-alpha))))))
green
> (defun blue
    (((map 'blue src-val 'alpha src-alpha)
      (map 'blue dst-val 'alpha dst-alpha))
     (* src-val
        (+ src-alpha
           (* dst-val dst-alpha (- 1.0 src-alpha))))))
blue
```

Now let's try it out:

```lisp
> (set color-1 (new 0.3 0.4 0.5 1.0))
#M(alpha 1.0 blue 0.5 green 0.4 red 0.3)
> (set color-2 (new 1.0 0.8 0.1 0.3))
#M(alpha 0.3 blue 0.1 green 0.8 red 1.0)
> (blend color-1 color-2)
#M(alpha 1.0 blue 0.5 green 0.4 red 0.3)
> (blend color-2 color-1)
#M(alpha 1.0 blue 0.06499999999999999 green 0.46399999999999997
   red 0.51)
```

This example warrants some explanation.

First we define a couple macros to help with our guard tests. This is only here for convenience and to reduce syntax cluttering. Guards can be only composed of a limited set of functions, so we needed to use macros that would compile down to just the funtions allowed in guards. A full treatment of Lisp macros is beyond the scope of this tutorial, but there is a lot of good material available online for learning macros, including Paul Graham's book "On Lisp."

[more content coming ...]