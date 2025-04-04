# Généralités

## Vue d'ensemble

Classifications des types .NET :

1. Types valeur _vs_ types référence -- abrégés _TVal_ et _TRef_
2. Types primitifs _vs_ types composites
3. Types génériques
4. Types créés à partir de valeurs littérales
5. Types algébriques : somme _vs_ produit

## Types composites

Créés par combinaison d'autres types

| Types .NET     |         | `class`                | ✅        | ❌         |
| -------------- | ------- | ---------------------- | -------- | --------- |
|                |         | `struct`, `enum`       | ❌        | ✅         |
|                |         | `interface` (1)        | ❌        | ❌         |
| Spécifiques C♯ | C♯ 3.0  | Type anonyme           | ✅        | ❌         |
|                | C♯ 7.0  | _Value tuple_          | ❌        | ✅         |
|                | C♯ 9.0  | `record (class)`       | ✅        | ❌         |
|                | C♯ 10.0 | `record struct`        | ❌        | ✅         |
| Spécifiques F♯ |         | _Tuple, Record, Union_ | _Opt-in_ | _Opt-out_ |
|                | F♯ 4.6  | _Record_ anonyme       | _Opt-in_ | _Opt-out_ |

> **(1)** Une interface ne comporte pas de données. Ce n'est donc pas pertinent de la classer en type référence ou type valeur.

Tous ces types peuvent être génériques sauf une `enum`.

Localisation :

* _Top-level_ : `namespace`, _top-level_ module F♯
* _Nested_ : `class` (C♯), `module` (F♯)
* Non définissables dans méthode (C♯) ou valeur simple / fonction (F♯) !

En F♯ toutes les définitions de type se font avec mot clé `type`&#x20;

* y compris les classes, les enums et les interfaces !&#x20;
* mais les tuples n'ont pas besoin d'une définition de type

## Particularité des types F♯ / types .NET

_Tuple, Record, Union_ sont :

* Immuables
* Non nullables
* Égalité et comparison structurelles _(sauf avec champ fonction)_
* `sealed` : ne peuvent pas être hérités
* Déconstruction, avec même syntaxe que construction

Reflète approches différentes selon paradigme :

* FP : focus sur les données organisées en types
* OOP : focus sur les comportements, possiblement polymorphiques

## Types à valeurs littérales

Valeurs littérales = instances dont le type est inféré

* Types primitifs :          `true` (`bool`) • `"abc"` (`string`) • `1.0m` (`decimal`)
* Tuples C♯ / F♯ :           `(1, true)`
* Types anonymes C♯ : `new { Name = "Joe", Age = 18 }`
* Records F♯ :                `{ Name = "Joe"; Age = 18 }`

☝ **Note :**

* Les types doivent avoir été définis au préalable ❗
* Sauf tuples et types anonymes C♯

## Types algébriques

> Types composites, combinant d'autres types par produit ou par somme.

Soit les types `A` et `B`, alors on peut créer :

* Le type produit `A × B` :
  * Contient 1 composante de type `A` ET 1 de type `B`
  * Composantes anonymes ou nommées
* Le type somme `A + B` :
  * Contient 1 composante de type `A` OU 1 de type `B`

Idem par extension les types produit / somme de N types

### Pourquoi les termes "Somme" et "Produit" ?

Soit `N(T)` le nombre de valeurs dans le type `T`, par exemples :

* `bool` → 2 valeurs : `true` et `false`
* `unit` → 1 valeur `()`

Alors :

* Le nombre de valeurs dans le type somme `A + B` est `N(A) + N(B)`.
* Le nombre de valeurs dans le type produit `A × B` est `N(A) * N(B)`.

## Types algébriques _vs_ Types composites

| Type _custom_                    | Somme | Produit | Composantes nommées |
| -------------------------------- | ----- | ------- | ------------------- |
| `enum`                           | ✅     | ❌       | ➖                   |
| _Union_ F♯                       | ✅     | ❌       | ➖                   |
| `class` ⭐, `interface`, `struct` | ❌     | ✅       | ✅                   |
| _Record_ F♯                      | ❌     | ✅       | ✅                   |
| _Tuple_ F♯                       | ❌     | ✅       | ❌                   |

⭐ Classes + variations C♯ : type anonyme, _Value tuple_ et `record`

👉 En C♯, pas de type somme sauf `enum`, très limitée par rapport au type union 📍 [unions.md](unions.md "mention")

## Abréviation de type

**Alias** d'un autre type : `type [name] = [existingType]`

Différents usages :

```fsharp
// Documenter le code voire éviter répétitions
type ComplexNumber = float * float
type Addition<'num> = 'num -> 'num -> 'num  // 👈 Marche aussi avec les génériques

// Découpler (en partie) usages / implémentation pour faciliter son changement
type ProductCode = string
type CustomerId = int
```

:warning: Effacée à la compilation → ~~_type safety_~~ : compilateur autorise de passer `int` à la place de `CustomerId` !

💡 Il est aussi possible de créer un alias pour un module. La syntaxe est alors \
`module [name] = [existingModule]`
