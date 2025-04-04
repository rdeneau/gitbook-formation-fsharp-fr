# Membres

## Définition

Éléments complémentaires dans définition d'un type _(classe, record, union)_

* _(Événement)_
* Méthode
* Propriété
* Propriété indexée
* Surcharge d'opérateur

## Membres statiques et d'instance

Membre statique : `static member member-name ...`

Membre d'instance :

* Membre concret : `member self-identifier.member-name ...`
* Membre abstrait : `abstract member member-name : type-signature`
* Membre virtuel = nécessite 2 déclarations
  1. Membre abstrait
  2. Implémentation par défaut : `default self-identifier.member-name ...`
* Surcharge d'un membre virtuel : `override self-identifier.member-name ...`

☝ `member-name` en PascalCase _(convention .NET)_

☝ Pas de membre `protected` ou `private` !

## _Self identifier_

* En C♯, Java, TypeScript : `this`
* En VB : `Me`
* En F♯ : au choix → `this`, `self`, `me`, n'importe quel _identifier_ valide...

Définissable de 3 manières complémentaires :

1. Pour le constructeur primaire : avec `as` → `type MyClass() as self = ...`
2. Pour un membre : `member me.Introduce() = printfn $"Hi, I'm {me.Name}"`
3. Pour un membre ne l'utilisant pas : avec `_` → `member _.Hi() = printfn "Hi!"`\
   ☝ Depuis F# 6. Avant, on utilisait `__`&#x20;

## Appeler un membre

💡 Quasiment les mêmes règles qu'en C♯

Appeler un membre d'instance à l'intérieur du type \
→ Préfixer avec _self-identifier_ : `self-identifier.instance-member-name`

Appeler un membre d'instance depuis l'extérieur \
→ Préfixer avec le nom de l'instance : `instance-name.instance-member-name`

Appeler un membre statique \
→ Préfixer par le nom du type : `type-name.static-member-name`\
☝ Même à l'intérieur du type c'est nécessaire en F# (alors que c'est optionnel en C#)

## Méthode

Méthode ≃ Fonction attachée directement à un type

2 formes de déclaration des paramètres :

1. Paramètres curryfiés = Style FP
2. Paramètres en tuple = Style OOP
   * Meilleure interop avec C♯
   * Seul mode autorisé pour les constructeurs
   * Support des paramètres nommés, optionnels, en tableau
   * Support des surcharges _(overloads)_

```fsharp
// (1) Forme en tuple (la + classique)
type Product = { SKU: string; Price: float } with
    member this.TupleTotal(qty, discount) =
        (this.Price * float qty) - discount  // (A)

// (2) Forme currifiée
type Product' =
    { SKU: string; Price: float }
    member me.CurriedTotal qty discount =
        (me.Price * float qty) - discount  // (B)
```

☝ `with` nécessaire en ① mais pas en ② à cause de l'indentation     \
→ `end` peut terminer le bloc commencé avec `with`

☝ `this.Price` Ⓐ et `me.Price` Ⓑ     \
→ Accès à l'instance via le _self-identifier_ défini par le membre

## Arguments nommés

Permet d'appeler une méthode tuplifiée en spécifiant le nom des paramètres :

```fsharp
type SpeedingTicket() =
    member _.SpeedExcess(speed: int, limit: int) =
        speed - limit

    member x.CalculateFine() =
        if x.SpeedExcess(limit = 55, speed = 70) < 20 then 50.0 else 100.0
```

Pratique pour :

* Clarifier un usage pour le lecteur ou le compilateur (en cas de surcharges)
* Choisir l'ordre des arguments
* Ne spécifier que certains arguments, les autres étant optionnels

☝ Les arguments _après un argument nommé_ sont forcément nommés eux-aussi

## Paramètres optionnels

Cette fonctionnalité permet d'appeler une méthode tuplifiée sans spécifier tous les paramètres.

:warning: **Restriction :** Ce n'est que pour les méthodes tuplifiées, y compris les constructeurs. \
Mais cela ne marche ni pour les méthodes curryfiées, ni pour les fonctions, même tuplifiées.

Paramètre optionnel :&#x20;

* Déclaré avec `?` devant son nom → `?arg1: int`
* Dans le corps de la méthode, le paramètre a en fait le type `Option` → `arg1: int option`
  * On peut utiliser `defaultArg` pour indiquer la **valeur par défaut**
  * Mais cette valeur par défaut n'apparaît pas dans la signature, contrairement au C# !

