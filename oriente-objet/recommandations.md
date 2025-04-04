---
description: "Recommandations pour\_l'orienté-objet"
---

# Recommandations

## Pas d'orienté-objet là où F♯ est bon

Inférence marche mieux avec une expression telle que `fonction(objet)` que `objet.membre`

**Hiérarchie simple d'objets**

* ❌ Éviter héritage
* ✅ Préférer type _Union_ et _pattern matching_ exhaustif, + simple en général
* En particulier les types récursifs comme les arbres, épaulés par fonction `fold`
* https://fsharpforfunandprofit.com/series/recursive-types-and-folds/

**Égalité structurelle**

* ❌ Éviter classe _(égalité par référence par défaut)_
* ✅ Préférer un _Record_ ou une _Union_
* ❓ Envisager égalité structurelle custom / performance
* https://www.compositional-it.com/news-blog/custom-equality-and-comparison-in-f/

## Orienté-objet recommandé

1. Encapsuler état mutable → dans une classe
2. Grouper fonctionnalités → dans une interface
3. API expressive et user-friendly → méthodes tuplifiées
4. API F♯ consommée en C♯ → membres d'extension
5. Gestion des dépendances → injection dans constructeur
6. Dépasser limites des fonctions d'ordre supérieur

## Classe pour encapsuler un état mutable

```fsharp
// 😕 Encapsuler état mutable dans une closure → fonction impure contre-intuitif ⚠️
let counter =
    let mutable count = 0
    fun () ->
        count <- count + 1
        count

let x = counter ()  // 1
let y = counter ()  // 2

// ✅ Encapsuler état mutable dans une classe
type Counter() =
    let mutable count = 0   // Champ privé
    member _.Next() =
        count <- count + 1
        count
```

## Interface pour grouper des fonctionnalités

```fsharp
let checkRoundTrip serialize deserialize value =
    value = (value |> serialize |> deserialize)
// val checkRoundTrip :
//   serialize:('a -> 'b) -> deserialize:('b -> 'a) -> value:'a -> bool
//     when 'a : equality
```

`serialize` et `deserialize` forment un groupe cohérent → Les grouper dans un objet

```fsharp
let checkRoundTrip serializer data =
    data = (data |> serializer.Serialize |> serializer.Deserialize)
```

💡 Préférer une interface à un _Record_

```fsharp
// ❌ Éviter : ce n'est pas un bon usage d'un Record
type Serializer<'T> = {
    Serialize: 'T -> string
    Deserialize: string -> 'T
}

// ✅ Recommandé
type Serializer =
    abstract Serialize<'T> : value: 'T -> string
    abstract Deserialize<'T> : data: string -> 'T
```

* Paramètres sont nommés dans les méthodes
* Objet facilement instanciable avec une expression objet

## API expressive

```fsharp
// ❌ Éviter                        // ✅ Préférer
                                    [<AbstractClass; Sealed>]
module Utilities =                  type Utilities =
    let name = "Bob"                    static member Name = "Bob"
    let add2 x y = x + y                static member Add(x,y) = x + y
    let add3 x y z = x + y + z          static member Add(x,y,z) = x + y + z
    let log x = ...                     static member Log(x, ?retryPolicy) = ...
    let log' x retryPolicy = ...
```

* Méthode `Add` surchargée _vs_ `add2`, `add3`
* Une seule méthode `Log` avec paramètre optionnel `retryPolicy`

