# Vue d'ensemble

## Similarités

Modules et namespaces permettent de :

* Organiser le code en zones de fonctionnalités connexes
* Éviter collision de noms

## Différences

| Propriété        | Namespace      | Module                    |
| ---------------- | -------------- | ------------------------- |
| Compilation .NET | `namespace`    | `static class`            |
| Type             | _Top-level_    | _Top-level_ ou local      |
| Contient         | Modules, Types | Idem + Valeurs, Fonctions |
| Annotable        | ❌ Non          | ✅ Oui                     |

**Portée :** Namespaces > Fichiers > Modules

## Utiliser un module ou un namespace

💡 Comme en C♯ :

1. Soit qualifier chaque élément du module/namespace utilisé
2. Soit tout importer avec `open` _(placé en haut du fichier ou juste avant)_
   * En C♯ ≡ `using` pour un namespace
   * En C♯ ≡ `using static` pour un module F♯ _(équivalent d'une classe statique .NET)_

```fsharp
// Option 1. Qualifier les usages
let result1 = Arithmetic.add 5 9

// Option 2. Importer tout le module
open Arithmetic
let result2 = add 5 9
```

:warning: **Attention :** l'ordre des imports est pris en compte, de haut en bas.\
→ Cela autorise le _shadowing_ (cf. ci-dessous).\
→ De ce fait, réordonner des imports dans un fichier existant peut faire échouer la compilation !\
→ Donc, même s'il est conseillé d'avoir les imports triés par ordre alphabétique, cela n'est pas toujours possible. Alors, il est conseillé de l'indiquer dans le code au moyen d'un commentaire.

☝️ Depuis F♯ 5, il existe le mot clé `open type` qui permet d'importer les éléments statiques d'un type.\
→ Ne pas le confondre avec `open` appliqué à un module, même s'il y a une équivalence au niveau .NET.\
🔗Infos complémentaires : [Article du blog de compositional-IT](https://www.compositional-it.com/news-blog/open-type-declarations-in-fsharp-5/), [MSDN](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/import-declarations-the-open-keyword#open-type-declarations)

## Import : _shadowing_

L'import se fait sans conflit de nom mais en mode le dernier importé gagne i.e. masque un précédent élément importé de même nom

:warning: Peut créer des problèmes difficiles à comprendre ❗

Exemple : ci-dessous, on a une erreur de compilation car la fonction `add` qui est appelée est celle du module `FloatHelper` !

```fsharp
module IntHelper =
    let add x y = x + y

module FloatHelper =
    let add x y : float = x + y

open IntHelper
open FloatHelper // 👈 Shadowing de la fonction add de IntHelper

let result = add 1 2 // 💥 Error FS0001: Le type 'float' ne correspond pas au type 'int'
```

> ☝ **Conseil :** pour tous les éléments qui ont des noms communs, propices au _shadowing_ \
> ‒ tels que `add` dans l'exemple précédent ou `map`, `filter`… des modules `List`, `Seq`… \
> ‒ il est conseillé de les appeler en les qualifiant, i.e. sans faire d'import de leur module.
>
> 💡 Peut être rendu obligatoire en décorant le module avec `[<RequireQualifiedAccess>]`.
>
> ## Mot clé `global`
>
> Du fait du _shadowing_, on peut tomber sur des cas où l'on ne peut importer le bon namespace. Par exemple, ayant fait `open FsCheck` et ayant référencé la librairie `FsCheck.Xunit`, lorsque l'on fait `open Xunit`, on importe non pas le namespace `Xunit` de la librairie `Xunit` mais le namespace `FsCheck.Xunit`!
>
> 💡 On se sort de cet imbroglio en faisant `open global.Xunit`.
