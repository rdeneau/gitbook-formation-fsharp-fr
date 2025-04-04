# Types

## Type `List`

Implémentée sous forme de **liste simplement chaînée** : → 1 liste = 1 élément _(Head)_ + 1 sous-liste _(Tail)_ → Construction nommée _Cons_ et notée `::`

Pour éviter récursion infinie, besoin d'un cas de "sortie" : → Liste vide nommée _Empty_ et notée `[]`

👉 **Type union générique et récursif** :

```fsharp
type List<'T> =
  | ( [] )
  | ( :: ) of head: 'T * tail: List<'T>
```

### Alias de types

* Par défaut, `List` fait référence au type (ou au module) F#
* `list` est un alias du type `List` de F#, utilisé souvent avec la notation OCaml : \
  → `let l : string list = ...`
* Si on fait un `open System.Collections.Generic`, `List` fera référence à la liste mutable C# et masquera le type `List` F# mais sera fusionné avec le module `List` F# !
* Pour éviter cela, on peut utiliser l'alias `ResizeArray` pour référencer la liste mutable C# directement, sans devoir faire `open System.Collections.Generic`.

### Littéraux

| Nb | Notation    | Notation explicite  | Signification                       |
| -- | ----------- | ------------------- | ----------------------------------- |
| 0  | `[]`        | `[]`                | Empty                               |
| 1  | `[1]`       | `1 :: []`           | Cons (1, Empty)                     |
| 2  | `[2; 1]`    | `2 :: 1 :: []`      | Cons (2, Cons (1, Empty))           |
| 3  | `[3; 2; 1]` | `3 :: 2 :: 1 :: []` | Cons (3, Cons (2, Cons (1, Empty))) |

