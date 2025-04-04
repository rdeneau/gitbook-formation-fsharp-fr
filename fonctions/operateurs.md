# Opérateurs

## Opérateur

Est défini comme une fonction :

* Opérateur unaire : `let (~symbols) = ...`
* Opérateur binaire : `let (symbols) = ...`
* _Symbols_ = combinaison de `% & * + - . / < = > ? @ ^ | ! $`

2 façons d'utiliser les opérateurs :

* En tant qu'opérateur → infixe `1 + 2` ou préfixe `-1`
* En tant que fonction → chars entre `()` : `(+) 1 2` ≡ `1 + 2`

## Opérateurs standards

Également définis dans `FSharp.Core`

* Opérateurs arithmétiques : `+`, `-`...
* Opérateurs de pipeline
* Opérateurs de composition

## Opérateurs _Pipe_

Opérateurs binaires, placés entre une valeur simple et une fonction

* Appliquent la valeur à la fonction = Passe la valeur en argument
* Permettent d'éviter la mise entre parenthèses / précédence
* ∃ plusieurs _pipes_
  * _Pipe right_ `|>` : le _pipe_ "classique"
  * _Pipe left_ `<|` a.k.a. _pipe_ inversé
  * _Pipe right 2_ `||>`
  * Etc.

### Opérateur _Pipe right_ `|>`

Inverse l'ordre entre fonction et valeur : `val |> fn` ≡ `fn val`

* Ordre naturel "sujet verbe", comme appel méthode d'un objet (`obj.M(x)`)
* _Pipeline_ : enchaîner appels de fonctions, sans variable intermédiaire
* Aide inférence d'objet. Exemple :

```fsharp
let items = ["a"; "bb"; "ccc"]

let longestKo = List.maxBy (fun x -> x.Length) items  // ❌ Error FS0072
//                                   ~~~~~~~~

let longest = items |> List.maxBy (fun x -> x.Length) // ✅ Renvoie "ccc"
```

### Opérateur _Pipe left_ `<|`

`fn <| expression` ≡ `fn (expression)`&#x20;

* ☝ Usage un peu moins courant que `|>`&#x20;
* ✅ Avantage mineur : permet d'éviter des parenthèses
* ❌ Inconvénient majeur : se lit de droite à gauche → Inverse du sens lecture naturel en anglais et ordre exécution

```fsharp
printf "%i" 1+2          // 💥 Erreur
printf "%i" (1+2)        // Avec parenthèses
printf "%i" <| 1+2       // Avec pipe inversé
```

#### Quid d'une expression telle que `x |> fn <| y` ❓

Exécutée de gauche à droite : `(x |> fn) <| y` ≡ `(fn x) <| y` ≡ `fn x y`

* En théorie : permettrait d'utiliser `fn` en position infixée
* En pratique : difficile à lire à cause du double sens de lecture ❗

👉 Conseil : **À ÉVITER**

### Opérateur _Pipe right 2_ `||>`

`(x, y) ||> fn` ≡ `fn x y`

* Pour passer 2 arguments à la fois, sous la forme d'un tuple&#x20;
* Usage peu fréquent mais pratique avec `fold` pour passer la valeur initiale (`seed`) et la liste avant de définir la fonction `folder`.

```fsharp
let items = [1..5]

// 😕 On peut manquer le 0 au bout (le seed)
let sumOfEvens =
    items |> List.fold (fun acc x -> if x % 2 = 0 then acc + x else acc) 0

let sumOfEvens' =
    (0, items)
    ||> List.fold (fun acc x -> if x % 2 = 0 then acc + x else acc)

// 💡 Remplacer lambda par fonction nommée
let addIfEven acc x = if x % 2 = 0 then acc + x else acc
let sumOfEvens'' = items |> List.fold addIfEven 0
```

☝️ Cet opérateur correspond à la notion de "décurryfication" c'est-à-dire à pouvoir appeler une fonction `f` curryfiée en lui passant ces 2 paramètres sous la forme d'un tuple :&#x20;

