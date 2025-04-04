---
description: Formation F# 5-9, en français 🇫🇷 🔵⚪🔴
---

# Intro

## Auteur

Romain DENEAU

* Senior Developer F♯ C♯ TypeScript
* Github [/rdeneau](https://github.com/rdeneau)
* Linked-in [romain-deneau](https://www.linkedin.com/in/romain-deneau-95481143/)
* ~~Twitter/X~~ [~~@DeneauRomain~~](https://x.com/DeneauRomain)

## Pourquoi F♯

[En un tweet](https://nitter.net/MokoSharma/status/1458151277343379457) :

* Succinct
* Robuste
* Performant

En détail :

* Langage multi-paradigme avec une forte orientation fonctionnelle
  * Principes fonctionnels : immutabilité et composition
  * Briques : fonctions et types algébriques
* "Fun" ! Très agréable à écrire et à lire
  * Expressif et concis (syntaxe peu verbeuse)
  * Sensible à l'indentation → facilite la lecture
  * Typage statique fort mais quasi-transparent grâce à l'inférence de type
* Langage entreprise friendly
  * Tourne sur la plateforme .NET → performant, Interop aisée avec projets C#
  * Très bon tooling : VS, VsCode, Rider
  * Code robuste : résultats prédictibles et reproductibles (immuabilité, égalité structurelle, absence de null, vérification exhaustive des cas dans _pattern matching_)
  * Communauté solide, nombreuses librairies _F# friendly_
  * Programmation interactive : vérifier un code en l'évaluant dans la console FSI
* F# par rapport aux autres langages fonctionnels "Back-end"
  * Sa syntaxe n'est pas hybride contrairement à Scala et Kotlin (mixe OOP/FP)
  * Plus facile à apprendre que Haskell ou OCaml _(mais qui ont plus de fonctionnalités FP)_
  * Typage statique contrairement à Closure _(et beaucoup beaucoup moins de parenthèses)_

## Convention

📍 → notion abordée plus tard, généralement avec le lien associé

💥 → erreur de compilation ou une exception au _runtime_&#x20;

🚀 → chapitre d'un niveau + avancé

🍔 ou 🎲 → Quiz

📜 → Récap’

## Public cible

* Développeur C#
* Pour les développeurs non .NET, en voici une [courte introduction](https://carpenoctem.dev/blog/fsharp-for-linux-people/#versions).

## Installation

### SDK .NET <a href="#install-net-sdk" id="install-net-sdk"></a>

Kit de développement logiciel :

* Contient : CLI `dotnet`, librairies et runtime .NET
* Gratuit et multiplateforme

Procédure :&#x20;

* Télécharger et installer le SDK .NET 5.0 : [https://dotnet.microsoft.com/download/dotnet/5.0](https://dotnet.microsoft.com/download/dotnet/5.0)
* Vérifier l'installation en ouvrant un terminal et en entrant la commande `dotnet --version`qui renverra par exemple `5.0.302`.

### Visual Studio Code

Éditeur de texte gratuit, open source et multiplateforme

* Télécharger et installer vscode : [https://code.visualstudio.com/#alt-downloads](https://code.visualstudio.com/#alt-downloads)
* Vérifier l'installation en ouvrant un terminal et en entrant la commande `code .`qui doit ouvrir vscode et parcourir le répertoire courant.
* Installer l'extension [Ionide-fsharp](https://marketplace.visualstudio.com/items?itemName=Ionide.Ionide-fsharp) pour faire de vscode un IDE complet pour F#

### Linux

🔗 [Compléments pour développer du F# sur Linux](https://carpenoctem.dev/blog/fsharp-for-linux-people/)

## Changelog

### 2025-04-02

* F# 8.0 et F# 9.0

### 2024-01-24

* Mention de [#lazy-active-pattern](fonctions/fonctions-complements.md#lazy-active-pattern "mention")
* Astuce pour instancier un `Nullable` (cf. [#option-vs-nullable](types-monadiques/type-option.md#option-vs-nullable "mention"))
* Mention de `ResizeArray` (cf. [vue-densemble.md](collections/vue-densemble.md "mention"))
* Mention de `readOnlyDict` et `KeyValue` (cf. [#dictionnaires](collections/types.md#dictionnaires "mention"))
* Mention du mot clé `global` pour les imports (cf. [#import-shadowing](module-and-namespace/vue-densemble.md#import-shadowing "mention"))
* Précisions sur les attributs `AutoOpen` et `RequireQualifiedAccess`\
  &#xNAN;_(cf._ [#annotation-dun-module](module-and-namespace/module.md#annotation-dun-module "mention")_)_
* Précisions et exemples pour choisir entre méthodes et fonctions \
  &#xNAN;_(cf._ [#methode-vs-fonction](oriente-objet/membres.md#methode-vs-fonction "mention")_)_
* Précisions sur le gain en lisibilité quand on utilise des actives patterns\
  &#xNAN;_(cf._ [#active-pattern-partiel](pattern-matching/active-patterns.md#active-pattern-partiel "mention")_)_

### 2022-11-10

* F♯ 7.0 : RFC [FS-1083](https://github.com/fsharp/fslang-design/blob/main/FSharp-7.0/FS-1083-srtp-type-no-whitespace.md) ([#srtp](types-complements/generiques.md#srtp "mention")),[ ](https://github.com/fsharp/fslang-design/blob/main/FSharp-7.0/FS-1083-srtp-type-no-whitespace.md)[FS-1123 ](https://github.com/fsharp/fslang-design/blob/main/FSharp-7.0/FS-1123-result-module-parity-with-option.md)([#module-result](types-monadiques/type-result.md#module-result "mention")), [FS-1126](https://github.com/fsharp/fslang-design/blob/main/FSharp-7.0/FS-1126-allow-lower-case-du-cases%20when-require-qualified-access-is%20specified.md) ([#casse-des-tags](types-composites/unions.md#casse-des-tags "mention"))
