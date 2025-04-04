# Namespace

## Syntaxe

Syntaxe : `namespace [rec] [parent.]identifier`

* `rec` pour récursif _(slide suivante)_
* `parent` permet de regrouper des namespaces
* Tout ce qui suit appartiendra à ce namespace

## Contenu

Un `namespace` F♯ ne peut contenir que des types et des modules locaux

* Ne peut contenir ni valeurs ni fonctions

Par équivalence avec la compilation .NET

* idem `namespace` C♯ qui ne peut contenir que des classes / enums

Quid des namespaces imbriqués ?

* Se passe uniquement de manière déclarative `namespace [parent.]identifier`
* 2 namespaces déclarés à la suite = ~~pas imbriqués~~ mais indépendants

## Portée

* Plusieurs fichiers peuvent partager le même namespace
* Dans un fichier, on peut déclarer plusieurs namespaces\
  → Ils ne seront pas imbriqués\
  → Peut être source de confusion ❗

## **Recommandations**

* Utiliser les namespaces dans un projet `.fsproj`\
  &#xNAN;_(Dans un script `.fsx`, cela a moins d'intérêt)_
* Ne déclarer qu'**1 seul** namespace par fichier
* Organiser les fichiers en les groupant dans des répertoires de même nom que le namespace

<table><thead><tr><th width="163.33333333333331">Niveau</th><th>Répertoire</th><th>Namespace</th></tr></thead><tbody><tr><td>0 <em>(racine)</em></td><td><code>/</code></td><td><code>*</code></td></tr><tr><td>1</td><td><code>/Niv1</code></td><td><code>*.Niv1</code></td></tr><tr><td>2</td><td><code>/Niv1/Niv2</code></td><td><code>*.Niv1.Niv2</code></td></tr></tbody></table>

## 🚀 Namespace récursif

Permet d'étendre la visibilité par défaut unidirectionnelle, de bas en haut, pour que des éléments les uns au-dessous des autres se voient mutuellement

```fsharp
namespace rec Fruit

type Banana = { Peeled: bool } with
    member this.Peel() =
        BananaHelper.peel this // 👈 `peel` non visible ici sans le `rec`

module BananaHelper =
    let peel banana = { banana with Peeled = true }
```

:warning:️ **Inconvénients :** compilation + lente et risque de référence circulaire ❗

☝ **Recommandation :** pratique mais à utiliser avec parcimonie
