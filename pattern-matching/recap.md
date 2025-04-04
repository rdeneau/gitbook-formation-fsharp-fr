# 📜 Récap’

## Pattern matching

* Brique fondamentale de F♯
* Combine "comparaison de structure de données" et "déconstruction"
* S'utilisent presque partout :
  * `match expression` et bloc `function`
  * bloc `try/with`
  * `let binding`, y.c. paramètre de fonction
* Peut s'abstraire en fonction `fold` associée à un type union

## Patterns

| Pattern                            | Exemple                           |
| ---------------------------------- | --------------------------------- |
| Constant - Identifier - Wilcard    | `1`, `Color.Red` - `Some 1` - `_` |
| _Collection_ : Cons - List - Array | `head :: tail` - `[1; 2]`         |
| _Product type_ : Record - Tuple    | `{ A = a }` - `a, b`              |
| Type Test                          | `:? Subtype`                      |
| _Logique_ : OR, AND                | `1 \| 2`, `P1 & P2`               |
| Variables - Alias                  | `head :: _` - `(0, 0) as origin`  |

\+ Les guards `when` dans les match expressions

## Active Patterns

* Extension du pattern matching
* Basés sur fonction + metadata → Citoyens de 1ère classe
* 4 types : total simple/multiple, partiel (simple), paramétré
* Un peu tricky à comprendre mais on s'habitue vite
* S'utilisent pour :
  * Ajouter de la sémantique sans recourir aux types union
  * Simplifier / factoriser des guards
  * Wrapper des méthodes de la BCL
  * Extraire un ensemble de données d'un objet
  * ...

## Compléments

📜 [Match expressions](https://fsharpforfunandprofit.com/posts/match-expression/), F# for fun and profit, Juin 2012

📜 [Domain modelling et pattern matching : Roman numeral](https://fsharpforfunandprofit.com/posts/roman-numerals/), F# for fun and profit, Juin 2012

📜 [Recursive types and folds](https://fsharpforfunandprofit.com/series/recursive-types-and-folds/) _(6 articles),_ F# for fun and profit, Août 2015

📹 [A Deep Dive into Active Patterns](https://www.youtube.com/watch?v=Q5KO-UDx5eA) - [Repo github](https://github.com/pblasucci/DeepDiveAP)

## Exercices

Les exercices suivants sur [exercism.io](https://exercism.io/tracks/fsharp) _(se créer un compte)_ peuvent se résoudre avec des active patterns :

* Collatz Conjecture _(easy)_
* Darts _(easy)_
* Queen Attack _(medium)_
* Robot Name _(medium)_
