# Type Option

## Présentation

A.k.a `Maybe` _(Haskell),_ `Optional` _(Java 8)_

Modélise l'absence de valeur \
→ Dans le sens d'une possibilité pour une valeur d'être absente\
→ Différent de `unit` qui est utilisé dans le cas où il n'y a jamais de valeur

Défini sous la forme d'une union avec 2 _cases_ :

```fsharp
type Option<'Value> =
    | None              // Case sans donnée → quand valeur absente
    | Some of 'Value    // Case avec donnée → quand valeur présente
```

## Cas d'utilisation

### Modéliser un champ optionnel

```fsharp
type Civility = Mr | Mrs
type User = { Name: string; Civility: Civility option }

let joey  = { Name = "Joey"; Civility = Some Mr }
let guest = { Name = "Guest"; Civility = None }
```

→ Rend explicite le fait que `Name` est obligatoire et `Civility` facultatif

☝ **Attention :** ce design n'empêche pas ici d'avoir `Name = null` _(limite BCL)_

### Opération partielle

Opération où aucune valeur de sortie n'est possible pour certaines entrées.

## Exemples

### Exemple 1 - Inverse d'un nombre

```fsharp
let inverse n = 1.0 / n

let tryInverse n =
    match n with
    | 0.0 -> None
    | n   -> Some (1.0 / n)
```

| **Fonction** | **Opération** | **Signature**           | `n = 0.5`  | `n = 0.0`    |
| ------------ | ------------- | ----------------------- | ---------- | ------------ |
| `inverse`    | Partielle     | `float -> float`        | `2.0`      | `infinity` ❓ |
| `tryInverse` | Totale        | `float -> float option` | `Some 2.0` | `None` 👌    |

### Exemple 2 - Recherche d'un élément dans une collection

* Opération partielle : `find predicate` → 💥 quand élément non trouvé
* Opération totale : `tryFind predicate` → `None` ou `Some item`

## Avantages 👍

* Explicite (honnête) concernant la partialité de l'opération
  * Pas de valeur spéciale (et cachée) : `null`, `infinity`
  * Pas d'exception
* Force le code appelant à gérer la totalité des cas :
  * Présence d'une valeur en sortie : `Some value`
  * Absence d'une valeur en sortie : `None`

## Flux de contrôle

Pour tester la présence de la valeur _(de type `'T`)_ dans l'option

* ❌ Ne pas utiliser `IsSome`, `IsNone` et `Value` (🤞💥)
  * ~~if option.IsSome then option.Value...~~
* 👌 A la main avec _pattern matching_
* ✅ Fonctions du module `Option`

### Manuel avec _pattern matching_

Exemple :

```fsharp
let print option =
    match option with
    | Some x -> printfn "%A" x
    | None   -> printfn "None"

print (Some 1.0)  // 1.0
print None        // None
```

### Intégré au module `Option`

Opération de _Mapping_ de la valeur (de type `'T`) **si ∃** :

* `option |> Option.map  f` avec `f` opération totale `'T -> 'U`
* `option |> Option.bind f` avec `f` opération partielle `'T -> 'U option`

Conserver la valeur **si ∃** et si respecte condition :

* `option |> Option.filter predicate` avec `predicate: 'T -> bool` appelé que si valeur ∃

### Exercice

Implémenter `map`, `bind` et `filter` avec _pattern matching_

### Solution

```fsharp
let map f option =             // (f: 'T -> 'U) -> 'T option -> 'U option
    match option with
    | Some x -> Some (f x)
    | None   -> None           // 🎁 1. Pourquoi on ne peut pas écrire `None -> option` ?

let bind f option =            // (f: 'T -> 'U option) -> 'T option -> 'U option
    match option with
    | Some x -> f x
    | None   -> None

let filter predicate option =  // (predicate: 'T -> bool) -> 'T option -> 'T option
    match option with
    | Some x when predicate x -> option
    | _ -> None                // 🎁 2. Implémenter `filter` avec `bind` ?
```

