---
description: Les fondamentaux
---

# Syntaxe

## Syntaxe - Clé

1er point fort de F♯ : langage **succinct**

Pour s'en rendre compte :

1. Passons rapidement en revue sa syntaxe
2. Ensuite vous pourrez commencer à jouer avec
   * ☝ C'est à l'usage que l'on mesure le côté succinct de F♯

> 💡 F♯ est généralement plus succinct que C♯ sauf ce qui concerne le fait d'être **explicite**.

## Commentaires

```fsharp
(* This is block
   comment *)


// And this is line comment


/// XML doc summary


/// <summary>
/// Full XML doc
/// </summary>
```

## ~~Variables~~ → Valeurs

* Mot clé `let` pour déclarer/nommer une valeur
* Pas besoin de `;` en fin de déclaration
* Liaison/_Binding_ est immutable par défaut
  * ≃ `const` en JS, `readonly` pour un membre en C♯
* Mutable avec `let mutable` et opérateur d'assignation `<-`
  * ≃ `let` en JS, `var` en C♯
  * Avec parcimonie, sur _scope_ limité

```fsharp
let x = 1
x <- 2 // ❌ Error FS0027: Cette valeur n'est pas mutable.

let mutable x = 1
x <- 2 // ✅ Autorisé
```

### Pièges ⚠️

Ne pas confondre la mutabilité d'une variable (la partie à gauche dans le binding) et la mutabilité d'un objet (la partie à droite du binding).

* `let` empêche le binding d'être modifié c'est-à-dire affecter une nouvelle valeur à une variable.
* `let` n'empêche pas un objet de muter...
* ... à une exception près : les **structures mutables** (exemple : [ArrayCollector](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-core-compilerservices-arraycollector-1.html)) qui fonctionnent par valeur : quand on appelle une méthode qui modifie une structure (ex: `Add` ou `Close`), on obtient une nouvelle copie. C'est `let mutable` qui permet de garder une référence à cette nouvelle valeur.

```fsharp
open FSharp.Core.CompilerServices

let collectorKo = ArrayCollector<int>()
collectorKo.Add(1)
collectorKo.Add(2)
let result = collectorKo.Close()
// val result: int array = [||]

let mutable collector = ArrayCollector<int>()
collector.Add(1)
collector.Add(2)
let result = collector.Close()
// result it: int array = [|1; 2|]
```

## Noms

* Mêmes contraintes qu'en C♯
* Sauf apostrophe `'`  _(tick)_
  * permise dans nom au milieu ou à la fin _(mais pas au début)_
  * en fin de nom → indique une variante _(convention)_
* Entre doubles _backticks_&#x20;
  * permettent n'importe quel caractère, en particulier les espaces, à l'exception des sauts de ligne)

```fsharp
let x = 1
let x' = x + 1  // Se prononce "x prime" ou "x tick"

// Marche aussi avec les mots clés !
// Mais mieux vaut éviter car c'est source de confusion !
let if' b t f = if b then t else f

let ``123 456`` = "123 456"
// 💡 Auto-complétion : pas besoin de taper les ``, directement 123 (quand ça veut marcher)
```

## _Shadowing_

* Consiste à redéfinir une valeur avec un nom existant\
  → La valeur précédente n'est plus accessible dans le scope courant
* Interdit au niveau d'un module mais autorisé dans un sous-scope
* Pratique mais peut être trompeur\
  → Peu recommandé, sauf cas particulier

```fsharp
let a = 2

let a = "ko"  // 💥 Error FS0037: Définition dupliquée de value 'a'

let b =
    let a = "ok" // 👌 Pas d'erreur de compilation
    // `a` ne vaut plus 2 mais "ok" dans tout le reste de l'expression `b`
    let a = "ko" // 👌 Plusieurs shadowings successifs sont possible !
    ...
```

## Annotation de type

* Optionnelle grâce à l'inférence
* Type déclaré après nom `name: type` _(comme en TypeScript)_
* Valeur obligatoire, même si `mutable`  👍

```fsharp
let x = 1       // Inféré (int)
let y: int = 2  // Explicite

let z1: int           // 💥 Error FS0010: Construction structurée incomplète à cet emplacement...
let mutable z2: int   // 💥 ... ou avant dans la liaison. '=' ou autre jeton attendu.
```

