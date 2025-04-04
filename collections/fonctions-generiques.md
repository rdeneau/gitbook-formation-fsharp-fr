# Fonctions génériques

## Accès à un élément

| ↓ Accès \ Renvoie → | `'T` ou 💥     | `'T option`     |
| ------------------- | -------------- | --------------- |
| Par index           | `list.[index]` |                 |
|                     | `item index`   | `tryItem index` |
| Premier élément     | `head`         | `tryHead`       |
| Dernier élément     | `last`         | `tryLast`       |

→ Fonctions à préfixer par le module associé : `Array`, `List` ou `Seq` \
→ Dernier paramètre, la "collection", omis par concision \
→ 💥 : `ArgumentException` ou `IndexOutOfRangeException`

```fsharp
[1; 2] |> List.tryHead    // Some 1
[1; 2] |> List.tryItem 2  // None
```

### Coût :warning:

| Fonction \ Module | `Array` | `List` | `Seq`  |
| ----------------- | ------- | ------ | ------ |
| `head`            | O(1)    | O(1)   | O(1)   |
| `item`            | O(1)    | O(n) ❗ | O(n) ❗ |
| `last`            | O(1)    | O(n) ❗ | O(n) ❗ |
| `length`          | O(1)    | O(n) ❗ | O(n) ❗ |

## Combiner des collections

<table><thead><tr><th width="168.33333333333331">Fonction</th><th>Paramètre(s)</th><th>Taille finale</th></tr></thead><tbody><tr><td><code>append</code> / <code>@</code></td><td>2 collections de tailles N1 et N2</td><td>N1 + N2</td></tr><tr><td><code>concat</code></td><td>P collections de tailles N1..Np</td><td>N1 + N2 + ... + Np</td></tr><tr><td><code>zip</code></td><td>2 collections de même taille N ❗</td><td>N tuples</td></tr><tr><td><code>allPairs</code></td><td>2 collections de taille N1 et N2</td><td>N1 * N2 tuples</td></tr></tbody></table>

💡 `@` = opérateur infixe alias de `List.append` uniquement ~~(Array, Seq)~~

```fsharp
List.append [1;2;3] [4;5;6]    // [1; 2; 3; 4; 5; 6]
[1;2;3] @ [4;5;6]              // idem

List.concat [ [1]; [2; 3] ]    // [1; 2; 3]

List.zip [1; 2] ['a'; 'b']     // [(1, 'a'); (2, 'b')]

List.allPairs [1; 2] ['a'; 'b'] // [(1, 'a'); (1, 'b'); (2, 'a'); (2, 'b')]
```

### Cons `::` _vs_ Append `@`

_Cons_ `1 :: [2; 3]`&#x20;

* Élément ajouté en tête de liste&#x20;
* Liste paraît en ordre inverse 😕&#x20;
* Mais opération en **O(1)** 👍 -- _(`Tail` conservée)_

_Append_ `[1] @ [2; 3]`&#x20;

* Liste en ordre normal&#x20;
* Mais opération en **O(n)** ❗ -- _(Nouvelle `Tail` à chaque niveau)_

## Recherche d'un élément

Via un prédicat `f : 'T -> bool` :

| Quel élément \ Renvoie | `'T` ou 💥      | `'T option`        |
| ---------------------- | --------------- | ------------------ |
| Premier trouvé         | `find`          | `tryFind`          |
| Dernier trouvé         | `findBack`      | `tryFindBack`      |
| Index du 1er trouvé    | `findIndex`     | `tryFindIndex`     |
| Index du der trouvé    | `findIndexBack` | `tryFindIndexBack` |

```fsharp
[1; 2] |> List.find (fun x -> x < 2)      // 1
[1; 2] |> List.tryFind (fun x -> x >= 2)  // Some 2
[1; 2] |> List.tryFind (fun x -> x > 2)   // None
```

## Recherche de N éléments

