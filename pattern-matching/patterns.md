# Patterns

## Généralités

_Patterns_ = règles pour détecter la structure de données en entrée

Utilisés abondamment en F♯

* Dans _match expression_, _let binding_ de valeurs et de paramètres de fonctions
* Très pratiques pour manipuler les types algébriques F♯ (tuple, record, union)
* Composables : supporte plusieurs niveaux d'imbrication
* Assemblables par ET/OU logiques
* Supporte les littéraux : `1.0`, `"test"`...

## Wildcard Pattern

Représenté par `_`, seul ou combiné avec un autre _pattern_

Toujours vrai → A placer en dernier dans une _match expression_

:warning: Toujours chercher en 1er à traiter exhaustivement/explicitement tous les cas\
&#x20;     Quand impossible, utiliser alors le `_`

```fsharp
match option with
| Some 1 -> ...
| _ -> ...              // ⚠️ Non exhaustif

match option with
| Some 1 -> ...
| Some _ | None -> ...  // 👌 \+ exhaustif
```

## Constant Pattern

Détecte constantes, `null` et littéraux de nombre, `char`, `string`, `enum`

```fsharp
[<Literal>]
let Three = 3   // Constante

let is123 num = // int -> bool
    match num with
    | 1 | 2 | Three -> true
    | _ -> false
```

☝ **Notes :**

* Le pattern de `Three` est aussi classé en tant que _Identifier Pattern_ 📍
* Pour le matching de `null`, on parle aussi de _Null Pattern_

## Identifier Pattern

Détecte les _cases_ d'un type union ainsi que leur éventuel contenu

```fsharp
type PersonName =
    | FirstOnly of string
    | LastOnly  of string
    | FirstLast of string * string

let classify personName =
    match personName with
    | FirstOnly _ -> "First name only"
    | LastOnly  _ -> "Last name only"
    | FirstLast _ -> "First and last names"
```

## Variable Pattern

Assigne la valeur détectée à une "variable" pour l'utiliser ensuite

Exemple : variables `firstName` et `lastName` ci-dessous

```fsharp
type PersonName =
    | FirstOnly of string
    | LastOnly  of string
    | FirstLast of string * string

let confirm personName =
    match personName with
    | FirstOnly (firstName) -> printf "May I call you %s?" firstName
    | LastOnly  (lastName) -> printf "Are you Mr. or Ms. %s?" lastName
    | FirstLast (firstName, lastName) -> printf "Are you %s %s?" firstName lastName
```

:warning: On ne peut pas lier à plusieurs reprises vers la même variable

```fsharp
let elementsAreEqualKo tuple =
    match tuple with
    | (x,x) -> true  // 💥 Error FS0038: 'x' est lié à deux reprises dans ce modèle
    | (_,_) -> false
```

**Solutions :** utiliser 2 variables puis vérifier l'égalité

```fsharp
// 1. Guard clause📍
let elementsAreEqualOk = function
    | (x,y) when x = y -> true
    | (_,_) -> false

// 2. Déconstruction
let elementsAreEqualOk' (x, y) = x = y
```

### Champs nommés de _cases_ d'union

Plusieurs possibilités :

1. Pattern "anonyme" du tuple complet\
   → Il faut déconstruire tous les champs du tuple.\
   → On utilise la virgule `,` pour séparer ces champs.
2. Pattern d'un seul champ par son nom → `Field = value`
3. Pattern de plusieurs champs par leur nom → `F1 = v1; F2 = v2`\
   :warning: Ne pas confondre avec l'option 1 : \
   → Le délimiteur est ici le **point-virgule** `;` et non la virgule `,` !

```fsharp
type Shape =
    | Rectangle of Height: int * Width: int
    | Circle of Radius: int

let describe shape =
    match shape with
    | Rectangle (0, _)                                              // (1)
    | Rectangle (Height = 0)            -> "Flat rectangle"         // (2)
    | Rectangle (Width = w; Height = h) -> $"Rectangle {w} × {h}"   // (3)
    | Circle radius                     -> $"Circle ∅ {2*radius}"
```

## Alias Pattern

`as` permet de nommer un élément dont le contenu est déconstruit

```fsharp
let (x, y) as coordinate = (1, 2)
printfn "%i %i %A" x y coordinate  // 1 2 (1, 2)
```

💡 Marche aussi dans les fonctions :

```fsharp
type Person = { Name: string; Age: int }

let acceptMajorOnly ({ Age = age } as person) =
    if age < 18 then None else Some person
```

## OR et AND Patterns

Permettent de combiner deux patterns _(nommés `P1` et `P2` ci-après)_

* `P1 | P2` → P1 ou P2. Ex : `Rectangle (0, _) | Rectangle (_, 0)`
* `P1 & P2` → P1 et P2. Utilisé surtout avec [active-patterns.md](active-patterns.md "mention")📍

💡 On peut utiliser la même variable _(`name` ci-dessous)_ dans 2 patterns :

```fsharp
type Upload = { Filename: string; Title: string option }

let titleOrFile ({ Title = Some name } | { Filename = name }) = name

titleOrFile { Filename = "Report.docx"; Title = None }            // Report.docx
titleOrFile { Filename = "Report.docx"; Title = Some "Report+" }  // "Report+"
```

## Parenthesized Pattern

Usage des parenthèses `()` pour grouper des patterns, pour gérer la précédence

```fsharp
type Shape = Circle of Radius: int | Square of Side: int

let countFlatShapes shapes =
    let rec loop rest count =
        match rest with
        | (Square (Side = 0) | (Circle (Radius = 0))) :: tail -> loop tail (count + 1) // ①
        | _ :: tail -> loop tail count
        | [] -> count
    loop shapes 0
```