## Constante

* _What:_ Variable effacée à la compilation, remplacée par sa valeur
  * ≃ `const` C♯ - même idée en TS que `const enum`
* _How:_ Valeur décorée avec attribut `Literal` \
  ⚠️ Attributs entre `[< >]` \
  → Erreur fréquente au début : utiliser `[ ]` _(comme en C♯)_
* Convention de nommage recommandée : PascalCase

```fsharp
[<Literal>] // Saut de ligne nécessaire car avant le `let`
let AgeOfMajority = 18

let [<Literal>] Pi = 3.14 // Possible aussi après le `let`
```

## Nombre

```fsharp
let pi = 3.14             // val pi : float = 3.14       • System.Double
let age = 18              // val age : int = 18          • System.Int32
let price = 5.95m         // val price : decimal = 5.95M • System.Decimal
```

⚠️ Pas de conversion implicite entre nombre

→ 💡 Utiliser fonctions `int`, `float`, `decimal`

<pre class="language-fsharp"><code class="lang-fsharp"><strong>let i = 1
</strong><strong>i * 1.2;;  // 💣 error FS0001: The type 'float' does not match the type 'int'
</strong><strong>
</strong><strong>float 3;;             // val it : float = 3.0
</strong>decimal 3;;           // val it : decimal = 3M
int 3.6;;             // val it : int = 3
int "2";;             // val it : int = 2
</code></pre>

