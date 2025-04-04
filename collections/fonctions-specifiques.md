---
description: Fonctions propres à un type
---

# Fonctions spécifiques

## Module `Map`

Ce module comporte quelques fonctions spécifiques qui sont intéressantes à connaître :

### `Map.change` : modification intelligente

Signature : `Map.change key (f: 'T option -> 'T option) table`

Selon la fonction `f` passée en argument, on peut : → Ajouter, modifier ou supprimer l'élément d'une clé donnée

| Entrée                                                        | Renvoie `None`                                                | Renvoie `Some newVal`                                                                     |
| ------------------------------------------------------------- | ------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| <p><code>None</code><br><em>(Élément absent)</em></p>         | Ignore cette clé                                              | <p>Ajoute l'élément <em>(key, newVal)</em><br>≡ <code>Map.add key newVal table</code></p> |
| <p><code>Some value</code><br><em>(Élément existant)</em></p> | <p>Supprime la clé<br>≡ <code>Map.remove key table</code></p> | <p>Passe la valeur à <em>newVal</em><br>≡ <code>Map.add key newVal table</code></p>       |

### `containsKey` _vs_ `exists` _vs_ `filter`

```
Fonction      Signature                        Commentaire                                             
------------+--------------------------------+---------------------------------------------------------
containsKey   'K -> Map<'K,'V> -> bool         Indique si la clé est présente                          
exists         f -> Map<'K,'V> -> bool         Indique si un couple clé/valeur satisfait le prédicat   
filter         f -> Map<'K,'V> -> Map<'K,'V>   Conserve les couples clé/valeur satisfaisant le prédicat

Avec prédicat f: 'K -> 'V -> bool
```

```fsharp
let table = Map [ (2, "A"); (1, "B"); (3, "D") ]

table |> Map.containsKey 0  // false
table |> Map.containsKey 2  // true

let isEven i = i % 2 = 0
let isFigure (s: string) = "AEIOUY".Contains(s)

table |> Map.exists (fun k v -> (isEven k) && (isFigure v))  // true
table |> Map.filter (fun k v -> (isEven k) && (isFigure v))  // map [(2, "A")]
```

## Module `Seq`

Ce module comporte une fonction spécifique très intéressante :

### `Seq.cache`

Le principe d'une séquence (aka énumérable) est qu'elle est reconstruit à chaque fois qu'elle est parcourue. Cette reconstruction peut être coûteuse. Un algorithme parcourant (même partiellement) plusieurs fois une séquence invariante peut être optimisé en mettant en cache cette séquence au moyen de la fonction `Seq.cache`.&#x20;

Signature : `Seq.cache: source: 'T seq -> 'T seq`

La mise en cache est optimisée en étant différée et en ne se faisant que sur les éléments parcourus.

## Type `string`

Pour manipuler les caractères d'une chaîne, plusieurs options sont possibles.

### Module `Seq`

Le type `string` implémente `Seq<char>` (i.e. `System.Collections.Generic.IEnumerable<char>`). \
→ On peut donc utiliser les fonctions du module `Seq`.

### Module `Array`

La méthode d'instance `ToCharArray()` permet de renvoyer les caractères d'une chaîne sous la forme d'un `char array`.\
→ On peut donc utiliser les fonctions du module `Array`.

### Module `String`

Il existe également un module `String` (venant de `FSharp.Core`) proposant quelques fonctions intéressantes car + performantes individuellement que celles de `Seq` et `Array` :&#x20;

→ Module `(FSharp.Core.)String` (≠ `System.String`)

→ Propose quelques fonctions similaires à celles de `Seq` en + performantes :

```fsharp
String.concat (separator: string) (strings: seq<string>) : string
String.init (count: int) (f: (index: int) -> string) : string
String.replicate (count: int) (s: string) : string

String.exists (predicate: char -> bool) (s: string) : bool
String.forall (predicate: char -> bool) (s: string) : bool
String.filter (predicate: char -> bool) (s: string) : string

String.collect (mapping:        char -> string) (s: string) : string
String.map     (mapping:        char -> char)   (s: string) : string
String.mapi    (mapping: int -> char -> char)   (s: string) : string
// Idem iter/iteri qui renvoie unit
```

**Exemples :**

