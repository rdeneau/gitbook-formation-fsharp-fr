# 🍔 Quiz

## Question 1 - ⏱ 10’’

```fsharp
type Address = { City: string; Country: string }

let format address = $"{address.City}, {address.Country}"

let addresses: Address list = ...
```

**Quelle fonction de `List` peut-on utiliser sur `addresses` pour appliquer `format` aux éléments ?**

**A.** `List.iter()`&#x20;

**B.** `List.map()`&#x20;

**C.** `List.sum()`

## Question 2 - ⏱ 10’’

**Que vaut `[1..4] |> List.head` ?**

**A.** `[2; 3; 4]`

**B.** `1`

**C.** `4`

## Question 3 - ⏱ 10’’

**Quelle est la bonne manière d'obtenir la moyenne d'une liste ?**

**A.** `[2; 4] |> List.average`

**B.** `[2; 4] |> List.avg`

**C.** `[2.0; 4.0] |> List.average`







## Réponses

### Question 1

**A.** `List.iter()` ❌

**B.** `List.map()` ✅

**C.** `List.sum()` ❌

### Question 2

**`[1..4] |> List.head =`**

**A.** `[2; 3; 4]` ❌     _(Ne pas confondre avec `List.tail`)_

**B.** `1` ✅

**C.** `4` ❌     _(Ne pas confondre avec `List.last`)_

### Question 3

**Bonne manière d'obtenir la moyenne d'une liste :**

**A.** `[2; 4] |> List.average` ❌     💥 Error FS0001: Le type `int` ne prend pas en charge l'opérateur `DivideByInt`

**B.** `[2; 4] |> List.avg`     💥 Error FS0039: La valeur \[...] `avg` n'est pas définie.

**C.** `[2.0; 4.0] |> List.average` ✅     `val it : float = 3.0`
