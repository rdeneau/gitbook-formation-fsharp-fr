# Signature

## De void à unit

### Problèmes avec `void` en C♯

`void` oblige à faire du spécifique = 2 fois + de boulot 😠

* 2 types de délégués : `Action` vs `Func<T>`
* 2 types de tâches : `Task` vs `Task<T>`

### Exemple : `ITelemetry`

```csharp
interface ITelemetry
{
  void Run(Action action);
  T Run<T>(Func<T> func);

  Task RunAsync(Func<Task> asyncAction);
  Task<T> RunAsync<T>(Func<Task<T>> asyncFunc);
}
```

### Type Void

☝ Le problème avec `void`, c'est que ce n'est ni un type, ni une valeur.

💡 Si on avait le type `Void` suivant : \
→ Sans donnée\
→ Instance unique _(Singleton)_

```csharp
public class Void
{
    public static readonly Void Instance = new Void();

    private Void() {}
}
```

### Simplification de `ITelemetry`

On peut définir les _helpers_ suivants pour convertir vers `Void` :

```csharp
public static class VoidExtensions
{
    // Action -> Func<Void>
    public static Func<Void> AsFunc(this Action action)
    {
        action();
        return Void.Instance;
    }

    // Func<Task> -> Func<Task<Void>>
    public async static Func<Task<Void>> AsAsyncFunc(this Func<Task> asyncAction)
    {
        await asyncAction();
        return Void.Instance;
    }
}
```

Alors, on peut écrire une implémentation par défaut (C♯ 8) pour 2 des 4 méthodes :

```csharp
interface ITelemetry
{
    void Run(Action action) =>
        Run(action.AsFunc());

    T Run<T>(Func<T> func);

    Task RunAsync(Func<Task> asyncAction) =>
        RunAsync(asyncAction.AsAsyncFunc());

    Task<T> RunAsync<T>(Func<Task<T>> asyncFunc);
}
```

### Type `unit`

En F♯ `Void` s'appelle `unit` car le type n'a qu'une seule instance, notée `()` et qui se manipule comme tout autre littéral.

### Impact sur la signature des fonctions

Plutôt que des fonctions `void`, on a des fonctions avec type de retour `unit`

`unit` sert aussi à modéliser des fonctions sans paramètre :

```fsharp
let oneParam arg = ...
let noParam () = ... // 👈 Avec
let noParam2() = ... // 👈 ou sans espace
```

💡 Intérêt de la notation `()` : on dirait une fonction C♯.

:warning: **Attention :** on a vite fait d'oublier les `()` !

* Oubli dans la déclaration : \
  → `let noParam = ... // T`  ❌ simple valeur plutôt que fonction !
* Oubli dans l'appel :\
  → `let res = noParam // unit -> T`   ❌ Alias de la fonction sans l'exécuter !\
  → `let res = noParam () // T`  ✅

## Fonction `ignore`

En F♯, "tout est expression" \
→ Aucune valeur n'est ignorée, sauf `()`/`unit`  \
→ Au début d'une expression ou entre plusieurs `let` bindings, on peut insérer des expressions valant `()`/`unit`, par exemple `printf "mon message"`