```fsharp
let a = String.concat "-" ["a"; "b"; "c"]  // "a-b-c"
let b = String.init 3 (fun i -> $"#{i}")   // "#0#1#2"
let c = String.replicate 3 "0"             // "000"

let d = "abcd" |> String.exists (fun c -> c >= 'b')  // true
let e = "abcd" |> String.forall (fun c -> c >= 'b')  // false
let f = "abcd" |> String.filter (fun c -> c >= 'b')  // "bcd"

let g = "abcd" |> String.collect (fun c -> $"{c}{c}")  // "aabbccdd"

let h = "abcd" |> String.map (fun c -> (int c) + 1 |> char)  // "bcde"
```

☝️ **Note :** lorsque l'on a fait un `open System`, `String` représente 3 choses que le compilateur arrive à différencier :&#x20;

* Les constructeurs `(new) String(...)`.
* `String.` donne accès à l'ensemble des fonctions du module `String`  (en _camelCase_) et des méthodes statiques de `System.String` (en _PascalCase_).

## Array2D

Au lieu de fonctionner avec une imbrication de collections à N niveaux, F# propose des tableaux multidimensionnels (également appelés matrices). En revanche, pour les créer, il n'y a pas de syntaxe spécifique, il faut passer par une fonction de création. Voyons cela pour les tableaux de dimension 2.

### Type

Le type d'un tableau à 2 dimensions est `'t[,]` mais l'IntelliSense donne parfois juste `'t array` qui est moins précis, n'indiquant plus le nombre de dimensions.

### Création

`array2D` permet de créer un tableau à 2 dimensions à partir d'une collection de collections ayant toutes la même longueur.

`Array2D.init` permet de générer un tableau en spécifiant : \
→ Sa longueur selon les 2 dimensions N et P\
→ Une fonction générant l'élément aux deux indices spécifiés.

```fsharp
let rows = [1..3]
let cols = ['A'..'C']

let cell row col = $"%c{col}%i{row}"
let rowCells row = cols |> List.map (cell row)

let list = rows |> List.map rowCells
// val arr: string list list =
//   [["A1"; "B1"; "C1"]; ["A2"; "B2"; "C2"]; ["A3"; "B3"; "C3"]]

let matrix = array2D list
// val matrix: string[,] = [["A1"; "B1"; "C1"]
//                          ["A2"; "B2"; "C2"]
//                          ["A3"; "B3"; "C3"]]

let matrix' = Array2D.init 3 3 (fun i j -> cell i cols[j])
```

### Accès par indices

```fsharp
let a1  = list[0][0]    // "A1" // arr.[0].[0] avant F# 6
let a1' = matrix[0,0]  // "A1" // matrix.[0,0] avant F# 6
```

### Longueurs

`Array2D.length1` et `Array2D.length2` renvoient le nombre de lignes et de colonnes.

```fsharp
let rowCount =
    list.Length,
    list |> List.length,
    matrix |> Array2D.length1
// val rowCount: int * int * int = (3, 3, 3)

let columnCount =
    list[0] |> List.length,
    matrix |> Array2D.length2
// val columnCount: int * int = (3, 3)
```

### Slicing

Syntaxe permettant de ne prendre qu'une tranche horizontale et/ou verticale au moyen de l'opérateur `..`  et du caractère wilcard `*` pour prendre tous les indices

```fsharp
let m1 = matrix[1.., *]  // Remove first row: A2..C3
let m2 = matrix[0..1, *] // Remove last row : A1..C2
let m3 = matrix[*, 1..]  // Remove first column: B1..C3
let m4 = matrix[*, 0]    // First column: [|"A1"; "A2"; "A3"|]
                         // 💡 1 dimension array (i.e. vector)
```

Cette fonctionnalité offre beaucoup de souplesse. Comme pour une collection de collections, une matrice permet un accès par ligne (`matrix[row, *]` vs `list[row]`). Une matrice autorise en plus un accès direct par colonne (`matrix[*, col]`), ce qui n'est possible pas avec une collection de collections sans la transposer au préalable - cf. fonction `transpose` ([doc](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-listmodule.html#transpose)).

### Autres utilisations

```fsharp
// Mapping avec indices
let doubleNotations =
    matrix
    |> Array2D.mapi (fun row col value -> $"{value} => [R{row+1}C{col+1}]")
//   [["A1 => [R1C1]"; "B1 => [R1C2]"; "C1 => [R1C3]"]
//    ["A2 => [R2C1]"; "B2 => [R2C2]"; "C2 => [R2C3]"]
//    ["A3 => [R3C1]"; "B3 => [R3C2]"; "C3 => [R3C3]"]]

```

Autres fonctions :\
🔗 [https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-array2dmodule.html](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-array2dmodule.html)
