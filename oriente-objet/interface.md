# Interface

## Syntaxe

Idem classe abstraite avec :

* Que des membres abstraits, définis par leur signature
* Sans l'attribut `[<AbstractClass>]`&#x20;
* Avec l'attribut `[<Interface>]` de manière optionnelle

```fsharp
type [accessibility-modifier] interface-name =
    abstract memberN : [ argument-typesN -> ] return-typeN
```

* Le **nom** d'une interface commence par `I` pour suivre la convention .NET
* Les **arguments** des méthodes peuvent être nommés :&#x20;

```fsharp
[<Interface>]
type IPrintable =
    abstract member Print : format:string -> unit
```

On peut aussi utiliser la syntaxe verbeuse avec un bloc `interface ... end`, \
→ Non idiomatique sauf dans le cas d'une interface sans membre a.k.a _marker interface_.

```fsharp
type IMarker = interface end
```

## Implémentation

2 manières d'implémenter une interface :

1. Dans un type _(comme en C♯)_
2. Dans une expression objet 📍

### Dans un type

```fsharp
type IPrintable =
    abstract member Print : unit -> unit

type Range = { Min: int; Max: int } with
    interface IPrintable with
        member this.Print() = printfn $"[{this.Min}..{this.Max}]"
```

:warning: **Piège :** mot clé `interface` en F♯  ≠ mot clé `interface` en C♯, Java, TS  ≡ mot clé `implements` Java, TS

### Dans une expression objet

```fsharp
type IConsole =
    abstract ReadLine : unit -> string
    abstract WriteLine : string -> unit

let console =
    { new IConsole with
        member this.ReadLine () = Console.ReadLine ()
        member this.WriteLine line = printfn "%s" line }
```

## Implémentation par défaut

F♯ 5.0 supporte les interfaces définissant des méthodes avec implémentations par défaut écrites en C♯ 8+ mais ne permet pas de les définir.

:warning: Mot clé `default` : supporté que dans les classes, pas dans les interfaces !

## Une interface F♯ est explicite

Implémentation d'une interface en F♯ ≡ Implémentation explicite d'une interface en C♯

→ Les méthodes de l'interface ne sont consommables que par _upcasting_ :

```fsharp
type IPrintable =
    abstract member Print : unit -> unit

type Range = { Min: int; Max: int } with
    interface IPrintable with
        member this.Print() = printfn $"[{this.Min}..{this.Max}]"

let range = { Min = 1; Max = 5 }
(range :> IPrintable).Print()  // Opérateur `:>` de upcast 📍
// [1..5]
```

## Implémenter une interface générique

```fsharp
type IValue<'T> =
    abstract member Get : unit -> 'T

type BiValue() =
    interface IValue<int> with
        member _.Get() = 1
    interface IValue<string> with
        member _.Get() = "hello"

let o = BiValue()
let i = (o :> IValue<int>).Get() // 1
let s = (o :> IValue<string>).Get() // "hello"
```

## Héritage

Défini avec mot clé `inherit`

```fsharp
type Base(x: int) =
    do
        printf "Base: "
        for i in 1..x do printf "%d " i
        printfn ""

type Child(y: int) =
    inherit Base(y * 2)
    do
        printf "Child: "
        for i in 1..y do printf "%d " i
        printfn ""

let child = Child(1)

// Base: 1 2 3 4
// Child: 1
```