### Réponses 🎁

```fsharp
// 🎁 1. Pourquoi on ne peut pas écrire `None -> option` :
let map (f: 'T -> 'U) (option: 'T option) : 'U option =
    match option with
    | Some x -> Some (f x)
    | None   -> (*None*) option  // 💥 Erreur de typage : `'U option` attendu != `'T option`
```

```fsharp
// 🎁 2. Implémenter `filter` avec `bind` :
let filter predicate option =  // (predicate: 'T -> bool) -> 'T option -> 'T option
    let f x = if predicate x then option else None
    bind f option
```

### Exemple

```fsharp
// Application console de questions/réponses
type Answer = A | B | C | D

let tryParseAnswer text =
    match text with
    | "A" -> Some A
    | "B" -> Some B
    | "C" -> Some C
    | "D" -> Some D
    | _   -> None

// Fonction appelée quand l'utilisateur saisit la réponse au clavier à la question posée
let checkAnswer (expectedAnswer: Answer) (givenAnswer: string) =
    tryParseAnswer givenAnswer
    |> Option.filter ((=) expectedAnswer)
    |> Option.map (fun _ -> "✅")
    |> Option.defaultValue "❌"

["X"; "A"; "B"] |> List.map (checkAnswer B)  // ["❌"; "❌"; "✅"]
```

### Bénéfices

Rend logique métier + lisible

* Pas de `if hasValue then / else`
* Met en valeur le _happy path_
* Centralise à la fin la gestion de l'absence de valeur

💡 Les [computation-expression-ce.md](computation-expression-ce.md "mention")📍 fournissent une syntaxe alternative + légère

## `Option` _vs_ `List`

Option ≃ Liste de 0 ou 1 élément → cf. fonction `Option.toList`

```fsharp
let noneIsEmptyList       = Option.toList(None)   = []   // true
let someIsListWithOneItem = Option.toList(Some 1) = [1]  // true
```

☝ Une `List` peut avoir + de 1 élément \
→ Type `Option` modélise mieux l'absence de valeur que type `List`

💡 Module `Option` : beaucoup de même fonctions que module `List` \
→ `contains`, `count`, `exist`, `filter`, `fold`, `forall`, `map`

## `Option` _vs_ `null`

De part ses interactions avec la BCL, F♯ autorise parfois la valeur `null`

👉 **Bonne pratique**\
→ Isoler ces cas de figure et wrapper dans un type `Option`\
→ Par exemple en utilisant la fonction `Option.ofObj`

```fsharp
let readLine (reader: System.IO.TextReader) =
    reader.ReadLine() |> Option.ofObj

    // Équivalent à faire :
    match reader.ReadLine() with
    | null -> None
    | line -> Some line
```

## `Option` _vs_ `Nullable`

Type `System.Nullable<'T>` ≃ `Option<'T>` en + limité

* ❗ Ne marche pas pour les types références
* ❗ Manque comportement monadique i.e. fonctions `map` et `bind`
* ❗ En F♯, pas de magie comme en C♯ / mot clé `null`

👉 `Option` est le type idiomatique en F♯

💡 On utilise le type `Nullable` en cas d'interop. Pour l'instancier : \
&#x20;  → ❌ Contrairement au C♯, le mot clé `null` ne marche pas !\
&#x20;  → 👎 L'emploie de `Unchecked.defaultof` marche mais n'est pas élégant.\
&#x20;  → 👍 Utiliser l'un des constructeurs : `Nullable()` ou `Nullable(value)`

```fsharp
open System

let x: Nullable<int> = null
// ⚠️ The type 'Nullable<int>' does not have 'null' as a proper value. To create a null value for a Nullable type use 'System.Nullable()' (FS43)

let x = Unchecked.defaultof<Nullable<int>>
// val x: Nullable<int> = <null>

let x' = Nullable<int>()
// val x': Nullable<int> = <null>

let y = Nullable(1)
// val y: Nullable<int> = 1
```
