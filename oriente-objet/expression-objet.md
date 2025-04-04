# Expression objet

## Présentation

Expression permettant d'implémenter à la volée un type abstrait

💡 Similaire à une classe anonyme en Java

```fsharp
let makeResource (resourceName: string) =
    printfn $"create {resourceName}"
    { new System.IDisposable with
        member _.Dispose() =
            printfn $"dispose {resourceName}" }
```

☝ La signature de `makeResource` est `string -> System.IDisposable`.

## Implémenter 2 interfaces

Possible mais 2e interface non consommable facilement et sûrement

```fsharp
let makeDelimiter (delim1: string, delim2: string, value: string) =
    { new System.IFormattable with
        member _.ToString(format: string, _: System.IFormatProvider) =
            if format = "D" then
                delim1 + value + delim2
            else
                value
      interface System.IComparable with
        member _.CompareTo(_) = -1 }

let o = makeDelimiter("<", ">", "abc")
// val o : System.IFormattable
let s = o.ToString("D", System.Globalization.CultureInfo.CurrentCulture)
// val s : string = "<abc>"
let i = (d :?> System.IComparable).CompareTo("cde")  // ❗ Dangereux
// val i : int = -1
```
