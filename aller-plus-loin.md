# 🦚 Aller plus loin

* [F# 6.0](https://devblogs.microsoft.com/dotnet/whats-new-in-fsharp-6/) : sorti en [novembre 2021](https://devblogs.microsoft.com/dotnet/fsharp-6-is-officially-here/)
* [F# 7.0](https://devblogs.microsoft.com/dotnet/announcing-fsharp-7/) : sorti en novembre 2022
* 📜 [Choisir entre C# et F#](https://www.partech.nl/nl/publicaties/2021/06/key-differences-between-c-sharp-and-f-sharp) _(Juin 2021)_
* 📜 [Effective F#, tips and tricks](https://gist.github.com/swlaschin/31d5a0a2c4478e82e3ed60d653c0206b) de Scott Wlaschin
* 📜 [Writing high performance F# code](https://bartoszsypytkowski.com/writing-high-performance-f-code/) de Bartosz Sypytkowski
* _Domain modeling_
  * 📹 [Domain Modeling Made Functional](https://www.youtube.com/watch?v=2JB1_e5wZmU) _(Dec 2019)_ de Scott Wlaschin
  * 📘 [Domain modeling made functional](https://www.goodreads.com/book/show/34921689-domain-modeling-made-functional) _(Nov 2017)_ de Scott Wlaschin
  * 📜 [Alternate Ways of Creating Single-Case Unions in F#](https://trustbit.tech/blog/2021/05/01/alternate-ways-of-creating-single-case-discriminated-unions-in-f-sharp) _(May 2021)_, par Ian Russel
  * 📜 Single-Case Unions: [Part 1](https://paul.blasuc.ci/posts/really-scu.html) _(May 2021)_, [Part 2](https://paul.blasuc.ci/posts/even-more-scu.html) _(Jul 2021)_ de Paul Blasucci
* Outils
  * [Paket](https://fsprojects.github.io/Paket/index.html) plutôt que Nuget
  * [FAKE](https://fake.build/) : outil de build
  * [Fantomas](https://github.com/fsprojects/fantomas) : formateur de code F#
* Tests unitaires
  * Librairies : xUnit + [FsUnit](http://fsprojects.github.io/FsUnit/), [Unquote](https://github.com/SwensenSoftware/unquote), [Expecto](https://github.com/haf/expecto)
  * [Review: F# unit testing frameworks and libraries](https://devonburriss.me/review-fsharp-test-libs/)
  * BDD : [TickSpec](https://github.com/mchaloupka/tickspec)
  * Property-based testing : [FsCheck](https://fscheck.github.io/FsCheck/)
* Langage
  * [Query expressions](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/query-expressions) : support de LINQ en F#
  * [Code quotation](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/code-quotations)
* Programmation concurrente
  * Mot-clé `lock` → [Exemple](https://www.compositional-it.com/news-blog/testing-for-breaking-changes/)
  * [MailboxProcessor](https://fsharpforfunandprofit.com/posts/concurrency-actor-model/) _(actor-based concurrent programing model)_
* Accès aux données
  * [Guide : data access with F#](https://fsharp.org/guides/data-access/) sur fsharp.org
  * [Type providers](https://docs.microsoft.com/en-us/dotnet/fsharp/tutorials/type-providers/)
  * ORM : EFCore, Dapper, [RepoDb](https://repodb.net/)
* [Développement Web](https://docs.microsoft.com/fr-fr/dotnet/fsharp/scenarios/web-development)
  * [Giraffe](https://github.com/giraffe-fsharp/Giraffe#giraffe) : surcouche à ASP.NET Core
  * [Saturne](https://saturnframework.org/) : framework alternatif à ASP.NET Core
  * [Fable](https://fable.io/) : compilateur F# → JavaScript
  * [SAFE stack](https://safe-stack.github.io/) : stack complète comprenant (entre autres) **S**aturn, **A**zure, **F**able, **E**lmish
* Cloud
  * [Guide sur fsharp.org](https://fsharp.org/guides/cloud/)
  * Infra-as-Code : Azure + [Farmer](https://compositionalit.github.io/farmer/) ([intro](https://docs.microsoft.com/fr-fr/dotnet/fsharp/using-fsharp-on-azure/deploying-and-managing))
* Data Science et Notebook&#x20;
  * [Guide sur fsharp.org](https://fsharp.org/guides/data-science/)

### Autres ressources

* 📩 [F# Weekly](https://sergeytihon.com/fsharp-weekly/) de Serge Tihon
* 🔗 [Awesome F#](https://github.com/fsprojects/awesome-fsharp)
* 🔗 [Microsoft F# Style guide](https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/conventions#use-classes-to-contain-values-that-have-side-effects) : organisation du code, usage adéquate des classes, gestion des erreurs (par types ou exceptions), application partielle et style point-free, encapsulation, inférence de types et génériques, performance