Vérification par décompilation avec [SharpLab.io](https://sharplab.io/#v2:DYLgZgzgPsCmAuACAbgBkQXkQbQLoFgAoOJZARkxzIOIRQCZLt6BuRaoklAZie7dbsaRIA==) :

```cs
//...
v1@2 = FSharpList<int>.Cons(1, FSharpList<int>.Empty);
v2@3 = FSharpList<int>.Cons(2, FSharpList<int>.Cons(1, FSharpList<int>.Empty));
//...
```

### Immuable

Il n'est pas possible de modifier une liste existante. \
→ C'est cela qui permet de l'implémenter en liste chaînée.

💡 L'idée est de créer une nouvelle liste pour signifier un changement.\
&#x20;→ Utiliser les opérateurs _Cons_ (`::`) et _Append_ (`@`) 📍

### Initialisation

```fsharp
// Range: Start..End (Step=1)
let numFromOneToFive = [1..5]     // [1; 2; 3; 4; 5]

// Range: Start..Step..End
let oddFromOneToNine = [1..2..9]  // [1; 3; 5; 7; 9]

// Comprehension
let pairs =
    [ for i in [1..3] do
      for j in [1..3] do
        (i, j) ]              // [(1, 1); (1; 2); (1; 3); (2, 1); ... (3, 3)]
```

### Exercices 🕹️

#### **1.** Implémenter la fonction `rev`

Inverse une liste : `rev [1; 2; 3]` ≡ `[3; 2; 1]`

#### **2.** Implémenter la fonction `map`

Transforme chaque élément : `[1; 2; 3] |> map ((+) 1)` ≡ `[2; 3; 4]`

💡 **Astuces**\
→ Pattern matching liste vide `[]` ou _Cons_ `head :: tail` \
→ Sous-fonction _(tail-) recursive_

#### Solution 🎲

```fsharp
let rev list =
    let rec loop acc rest =
        match rest with
        | [] -> acc
        | x :: xs -> loop (x :: acc) xs
    loop [] list

let map f list =
    let rec loop acc rest =
        match rest with
        | [] -> acc
        | x :: xs -> loop (f x :: acc) xs
    list |> loop [] |> rev
```

💡 Vérification avec [sharplab.io](https://sharplab.io/#v2:DYLgZgzgNAJiDUAfYBTALgAgE4oG4eAEsJMBeAWACgMaD1sUBjAgexYAcMBDR5nEjBWq0RAWy5pGACwYCA7oTRSqIkYgwBtALoYAtAD5uvFaprqAHhhAgM5iHsPA2nABSXrRxgEpbEEzScOTR0iEiogA) de la _tail recursion_ compilée en boucle `while`

#### Vérification en console FSI ✅

```fsharp
// Tests en console FSI
let (=!) actual expected =
    if actual = expected
    then printfn $"✅ {actual}"
    else printfn $"❌ {actual} != {expected}"

[1..3] |> rev =! [3; 2; 1];;
// ✅ [3; 2; 1]

[1..3] |> map ((+) 1) =! [2; 3; 4];;
// ✅ [2; 3; 4]
```

## Type `Array`

* Différences / `List` : mutable, taille fixe, accès indexé en O(1)
* Signature générique : `'T array` _(récemment recommandée)_ ou `'T []`
* Littéral et _comprehension_ : similaires à `List`

```fsharp
// Littéral
[| 1; 2; 3; 4; 5 |]  // val it : int [] = [|1; 2; 3; 4; 5|]

// Comprehension using range
[| 1 .. 5 |] = [| 1; 2; 3; 4; 5 |]  // true
[| 1 .. 3 .. 10 |] = [| 1; 4; 7; 10 |] // true

// Comprehension using generator
[| for a in 1 .. 5 do (a, a * 2) |]
// [|(1, 2); (2, 4); (3, 6); (4, 8); (5, 10)|]
```

### Accès indexé & mutation

Accès par index : `my-array.[my-index]`

:warning: **Piège :** ne pas oublier le `.` avant les crochets `[]` ❗\
🎁 **F♯ 6.0** supporte sans le `.` : `my-array[my-index]`

```fsharp
let names = [| "Juliet"; "Monique"; "Rachelle"; "Tara"; "Sophia" |]
names.[4] <- "Kristen" // "Rachelle"
names    // [| "Juliet"; "Monique"; "Rachelle"; "Tara"; "Kristen" |]
         //                                              ^^^^^^^
```

### _Slicing_

Renvoie un sous-tableau entre les indices `start..end` optionnels

```fsharp
let names = [|"0: Juliet"; "1: Monique"; "2: Rachelle"; "3: Tara"; "4: Sophia"|]

names.[1..3]  // [|"1: Monique"; "2: Rachelle"; "3: Tara"|]
names.[2..]   // [|"2: Rachelle"; "3: Tara"; "4: Sophia"|]
names.[..3]   // [|"0: Juliet"; "1: Monique"; "2: Rachelle"; "3: Tara"|]
```

💡 Marche aussi avec une `string` : `"012345".[1..3]` ≡ `"123"`

## Alias `ResizeArray`

☝️ Ne pas confondre le type `List<T>` de F♯ avec celui de  .NET dans le namespace [System.Collections.Generic](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic?view=net-8.0).

Même si les listes .NET ne sont pas très souvent utilisées en F♯, quand on en a besoin (par exemple en interop), il vaut mieux utiliser l'alias `ResizeArray` qui permet d'éviter la confusion avec les listes F♯.

```fsharp
let rev items = items |> Seq.rev |> ResizeArray
let initial = ResizeArray [ 1..5 ]
let reversed = rev initial // ResizeArray [ 5..-1..0 ]
```

☝️ Ne pas confondre l'alias `ResizeArray` avec le type `Array` de F♯.

## Type `Seq`

`type Seq<'T> = IEnumerable<'T>` → Série d'éléments de même type

_Lazy_ : séquence construite au fur et à mesure lors de son itération ≠ `List` construite dès la déclaration

→ Peut offrir de meilleures performances qu'un `List` pour une collection avec beaucoup d'éléments et qu'on ne souhaite pas parcourir entièrement.

### Syntaxe

`seq { comprehension }`

```fsharp
seq { yield 1; yield 2 }   // 'yield' explicites 😕
seq { 1; 2; 3; 5; 8; 13 }  // 'yield' implicites 👍

// Range
seq { 1 .. 10 }       // seq [1; 2; 3; 4; ...]
seq { 1 .. 2 .. 10 }  // seq [1; 3; 5; 7; ...]

// Générateur
seq { for a in 1 .. 5 do (a, a * 2) }
// seq [(1, 2); (2, 4); (3, 6); (4, 8); ...]
```

### Séquence infinie

**Option 1** : appeler la fonction `Seq.initInfinite` : → `Seq.initInfinite : (initializer: (index: int) -> 'T) -> seq<'T>` → Paramètre `initializer` sert à créer l'élément d'index (>= 0) spécifié

**Option 2** : écrire une fonction récursive générant la séquence

```fsharp
// Option 1
let seqOfSquares = Seq.initInfinite (fun i -> i * i)

// Option 2
let seqOfSquares' =
    let rec loop n = seq { yield n * n; yield! loop (n+1) }
    loop 0

// Test
let firstTenSquares = seqOfSquares |> Seq.take 5 |> List.ofSeq // [0; 1; 4; 9; 16]
```

## Type `Set`

Collection auto-ordonnée d'éléments uniques _(sans doublon)_ → Implémentée sous forme d'arbre binaire

```fsharp
// Création
set [ 2; 9; 4; 2 ]          // set [2; 4; 9]  // ☝ Élément 2 dédoublonné
Set.ofArray [| 1; 3 |]      // set [1; 3]
Set.ofList [ 1; 3 ]         // set [1; 3]
seq { 1; 3 } |> Set.ofSeq   // set [1; 3]

// Ajout/retrait d'élément
Set.empty         // set []
|> Set.add 2      // set [2]
|> Set.remove 9   // set [2]    // ☝ Pas d'exception
|> Set.add 9      // set [2; 9]
|> Set.remove 9   // set [2]
```

### Informations

→ `count`, `minElement`, `maxElement`

```fsharp
let oneToFive = set [1..5]          // set [1; 2; 3; 4; 5]

// Nombre d'éléments : propriété `Count` ou fonction `Set.count` - ⚠️ O(N)
// ☝ Ne pas confondre avec `Xxx.length` pour Array, List, Seq
let nb = Set.count oneToFive  // 5

// Élément min, max
let min = oneToFive |> Set.minElement   // 1
let max = oneToFive |> Set.maxElement   // 5
```

### Opérations

→ **Union**, **Différence**, **Intersection** _(idem ensembles en Math)_

| Opération    | ? | Opérateur | Fonction 2 sets  | Fonction N sets      |
| ------------ | - | --------- | ---------------- | -------------------- |
| Union        | ∪ | `+`       | `Set.union`      | `Set.unionMany`      |
| Différence   | ⊖ | `-`       | `Set.difference` | `Set.differenceMany` |
| Intersection | ∩ |           | `Set.intersect`  | `Set.intersectMany`  |

**Exemples :**

```fsharp
let oneToFive = set [1..5]                 // A - set [1; 2; 3; 4; 5]
let evenToSix = set [2; 4; 6]              // B - set [2; 4; 6]

let union = oneToFive + evenToSix              // set [1; 2; 3; 4; 5; 6]
let diff  = oneToFive - evenToSix              // set [1; 3; 5]
let inter = Set.intersect oneToFive evenToSix  // set [2; 4]
```

| Valeur   | Union                                        | Différence                                   | Intersection                                 |
| -------- | -------------------------------------------- | -------------------------------------------- | -------------------------------------------- |
| A        | `[ 1 2 3 4 5   ]`                            | `[ 1 2 3 4 5   ]`                            | `[ 1 2 3 4 5   ]`                            |
| B        | `[   2   4   6 ]`                            | `[   2   4   6 ]`                            | `[   2   4   6 ]`                            |
| Résultat | <p>A + B<br><code>[ 1 2 3 4 5 6 ]</code></p> | <p>A - B<br><code>[ 1   3   5   ]</code></p> | <p>A ∩ B<br><code>[   2   4     ]</code></p> |

## Type `Map`

Tableau associatif { _Clé_ → _Valeur_ } ≃ `Dictionary` immutable en C♯

```fsharp
// Création : depuis collection de tuples (key, val)
// → Fonction `Map.ofXxx` (Array, List, Seq)
let map1 = seq { (2, "A"); (1, "B") } |> Map.ofSeq
// → Constructeur `Map(tuples)`
let map2 = Map [ (2, "A"); (1, "B"); (3, "C"); (3, "D") ]
// map [(1, "B"); (2, "A"); (3, "D")]
// 👉 Ordonnés par clés (1, 2, 3) et dédoublonnés en last win - cf. { 3 → "D" }

// Ajout/retrait d'élément
Map.empty         // map []
|> Map.add 2 "A"  // map [(2, "A")]
|> Map.remove 5   // map [(2, "A")] // ☝ Pas d'exception si clé absente
|> Map.add 9 "B"  // map [(2, "A"); (9, "B")]
|> Map.remove 2   // map [(9, "B")]
```

### Accès par clé

```fsharp
let table = Map [ (2, "A"); (1, "B"); (3, "D") ]

// Syntaxe `.[key]` ou juste `[key]` depuis F♯ 6
table.[1]  // "B"  // ⚠️ `1` est bien une clé et pas un indice
table.[0]  // 💥 KeyNotFoundException

// Fonction `Map.find` : renvoie valeur ou 💥 si clé absente
table |> Map.find 3     // "D"

// Fonction `Map.tryFind` : renvoie `'V option`
table |> Map.tryFind 3  // Some "D"
table |> Map.tryFind 9  // None
```

## Dictionnaires

### `dict` function

Construit un `IDictionary<'k, 'v>` immutable ❕ à partir d'une séquence de pairs clé/valeur

```fsharp
let table = dict [ (1, 100); (2, 200) ] // System.Collections.Generic.IDictionary<int,int>

table[1]  // → 100

table[99] // 💣 KeyNotFoundException

table[1] <- 111 // 💣 NotSupportedException: This value cannot be mutated
table.Add(3, 300) // 💣 NotSupportedException: This value cannot be mutated
```

### `readOnlyDict` function

Construit un `IReadOnlyDictionary<'k, 'v>` à partir d'une séquence de pairs clé/valeur

```fsharp
let table = readOnlyDict [ (1, 100); (2, 200) ]
table[1]  // 100
table[99] // 💣 KeyNotFoundException

// Ne compile pas :
table[1] <- 111   // ⚠️ Property 'Item' cannot be set (FS810)
table.Add(3, 300) // ⚠️ The type 'IReadOnlyDictionary<_,_>' does not define the field, constructor or member 'Add' (FS39)
```

> 👉 **Recommendation**\
> Préférer `readOnlyDict` à `dict` car l'interface `IReadOnlyDictionary<'k, 'v>` est + fidèle à l'implémentation sous-jacente. Au contraire, `dict` ne respecte pas le principe de substitution de Liskov !

### `KeyValue` active pattern

* Permet de déconstruire une entrée `KeyValuePair` d'un dictionnaire \
  en passant par un tuple `(clé, valeur)`
*   Signature: &#x20;

    ```fsharp
    ( |KeyValue| ) : KeyValuePair<'Key,'Value> -> 'Key * 'Value
    ```

```fsharp
let table =
    readOnlyDict
        [ (1, 100)
          (2, 200)
          (3, 300) ]

// Parcourt du dictionnaire
for kv in table do // kv: KeyValuePair<int,int>
    printfn $"{kv.Key}, {kv.Value}"

// De manière + élégante avec l'active pattern
for KeyValue (key, value) in table do
    printfn $"{key}, {value}"
```

## Performance des lookups (`find`)

🔗 [High Performance Collections in F#](https://www.compositional-it.com/news-blog/high-performance-hash-tables-for-f/), Compositional IT, Jan 2021

### `Map` vs `Dictionary`

Fonction `readOnlyDict` permet de créer rapidement un `IReadOnlyDictionary` → à partir d'une séquence de tuples `key, item` → très performant : 10x plus rapide que `Map` pour le _lookup_

### `Dictionary` vs `Array`

→ `Array` suffit si peu de lookups (< 100) et peu d'éléments (< 100) \
→ `Dictionary` sinon

## `Set` et `Map` _vs_ `IComparable`

Ne marchent que si les éléments (d'un `Set`) ou les clés (d'une `Map`) sont **comparables** !

🎉 Compatibles avec tous les types F♯ _(cf. égalité et comparaison structurelles)_

:warning: Pour les classes : implémenter `IComparable` _(mais pas `IComparable<'T>`)_
