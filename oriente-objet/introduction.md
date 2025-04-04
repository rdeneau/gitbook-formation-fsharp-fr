---
description: Orienté-objet, polymorphisme
---

# Introduction

En F♯, l'orienté-objet est parfois + pratique que style fonctionnel.

Briques permettant l'orienté-objet en F♯ :

1. Membres
   * Méthodes, propriétés, opérateurs
   * Consistent à attacher des fonctionnalités directement dans le type
   * Permettent d'encapsuler l'état (en particulier mutable) de l'objet
   * S'utilisent avec la notation "pointée" `my-object.my-member`
2. Interfaces et classes
   * Supports de l'abstraction par héritage

## Polymorphisme

4e pilier de l'orienté-objet

En fait, il existe plusieurs polymorphismes. Les principaux :

1. Par sous-typage : celui évoqué classiquement avec l'orienté-objet&#x20;
   * Type de base définissant membres abstraits ou virtuels
   * Sous-types en héritant et implémentant ces membres
2. Ad hoc/overloading → surcharge de membres de même nom
3. Paramétrique → génériques en C♯, Java, TypeScript
4. Structurel/duck-typing → SRTP en F♯, typage structurel en TypeScript
5. Higher-kinded → classes de type en Haskell
