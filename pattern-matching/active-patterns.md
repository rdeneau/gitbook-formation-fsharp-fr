# 🚀 Active Patterns

## Limitations du _Pattern Matching_

Nombre limité de patterns

Impossibilité de factoriser l'action de patterns avec leur propre guard

* `Pattern1 when Guard1 | Pattern2 when Guard2 -> do` 💥
* `Pattern1 when Guard1 -> do | Pattern2 when Guard2 -> do` 😕

Patterns ne sont pas des citoyens de 1ère classe

* _Ex : une fonction ne peut pas renvoyer un pattern_
* → Juste une sorte de sucre syntaxique

Patterns interagissent mal avec un style OOP

## Origine des _Active Patterns_

> 🔗 [_Extensible pattern matching via a lightweight language extension_](https://www.microsoft.com/en-us/research/publication/extensible-pattern-matching-via-a-lightweight-language-extension/)&#x20;
>
> ℹ️ Publication de 2007 de Don Syme, Gregory Neverov, James Margetson

Intégré à F♯ 2.0 (2010)

💡 **Idées**

* Permettre le _pattern matching_ sur d'autres structures de données
* Faire de ces nouveaux patterns des citoyens de 1ère classe

## Syntaxe

Syntaxe générale : `let (|Cases|) [arguments] valueToMatch = expression`

1. **Fonction** avec un nom spécial défini dans une "banane" `(|...|)`
2. Ensemble de 1..N **cases** où ranger `valueToMatch`

💡 Sorte de fonction _factory_ d'un **type union** "anonyme", défini _inline_

## Types

Il existe 4 types d'active patterns :

1. Pattern total simple
2. Pattern total multiple
3. Pattern partiel
4. Pattern paramétré

💡 _Partiel_ et _total_ indique la faisabilité du « rangement dans le(s) case(s) » de la valeur en entrée     &#x20;

* **Partiel** : il n'existe pas toujours une case correspondante
* **Total** : il existe forcément une case correspondante → pattern exhaustif

### Active pattern total simple

_A.k.a Single-case Total Pattern_

Syntaxe : `let (|Case|) [...parameters] value = Case [data]` Usage : déconstruction en ligne

```fsharp
// Avec paramètre => pas très lisible 😕
let (|Default|) = Option.defaultValue  // 'T -> 'T option -> 'T

let (Default "unknown" name1) = Some "John"  // name1 = "John"
let (Default "unknown" name2) = None         // name2 = "unknown"

// Sans paramètre => mieux 👌
let (|ValueOrUnknown|) = Option.defaultValue "unknown"  // 'T option -> 'T

let (ValueOrUnknown name1) = Some "John"  // name1 = "John"
let (ValueOrUnknown name2) = None         // name2 = "unknown"
```

Autre exemple : extraction de la forme polaire d'un nombre complexe

```fsharp
open System.Numerics

let (|Polar|) (x : Complex) =
    Polar (x.Magnitude, x.Phase)

let multiply (Polar (m1, p1)) (Polar (m2, p2)) =  // Complex -> Complex -> Complex
    Complex(m1 + m2, p1 + p2)
```

Sans l'active pattern, c'est un autre style mais de lisibilité équivalente :

```fsharp
let multiply x y =
    Complex (x.Magnitude + y.Magnitude, x.Phase + y.Phase)
```

### Active pattern total multiple

_A.k.a Multiple-case Total Pattern_

Syntaxe : `let (|Case1|...|CaseN|) value = CaseI [dataI]`&#x20;

☝ Ne peut pas prendre de paramètre d’entrée❗

```fsharp
// Ré-écriture d'un exemple précédent

// ❌ type Parity = Even of int | Odd of int
// ❌ let parityOf value = if value % 2 = 0 then Even value else Odd value

let (|Even|Odd|) x =  // int -> Choice<int, int>
    if x % 2 = 0 then Even x else Odd x

let hasSquare square value =
    // ❌ match parityOf square, parityOf value with
    match square, value with
    | Even x2, Even x | Odd x2, Odd x when x2 = x*x -> true
    | _ -> false
```

### Active pattern partiel

Syntaxe : `let (|Case|_|) value = Some Case | Some data | None`

* Renvoie type `'T option` si _Case_ comprend des données, sinon `unit option`
* Pattern matching est non exhaustif → il faut un cas par défaut

```fsharp
let (|Integer|_|) (x: string) = // (x: string) -> int option
    match System.Int32.TryParse x with
    | true, i -> Some i
    | false, _ -> None

let (|Float|_|) (x: string) = // (x: string) -> float option
    match System.Double.TryParse x with
    | true, f -> Some f
    | false, _ -> None

let detectNumber = function
    | Integer i -> $"Integer {i}"   // detectNumber "10"
    | Float f -> $"Float {f}"       // detectNumber "1,1" = "Float 1,1" (en France)
    | s -> $"NaN {s}"               // detectNumber "abc" = "NaN abc"
```

Exemple similaire, où les active patterns sont écrits la fonction `Option.ofTuple` :

```fsharp
module Option =
    let ofTuple =
        function
        | true, value -> Some value
        | false, _ -> None

module Parsing =
    open System

    let (|AsBoolean|_|) (value: string) =
        Boolean.TryParse value |> Option.ofTuple

    let (|AsInteger|_|) (value: string) =
        Int32.TryParse value |> Option.ofTuple

    let tryParseBoolean =
        function
        | AsBoolean b -> Ok b
        | AsInteger 0 -> Ok false
        | AsInteger 1 -> Ok true
        | value -> Error $"{value} is not a valid boolean"
```

💡 Pour bien se rendre du gain de lisibilité du code, il suffit d'écrire une version de + bas niveau de `tryParseBoolean` où l'on constate :&#x20;

* Imbrication des expressions `match`
* Difficultés à lire les lignes 6 et 7 du fait des doubles booléens (`true..false`, `true..true`)

{% code lineNumbers="true" %}
```fsharp
    let tryParseBoolean' (value: string) =
        match Boolean.TryParse value with
        | true, b -> Ok b
        | false, _ ->
            match Int32.TryParse value with
            | true, 0 -> Ok false
            | true, 1 -> Ok true
            | _ -> Error $"{value} is not a valid boolean"
```
{% endcode %}

### Active pattern partiel paramétré

Syntaxe : `let (|Case|_|) ...arguments value = Some Case | Some data | None`

**Exemple 1 :** année bissextile = multiple de 4 mais pas 100 sauf 400

```fsharp
let (|DivisibleBy|_|) factor x =  // (factor: int) -> (x: int) -> unit option
    match x % factor with
    | 0 -> Some DivisibleBy
    | _ -> None

let isLeapYear year =  // (year: int) -> bool
    match year with
    | DivisibleBy 400 -> true
    | DivisibleBy 100 -> false
    | DivisibleBy   4 -> true
    | _               -> false
```

**Exemple 2 :** expression régulière

```fsharp
let (|Regexp|_|) pattern value =  // string -> string -> string list option
    let m = System.Text.RegularExpressions.Regex.Match(value, pattern)
    if not m.Success || m.Groups.Count < 1 then
        None
    else
        [ for g in m.Groups -> g.Value ]
        |> List.tail // drop "root" match
        |> Some
```

**Exemple :** Couleur hexadécimale

```fsharp
let hexToInt hex =  // string -> int // E.g. "FF" -> 255
    System.Int32.Parse(hex, System.Globalization.NumberStyles.HexNumber)

let (|HexaColor|_|) = function  // string -> (int * int * int) option
    // 💡 Utilise l'active pattern précédent
    // 💡 La Regex recherche 3 groupes de 2 chars étant un chiffre ou une lettre A..F
    | Regexp "#([0-9A-F]{2})([0-9A-F]{2})([0-9A-F]{2})" [ r; g; b ] ->
        Some <| HexaColor ((hexToInt r), (hexToInt g), (hexToInt b))
    | _ -> None

match "#0099FF" with
| HexaColor (r, g, b) -> $"RGB: {r}, {g}, {b}"
| otherwise -> $"'{otherwise}' is not a hex-color"
// "RGB: 0, 153, 255"
```

### Récapitulatif

| Type           | Syntaxe                    | Signature                            |
| -------------- | -------------------------- | ------------------------------------ |
| Total multiple | `let (｜Case1｜…｜CaseN｜) x`  | `'T -> Choice<'U1, …, 'Un>`          |
| Total simple   | `let (｜Case｜) x`           | `'T -> 'U`                           |
| Partiel simple | `let (｜Case｜_｜) x`         | `'T -> 'U option`                    |
| ... paramétré  | `let (｜Case｜_｜) p1 … pN x` | `'P1 -> … -> 'Pn -> 'T -> 'U option` |

## Comprendre un active pattern

> Comprendre comment utiliser un active pattern... peut s'avérer un vrai **jonglage intellectuel** !

👉 Explications en utilisant les exemples précédents

### Comprendre un active pattern total

* Active pattern total ≃ Fonction _factory_ d'un type union "anonyme"
* Usage : idem pattern matching d'un type union normal

```fsharp
// Single-case
let (|Cartesian|) (x: Complex) = Cartesian (x.Real, x.Imaginary)

let Cartesian (r, i) = Complex (1.0, 2.0)  // r = 1.0, i = 2.0

// Double-case
let (|Even|Odd|) x = if x % 2 = 0 then Even else Odd

let parityOf = function  // int -> string
    | Even -> "Pair"
    | Odd  -> "Impair"
```

### Comprendre un active pattern partiel

☝ Bien distinguer les éventuels paramètres des éventuelles données

Examiner la signature de l'active pattern : `[...params ->] value -> 'U option`

* Les 1..N-1 paramètres = paramètres de l'active pattern
* Son retour : `'U option` → données de type `'U` ; si `'U` = `unit` → pas de donnée

À l'usage : `match value with Case [params] [data]`

* `Case params` ≃ **application partielle**, donnant active pattern sans paramètre
* `CaseWithParams data` ≃ déconstruction d'un case de type union

→ Exemples vus :

1. `let (|Integer|_|) (s: string) : int option`
   * Usage `match s with Integer i`, avec `i: int` donnée en sortie
2. `let (|DivisibleBy|_|) (factor: int) (x: int) : unit option`
   * Usage `match year with DivisibleBy 400`, avec `400` le paramètre `factor`
3. `let (|Regexp|_|) (pattern: string) (value: string) : string list option`
   * Usage `match s with Regexp "#([0-9...)" [ r; g; b ]`
   * Avec `"#([0-9...)"` le paramètre `pattern`
   * Et `[ r; g; b ]` la liste en sortie décomposée en 3 chaînes

## Exercice : fizz buzz

Ré-écrire ce fizz buzz en utilisant un active pattern `DivisibleBy`

```fsharp
let isDivisibleBy factor number =
    number % factor = 0

let fizzBuzz = function
    | i when i |> isDivisibleBy 15 -> "FizzBuzz"
    | i when i |> isDivisibleBy  3 -> "Fizz"
    | i when i |> isDivisibleBy  5 -> "Buzz"
    | other -> string other

[1..15] |> List.map fizzBuzz
// ["1"; "2"; "Fizz"; "4"; "Buzz"; "Fizz";
//  "7"; "8"; "Fizz"; "Buzz"; "11";
//  "Fizz"; "13"; "14"; "FizzBuzz"]
```

### Solution

```fsharp
let isDivisibleBy factor number =
    number % factor = 0

let (|DivisibleBy|_|) factor number =
    if number |> isDivisibleBy factor
    then Some DivisibleBy // 💡 Ou `Some ()`
    else None

let fizzBuzz = function
    | DivisibleBy 3 &
      DivisibleBy 5 -> "FizzBuzz"  // 💡 Ou `DivisibleBy 15`
    | DivisibleBy 3 -> "Fizz"
    | DivisibleBy 5 -> "Buzz"
    | other -> string other

[1..15] |> List.map fizzBuzz
// ["1"; "2"; "Fizz"; "4"; "Buzz"; "Fizz";
//  "7"; "8"; "Fizz"; "Buzz"; "11";
//  "Fizz"; "13"; "14"; "FizzBuzz"]
```

💡 L'active pattern `DivisibleBy 3`  ne renvoie pas de donnée. C'est juste du sucre syntaxique équivalent de `if y |> isDivisibleBy 3` . Dans un tel cas, [F# 9](https://learn.microsoft.com/en-us/dotnet/fsharp/whats-new/fsharp-9#partial-active-patterns-can-return-bool-instead-of-unit-option) autorise à renvoyer directement le booléen plutôt que de devoir passer par le type `Option` :

```fsharp
let (|DivisibleBy|_|) factor number =
    number |> isDivisibleBy factor
```

### Alternative

```fsharp
let isDivisibleBy factor number =
    number % factor = 0

let boolToOption b =
    if b then Some () else None

let (|Fizz|_|) number = number |> isDivisibleBy 3 |> boolToOption
let (|Buzz|_|) number = number |> isDivisibleBy 5 |> boolToOption

let fizzBuzz = function
    | Fizz & Buzz -> "FizzBuzz"
    | Fizz -> "Fizz"
    | Buzz -> "Buzz"
    | other -> string other
```

→ Les 2 solutions se valent. C'est une question de style / de goût personnel.

## Cas d'utilisation des actives patterns

1. Factoriser une guard _(cf. exercice précédent du fizz buzz)_
2. Wrapper une méthode de la BCL _(cf. `(|Regexp|_|)` et ci-dessous)_
3. Améliorer l'expressivité, aider à comprendre la logique _(cf. après)_

```fsharp
let (|ParsedInt|UnparsableInt|) (input: string) =
    match input with
    | _ when fst (System.Int32.TryParse input) -> ParsedInt(int input)
    | _ -> UnparsableInt

let addOneOrZero = function
    | ParsedInt i -> i + 1
    | UnparsableInt -> 0

let v1 = addOneOrZero "1"  // 2
let v2 = addOneOrZero "a"  // 0
```

## Expressivité grâce aux actives patterns

```fsharp
type Movie = { Title: string; Director: string; Year: int; Studio: string }

module Movie =
    let private boolToOption b =
        if b then Some () else None

    let (|Director|_|) director movie =
        movie.Director = director |> boolToOption

    let (|Studio|_|) studio movie =
        movie.Studio = studio |> boolToOption

    let private matchYear comparator year movie =
        (comparator movie.Year year) |> boolToOption

    let (|After|_|) = matchYear (>)
    let (|Before|_|) = matchYear (<)
    let (|In|_|) = matchYear (=)

open Movie

let ``Is anime rated 10/10`` = function
    | ((After 2001 & Before 2007) | In 2014) & Studio "Bones"
    | Director "Hayao Miyazaki" -> true
    | _ -> false
```

## Active pattern : citoyen de 1ère classe

Un active pattern ≃ fonction avec des métadonnées

Citoyen de 1ère classe :

```fsharp
// 1. Renvoyer un active pattern depuis une fonction
let (|Hayao_Miyazaki|_|) movie =
    (|Director|_|) "Hayao Miyazaki" movie

// 2. Prendre un active pattern en paramètre -- Un peu tricky 
let firstItems (|Ok|_|) list =
    let rec loop values = function
        | Ok (item, rest) -> loop (item :: values) rest
        | _ -> List.rev values
    loop [] list

let (|Even|_|) = function
    | item :: rest when (item % 2) = 0 -> Some (item, rest) | _ -> None

let test = [0; 2; 4; 5; 6] |> firstItems (|Even|_|)  // [0; 2; 4]
```
