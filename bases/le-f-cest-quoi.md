# Le F♯, c'est quoi ?

## Points clés

Famille des langages Microsoft - Plateforme **.NET**

* Son concepteur : Don Syme @ Microsoft Research
* ≃ Implémentation de OCaml pour .NET
* ≃ Inspirée par Haskell _(Version 1.0 en 1990)_
* `dotnet new -lang F#`
* Inter-opérabilité entre projets/assemblies C♯ et F♯

Langage multi-paradigme _**Functional-first**_ et très concis

Là où C♯ est _imperative/object-oriented-first_ et plutôt verbeux _(même s’il s'inspire de F♯ pour être + succinct)_

## Historique

| Date     | C♯      | F♯     | .NET                              | Visual Studio |
| -------- | ------- | ------ | --------------------------------- | ------------- |
| 2002     | C♯ 1.0  |        | .NET Framework 1.0                | VS .NET 2002  |
| **2005** |         | F♯ 1.x | .NET Framework 1.0                | VS 2005 ?     |
| 2010     | C♯ 4.0  | F♯ 2.0 | .NET Framework 4                  | VS 2010       |
| 2015     | C♯ 6.0  | F♯ 4.0 | .NET Framework 4.6, .NET Core 1.x | VS 2015       |
| 2018     | C♯ 7.3  | F♯ 4.5 | .NET Framework 4.8, .NET Core 2.x | VS 2017       |
| 2019     | C♯ 8.0  | F♯ 4.7 | .NET Core 3.x                     | VS 2019       |
| 2020     | C♯ 9.0  | F♯ 5.0 | .NET 5.0                          | VS 2019       |
| 2021     | C♯ 10.0 | F♯ 6.0 | .NET 6.0 (LTS)                    |               |
| ...      | ...     | ...    | ...                               |               |
| 2024     | C♯ 13.0 | F♯ 9.0 | .NET 9.0                          |               |

🔗 Détails de chaque version : [https://learn.microsoft.com/en-us/dotnet/fsharp/whats-new/](https://learn.microsoft.com/en-us/dotnet/fsharp/whats-new/)

## Éditeurs / IDE

* VsCode + [Ionide](https://marketplace.visualstudio.com/items?itemName=Ionide.Ionide-fsharp)\
  → ☝ Moins d'aide (autocompletion, ajout d'imports, ...) qu'un IDE\
  → ☝ Permissif : ne remonte pas toujours toutes les erreurs de compilation
* Visual Studio / Rider\
  → ☝ Moins de refacto que pour C♯
* [https://try.fsharp.org/](le-f-cest-quoi.md#points-cles)\
  → Online [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) avec exemples

## Setup du poste

[https://docs.microsoft.com/en-us/learn/modules/fsharp-first-steps/ 4-set-up-development-environment-exercise](https://docs.microsoft.com/en-us/learn/modules/fsharp-first-steps/4-set-up-development-environment-exercise)

* Installation du SDK .NET (5.0 ou 6.0)
* Installation de VScode
* Ajout de l'extension Ionide-fsharp

_(Optionnel)_ extensions complémentaires : \
[https://www.compositional-it.com/news-blog/fantastic-f-and-azure-developer-extensions-for-vscode/](le-f-cest-quoi.md#points-cles)

## F♯ interactive _(FSI)_

* REPL disponible dans VS, Rider, vscode + `dotnet fsi`
* Usage : vérifier en live un bout de code
  * 💡 Terminer expression par `;;` pour l'évaluer
* Existe depuis le départ _(cf. aspect scripting du F#)_
  * _C♯ interactive_ + récent (VS 2015 Update 1)
* Alternative : [LINQPad](https://www.linqpad.net/)

## Types de fichier

4 types de fichier : `.fs`, `.fsi`, `.fsx`, `.fsproj`

* Mono langage : purement pour/en F♯
* Standalone vs Projet

### Fichier standalone

* Fichier de script `.fsx`
  * Exécutable _(d'où le **x**)_ dans la console FSI
  * Indépendant mais peut référencer autre fichier, DLL, package NuGet.

### Fichiers de projet

* En C♯ : `.sln` contient `.csproj` qui contient `.cs`
* En F♯ : `.sln` contient `.fsproj` qui contient `.fs` et `.fsi`
  * Fichier projet `.fsproj`
  * Fichier de code `.fs`
  * Fichier de signature `.fsi` _(**i** comme interface)_
    * Associé à un fichier `.fs` de même nom
    * Optionnel et plutôt rare -- + d'info : [MSDN](https://docs.microsoft.com/fr-fr/dotnet/fsharp/language-reference/signature-files)
    * Renforcer encapsulation _(idem `.h` en C)_
    * Séparer longue documentation (xml-doc)

💡 **Interop C♯ - F♯** = Il est facile de faire cohabiter et inter-référencer des projets`.csproj` et `.fsproj` dans une `.sln`

{% content-ref url="../programmation-asynchrone/interop-avec-la-tpl-.net.md" %}
[interop-avec-la-tpl-.net.md](../programmation-asynchrone/interop-avec-la-tpl-.net.md)
{% endcontent-ref %}

### Projet F♯

Création dans un IDE ou avec la CLI `dotnet` :

* `dotnet new -l` : lister les types de projet supportés
* `dotnet new console --language F# -o MyFSharpApp`
  * Création d'un projet console nommé `MyFSharpApp`
  * `--language F#` à spécifier ; sinon C#
* `dotnet build` : builder le projet
* `dotnet run` : builder le projet et lancer l'exécutable résultant
