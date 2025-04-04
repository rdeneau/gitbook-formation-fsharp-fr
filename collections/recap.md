# 📜 Récap’

## Types

5 collections dont 4 fonctionnelles/immutables

* `List` : choix par défaut
  * _Passe-partout_
  * _Pratique_ : pattern matching, opérateurs _Cons_ `::` et _Append_ `@`...
* `Array` : mutabilité / performance
* `Seq` : évaluation différée (_Lazy_), séquence infinie
* `Set` : unicité des éléments
* `Map` : classement des éléments par clé

## API

**Riche** \
→ Centaine de fonctions >> Cinquantaine pour LINQ

**Homogène** \
→ Conservation du type de la collection (≠ avec LINQ, bascule sur `IEnumerable<>`)\
→ Syntaxe et fonctions communes entre les types de collection

**Sémantique** \
→ Nom des fonctions proche du JS (cf. tableau ci-dessous)

### Comparatif C♯, F♯, JavaScript

| C♯ LINQ                       | F♯                    | JS `Array`           |
| ----------------------------- | --------------------- | -------------------- |
| `Select()`, `SelectMany()`    | `map`, `collect`      | `map()`, `flatMap()` |
| `Any(predicate)`, `All()`     | `exists`, `forall`    | `some()`, `every()`  |
| `Where()`, ×                  | `filter`, `choose`    | `filter()`, ×        |
| `First()`, `FirstOrDefault()` | `find`, `tryFind`     | ×, `find()`          |
| ×                             | `pick`, `tryPick`     | ×                    |
| `Aggregate([seed]])`          | `fold`, `reduce`      | `reduce()`           |
| `Average()`, `Sum()`          | `average`, `sum`      | ×                    |
| `ToList()`, `AsEnumerable()`  | `List.ofSeq`, `toSeq` | ×                    |
| `Zip()`                       | `zip`                 | ×                    |

## Exercices

Sur [exercism.io](https://exercism.io/tracks/fsharp), voici quelques exercices sur les collections :&#x20;

| Exercice            | Niveau | Sujets                 |
| ------------------- | ------ | ---------------------- |
| High Scores         | Facile | `List`                 |
| Protein Translation | Moyen+ | `Seq`/`List` 💡        |
| ETL                 | Moyen  | `Map` de `List`, Tuple |
| Grade School        | Moyen+ | `Map` de `List`        |

☝ Pré-requis : \
→ Se créer un compte, avec GitHub par exemple\
→ Résoudre les 1ers exercices pour arriver à ceux-là

## Ressources complémentaires

* [Toutes les fonctions, avec leur coût en O(?)](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/fsharp-collection-types#table-of-functions)
* [Choosing between collection functions](https://fsharpforfunandprofit.com/posts/list-module-functions/), F# for fun and profit, Août 2015
* [An F# Primer for curious C# developers - Work with collections](https://laenas.github.io/posts/01-fs-primer.html#work-with-collections), 2020
* [Formatage des collections](https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/formatting#formatting-lists-and-arrays)
