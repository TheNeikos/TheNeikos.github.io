---
layout: post
date: 2014-10-23 12:26:58
title: Functional Programming
---

Functional Programming. I have often heard about the difficulty in functional
programming. Immutability, pure functions, Monads a few dozen more of things to
know to get started.

For my part, I didn't find it too difficult in getting started in Racket. While
Racket doesn't seem to be as strict as say Haskell, it seems to be rather nice
nonetheless and *fast*! I wrote two simple programs in it, both are simple games
so not purely functional (as they have side effects, namely drawing on the
screen).

The first one is a simple "Guess the Number" style game. A random value is
generated and then the user has to guess it.


```racket
#lang racket


(define number (random 100))

(define (win)
  (display "You won the game! In ")
  (display tries)
  (display " tries!\n")
  (exit))

(define (give-hint g)
  (if (> g number)
    (display "You have to go smaller\n")
    (display "You have to go bigger\n")))

(define (try-again g)
  (display "Nope! ")
  (give-hint g)
  (set! tries (add1 tries)))


(define (check-input g)
    (if (= g number)
      (win)
      ((try-again g)
       (guessing))))



(define tries 1)
(define (guessing)
  (let ([g (read)])
    (check-input g)))

(guessing)
```

I refactored it a bit, although that was just to see if it looked better like
that.

I am also in the process of writing a simple game where you have to dodge blocks
falling down, losing if one hits you. It's a bit more complicated, but that was
to be expected with graphical output.
