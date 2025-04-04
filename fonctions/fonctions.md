# Fonctions

## Binding d'une fonction

`let f x y = x + y + 1`

* Binding réalisé avec mot clé `let`
* Associe à la fois un nom (`f`) et les paramètres (`x` et `y`)
* Annotation de type optionnelle pour paramètres et/ou retour
  * `let f (x: int) (y: int) : int = ...`
  * Sinon, inférence de type, avec possible généralisation auto
* Dernière expression → valeur de retour de la fonction
* Possible définition de sous-fonctions (non génériques)

## Fonction générique

* Dans beaucoup de cas, inférence marche avec généralisation auto
  * `let listOf x = [x]` → `(x: 'a) -> 'a list`
* Annotation explicite de params génériques
  * `let f (x: 'a) = ...` (pas besoin de faire `f<'a>` grâce au `'` 👍)
* Annotation explicite avec inférence du type générique
  * `let f (list: list<_>) = ...`

## Fonction anonyme / Lambda

Expression définissant une fonction

Syntaxe : `fun parameter1 parameter2 etc -> expression`

☝ **À noter :**

* Mot clé `fun` obligatoire
* Flèche fine `->` (Java) ≠ flèche grasse `=>` (C♯, Js)

### Lambda : quelques usages

#### **1.** En argument d'une _high-order function_

* Pour éviter de devoir définir une fonction nommée
* Recommandée pour une fonction courte, pour que cela reste lisible

```fsharp
[1..10] |> List.map (fun i -> i + 1) // 👈 () autour de la lambda

// Versus en passant par une fonction nommée
let add1 i = i + 1
[1..10] |> List.map add1
```

:warning: Attention au lambda inutile : `List.map (fun x -> f x)` ≡ `List.map f`

#### **2.** Dans un _let binding_ avec inférence

* Pour rendre explicite quand la fonction renvoie une fonction
* Sorte de curryfication manuelle
* À utiliser avec parcimonie :exclamation:

```fsharp
let add x y = x + y                     // Version normale, curryfiée automatiquement
let add' x = fun y -> x + y             // Idem avec une sous lambda
let add'' = fun x -> (fun y -> x + y)   // Idem en totalement "lambda-isée"
```

#### **3.** Dans un _let binding_ annoté

* Signature de la fonction pré-définie sous forme d'un type
* Type "fonction" s'utilise un peu comme une `interface` C♯
  * Pour contraindre implémentation à suivre signature
  * Ex : _Domain modelling made functional_ par Scott Wlaschin

```fsharp
type Add = int -> int -> int

let add: Add =
    fun x y ->
        x + y

// val add: x: int -> y: int -> int
// ☝ Les paramètres sont nommés 👍
```

### Conversion en Func

On peut convertir une lambda en `Func<>` mais de manière explicite en passant par son constructeur :&#x20;

```fsharp
let isNull = System.Func<_, _> (fun x -> x = null)
// val isNull: System.Func<'a,bool> when 'a: equality and 'a: null
```

### Conversion en Expression

C'est un peu plus complexe d'écrire une `Expression<Func<>>` à partir d'une lambda. Il faut passer par une _quotation_ (\*) puis la convertir en expression en utilisant soit le `FSharp.Linq.RuntimeHelpers.LeafExpressionConverter` pour les cas simples (cf. exemple ci-dessous), soit le package NuGet `FSharp.Quotations.Evaluator` pour les cas plus complexes.

```fsharp
open System
open System.Linq.Expressions
open Microsoft.FSharp.Linq.RuntimeHelpers

let isNullQuot = <@ Func<_, _> (fun (x: obj) -> x = null) @>
// val isNullQuot: Quotations.Expr<System.Func<obj,bool>> =
//   NewDelegate (Func`2, x, Call (None, op_Equality, [x, Value (<null>)]))

let isNullExpr =
    isNullQuot
    |> LeafExpressionConverter.QuotationToExpression 
    |> unbox<Expression<Func<obj, bool>>>
