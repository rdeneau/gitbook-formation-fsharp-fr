# Unités de mesure

## Présentation

Moyen d'associer un **type numérique** à une unité de mesure

* Durée : `s` _aka_ `second`
* Masse : `kg`
* Longueur : `m` _aka_ `metre`
* ...

Les unités sont **vérifiées à la compilation**

* Empêche d'ajouter des 🥦 à des 🥕 → **code + sûr**
* Permet de les **combiner** : Vitesse = Distance / Durée → `m/s`

## Déclaration

Syntaxe basée sur attribut `[<Measure>]`

```fsharp
// 👉 Nouvelles unités "from scratch"
[<Measure>] type kilogram
[<Measure>] type metre
[<Measure>] type second

// 👉 Alias d'unités existantes
[<Measure>] type kg = kilogram
[<Measure>] type m  = metre
[<Measure>] type s  = second

// 👉 Combinaison d'unités existantes
[<Measure>] type Hz = / s
[<Measure>] type N = kg m / s^2
```

## SI

Les unités du **Système International** sont prédéfinies dans les namespaces :

`FSharp.Data.UnitSystems.SI.UnitNames` :

* `ampere`, `hertz`, `joule`, `kelvin`, `kilogram`, `metre`...
* https://fsharp.github.io/fsharp-core-docs/reference/fsharp-data-unitsystems-si-unitnames.html

`FSharp.Data.UnitSystems.SI.UnitSymbols`

* `A`, `Hz`, `J`, `K`, `kg`, `m`...
* https://fsharp.github.io/fsharp-core-docs/reference/fsharp-data-unitsystems-si-unitsymbols.html

## Usage

```fsharp
// Unité définie en annotant le nombre
let distance = 1.0<m>               // val distance : float<m> = 1.0
let time = 2.0<s>                   // val time : float<s> = 2.0

// Unité combinée, inférée
let speed = distance / time         // val speed : float<m/s> = 0.5

// Unité combinée, définie par annotation
let [<Literal>] G = 9.806<m/s^2>    // val G : float<m/s ^ 2> = 9.806

// Comparaison
let sameFrequency = (1<Hz> = 1</s>)   // ✅ true
let ko1 = (distance = 1.0)            // ❌ Error FS0001: Incompatibilité de type.
                                      // 💥 Attente de 'float<m>' mais obtention de 'float'
let ko2 = (distance = 1<m>)           // 💥 Attente de 'float<m>' mais obtention de 'int<m>'
let ko3 = (distance = time)           // 💥 Attente de 'float<m>' mais obtention de 'float<s>'
```

## Symbole

💡 **Astuce :** utilisation des _doubles back ticks_

```fsharp
[<Measure>] type ``Ω``
[<Measure>] type ``°C``
[<Measure>] type ``°F``

let waterFreezingAt = 0.0<``°C``>
// val waterFreezingAt : float<°C> = 0.0

let waterBoilingAt = 100.0<``°C``>
// val waterBoilingAt : float<°C> = 100.0
```

## Conversion

* Facteur multiplicatif avec une unité `<target/source>`
* Fonction de conversion utilisant ce facteur

```fsharp
[<Measure>] type m
[<Measure>] type cm
[<Measure>] type km

module Distance =
    let toCentimeter (x: float<m>) = // (x: float<m>) -> float<cm>
        x * 100.0<cm/m>

    let toKilometer (x: float<m>) = // (x: float<m>) -> float<km>
        x / 1000.0<m/km>

let a = Distance.toCentimeter 1.0<m>   // val a : float<cm> = 100.0
let b = Distance.toKilometer 500.0<m>  // val b : float<km> = 0.5
```

Exemple 2 : degré Celsius (°C) → degré Fahrenheit (°F)

```fsharp
[<Measure>] type ``°C``
[<Measure>] type ``°F``

module Temperature =
    let toFahrenheit ( x: float<``°C``> ) =  // (x: float<°C>) -> float<°F>
        9.0<``°F``> / 5.0<``°C``> * x + 32.0<``°F``>

let waterFreezingAt = Temperature.toFahrenheit 0.0<``°C``>
// val waterFreezingAt : float<°F> = 32.0

let waterBoilingAt = Temperature.toFahrenheit 100.0<``°C``>
// val waterBoilingAt : float<°F> = 212.0
```

## Ajouter/supprimer

Ajouter une unité à un nombre nu :

* ✅ `number * 1.0<target>`

Supprimer l'unité d'un nombre `number : float<source>` :

* ✅ `number / 1.0<source>`
* ✅ `float number`

Créer une liste de nombres avec unité :

* ✅ `[1<m>; 2<m>; 3<m>]`
* ❌ `[1<m>..3<m>]` _(un range nécessite des nombres nus)_
* ✅ `[ for i in [1..3] -> i * 1<m> ]`

## Effacées au runtime !

Les unités de mesure sont propres au compilateur F♯. → Elles ne sont pas compilées en .NET

## Type avec unité générique

Besoin de distinguer d'un type générique classique → Annoter l'unité générique avec `[<Measure>]`

```fsharp
type Point<[<Measure>] 'u, 'data> =
    { X: float<'u>; Y: float<'u>; Data: 'data }

let point = { X = 10.0<m>; Y = 2.0<m>; Data = "abc" }
// val point : Point<m, string> = { X = 10.0; Y = 2.0; Data = "abc" }
```

## Unité pour primitive non numérique

💡 Nuget [FSharp.UMX](https://github.com/fsprojects/FSharp.UMX) _(Unit of Measure Extension)_ \
→ Pour autres primitives `bool`, `DateTime`, `Guid`, `string`, `TimeSpan`

```fsharp
open System

#r "nuget: FSharp.UMX"
open FSharp.UMX

[<Measure>] type ClientId
[<Measure>] type OrderId

type Order = { Id: Guid<OrderId>; ClientId: string<ClientId> }

let order = { Id = % Guid.NewGuid(); ClientId = % "RDE" }
```
