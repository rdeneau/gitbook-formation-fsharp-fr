# Classe, structure

## Classe

Classe en F♯ ≡ classe en C♯

* Brique de base pour l'orienté-objet
* Constructeur d'objets contenant des données de type défini et des méthodes

Définition d'une classe

* Commence par `type` _(comme tout type en F♯)_
* Nom de la classe généralement suivi du **constructeur primaire**

```fsharp
type CustomerName(firstName: string, lastName: string) =
    // Corps du constructeur primaire
    // Membres...
```

☝ Paramètres `firstName` et `lastName` visibles dans tout le corps de la classe

## Classe générique

Paramètres génériques à spécifier car non inférés

```fsharp
type Tuple2_KO(item1, item2) = // ⚠️ 'item1' et 'item2': type 'obj' !
    // ...

type Tuple2<'T1, 'T2>(item1: 'T1, item2: 'T2) =  // 👌
    // ...
```

## Constructeur secondaire

Syntaxe pour définir un autre constructeur : `new(argument-list) = constructor-body`

☝ Doit appeler le constructeur primaire !

```fsharp
type Point(x: float, y: float) =
    new() = Point(0, 0)
    // Membres...
```

☝ Paramètres des constructeurs : que en tuple, pas curryfiés !

## Instanciation

Appel d'un des constructeurs, avec arguments en tuple

* Ne pas oublier `()` si aucun argument, sinon on obtient une fonction !

Dans un `let` binding : `new` optionnel et non recommandé

* `let v = Vector(1.0, 2.0)` 👌
* `let v = new Vector(1.0, 2.0)` ❌

Dans un `use` binding : `new` obligatoire

* `use d = new Disposable()`

## Initialisation des propriétés

On peut initialiser des propriétés avec setter à l'instanciation

* Les spécifier en tant que **arguments nommés** dans l'appel au constructeur
* Les placer après les éventuels arguments du constructeur :

```fsharp
type PersonName(first: string) =
    member val First = first with get, set
    member val Last = "" with get, set

let p1 = PersonName("John")
let p2 = PersonName("John", Last="Paul")
let p3 = PersonName(first="John", Last="Paul")
```

💡 Équivalent de la syntaxe C♯ `new PersonName("John") { Last = "Paul" }`

## Classe vide

Pour définir une classe sans corps, on utilise la syntaxe verbeuse `class end` :&#x20;

```fsharp
type K = class end
let k = K()
```

## Singleton

On peut définir une classe _Singleton_ en rendant privé le constructeur primaire :&#x20;

```fsharp
type S private() =
    static member val Instance = S()

let s = S.Instance  // val s : S
```

:warning: Pas thread safe !

💡 On peut utiliser simplement une _single-case union_ :&#x20;

```fsharp
type S = S
let s = S
```

## Classe abstraite

Annotée avec `[<AbstractClass>]`

Un des membres est **abstrait** :

1. Déclaré avec mot clé `abstract`
2. Pas d'implémentation par défaut avec mot clé `default` _(Sinon le membre est virtuel)_

Héritage via mot clé `inherit`, suivi de l'appel au constructeur de la classe de base

**Exemple :**

```fsharp
[<AbstractClass>]
type Shape2D() =
    member val Center = (0.0, 0.0) with get, set
    member this.Move(?deltaX: float, ?deltaY: float) =
        let x, y = this.Center
        this.Center <- (x + defaultArg deltaX 0.0,
                        y + defaultArg deltaY 0.0)
    abstract GetArea : unit -> float
    abstract Perimeter : float  with get

type Square(side) =
    inherit Shape2D()
    member val Side = side
    override _.GetArea () = side * side
    override _.Perimeter = 4.0 * side

let o = Square(side = 2.0, Center = (1.0, 1.0))
printfn $"S={o.Side}, A={o.GetArea()}, P={o.Perimeter}"  // S=2, A=4, P=8
o.Move(deltaY = -2.0)
printfn $"Center {o.Center}"  // Center (1, -1)
```

## Champs

Convention de nommage : camelCase

2 types de champs : implicite ou explicite

* Implicite ≃ Variable à l'intérieur du constructeur primaire
* Explicite ≡ Champ classique d'une classe en C♯ / Java

### Champ implicite

Syntaxe :

* Variable  : `[static] let [ mutable ] variable-name = expression`
* Fonction : `[static] let [ rec ] function-name function-args = expression`

☝ **Notes**

* Déclaré avant les membres de la classe
* Valeur initiale obligatoire
* Privé
* S'utilise sans devoir préfixer par le `self-identifier`

### D'instance

```fsharp
type Person(firstName: string, lastName: string) =
    let fullName = $"{firstName} {lastName}"
    member _.Hi() = printfn $"Hi, I'm {fullName}!"

let p = Person("John", "Doe")
p.Hi()  // Hi, I'm John Doe!
```

### Statique

```fsharp
type K() =
    static let mutable count = 0

    // do binding exécuté à chaque construction
    do
        count <- count + 1

    member _.CreatedCount = count

let k1 = K()
let count1 = k1.CreatedCount  // 1
let k2 = K()
let count2 = k2.CreatedCount  // 2
```

## Champ explicite

Déclaration du type, sans valeur initiale : `val [ mutable ] [ access-modifier ] field-name : type-name`

* `val mutable a: int` → champ publique
* `val a: int` → champ interne `a@` + propriété `a => a@`

## Champ _vs_ propriété

```fsharp
// Champs explicites readonly
type C1 =
    val a: int
    val b: int
    val mutable c: int
    new(a, b) = { a = a; b = b; c = 0 } // 💡 Constructeur 2ndaire "compacte"

// VS propriétés readonly => ordre inversé dans SharpLab : b avant a
type C2(a: int, b: int) =
    member _.A = a
    member _.B = b
    member _.C = 0

// VS propriétés auto-implémentées
type C3(a: int, b: int) =
    member val A = a
    member val B = b with get
    member val C = 0 with get, set
```

## Champ explicite ou implicite ou propriété

Champ explicite **peu utilisé** :

* Ne concerne que les classes et structures
* Utile avec fonction native manipulant la mémoire directement     _(Car ordre des champs préservés - cf._ [_SharpLab_](https://sharplab.io/#v2:DYLgZgzgNAJiDUAfA9MgBAYQBYEMC2ADhGgKYAeBwAlgMZUAuJxATiTjAPYB2wAngLAAoerwIlMARjQBeIWnloAbjmBocINFS705C5aoBGGrTsEK0XEgHcAFDihoDAShloA3mtc4A3I9cHfAF80VDRAXg3AQp3Mbgh6ZgBXGkZ45jQAJi4YHCpWNAAiGg5CHCSSPKEhUIA1AGU0AmYOBqoAS/oWljZOHgFhUXEMNLtjbQcjTW0XWTMFPBI8AxJUgH0AOgBBL115OYWltDWAIX8hIA===)_)_
* Besoin d'une variable `[<ThreadStatic>]`
* Interaction avec classe F♯ de code généré sans constructeur primaire

Champ implicite - `let` binding

* Variable intermédiaire lors de la construction

Autres cas d'usages → propriété auto-implémentée

* Exposer une valeur → `member val`
* Exposer un "champ" mutable → `member val ... with get, set`

## Structures

Alternatives aux classes mais + limités / héritage et récursivité

Même syntaxe que pour les classes mais avec en plus :

* Soit attribut `[<Struct>]`
* Soit bloc `struct...end` _(fréquent)_

```fsharp
type Point =
    struct
        val mutable X: float
        val mutable Y: float
        new(x, y) = { X = x; Y = y }
    end

let x = Point(1.0, 2.0)
```
