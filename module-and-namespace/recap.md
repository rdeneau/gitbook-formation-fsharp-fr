# 📜 Récap’

Modules et namespaces servent à :

* Regrouper par fonctionnalité
* Scoper : namespaces > fichiers > modules

<table><thead><tr><th width="312">Propriété</th><th width="210.33333333333331">Namespace</th><th>Module</th></tr></thead><tbody><tr><td>Compilation .NET</td><td><code>namespace</code></td><td><code>static class</code></td></tr><tr><td>Type</td><td><em>Top-level</em></td><td>Local (ou <em>top-level</em>)</td></tr><tr><td>Contient</td><td>Modules, Types</td><td>Valeurs, Fonctions, Type,<br>Sous-modules</td></tr><tr><td><code>[&#x3C;RequireQualifiedAccess>]</code></td><td>❌ Non</td><td>✅ Oui <em>(vs shadowing)</em></td></tr><tr><td><code>[&#x3C;AutoOpen>]</code></td><td>❌ Non</td><td>✅ Oui mais prudence❗</td></tr></tbody></table>
