# 🍔 Quiz

## Question 1

Code ci-dessous : valide ou non ?

```fsharp
namespace A

let a = 1
```

**A.** Oui

**B.** Non

### Réponse

**B.** Non, un namespace ne peut pas contenir de valeurs !

## Question 2

Code ci-dessous : valide ou non ?

```fsharp
namespace A

module B

let a = 1
```

**A.** Oui

**B.** Non

### Réponse

**B.** Non, module B est ici top-level => interdit après un namespace

Code équivalent valide :

* Option 1 : module top-level qualifié

```fsharp
module A.B

let a = 1
```

* Option 2 : namespace + module local

```fsharp
namespace A

module B =
    let a = 1
```

## Question 3

Quel est le nom qualifié / le _full name_ de `add` ?

```fsharp
namespace Common.Utilities

module IntHelper =
    let add x y = x + y
```

**A.** `add`

**B.** `IntHelper.add`

**C.** `Utilities.IntHelper.add`

**D.** `Common.Utilities.IntHelper.add`

### Réponse

**D.** `Common.Utilities.IntHelper.add`

* `IntHelper` pour le module parent
* `Common.Utilities` pour le namespace racine
