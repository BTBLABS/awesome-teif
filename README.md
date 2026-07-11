# awesome-teif

> Ressources techniques pour l'écosystème de la facturation électronique tunisienne (**TEIF** — Tunisian Electronic Invoice Format).

[![Licence: CC0-1.0](https://img.shields.io/badge/Licence-CC0--1.0-lightgrey.svg)](LICENSE)
![Statut](https://img.shields.io/badge/statut-en%20construction-orange)

Une liste organisée de documentation officielle, normes, schémas, outils et ressources pour tout développeur, architecte SI ou intégrateur qui travaille avec le format TEIF et le système national **El Fatoora** opéré par **Tunisie TradeNet (TTN)**.

## Sommaire

- [Documentation officielle](#documentation-officielle)
- [Schémas XSD](#schémas-xsd)
- [Normes et spécifications de signature électronique](#normes-et-spécifications-de-signature-électronique)
- [⚠️ Mises à jour critiques et pièges fréquents](#️-mises-à-jour-critiques-et-pièges-fréquents)
- [Autorité de certification](#autorité-de-certification)
- [Bibliothèques et outils](#bibliothèques-et-outils)
- [Ce que ce dépôt n'est pas](#ce-que-ce-dépôt-nest-pas)
- [Contribuer](#contribuer)
- [Licence](#licence)

## Documentation officielle

| Ressource | Source | Description |
|---|---|---|
| [Elfatoora User Guide (PDF)](https://www.tradenet.com.tn/upload/ElFatoora/Elfatoora_User_Guide.pdf) | Tunisie TradeNet | Guide utilisateur officiel de la plateforme El Fatoora |
| [Spécifications techniques Elfatoora (ZIP)](https://www.tradenet.com.tn/upload/ElFatoora/Specifications_Techniques_Elfatoora.zip) | Tunisie TradeNet | Spécifications techniques du format TEIF |
| [Spécifications techniques Signature Fournisseur V3.0 (PDF)](https://www.tradenet.com.tn/upload/ElFatoora/Specifications_Techniques_Signature_Fournisseur_V3.0.pdf) | Tunisie TradeNet | Spécifications de la signature électronique côté fournisseur |
| [Dossier d'abonnement (ZIP)](https://www.tradenet.com.tn/upload/ElFatoora/DOSSIER_D'ABONNEMENT.zip) | Tunisie TradeNet | Dossier administratif d'adhésion au système |
| [tradenet.com.tn/elfatoora.html](https://www.tradenet.com.tn/elfatoora.html) | Tunisie TradeNet | Page officielle El Fatoora — point d'entrée pour toute la documentation TTN |
| [adhesion.elfatoora.tn](https://adhesion.elfatoora.tn/home) | Tunisie TradeNet | Portail d'adhésion en ligne (mode Web ou EDI, authentification E-Houwiya) |

> ⚠️ Ces liens pointent vers le site officiel de TTN — vérifiez leur disponibilité, le contenu peut évoluer sans préavis de leur côté.

## Schémas XSD

TTN ne conserve pas d'archive publique des anciennes versions du schéma. Ce dépôt en garde une copie versionnée :

- [`schemas/elfatoora-1.8.9.xsd`](schemas/elfatoora-1.8.9.xsd) — version **1.8.9**, en **production** (à ce jour, juillet 2026)
- [`schemas/elfatoora-1.9.0-beta.xsd`](schemas/elfatoora-1.9.0-beta.xsd) — version **1.9.0**, en **bêta/test**, pas encore en production
- [Comparaison détaillée 1.8.9 vs 1.9.0](docs/xsd-diff-1.8.9-vs-1.9.0.md) — changements structurels, règles métier ajoutées, contrôle de clé sur la matricule fiscale

Voir [`schemas/README.md`](schemas/README.md) pour la provenance et les conditions de réutilisation de ces fichiers.

## Normes et spécifications de signature électronique

La signature des factures TEIF s'appuie sur les normes ETSI de signature électronique avancée :

| Norme | Objet |
|---|---|
| [ETSI EN 319 132](https://www.etsi.org/deliver/etsi_en/319100_319199/31913201/) | XAdES — signatures XML avancées (format utilisé pour la facturation électronique) |
| [ETSI EN 319 122](https://www.etsi.org/deliver/etsi_en/319100_319199/31912201/) | CAdES — signatures CMS avancées |
| [ETSI EN 319 411-2](https://www.etsi.org/deliver/etsi_en/319400_319499/31941102/) | Exigences de politique pour les autorités de certification délivrant des certificats qualifiés — norme sous laquelle TunTrust est certifiée |
| [ETSI EN 319 421](https://www.etsi.org/deliver/etsi_en/319400_319499/319421/) | Politique et exigences de sécurité pour les autorités d'horodatage — également citée par TunTrust |
| [XML-DSig (W3C/IETF)](https://www.w3.org/TR/xmldsig-core/) | Spécification de base de la signature XML sur laquelle XAdES s'appuie |
| [RFC 5280](https://www.rfc-editor.org/rfc/rfc5280) | Profil de certificat X.509 et de liste de révocation (CRL) |
| [RFC 2560 / RFC 6960](https://www.rfc-editor.org/rfc/rfc6960) | OCSP — vérification en ligne du statut de révocation d'un certificat |

## ⚠️ Mises à jour critiques et pièges fréquents

### Politique de signature — en vigueur depuis le 07/07/2026

La [Spécification technique Signature Fournisseur V3.0](https://www.tradenet.com.tn/upload/ElFatoora/Specifications_Techniques_Signature_Fournisseur_V3.0.pdf) impose des règles strictement contrôlées lors de la validation de conformité des factures transmises en production :

- **Politique de signature obligatoire** : le bloc `<xades:SigPolicyId>` doit référencer l'OID `urn:2.16.788.1.2.1.3` avec l'attribut `Qualifier="OIDAsURN"` (Politique de Signature Électronique de Tunisie TradeNet).
- **Transformations XPath obligatoires** avant le calcul de la signature, en plus de la canonicalisation C14N exclusive : exclusion explicite des blocs `<ds:Signature>` et `<RefTtnVal>` du périmètre signé.

```xml
<ds:Transform Algorithm="http://www.w3.org/TR/1999/REC-xpath-19991116">
  <ds:XPath>not(ancestor-or-self::ds:Signature)</ds:XPath>
</ds:Transform>
<ds:Transform Algorithm="http://www.w3.org/TR/1999/REC-xpath-19991116">
  <ds:XPath>not(ancestor-or-self::RefTtnVal)</ds:XPath>
</ds:Transform>
```

⚠️ **Une facture signée sans ces éléments est rejetée** lors du contrôle de conformité en production. Le calendrier d'application stricte (07/07/2026) a été confirmé par une communication officielle de Tunisie TradeNet aux prestataires homologués — cette date n'apparaît pas dans le PDF public lui-même, seules les règles techniques y figurent.

### e-Houwiya (Mobile ID) comme méthode de signature

La spécification V3.0 confirme que tous les certificats acceptés par l'ANCE sont reconnus par El Fatoora : **ID-Trust**, **Cachet Entreprise**, **DigiGo** et **e-Houwiya (Mobile ID)**.

### Outil de vérification recommandé par TTN elle-même

TTN recommande dans sa propre documentation l'usage de l'[ETSI Signature Conformance Checker](https://signatures-conformance-checker.etsi.org/) (inscription gratuite requise) pour valider la conformité structurelle d'une signature XAdES avant envoi. Précision utile de l'outil lui-même : il vérifie la **structure** de la signature par rapport aux spécifications ETSI, pas la validité cryptographique ni la chaîne de certificats.

## Autorité de certification

**TunTrust** est l'unique autorité de certification électronique habilitée à délivrer des certificats qualifiés en Tunisie (rôle couvrant également ce qui relève de l'ANCE — Agence Nationale de Certification Électronique).

- [tuntrust.tn](https://www.tuntrust.tn/) — site officiel
- [Politiques de certification](https://www.tuntrust.tn/fr/documents-utiles?field_theme_target_id=53)
- [Certificats racines](https://www.tuntrust.tn/fr/documents-utiles?field_theme_target_id=55)
- [Liste des certificats révoqués (LCR)](https://www.tuntrust.tn/fr/documents-utiles?field_theme_target_id=57)
- [Annuaire des certificats (LDAP)](https://www.tuntrust.tn/LdapSearch/)
- [Révocation d'un certificat](https://www.tuntrust.tn/fr/content/revocation-certificat)

## Bibliothèques et outils

| Ressource | Type | Description |
|---|---|---|
| [esig/dss](https://github.com/esig/dss) | Bibliothèque Java | **Digital Signature Service** — bibliothèque open source de la Commission Européenne pour la création/validation de signatures avancées (XAdES, CAdES, PAdES, ASiC), conforme ETSI. **Confirmé par TTN elle-même** comme base de son implémentation de signature (Spécification technique V3.0, section "Avant-propos") — pas seulement une recommandation externe. |
| [xml-crypto](https://github.com/node-saml/xml-crypto) | Bibliothèque Node.js | Signature et vérification XMLDSig, avec support de la canonicalisation C14N Exclusive. |
| [ETSI Signature Conformance Checker](https://signatures-conformance-checker.etsi.org/) | Outil en ligne | Vérification de la conformité structurelle d'une signature XAdES/CAdES/PAdES — recommandé par TTN elle-même (inscription gratuite requise). Ne valide pas la cryptographie ni la chaîne de certificats. |

*Section volontairement courte au lancement — contributions bienvenues pour l'étoffer (.NET, Python, Go...).*

## Ce que ce dépôt n'est pas

`awesome-teif` est une ressource technique communautaire maintenue par [BTBLABS](https://github.com/BTBLABS). Ce n'est **pas** de la documentation officielle TTN/TunTrust, et ce n'est **pas** un produit.

BTBLABS édite par ailleurs [**TEIF Manager**](https://www.teif.tn), une solution commerciale de conformité à la facturation électronique tunisienne (connecteurs ERP/Oracle/SQL Server/PostgreSQL, orchestration, signature HSM TunTrust). Ce dépôt n'en fait pas la promotion — il documente l'écosystème TEIF au sens large, indépendamment de tout outil commercial.

## Contribuer

Ce dépôt n'existe que par ses contributions. Pour proposer une ressource (lien, outil, bibliothèque, retour d'expérience) : ouvrez une issue ou une pull request. Voir [`CONTRIBUTING.md`](CONTRIBUTING.md).

## Licence

Le contenu de curation de ce dépôt (README, comparatifs, listes de liens) est sous licence [CC0 1.0](LICENSE) — domaine public, réutilisable sans restriction.

Les schémas XSD dans `schemas/` restent la propriété de Tunisie TradeNet — voir [`schemas/README.md`](schemas/README.md) pour les détails de provenance.
