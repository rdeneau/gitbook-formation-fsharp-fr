# Vue d'ensemble

## Types et Modules

Collections en F♯ avec leur module associé :

<table><thead><tr><th width="172">Type F♯</th><th>Module</th><th width="191">Equivalent BCL</th><th data-type="checkbox">Immutable</th><th data-type="checkbox">Comp. structurelle</th><th>Trié par</th><th data-hidden>Type F♯</th><th data-hidden>Type .NET équivalent</th><th data-hidden data-type="checkbox">Immutable</th><th data-hidden>Trié par</th></tr></thead><tbody><tr><td><code>'t array</code><br><em><code>'t[]</code></em><br><em><code>Array&#x3C;'t></code></em></td><td><code>Array</code></td><td>≡ <code>Array&#x3C;T></code></td><td>false</td><td>true</td><td>Ordre ajout</td><td><code>'T array</code><br><code>'T[]</code></td><td>≡ <code>Array&#x3C;T></code></td><td>false</td><td>Ordre ajout</td></tr><tr><td><code>'t list</code><br><em><code>List&#x3C;'t></code></em></td><td><code>List</code></td><td>≃ <code>Immutable-List&#x3C;T></code></td><td>true</td><td>true</td><td>Ordre ajout</td><td><code>'T list</code></td><td>≃ <code>Immutable-List&#x3C;T></code></td><td>true</td><td>Ordre ajout</td></tr><tr><td><code>ResizeArray</code><br><em>(alias)</em></td><td>❌</td><td>≡ <code>List&#x3C;T></code> ❕</td><td>false</td><td>false</td><td>Ordre ajout</td><td><code>ResizeArray</code></td><td>≡ <code>List&#x3C;T></code></td><td>false</td><td>Ordre ajout</td></tr><tr><td><code>'t seq</code><br><em><code>seq&#x3C;'t></code></em></td><td><code>Seq</code></td><td>≡ <code>IEnumerable&#x3C;T></code></td><td>true</td><td>true</td><td>Ordre ajout</td><td><code>seq&#x3C;'T></code></td><td>≡ <code>IEnumerable&#x3C;T></code></td><td>true</td><td>Ordre ajout</td></tr><tr><td><code>'t set</code><br><em><code>Set&#x3C;'t></code></em></td><td><code>Set</code></td><td>≃ <code>Immutable-HashSet&#x3C;T></code></td><td>true</td><td>true</td><td>Valeur</td><td><code>Set&#x3C;'T></code></td><td>≃ <code>Immutable-HashSet&#x3C;T></code></td><td>true</td><td>Valeur</td></tr><tr><td><code>Map&#x3C;'K, 'V></code></td><td><code>Map</code></td><td>≃ <code>Immutable-Dictionary&#x3C;K, V></code></td><td>true</td><td>true</td><td>Clé</td><td><code>Map&#x3C;'K, 'V></code></td><td>≃ <code>Immutable-Dictionary&#x3C;K, V></code></td><td>true</td><td>Clé</td></tr><tr><td><code>dict</code> <em>(cons)</em></td><td>❌</td><td>≡ <code>IDictionary&#x3C;K, V></code></td><td>false</td><td>false</td><td>Clé</td><td></td><td></td><td>false</td><td></td></tr><tr><td><code>readOnlyDict</code> <em>(cons)</em></td><td>❌</td><td>≡ <code>IReadOnly-Dictionary&#x3C;K, V></code></td><td>true</td><td>false</td><td>Clé</td><td></td><td></td><td>false</td><td></td></tr></tbody></table>

## Homogénéité des fonctions 👍

Communes aux 5 modules :

* `empty`/`isEmpty`, `exists`/`forall`
* `find`/`tryFind`, `pick`/`tryPick`, `contains` (`containsKey` pour `Map`)
* `map`/`iter`, `filter`, `fold`

Communes à `Array`, `List`, `Seq` :

* `append`/`concat`, `choose`, `collect`
* `item`, `head`, `last`
* `take`, `skip`
* ... _une centaine de fonctions en tout !_

## Homogénéité de la syntaxe 👍

| Type    | Éléments       | _Range_        | _Comprehension_ |
| ------- | -------------- | -------------- | --------------- |
| `Array` | `[∣ 1; 2 ∣]`   | `[∣ 1..5 ∣]`   | ...             |
| `List`  | `[ 1; 2 ]`     | `[ 1..5 ]`     | ...             |
| `Seq`   | `seq { 1; 2 }` | `seq { 1..5 }` | ...             |
| `Set`   | `set [ 1; 2 ]` | `set [ 1..5 ]` | ...             |

## Piège de la syntaxe :warning:

Les crochets `[]` sont utilisés pour :

* _Valeur_ : instance d'une liste `[ 1; 2 ]` (de type `int list`)
* _Type_ : tableau `int []`, par ex. de `[| 1; 2 |]`

☝ **Recommendations**

* Bien distinguer type _vs_ valeur ❗
* Préférer écrire `int array` plutôt que `int[]`
  * _N.B. En console FSI, le type affiché est encore `int []`_

## Création par _Comprehension_

* Syntaxe similaire à boucle `for`
* Même principe que générateurs en C♯, JS
  * Mot clé `yield` mais souvent optionnel (F♯ 4.7 / .NET Core 3)
  * Mot clé `yield!` ≡ `yield*` JS
  * Fonctionne pour toutes les collections 👍

**Exemples :**

```fsharp
// Syntaxes équivalentes
seq { for i in 1 .. 10 -> i * i }         // Plutôt obsolète
seq { for i in 1 .. 10 do yield i * i }   // 'yield' explicite
seq { for i in 1 .. 10 do i * i }         // 'yield' omis 👍

// Avec 'if'
let halfEvens =
    [ for i in [1..10] do
        if (i % 2) = 0 then i / 2 ]  // [1; 2; 3; 4; 5]

// 'for' imbriqués
let pairs =
    [ for i in [1..3] do
      for j in [1..3] do
        (i, j) ]              // [(1, 1); (1; 2); (1; 3); (2, 1); ... (3, 3)]

// Même ici les 'yield' peuvent être omis 👍
let twoToNine =
    [ for i in [1; 4; 7] do
        if i > 1 then i
        i + 1
        i + 2 ]  // [2; 3; 4; 5; 6; 7; 8; 9]
```

`yield!` permet d'aplatir des collections imbriquées :

```fsharp
let oneToSix =
    [ for i in [1; 3; 5] do
        yield! set [i; i+1] ]
```
