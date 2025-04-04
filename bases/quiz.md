# 🍔 Quiz

## 1. Qui est le papa de F♯ ? ⏱ 10’’

**A.** Anders Hejlsberg

**B.** Don Syme

**C.** Scott Wlaschin

### Réponse

**A.** Anders Hejlsberg ❌

Papa de C♯ et de TypeScript

**B.** Don Syme ✅

&#x20;[dsymetweets](https://twitter.com/dsymetweets) • 🎥 [F♯ Code I Love](https://www.youtube.com/watch?v=1AZA1zoP-II)

**C.** Scott Wlaschin ❌

Auteur du blog [F♯ for Fun and Profit](https://fsharpforfunandprofit.com/), mine d'or pour F♯

## 2. Comment se nomme l'opérateur `::` ? ⏱ 10’’

**A.** Append

**B.** Concat

**C.** Cons

### Réponse

**A.** Append ❌

`List.append` : concatène 2 listes

**B.** Concat ❌

`List.concat` : concatène un ensemble de listes

**C.** Cons ✅

`newItem :: list` est la manière la + rapide de créer une nouvelle liste avec un nouvel élément en tête : `1 :: [2; 3]` renvoie `[1; 2; 3]`.

## 3. Cherchez l'intrus ⏱ 15’’

**A.** `let a = "a"`

**B.** `let a () = "a"`

**C.** `let a = fun () -> "a"`

### Réponse

B et C sont des fonctions, A est juste une `string`.

**A.** `let a = "a"` ✅

**B.** `let a () = "a"` ❌

**C.** `let a = fun () -> "a"` ❌

## 4. Quelle ligne ne compile pas ? ⏱ 15’’

```fsharp
let evens list =
    let isEven x =
    x % 2 = 0
    List.filter isEven list
```

Ligne **1.** `let evens list =`

Ligne **2.** `let isEven x =`

Ligne **3.** `x % 2 = 0`

Ligne **4.** `List.filter isEven list`

### Réponse

Ligne **3.** `x % 2 = 0` : problème d'indentation

```fsharp
let evens list =
    let isEven x =
    x % 2 = 0       // 👈 Manque une indentation ici
    List.filter isEven list
```

## 5. Comment se nomme l'opérateur `|>` ? ⏱ 10’’

**A.** Compose

**B.** Chain

**C.** Pipeline

**D.** Pipe

### Réponse

**A.** Compose ❌ - L'opérateur de composition est `>>` 📍

**B.** Chain ❌

**C.** Pipeline ❌

**D.** Pipe ✅

## 6. Quelle expression compile ? ⏱ 15’’

**A.** `a == "a" && b != "*"`

**B.** `a == "a" && b <> "*"`

**C.** `a = "a" && b <> "*"`

**D.** `a = "a" && b != "*"`

### Réponse

☝ En F♯, les opérateurs d'égalité et d'inégalité sont respectivement `=` et `<>`.

**A.** `a == b && b != ""` ❌

**B.** `a == b && b <> ""` ❌

**C.** `a = b && b <> ""` ✅

**D.** `a = b && b != ""` ❌
