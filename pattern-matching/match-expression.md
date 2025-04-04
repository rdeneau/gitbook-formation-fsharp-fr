# Match expression

## Présentation

Similaire à une expression `switch` en C♯ 8.0 en + puissant grâce aux patterns

Syntaxe :

```fsharp
match test-expression with
| pattern1 [ when condition ] -> result-expression1
| pattern2 [ when condition ] -> result-expression2
| ...
```

Renvoie le résultat de la 1ère branche dont le pattern "match" `test-expression`

☝ **Note :** toutes les branches doivent renvoyer le même type !

## Exhaustivité

Un `switch` doit toujours définir un cas par défaut _(cf. pattern wildcard `_`)_ \
Sinon : warning à la compilation, risque de 💥 `MatchFailureException` au runtime

Pas nécessaire dans une _match expression_ si les branches couvrent tous les cas car le compilateur vérifie leur exhaustivité et les branches "mortes"

```fsharp
let fn x =
    match x with
    | Some true  -> "ok"
    | Some false -> "ko"
    | None       -> ""
    | _          -> "?"  // ⚠️ Warning FS0026: Cette règle n'aura aucune correspondance
```

☝ **Conseil :** + les branches sont exhaustives, + le code est explicite et sûr

Exemple : matcher tous les cases d'un type union permet de gérer l'ajout d'un case par un warning à la compilation : `Warning FS0025: Critères spéciaux incomplets dans cette expression`

* Détection d'un ajout accidentel
* Identification du code à changer pour gérer le nouveau case

## Guard

Syntaxe : `pattern1 when condition` Usage : raffiner un pattern, via contraintes sur des variables

```fsharp
let classifyBetween low top value =
    match value with
    | x when x < low -> "Inf"  // 💡 Alternative : `_ when value < low`
    | x when x = low -> "Low"
    | x when x = top -> "Top"
    | x when x > top -> "Sup"
    | _ -> "Between"

let test1 = 1 |> classifyBetween 1 5  // "Low"
let test2 = 6 |> classifyBetween 1 5  // "Sup"
```

💡 La _guard_ n'est évaluée que si le pattern est satisfait.

## Guard et Pattern OR

Le pattern OR a une _precedence/priorité_ plus élevée que la _guard_ :

```fsharp
type Parity = Even of int | Odd of int

let parityOf value =
    if value % 2 = 0 then Even value else Odd value

let hasSquare square value =
    match parityOf square, parityOf value with
    | Even x2, Even x
    | Odd  x2, Odd  x
        when x2 = x*x -> true  // 👈 Porte sur les 2 patterns précédents
    | _ -> false

let test1 = 2 |> hasSquare 4  // true
let test2 = 3 |> hasSquare 9  // true
```

## Mot clé `function`

Syntaxe :

```fsharp
function
| pattern1 [ when condition ] -> result-expression1
| pattern2 [ when condition ] -> result-expression2
| ...
```

Equivalent à une lambda prenant un paramètre implicite qui est "matché" :

```fsharp
fun value ->
    match value with
    | pattern1 [ when condition ] -> result-expression1
    | pattern2 [ when condition ] -> result-expression2
    | ...
```

### Intérêts

**a.** Dans un pipeline :

```fsharp
value
|> is123
|> function
    | true  -> "ok"
    | false -> "ko"
```

**b.** Écriture plus succincte d'une fonction :

```fsharp
// ⚠️ Paramètre implicite => peut rendre le code \+ difficile à comprendre !
let is123 = function
    | 1 | 2 | 3 -> true
    | _ -> false
```

### Limites

:warning: Paramètre implicite => peut rendre le code + difficile à comprendre !

Exemple : fonction déclarée avec d'autres paramètres eux explicites → On peut se tromper sur le nombre de paramètres et leur ordre :

```fsharp
let classifyBetween low high = function  // 👈 3 paramètres : low, high + 1 implicite
    | x when x < low  -> "Inf"
    | x when x = low  -> "Low"
    | x when x = high -> "High"
    | x when x > high -> "Sup"
    | _ -> "Between"

let test1 = 1 |> classifyBetween 1 5  // "Low"
let test2 = 6 |> classifyBetween 1 5  // "Sup"
```

## Fonction `fold`

Fonction associée à un type union et masquant la logique de _matching_ Prend N+1 paramètres pour un type union avec N _cases_ `CaseI of 'DataI`

* N fonctions `'DataI -> 'T` (1 / _case_), avec `'T` le type de retour de `fold`
* En dernier, l'instance du type union à matcher

```fsharp
type [<Measure>] C
type [<Measure>] F

type Temperature =
    | Celsius     of float<C>
    | Fahrenheint of float<F>

module Temperature =
    let fold mapCelsius mapFahrenheint temperature : 'T =
        match temperature with
        | Celsius x     -> mapCelsius x      // mapCelsius    : float<C> -> 'T
        | Fahrenheint x -> mapFahrenheint x  // mapFahrenheint: float<F> -> 'T
```

### Utilisation

```fsharp
module Temperature =
    // ...
    let [<Literal>] FactorC2F = 1.8<F/C>
    let [<Literal>] DeltaC2F = 32.0<F>

    let celsiusToFahrenheint x = (x * FactorC2F) + DeltaC2F  // float<C> -> float<F>
    let fahrenheintToCelsius x = (x - DeltaC2F) / FactorC2F  // float<F> -> float<C>

    let toggleUnit temperature =
        temperature |> fold
            (celsiusToFahrenheint >> Fahrenheint)
            (fahrenheintToCelsius >> Celsius)

let t1 = Celsius 100.0<C>
let t2 = t1 |> Temperature.toggleUnit  // Fahrenheint 212.0
```

### Intérêt

`fold` masque les détails d'implémentation du type

Par exemple, on peut ajouter un _case_ `Kelvin` et n'impacter que `fold`, pas les fonctions qui l'appellent comme `toggleUnit` :

```fsharp
type [<Measure>] C
type [<Measure>] F
type [<Measure>] K  // 🌟

type Temperature =
    | Celsius     of float<C>
    | Fahrenheint of float<F>
    | Kelvin      of float<K>  // 🌟

module Temperature =
    let fold mapCelsius mapFahrenheint temperature : 'T =
        match temperature with
        | Celsius x     -> mapCelsius x      // mapCelsius: float<C> -> 'T
        | Fahrenheint x -> mapFahrenheint x  // mapFahrenheint: float<F> -> 'T
        | Kelvin x      -> mapCelsius (x * 1.0<C/K> + 273.15<C>)  // 🌟

Kelvin 273.15<K>
|> Temperature.toggleUnit
|> Temperature.toggleUnit
// Celsius 0.0<C>
```