☝️ Cette règle a été assouplie dans certains cas avec [F# 6](https://learn.microsoft.com/en-us/dotnet/fsharp/whats-new/fsharp-6#additional-implicit-conversions).

## String

```fsharp
let name = "Bob"              // val name : string = "Bob"

// Chaîne avec interpolation
let name2 = $"{name} Marley"  // val name' : string = "Bob Marley"
let name2 = sprintf "{0} Marley" name // Equivalent avant F♯ 5

// Accès à un caractère par son index (>= 0)
let initial = name2[0]         // val initial :  char = 'B'
let initial = name2.[0]        // ☝️ Syntaxe avant F♯ 6

// Accès à une sous-chaîne via une plage (EN: range)
// 💡 Alternative à la méthode Substring(index [, length])
let firstName = name2[0..2]   // val firstName : string = "Bob"
let firstName = name2.[0..2]  // ☝️ Syntaxe avant F♯ 6
```

Syntaxes complémentaires :

```fsharp
// Verbatim string: idem C♯
let verbatimXml = @"<book title=""Paradise Lost"">"

// Triple-quoted string : pas besoin d'échapper les guillemets `"`
let tripleXml = """<book title="Paradise Lost">"""

// Les chaînes normales acceptent les sauts de ligne mais ne trimment pas les whitespaces
let poemIndented = "
    The lesser world was daubed
    By a colorist of modest skill
    A master limned you in the finest inks
    And with a fresh-cut quill."

// Solution : les backslash strings :
// Les whitespaces (espace et saut de ligne) sont ignorés 
// entre le \ terminant une ligne et le caractère non-whitespace suivant
// (d'où les \n pour remettre des sauts de ligne) :
let poem = "\
    The lesser world was daubed\n\
    By a colorist of modest skill\n\
    A master limned you in the finest inks\n\
    And with a fresh-cut quill."

// On peut aussi combiner sauts de ligne et backslash strings :
let poemWithoutBackslashN = "\
    The lesser world was daubed
\
    By a colorist of modest skill
\
    A master limned you in the finest inks
\
    And with a fresh-cut quill."
```

### Chaîne avec interpolation en F# 8

Une chaîne interpolée ne peut pas contenir d'accolades (`$"{xxx}"`) sauf à les doubler (`$"{{xxx}}"`) . Depuis F# 8, c'est le caractère $ que l'on double (`$$`) pour indiquer le nombre d'accolades à partir duquel il y a interpolation.

```fsharp
let classAttr = "bold"
let cssNew = $$""".{{classAttr}}:hover {background-color: #eee;}"""
```

### Encodage

Les littéraux de chaînes (EN: _string literals_) sont encodés en Unicode.&#x20;

```fsharp
let unicodeString1 = "abc"  // val unicodeString1: string = "abc"
let unicodeString2 = "ab✅" // val unicodeString2: string = "ab✅"
```

On peut travailler en ASCII grâce au suffixe `B` mais on manipule alors un tableau de bytes :

```fsharp
let asciiBytes1 = "abc"B
// val asciiBytes1: byte array = [|97uy; 98uy; 99uy|]

let asciiBytes2 = "ab✅"B
// 💥 Error FS1140: This byte array literal contains characters
//    that do not encode as a single byte
```

## Listes

Liste immuable → type spécial F♯ ≠ `System.Collection.Generic.List<T>`

```fsharp
let abc = [ 'a'; 'b'; 'c' ] // val abc : char list = ['a'; 'b'; 'c']
let a =
  [ 2
    3 ]  // val a : int list = [2; 3]
```

* Création avec `[]`, éléments séparés par `;` ou saut de ligne + indentation
  * :warning: Piège : ne pas utiliser `,` sinon on a en fait un seul élément, un tuple ! 📍[tuples.md](../types-composites/tuples.md "mention")
* Notation ML du type `int list` = `List<int>`
  * ☝ Idiomatique que pour `array`, `list` et `option` 📍[type-option.md](../types-monadiques/type-option.md "mention")

### Opérateurs sur les listes

* `::`  _Cons_ _(pour "construction")_\
  → Ajoute un element en tête de liste
* `@` _Append_\
  → Concatène 2 listes
* `..` _Range_\
  → `min..max`: plage de nombres ou de caractères entre le min et le max spécifiés _(inclus)_\
  → `min..step..max`: `step` définit l'écart entre 2 nombres successifs dans la plage

```fsharp
let nums = [2..5]                 // val nums : int list = [2; 3; 4; 5]
let nums' = 1 :: nums             // val nums' : int list = [1; 2; 3; 4; 5]

let chars = [ 'a' .. 'd' ]        // val chars : char list = ['a'; 'b'; 'c'; 'd']
let chars' = chars @ [ 'e'; 'f' ] // val chars' : char list = ['a'; 'b'; 'c'; 'd'; 'e'; 'f']
let e = chars'[4]                 // val e: char = 'e'
```

⚠️ Il faut mettre un espace avant `[]` pour créer une liste, pour différencier de l'accès par index.

### Module `List`

Ce module contient les fonctions pour manipuler une ou plusieurs listes.

| F♯ List            | C♯ LINQ (IEnumerable)      | JS Array             |
| ------------------ | -------------------------- | -------------------- |
| `map`, `collect`   | `Select()`, `SelectMany()` | `map()`, `flatMap()` |
| `exists`, `forall` | `Any(predicate)`, `All()`  | `some()`, `every()`  |
| `filter`           | `Where()`                  | `filter()`           |
| `find`, `tryFind`  | ×                          | `find()`             |
| `fold`, `reduce`   | `Aggregate([seed]])`       | `reduce()`           |
| `average`, `sum`   | `Average()`, `Sum()`       | ×                    |

☝ **Autres fonctions :** cf. [documentation](syntaxe.md#syntaxe-cle)

## Fonctions

* Fonction nommée : déclarée avec `let`
* Convention de nommage : **camelCase**
* Pas de `return` : la fonction renvoie toujours la dernière expression dans son corps
* Pas de `()` autour de tous les paramètres
  * `()` autour d'un paramètre avec annotation de type (1) ou déconstruit tel qu'un tuple (2)

```fsharp
let square x = x * x  // Fonction à 1 paramètre
let res = square 2    // Vaut 4

// (1) Parenthèses nécessaires lors d'annotations de type
let square' (x: int) : int = x * x

// (2) Parenthèses nécessaires lors de déconstruction d'un paramètre
let addTuple (x, y) = x + y
```

### Fonctions de 0-n paramètres

* Paramètres et arguments séparés par **espace**
  * :warning:️ Piège : `,` sert à instancier 1 tuple = 1! paramètre
* `()` : fonction sans paramètre, sans argument
* Sans `()`, on déclare une valeur "vide", pas une fonction :

```fsharp
let add x y = x + y  // Fonction à 2 paramètres
let res = add 1 2    // `res` vaut 3

