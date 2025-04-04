# Enums

## Déclaration

Ensemble de constantes nommées dont la valeur est de type entier :\
→ Contrairement au C♯, il faut définir la valeur de tous les membres de l'enum :&#x20;

```fsharp
type ColorN =
    | Red   = 1
    | Green = 2
    | Blue  = 3
```

☝️ Noter la différence de syntaxe avec un type union "enum-like" ([#style-enum](unions.md#style-enum "mention")) :&#x20;

```fsharp
type Color = Red | Green | Blue
```

### Type sous-jacent

* Contrairement au C♯, il n'y a pas de type sous-jacent par défaut en F♯.
* Le type sous-jacent est défini au moyen des littéraux définissant les valeurs des membres :&#x20;
  * `1`, `2`, `3` → `int`
  * `1uy`, `2uy`, `3uy` → `byte`
  * Etc. - cf. [Literals](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/literals)

Corolaire : il faut utiliser le même type pour tous les membres, sinon cela ne compile pas !

```fsharp
type ColorN =
    | Red   = 1
    | Green = 2
    | Blue  = 3uy
// 💥 ~~~~~~~~~~~
// Cette expression était censée avoir le type `int` 
// mais elle a ici le type `byte`
```

### Char

Le type `char` peut être utilisé comme type sous-jacent :&#x20;

```fsharp
type AnswerChar = Yes='Y' | No='N'

// L'équivalent ne marche pas avec 'string'
type AnswerChar = Yes="Y" | No="N"
// 💥 ~~~~~~~~~~ Littéraux énumérés doivent être de type 'int'...
```

### Casse

Autre différence avec les types union, les membres peuvent être en camelCase :&#x20;

```fsharp
type File =
    | a = 'a'
    | b = 'b'
    | c = 'c'
```

## Usage

:warning: Contrairement aux unions, l'emploi d'un membre _(a.k.a littéral)_ d'enum est forcément **qualifié**

```fsharp
let answerKo = Yes            // 💥 Error FS0039
//             ~~~ La valeur ou le constructeur 'Yes' n'est pas défini.
let answer = AnswerChar.Yes   // 👌 OK
```

Cast via helpers `int` et `enum` (mais pas `char`) :

```fsharp
let redValue = int ColorN.Red         // enum -> int
let redAgain = enum<ColorN> redValue  // int -> enum via type générique
let red: ColorN = enum redValue       // int -> enum via annotation

// ⚠️ Ne marche pas avec char enum
let ko = char AnswerChar.No   // 💥 Error FS0001
let no: AnswerChar = enum 'N' // 💥 Error FS0001
```

## Pattern matching

:warning: Contrairement aux unions, le _pattern matching_ **n'est pas exhaustif**

```fsharp
type ColorN = Red=1 | Green=2 | Blue=3

let toHex color =
    match color with
    | ColorN.Red   -> "FF0000"
    | ColorN.Green -> "00FF00"
    | ColorN.Blue  -> "0000FF"
    // ⚠️ Warning FS0104: Les enums peuvent accepter des valeurs en dehors des cas connus.
    // Par exemple, la valeur 'enum<ColorN> (0)' peut indiquer un cas non traité...

    // 💡 Pour enlever le warning, il faut ajouter un pattern générique
    | _ -> invalidArg (nameof color) $"Color {color} not supported"
```

## Valeurs

On peut utiliser la méthode `System.Enum.GetValues()` pour obtenir la liste des membres d'une `enum`. Par contre, le type de retour est faiblement typé : `Array` (tableau non générique). \
→ Il suffit d'encapsuler cette méthode dans une fonction _helper_ telle que :&#x20;

```fsharp
let enumValues<'a> () =
    Enum.GetValues(typeof<'a>)
    :?> ('a array)
    |> Array.toList

let allPermissions = enumValues<PermissionFlags>()
// val allPermissions: PermissionFlags list = [Read; Write; Execute]
```

💡 Voir également  [#extras](enums.md#extras "mention")

## Flags

Même principe qu'en C♯ où l'on choisit comme valeurs des multiples de 2 afin de pouvoir les combiner :

```fsharp
open System

[<Flags>]
type PermissionFlags =
    | Read    = 1
    | Write   = 2
    | Execute = 4

let permissions = PermissionFlags.Read ||| PermissionFlags.Write
// val permissions: PermissionFlags = Read, Write

let canRead    = permissions.HasFlag PermissionFlags.Read    // true
let canWrite   = permissions.HasFlag PermissionFlags.Write   // true
let canExecute = permissions.HasFlag PermissionFlags.Execute // false
```

💡 **Notes :**&#x20;

* L'attribut `System.FlagsAttribute` est facultatif mais permet d'avoir un code plus explicite. En outre, il améliore le rendu des combinaisons de flags : dans l'exemple précédent, la valeur affichée de `permissions` est `Read, Write`. Sans l'attribut `Flags`, cela aurait afficher `3`.
* Opérateur OU binaire `|||` _(`|` en C♯)_ pour combiner des flags

💡 **Astuce :** utiliser la notation binaire pour la valeur des flags : &#x20;

```fsharp
[<Flags>]
type PermissionFlags =
    | Read    = 0b001
    | Write   = 0b010
    | Execute = 0b100
```

### Combinaisons

En C♯, l'on peut directement combiner des flags, comme `ReadWrite` et `All` ci-dessous :&#x20;

```csharp
[Flags]
public enum PermissionFlags 
{
    Read    = 0b001,
    Write   = 0b010,
    Execute = 0b100,

    ReadWrite = Read | Write,
    All = Read | Write | Execute
}
```

Ce n'est pas possible en F♯ mais 2 alternatives sont possibles :&#x20;

1\) Procéder manuellement aux combinaisons binaires : ce n'est pas dur à faire mais cela nuit à la visibilité car cela demande un peu de réflexion pour retrouver les flags initiaux. On notera aussi que ces combinaisons font partie intégrante de l'enum.

```fsharp
[<Flags>]
type PermissionFlags =
    | Read      = 0b001
    | Write     = 0b010
    | Execute   = 0b100
    | ReadWrite = 0b011 // ❌ Moins explicite que Read | Write
    | All       = 0b111 // ❌ Moins explicite que Read | Write | Execute 

let all = PermissionFlags.All
// val all: PermissionFlags = All

let all' = PermissionFlags.Read ||| PermissionFlags.Write
// val all': PermissionFlags = ReadWrite
// ✔️ Recombinaison automatique du compilateur reconnaissant "All"
```

2\) Utiliser un module compagnon : pour rendre les combinaisons explicites dans le code mais non reconnues / recombinées par le compilateur.

```fsharp
[<System.Flags>]
type Spacing =
    | Left   = 0b0001
    | Right  = 0b0010
    | Top    = 0b0100
    | Bottom = 0b1000

[<RequireQualifiedAccess>]
module Spacing =
    let Horizontal = Spacing.Left ||| Spacing.Right  // ✔️ Human-friendly
    let Vertical = Spacing.Top ||| Spacing.Bottom
    let All = Horizontal ||| Vertical

let horizontal = Spacing.Horizontal
// val horizontal: Spacing = Left, Right
// ❌ Not "Horizontal"
```

☝ **Note :** la méthode `HasFlag` a un comportement différent selon l'option utilisée. C'est mis en valeur dans l'exemple ci-dessous définissant les 2 helpers `Enum.values` déjà vu plus haut et `Enum.flags` qui décompose une valeur d'enum en ses flags élémentaires.

```fsharp
[<RequireQualifiedAccess>]
module Enum =
    let values<'enum when 'enum :> System.Enum> =
        System.Enum.GetValues(typeof<'enum>)
        :?> 'enum array
        |> Array.toList

    let flags<'enum when 'enum :> System.Enum> (enumValue: 'enum) =
        values<'enum>
        |> List.filter (enumValue.HasFlag)

let flagsInline = Enum.flags PermissionFlags.All
// val flagsInline: PermissionFlags list = [Read; Write; ReadWrite; Execute; All]
// 👉 Includes "ReadWrite" and "All". Could even include a "None = 0" ❗

let flagsCompanion = Enum.flags Spacing.All
// val flagsCompanion: Spacing list = [Left; Right; Top; Bottom]
// 👍 Only includes core flags
```

## Enum _vs_ Union

| Type               | Enum             | Union                |
| ------------------ | ---------------- | -------------------- |
| Type sous-jacent   | Entières ou char | Quelconques          |
| Qualification      | Obligatoire      | Qu'en cas de conflit |
| Matching exhaustif | ❌ Non            | ✅ Oui                |
| PascalCase         | ✅ Oui            | ✅ Oui                |
| camelCase          | ✅ Oui            | ❌ Non                |

☝ **Recommandation :**

* Préférer une Union dans la majorité des cas
* Choisir une Enum pour :
  * Interop .NET
  * Besoin de lier des données de type `int`

## Conversion

`enum<'enum>` : permet de convertir une valeur entière en l'enum `'enum` spécifiée

```fsharp
type AnswerNum =
    | Yes = 1
    | No  = 0

let y1 = enum<AnswerNum> 1  // val y1 : AnswerNum = Yes
let y2: AnswerNum = enum 0  // val y2 : AnswerNum = No
```

`int` permet la conversion inverse pour récupérer la valeur sous-jacente d'un membre d'une enum

```fsharp
let yesNum = int AnswerNum.Yes  // val yesNum : int = 1
```

### Char enum

Pour les enums dont le type sous-jacent est `char`, les fonctions `enum`, `int` et `char` ne marchent pas :&#x20;

```fsharp
type AnswerChar =
    | Yes = 'Y'
    | No  = 'N'

let no_ko: AnswerChar = enum 'N' // 💥 Le type 'int32' ne correspond pas au type 'char'

let y1_ko = int AnswerChar.Yes   // 💥 Le type 'AnswerChar' ne prend pas en charge une conversion vers le type 'int'
let y2_ko = char AnswerChar.Yes  // 💥 Le type 'AnswerChar' ne prend pas en charge une conversion vers le type 'char'
```

Il faut alors utiliser le module `LanguagePrimitives` :&#x20;

```fsharp
let no_ok: AnswerChar = LanguagePrimitives.EnumOfValue 'N' // val no_ok : AnswerChar = No

let y1_ok = LanguagePrimitives.EnumToValue AnswerChar.Yes // val y1_ok : char = 'Y'
let y2_ok = unbox<char> AnswerChar.Yes  // val y2_ok : char = 'Y'
```

## Extras

Le package NuGet [FSharpx.Extras](https://github.com/fsprojects/FSharpx.Extras) comporte un module Enum proposant ces helpers :&#x20;

* `parse<'enum>: string -> 'enum`
* `tryParse<'enum>: string -> 'enum option`
* `getValues<'enum>: unit -> 'enum seq`

```fsharp
#r "nuget: FSharpx.Extras"
open FSharpx

type ColorN =
    | Red   = 1
    | Green = 2
    | Blue  = 3

let red = "Red" |> Enum.tryParse<ColorN> // val red: ColorN option = Some Red
let none = "xx" |> Enum.tryParse<ColorN> // val none: ColorN option = None

let blue: ColorN = "Blue" |> Enum.parse // val blue: ColorN = Blue
let ko =
    try
        "Ko" |> Enum.parse<ColorN>
    with ex ->
        printfn "💥 %s %s" (ex.GetType().FullName) ex.Message
        // 💥 System.ArgumentException: Requested value 'Ko' was not found
        enum<ColorN> 0

let colors = Enum.getValues<ColorN> () |> Seq.toList
// val colors: ColorN list = [Red; Green; Blue]
```