Lors de l'appel de la méthode, on peut spécifier la valeur d'un paramètre optionnel via un argument nommé, ceci de 2 façons possibles :&#x20;

* Nom et type d'origine → `M(arg1 = 1)`
* Nom préfixé par `?` et type wrappé dans une `Option` → `M(?arg1 = Some 1)`

**Exemple :**

```fsharp
type DuplexType = Full | Half

type Connection(?rate: int, ?duplex: DuplexType, ?parity: bool) =
    let duplex = defaultArg duplex Full
    let parity = defaultArg parity false
    let defaultRate = match duplex with Full -> 9600 | Half -> 4800
    let rate = defaultArg rate defaultRate
    do printfn "Baud Rate: %d - Duplex: %A - Parity: %b" rate duplex parity

let conn1 = Connection(duplex = Full)
let conn2 = Connection(?duplex = Some Half)
let conn3 = Connection(300, Half, true)
```

☝ A noter : le _shadowing_ des paramètres par des variables de même nom, par exemple `parity`

`let parity (* bool *) = defaultArg parity (* bool option *) Full`

### Interop .NET

Pour l'interop avec .NET, on utilise une syntaxe différente, basée sur des attributs, ce qui permet de pouvoir spécifier une valeur par défaut : \
`[<Optional; DefaultParameterValue(...)>] arg`

Les attributs `Optional` et `DefaultParameterValue` sont disponibles dans `open System.Runtime.InteropServices`.

Exemple : méthode pour tracer l'appel à une fonction en récupérant son nom grâce à l'attribut `CallerMemberName` dans `System.Runtime.CompilerServices` (\*)

```fsharp
open System.Runtime.CompilerServices
open System.Runtime.InteropServices

type Tracer() =
    static member trace(message: string,
                [<CallerMemberName; Optional; DefaultParameterValue("")>] memberName: string,
                [<CallerFilePath; Optional; DefaultParameterValue("")>] path: string,
                [<CallerLineNumber; Optional; DefaultParameterValue(0)>] line: int) =
        printfn $"Message: {message}"
        printfn $"Member name: {memberName}"
        printfn $"Source file path: {path}"
        printfn $"Source line number: {line}"

open type Tracer

let main() =
    trace "foo"

main();;
// Message: foo
// Member name: main
// Source file path: C:\Users\xxx\stdin
// Source line number: 18
```

