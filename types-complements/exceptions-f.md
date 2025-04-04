# Exceptions F\#

F# supporte 2 types d'exception : &#x20;

1. Les exceptions .NET&#x20;
2. Les exceptions F#

Les exceptions F# sont une extension des exceptions .NET présentant 2 améliorations dans l'esprit des types union :&#x20;

* Définition succincte
* Support du pattern matching exhaustif dans les expressions `try...with`

Exemple :

```fsharp
exception Error1 of string
exception Error2 of string * int

let function1 x y =
   try
      if x = y then raise (Error1 "x")
      else raise (Error2("x", 10))
   with
      | Error1 str -> printfn "Error1 %s" str
      | Error2(str, i) -> printfn "Error2 %s %d" str i

function1 10 10
// Error1 x

function1 9 2
// Error2 x 10
```

Pour comparer, voici un exemple de `try...with` avec une exception .NET :&#x20;

```fsharp
let divide1 x y =
   try
      Some (x / y)
   with
      | :? System.DivideByZeroException ->
        printfn "Division by zero!"
        None

let result1 = divide1 100 0
```

**Note :** les exceptions F# sont compilées en .NET de manière classique : \
\- En tant que classes dérivant de `Exception`\
\- Supportant l'égalité structurelle via l'interface `IStructuralEquatable`\
→ Cf. [sharplab.io](https://sharplab.io/#v2:DYLgZgzgNAJiDUAfApgDwMbIA4BcCWA9gHYAEAogE4UEUCMJBYJEOFeRA5gLABQQA===)