**Problème :** on appelle une fonction qui déclenche un effet de bord mais également qui renvoie une valeur qui ne nous intéresse pas.\
→ Ex 1 : fonction `save` qui enregistre en base de données et qui renvoie `true` ou `false` \
→ Ex 2 : méthode d'un fluent builder mutable : la méthode mute l'objet `this` et le renvoie _(pour permettre d'enchaîner l'appel à d'autres méthodes du builder)_

**Solution 1 :** utiliser un "discard binding" `let _ = ...`

**Solution 2 :** utiliser la fonction `ignore` de signature `'a -> unit`\
→ Quelle que soit la valeur fournie en paramètre, elle l'ignore et renvoie `()`.

```fsharp
let save entity = true

let problem =
    save "bonjour"
    //~~~~~~~~~~~~
    //⚠️ Warning FS0020: Le résultat de cette expression a le type 'bool' et est implicitement ignoré.

    "ok" // Valeur finale quelconque

let solution1 =
    let _ = save "bonjour" // 👌
    "ok"

let solution2 =
    save "bonjour" |> ignore // 👍
    "ok"
```

> :warning: **Attention :** n'utiliser `ignore` qu’avec précaution, en vérifiant le type de retour.\
> Exemple d'utilisation erronée :&#x20;

```fsharp
[ 1..5 ]
|> Seq.map (fun i -> save i)
|> ignore
```

Ici, l'auteur du code pensait que les `save`s étaient effectués ce qui n'est pas le cas avec `Seq.map`. Il faut plutôt utiliser `Seq.iter` (par exemple) qui n'a pas besoin de `ignore` car elle renvoie `unit`, indiquant ainsi qu'elle procède à des effets de bord.

Autres exemples :&#x20;

```fsharp
// Effet de bord / file system
System.IO.Directory.CreateDirectory folder |> ignore
// Ignore le DirectoryInfo renvoyé

// Fluent builder:
let configureAppConfigurationForEnvironment (env: Environment) (basePath: string) (builder: IConfigurationBuilder) =
    builder
        .SetBasePath(basePath)
        .AddJsonFile("appsettings.json", optional = false, reloadOnChange = true)
        .AddJsonFile($"appsettings.{env.name}.json", optional = false, reloadOnChange = true)
    |> ignore

// Event handler:
textbox.onValueChanged(ignore)
```

## Notation fléchée

* Fonction à 0 paramètre : `unit -> TResult`
* Fonction à 1 paramètre : `T -> TResult`
* Fonction à 2 paramètres : `T1 -> T2 -> TResult`
* Fonction à 3 paramètres : `T1 -> T2 -> T3 -> TResult`

❓ **Quiz** : Pourquoi des `->` entre les paramètres ? Quel est le concept sous-jacent ?

## Curryfication

> ☝ _Currying_ en anglais

### Définition

Consiste à transformer :

* une fonction prenant N paramètres\
  `Func<T1, T2, ...Tn, TReturn>` en C♯
* en une chaîne de N fonctions prenant 1 paramètre\
  `Func<T1, Func<Tn, ...Func<Tn, TReturn>>>`

### Application partielle

Appel d'une fonction avec moins d'arguments que son nombre de paramètres

* Possible grâce à la curryfication
* Renvoie fonction prenant en paramètre le reste d'arguments à fournir

```fsharp
// Template à 2 paramètres
let insideTag (tagName: string) (content: string) =
    $"<{tagName}>{content}</{tagName}>"

// Helpers à 1! paramètre `content`, `tagName` étant fixé par application partielle
let emphasize = insideTag "em"     // `tagName` fixé à la valeur "em"
let strong    = insideTag "strong" // `tagName` fixé à la valeur "strong"

// Equivalent - élégant mais + explicite
let em content = insideTag "em" content
```

:warning: **Attention :** l'application partielle impacte la signature :&#x20;

<pre><code><strong>val insideTag: tagName: string -> content: string -> string
</strong>val emphasize: (string -> string)  👈 1 👈 2
val em: content: string -> string
</code></pre>

1. Perte du nom des paramètres restant dans la signature (`content`)
2. Signature d'une valeur de type fonction d'où les parenthèses (`(string -> string)`)

### Syntaxe des fonctions F♯

Paramètres séparés par des espaces \
→ Indique que les fonctions sont curryfiées \
→ D'où les `->` dans la signature entre les paramètres

```fsharp
let fn () = result         // unit -> TResult
let fn arg = ()            // T    -> unit
let fn arg = result        // T    -> TResult

let fn x y = (x, y)        // T1 -> T2 -> (T1 * T2)

// Equivalents, explicitement curryfiés :
let fn x = fun y -> (x, y) // 1. Avec une lambda
let fn x =                 // 2. Avec une sous-fonction nommée
    let fn' y = (x, y)     // N.B. `x` vient du scope
    fn'
```

### IntelliSense Ionide

Dans VsCode avec Ionide, l'IntelliSense fournit un descriptif plus lisible des fonctions, en mettant chaque argument dans une nouvelle ligne :&#x20;

```fsharp
let pair x y = (x, y)
// val pair:
//    x: 'a ->
//    y: 'b
//    -> 'a * 'b

let triplet x y z = (x, y, z)
// val triplet:
//    x: 'a ->
//    y: 'b ->
//    z: 'c
//    -> 'a * 'b * 'c
```

### Compilation .NET d’une fonction curryfiée

☝ Une fonction curryfiée est compilée différemment selon comment elle est appelée !

* De base, elle est compilée en méthode avec paramètres tuplifiés \
  → Vue comme méthode normale quand consommée en C♯

Exemple : F♯ puis équivalent C♯ _(version simplifiée de_ [_SharpLab_](https://sharplab.io/#v2:DYLgZgzgNAJiDUAfAtgexgV2AUwAQEFcBeAWAChdLccAXXAQxhlwA9cBPY13eD8q6tjoA3esAx4iuAEy5EAPgZNcARnJA===)_)_ :

```fsharp
module A =
    let add x y = x + y
    let value = 2 |> add 1
```

```csharp
public static class A
{
    public static int add(int x, int y) => x + y;
    public static int value => 3;
}
```

* Lorsqu'elle est appliquée partiellement, elle est compilée sous forme de classe pseudo `Delegate` étendant `FSharpFunc<int, int>` avec une méthode `Invoke` qui encapsule les 1ers arguments fournis

Exemple : F♯ puis équivalent C♯ _(version simplifiée de_ [_SharpLab_](https://sharplab.io/#v2:DYLgZgzgNAJiDUAfAtgexgV2AUwAQEFcBeAWAChdLccAXXAQxhlwA9cBPY13eD8q6tjqMYAeQB2eIgya4AjEA===)_)_ :

```fsharp
module A =
    let add x y = x + y
    let addOne = add 1
```

```csharp
public static class A
{
    internal sealed class addOne@3 : FSharpFunc<int, int>
    {
        internal static readonly addOne@3 @_instance = new addOne@3();

        internal addOne@3() { }

        public override int Invoke(int y) => 1 + y;
    }

    public static FSharpFunc<int, int> addOne => addOne@3.@_instance;

    public static int add(int x, int y) => x + y;
}
```

## Conception unifiée des fonctions

Le type `unit` et la curryfication permettent de concevoir les fonctions simplement comme :

* **Prend un seul paramètre** de type quelconque
  * y compris `unit` pour une fonction "sans paramètre"
  * y compris une autre fonction _(callback)_
* **Renvoie une seule valeur** de type quelconque
  * y compris `unit` pour une fonction "ne renvoyant rien"
  * y compris une autre fonction

👉 **Signature universelle** d'une fonction en F♯ : `'T -> 'U`

## Ordre des paramètres

Entre C♯ et F♯, on ne place pas au même endroit le paramètre concernant l'objet principal (le `this` dans le cas d'une méthode) :

* Dans méthode extension C♯, l'objet `this` est le 1er paramètre
  * Ex : `items.Select(x => x)`
* En F♯, l'objet principal est _plutôt_ le **dernier paramètre** : \
  &#xNAN;_(ce qui s'appelle le style data-last)_
  * Ex : `List.map (fun x -> x) items`

Style _data-last_ favorise :

* Pipeline : `items |> List.map square |> List.sum`
* Application partielle : `let sortDesc = List.sortBy (fun i -> -i)`
* Composition de fonctions appliquées partiellement jusqu'au param "_data_"
  * `(List.map square) >> List.sum`

:warning: Friction avec BCL .NET car plutôt _data-first_

☝ Solution : wrapper dans une fonction avec params dans ordre sympa en F♯

```fsharp
let startsWith (prefix: string) (value: string) =
    value.StartsWith(prefix)
```

💡 **Tips** : utiliser `Option.defaultValue` plutôt que `defaultArg` avec les options

* Fonctions font la même chose mais params `option` et `value` sont inversés
* `defaultArg option value` : param `option` en 1er 😕
* `Option.defaultValue value option` : param `option` en dernier 👍

De même, préférer mettre **en 1er** les paramètres les + statiques = Ceux susceptibles d'être prédéfinis par application partielle

Ex : "dépendances" qui seraient injectées dans un objet en C♯

👉 Application partielle = moyen de simuler l'injection de dépendances
