# Génériques

## Génériques

Fonctions et types peuvent être génériques, avec + de flexibilité qu'en C♯.

Par défaut, généricité **implicite**

* Inférée
* Voire généralisée, grâce à « généralisation automatique »

Sinon, généricité peut être explicite ou résolue statiquement.

:warning: Notations différentes _(avant F# 7)_ :

* `'T` : paramètre de type générique
* `^T` : paramètre de type résolu statiquement _(SRTP)_
* Depuis F# 7, `'T`  peut désigner l'un ou l'autre

## Généricité implicite

Le compilateur est capable d'inférer qu'une fonction est générique.\
→ Simplifie le code

```fsharp
module ListHelper =
    let singleton x = [x]
    // val singleton : x:'a -> 'a list

    let couple x y = [x; y]
    // val couple : x:'a -> y:'a -> 'a list
```

👉 **Explications :**

* `singleton` : `x` est juste mis dans une liste générique → son type est donc quelconque
* `couple` : ses 2 arguments `x` et `y` doivent être du même type pour pouvoir être dans une liste

☝️ Les noms des types génériques inférés rendent parfois une signature de fonction difficile à comprendre. Ne pas hésiter à ajouter alors des annotations de types plus explicites.\
Exemple: `Result<'a, 'b>` → `Result<'ok, 'err>`

## Généricité explicite

```fsharp
let print2 x y = printfn "%A, %A" x y
// val print2 : x:'a -> y:'b -> unit
```

→ Inférence de la généricité de `x` et `y` 👍

❓ **Comment indiquer que `x` et `y` doivent avoir le même type ?**

→ Besoin de l'indiquer explicitement :

```fsharp
let print2<'T> (x: 'T) (y: 'T) = printfn "%A, %A" x y
// val print2 : x:'T -> y:'T -> unit
```

### Généricité explicite - Forme inline

💡 **Astuce :** la convention en `'x` permet ici d'être + concis :

```fsharp
// AVANT
let print2<'T> (x: 'T) (y: 'T) = printfn "%A, %A" x y

// APRES
let print2 (x: 'T) (y: 'T) = printfn "%A, %A" x y
```

### Généricité explicite - Type

La définition des types génériques est explicite :

```fsharp
type Pair = { Item1: 'T ; Item2: 'T }
// 💥                ~~          ~~
// Error FS0039: Le paramètre de type `'T` n'est pas défini.

// ✅ Records et unions avec 1 ou 2 paramètres de type
type Pair<'T> = { Item1: 'T; Item2: 'T }

type Tuple<'T, 'U> = { Item1: 'T; Item2: 'U }

type Option<'T> = None | Some of 'T

type Result<'TOk, 'TErr> =
    | Ok of 'TOk
    | Error of 'TErr
```

## Généricité ignorée

Le _wildcard_ `_` permet de remplacer un paramètre de type ignoré :

```fsharp
let printSequence (sequence: seq<'T>) = sequence |> Seq.iteri (printfn "%i: %A")
// Versus
let printSequence (sequence: seq<_>) = ...
```

Encore + utile avec type flexible📍[types-flexibles.md](types-flexibles.md "mention") :

```fsharp
let tap action (sequence: 'TSeq when 'TSeq :> seq<_>) =
    sequence |> Seq.iteri action
    sequence
// action:(int -> 'a -> unit) -> sequence:'TSeq -> 'TSeq when 'TSeq :> seq<'a>

// Versus
let tap action (sequence: #seq<_>) = ...
```

## SRTP

F♯ propose deux catégories de types de paramètre :

* `'X` : type de paramètre générique comme en C# : le type concret est défini au runtime.
* `^X` : type de paramètre résolu statiquement : le type concret est défini lors de la compilation.

☝ **SRTP :** abréviation fréquente de _Statically Resolved Type Parameter_

### SRTP : pourquoi ?

Sans SRTP :

```fsharp
let add x y = x + y
// val add : x:int -> y:int -> int
```

→ Inférence du type `int` pour `x` et `y`, sans généralisation (aux `float` par ex.) !

Avec SRTP, de pair avec fonction `inline` :

```fsharp
let inline add x y = x + y
// val inline add : x: ^a -> y: ^b -> ^c
//    when ( ^a or ^b ) : (static member (+) : ^a * ^b -> ^c)
//    ☝ Contrainte de membre 📍

let x = add 1 2       // ✅ val x: int = 3
let y = add 1.0 2.0   // ✅ val y: float = 3.0
```

### SRTP : duck typing

On peut utiliser les SRTP pour appeler un membre en même temps que l'on contraint son existence dans le type associé.

#### Duck typing d'une propriété

```fsharp
// -- Déclaration --
let inline Length x = (^a : (member Length : _) x)
// FSI: val inline Length: x:  ^a -> 'a0 when  ^a: (member get_Length:  ^a -> 'a0)

// -- Usages --
let textLength = "text" |> Length
// FSI: val textLength: int = 4

let listLength = [ 1..10 ] |> Length
// FSI: val listLength: int = 10
```

💡 Dans vscode avec Ionide, l'IntelliSense est plus friendly :&#x20;

```
val inline Length:
   x: 'a (requires member Length )
   -> 'a
```

#### Duck typing d'une méthode

Déclaration :&#x20;

```fsharp
let inline ToString format x = (^x : (member ToString : ^format -> string) (x, format))
(*
val inline ToString:
  format:  ^format -> x:  ^x -> string
    when  ^x: (member ToString:  ^x *  ^format -> string)
*)
```

💡 IntelliSense dans VsCode + Ionide :&#x20;

```
val inline ToString:
   format: 'format ->
   x     : 'x      (requires member ToString )
        -> string
```

Usages (exemples) :&#x20;

```fsharp
1 |> ToString "c" // "1,00 €", or "£1.00"... (selon paramètres locaux)

open System
DateTime.Now |> ToString "s" // "2023-02-24T14:04:37"
DateTime.Now |> ToString Globalization.CultureInfo.CurrentCulture // "24/02/2023 14:04:37"
```

#### Duck typing : compléments

💡 Plus d'informations et de conseils sur le duck typing dans l'article ci-dessous duquel sont extraits certains exemples :&#x20;

{% embed url="https://www.compositional-it.com/news-blog/static-duck-typing-in-f/" %}

Un autre exemple de SRTP est détaillé dans cette [réponse sous StackOverflow](https://stackoverflow.com/a/72088241/8634147) à propos du type `Functor` dans la librairie [FSharpPlus](https://github.com/fsprojects/FSharpPlus).

### SRTP en F♯ 7.0

* Plusieurs améliorations ont été introduites en [F# 7.0](https://devblogs.microsoft.com/dotnet/announcing-fsharp-7/#making-working-with-srtps-easier) pour améliorer la syntaxe des SRTP.
* Jusqu'en F♯ 6.0, il fallait mettre un espace entre le chevron ouvrant et le SRTP.
* Depuis F♯ 7.0 (novembre 2022), cela n'est pas nécessaire. Cela permet d'être uniforme avec les types génériques :

```fsharp
// Avant F# 7.0
type C< ^T> = class end
let f< ^T> (x:^T) = ()

// Depuis F# 7.0
type C<^T> = class end
let f<^T> (x:^T) = ()
```

### SRTP : recommandation

* Plusieurs améliorations ont été introduites en [F# 7.0](https://devblogs.microsoft.com/dotnet/announcing-fsharp-7/#making-working-with-srtps-easier) pour améliorer la syntaxe des SRTP. Elles rendent le code plus lisible.
* Cependant, la syntaxe reste encore un peu difficile à lire, à dessein : pour encourager les solutions alternatives.
* De plus, on ne peut pas savoir à l'avance tous les types compatibles avec une fonction avec SRTP.
* Enfin, cela peut ralentir beaucoup la compilation. C'est un des problèmes remontés par des utilisateurs de la librairie FSharpPlus.

> 👉 Les SRTP sont à utiliser avec parcimonie car leur syntaxe est difficile à lire.

## Contraintes

Les contraintes sur paramètres de type en F♯ reposent sur le même principe qu'en C♯, avec quelques différences :&#x20;

| Contrainte  | Syntaxe F♯                     | Syntaxe C♯                      |
| ----------- | ------------------------------ | ------------------------------- |
| Mots clés   | `when xxx and yyy`             | `where xxx, yyy`                |
| Emplacement | Juste après type :             | Fin de ligne :                  |
|             | `fn (arg: 'T when 'T ...)`     | `Method<T>(arg: T) where T ...` |
|             | Dans chevrons :                |                                 |
|             | `fn<'T when 'T ...> (arg: 'T)` |                                 |

### Vue d'ensemble

| Contrainte              | Syntaxe F♯               | Syntaxe C♯                 |
| ----------------------- | ------------------------ | -------------------------- |
| Type de base            | `'T :> my-base`          | `T : my-base`              |
| Type valeur             | `'T : struct`            | `T : struct`               |
| Type référence          | `'T : not struct`        | `T : class`                |
| Type référence nullable | `'T : null`              | `T : class?`               |
| Constructeur sans param | `'T : (new: unit -> 'T)` | `T : new()`                |
| Énumération             | `'T : enum<my-enum>`     | `T : System.Enum`          |
| Comparaison             | `'T : comparison`        | ≃ `T : System.IComparable` |
| Égalité                 | `'T : equality`          | _(pas nécessaire)_         |
| Membre explicite        | `^T : member-signature`  | _(pas d'équivalent)_       |

### Contraintes de type

Pour forcer le type de base : classe mère ou interface

```fsharp
let check<'error' when 'error' :> System.Exception> condition (error: 'error') =
    if not condition then raise error
```

→ Équivalent en C♯ :

```csharp
static void check<TError>(bool condition, TError error) where TError : System.Exception
{
    if (!condition) throw error;
}
```

💡 Syntaxe alternative : `let check condition (error: #System.Exception)` \
→ Cf. [types-flexibles.md](types-flexibles.md "mention") 📍

### Contrainte de nullabilité

Exemple de fonction avec une telle contrainte : la fonction `Option.ofObj` ([type-option.md](../types-monadiques/type-option.md "mention")📍) prend en entrée une valeur nullable venant "de l'extérieur" et la convertit en type `Option` plus sûr à utiliser.

```fsharp
module Option =
    let ofObj value = // 'a -> 'a option when 'a: null
        match value with
        | null -> None
        | _ -> Some value
```

👉 Le paramètre générique `'a` comporte une contrainte de nullabilité : `when 'a: null`.

:warning: **Attention :** ne pas confondre avec le type `System.Nullable<T>` qui est un type valeur alors que la contrainte de nullabilité ne s'applique à qu'un type référence.

☝️ **Note :** cette contrainte ne s'applique pas aux types F♯ (Tuple, Record, Union) qui sont bien des types référence mais qui ne peuvent pas être instanciés `null` dans les cas d'usage standard. Cependant, lors d'une interop avec une librairie .NET telle que le micro-ORM Dapper, on peut obtenir une valeur `null` à la barbe du compilateur F♯. Pour retomber sur nos pieds, on peut :&#x20;

* Soit décorer le type avec un attribut `AllowNullLiteral`, mais on perd alors la sécurité des types non nullables.
* Soit utiliser la fonction `Unchecked.defaultof` pour tester cette nullité sans perdre en sécurité dans le reste de la codebase :&#x20;

```fsharp
module Option =
    let private isHiddenNull (arg: 'a when 'a: not struct) =
        arg = Unchecked.defaultof<'a> // ☝️ Ajoute contrainte `'when a: equality`

    let ofHiddenNullable value =
        if value |> isHiddenNull
        then None
        else Some value
```

### Contrainte d'enum

```fsharp
open System

let getValues<'T when 'T : enum<int>>() =
    Enum.GetValues(typeof<'T>) :?> 'T array

type ColorEnum = Red = 1 | Blue = 2
type ColorUnion = Red | Blue

let x = getValues<ColorEnum>()   // [| Red; Blue |]
let y = getValues<ColorUnion>()  // 💥 Exception ou erreur de compilation (1)
```

(1) La contrainte `when 'T : enum<int>` permet :&#x20;

* D'éviter la `ArgumentException` au runtime _(Type provided must be an Enum)_&#x20;
* Au profit d'une erreur dès la compilation _(The type 'ColorUnion' is not an enum)_

### Contrainte de comparaison

Syntaxe : `'T : comparison`

Indique que le type `'T` doit :&#x20;

* soit implémenter `IComparable` (1)&#x20;
* soit être un collection d'éléments comparables (2)

☝ **Notes :**

1. `'T : comparison` > `'T : IComparable` ❗
2. `'T : comparison` ≠ `'T : IComparable<'T>` ❗
3. Pratique pour méthodes génériques `compare` ou `sort` 💡

**Exemple :**&#x20;

```fsharp
let compare (x: 'T) (y: 'T when 'T : comparison) =
    if   x < y then -1
    elif x > y then +1
    else 0

// Comparaison de nombres et de chaînes
let x = compare 1.0 2.0  // -1
let y = compare "a" "A"  // +1

// Comparaison de listes d'entier
let z = compare [ 1; 2; 3 ] [ 2; 3; 1 ]  // -1

// Comparaison de listes de fonctions
let a = compare [ id; ignore ] [ id; ignore ]
// 💥             ~~
// error FS0001: Le type '('a -> 'a)' ne prend pas en charge la contrainte 'comparison'.
// Par exemple, il ne prend pas en charge l'interface 'System.IComparable'
```

### Contrainte de membre explicite

> **Pb :** Comment indiquer qu'un objet doit disposer d'un certain membre ?

• Manière classique en .NET : typage nominal \
→ Contrainte spécifiant type de base (interface ou classe parent)

• Alternative en F♯ : typage structurel _(a.k.a duck-typing des langages dynamiques)_ \
→ Contrainte de membre explicite \
→ Utilisée avec les SRTP _(statically resolved type parameter)_

```fsharp
let inline add (value1 : ^T when ^T : (static member (+) : ^T * ^T -> ^T), value2: ^T) =
    value1 + value2

let x = add (1, 2)
// val x : int = 3
let y = add (1.0, 2.0)
// val y : float = 3.0
```

⚖️ Pour et contre :

* 👍 Permet de rendre code générique pour types hétérogènes
* 👎 Difficile à lire, à maintenir. Ralentit la compilation
* 👉 À utiliser dans une librairie, pas pour modéliser un domaine