// val isNullExpr: System.Linq.Expressions.Expression<System.Func<obj,bool>> =
//   x => (x == null)
```

**(\*) Note :** Les _quotations_ sont hors scope. Juste 2 mots à leur propos :\
• Elles s'écrivent entre `<@ ... @>`.\
• Ce sont un peu l'équivalent des Expressions en C# - plus d'info [ici](https://stackoverflow.com/a/8134113/8634147).

### Mot clé `function`

* Permet aussi de définir une fonction anonyme définissant une expression de match
* Syntaxe abrégée équivalente à `fun x -> match x with`
* Prend 1 paramètre qui est implicite

```fsharp
let ouiNon x =
  match x with
  | true  -> "Oui"
  | false -> "Non"
// val ouiNon: x: bool -> string

// 👉 Réécrit avec `function`:
let ouiNon = function
  | true  -> "Oui"
  | false -> "Non"
// val ouiNon: _arg1: bool -> string
```

☝ Son emploi est une question de goût : \
• Pour les adeptes du point-free - cf. [#style-point-free](operateurs.md#style-point-free "mention") 📍\
• Plus généralement dans une succession de _pipes_ `arg |> f1 |> function ... |> f2...`\
→ L'idée est de faire un pattern matching sur une large expression sans pour autant la stocker dans une variable intermédiaire.

## Déconstruction de paramètres

* Comme en JavaScript, on peut déconstruire _inline_ un paramètre
* C'est également une façon d'indiquer le type du paramètre
* Le paramètre apparaît sans nom dans la signature

Exemple avec un type _Record_ 📍[records.md](../types-composites/records.md "mention")

```fsharp
type Person = { Name: string; Age: int }

let name { Name = x } = x     // Person -> string
let age { Age = x } = x       // Person -> int
let age' person = person.Age  // Equivalent explicite

let bob = { Name = "Bob"; Age = 18 } // Person
let bobAge = age bob // int = 18
```

On parle aussi de _pattern matching_ \
→ Mais je préfère réserver ce terme pour l'usage de `match x with ...`

Déconstruction pas adaptée pour un type union avec plusieurs cas 📍 [unions.md](../types-composites/unions.md "mention")

* 💡 **Exemple :** liste F♯ _(soit vide `[]`, soit valeur + sous-liste `head::tail`)_&#x20;
* :point\_up: **Solution :** faire un _pattern matching_ de tous les cas de l'union

```fsharp
let printFirstItem (x::_) = // 'a list -> unit
//                  ~~~~  Warning FS0025: Critères spéciaux incomplets dans cette expression.
   printfn $"first element: {x}"

let printFirstItemOk = function
    | x::_ -> printfn $"first element: {x}"
    | []   -> printfn "none"
```

## Paramètre tuple

* Comme en C♯, on peut vouloir regrouper des paramètres d'une fonction
  * Par soucis de cohésion, quand ces paramètres forment un tout
  * Pour éviter le _code smell_ [long parameter list](https://refactoring.guru/smells/long-parameter-list)
* On peut les regrouper dans un tuple et même le déconstruire

```fsharp
// V1 : trop de paramètres
let f x y z = ...

// V2 : paramètres regroupés dans un tuple
let f params =
    let (x, y, z) = params
    ...

// V3 : idem avec tuple déconstruit sur place
let f (x, y, z) = ...
```

* `f (x, y, z)` ressemble furieusement à une méthode C♯ !
* La signature signale le changement : `(int * int * int) -> TResult`
  * La fonction n'a effectivement plus qu'1! paramètre plutôt que 3
  * Perte possibilité application partielle de chaque élément du tuple

☝ **Conclusion** :

* Resister à la tentation d'utiliser tout le temps un tuple _(car familier - C♯)_
* Réserver cet usage quand c'est pertinent de regrouper les paramètres
  * Sans pour autant déclarer un type spécifique pour ce groupe

## Fonction récursive

* Fonction qui s'appelle elle-même
* Syntaxe spéciale avec mot clé `rec` sinon erreur `FS0039: … is not defined`
* Très courant en F♯ pour remplacer les boucles `for`
  * Car c'est souvent + facile à concevoir

Exemple : trouver nb étapes pour atteindre 1 dans la [suite de Syracuse](https://fr.wikipedia.org/wiki/Conjecture_de_Syracuse) / Collatz

```fsharp
let rec steps (n: int) : int =
    if n = 1       then 0
    elif n % 2 = 0 then 1 + steps (n / 2)
    else                1 + steps (3 * n + 1)