```fsharp
let uncurry f (a,b) = f a b
```

## Opérateur _Compose_ `>>`

Opérateurs binaires, placés **entre deux fonctions** → Le résultat de la 1ère fonction servira d'argument à la 2e fonction

`f >> g` ≡ `fun x -> g (f x)` ≡ `fun x -> x |> f |> g`

💡 Peut se lire « `f` ensuite `g` »

:warning:️ Les types doivent correspondre : `f: 'T -> 'U` et `g: 'U -> 'V` → On obtient une fonction de signature `'T -> 'V`

```fsharp
let add1 x = x + 1
let times2 x = x * 2

let add1Times2 x = times2(add1 x) // 😕 Style explicite mais + chargé
let add1Times2' = add1 >> times2  // 👍 Style concis
```

## Opérateur _Compose_ inverse `<<`

Sert rarement, sauf pour retrouver un ordre naturel des termes

Exemple avec opérateur `not` (qui remplace le `!` du C♯) :

```fsharp
let Even x = x % 2 = 0

// Pipeline classique
let Odd x = x |> Even |> not

// Réécrit avec composition inverse
let Odd = not << Even
```

## _Pipe_ `|>` ou _Compose_ `>>` ?

_**Compose**_ **`let h = f >> g`**

* Composition de 2 fonctions `f` et `g`
* Renvoie une nouvelle fonction
* Les fonctions `f` et `g` ne sont exécutées que lorsque `h` l'est

_**Pipe**_ **`let result = value |> f`**

* Juste une syntaxe différente pour passer un argument
* La fonction `f` est :
  * Exécutée si elle n'a qu'1! param → `result` est une valeur
  * Appliquée partiellement sinon → `result` est une fonction

## Style _Point-free_

A.k.a _Programmation tacite_

Fonction définie par composition ou application partielle ou avec `function` \
→ **Paramètre implicite** \
(point fait référence à un point dans un espace)

```fsharp
let add1 x = x + 1                // (x: int) -> int
let times2 x = x * 2              // (x: int) -> int
let add1Times2 = add1 >> times2   // int -> int • x implicite • Par composition

let isEven x = x % 2 = 0
let evens list = List.filter isEven list // (list: int list) -> int list
let evens' = List.filter isEven // int list -> int list • Par application partielle

let greet name age = printfn $"My name is {name} and I am %d{age} years old!" // name:string -> age:int -> unit
let greet' = printfn "My name is %s and I am %d years old!" // (string -> int -> unit)
```

> 💡 **Conseil :** l'expression anglaise _Point-free_ est trompeuse en français car le terme _Point_ désigne un point dans l'espace et non pas le point pour accéder à un membre d'un objet : `obj.Member` qui correspond à _dot_ en Anglais. Mieux vaut employer _Programmation tacite_.

### Pros/Cons ⚖️

#### ✅ Avantages

Style concis • Abstraction des paramètres, opère au niveau fonctions

#### ❌ Inconvénients

Perd le nom du paramètre devenu implicite dans la signature \
→ Sans importance si la fonction reste compréhensible :

* Nom du param non significatif (ex. `x`)&#x20;
* Type du param et nom de la fonction suffisent \
  → Déconseillé pour une API publique

### Limite 🛑

Marche mal avec fonctions génériques :

```fsharp
let isNotEmptyKo = not << List.isEmpty          // 💥 Error FS0030: Restriction de valeur
let isNotEmpty<'a> = not << List.isEmpty<'a>    // 👌 Avec annotation
let isNotEmpty' list = not (List.isEmpty list)  // 👌 Style explicite
```

