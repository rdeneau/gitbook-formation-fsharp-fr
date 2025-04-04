# Premiers concepts

## Expression _vs_ Instruction _(Statement)_

> Une **instruction** produit un effet de bord. \
> Une **expression** produit une valeur... et un éventuel effet de bord **(à éviter** toutefoi&#x73;**)**.

* F♯ est un langage fonctionnel, à base **d'expressions** uniquement.
* C♯ est un langage impératif, à base **d'instructions** _(statements)_ mais comporte de + en + de sucre syntaxique à base d'expressions :
  * Opérateur ternaire `b ? x : y`
  * [Null-conditional operator](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#null-conditional-operators--and-) `?.` en C♯ 6 : `model?.name`
  * [Null-coalescing operator](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator) `??` en C♯ 8 : `label ?? '(Vide)'`
  * Expression lambda en C♯ 3 avec LINQ : `numbers.Select(x => x + 1)`
  * [Expression-bodied members](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members) en C♯ 6 et 7
  * Expression `switch` en C♯ 8

### ⚖️ Avantages des expressions / instructions

* **Concision** : code + compact == + lisible
* **Composabilité** : composer expressions == composer valeurs
  * Addition, multiplication... de nombres,
  * Concaténation dans une chaîne,
  * Collecte dans une liste...
* **Compréhension** : pas besoin de connaître les instructions précédentes
* **Testabilité** : expressions pures _(sans effet de bord)_ + facile à tester
  * _Prédictible_ : même inputs produisent même outputs
  * _Isolée_ : phase _arrange/setup_ allégée _(pas de mock...)_

## En F♯ « Tout est expression »

* Une fonction se déclare et se comporte comme une valeur
  * En paramètre ou en sortie d'une autre fonction _(dite fonction d'ordre supérieur, high-order function en anglais)_
* Éléments du _control flow_ sont aussi des expressions
  * `if/else` et expression `match` (`~=switch`) renvoient une valeur.
  * Bloc `for` aussi : renvoie juste "rien" i.e. `unit` :round\_pushpin:[#de-void-a-unit](../fonctions/signature.md#de-void-a-unit "mention")

**Conséquences**

* Pas de `void` \
  → Remplacé avantageusement par le type `unit` ayant 1! valeur notée `()` 📍[type-unit.md](../types-complements/type-unit.md "mention")
* Absence de _**Early Exit**_
  * En C#, on peut sortir d'une fonction avec `return` et sortir d'une boucle `for/while` avec `break`&#x20;
  * En F# on n'a pas ces mot-clés dans le langage ❌

### Solutions à l'absence de _early exit_

La solution la plus discutable consiste à émettre une exception 💩 (cf. [réponse StackOverflow](https://stackoverflow.com/a/42018355/8634147))

Une solution en style impératif consiste à utiliser des variables mutables 😕

```fsharp
let firstItemOrDefault defaultValue predicate (items: 't array) =
    let mutable result = None
    let mutable i = 0
    while i < items.Length && result.IsNone do
        let item = items[i]
        if predicate item then
            result <- Some item
        i <- i + 1

    result
    |> Option.defaultValue defaultValue

let test1' = firstItemOrDefault -1 (fun x -> x > 5) [| 1 |]     // -1
```

La solution la plus recommandée et la plus idiomatique en programmation fonctionnelle est d'utiliser une fonction récursive 📍[#fonction-recursive](../fonctions/fonctions.md#fonction-recursive "mention")

* _On décide de continuer la "boucle" en rappelant la fonction_

```fsharp
[<TailCall>] // 👈 F# 8
let rec firstOr defaultValue predicate list =
    match list with
    | [] -> defaultValue                                // 👈 Sortie
    | x :: _ when predicate x -> x                      // 👈 Sortie
    | _ :: rest -> firstOr defaultValue predicate rest  // 👈 Appel récursif → continue

let test1 = firstOr -1 (fun x -> x > 5) [1]     // -1
let test2 = firstOr -1 (fun x -> x > 5) [1; 6]  // 6
```

&#x20;💡 Comme l'indique l'attribut `TailCall` (introduit en F# 8), on est dans un cas de [récursion terminale](https://fr.wikipedia.org/wiki/R%C3%A9cursion_terminale) : le compilateur va convertir l'appel à cette fonction récursive en simple boucle beaucoup plus performante. On peut le vérifier dans [SharpLab](https://sharplab.io/#v2:DYLgZgzgNAJiDUAfA2gHgCoEMCWwDCmwwAfALoCwAUMAKYAuABAE40DGDY2TEdA8kwxg0wmAK7A6ANUKiaDAA4sY2Vpjpzg2HgwC8VBgYYBbNawAWDTdoDu2Omf2HEDZKQYBaYoOFiJ04LKGQcEhoaEA9OEMgLwbgBI7DADKAPZMdNg0jgbOAB4MICAMAPoM1mY0AHYKSipqcrmeDLlhzUGRMfHJqemZDM7F+cw02g2c3HwCQiLiUjJyijTKquqD2gxtcQwAgvLyNMDMAJesotzYYAyASYQMrEnlaeWyVEA=).

## Typage, inférence et cérémonie

Poids de la cérémonie ≠ Force du typage → Cf. https://blog.ploeh.dk/2019/12/16/zone-of-ceremony/

| Lang | Force du typage                  | Inférence | Cérémonie |
| ---- | -------------------------------- | --------- | --------- |
| JS   | Faible (dynamique)               | ×         | Faible    |
| C♯   | Moyen (statique nominal)         | Faible    | Fort      |
| TS   | Fort (statique structurel + ADT) | Moyenne   | Moyen     |
| F♯   | Fort (statique nominal + ADT)    | Élevée    | Faible    |

ADT = _Algebraic Data Types_ = product types + sum types

## Inférence de type

Objectif : Typer explicitement le moins possible

* Moins de code à écrire 👍
* Compilateur garantit la cohérence
* IntelliSense aide le codage et la lecture
  * Importance du nommage pour lecture hors IDE ⚠️

### Inférence de type en C♯ : plutôt faible

* Déclaration d’une méthode → paramètres et retour ❌
* Argument lambda : `list.Find(i => i == 5)` ✔️
* Variable, y.c. objet anonyme : `var o = new { Name = "John" }` ✔️
  * Sauf lambda : `Func<int, int> fn = (x: int) => x + 1;` → KO avec `var`
    * 💡 LanguageExt : `var fn = fun( (x: int) => x + 1 );` ✔️
* Initialisation d'un tableau : `new[] { 1, 2 }` ✔️
* Appel à une méthode générique avec argument, sauf constructeur :
  * `Tuple.Create(1, "a")` ✔️
  * `new Tuple<int, string>(1, "a")` ❌
* C♯ 9 _target-typed expression_ `StringBuilder sb = new();` ✔️

### Inférence en TypeScript - _The good parts_ 👍

👉 Code pur JavaScript _(modulo `as const` qui reste élégant)_

```typescript
const obj1 = { a: 1 };                // { a: number }
const obj2 = Object.freeze({ a: 1 }); // { readonly a: number }
const obj3 = { a: 1 } as const;       // { readonly a: 1 }

const arr1 = [1, 2, null]; // (number | null)[]
const arr2 = [1, 2, 3]; // number[]
const arr3 = arr2.map(x => x * x); // ✔️ Pure lambda

// Type littéral
let s   = 'a';   // string
const a = 'a';   // "a"
```

### Inférence en TypeScript - Limites 🛑

```typescript
// 1. Combinaison de littéraux
const a  = 'a';   // "a"
const aa = a + a; // string (et pas "aa")

// 2. Tuple, immuable ou non
const tupleMutableKo = [1, 'a']; // ❌ (string | number)[]
const tupleMutableOk: [number, string] = [1, 'a'];

const tupleImmutKo = Object.freeze([1, 'a']); // ❌ readonly (string | number)[]
const tupleImmutOk = [1, 'a'] as const; // readonly [1, "a"]

// 3. Paramètres d'une fonction → gêne *Extract function* 😔
// => Refacto de `arr2.map(x => x * x)` en `arr2.map(square)`
const square = x => x * x; // ❌ Sans annotation
//             ~ Parameter 'x' implicitly has an 'any' type.(7006)
const square = (x: number) => x * x; // (x: number) => number
```

### Inférence de type en F♯ : forte 💪

Méthode [Hindley–Milner](https://en.wikipedia.org/wiki/Hindley%E2%80%93Milner_type_system)

* Capable de déduire le type de variables, expressions et fonctions d'un programme dépourvu de toute annotation de type
* Se base sur implémentation et usage

```fsharp
let helper instruction source =
    if instruction = "inc" then // 1. `instruction` a même type que `"inc"` => `string`
      source + 1                // 2. `source` a même type que `1` => `int`
    elif instruction = "dec" then
      source - 1
    else
      source                    // 3. `return` a même type que `source` => `int`
```

#### Inférence en F♯ - Généralisation automatique

```fsharp
// Valeurs génériques
let a = [] // 'a list

// Fonctions génériques : 2 param 'a, renvoie 'a list
let listOf2 x y = [x; y]

// Idem avec 'a "comparable"
let max x y = if x > y then x else y
```

* ☝ En F♯, type générique précédé d'une apostrophe : `'a`
  * Partie `when 'a : comparison` = contraintes sur type
* 💡 Généralisation rend fonction utilisable dans + de cas 🥳
  * `max` utilisable pour 2 args de type `int`, `float`, `string`...
* ☝ D'où l'intérêt de laisser l'inférence plutôt que d'annoter les types

#### Inférence en F♯ - Résolution statique

**Problème :** type inféré + restreint qu'attendu 😯

```fsharp
let sumOfInt x y = x + y // Seulement int
```

* Juste `int` ? Pourtant `+` marche pour les nombres et les chaînes 😕

**Solution :** fonction `inline`

```fsharp
let inline sum x y = x + y // Full generic: 2 params ^a ^b, retour ^c
```

* Paramètres ont un type résolu statiquement = à la compilation
  * Noté avec un _caret_ : `^a`
  * ≠ Type générique `'a`, résolu au runtime

#### Inférence en F♯ - Limites

⚠️ Type d'un objet non inférable depuis ses méthodes

```fsharp
let helperKo instruction source = // 💥 Error FS0072: Recherche d'un objet de type indéterminé...
    match instruction with
    | 'U' -> source.ToUpper()
    | _   -> source

let helper instruction (source: string) = [...] // 👈 Annotation nécessaire

let info list = if list.Length = 0 then "Vide" else "..." // 💥 Error FS0072...
let info list = if List.length list = 0 then "Vide" else $"{list.Length} éléments" // 👌
```

☝ D'où l'intérêt de l'approche FP _(fonctions séparées des données)_    _Vs_ approche OO _(données + méthodes ensemble dans objet)_

#### Inférence en F♯ - Gestion de la précédence

:warning: L'ordre des termes impacte l'inférence

```fsharp
let listKo = List.sortBy (fun x -> x.Length) ["three"; "two"; "one"]
  // 💥 Error FS0072: Recherche d'un objet de type indéterminé...
```

💡 **Solutions**

**1.** Inverser ordre des termes en utilisant le _pipe_

```fsharp
let listOk = ["three"; "two"; "one"] |> List.sortBy (fun x -> x.Length)
```

**2.** Utiliser fonction plutôt que méthode

```fsharp
let listOk' = List.sortBy String.length ["three"; "two"; "one"]
```

## Complément

[https://blog.ploeh.dk/2015/08/17/when-x-y-and-z-are-great-variable-names](https://blog.ploeh.dk/2015/08/17/when-x-y-and-z-are-great-variable-names)

En F♯, les fonctions et variables ont souvent des noms courts : `f`, `x` et `y`. Mauvais nommage ? Non, pas dans les cas suivants :

* Fonction hyper générique → paramètres avec nom générique
* Portée courte → code + lisible avec nom court que nom long
