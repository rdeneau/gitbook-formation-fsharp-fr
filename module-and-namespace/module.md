# Module

## Syntaxe

```fsharp
// Top-level module
module [accessibility-modifier] [qualified-namespace.]module-name
declarations

// Local module
module [accessibility-modifier] module-name =
    declarations
```

`accessibility-modifier` : restreint l'accessibilité → `public` _(défaut)_, `internal` _(assembly)_, `private` _(parent)_

Le nom complet (`[namespace.]module-name`) doit être unique → 2 fichiers ne peuvent pas déclarer des modules de même nom

## Module top-level

* Doit être déclaré en 1er dans un fichier ❗
* Contient tout le reste du fichier
  * Contenu non indenté
  * Ne peut pas contenir de namespace
* Peut être qualifié = inclus dans un namespace parent _(existant ou non)_

## Module top-level implicite

* Si fichier sans module/namespace top-level
* Nom du module = nom du fichier
  * Sans l'extension
  * Avec 1ère lettre en majuscule
  * Ex : `program.fs` → `module Program`

## Module local

Syntaxe similaire au `let` mais ne pas oublier :

* Le signe `=` après le nom du module local ❗
* D'indenter tout le contenu du module local
  * Non indenté = ne fait pas partie du module local

## Contenu d'un module

Un module, local comme _top-level_, peut contenir :&#x20;

* Types et sous modules locaux
* Valeurs, fonctions

Différence : l'indentation du contenu

* Module top-level : contenu non indenté
* Module local : contenu indenté

## Équivalence module / classe statique

```fsharp
module MathStuff =
    let add x y  = x + y
    let subtract x y = x - y
```

Ce module F♯ est équivalent à la classe statique suivante :

```csharp
public static class MathStuff
{
    public static int add(int x, int y) => x + y;
    public static int subtract(int x, int y) => x - y;
}
```