let printHello () = printfn "Hello"  // Fonction sans paramètre
printHello ()                        // Affiche "Hello" en console

let notAFunction = printfn "Hello"   // Affiche "Hello" en console et renvoie "vide"
```

### Fonction multi-ligne

**Indentation** nécessaire, mais pas de `{}`

```fsharp
let evens list =
    let isEven x =  // 👈 Sous-fonction
        x % 2 = 0   // 💡 `=` opérateur d'égalité - Pas de `==`
    List.filter isEven list

let res = evens [1;2;3;4;5] // Vaut [2;4]
```

### Fonction anonyme

A.k.a. **Lambda**, arrow function

* Déclarée avec `fun` et `->`
* En général entre `()` pour question de précédence

```fsharp
let evens' list = List.filter (fun x -> x % 2 = 0) list
```

☝ **Note** : taille de la flèche

* Fine `->` en F♯, Java
* Large / _fat_ `=>` en C♯, JS

## Convention de noms courts

* `x`, `y`, `z` : paramètres de type valeurs simples
* `f`, `g`, `h` : paramètres de type fonction
* `xs` : liste de `x` → `x::xs` (ou `h::t`) = _head_ et _tail_ d'une liste non vide
* `_` : _discard_ / élément ignoré car non utilisé _(comme en C♯ 7.0)_

Bien adapté quand fonction courte ou très générique :

```fsharp
// Fonction qui renvoie son paramètre d'entrée, quel qu'il soit
let id x = x

// Composition de 2 fonctions
let compose f g = fun x -> g (f x)
```

## _Piping_

Opérateur _pipe_ `|>` : même idée que `|` UNIX \
→ Envoyer la valeur à gauche dans une fonction à droite \
→ Ordre naturel "sujet verbe" - idem appel méthode d'un objet

```fsharp
let a = 2 |> add 3  // Se lit comme "2 + 3"

let nums = [1;2;3;4;5]
let evens = nums |> List.filter (fun x -> x % 2 = 0)
// Idem     List.filter (fun x -> x % 2 = 0) nums
```

```csharp
// ≃ C♯
var a = 2.Add(3);
var nums = new[] { 1, 2, 3, 4, 5 };
var evens = nums.Where(x => x % 2 == 0);
```

### Chainage de _pipes_ - _Pipeline_

Comme fluent API en C♯ mais natif : pas besoin de méthode d'extension 👍

Façon naturelle de représenter le flux de données entre différentes opérations → Sans variable intermédiaire 👍

Écriture :

```fsharp
// Sur une seule ligne (courte)
let res = [1;2;3;4;5] |> List.filter (fun x -> x % 2 = 0) |> List.sum

// Sur plusieurs lignes => fait ressortir les différentes opérations
let res' =
    [1; 2; 3; 4; 5]
    |> List.filter isOdd  // Avec `let isOdd x = x % 2 <> 0`
    |> List.map square    //      `let square x = x * x`
    |> List.map addOne    //      `let addOne x = x + 1`
```

## Expression `if/then/else`

💡 `if b then x else y` ≃ Opérateur ternaire C♯ `b ? x : y`

```fsharp
let isEven n =
    if n % 2 = 0 then
        "Even"
    else
        "Odd"
```

☝ Si `then` ne renvoie pas de valeur, `else` facultatif

```fsharp
let printIfEven n msg =
    if n |> isEven then
        printfn msg
```

## Expression `match`

```fsharp
let translate civility =
    match civility with
    | "Mister" -> "Monsieur"
    | "Madam"  -> "Madame"
    | "Miss"   -> "Mademoiselle"
    | _        -> ""   // 👈 wilcard `_`
```

Équivalent en C♯ 8 :

```csharp
public static string Translate(string civility) =>
    civility switch {
        "Mister" => "Monsieur"
        "Madam"  => "Madame"
        "Miss"   => "Mademoiselle"
        _        => ""
    }
```

## Exception

### Handling Exception

→ Bloc `try/with` -- :warning: Piège : ~~try/catch~~

```fsharp
let tryDivide x y =
   try
       Some (x / y)
   with :? System.DivideByZeroException ->
       None
