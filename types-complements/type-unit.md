# Type unit

## Pourquoi ?

> **Contrainte :** l'évaluation d'une expression doit produire une valeur.

Quid des expressions ne produisant pas de valeur significative ?

* `void` ? Non, `void` en C♯, Java n'est pas une valeur !
* `null` ? Non, `null` n'est pas un type en .NET ! _(≠ TypeScript)_

Il faut donc un type spécifique, avec une seule valeur signifiant par convention : « Valeur non significative, à ignorer. »

* Ce type s'appelle `unit`.
* Sa valeur est notée `()`.Lien avec les fonctions

Fonction `unit -> 'T` ne prend pas de paramètre. → Ex : `System.DateTime.Now` _(fonction cachée derrière une propriété)_

Fonction `'T -> unit` ne renvoie pas de valeur. → Ex : `printf`

👉 Fonctions impliquant un **effet de bord** !

## Ignorer une valeur

F♯ n'est pas une langage fonctionnel pur, sans effet de bord. Mais il encourage l'écriture de programme fonctionnel pur.

👉 **Règle :** Toute expression produisant une valeur doit être utilisée.

* Sinon, le compilateur émet un warning _(sauf en console FSI)._
* Exception: `()` est la seule valeur que le compilateur autorise à ignorer.

☝ **Avertissement :** ignorer une valeur est généralement un _code smell_ en FP.

👉 Une expression avec effet de bord doit le signaler avec type de retour `unit`

## Fonction `ignore`

> ❓ Comment _(malgré tout)_ ignorer la valeur produit par une expression ?

Avec la fonction `ignore` :

* Prend un paramètre d'entrée ignoré, "avalé"
* Renvoie `unit`

```fsharp
let inline ignore _ = ()
// Signature: 'T -> unit
```

Usage : `expression |> ignore`
