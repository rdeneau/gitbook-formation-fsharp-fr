# Types flexibles

## Besoin

Lors de la création de certaines fonctions génériques, il faut spécifier qu'un paramètre de type est un sous-type d'un certain autre type.

→ Illustration grâce à un exemple :

```fsharp
open System.Collections.Generic

// V1
let add item (collection: ICollection<_>) =
    collection.Add item
    collection

let a = List([1..3])    // List<int>
let b = a |> add 4      // ICollection<int> != List<int> ❗
```

Solutions :

* **V2 :** indiquer une **contrainte de type**
* **V3 :** indiquer un **type flexible**

```fsharp
(* V1  ❌ *)  let add item (collection: ICollection<_>) = …
(* V2a 😖 *)  let add<'t, 'u when 'u :> ICollection<'t>> (item: 't) (collection: 'u) : 'u = …
(* V2b 😕 *)  let add (item: 't) (collection: 'u when 'u :> ICollection<'t>) : 'u = …
(* V3  ✅ *)  let add item (collection: #ICollection<_>) = …
```

⚖️ **Bilan :**

* **V2a** : syntaxe similaire au C♯ → verbeux et pas très lisible ! 😖
* **V2b** : version améliorée en F♯ → + lisible mais encore un peu verbeux ! 😕
* **V3**   : syntaxe proche de **V1** → concision « dans l'esprit F♯ » ✅

## Autres usages

Faciliter l'usage de la fonction sans avoir besoin d'un _upcast_

```fsharp
let join separator (generate: unit -> seq<_>) =
    let items = System.String.Join (separator, generate() |> Seq.map (sprintf "%A"))
    $"[ {items} ]"

let s1 = join ", " (fun () -> [1..5])               // 💥 Error FS0001
let s2 = join ", " (fun () -> [1..5] :> seq<int>)   // 😕 Marche mais pénible à écrire
```

Avec un type flexible :

```fsharp
let join separator (generate: unit -> #seq<_>) =
    // [...]

let s1 = join ", " (fun () -> [1..5])               // ✅ Marche naturellement
```

Dans l'exemple ci-dessous, `items` est inféré avec la bonne contrainte :

```fsharp
let tap f items =
    items |> Seq.iter f
    items
// val tap : f:('a -> unit) -> items:'b -> 'b when 'b :> seq<'a>
```

💡 Quid de faciliter la lecture du code avec un type flexible ?

```fsharp
let tap f (items: #seq<_>) =
    // [...]
```

:warning: Astuce précédente ne marche pas toujours !

```fsharp
let max x y =
    if x > y then x else y
// val max : x:'a -> y:'a -> 'a when 'a : comparison
```

`x` et `y` doivent satisfaire 2 conditions

1. `'a : comparison` ≃ les types de `x` et `y` implémentent `IComparable` \
   → `(x: #IComparable) (y: #IComparable)` ?
2. `x:'a` et `y:'a` \
   → `x` et `y` ont le même type \
   → Non exprimable sous forme de type flexible ! 😞

## Résumé

Type flexible :

* Utilisé dans la déclaration de certaine fonction générique
* Indique qu'un paramètre de type est un sous-type d'un type spécifié
* Sucre syntaxique au format `#super-type`
* Équivalent de `'T when 'T :> super-type`

Autres usages :

* Faciliter l'usage de la fonction sans avoir besoin d'un _upcast_
* Faciliter la lecture du code ?
