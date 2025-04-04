---
description: A.k.a. Discriminated Unions (DU)
---

# Unions

## Points clés

* Terme exacte : « Union discriminée », _Discriminated Union (DU)_
* Types Somme : représente un **OU**, un **choix** entre plusieurs _Cases_
  * Même principe que pour une `enum` mais généralisé
* Chaque _case_ doit avoir un _Tag_ _(a.k.a Label)_
  * C'est le **discriminant** de l'union pour identifier le _case_
* Chaque _case_ **peut** contenir des données

```fsharp
type Billet =
    | Adulte                 // aucune donnée -> ≃ singleton stateless
    | Senior of int          // contient un 'int' (mais on ne sait pas ce que c'est)
    | Enfant of age: int     // contient un 'int' de nom 'age'
    | Famille of Billet list // contient une liste de billet
                             // type récursif -- pas besoin de 'rec'
```

### Qualification des _Tags_

Les _Tags_ peuvent être utilisés :&#x20;

* sans qualification → `Adulte`
* sauf pour résoudre un conflit de noms ou par choix de nommage → `Billet.Adulte`

☝ On peut forcer l'usage avec qualification en décorant l'union de l'attribut `RequireQualifiedAccess`, essentiellement pour des raisons de nommage de l'union et de ses tags, pour que le code se lise sans ambiguïté.

### Casse des _Tags_

Les _Tags_ doivent être nommés en PascalCase ❗

💡 Depuis F# 7.0, la camelCase est possible si l'union est décorée avec  `RequireQualifiedAccess.`

### Champs nommés - _Labelled Fields_

Pratiques pour :&#x20;

* Ajouter un sens à un type primitif : \
  → Dans l'exemple précédent, en ligne 4, le _case_ `Enfant` contient un champ de type `int` qui est nommé `age`.
* Distinguer deux champs du même type au sein d'un _Tuple_ : \
  → Exemple :&#x20;

```fsharp
type ComplexNumber =
    | Cartesian of Real: float * Imaginary: float
    | Polar of Magnitude: float * Phase: float
```

☝ **Notes :**

* Le nommage des champs est optionnel.
* En tant que champ, on optera pour le PascalCase.\
  Mais on peut aussi les voir en tant que paramètres, alors en camelCase.
* Quand un _case_ contient plusieurs champs, on peut n'en nommer que certains.\
  → Mais je ne recommande pas cette dissymétrie.

## Déclaration

Sur plusieurs lignes : 1 ligne / _case_ → ☝ Ligne indentée et commençant par `|`

Sur une seule ligne -- si déclaration reste **courte** ❗ → 💡 Pas besoin du 1er `|`

```fsharp
open System

type IntOrBool =
    | Int32 of Int32                        // 💡 Tag de même nom que ses données
    | Boolean of Boolean

type OrderId = OrderId of int               // 👈 Single-case union
                                            // 💡 Tag de même nom que l'union parent
type Found<'T> = Found of 'T | NotFound     // 💡 Type générique
```

## Instanciation

_Tag_ ≃ **constructeur** → Fonction appelée avec les éventuelles données du _case_

```fsharp
type Shape =
    | Circle of radius: int
    | Rectangle of width: int * height: int

let circle = Circle 12          // Type: 'Shape', Valeur: 'Circle 12'
let rect = Rectangle (4, 3)     // Type: 'Shape', Valeur: 'Rectangle (4, 3)'

// Utilisation du nom des champs
let rec2 = Rectangle (height = 4, width = 6)

let circles = [1..4] |> List.map Circle     // 👈 Tag employé comme fonction
```

## Conflit de noms

Quand 2 unions ont des tags de même nom → Qualifier le tag avec le nom de l'union

```fsharp
type Shape =
    | Circle of radius: int
    | Rectangle of width: int * height: int

type Draw = Line | Circle       // 'Circle' sera en conflit avec le tag de 'Shape'

let draw = Circle              // Type='Draw' (type le + proche) -- ⚠️ à éviter car ambigu

// Tags qualifiés par leur type union
let shape = Shape.Circle 12
let draw' = Draw.Circle
```

## Accès aux données internes

Uniquement via _pattern matching_ Matching d'un type Union est **exhaustif**

```fsharp
type Shape =
    | Circle of radius: float
    | Rectangle of width: float * height: float

let area shape =
    match shape with
    | Circle r -> Math.PI * r * r   // 💡 Même syntaxe que instanciation
    | Rectangle (w, h) -> w * h

let isFlat = function 
    | Circle 0.                     // 💡 Constant pattern
    | Rectangle (0., _)
    | Rectangle (_, 0.) -> true     // 💡 OR pattern
    | Circle _
    | Rectangle _ -> false
```

## &#x53;_&#x69;ngle-case union_

Union avec un seul cas encapsulant un type (généralement primitif)

```fsharp
type CustomerId = CustomerId of int
type OrderId = OrderId of int

let fetchOrder (OrderId orderId) =    // 💡 Déconstruction directe sans 'match'
    ...
```

Assure _type safety_ contrairement au simple type alias → Impossible de passer un `CustomerId` à une fonction attendant un `OrderId` 👍

Permet d'éviter _Primitive Obsession_ à coût minime

## Style "enum"

Tous les _cases_ sont vides = dépourvus de données → ≠ `enum` .NET 📍[enums.md](enums.md "mention")

L'instanciation et le pattern matching se font juste avec le _tag_ → Le _tag_ n'est plus une ~~fonction~~ mais une valeur _(_[_singleton_](https://fsharpforfunandprofit.com/posts/fsharp-decompiled/#enum-style-unions)_)_

```fsharp
type Answer = Yes | No | Maybe
let answer = Yes

let print answer =
    match answer with
    | Yes   -> printfn "Oui"
    | No    -> printfn "Non"
    | Maybe -> printfn "Peut-être"
```
