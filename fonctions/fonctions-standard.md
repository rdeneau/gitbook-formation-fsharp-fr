# Fonctions standard

La librairie `FSharp.Core.dll` est automatiquement importée dans un projet F# ou en console FSI. Elle fournit les opérateurs et fonctions standard ([doc](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-core-operators.html)). En voici les fonctions principales :

## Conversion

`box`, `tryUnbox`, `unbox` : _boxing_ et (tentative de) _unboxing_

```fsharp
let i = 123     // val i : int = 123
let o = box i   // val o : obj = 123

let i2: int = unbox o     // val i3 : int = 123
let i3 = unbox<int> o     // val i2 : int = 123

let ok = tryUnbox<int> o      // val ok : int option = Some 123
let ko = tryUnbox<string> o   // val ko : string option = None

unbox<float> o  //  💥 System.InvalidCastException
```

`byte`, `char`, `decimal`, `float`, `int`, `string` : conversion en `byte`, `char`, ...

```fsharp
let c_ko = char "ab"  // 💥 System.FormatException
let c1 = char "c"     // val c1 : char = 'c'
let c2 = char 32      // val c2 : char = ' '

let i_ko = int " "  // 💥 System.FormatException
let i1 = int "30"   // val i1 : int = 30
let i2 = int ' '    // val i2 : int = 32
let i3 = int 12.6   // val i3 : int = 12
```

`enum<'TEnum>` : conversion en l'enum spécifié \
→ Cf. [enums.md](../types-composites/enums.md "mention") > [#conversion](../types-composites/enums.md#conversion "mention")

## Reflection

`nameof`, `typeof` :

```fsharp
let myVariable = None
let myVariableName = nameof myVariable
// val myVariableName : string = "myVariable"

let t1 = typeof<string option>
// val t1 : System.Type = Microsoft.FSharp.Core.FSharpOption`1[System.String]

```

`typedefof` est intéressant avec un type générique :&#x20;

```fsharp
let t2 = typedefof<string option>
// val t2 : System.Type = Microsoft.FSharp.Core.FSharpOption`1[T]

// t2 est équivalent à :
let t2' = typedefof<_ option>
let t2'' = t1.GetGenericTypeDefinition ()