(\*) Documentation :link: : [Caller information - F# | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/caller-information)

### Utilisation du pipe `|>`

On peut utiliser le _pipe_ avec une méthode à :\
• 1 paramètre,\
• 2 paramètres dont le dernier est optionnel.

```fsharp
open System.Runtime.InteropServices

type LogLevel =
    | Trace = 1
    | Error = 2

type Logger() =
    member _.Log(message: string, [<Optional; DefaultParameterValue(LogLevel.Trace)>] level: LogLevel) =
        printfn $"[{level}] {message}"

let logger = Logger()

"Mon message" |> logger.Log
// > [Trace] Mon message
```

💡 Si l'on veut avoir un 3e paramètre, il faut passer par une lambda intermédiaire, un peu comme si on écrivait nous-même la curryfication :

```fsharp
// ...
type Logger() =
    // ...
    member this.LogFrom(origin: string, [<Optional; DefaultParameterValue(LogLevel.Trace)>] level: LogLevel) =
        fun message -> this.Log($"Origin: {origin} | {message}", level)

let logger = Logger()

"Mon message" |> logger.LogFrom "Root"
// > [Trace] Origin: Root | Mon message
```

## Tableau de paramètres

Permet de spécifier un nombre variable de paramètres de même type → Via attribut `System.ParamArray` sur le **dernier** argument de la méthode

```fsharp
open System

type MathHelper() =
    static member Max([<ParamArray>] items) =
        items |> Array.max

let x = MathHelper.Max(1, 2, 4, 5)  // 5
```

💡 Équivalent en C♯ de `public static T Max<T>(params T[] items)`

## Appeler méthode C♯ _TryXxx()_

❓ Comment appeler en F♯ une méthode C♯ `bool TryXxx(args, out T outputArg)` ? _(Exemple : `int.TryParse`, `IDictionnary::TryGetValue`)_

* 👎 Utiliser équivalent F♯ de `out outputArg` mais utilise mutation 🤮
* ✅ Ne pas spécifier l'argument `outputArg`
  * Change le type de retour en tuple `bool * T`
  * `outputArg` devient le 2e élément de ce tuple

```fsharp
  match System.Int32.TryParse text with
  | true, i  -> printf $"It's the number {value}."
  | false, _ -> printf $"{text} is not a number."
```

## Appeler méthode _Xxx(tuple)_

❓ Comment appeler une méthode dont 1er param est lui-même un tuple ?!

Essayons :

```fsharp
let friendsLocation = Map.ofList [ (0,0),"Peter" ; (1,0),"Jane" ]
// Map<(int * int), string>
let peter = friendsLocation.TryGetValue (0,0)
// 💥 error FS0001: expression censée avoir le type `int * int`, pas `int`
```

💡 Explications : `TryGetValue(0,0)` = appel méthode en mode tuplifié \
→ Spécifie 2 paramètres, `0` et `0`. \
→ `0` est un `int` alors qu'on attend un tuple `int * int` !

### Solutions

1. 😕 Doubles parenthèses, mais syntaxe confusante
   * `friendsLocation.TryGetValue((0,0))`
2. 😕 _Backward pipe_, mais confusant aussi
   * `friendsLocation.TryGetValue <| (0,0)`
3. ✅ Utiliser une fonction plutôt qu'une méthode
   * `friendsLocation |> Map.tryFind (0,0)`

## Méthode _vs_ Fonction

| Fonctionnalité                       | Fonction  | Méthode currifiée | Méthode tuplifiée |
| ------------------------------------ | --------- | ----------------- | ----------------- |
| Application partielle                | ✅ oui     | ✅ oui             | ❌ non             |
| Arguments nommés                     | ❌ non     | ❌ non             | ✅ oui             |
| Paramètres optionnels                | ❌ non     | ❌ non             | ✅ oui             |
| Tableau de paramètres                | ❌ non     | ❌ non             | ✅ oui             |
| Surcharge / _overload_               | ❌ non     | ❌ non             | ✅ oui  1️⃣        |
| Sensibilité à l'ordre de déclaration | ✅ oui     | ❌ non 2️⃣         | ❌ non 2️⃣         |
| Séparation données / comportement    | ✅ oui 3️⃣ | ❌ non             | ❌ non             |

<table><thead><tr><th>Fonctionnalité</th><th width="170">Fonction</th><th>Méthode statique</th><th>Méthode d'instance</th></tr></thead><tbody><tr><td>Nommage</td><td>camelCase</td><td>PascalCase</td><td>PascalCase</td></tr><tr><td>Support du <code>inline</code></td><td>✅ oui</td><td>✅ oui</td><td>✅ oui</td></tr><tr><td>Récursive</td><td>✅ si <code>rec</code></td><td>✅ oui</td><td>✅ oui</td></tr><tr><td>Inférence de <code>x</code> dans</td><td><code>f x</code> → ✅ oui</td><td>➖</td><td><code>x.M()</code> → ❌ non</td></tr><tr><td>Passable en argument</td><td>✅ oui : <code>g f</code></td><td>✅ oui : <code>g T.M</code></td><td>❌ non : <code>g x.M</code>  4️⃣</td></tr></tbody></table>

**Notes**

1️⃣ Si possible, préférer des paramètres optionnels à des surcharges.

2️⃣ Concernant l'ordre de déclaration :&#x20;

* Il est malgré tout pris en compte dans le cas de membres génériques\
  → Cf. [https://stackoverflow.com/q/66358718/8634147](https://stackoverflow.com/q/66358718/8634147)
* Il est recommandé de déclarer les membres de haut en bas, par homogénéité avec le reste du code.

3️⃣ La séparation des données et du comportement est un principe fort en programmation fonctionnelle. \
→ En orienté-objet, on privilégie le polymorphisme ad-hoc : le comportement est en façade et les données sont encapsulées dans les classes. \
→ Si on veut ajouter du comportement dans un périmètre + limité, on passe par une nouvelle classe, par composition si possible ou par héritage ou par copie des données.\
→ En C♯, on peut également le faire au moyen de méthodes d'extension, au détriment du polymorphisme.\
→ En F♯, on privilégie la séparation des données et du comportement. Les données sont mises dans un type immutable pour contrer cette perte (relative) d'encapsulation. Cela permet de partager un type et de lui affecter des comportements spécifiques, localisés uniquement là où on en a besoin. \
→ Par exemple dans l'architecture [Elmish](https://elmish.github.io/elmish/), on fait en sorte de séparer _State_ et _View_ : chacun agit sur le _Model_ dans son registre, sans s'interférer l'un l'autre : le _State_ gère l'_Init_ du _Model_ et son _Update_ en fonction du Message reçu, la _View_ s'occupe du rendu du _Model_. Si l'on souhaite ajouter des méthodes dans le _Model_, il faut faire attention à ce qu'elles soient suffisamment génériques pour être utilisables à la fois le _State_ et dans la _View_.

4️⃣ Solutions alternatives :&#x20;

* Wrapper dans lambda : `g (fun x -> x.M())`
* Passer à F♯8 : `g _.M()`

## Méthodes statiques ou module compagnon ?

Le choix entre méthode et fonction se pose également dans le cas de méthodes statiques ou de fonctions pour un module compagnon. L'usage dans le code appelant sera le même, à la casse près (_camelCase_ vs _PascalCase_).

La question est pertinente en particulier dans le cas d'un _SmartConstructor_ ou d'une _Factory_ c'est-à-dire pour créer une instance du type. Ce qui peut faire pencher la balance du côté d'une méthode statique est la possibilité d'avoir des paramètres optionnels que l'on peut mettre en relation avec des champs optionnels du type.

```fsharp
type PersonName =
    { FirstName  : string
      MiddleName : string option
      LastName   : string }
    
    static member Create(firstName, lastName, ?middleName) =
      { FirstName  = firstName
        MiddleName = middleName
        LastName   = lastName }

// Usage
let johnDoe = PersonName.Create("John", "Doe")

// Equivalent avec le constructeur primaire
let johnDoe' =
    { FirstName  = "John"
      LastName   = "Doe"
      MiddleName = None }


// 💡 Egalement élégant pour définir les 3 champs
let jfk = PersonName.Create("John", "Fitzgerald", "Kennedy")

// Equivalent avec le constructeur primaire
let jfk' =
    { FirstName  = "John"
      MiddleName = Some "Fitzgerald"
      LastName   = "Kennedy" }


// 💡 Permet enfin de bénéficer des arguments nommés
let pierreLaurent = PersonName.Create(firstName = "Pierre", lastName = "Laurent")
```

💡Une telle méthode présente un autre avantage : permettre de créer un `Record` en spécifiant le nom de son type.\
→ C'est parfois nécessaire, pour enlever une ambiguïté avec un autre `Record` de même structure.\
→ Cela n'est pas toujours pratique/élégant à faire à l'aide d'une annotation de type (`{ FirstName = ...} : PersonName`) ou en qualifiant les champs (`{ PersonName.FirstName = ... }`)

## Propriétés

≃ Sucre syntaxique masquant un _getter_ et/ou un _setter_

→ Permet d'utiliser la propriété comme s'il s'agissait d'un champ

2 façons de déclarer une propriété :

### Getter

* Syntaxe 1 : `member this.Property = expression`
* Syntaxe 2 : `member this.Property with get () = expression`

:point\_up: **Remarques :**&#x20;

* `expression` peut accéder aux variables et autres membres de l'objet grâce au `this`
* `expression` est ré-évaluée à chaque appel

Exemple 1 :&#x20;

```fsharp
type Person = { FirstName: string; LastName: string } with
    member it.FullName = $"{it.LastName.ToUpper()} {it.FirstName}"

let joe = { FirstName = "Joe"; LastName = "Dalton" }
let s = joe.FullName  // "DALTON Joe"
```

Exemple 2 : \
→ Permet de voir que la valeur d'un Getter est réévaluée à chaque appel \
→ Mais dans un tel cas préférer une méthode à une propriété ❗

```fsharp
type Generator() =
    let random = new System.Random()
    member _.NextValue = random.Next()  // La valeur change à chaque appel
```

### Automatique

* _Read-only_ :  `member val Property = value`\
  → Equivalent d'une propriété C# `{ get; }`
* _Read/write_ : `member val Property = value with get, set`\
  → Equivalent d'une propriété C# `{ get; set; }`

:point\_up: **Remarques :**&#x20;

* `value` est la valeur d'initialisation. Ce n'est pas le _backing field_ de la propriété car il est, lui, implicite, généré par le compilateur.
* La propriété conserve la même valeur à chaque appel au `get`. Seul l'éventuel `set` permet de modifier sa valeur.

Exemple 1 :&#x20;

```fsharp
type PersonName(first: string, last: string) =
    member val First = first
    member val Last = last
    member val Full = $"{last.ToUpper()} {first}"

let joe = PersonName(first = "Joe", last = "Dalton")
let s = joe.Full  // "DALTON Joe"
```

Exemple 2 :

```fsharp
type Generator() =
    let random = new System.Random()
    member val FirstValue = random.Next() // La valeur ne change pas à chaque appel
```

### Autres cas

Dans les autres cas, la syntaxe est verbeuse _(_[_détails_](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/members/properties)_)_ \
👉 Préférer des méthodes, c'est plus explicite

### Pattern matching

:warning: Les propriétés ne sont pas déconstructibles.

→ Peuvent participer à un pattern matching que dans partie `when`

```fsharp
type Person = { First: string; Last: string } with
    member this.FullName = // Getter
        $"{this.Last.ToUpper()} {this.First}"

let joe = { First = "Joe"; Last = "Dalton" }
let { First = first } = joe  // val first : string = "Joe"
let { FullName = x } = joe
// 💥 ~~~~~~~~ Error FS0039: undefined record label 'FullName'

let salut =
    match joe with
    | _ when joe.FullName = "DALTON Joe" -> "Salut, Joe !"
    | _ -> "Bonjour !"
// val salut : string = "Salut, Joe !"
```

## Propriétés indexées

Permet accès par indice, comme si la classe était un tableau : `instance.[index]` → Intéressant pour une collection ordonnée, pour masquer l'implémentation

Mise en place en déclarant membre `Item`

```fsharp
member self-identifier.Item
    with get(index) =
        get-member-body
    and set index value =
        set-member-body
```

💡 Propriété _read-only_ (_write-only_) → ne déclarer que le _getter_ (_setter_)

☝ Paramètre en tuple pour _getter_ ≠ paramètres curryfiés _setter_

**Exemple :**

```fsharp
type Lang = En | Fr

type DigitLabel() =
    let labels = // Map<Lang, string[]>
        [| (En, [| "zero"; "one"; "two"; "three" |])
           (Fr, [| "zéro"; "un"; "deux"; "trois" |]) |] |> Map.ofArray

    member val Lang = En with get, set
    member me.Item with get(i) = labels.[me.Lang].[i]
    member _.En with get(i) = labels.[En].[i]

let digitLabel = DigitLabel()
let v1 = digitLabel.[1]     // "one"
digitLabel.Lang <- Fr
let v2 = digitLabel.[2]     // "deux"
let v3 = digitLabel.En(2)   // "two"
// 💡 Notez la différence de syntaxe de l'appel à la propriété `En`
```

## Slice

> Idem propriété indexée mais renvoie plusieurs valeurs

Définition : via méthode _(normale ou d'extension)_ `GetSlice(?start, ?end)`

Usage : via opérateur `..`

```fsharp
type Range = { Min: int; Max: int } with
    member this.GetSlice(min, max) =
        { Min = System.Math.Max(defaultArg min this.Min, this.Min)
        ; Max = System.Math.Min(defaultArg max this.Max, this.Max) }

let range = { Min = 1; Max = 5 }
let slice1 = range.[0..3] // { Min = 1; Max = 3 }
let slice2 = range.[2..]  // { Min = 2; Max = 5 }
```

## Surcharge d'opérateur

Opérateur surchargé à 2 niveaux possibles :

1. Dans un module, sous forme de fonction
   * `let [inline] (operator-symbols) parameter-list = ...`
   * 👉 Cf. session sur les fonctions
   * ☝ Limité : 1 seule surcharge possible
2. Dans un type, sous forme de membre
   * `static member (operator-symbols) (parameter-list) =`
   * Mêmes règles que pour la forme de fonction
   * 👍 Plusieurs surcharges possibles (N types × P _overloads_)

**Exemple :**

```fsharp
type Vector(x: float, y: float) =
    member _.X = x
    member _.Y = y

    override me.ToString() =
        let format n = (sprintf "%+.1f" n)
        $"Vector (X: {format me.X}, Y: {format me.Y})"

    static member (*)(a, v: Vector) = Vector(a * v.X, a * v.Y)
    static member (*)(v: Vector, a) = a * v
    static member (~-)(v: Vector) = -1.0 * v
    static member (+) (v: Vector, w: Vector) = Vector(v.X + w.X, v.Y + w.Y)

let v1 = Vector(1.0, 2.0)   // Vector (X: +1.0, Y: +2.0)
let v2 = v1 * 2.0           // Vector (X: +2.0, Y: +4.0)
let v3 = 0.75 * v2          // Vector (X: +1.5, Y: +3.0)
let v4 = -v3                // Vector (X: -1.5, Y: -3.0)
let v5 = v1 + v4            // Vector (X: -0.5, Y: -1.0)
```