🔗 [F♯ component design guidelines - Libraries used in C♯](https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/component-design-guidelines#guidelines-for-libraries-for-use-from-other-net-languages)

## API F♯ consommée en C♯ - Type

Ne pas exposer ce type tel quel :

```fsharp
type RadialPoint = { Angle: float; Radius: float }

module RadialPoint =
    let origin = { Angle = 0.0; Radius = 0.0 }
    let stretch factor point = { point with Radius = point.Radius * factor }
    let angle (i: int) (n: int) = (float i) * 2.0 * System.Math.PI / (float n)
    let circle radius count =
        [ for i in 0..count-1 -> { Angle = angle i count; Radius = radius } ]
```

💡 Pour faciliter la découverte du type et l'usage de ses fonctionnalités en C♯

* Mettre le tout dans un namespace
* Augmenter le type avec fonctionnalités du module compagnon

```fsharp
namespace Fabrikam

type RadialPoint = {...}
module RadialPoint = ...

type RadialPoint with
    static member Origin = RadialPoint.origin
    static member Circle(radius, count) = RadialPoint.circle radius count |> List.toSeq
    member this.Stretch(factor) = RadialPoint.stretch factor this
```

👉 L'API consommée en C♯ est +/- équivalente à :

```csharp
namespace Fabrikam
{
    public static class RadialPointModule { ... }

    public sealed record RadialPoint(double Angle, double Radius)
    {
        public static RadialPoint Origin => RadialPointModule.origin;

        public static IEnumerable<RadialPoint> Circle(double radius, int count) =>
            RadialPointModule.circle(radius, count);

        public RadialPoint Stretch(double factor) =>
            new RadialPoint(Angle@, Radius@ * factor);
    }
}
```

## Gestion des dépendances

### Technique FP

**Paramétrisation des dépendances + application partielle**

* Marche à petite dose : peu de dépendances, peu de fonctions concernées
* Sinon, vite pénible à coder et à utiliser 🥱

```fsharp
module MyApi =
    let function1 dep1 dep2 dep3 arg1 = doStuffWith dep1 dep2 dep3 arg1
    let function2 dep1 dep2 dep3 arg2 = doStuffWith' dep1 dep2 dep3 arg2
```

### Technique OO

**Injection de dépendances**

* Injecter les dépendances dans le constructeur de la classe
* Utiliser ces dépendances dans les méthodes

👉 Offre une API + user-friendly 👍

```fsharp
type MyParametricApi(dep1, dep2, dep3) =
    member _.Function1 arg1 = doStuffWith dep1 dep2 dep3 arg1
    member _.Function2 arg2 = doStuffWith' dep1 dep2 dep3 arg2
```

✅ Particulièrement recommandé pour encapsuler des **effets de bord** :

* Connexion à une BDD, lecture de settings...

### Techniques FP++

_Dependency rejection_ = pattern sandwich

* Rejeter dépendances dans couche Application, hors de couche Domaine
* Puissant et simple 👍
* ... quand c'est adapté ❗

Monade _Reader_

* Pour fans de Haskell, sinon trop disruptif 😱

🔗 [Six approaches to dependency injection](https://fsharpforfunandprofit.com/posts/dependencies/), F# for Fun and Profit, Dec 2020

## Limites des fonctions d'ordre supérieur

Mieux vaut passer un objet plutôt qu'une lambda en paramètre d'une fonction d'ordre supérieure dans certains cas :

### Lambda est une **commande** `'T -> unit`

* ✅ Préférer déclencher un effet de bord via un objet : `type ICommand = abstract Execute : 'T -> unit`

### Arguments de la lambda pas explicites

* ❌ `let test (f: float -> float -> string) =...`
* ✅ Solution 1 : type wrappant les 2 args `float` : `f: Point -> string` avec `type Point = { X: float; Y: float }`
* ✅ Solution 2 : interface + méthode pour avoir paramètres nommés : `type IXxx = abstract Execute : x:float -> y:float -> string`

### Lambda "vraiment" générique

```fsharp
let test42 (f: 'T -> 'U) =
    f 42 = f "42"
// ❌ ^^     ~~~~
// ^^ Cette construction rend le code moins générique :
//    paramètre de type 'T contraint de représenter le type `int`
// ~~ Type `int` attendu != type `string` actuel
```

✅ Solution : wrapper la fonction dans un objet

```fsharp
type Func2<'U> =
    abstract Invoke<'T> : 'T -> 'U

let test42 (f: Func2<'U>) =
    f.Invoke 42 = f.Invoke "42"
```