```

## _Tail recursion_

* Type de récursivité où l'appel récursif est la dernière instruction
* Détecté par le compilateur et optimisé sous forme de boucle
  * Permet d'éviter les `StackOverflow`
* Procédé classique pouvant rendre tail récursif :
  * Ajouter un param "accumulateur", comme `fold`/`reduce`

```fsharp
let steps (number: int) : int =
    let rec loop count n = // 👈 `loop` = nom idiomatique de ce type de fonction interne récursive
        if n = 1       then count
        elif n % 2 = 0 then loop (count + 1) (n / 2)      // 👈 Dernier appel : `loop`
        else                loop (count + 1) (3 * n + 1)  // 👈 idem
    loop 0 number // 👈 Lancement de la boucle avec 0 comme valeur initiale pour `count`
```

## Fonctions mutuellement récursives

* Fonctions qui s'appellent l'une l'autre
* Doivent être déclarées ensemble :
  * 1ère fonction indiquée comme récursive avec `rec`
  * autres fonctions ajoutées à la déclaration avec `and`

```fsharp
// ⚠️ Algo un peu alambiqué servant juste d'illustration

let rec Even x =        // 👈 Mot clé `rec`
    if x = 0 then true
    else Odd (x-1)      // 👈 Appel à `Odd` définie + bas
and Odd x =             // 👈 Mot clé `and`
    if x = 0 then false
    else Even (x-1)     // 👈 Appel à `Even` définie + haut
```

## Surcharge / _overload_ de fonctions

:warning: Pas possible de surcharger une fonction

💡 Noms différents :

* `List.map (mapping: 'T -> 'U) list`
* `List.mapi (mapping: (index: int) -> 'T -> 'U) list`

💡 Implémentation via fonction template 👇

## Fonction template

Permet de créer des "surcharges" spécialisées :

```fsharp
type ComparisonResult = Bigger | Smaller | Equal

// Fonction template, 'private' pour la "cacher"
let private compareTwoStrings (comparison: StringComparison) string1 string2 =
    let result = System.String.Compare(string1, string2, comparison)
    if result > 0 then
        Bigger
    else if result < 0 then
        Smaller
    else
        Equal

// Application partielle du paramètre 'comparison'
let compareCaseSensitive   = compareTwoStrings StringComparison.CurrentCulture
let compareCaseInsensitive = compareTwoStrings StringComparison.CurrentCultureIgnoreCase
```

☝ Emplacement du paramètre de spécialisation :

* En C♯, en dernier :

```csharp
String.Compare(String, String, StringComparison)
String.Compare(String, String)
```

* En F♯, en premier pour permettre application partielle :

```fsharp
compareTwoStrings    : StringComparison -> String -> String -> ComparisonResult
compareCaseSensitive :                     String -> String -> ComparisonResult
```

## Fonction `inline`

### Principe

> **Extension inline** ou **inlining :** optimisation d'un compilateur qui remplace un appel de fonction par le code _(le corps)_ de cette fonction.\
> 🔗 [https://fr.wikipedia.org/wiki/Extension\_inline](https://fr.wikipedia.org/wiki/Extension_inline)

* Gain de performance 👍
* Compilation + longue :warning:

💡 Similaire au refacto [_Inline Method_](https://refactoring.guru/inline-method)

### Mot clé `inline`

Indique au compilateur de _"inliner"_ la fonction

Usage typique : petite fonction/opérateur de "sucre syntaxique"

Exemples issus de [FSharp.Core](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/prim-types.fs) : \
→ Fonction `ignore`    📍 [fonctions-standard.md](fonctions-standard.md "mention")\
→ Opérateur `|>`          📍 [#operateur-pipe-right-or-greater-than](operateurs.md#operateur-pipe-right-or-greater-than "mention")

```fsharp
let inline ignore _ = ()
let inline (|>) x f = f x

let t = true |> ignore
//   ~= ignore true // Après inline du pipe
//   ~= ()          // Après inline de ignore
```

L'autre usage de fonctions `inline` concerne le [#srtp](../types-complements/generiques.md#srtp "mention") 📍

