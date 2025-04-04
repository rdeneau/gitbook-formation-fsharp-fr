# 🍔 Quiz

## Question 1

> Comment définir la valeur de retour `v` d'une fonction `f` ? ⏱ 10’’

**A.** Il suffit de nommer la valeur `result`

**B.** Faire `return v`

**C.** `v` constitue la dernière ligne de `f`

### Réponse

**A.** Il suffit de nommer la valeur `result` ❌

**B.** Faire `return v` ❌

**C.** `v` constitue la dernière ligne de `f` ✅

## Question 2

> Comment écrire fonction `add` prenant 2 `string`s et renvoyant un `int` ⏱ 20’’

**A.** `let add a b = a + b`

**B.** `let add (a: string) (b: string) = (int a) + (int b)`

**C.** `let add (a: string) (b: string) : int = a + b`

### Réponse

**A.** `let add a b = a + b` ❌

> Mauvais type inféré pour `a` et `b` : `int`

**B.** `let add (a: string) (b: string) = (int a) + (int b)` ✅

> * Il faut spécifier le type de `a` et `b`.
> * Il faut les convertir en `int`.
> * Le type de retour `int` peut être inféré.

**C.** `let add (a: string) (b: string) : int = a + b`

> Ici, `+` concatène les `string`s.

## Question 3

> Que fait le code `add >> multiply` ? ⏱ 10’’

**A.** Créer un pipeline

**B.** Définir une fonction

**C.** Créer une composition

### Réponse

**A.** Créer un pipeline ❌

**B.** Définir une fonction ❌ _(mais ce n'est pas faux)_

**C.** Créer une composition ✅

## Question 4

> Retrouvez le nom de ces fonctions [Core](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/) ⏱ 60’’

**A.** `let ? _ = ()`

**B.** `let ? x = x`

**C.** `let ? f x = f x`

**D.** `let ? x f = f x`

**E.** `let ? f g x = g (f x)`

💡 Il peut s'agir d'opérateurs

### Réponse

**A.** `let inline ignore _ = ()`

> **Ignore** : [prim-types.fs#L459](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/prim-types.fs#L459)

**B.** `let id x = x`

> **Identity** : [prim-types.fs#L4831](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/prim-types.fs#L4831)

**C.** `let inline (<|) func arg = func arg`

> **Pipe Left** : [prim-types.fs#L3914](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/prim-types.fs#L3914)

**D.** `let inline (|>) arg func = func arg`

> **Pipe Right** : [prim-types.fs#L3908](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/prim-types.fs#L3908)

**E.** `let inline (>>) func1 func2 x = func2 (func1 x)`

> **Compose Right** : [prim-types.fs#L3920](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/prim-types.fs#L3920)

## Question 5

> Que signifie ces signatures de fonction : Combien de paramètres ? Leur type ? Le type du retour ? ⏱ 60’’

**A.** `int -> unit`

**B.** `unit -> int`

**C.** `string -> string -> string`

**D.** `('T -> bool) -> 'T list -> 'T list`

### Réponse

**A.** `int -> unit`

> 1 paramètre `int` - pas de valeur renvoyée

**B.** `unit -> int`

> Aucun paramètre - renvoie un `int`

**C.** `string -> string -> string`

> 2 paramètres `string` - renvoie une `string`

**D.** `('T -> bool) -> 'T list -> 'T list`

> 2 paramètres : un prédicat et une liste - renvoie une liste

💡 Il s'agit de la fonction `List.filter predicate list`

## Question 6

> Quelle est la signature de la fonction `h` ci-dessous ? ⏱ 30’’

```fsharp
let f x = x + 1
let g x y = $"%i{x} + %i{y}"
let h = f >> g
```

**A.** `int -> int`

**B.** `int -> string`

**C.** `int -> int -> string`

**D.** `int -> int -> int`

💡 `%i{a}` indique que `a` est un `int`

### Réponse

**C.** `int -> int -> string` ✅

`let f x = x + 1` → `f: (x: int) -> int` » `1` → `int` → `x: int` → `x + 1: int`

`let g x y = $"{+x} + {+y}"` → `(x: int) -> (y: int) -> string` » `%i{x}` → `int` » `$"..."` → `string`

`let h = f >> g` » `h` peut s'écrire `let h x y = g (f x) y` » Même `x` que `f` → `int`, même `y` que `g` → `int`

☝️ **Note :** cet exemple illustre une **mauvaise utilisation** de `>>` car les fonctions composées ont une arité différente (`f` a 1 paramètre, `g` en a 2)

## Question 7

> Combien vaut `f 2` ? ⏱ 10’’

```fsharp
let f = (-) 1;
f 2 // ?
```

**A.** `1`

**B.** `3`

**C.** `-1`

### Réponse

```fsharp
let f = (-) 1
f 2 // ?
```

**C.** `-1` ❗

* Contre-intuitif, non ? On s'attend à ce que f décrémente de 1.
* On comprend mieux ce qui se passe en écrivant `f` ainsi : `let f x = 1 - x`

💡 **Astuce :** la fonction qui décrémente de 1 peut s'écrire :

* `let f = (+) -1` _(cela marche ici car `+` est commutatif alors que `-` ne l'est pas)_
* `let f x = x - 1`