Cf. [sharplab.io](https://sharplab.io/#v2:DYLgZgzgPgtg9gEwK7AKYAICyBDALgCwGVckwx0BeAWAChb0H01d1sEF0APdATwYq7oA1L3qNm6CEgBGuAE7YAxi259KggLS8gA=)

## Module imbriqué

Comme en C♯ et les classes, les modules F♯ peuvent être imbriqués

```fsharp
module Y =
    module Z =
        let z = 5

printfn "%A" Y.Z.z
```

☝ **Notes :**

* Intéressant avec module imbriqué privé pour isoler/regrouper
* Sinon, préférer une vue aplanie

## Module top-level _vs_ local

| Propriété                                            | Top-level | Local |
| ---------------------------------------------------- | --------- | ----- |
| Qualifiable                                          | ✅         | ❌     |
| <p>Signe <code>=</code> </p><p>+ Contenu indenté</p> | ❌         | ✅ ❗   |

* Module _top-level_ → 1er élément déclaré dans un fichier&#x20;
* Sinon _(après un module/namespace top-level)_ → module local

## Module récursif

Même principe que namespace récursif → Pratique pour qu'un type et un module associé se voient mutuellement

☝ **Recommandation :** limiter au maximum la taille des zones récursives

## Annotation d'un module

2 attributs influencent l'usage d'un module

### `[<RequireQualifiedAccess>]`&#x20;

→ Force l'usage qualifié des éléments d'un module\
→ Empêche l'import du module

* 💡 Pratique pour éviter le _shadowing_ pour des noms communs : `add`, `parse`...
* ☝️ Il n'y a pas d'attribut équivalent pour les méthodes statiques d'un type :&#x20;
  * Par défaut, on utilise l'accès qualifié, y compris à l'intérieur du type, alors qu'à l'intérieur d'un module on n'accède directement aux autres fonctions, la qualification n'étant pas possible (sauf peut-être en rendant le module récursif 🤷)
  * L'accès direct est possible en important le type à l'aide du mot clé `open type`.

### `[<AutoOpen>]`

Import du module en même temps que le module/namespace parent

* 💡 Pratique pour "monter" valeurs/fonctions au niveau d'un namespace\
  &#xNAN;_(un namespace ne pouvant pas en contenir normalement)_
* :warning:️ Pollue le _scope_ courant

:warning:️ Portée de `AutoOpen`\
Quand on n'a pas importé le module/namespace parent, `AutoOpen` est sans effet : pour accéder au contenu du module enfant, la qualification comprend non seulement le module/namespace parent mais également le module enfant : \
→ `Parent.childFunction` ❌\
→ `Parent.Child.childFunction` ✅

💡On emploie couramment `AutoOpen` pour mieux organiser un module, en regroupant des éléments dans des modules enfants\
→ à condition qu'ils restent de taille raisonnable, sinon il vaudrait mieux considérer de les répartir dans différents fichiers\
→ on peut combiner cela avec le fait de rendre certains modules `private` pour cacher l'ensemble de son contenu au code appelant tout en gardant ce contenu accessible directement au reste du module courant.

👉 Avoir un module `AutoOpen` à l'intérieur d'un module `RequireQualifiedAccess` ne fait sens que si le module est `private`.

### `AutoOpen`, `RequireQualifiedAccess` ou rien ?

Soit un type `Cart` avec son module compagnon `Cart` \
→ Comment appeler la fonction qui ajoute un élément au panier ?

* `addItem item cart`\
  → `[<RequireQualifiedAccess>]` intéressant pour forcer à avoir `Cart.addItem` dans le code appelant et lever toute ambiguïté sur le containeur (`Cart`) dans lequel on ajoute l'élément
* `addItemToCart item cart`\
  → `addItemToCart` est _self-explicit_, `Cart.addItemToCart` pléonastique.\
  → `[<AutoOpen>]` intéressant pour éviter de devoir importer le module

Si le module `Cart` contient d'autres fonctions de ce type, mieux vaudrait leur appliquer toute la même convention de nommage.

## Module et Type

> Un module sert typiquement à regrouper des fonctions agissant sur un type de donnée bien spécifique.

2 styles, selon localisation type / module :

* Type défini avant le module → module compagnon
* Type défini dans le module

## Module compagnon d'un type

* Style par défaut - cf. `List`, `Option`, `Result`...
* Bonne interop autres langages .NET
* Module peut porter le même nom que le type

```fsharp
type Person = { FirstName: string; LastName: string }

module Person =
    let fullName person = $"{person.FirstName} {person.LastName}"

let person = { FirstName = "John"; LastName = "Doe" }   // Person
person |> Person.fullName // "John Doe"
```

## Module wrappant un type

* Type défini à l'intérieur du module
* On peut nommer le type `T` ou comme le module

```fsharp
module Person =
    type T = { FirstName: string; LastName: string }

    let fullName person = $"{person.FirstName} {person.LastName}"

let person = { FirstName = "John"; LastName = "Doe" }   // Person.T ❗
person |> Person.fullName // "John Doe"
```

Recommandé pour améliorer encapsulation

* Constructeur du type `private`&#x20;
* Module contient un _smart constructor_

```fsharp
module Person =
    type T = private { FirstName: string; LastName: string }

    let create first last =
        if System.String.IsNullOrWhiteSpace first
        then Error "FirstName required"
        else Ok { FirstName = first; LastName = last }

    let fullName person =
        $"{person.FirstName} {person.LastName}".Trim()

Person.create "" "Doe"                                // Error "LastName required"
Person.create "Joe" "" |> Result.map Person.fullName  // Ok "Joe"
```

## Top-level module _ou_ namespace

* Préférer un namespace pour :&#x20;
  * 1 ou plusieurs types, avec quelques modules compagnons
  * Interop avec C#\
    → Cf. [docs.microsoft.com/.../fsharp/style-guide/conventions#organizing-code](https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/conventions#organizing-code)
* Dans les autres cas, préférer un top-level module, pour gagner un niveau d'indentation.