🔗 [https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/conventions #partial-application-and-point-free-programming](https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/conventions#partial-application-and-point-free-programming)

## Opérateurs personnalisés

2 possibilités :

* Surcharge d'opérateurs
* Création d'un nouvel opérateur

## Surcharge d'opérateurs

En général, concerne un type spécifique → Surcharge définie à l'intérieur du type associé _(comme en C♯)_

```fsharp
type Vector = { X: int; Y: int } with
    // Opérateur unaire (cf ~ et 1! param) d'inversion d'un vecteur
    static member (~-) (v: Vector) =
        { X = -v.X
          Y = -v.Y }

    // Opérateur binaire d'addition de 2 vecteurs
    static member (+) (a: Vector, b: Vector) =
        { X = a.X + b.X
          Y = a.Y + b.Y }

let v1 = -{ X=1; Y=1 } // { X = -1; Y = -1 }
let v2 = { X=1; Y=1 } + { X=1; Y=3 } // { X = 2; Y = 4 }
```

## Création d'un nouvel opérateur

* Définition plutôt dans un module ou dans un type associé
* Cas d'usage classique : alias fonction existante, utilisé en infixe

```fsharp
// "OR" Composition of 2 functions (fa, fb) which return an optional result
let (<||>) fa fb x =
    match fa x with
    | Some v -> Some v // Return value produced by (fa x) call
    | None   -> fb x   // Return value produced by (fb x) call

// Functions: int -> string option
let tryMatchPositiveEven x = if x > 0 && x % 2 = 0 then Some $"Even {x}" else None
let tryMatchPositiveOdd x = if x > 0 && x % 2 <> 0 then Some $"Odd {x}" else None
let tryMatch = tryMatchPositiveEven <||> tryMatchPositiveOdd

tryMatch 0;; // None
tryMatch 1;; // Some "Odd 1"
tryMatch 2;; // Some "Even 2"
```

## Symboles autorisés dans un opérateur

**Opérateur unaire "tilde"** \
→ `~` suivi de `+`, `-`, `+.`, `-.`, `%`, `%%`, `&`, `&&`

**Opérateur unaire "snake"** \
→ Plusieurs `~`, ex : `~~~~`

**Opérateur unaire "bang"** \
→ `!` suivi combinaison de `!`, `%`, `&`, `*`, `+`, `.`, `/`, `<`, `=`, `>`, `@`, `^`, `|`, `~`, `?` \
→ Sauf `!=` (!=) qui est binaire

**Opérateur binaire** \
→ Toute combinaison de `!`, `%`, `&`, `*`, `+`, `.`, `/`, `<`, `=`, `>`, `@`, `^`, `|`, `~`, `?` \
→ qui ne correspond pas à un opérateur unaire

## Symboles à l'usage

Tout opérateur s'utilise tel quel ❗ Sauf opérateur unaire "tilde" : s'utilise sans le `~` initial

| Opérateur    | Déclaration         | Usage     |
| ------------ | ------------------- | --------- |
| Unaire tilde | `let (~&&) x = …`   | `&&x`     |
| Unaire snake | `let (~~~) x = …`   | `~~~x`    |
| Unaire bang  | `let (!!!) x = …`   | `!!!x`    |
| Binaire      | `let (<ˆ>) x y = …` | `x <ˆ> y` |

☝ Pour définir un opérateur commençant ou terminant par un `*`, il faut mettre un espace entre `(` et `*` ainsi qu'entre `*` et `)` pour distinguer d'un bloc de commentaires F# `(* *)` \
→ `let ( *+ ) x y = x * y + y` ✅

## Opérateur ou fonction ?

### Opérateur infixe _vs_ fonction

👍 **Pour** :&#x20;

* Respecte l'ordre naturel de lecture (gauche → droite)&#x20;
* Permet d'éviter des parenthèses  \
  → `1 + 2 * 3` _vs_ `multiply (add 1 2) 3`

:warning: **Contre** :&#x20;

* Un opérateur "folklorique" (ex : `@!`) sera moins compréhensible qu'une fonction dont le nom utilise le **langage du domaine**

### Utiliser un opérateur en tant que fonction

💡 On peut utiliser l'application partielle d'un opérateur binaire :

Exemples :&#x20;

* A la place d'une lambda : \
  → `(+) 1` ≡ `fun x -> x + 1`&#x20;
* Pour définir une nouvelle fonction :\
  → `let isPositive = (<) 0` ≡ `let isPositive x = 0 < x` ≡ `x >= 0`

##

