# Records

## Points clés

Type produit, alternative au Tuple quand type imprécis sous forme de tuple

* Exemple : `float * float`
  * Point dans un plan ?
  * Coordonnées géographiques ?
  * Parties réelle et imaginaire d'un nombre complexe ?
* _Record_ permet de lever le doute en nommant le type et ses éléments

```fsharp
type Point = { X: float; Y: float }
type Coordinate = { Latitude: float; Longitude: float }
type ComplexNumber = { Real: float; Imaginary: float }
```

## Déclaration

* Membres nommés en PascalCase, pas en ~~camelCase~~
* Membres séparés `;` ou retours à la ligne
* Saut de ligne après `{` qu'en cas de membre additionnels

```fsharp
type PostalAddress =        ┆     type PostalAddress =
    { Address: string       ┆         {
      City: string          ┆             Address: string
      Zip: string }         ┆             City: string
                            ┆             Zip: string
                            ┆         }
                            ┆         member x.ZipAndCity = $"{x.Zip} {x.City}"
```

🔗 [https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/formatting](https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/formatting)

* [#use-pascalcase-for-type-declarations-members-and-labels](https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/formatting#use-pascalcase-for-type-declarations-members-and-labels)
* [#formatting-record-declarations](https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/formatting#formatting-record-declarations)

## Instanciation

* Même syntaxe qu'un objet anonyme C♯ sans `new`
  * Mais le record doit avoir été déclaré au-dessus !
* Membres peuvent être renseignés dans n'importe quel ordre
  * Mais doivent tous être renseignés → pas de membres optionnels !

```fsharp
type Point = { X: float; Y: float }

let point1  = { X = 1.0; Y = 2.0 }
let point2  = { Y = 2.0; X = 1.0 }   // 👈 Possible mais confusant dans ce cas
let pointKo = { Y = 2.0 }            // 💥 Error FS0764:
//            ~~~~~~~~~~~ Aucune assignation spécifiée pour le champ 'X' de type 'Point'
```

:warning: **Piège :** cette syntaxe est similaire mais pas identique à celle de la déclaration\
→ `:` pour définir le type du membre\
→ `=` pour définir la valeur du membre

Les instances "longues" devraient être écrites sur plusieurs lignes :

```fsharp
// Style 1 : le + classique
let rainbow =
    { Boss = "Jeffrey"
      Lackeys = ["Zippy"; "George"; "Bungle"] }

// Style 2 : similaire au C#
// → Plus facile de ré-indenter et ré-ordonner (par ex. Lackeys avant Boss)
let rainbow =
    {
        Boss = "Jeffrey"
        Lackeys = ["Zippy"; "George"; "Bungle"]
    }
```

## Déconstruction

Même syntaxe pour déconstruire un _Record_ que pour l'instancier :

```fsharp
let { X = x1 } = point1; // (1)
let { X = x2; Y = y2 } = point1;

// On peut aussi accéder aux membres via le point '.'
let x3 = point1.X;
let y3 = point1.Y;
```

💡 On peut ignorer certains champs\
→ Rend explicite les champs utilisés 👍

:warning: On ne peut pas déconstruire les membres additionnels tels que les propriétés !

```fsharp
type PostalAddress =
    {
        Address: string
        City: string
        Zip: string
    }
    member x.CityLine = $"{x.Zip} {x.City}"

let address = { Address = ""; City = "Paris"; Zip = "75001" }

let { CityLine = cityLine } = address   // 💥 Error FS0039
//    ~~~~~~~~ L'étiquette d'enregistrement 'CityLine' n'est pas définie
let cityLine = address.CityLine         // 👌 OK
```

## Inférence

* L'inférence de type ne marche pas quand on _"dot"_ une `string`
* ... mais elle marche avec un _Record_ ?!

```fsharp
type PostalAddress =
    { Address: string
      City   : string
      Zip    : string }

let department address =
    address.Zip.Substring(0, 2) |> int
    //     ^^^^ 💡 Permet d'inférer que address est de type `PostalAddress`

let departmentKo zip =
    zip.Substring(0, 2) |> int
//  ~~~~~~~~~~~~~ Error FS0072
```

## Pattern matching

Fonction `inhabitantOf` donnant le nom des habitants à une adresse :

```fsharp
type Address = { Street: string; City: string; Zip: string }

let department { Zip = zip } = zip.Substring(0, 2) |> int

let inIleDeFrance departmentNumber =
    [ 75; 77; 78 ] @ [ 91..95 ] |> List.contains departmentNumber

let inhabitantOf address =
    match address with
    | { Street = "Pôle"; City = "Nord" } -> "Père Noël"
    | { City = "Paris" } -> "Parisien"
    | _ when department address = 78 -> "Yvelinois"
    | _ when department address |> inIleDeFrance -> "Francilien"
    | _ -> "Français"   // Le discard '_' sert de pattern par défaut (obligatoire)
```

## Conflit de noms

En F♯, le typage est nominal et non pas structurel comme en TypeScript.\
→ On peut écrire plusieurs fois le même Record avec les mêmes champs \
&#xNAN;_(cf. `First` et `Last` ci-dessous)_.\
→ Il faut qualifier un champ avec le nom du Record pour lever l'ambiguïté sur le type.

```fsharp
type Person1 = { First: string; Last: string }
type Person2 = { First: string; Last: string }
let alice = { First = "Alice"; Last = "Jones"}  // val alice: Person2...
// (car Person2 est le type le + proche qui correspond aux étiquettes First et Last)

// ⚠️ Déconstruction
let { First = firstName } = alice   // Warning FS0667
//  ~~~~~~~~~~~~~~~~~~~~~  Les étiquettes et le type attendu du champ de ce Record
//                         ne déterminent pas de manière unique un type Record correspondant

let { Person2.Last = lastName } = alice     // 👌 OK avec type en préfixe
let { Person1.Last = lastName } = alice     // 💥 Error FS0001
//                                ~~~~~ Type 'Person1' attendu, 'Person2' reçu
```

☝ **Conseil :** mieux vaut écrire des types distincts ou les séparer dans ≠ modules.

## Modification

Record immuable mais facile de créer nouvelle instance ou copie modifiée

* Expression de _**copy and update**_ d'un _Record_
* Syntaxe spéciale pour ne modifier que certains champs
* Multi-lignes si expression longue

```fsharp
let address2 = { address with Street = "Rue Vivienne" }

let { City = city; Zip = zip } = address
let address2' = { Street = "Rue Vivienne"; City = city; Zip = zip }
// address2 = address2'

let address3 =                  ┆      let address3' =
    { address with              ┆          { Street = address.Street
        City = "Lyon"           ┆            City   = "Lyon"
        Zip  = "69001" }        ┆            Zip    = "69001" }
// address3 = address3'
```

### Différences C♯ / F♯ / JS

```csharp
// Record C♯ 9.0
address with { Street = "Rue Vivienne" }
```

```fsharp
// F♯ copy and update
{ address with Street = "Rue Vivienne" }
```

```typescript
// Object destructuring with spread operator
{ ...address, street: "Rue Vivienne" }
```

### Limites 🛑

Lisibilité réduite quand plusieurs niveaux imbriqués

```fsharp
type Street = { Num: string; Label: string }
type Address = { Street: Street }
type Person = { Address: Address }

let person = { Address = { Street = { Num = "15"; Label = "rue Neuf" } } }

let person' =
    { person with
        Address =
          { person.Address with
              Street =
                { person.Address.Street with
                    Num = person.Address.Street.Num + " bis" } } }
```

## Record `struct`

Attribut `[<Struct>]` permet de passer d'un type référence à un type valeur :

```fsharp
[<Struct>]
type Point = { X: float; Y: float; Z: float }
```

⚖️ **Pros/Cons d'une `struct` :**

* ✅ Performant car ne nécessite pas de _garbage collection_
* :warning: Passée par valeur → pression sur la mémoire

👉 Adapté à un type "petit" en mémoire (\~2-3 champs)
