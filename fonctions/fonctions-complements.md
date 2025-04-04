# Fonctions : compléments

## Mémoïsation

💡 **Idée :** réduire le temps de calcul d'une fonction

❓ **Comment :** mise en cache des résultats \
→ Au prochain appel avec mêmes arguments, renverra résultat en cache

👉 **En pratique :** fonction `memoizeN` de la librairie [FSharpPlus](https://fsprojects.github.io/FSharpPlus/reference/fsharpplus-memoization.html#memoizeN)

:warning: **Attention :** Comme tout optimisation, à utiliser quand le besoin se fait sentir et en validant (mesurant) que cela marche sans désagrément annexe.

☝ Ne pas confondre avec expression `lazy` _(slide suivante)_

## Lazy expression

Sucre syntaxique pour créer un objet .NET `Lazy<'T>` à partir d'une expression \
→ Expression pas évaluée immédiatement mais qu'à la 1ère demande _(_[_Thunk_](https://en.wikipedia.org/wiki/Thunk)_)_ \
→ Intéressant pour améliorer performance sans trop complexifier le code

```fsharp
let printAndForward x = printfn $"{x}"; x

let a = lazy (printAndForward "a")

let b = printAndForward "b"
// > b

printfn $"{a.Value} et {b}"
// > a
// > a et b

printfn $"{a.Value} et c"
// > a et c
```

### `Lazy` active pattern

Extrait la valeur dans un objet `Lazy`

```fsharp
let triple (Lazy i) = 3 * i // Lazy<int> -> int

let v =
    lazy
        (printf "eval!"
         5 + 5) // Lazy<int>

triple v
// eval!val it: int = 30

triple v
// val it: int = 30
```

## Organisation des fonctions

3 façons d'organiser les fonctions = 3 endroits où les déclarer :

* _Module_ : fonction déclarée dans un module 📍 [module.md](../module-and-namespace/module.md "mention")
* _Nested_ : fonction déclarée à l'intérieur d'une valeur / fonction
  * 💡 Encapsuler des helpers utilisés juste localement
  * ☝ Paramètres de la fonction chapeau accessibles à fonction _nested_
* _Method_ : fonction définie comme méthode dans un type _(next slide)_

## Méthodes

* Définies avec mot-clé `member` plutôt que `let`
* Choix du _self-identifier_ : `this`, `me`, `_`...
* Paramètres sont au choix :
  * Tuplifiés : style OOP
  * Curryfiés : style FP

### Méthodes - Exemple

```fsharp
type Product = { SKU: string; Price: float } with  // 👈 `with` nécessaire pour l'indentation
    // Style avec tuplification et `this`          // Alternative : `{ SKU...}` à la ligne
    member this.TupleTotal(qty, discount) =
        (this.Price * float qty) - discount

    // Style avec curryfication et `me`
    member me.CurriedTotal qty discount = // 👈 `me` désigne le "this"
        (me.Price * float qty) - discount // 👈 `me.Price` pour accéder à la propriété `Price`
```

## Fonction _vs_ Méthode

| Fonctionnalité                                       | Fonction              | Méthode                                                                     |
| ---------------------------------------------------- | --------------------- | --------------------------------------------------------------------------- |
| Nommage                                              | camelCase             | PascalCase                                                                  |
| Curryfication                                        | ✅ oui                 | ✅ si ni tuplifiée, ni surchargée                                            |
| Paramètres nommés                                    | ❌ non                 | ✅ si tuplifiés                                                              |
| Paramètres optionnels                                | ❌ non                 | ✅ si tuplifiés                                                              |
| Surcharge / _overload_                               | ❌ non                 | ✅ si tuplifiés                                                              |
| <p>Inférence des arguments<br>→ à la déclaration</p> | <p><br>➖ Possible</p> | <p>✅ oui, du <code>this</code><br>➖ Possible pour les autres arguments</p>  |
| → à l'usage                                          | ✅ oui                 | ❌ non, il faut annoter le type de l'objet pour utiliser une de ses méthodes |
| En argument d'une _high-order fn_                    | ✅ oui                 | ❌ non, lambda nécessaire                                                    |
| Support du `inline`                                  | ✅ oui                 | ✅ oui                                                                       |
| Récursive                                            | ✅ si `rec`            | ✅ oui                                                                       |

## How To: Pipeline avec une méthode d'instance ?

Exemple : comment "piper" la méthode `ToLower()` d'une `string` ?

* Via lambda : `"MyString" |> (fun x -> x.ToLower())`
* Idem via fonction nommée telle que :
  * `String.toLower` de la librairie [FSharpPlus](https://fsprojects.github.io/FSharpPlus/reference/fsharpplus-string.html)
  * `"MyString" |> String.toLower`

Alternative au pipeline : passer par une valeur intermédiaire : \
→ `let low = "MyString".ToLower()`

## Interop avec la BCL

> BCL = Base Class Library .NET

### Méthode void

Une méthode .NET `void` est vue en F# comme renvoyant `unit`.

```fsharp
let liste = System.Collections.Generic.List<int>()
liste.Add
(* IntelliSense Ionide:
abstract member Add:
   item: int
      -> unit
*)
```

Réciproquement, une fonction F# renvoyant `unit` est compilée en une méthode `void`.

```fsharp
// F#
let ignore _ = ()
```

Equivalent C# d'après [SharpLab](https://sharplab.io/#v2:DYLgZgzgNAJiDUAfYBTALgAgJYHMB2A9gE4oYD6GAvBgBQCUQA==) :&#x20;

```csharp
public static void ignore<a>(a _arg1) {}
```

### Appel à une méthode de la BCL à N arguments

Une méthode .NET avec plusieurs arguments est "pseudo-tuplifiée" :

* Les arguments doivent tous être spécifiés (1)
* L'application partielle des paramètres n'est pas supportée (2)
* Les appels ne marche pas avec un vrai tuple F♯ :warning: (3)

```fsharp
System.String.Compare("a", "b") // ✅ (1)
System.String.Compare ("a","b") // ✅

System.String.Compare "a" "b"   // ❌ (2)
System.String.Compare "a","b"   // ❌

let tuple = ("a","b")
System.String.Compare tuple     // ❌ (3)
```

💡 **Note :** on peut utiliser l'opérateur pipe `|>` pour appeler une méthode tuplifiée\
→ Cf. [Broken link](broken-reference "mention") / [membres.md](../oriente-objet/membres.md "mention") 📍

### Paramètre `out` - En C♯

`out` utilisé pour avoir plusieurs valeurs en sortie \
→ Ex : `Int32.TryParse`, `Dictionary<,>.TryGetValue` :

```csharp
if (int.TryParse(maybeInt, out var value))
    Console.WriteLine($"It's the number {value}.");
else
    Console.WriteLine($"{maybeInt} is not a number.");
```

### Paramètre `out` - En F♯

Possibilité de consommer la sortie sous forme de tuple 👍

```fsharp
  match System.Int32.TryParse maybeInt with
  | true, i  -> printf $"It's the number {value}."
  | false, _ -> printf $"{maybeInt} is not a number."
```

💡 Fonctions F♯ `tryXxx` s'appuient plutôt sur le type `Option<T>` 📍

### Instancier une classe avec `new` ?

```fsharp
// (1) `new` autorisé mais non recommandé
type MyClass(i) = class end

let c1 = MyClass(12)      // 👍
let c2 = new MyClass(234) // 👌 mais pas idiomatique

// (2) IDisposable => `new` obligatoire et `use` plutôt que `let`
open System.IO
let fn () =
    let _ = FileStream("hello.txt", FileMode.Open)
    // ⚠️ Warning : Il est recommandé que les objets prenant en charge
    // l'interface IDisposable soient créés avec la syntaxe 'new Type(args)'

    use f = new FileStream("hello.txt", FileMode.Open)
    f.Close()
```

### Appel d'une méthode surchargée

* Compilateur peut ne pas comprendre quelle surcharge est appelée
* Astuce : faire appel avec argument nommé

```fsharp
let createReader fileName =
    new System.IO.StreamReader(path=fileName)
    // ☝️ Param `path` → `filename` inféré en `string`

let createReaderByStream stream =
    new System.IO.StreamReader(stream=stream)
    // ☝️ Param `stream` de type `System.IO.Stream`
```
