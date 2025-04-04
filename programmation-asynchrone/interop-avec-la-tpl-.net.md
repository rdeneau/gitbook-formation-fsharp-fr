---
description: 'TPL : Task Parallel Library'
---

# Interop avec la TPL .NET

## Interaction avec librairie .NET

Librairies asynchrones en .NET et pattern `async`/`await` C♯ : \
→ Basés sur **TPL** et le type `Task`

Passerelles avec worflow asynchrone F♯ :

* Fonctions `Async.AwaitTask` et `Async.StartAsTask`
* Bloc `task {}`

## Fonctions passerelles

`Async.AwaitTask: Task<'T> -> Async<'T>` \
→ Consommer une librairie .NET asynchrone dans bloc `async`

`Async.StartAsTask: Async<'T> -> Task<'T>` \
→ Lancer un calcul async sous forme de `Task`

```fsharp
let getValueFromLibrary param = async {
    let! value = DotNetLibrary.GetValueAsync param |> Async.AwaitTask
    return value
}

let computationForCaller param =
    async {
        let! result = getAsyncResult param
        return result
    } |> Async.StartAsTask
```

## Bloc `task {}`

> Permet de consommer directement une librairie .NET asynchrone en ne faisant qu'un seul `Async.AwaitTask` plutôt que 1 à chaque méthode appelée

💡 Disponible en F♯ 6 ou via package nuget [Ply](https://github.com/crowded/ply)

```fsharp
#r "nuget: Ply"
open FSharp.Control.Tasks

task {
    use client = new System.Net.Http.HttpClient()
    let! response = client.GetStringAsync("https://www.google.fr/")
    response.Substring(0, 300) |> printfn "%s"
}  // Task<unit>
|> Async.AwaitTask  // Async<unit>
|> Async.RunSynchronously
```

## `Async` _vs_ `Task`

**1. Mode de démarrage du calcul**

`Task` = _hot tasks_ → calculs démarrés immédiatement❗

`Async` = _task generators_ = spécification de calculs, indépendante du démarrage \
→ Approche fonctionnelle : sans effet de bord ni mutation, composabilité \
→ Contrôle du mode de démarrage : quand et comment 👍

**2. Support de l'annulation**

`Task` : en ajoutant un paramètre `CancellationToken` aux méthodes async \
→ Oblige à tester manuellement si token est annulé = fastidieux + _error prone❗_

`Async` : support automatique dans les calculs - token à fournir au démarrage 👍

## Pièges du pattern `async`/`await` en C♯

### Piège 1 - Vraiment asynchrone ?

En C♯ : méthode `async` reste sur le thread appelant jusqu'au 1er `await` \
→ Sentiment trompeur d'être asynchrone dans toute la méthode

```csharp
async Task WorkThenWait() {
    Thread.Sleep(1000);           // ⚠️ Bloque thread appelant !
    await Task.Delay(1000);       // Vraiment async à partir d'ici 🤔
}
```

En F♯ : `async` ne définit pas une fonction mais un **bloc**

```fsharp
let workThenWait () =
    Thread.Sleep(1000)
    async { do! Async.Sleep(1000) }   // Async que dans ce bloc 🧐
```

### Préconisation pour fonction asynchrone en F♯

Fonction asynchrone = renvoyant un `Async<_>` \
→ On s'attend à ce qu'elle soit **totalement** asynchrone \
→ Fonction précédente `workThenWait` ne respecte pas cette attente

☝ **Préconisation :** » Mettre tout le corps de la fonction asynchrone dans un bloc `async`

```fsharp
let workThenWait () = async {
    Thread.Sleep(1000)
    printfn "work"
    do! Async.Sleep(1000)
}
```

### Piège 2 - Omettre le `await` en C♯

```csharp
async Task PrintAfterOneSecond(string message) {
    await Task.Delay(1000);
    Console.WriteLine($"[{DateTime.Now:T}] {message}");
}

async Task Main() {
    PrintAfterOneSecond("Before"); // ⚠️ Manque `await`→ warning CS4014
    Console.WriteLine($"[{DateTime.Now:T}] After");
    await Task.CompletedTask;
}
```

Cela compile 📍 et produit un résultat inattendu _After_ avant _Before❗_

```
[11:45:27] After
[11:45:28] Before
```

Équivalent en F♯ :

```fsharp
let printAfterOneSecond message = async {
    do! Async.Sleep 1000
    printfn $"[{DateTime.Now:T}] {message}"
}

async {
    printAfterOneSecond "Before" // ⚠️ Manque `do!` → warning FS0020
    printfn $"[{DateTime.Now:T}] After"
} |> Async.RunSynchronously
```

Cela compile aussi 📍 et produit un autre résultat inattendu : pas de _Before❗_

```
[11:45:27] After
```

#### Étude des warnings

Les exemples précédemment compilent mais avec des gros _warnings_ !

En C♯, le [_warning CS4014_](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/cs4014) indique&#x20;

> _Because this call is not awaited, execution of the current method continues before the call is completed. Consider applying the `await` operator..._

En F♯, le _warning FS0020_ est accompagné du message&#x20;

> _The result of this expression has type `Async<unit>` and is implicitly ignored._\
> _Consider using `ignore` to discard this value explicitly..._

☝ **Préconisation :** veillez à **toujours** traiter ce type de _warning_ ! _C'est encore + crucial en F♯ où la compilation est + délicate._