```

💡  Il n'y a pas de `try/with/finally` mais on peut faire imbriquer un `try/finally` dans un `try/with`.

### Throwing Exception

→ Helpers `failwith`, `invalidArg` et `nullArg`

```fsharp
let fn arg =
    if arg = null then nullArg (nameof arg)
    failwith "Not implemented"

let divide x y =
    if y = 0
    then invalidArg (nameof y) "Divisor cannot be zero"
    else x / y
```

☝ Pour erreurs métier i.e. cas prévus, non exceptionnels : \
→ Préférer le type `Result` a.k.a _Railway-oriented programming_ 📍 [Broken link](broken-reference "mention")

🔗 Handling Errors Elegantly [https://devonburriss.me/how-to-fsharp-pt-8/](https://devonburriss.me/how-to-fsharp-pt-8/)

## Attributs

Les attributs fonctionnent comme en C# pour décorer une classe, une méthode, etc.

La subtilité vient de la syntaxe :&#x20;

* C# `[AttributUn][DeuxiemeAttribut]` ou `[AttributUn, DeuxiemeAttribut]`\
  → Attributs entre `[...]` et séparés par des `,`
* F# `[<AttributUn>][<DeuxiemeAttribut>]` ou `[<AttributUn>; <DeuxiemeAttribut>]` \
  → Attributs entre `[<...>]` et séparés par des `;`

## Ordre des déclarations

:warning: Déclarations ordonnées de haut en bas

* Déclaration précède usage
* Au sein d'un fichier
* Entre fichiers dépendants
  * ☝ Importance de l'ordre des fichiers dans un `.fsproj`
* Bénéfice : pas de dépendances cycliques 👍

```fsharp
let result = fn 2
//           ~~ 💥 Error FS0039: La valeur ou le constructeur 'fn' n'est pas défini

let fn i = i + 1 // ☝ Doit être déclarée avant `result`
```

## Indentation

* Très importante pour lisibilité du code
  * Crée struct. visuelle qui reflète struct. logique / hiérarchie
  * `{}` alignées verticalement (C♯) = aide visuelle mais < indentation
* Essentielle en F♯ :
  * Façon de définir des blocs de code
  * Compilateur assure que indentation est correcte

👉 **Conclusion :**

* F♯ force à bien indenter
* Mais c'est pour notre bien
* Car c'est bénéfique pour lisibilité du code 👍

### Ligne verticale d'indentation

→ Démarre après `let ... =`, `(`, `then`/`else`, `try`/`finally`, `do`, `->` (dans clause `match`) mais pas `fun` ! → Commence au 1er caractère non _whitespace_ qui suit → Tout le reste du bloc doit s'aligner verticalement → L'indentation peut varier d'un bloc à l'autre

```
let f =
  let x=1     // ligne d'indentation fixée en column 3 (indépendamment des lignes précédentes)
  x+1         // 👉 cette ligne (du même bloc) doit commencer en column 3 ; ni 2, ni 4 ❗

let f = let x=1  // Indentation en column 10
        x+1      // 👉 alignement vertical des autres lignes du bloc en column 10
```

🔗 [https://fsharpforfunandprofit.com/posts/fsharp-syntax/](https://fsharpforfunandprofit.com/posts/fsharp-syntax/)

### Recommendations / Indentation

* Utiliser des **espaces**, pas des ~~tabulations~~
* Utiliser **4 espaces** par indentation
  * Facilite la détection visuelle des blocs
  * ... qui ne peut se baser sur les `{ }` comme en C♯
* Éviter un **alignement sensible au nom**, a.k.a _vanity alignment_
  * Risque de rupture de l'alignement après renommage → 💥 Compilation
  * Bloc trop décalé à droite → nuit à la lisibilité

```fsharp
// 👌 OK
let myLongValueName =
    someExpression
    |> anotherExpression

// ⚠️ À éviter
let myLongValueName = someExpression
                      |> anotherExpression  // 👈 Dépend de la longueur de `myLongValueName`
```

&#x20;☝️ Les règles concernant l'indentation ont été assouplies en [F# 6](https://learn.microsoft.com/en-us/dotnet/fsharp/whats-new/fsharp-6#indentation-syntax-revisions).

## Complément

:link: [Troubleshooting F# - Why won't my code compile?](https://fsharpforfunandprofit.com/troubleshooting-fsharp/)
