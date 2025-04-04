# Tuples

## Points clés

* Types à valeurs littérales
* Types "anonymes" mais on peut leur définir des alias
* Types produit par excellence
  * Signe `*` dans signature `A * B`
  * **Produit cartésien** des ensembles de valeurs de A et de B
* Nombre d'éléments:
  * 👌 2 ou 3 (`A * B * C`)
  * :warning: > 3 : possible mais préférer _Record_
* Ordre des éléments est important
  * Si `A` ≠ `B`, alors `A * B` ≠ `B * A`

## Construction

* Syntaxe des littéraux : `a,b` ou `a, b` ou `(a, b)`
  * Virgule `,` caractéristique des tuples
  * Espaces optionnels
  * Parenthèses `()` peuvent être nécessaires
* :warning: Piège : séparateur différent entre un littéral et sa signature
  * `,` pour littéral
  * `*` pour signature
  * Ex : `true, 1.2` → `bool * float`

## Déconstruction

👍 Même syntaxe que construction

:warning: Tous les éléments doivent apparaître dans la déconstruction\
→ 💡 Utiliser la discard `_` pour ignorer l'un des éléments

```fsharp
let point = 1.0,2.5
let x, y = point

let x, y = 1, 2, 3 // 💥 Erreur FS0001: Incompatibilité de type...
                   // ... Les tuples ont des longueurs différentes de 2 et 3

let result = System.Int32.TryParse("123") // (bool * int)
let _, value = result // Ignore le "bool"
```

## En pratique

Utiliser un tuple pour une structure de données :

* Petite : 2 à 3 éléments
* Légère : pas besoin de nom pour les éléments
* Locale : échange local de données qui n'intéresse pas toute la _codebase_
  * Renvoyer plusieurs valeurs - cf. `Int32.TryParse`

Tuple immuable : les modifications se font en créant un nouveau tuple

```fsharp
let addOneToTuple (x,y,z) = (x+1,y+1,z+1)
```

**Égalité structurelle**, mais uniquement entre 2 tuples de même signature !

```fsharp
(1,2) = (1,2)       // true
(1,2) = (0,0)       // false
(1,2) = (1,2,3)     // 💥 Erreur FS0001: Incompatibilité de type...
                    // ... Les tuples ont des longueurs différentes de 2 et 3
(1,2) = (1,(2,3))   // 💥 Erreur FS0001: Cette expression était censée avoir le type `int`
                    // ... mais elle a ici le type `'a * 'b`
```

**Imbrication** de tuples grâce aux `()`

```fsharp
let doublet = (true,1), (false,"a")     // (bool * int) * (bool * string) → pair de pairs
let quadruplet = true, 1, false, "a"    // bool * int * bool * string     → quadruplet
doublet = quadruplet                    // 💥 Erreur FS0001: Incompatibilité de type...
```

## Pattern matching

Patterns reconnus avec les tuples :

```fsharp
let print move =
    match move with
    | 0, 0 -> "No move"                   // Constante 0
    | 0, y -> $"Vertical {y}"             // Variable y (!= 0)
    | x, 0 -> $"Horizontal {x}"
    | x, y when x = y -> $"Diagonal {x}"  // Condition x et y égaux
                                          // Car pattern `x, x` pas possible ❗
    | x, y -> $"Other ({x}, {y})"
```

☝ **Notes :**

* Les patterns sont à ordonner du + spécifique au + générique
* Le dernier pattern `(x, y)` correspond au pattern par défaut (obligatoire)

## Paires

* Tuples à 2 éléments
* Tellement courant que 2 helpers leur sont associés :
  * `fst` comme _first_ pour extraire le 1° élément de la paire
  * `snd` comme _second_ pour extraire le 2° élément de la paire
  * :warning:️ Ne marche que pour les paires

```fsharp
let pair = ('a', "b")
fst pair  // 'a' (char)
snd pair  // "b" (string)
```

## Quiz 🕹️

**1. Comment implémenter soi-même `fst` et `snd` ?**

```fsharp
let fst ... ?
let snd ... ?
```

**2. Quelle est la signature de cette fonction ?**

```fsharp
let toList (x, y) = [x; y]
```

### Réponse

**1. Implémenter soi-même `fst` et `snd` ?**

```fsharp
let inline fst (x, _) = x  // Signature : 'a * 'b -> 'a
let inline snd (_, y) = y  // Signature : 'a * 'b -> 'b
```

* Déconstruction avec _discard_, le tout entre `()`
* Fonctions peuvent être `inline`

**2. Signature de `toList` ?**

```fsharp
let inline toList (x, y) = [x; y]
```

* Renvoie une liste avec les 2 éléments de la paire
* Les éléments sont donc du même type
* Ce type est quelconque → générique `'a`

**Réponse :** la signature est `x: 'a * y: 'a -> 'a list` \
ce qui correspond au type `'a * 'a -> 'a list`

## Tuple `struct`

* Littéral : `struct(1, 'b', "trois")`
* Signature : `struct (int * char * string)`
* Déconstruction: `let struct(i, _, _) = myStructTuple`
* Usage : optimiser performance

🔗 [https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/conventions #performance](https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/conventions#performance)