| Recherche        | Combien d'éléments | Méthode          |
| ---------------- | ------------------ | ---------------- |
| Par valeur       | Au moins un        | `contains value` |
| Par prédicat `f` | Au moins un        | `exists f`       |
| "                | Tous               | `forall f`       |

```fsharp
[1; 2] |> List.contains 0      // false
[1; 2] |> List.contains 1      // true
[1; 2] |> List.exists (fun x -> x >= 2)  // true
[1; 2] |> List.forall (fun x -> x >= 2)  // false
```

## Sélection d'éléments

| Quels éléments    | Par nombre   | Par prédicat `f` |
| ----------------- | ------------ | ---------------- |
| Tous ceux trouvés |              | `filter f`       |
| Premiers ignorés  | `skip n`     | `skipWhile f`    |
| Premiers trouvés  | `take n`     | `takeWhile f`    |
|                   | `truncate n` |                  |

☝ **Notes :**

* Avec `skip` et `take`, 💥 exception si `n > list.Length` ; pas avec `truncate`
* Alternative pour `Array` : sélection par _Range_ `arr.[2..5]`

## Mapping d'éléments

Fonction prenant en entrée :

* Une fonction de mapping `f`
* Une collection d'éléments de type `'T`

| Fonction  | Mapping `f`       | Retour      | Quel(s) élément(s) |
| --------- | ----------------- | ----------- | ------------------ |
| `map`     | `'T -> 'U`        | `'U list`   | Autant d'éléments  |
| `mapi`    | `int -> 'T -> 'U` | `'U list`   | idem               |
| `collect` | `'T -> 'U list`   | `'U list`   | _flatMap_          |
| `choose`  | `'T -> 'U option` | `'U list`   | Moins d'éléments   |
| `pick`    | `'T -> 'U option` | `'U`        | 1er élément ou 💥  |
| `tryPick` | `'T -> 'U option` | `'U option` | 1er élément        |

### `map` _vs_ `mapi`

`mapi` ≡ `map` _with index_

`map` : mapping `'T -> 'U` → Opère sur valeur de chaque élément

`mapi` : mapping `int -> 'T -> 'U` → Opère sur index et valeur de chaque élément

```fsharp
["A"; "B"]
|> List.mapi (fun i x -> $"{i+1}. {x}")
// ["1. A"; "2. B"]
```

### Alternative à `mapi`

Hormis `map` et `iter`, aucune fonction `xxx` n'a de variante en `xxxi`.

💡 Utiliser `indexed` pour obtenir les éléments avec leur index

```fsharp
let isOk (i, x) = i >= 1 && x <= "C"

["A"; "B"; "C"; "D"]
|> List.indexed       // [ (0, "A"); (1, "B"); (2, "C"); (3, "D") ]
|> List.filter isOk   //           [ (1, "B"); (2, "C") ]
|> List.map snd       //               [ "B" ; "C" ]
```

### `map` _vs_ `iter`

