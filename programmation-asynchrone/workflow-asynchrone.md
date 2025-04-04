# Workflow asynchrone

## Besoins

1. Ne pas bloquer le thread courant en attendant un calcul long
2. Permettre calculs en parallèle
3. Indiquer qu'un calcul peut prendre du temps

## Type `Async<'T>`

* Représente un calcul asynchrone
* Similaire au pattern `async/await` avant l'heure 📆
  * 2007 : `Async<'T>` F♯
  * 2012 : `Task<T>` .NET et pattern `async`/`await`
  * 2017 : `Promise` JavaScript et pattern `async`/`await`

## Méthodes renvoyant un objet `Async`

`Async.AwaitTask(task: Task or Task<'T>) : Async<'T>`

* Conversion d'une `Task` (.NET) en `Async` (F♯)

`Async.Sleep(milliseconds or TimeSpan) : Async<unit>`

* ≃ `await Task.Delay()` ≠ `Thread.Sleep` → ne bloque pas le thread courant

[FSharp.Control.CommonExtensions](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-control-commonextensions.html) : étend le type `System.IO.Stream`

* `AsyncRead(buffer: byte[], ?offset: int, ?count: int) : Async<int>`
* `AsyncWrite(buffer: byte[], ?offset: int, ?count: int) : Async<unit>`

[FSharp.Control.WebExtensions](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-control-webextensions.html) : étend le type `System.Net.WebClient`

* `AsyncDownloadData(address: Uri) : Async<byte[]>`
* `AsyncDownloadString(address: Uri) : Async<string`

## Lancement d'un calcul async

`Async.RunSynchronously(calc: Async<'T>, ?timeoutMs: int, ?cancellationToken) : 'T` → Attend la fin du calcul mais bloque le thread appelant ! (≠ `await` C♯) ⚠️

`Async.Start(operation: Async<unit>, ?cancellationToken) : unit` \
→ Exécute l'opération en background _(sans bloqué le thread appelant)_ \
⚠️ Si une exception survient, elle est "avalée" !

`Async.StartImmediate(calc: Async<'T>, ?cancellationToken) : unit` \
→ Exécute le calcul dans le thread appelant ! \
💡 Pratique dans une GUI pour la mettre à jour : barre de progression...

`Async.StartWithContinuations(calc, continuations..., ?cancellationToken)` \
→ Idem `Async.RunSynchronously` \
⚠️ ... avec 3 _callbacks_ de continuation : en cas de succès ✅, d'exception 💥 et d'annulation 🛑

## Bloc `async { expression }`

_A.k.a. Async workflow_

Syntaxe pour écrire de manière séquentielle un calcul asynchrone → Le résultat du calcul est wrappé dans un objet `Async`

**Mots clés**