☝ **Note :** la ligne ① ne compilerait sans faire `() :: tail`

:warning: Les parenthèses compliquent la lecture&#x20;

💡 Essayer de s'en passer quand c'est possible

```fsharp
let countFlatShapes shapes =
    let rec loop rest count =
        match rest with
        | Circle (Radius = 0) :: tail
        | Square (Side = 0) :: tail
          -> loop tail (count + 1)
        // [...]
```

## Construction Patterns

Reprennent syntaxe de construction d'un type pour le déconstruire

* Cons et List Patterns
* Array Pattern
* Tuple Pattern
* Record Pattern

### Cons et List Patterns

≃ Inverses de 2 types de construction d'une liste, avec même syntaxe

_Cons Pattern_ : `head :: tail` → décompose une liste _(avec >= 1 élément)_ en :

* _Head_ : 1er élément
* _Tail_ : autre liste avec le reste des éléments - peut être vide

_List Pattern_ : `[items]` → décompose une liste en 0..N éléments

* `[]` : liste vide
* `[x]` : liste avec 1 élément mis dans la variable `x`
* `[x; y]` : liste avec 2 éléments mis dans les variables `x` et `y`
* `[_; _]` : liste avec 2 éléments ignorés

💡 `x :: []` ≡ `[x]`, `x :: y :: []` ≡ `[x; y]`...

La _match expression_ par défaut combine les 2 patterns : → Une liste est soit vide `[]`, soit composée d'un item et du reste : `head :: tail`

Les fonctions récursives parcourant une liste utilise le pattern `[]` pour stopper la récursion :

```fsharp
let rec printList l =
    match l with
    | head :: tail ->
        printf "%d " head
        printList tail     // Récursion sur le reste
    | [] -> printfn ""     // Fin de récursion : liste parcourue entièrement
```

### Array Pattern

Syntaxe: `[| items |]` pour 0..N items entre `;`

```fsharp
let length vector =
    match vector with
    | [| x |] -> x
    | [| x; y |] -> sqrt (x*x + y*y)
    | [| x; y; z |] -> sqrt (x*x + y*y + z*z)
    | _ -> invalidArg (nameof vector) $"Vector with more than 4 dimensions not supported"
```

☝ Il n'existe pas de pattern pour les séquences, vu qu'elles sont _"lazy"_.

### Tuple Pattern

Syntaxe : `items` ou `(items)` pour 2..N items entre `,`

💡 Pratique pour pattern matcher plusieurs valeurs en même temps

```fsharp
type Color = Red | Blue
type Style = Background | Text

let css color style =
    match color, style with
    | Red, Background -> "background-color: red"
    | Red, Text -> "color: red"
    | Blue, Background -> "background-color: blue"
    | Blue, Text -> "color: blue"
```

### Record Pattern

Syntaxe : `{ Fields }` pour 1..N `Field = variable` entre `;`

* Pas obligé de spécifier tous les champs du Record
* En cas d'ambiguïté, qualifier le champ : `Record.Field`

💡 Marche aussi pour les paramètres d'une fonction :

```fsharp
type Person = { Name: string; Age: int }

let displayMajority { Age = age; Name = name } =
    if age >= 18
    then printfn "%s is major" name
    else printfn "%s is minor" name

let john = { Name = "John"; Age = 25 }
displayMajority john // John is major
```

:warning: **Rappel :** il n'y a pas de pattern pour les _Records_ anonymes !

```fsharp
type Person = { Name: string; Age: int }

let john = { Name = "John"; Age = 25 }
let { Name = name } = john  // 👌 val name : string = "John"

let john' = {| john with Civility = "Mister" |}
let {| Name = name' |} = john'  // 💥
```

## Type Test Pattern

**Syntaxe :** `my-object :? sub-type`&#x20;

Renvoie un `bool` \
→ `is` C♯ (`my-object is sub-type`)

**Usage :** avec une hiérarchie de types

```fsharp
open System.Windows.Forms

let RegisterControl (control: Control) =
    match control with
    | :? Button as button -> button.Text <- "Registered."
    | :? CheckBox as checkbox -> checkbox.Text <- "Registered."
    | :? Windows -> invalidArg (nameof control) "Window cannot be registered"
    | _ -> ()
```

:point\_up: **Remarque :** l'opérateur `:?` n'est pas conçu pour vérifier des évidences car il adopte alors un comportement déroutant : \
&#xNAN;_(Exemple tiré de cette_ [_question stackoverflow_](https://stackoverflow.com/q/70845434/8634147)_)_

```fsharp
type Car = interface end

type Mercedes() =
    interface Car

let merc = Mercedes ()

merc :? Mercedes        // true with warning FS0067: Ce test de type ... conviendra toujours.
box merc :? Mercedes    // true
merc :? Car             // error FS0193: Incompatibilité de contrainte de type.
box merc :? Car         // true
```

### Bloc `try`/`with`

On rencontre fréquemment ce pattern dans les blocs `try`/`with` :

```fsharp
try
    printfn "Difference: %i" (42 / 0)
with
| :? DivideByZeroException as x -> 
    printfn "Fail! %s" x.Message
| :? TimeoutException -> 
    printfn "Fail! Took too long"
```

### Boxing

Le _Type Test Pattern_ ne marche qu'avec des types références.&#x20;

→ Pour un type valeur ou inconnu, il faut le convertir en objet _(a.k.a boxing)_

```fsharp
let isIntKo = function :? int -> true | _ -> false
// 💥 Error FS0008: test de type au moment de l'exécution du type 'a en int...

let isInt x =
    match box x with
    | :? int -> true
    | _ -> false
```