<table><thead><tr><th width="149">Fonction</th><th>Lambda en paramètre</th><th>Type de retour</th></tr></thead><tbody><tr><td><code>map</code></td><td><code>'T -> 'U</code></td><td><code>'U list</code></td></tr><tr><td><code>iter</code></td><td><code>'T -> unit</code><br><em>(= <code>Action</code> en C#)</em></td><td><del>unit list</del><br><code>unit</code></td></tr></tbody></table>

On pourrait considérer `iter` comme inutile, `map` pouvant prendre une lambda `'T -> unit` en paramètre, mais c'est se tromper.

* Le type de retour n'est pas le même : `iter` renvoie `unit`, alors que `map` renverrait alors `unit list`. Par contre, le compilateur n'oblige pas à ignorer cette valeur (cf. [#fonction-ignore](../fonctions/signature.md#fonction-ignore "mention")), ce qui aiderait à détecter l'usage erroné de `map` au lieu de `iter` !?
* Avec une `Seq`, `map` ne consomme pas la séquence et donc ne déclenche pas immédiatement les actions pour chaque élément de la séquence !

👉 **Conclusion :** `iter` correspond à un cas d'utilisation différent de `map`. \
&#xNAN;_(Idem respectivement pour `iteri` et `mapi`)_

<pre class="language-fsharp"><code class="lang-fsharp">// ❌ À éviter
<strong>["A"; "B"; "C"] |> List.mapi (fun i x -> printfn $"Item #{i}: {x}")
</strong>(*
    Item #0: A
    Item #1: B
    Item #2: C
    val it: unit list = [(); (); ()]
*)

// ✅ Recommandé
<strong>["A"; "B"; "C"] |> List.iteri (fun i x -> printfn $"Item #{i}: {x}")
</strong>(*
    Item #0: A
    Item #1: B
    Item #2: C
    val it: unit = ()
*)
</code></pre>

### `choose`, `pick`, `tryPick`

Mapping `f: 'T -> 'U option` \
→ Opération partielle : peut échouer à produire une valeur \
→ D'où le type `Option` en sortie\
→ Exemple : `tryParseInt: string -> int option` _(cf. exemple ci-dessous)_

Les fonctions `choose`, `pick`, `tryPick` gèrent les `Option`s produites par le mapping des éléments de la collection, de manière à obtenir en sortie :&#x20;

* `choose  : 'U collection` \
  → toutes les valeurs que le mapping a réussi à produire
* `pick    : 'U` \
  → la 1ère valeur que le mapping a réussi à produire \
  → ou 💥 si aucune valeur n'a été produite
* `tryPick : 'U option` \
  → la même 1ère valeur wrappée dans `Some` \
  → ou `None` si aucune valeur n'a été produite

**Exemples :**

```fsharp
let tryParseInt (s: string) =
    match System.Int32.TryParse(s) with
    | true,  i -> Some i
    | false, _ -> None

["1"; "2"; "?"] |> List.choose tryParseInt   // [1; 2]
["1"; "2"; "?"] |> List.pick tryParseInt     // 1
["1"; "2"; "?"] |> List.tryPick tryParseInt  // Some 1
```

`choose` est équivalente conceptuellement à 3 opérations en 1 : \
1\. `map f` qui renvoie une collection de `'U option`\
2\. `filter (Option.isSome)` pour ne garder que les options contenant une valeur\
3\. `map (Option.get)` pour extraire ces valeurs

De même, `pick` et `tryPick` sont conceptuellement équivalentes à : \
→ `pick`  ≅ `choose f >> head`\
→ `tryPick`  ≅ `choose f >> tryHead`

## Sélection _vs_ mapping

* `filter` ou `choose` ?
* `find`/`tryFind` ou `pick`/`tryPick` ?

`filter`, `find`/`tryFind` opèrent avec un **prédicat** `'T -> bool`, sans mapping

`choose`, `pick`/`tryPick` opèrent avec un **mapping** `'T -> 'U option`

* `filter` ou `find`/`tryFind` ?
* `choose` ou `pick`/`tryPick` ?

`filter`, `choose` renvoient **tous** les éléments trouvés/mappés

`find`, `pick` ne renvoient que le **1er** élément trouvé/mappé

## Agrégation

### Fonctions spécialisées

| Opération | Sur élément | Sur projection 'T -> 'U |
| --------- | ----------- | ----------------------- |
| Maximum   | `max`       | `maxBy projection`      |
| Minimum   | `min`       | `minBy projection`      |
| Somme     | `sum`       | `sumBy projection`      |
| Moyenne   | `average`   | `averageBy projection`  |
| Décompte  | `length`    | `countBy projection`    |

```fsharp
[1; 2; 3] |> List.max  // 3
[ (1,"a"); (2,"b"); (3,"c") ] |> List.sumBy fst  // 6
[ (1,"a"); (2,"b"); (3,"c") ] |> List.map fst |> List.sum  // Equivalent explicite
```

#### Membre `Zero`

Les fonctions `sum` ne marchent que si le type des éléments dans la collection comporte un membre statique `Zero` et une surcharge de l'opérateur `+` :

```fsharp
type Point = Point of X:int * Y:int with
    static member Zero = Point (0, 0)
    static member (+) (Point (ax, ay), Point (bx, by)) = Point (ax + bx, ay + by)

let addition = (Point (1, 1)) + (Point (2, 2))
// val addition : Point = Point (3, 3)

let sum = [1..3] |> List.sumBy (fun i -> Point (i, -i))
// val sum : Point = Point (6, -6)
```

💡 On peut utiliser `LanguagePrimitives.GenericZero` pour récupérer le `Zero` d'un type :&#x20;

```fsharp
let zeroP: Point   = LanguagePrimitives.GenericZero // Point (0, 0)
let zeroI: int     = LanguagePrimitives.GenericZero // 0
let zeroF: float   = LanguagePrimitives.GenericZero // 0.0
let zeroM: decimal = LanguagePrimitives.GenericZero // 0M
let zeroS: string  = LanguagePrimitives.GenericZero // 💥 
// Le type 'string' ne prend pas en charge l'opérateur 'get_Zero' 
```

### Fonctions génériques

* `fold       (f: 'U -> 'T -> 'U) (seed: 'U) list`
* `foldBack   (f: 'T -> 'U -> 'U) list (seed: 'U)`
* `reduce     (f: 'T -> 'T -> 'T) list`
* `reduceBack (f: 'T -> 'T -> 'T) list`

☝ `f` prend 2 paramètres : un "accumulateur" `acc` et l'élément courant `x`

:warning: Fonctions `xxxBack` : tout est inversé / fonctions `xxx` !    \
→ Parcours des éléments en sens inverse : dernier → 1er élément    \
→ Paramètres `seed` et `list` inversés (pour `foldBack` _vs_ `fold`)    \
→ Paramètres `acc` et `x` de `f` inversés

💥 `reduceXxx` plante si liste vide car 1er élément utilisé en tant que _seed_

**Exemples :**

```fsharp
["a";"b";"c"] |> List.reduce (+)  // "abc"
[ 1; 2; 3 ] |> List.reduce ( * )  // 6

[1;2;3;4] |> List.reduce     (fun acc x -> 10 * acc + x)  // 1234
[1;2;3;4] |> List.reduceBack (fun x acc -> 10 * acc + x)  // 4321

("", [1;2;3;4]) ||> List.fold     (fun acc x -> $"{acc}{x}")  // "1234"
([1;2;3;4], "") ||> List.foldBack (fun x acc -> $"{acc}{x}")  // "4321"
```

## Changer l'ordre des éléments

| Opération        | Sur élément              | Sur projection 'T ->      |
| ---------------- | ------------------------ | ------------------------- |
| Inversion        | `rev list`               |                           |
| Tri ascendant    | `sort list`              | `sortBy f list`           |
| Tri descendant   | `sortDescending list`    | `sortDescendingBy f list` |
| Tri personnalisé | `sortWith comparer list` |                           |

```fsharp
[1..5] |> List.rev // [5; 4; 3; 2; 1]
[2; 4; 1; 3; 5] |> List.sort // [1..5]
["b1"; "c3"; "a2"] |> List.sortBy (fun x -> x.[0]) // ["a2"; "b1"; "c3"] cf. a < b < c
["b1"; "c3"; "a2"] |> List.sortBy (fun x -> x.[1]) // ["b1"; "a2"; "c3"] cf. 1 < 2 < 3
```

## Séparer

💡 Les éléments sont répartis en groupes.

Exemples en partant de `[1..10]`

<table><thead><tr><th width="197.33333333333331">Opération</th><th>Résultat (; omis)</th></tr></thead><tbody><tr><td><em>Valeur initiale</em></td><td><code>[ 1   2   3   4   5   6   7   8   9   10 ]</code></td></tr><tr><td><code>chunkBySize 3</code></td><td><code>[[1   2   3] [4   5   6] [7   8   9] [10]]</code></td></tr><tr><td><code>splitInto 3</code></td><td><code>[[1   2   3   4] [5   6   7] [8   9   10]]</code></td></tr><tr><td><code>splitAt 3</code></td><td><code>([1   2   3],[4   5   6   7   8   9   10])</code> Tuple ❗</td></tr></tbody></table>

## Grouper les éléments

### Par taille

💡 Les éléments peuvent être **dupliqués** dans différents groupes.

<table><thead><tr><th width="199">Opération</th><th>Résultat (' et ; omis)</th></tr></thead><tbody><tr><td><em>Valeur initiale</em></td><td><code>[ 1       2       3       4      5 ]</code></td></tr><tr><td><code>pairwise</code></td><td><code>[(1,2)   (2,3)   (3,4)   (4,5)     ]</code> Tuples ❗</td></tr><tr><td><code>windowed 2</code></td><td><code>[[1 2]   [2 3]   [3 4]   [4 5]     ]</code> Listes de 2</td></tr><tr><td><code>windowed 3</code></td><td><code>[[1 2 3] [2 3 4] [3 4 5]           ]</code> Listes de 3</td></tr></tbody></table>

### Par critère

<table><thead><tr><th width="163.33333333333331">Opération</th><th>Critère</th><th>Retour</th></tr></thead><tbody><tr><td><code>partition</code></td><td><code>predicate:  'T -> bool</code></td><td><code>('T list * 'T list)</code></td></tr><tr><td></td><td></td><td>→ 1 tuple <code>([OKs], [KOs])</code></td></tr><tr><td><code>groupBy</code></td><td><code>projection: 'T -> 'K</code></td><td><code>('K * 'T list) list</code></td></tr><tr><td></td><td></td><td>→ N tuples <code>[(clé, [éléments associés])]</code></td></tr></tbody></table>

```fsharp
let isOdd i = (i % 2 = 1)
[1..10] |> List.partition isOdd // (        [1; 3; 5; 7; 9] ,         [2; 4; 6; 8; 10]  )
[1..10] |> List.groupBy isOdd   // [ (true, [1; 3; 5; 7; 9]); (false, [2; 4; 6; 8; 10]) ]

let firstLetter (s: string) = s.[0]
["apple"; "alice"; "bob"; "carrot"] |> List.groupBy firstLetter
// [('a', ["apple"; "alice"]); ('b', ["bob"]); ('c', ["carrot"])]
```

## Changer de type de collection

Au choix : `Dest.ofSource` ou `Source.toDest`

| **De / vers** | `Array`        | `List`         | `Seq`         |
| ------------- | -------------- | -------------- | ------------- |
| `Array`       | ×              | `List.ofArray` | `Seq.ofArray` |
|               | ×              | `Array.toList` | `Array.toSeq` |
| `List`        | `Array.ofList` | ×              | `Seq.ofList`  |
|               | `List.toArray` | ×              | `List.toSeq`  |
| `Seq`         | `Array.ofSeq`  | `List.ofSeq`   | ×             |
|               | `Seq.toArray`  | `Seq.toList`   | ×             |

## Fonction _vs_ compréhension

Les fonctions de `List`/`Array`/`Seq` peuvent souvent être remplacées par une compréhension c'est-à-dire une expression de construction d'une liste `[...]`, d'un tableau `[|...|]` ou d'une séquence `seq {...}` respectivement. C'est une option à envisager pour améliorer la lisibilité.

Exemples pour les listes :

```fsharp
let list = [ 0..99 ]

list |> List.map f                   <->  [ for x in list do f x ]
list |> List.filter p                <->  [ for x in list do if p x then x ]
list |> List.filter p |> List.map f  <->  [ for x in list do if p x then f x ]
list |> List.collect g               <->  [ for x in list do yield! g x ]
```

## Ressources complémentaires

### Documentation de FSharp.Core 👍

* [Array module](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-arraymodule.html)
* [List module](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-listmodule.html)
* [Map module](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-mapmodule.html)
* [Seq module](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-seqmodule.html)
* [Set module](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-setmodule.html)
