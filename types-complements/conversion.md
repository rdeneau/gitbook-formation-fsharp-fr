# Conversion

## Conversion de nombre

Types numériques :

* Entier : `byte`, `int16`, `int`/`int32`, `int64`
* Flottant : `float`/`double` (64b), `single` (32b), `decimal`
* Autres : `char`, `enum`

Conversion entre eux **explicite** → Fonction de même nom que le type cible

```fsharp
let x = 1               // val x : int = 1
let y = float x         // val y : float = 1.0
let z = decimal 1.2     // val z : decimal = 1.2M
let s = char 160        // val s : char = ' '
```

## Conversion entre nombre et enum

Il faut utiliser le nom de l'enum pour convertir un nombre en enum :

* Soit en paramètre générique de la fonction `enum<my-enum>` ①
* Soit par annotation de type et la fonction `enum` sans paramètre générique ②

L'opération inverse utilise la fonction `int` ③

```fsharp
type Color =
    | Red   = 1
    | Green = 2
    | Blue  = 3

let color1 = enum<Color> 1      // (1)  val color1 : Color = Red
let color2 : Color = enum 2     // (2)  val color2 : Color = Green
let value3 = int Color.Blue     // (3)  val c1 : int = 3
```

## Casting d'objets

→ S'utilise pour un objet dont le type appartient à une hiérarchie

| Fonctionnalité | Précision             | Sûr        | Opérateur | Fonction   |
| -------------- | --------------------- | ---------- | --------- | ---------- |
| _Upcast_       | Vers type de base     | ✅ Oui      | `:>`      | `upcast`   |
| _Downcast_     | Vers type dérivé      | ❌ Non (\*) | `:?>`     | `downcast` |
| Test de type   | Dans pattern matching | ✅ Oui      | `:?`      |            |

(\*) Le _downcast_ peut échouer → risque de `InvalidCastException` au runtime ⚠️

## Upcasting d'objets

En C♯ : _upcast_ peut généralement être implicite

```csharp
object o = "abc";
```

En F♯ : _upcast_ peut parfois être implicite mais en général doit être **explicite**, avec opérateur `:>`

```fsharp
let o1: obj = "abc"             // Implicite 💥 Error FS0001...
let o2 = "abc" :> obj           // Explicite 👌

let toObject x : obj = x        // obj -> obj
let o3 = "abc" |> toObject      // Implicite 👌

let l1: obj list = [1; 2; 3]    // Implicite 👌
let l2: int seq = [1; 2; 3]     // Implicite 💥 Error FS0001...
```

☝️ Règles élargies/assouplies en F♯ 6\
→ Exemple : upcast implicite de `int list` vers `int seq`

```fsharp
let l2: int seq = [1; 2; 3]  // 👌 OK en F♯ 6
```

## Casting d'objets - Exemple

```fsharp
type Base() =
    abstract member F : unit -> string
    default _.F() = "F Base"

type Derived1() =
    inherit Base()
    override _.F() = "F Derived1"

type Derived2() =
    inherit Base()
    override _.F() = "F Derived2"

let d1 = Derived1()
let b1 = d1 :> Base         // val b1 : Base
let b1': Base = upcast d1   // val b1' : Base

let t1 = b1.GetType().Name  // val t1 : string = "Derived1"

let one = box 1  // val one : obj = 1

let d1' = b1 :?> Derived1           // val d1' : Derived1
let d2' = b1 :?> Derived2           // 💥 System.InvalidCastException

let d1'': Derived1 = downcast b1    // val d1'' : Derived1

let f (b: Base) =
    match b with
    | :? Derived1 as derived1 -> derived1.F()
    | :? Derived2 as derived2 -> derived2.F()
    | _ -> b.F()

let x = f b1            // val x : string = "F Derived1"
let y = b1.F()          // val y : string = "F Derived1"
let z = f (Base())      // val z : string = "F Base"
let a = f (Derived2())  // val a : string = "F Derived2"
// ☝️      ^^^^^^^^  Upcast implicite
```

## Test de type

L'opérateur `:?` réalise un test de type et renvoie un booléen.

```fsharp
let isDerived1 = b1 :? Derived1   // val isDerived1 : bool = true
let isDerived2 = b1 :? Derived2   // val isDerived2 : bool = false
```

☝️ Il faut _boxer_ un nombre pour tester son type :

```fsharp
let isIntKo = 1 :? int          // 💥 Error FS0016
let isInt32 = (box 1) :? int    // val isInt32 : bool = true
let isFloat = (box 1) :? float  // val isFloat : bool = false
```

💡 `box x` ≃ `x :> obj`