// t1 est équivalent à :
let t1' = t2.MakeGenericType(typeof<string>)
```

## Math

* `abs`, `sign` : valeur absolue, signe (-1 si < 0...)
* `(a)cos(h)`, `(a)sin`, `(a)tan` : (co)sinus/tangente (inverse/hyperbolique)
* `ceil`, `floor`, `round` : arrondi (inf, sup)
* `exp`, `log`, `log10` : exponentielle, logarithme...
* `pown x (n: int)` : _power_ = `x` à la puissance `n`
* `sqrt` : _square root_ / racine carrée

## Identité `id`

Définition `let id x = x`&#x20;

Signature : `(x: 'T) -> 'T` \
→ Fonction à un seul paramètre d'entrée \
→ Qui ne fait que renvoyer ce paramètre

Pourquoi une telle fonction ❓ \
→ Nom `id` = abréviation de `identity` \
→ Zéro / Élément neutre de la composition des fonctions

<table><thead><tr><th>Opération</th><th width="186.33333333333331">Identité</th><th>Exemple</th></tr></thead><tbody><tr><td>Addition <code>+</code></td><td><code>0</code></td><td><code>0 + 5</code> ≡ <code>5 + 0</code> ≡ <code>5</code></td></tr><tr><td>Multiplication <code>*</code></td><td><code>1</code></td><td><code>1 * 5</code> ≡ <code>5 * 1</code> ≡ <code>5</code></td></tr><tr><td>Composition <code>>></code></td><td><code>id</code></td><td><code>id >> fn</code> ≡ <code>fn >> id</code> ≡ <code>fn</code></td></tr></tbody></table>

#### Cas d'utilisation de `id`

Avec une _high-order function_ faisant 2 choses : \
• 1 opération \
• 1 mapping de valeur via param `'T -> 'U`

Ex : `List.collect fn list` = flatten + mapping

Comment faire juste l'opération et pas de mapping ?

* `list |> List.collect (fun x -> x)` 👎
* `list |> List.collect id` 👍
* ☝ Meilleure alternative : `List.concat list` 💯

## Autres

* `compare a b : int`: renvoie -1 si a < b, 0 si =, 1 si >
* `hash` : calcule le hash (`HashCode`)
* `max`, `min` : maximum et minimum de 2 valeurs comparables
* `ignore` : pour "avaler" une valeur et obtenir `unit`

## Extras

On utilise couramment les fonctions suivantes, non standard donc en les recodant ou en utilisant celles du module `Prelude` ([source](https://github.com/fsprojects/FSharpx.Extras/blob/master/src/FSharpx.Extras/Prelude.fs)) du package NuGet [FSharpx.Extras](https://github.com/fsprojects/FSharpx.Extras).

### Flip

`flip`, `flip3` et `flip4` permettent d'inverser l'ordre des paramètres d'une fonction, de sorte que le dernier devienne le premier.

```fsharp
let inline flip f a b = f b a
let inline flip3 f a b c = f c a b
let inline flip4 f a b c d = f d a b c

// Exemple: inversion de defaultArg (arg: option<'T> -> defaultValue: 'T -> 'T)
let inline defaultValue value option = defaultArg option value
let sansFlip       = Some 1 |> (fun x -> defaultArg x 0)
let avecFlipInline = Some 1 |> (flip defaultArg 0)
let avecFlipManuel = Some 1 |> (defaultValue 0)
```

### Curry

`curry` et `curry3` permettent de curryfier des fonctions tuplifiées de 2 ou 3 paramètres.\
`uncurry` et `uncurry3` font l'inverse.

```fsharp
let inline curry f a b = f (a, b)
let inline uncurry f (a, b) = f a b

let inline curry3 f a b c = f (a, b, c)
let inline uncurry3 f (a, b, c) = f a b c

// Exemple: DateTime(year, month day)
let dateIn2022 = curry3 System.DateTime 2022
let d = dateIn2022 1 31  // val d: System.DateTime = 31/01/2022 00:00:00
```

### Konst

`konst` est une fonction _sink_ par rapport à son deuxième argument, ignoré pour renvoyer toujours le premier argument, la constante.\
→ Exemple : générer un prédicat toujours vrai, pour le passer à une fonction d'ordre supérieur telle que `filterMap` ci-dessous :&#x20;

```fsharp
let inline konst a _ = a

// Exemple :
let inline filterMap predicate mapper list =
    list
    |> List.filter predicate
    |> List.map mapper

let onlyMappingF = [1..3] |> filterMap (fun _ -> true) ((+) 1)  // [2; 3; 4]
let onlyMappingK = [1..3] |> filterMap (konst true) ((+) 1)     // [2; 3; 4]
let onlyFilter   = [1..3] |> filterMap ((=) 2) id               // [2]
```

💡 `ignore` ≅ `konst ()`

### Tee

`tee` (appelée également `tap`) permet d'appliquer une valeur à une fonction et de renvoyer cette valeur, en ignorant l'éventuel retour de la fonction. C'est utile avec une fonction à effet de bord, par exemple pour logguer une valeur intermédiaire dans un pipeline :&#x20;

```fsharp
// fn: ('a -> 'b) -> x : 'a -> 'a
let inline tee fn x = fn x |> ignore; x

// Exemple
let test =
    [1..10]
    |> List.map ((*) 3)
    |> tee (printfn "[Debug] After *3: %A")
    |> List.map ((+) 1)
    |> tee (printfn "[Debug] After +1: %A")
    |> List.filter (fun x -> x % 4 = 0)
// [Debug] After *3: [3; 6; 9; 12; 15; 18; 21; 24; 27; 30]
// [Debug] After +1: [4; 7; 10; 13; 16; 19; 22; 25; 28; 31]
// val test: int list = [4; 16; 28]
```

### Parse

Les types primitifs `Boolean`, `Byte`, `SByte`, `UInt16`, `Int16`, `UInt32`, `Int32`, `UInt64`, `Int64`, `Decimal`, `Single`, `Double`, `DateTime`, `DateTimeOffset` sont étendus avec la méthode statique `parse` qui encapsule la classique méthode statique .NET `TryParse`. L'objectif est de renvoyer une `Option` (📍 [type-option.md](../types-monadiques/type-option.md "mention")) plutôt qu'un `bool` et une variable `out` (📍 [Broken link](broken-reference "mention")).

```fsharp
// https://github.com/fsprojects/FSharpx.Extras/blob/master/src/FSharpx.Extras/Prelude.fs#L89

open System
open System.Globalization

let inline toOption x =
    match x with
    | true,  v -> Some v
    | false, _ -> None

let inline tryWith f x = f x |> toOption

type Int32 with
    static member parseWithOptions style provider (x: string) =
        Int32.TryParse(x, style, provider) |> toOption

    static member parse x =
        Int32.parseWithOptions NumberStyles.Integer CultureInfo.InvariantCulture x

// Utilisation
let test1a = Int32.TryParse "1"  // val test1a : bool * int = (true, 1)
let test1b = Int32.parse "1"     // val test1b : int option = Some 1

let test2a = Int32.TryParse "z"  // val test2a : bool * int = (false, 0)
let test2b = Int32.parse "z"     // val test2b : int option = None
```

### String

Le module `String` encapsule les méthodes d'instance de string dans des fonctions pour être utilisées plus facilement dans les pipelines. En voici quelques exemples :

```fsharp
open System

module String =
    let inline startsWith (value: string) (s: string) = s.StartsWith(value)
    let inline contains   (value: string) (s: string) = s.Contains(value)
    let inline endsWith   (value: string) (s: string) = s.EndsWith(value)

    let inline insert startIndex value (s: string) = s.Insert(startIndex, value)

    let inline padLeft  totalWidth (s: string) = s.PadLeft(totalWidth)
    let inline padRight totalWidth (s: string) = s.PadRight(totalWidth)
    let inline padLeft'  totalWidth paddingChar (s: string) = s.PadLeft(totalWidth, paddingChar)
    let inline padRight' totalWidth paddingChar (s: string) = s.PadRight(totalWidth, paddingChar)

    let inline replace (oldChar: char) (newChar: char) (s: string) = s.Replace(oldChar, newChar)
    let inline replace' (oldValue: string) (newValue: string) (s: string) = s.Replace(oldValue, newValue)

    let inline toLower (s: string) = s.ToLower()
    let inline toLowerInvariant (s: string) = s.ToLowerInvariant()
    let inline toUpper (s: string) = s.ToUpper()
    let inline toUpperInvariant (s: string) = s.ToUpperInvariant()

    let inline trim (s: string) = s.Trim()
```

☝ **Note :** les fonctions surchargées auraient bénéficier d'un nom plus parlant, par exemple :&#x20;

```fsharp
open System
module String =
    let inline replaceChar (oldChar: char) (newChar: char) (s: string)   = s.Replace(oldChar,  newChar)
    let inline replace (oldValue: string) (newValue: string) (s: string) = s.Replace(oldValue, newValue)
```
