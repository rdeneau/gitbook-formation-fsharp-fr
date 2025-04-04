# Extensions de type

## Définition

Membres d'un type définis hors de son bloc `type` principal.

Chacun de ces membres est appelé une **augmentation**.

3 catégories d'extension :

* Extension intrinsèque
* Extension optionnelle
* Méthodes d'extension

## Extension intrinsèque

Définie dans même fichier et même namespace que le type

→ Membres intégrés au type à la compilation, visibles par _Reflection_

💡 **Cas d'usage**

Déclarer successivement :

1. Type (ex : `type List`)
2. Module compagnon de ce type (ex : fonction `List.length list`)
3. Extension utilisant ce module compagnon (ex : membre `list.Length`)

* 👉 Façon + propre en FP de séparer les fonctionnalités des données
* 💡 Inférence de types marche mieux avec fonctions que membres

**Exemple :**

```fsharp
namespace Example

type Variant =
    | Num of int
    | Str of string

module Variant =
    let print v =
        match v with
        | Num n -> printf "Num %d" n
        | Str s -> printf "Str %s" s

// Add a member to Variant as an extension
type Variant with
    member x.Print() = Variant.print x
```

## Extension optionnelle

Extension définie au-dehors du module/namespace/assembly du type étendu.

💡 Pratique pour les types dont la déclaration n'est pas modifiable directement, par exemple ceux issus d'une librairie.

```fsharp
module EnumerableExtensions

open System.Collections.Generic

type IEnumerable<'T> with
    /// Repeat each element of the sequence n times
    member xs.RepeatElements(n: int) =
        seq {
            for x in xs do
                for _ in 1 .. n -> x
        }
```

**Compilation :** en méthode statique → version simplifiée :

```csharp
public static class EnumerableExtensions
{
    public static IEnumerable<T> RepeatElements<T>(IEnumerable<T> xs, int n) {...}
}
```

**Usage :** comme un vrai membre, après avoir importé son module :

```fsharp
open EnumerableExtensions

let x = [1..3].RepeatElements(2) |> List.ofSeq
// [1; 1; 2; 2; 3; 3]
```

**Autre exemple :**

```fsharp
// File Person.fs
type Person = { First: string; Last: string }

// File PersonExtensions.fs
module PersonExtensions =
    type Person with
        member this.FullName =
            $"{this.Last.ToUpper()} {this.First}"

// Usage elsewhere
open PersonExtensions
let joe = { First = "Joe"; Last = "Dalton" }
let s = joe.FullName  // "DALTON Joe"
```

### Limites

* Doit être déclarée dans un module
* Pas compilée dans le type, pas visible par Reflection
* Membres visibles qu'en F#, invisibles en C#

## Extension de type et surcharges

☝ Implémenter des surcharges :

* Recommandé dans la déclaration initiale du type ✅
* Déconseillé dans une extension de type ⛔

```fsharp
type Variant = Num of int | Str of string with
    override this.ToString() = ... ✅

module Variant = ...

type Variant with
    override this.ToString() = ... ⚠️
    // Warning FS0060: Override implementations in augmentations are now deprecated...
```

## Extension de type et alias de type

Sont incompatibles :

```fsharp
type i32 = System.Int32

type i32 with
    member this.IsEven = this % 2 = 0
// 💥 Error FS0964: Les abréviations de type ne peuvent pas avoir d'augmentations
```

💡 **Solution :** il faut utiliser le vrai nom du type

```fsharp
type System.Int32 with
    member this.IsEven = this % 2 = 0
```

☝ Les tuples F♯ tels-que (ex : `int * int`) ne peuvent pas être augmentés ainsi.\
💡 Mais on peut avec une méthode d'extension à la C# 📍

## Limite

Extension autorisée sur type générique sauf quand les contraintes sont différentes :

```fsharp
open System.Collections.Generic

type IEnumerable<'T> with
    member this.Sum() = Seq.sum this
// 💥      ~~~~~~~~~~ Error FS0670
// Ce code n'est pas suffisamment générique. Impossible de généraliser la variable de type
// ^T when ^T: (static member get_Zero: -> ^T) and ^T: (static member (+) : ^T * ^T -> ^T)

// ☝ Cette contrainte provient de `Seq.sum`
```

**Solution :** méthode d'extension à la C# 📍

## Méthode d'extension

Méthode statique :

* Décorée de `[<Extension>]`
* Définie dans classe `[<Extension>]`
* Type du 1er argument = type étendu _(`IEnumerable<'T>` ci-dessous)_

```fsharp
namespace Extensions

open System.Collections.Generic
open System.Runtime.CompilerServices

[<Extension>]
type EnumerableExtensions =
    [<Extension>]
    static member inline Sum(xs: IEnumerable<'T>) = Seq.sum xs

// 💡 `inline` est nécessaire à cause des contraintes requises par Seq.num
```

Exemple simplifié :

```fsharp
open System.Runtime.CompilerServices

[<Extension>]
type EnumerableExtensions =
    [<Extension>]
    static member inline Sum(xs: seq<_>) = Seq.sum xs

let x = [1..3].Sum()
//------------------------------
// Output en console FSI (syntaxe verbeuse) :
type EnumerableExtensions =
  class
    static member
      Sum : xs:seq<^a> -> ^a
              when ^a : (static member ( + ) : ^a * ^a -> ^a)
              and  ^a : (static member get_Zero : -> ^a)
  end
val x : int = 6
```

Pseudo équivalent en C♯ :

```csharp
using System.Collections.Generic;

namespace Extensions
{
    public static class EnumerableExtensions
    {
        public static TSum Sum<TItem, TSum>(this IEnumerable<TItem> source) {...}
    }
}
```

☝ **Note :** en vrai, il y a plein de `Sum()` dans LINQ pour chaque type : `int`, `float`…

→ [_Code source_](https://github.com/dotnet/runtime/blob/main/src/libraries/System.Linq/src/System/Linq/Sum.cs)

### Tuples

On peut ajouter une méthode d'extension à tout tuple F♯ :

```fsharp
open System.Runtime.CompilerServices

[<Extension>]
type EnumerableExtensions =
    [<Extension>]
    // static member IsDuplicate : ('a * 'a) -> bool when 'a : equality
    static member inline IsDuplicate((x, y)) =
        x = y

let b1 = (1, 1).IsDuplicate()  // true
let b2 = ("a", "b").IsDuplicate()  // false
```

## Comparatif

| Fonctionnalité      | Extension de type            | Méthode d'extension    |
| ------------------- | ---------------------------- | ---------------------- |
| Méthodes            | ✅ instance, ✅ statique       | ✅ instance, ❌ statique |
| Propriétés          | ✅ instance, ✅ statique       | ❌ _Non supporté_       |
| Constructeurs       | ✅ intrinsèque, ❌ optionnelle | ❌ _Non supporté_       |
| Étendre contraintes | ❌ _Non supporté_             | ✅ _Supporte SRTP_      |

## Limites

Ne participent pas au polymorphisme :

* Pas dans table virtuelle
* Pas de membre `virtual`, `abstract`
* Pas de membre `override` _(mais surcharges 👌)_

## Extensions _vs_ classe partielle C♯

| Fonctionnalité      | Multi-fichiers | Compilé dans type | Tout type           |
| ------------------- | -------------- | ----------------- | ------------------- |
| Classe partielle C♯ | ✅ Oui          | ✅ Oui             | Que `partial class` |
| Extens° intrinsèque | ❌ Non          | ✅ Oui             | ✅ Oui               |
| Extens° optionnelle | ✅ Oui          | ❌ Non             | ✅ Oui               |