* `return` → valeur finale du calcul - `unit` si omis
* `let!` _(prononcer « let bang »)_    → accès au résultat d'un sous-calcul async _(≃ `await` en C♯)_
* `use!` → idem `use` _(gestion d'un `IDisposable`)_ + `let!`
* `do!` → idem `let!` pour calcul async sans retour (`Async<unit>`)

```fsharp
let repeat (computeAsync: int -> Async<string>) times = async {
    for i in [ 1..times ] do
        printf $"Start operation #{i}... "
        let! result = computeAsync i
        printfn $"Result: {result}"
}

let basicOp (num: int) = async {
    do! Async.Sleep 150
    return $"{num} * ({num} - 1) = {num * (num - 1)}"
}

repeat basicOp 5 |> Async.RunSynchronously

// Start operation #1... Result: 1 * (1 - 1) = 0
// Start operation #2... Result: 2 * (2 - 1) = 2
// Start operation #3... Result: 3 * (3 - 1) = 6
// Start operation #4... Result: 4 * (4 - 1) = 12
// Start operation #5... Result: 5 * (5 - 1) = 20
```

## Usage inapproprié de `Async.RunSynchronously`

`Async.RunSynchronously` lance le calcul et renvoie son résultat MAIS en bloquant le thread appelant ! Ne l'utiliser qu'en « bout de chaîne » et pas pour _unwrap_ des calculs asynchrones intermédiaires ! Utiliser plutôt un bloc `async`.

```fsharp
// ❌ À éviter
let a = calcA |> Async.RunSynchronously
let b = calcB a |> Async.RunSynchronously
calcC b

// ✅ À préférer
async {
    let! a = calcA
    let! b = calcB a
    return calcC b
} |> Async.RunSynchronously
```

## Calculs en parallèle

### `Async.Parallel`

`Async.Parallel(computations: seq<Async<'T>>, ?maxBranches) : Async<'T[]>`

≃ `Task.WhenAll` : modèle [Fork-Join](https://en.wikipedia.org/wiki/Fork%E2%80%93join_model)

* _Fork_ : calculs lancés en parallèle
* Attente de la terminaison de tous les calculs
* _Join_ : agrégation des résultats _(qui sont du même type)_
  * dans le même ordre que les calculs

```fsharp
let downloadSite (site: string) = async {
    do! Async.Sleep (100 * site.Length)
    printfn $"{site} ✅"
    return site.Length
}

[ "google"; "msn"; "yahoo" ]
|> List.map downloadSite  // string list
|> Async.Parallel         // Async<string[]>
|> Async.RunSynchronously // string[]
|> printfn "%A"

// msn ✅
// yahoo ✅
// google ✅
// [|6; 3; 5|]
```

### `Async.StartChild`

`Async.StartChild(calc: Async<'T>, ?timeoutMs: int) : Async<Async<'T>>`

Permet de lancer en parallèle plusieurs calculs \
→ ... dont les résultats sont de types différents _(≠ `Async.Parallel`)_

S'utilise dans bloc `async` avec 2 `let!` par calcul enfant _(cf. `Async<Async<'T>>`)_

Annulation conjointe 📍 [#annulation-dune-tache](workflow-asynchrone.md#annulation-dune-tache "mention")\
→ Le calcul enfant partage le jeton d’annulation du calcul parent

**Exemple :**

Soit le fonction `delay` \
→ qui renvoie la valeur spécifiée `x` \
→ au bout de `ms` millisecondes

```fsharp
let delay (ms: int) x = async {
    do! Async.Sleep ms
    return x
}
```

💡 Minutage avec la directive FSI `#time` _(_[_doc_](https://docs.microsoft.com/en-us/dotnet/fsharp/tools/fsharp-interactive/#f-interactive-directive-reference)_)_

```fsharp
#time "on"  // --> Minutage activé
"a" |> delay 100 |> Async.RunSynchronously // Réel : 00:00:00.111, Proc...
#time "off" // --> Minutage désactivé
```

```fsharp
let inSeries = async {
    let! result1 = "a" |> delay 100
    let! result2 = 123 |> delay 200
    return (result1, result2)
}

let inParallel = async {
    let! child1 = "a" |> delay 100 |> Async.StartChild
    let! child2 = 123 |> delay 200 |> Async.StartChild
    let! result1 = child1
    let! result2 = child2
    return (result1, result2)
}
```

```fsharp
#time "on"
inSeries |> Async.RunSynchronously    // Réel : 00:00:00.317, ...
#time "off"
#time "on"
inParallel |> Async.RunSynchronously  // Réel : 00:00:00.205, ...
#time "off"
```

## Annulation d'une tâche

Se base sur un `CancellationToken/Source` par défaut ou explicite :

* `Async.RunSynchronously(computation, ?timeout, ?cancellationToken)`
* `Async.Start(computation, ?cancellationToken)`

Déclencher l'annulation

* Token explicite + `cancellationTokenSource.Cancel()`
* Token explicite avec timeout `new CancellationTokenSource(timeout)`
* Token par défaut : `Async.CancelDefaultToken()` → `OperationCanceledException`💥

Vérifier l'annulation

* Implicite : à chaque mot clé dans bloc async : `let`, `let!`, `for`...
* Explicite local : `let! ct = Async.CancellationToken` puis `ct.IsCancellationRequested`
* Explicite global : `Async.OnCancel(callback)`

**Exemple :**

```fsharp
let sleepLoop = async {
    let stopwatch = System.Diagnostics.Stopwatch()
    stopwatch.Start()
    let log message = printfn $"""   [{stopwatch.Elapsed.ToString("s\.fff")}] {message}"""

    use! __ = Async.OnCancel (fun () ->
        log $"  Cancelled ❌")

    for i in [ 1..5 ] do
        log $"Step #{i}..."
        do! Async.Sleep 500
        log $"  Completed ✅"
}

open System.Threading

printfn "1. RunSynchronously:"
Async.RunSynchronously(sleepLoop)

printfn "2. Start with CancellationTokenSource + Sleep + Cancel"
use manualCancellationSource = new CancellationTokenSource()
Async.Start(sleepLoop, manualCancellationSource.Token)
Thread.Sleep(1200)
manualCancellationSource.Cancel()

printfn "3. Start with CancellationTokenSource with timeout"
use cancellationByTimeoutSource = new CancellationTokenSource(1200)
Async.Start(sleepLoop, cancellationByTimeoutSource.Token)
```

Résultat :

```
1. RunSynchronously:
   [0.009] Step #1...
   [0.532]   Completed ✅
   [0.535] Step #2...
   [1.037]   Completed ✅
   [1.039] Step #3...
   [1.543]   Completed ✅
   [1.545] Step #4...
   [2.063]   Completed ✅
   [2.064] Step #5...
   [2.570]   Completed ✅
2. Start with CancellationTokenSource + Sleep + Cancel
   [0.000] Step #1...
   [0.505]   Completed ✅
   [0.505] Step #2...
   [1.011]   Completed ✅
   [1.013] Step #3...
   [1.234]   Cancelled ❌
3. Start with CancellationTokenSource with timeout
... idem 2.
```
