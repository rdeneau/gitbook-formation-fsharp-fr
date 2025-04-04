# Type Result

## Présentation

A.k.a `Either` _(Haskell)_

Modélise une _double-track_ Succès/Échec

```fsharp
type Result<'Success, 'Error> =   // ☝ 2 paramètres génériques
    | Ok of 'Success    // Track "Succès" == Happy path
    | Error of 'Error   // Track "Échec"  == Voie de garage
```

Gestion fonctionnelle des erreurs métier _(les erreurs prévisibles)_

* Permet de limiter usage des exceptions aux erreurs exceptionnelles
* Dès qu'une opération échoue, les opérations restantes ne sont pas lancées
* [_Railway-oriented programming_](https://fsharpforfunandprofit.com/rop/), F# for Fun and Profit

## Module `Result`

_Ne contient que 3 fonctions (en F# 5.0) :_

`map f option` : sert à mapper le résultat

* `('T -> 'U) -> Result<'T, 'Error> -> Result<'U, 'Error>`

`mapError f option` : sert à mapper l'erreur

* `('Err1 -> 'Err2) -> Result<'T, 'Err1> -> Result<'T, 'Err2>`

`bind f option` : idem `map` avec fonction `f` qui renvoie un `Result`

* `('T -> Result<'U, 'Error>) -> Result<'T, 'Error> -> Result<'U, 'Error>`
* 💡 Le résultat est aplati, comme la fonction `flatMap` sur les arrays JS
* :warning: Même type d'erreur `'Error` pour `f` et le `result` en entrée

💡 **Depuis F# 7.0**, le module `Result` contient plus de fonctions afin d'être aussi fonctionnel que le type `Option` , avec les correspondances suivantes :&#x20;

| Option                                                                                                   | Result                                                                                                   | List                                                                                                     |
| -------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| `Some value`                                                                                             | `Ok value`                                                                                               | `[ value ]`                                                                                              |
| `None`                                                                                                   | `Error ()` / `Error _`                                                                                   | `[  ]`                                                                                                   |
| `isSome` / `isNone`                                                                                      | `isOk` / `isError`                                                                                       | × / `isEmpty`                                                                                            |
| `contains someValue`                                                                                     | `contains okValue`                                                                                       | `contains value`                                                                                         |
| `count`                                                                                                  | `count`                                                                                                  | `length`                                                                                                 |
| <p><code>defaultValue value</code><br><code>defaultWith defThunk</code></p>                              | <p><code>defaultValue value</code><br><code>defaultWith defThunk</code></p>                              |                                                                                                          |
| <p><code>exists predicate</code><br><code>forall predicate</code></p>                                    | <p><code>exists predicate</code><br><code>forall predicate</code></p>                                    | <p><code>exists predicate</code><br><code>forall predicate</code></p>                                    |
| `filter predicate`                                                                                       | ×                                                                                                        | `filter predicate`                                                                                       |
| <p><code>iter action</code><br><code>map f</code> / <code>bind f</code><br><code>fold f state</code></p> | <p><code>iter action</code><br><code>map f</code> / <code>bind f</code><br><code>fold f state</code></p> | <p><code>iter action</code><br><code>map f</code> / <code>bind f</code><br><code>fold f state</code></p> |
| ×                                                                                                        | `mapError f`                                                                                             | ×                                                                                                        |
| `toArray` / `toList`                                                                                     | `toArray` / `toList`                                                                                     | ×                                                                                                        |
| ×                                                                                                        | `toOption`                                                                                               | ×                                                                                                        |

## Quiz 🎲

Implémenter `Result.map` et `Result.bind`

💡 **Tips :**

* _Mapping_ sur la track _Succès_
* Accès à la valeur dans la track _Succès_ :
  * Utiliser _pattern matching_ (`match result with...`)
* Retour : simple `Result`, pas un `Result<Result>` !

### Solution

```fsharp
// ('T -> 'U) -> Result<'T, 'Error> -> Result<'U, 'Error>
let map f result =
    match result with
    | Ok x    -> Ok (f x)  // ☝ Ok -> Ok
    | Error e -> Error e   // ⚠️ Les 2 `Error e` n'ont pas le même type !

// ('T -> Result<'U, 'Error>) -> Result<'T, 'Error>
//                            -> Result<'U, 'Error>
let bind f result =
    match result with
    | Ok x    -> f x       // ☝ Ok -> Ok ou Error !
    | Error e -> Error e
```

## `Result` : tracks Success/Failure

`map` : pas de changement de track

```
Track      Input          Operation      Output
Success ─ Ok x    ───► map( x -> y ) ───► Ok y
Failure ─ Error e ───► map(  ....  ) ───► Error e
```

`bind` : routage possible vers track Failure mais jamais l'inverse

```
Track     Input              Operation           Output
Success ─ Ok x    ─┬─► bind( x -> Ok y     ) ───► Ok y
                   └─► bind( x -> Error e2 ) ─┐
Failure ─ Error e ───► bind(     ....      ) ─┴─► Error ~
```

☝ Opération de _mapping/binding_ jamais exécutée dans track Failure

## `Result` _vs_ `Option`

`Option` peut représenter le résultat d'une opération qui peut échouer ☝ Mais en cas d'échec, l'option ne contient pas l'erreur, juste `None`

`Option<'T>` ≃ `Result<'T, unit>`

* `Some x` ≃ `Ok x`
* `None` ≃ `Error ()`
* Cf. fonctions `Option.toResult` et `Option.toResultWith error` de [FSharpPlus](http://fsprojects.github.io/FSharpPlus/reference/fsharpplus-option.html#toResult)

```fsharp
let toResultWith (error: 'Error) (option: 'T option) : Result<'T, 'Error> =
    match option with
    | Some x -> Ok x
    | None   -> Error error
```

**Exemple :**

Modification de la fonction `checkAnswer` précédente pour indiquer l'erreur :

```fsharp
open FSharpPlus

type Answer = A | B | C | D
type Error = InvalidInput | WrongAnswer

let tryParseAnswer text = ... // string -> Answer option

let checkAnswer (expectedAnswer: Answer) (givenAnswer: string) =
    let check answer = if answer = expectedAnswer then Ok answer else Error WrongAnswer
    tryParseAnswer givenAnswer           // Answer option
    |> Option.toResultWith InvalidInput  // Result<Answer, Error>
    |> Result.bind check
    |> function
       | Ok _               -> "✅"
       | Error InvalidInput -> "❌ Invalid Input"
       | Error WrongAnswer  -> "❌ Wrong Answer"

["X"; "A"; "B"] |> List.map (checkAnswer B)  // ["❌ Invalid Input"; "❌ Wrong Answer"; "✅"]
```

## `Result` _vs_ `Validation`

`Result` est "monadique" : à la 1ère erreur, on "débranche"

`Validation` est "applicatif" : permet d'accumuler les erreurs

* ≃ `Result<'ok, 'error list>`
* Pratique pour valider saisie utilisateur et remonter ∑ erreurs
* Dispo dans librairies [FSharpPlus](https://github.com/fsprojects/FSharpPlus), [FsToolkit.ErrorHandling](https://github.com/demystifyfp/FsToolkit.ErrorHandling)

_Plus d'info :_ 🔗 [Validation with F# 5 and FsToolkit](https://www.compositional-it.com/news-blog/validation-with-f-5-and-fstoolkit/), Compositional IT, Dec 2020
