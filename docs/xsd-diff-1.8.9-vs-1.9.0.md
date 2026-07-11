# Comparaison XSD TEIF — 1.8.9 (production) vs 1.9.0 (bêta)

> Basé sur une lecture directe des deux schémas ([`schemas/elfatoora-1.8.9.xsd`](../schemas/elfatoora-1.8.9.xsd) et [`schemas/elfatoora-1.9.0-beta.xsd`](../schemas/elfatoora-1.9.0-beta.xsd)), pas sur une annonce officielle de changelog — TTN n'en publie pas. À prendre comme une analyse technique indépendante, pas comme une source normative.
>
> **Rappel de statut (juillet 2026)** : la 1.9.0 est en bêta/test, la 1.8.9 est la version en production. Ne migrez pas vers la 1.9.0 en production tant que TTN ne l'a pas officiellement basculée.

## Validation métier embarquée (XSD 1.1 `xs:assert`)

La 1.9.0 utilise les fonctionnalités XSD 1.1 pour embarquer des règles métier directement dans le schéma, absentes en 1.8.9 :

- L'identifiant de l'émetteur du message (`MessageSenderIdentifier`) doit correspondre au partenaire ayant le `functionCode='I-62'` dans le corps de la facture.
- Le champ date `I-31` (date de facture) doit apparaître **exactement une fois**, au format `ddMMyyyy`.
- Le compte bancaire/postal (`PytFii`) devient obligatoire dès que le client est un organisme public tunisien (`Country='TN'`), avec un numéro de compte à exactement 20 chiffres.
- Le montant total TTC (`amountTypeCode='I-180'`) doit apparaître **exactement une fois**, en devise `TND`.
- Cohérence forcée entre `TaxCategory` (`Rate` ou `Amount`) et les champs renseignés : un taux (`TaxDetails`) ne peut coexister avec un montant fixe (`TaxAmount`), et inversement — même logique appliquée aux allocations/remises (`AlcCategory`).

## Contrôle de clé sur la matricule fiscale

Nouveauté forte de la 1.9.0 : le type `SenderIdentifierType` (identifiant de l'émetteur) valide désormais la **clé de contrôle** de la matricule fiscale tunisienne par un calcul modulo 23 sur les 7 premiers chiffres, comparé à la lettre de clé fournie. En 1.8.9, seul le format (longueur + motif de caractères) était vérifié, pas la validité arithmétique de la clé.

```
x = Σ(chiffre[i] × poids[i])  pour i=1..7, poids = [7,6,5,4,3,2,1]
lettre_attendue = alphabet_23_lettres[(x mod 23)]
```

(alphabet excluant I, O, U — 23 lettres autorisées : A-H, J-N, P-T, V-Z)

## Renommages d'éléments (cassants pour un parseur strict)

| 1.8.9 | 1.9.0 |
|---|---|
| `AdditionnalDocuments` (élément racine, une seule instance) | `AdditionalDocumentsSection` → contient une liste `AdditionalDocument` (multiple, `maxOccurs="unbounded"`) |
| `AdditionnalDocumentIdentifier` / `AdditionnalDocumentName` | `AdditionalDocumentIdentifier` / `AdditionalDocumentName` (orthographe corrigée : un seul "n") |
| `PartnerAdresses` (multiple) | `PartnerAddress` (un seul, obligatoire pour `NadTestType`) |
| `Loc` direct dans `PartDetailType` | encapsulé dans `LocSection` (type `PartLocSectionType`) |
| `RffSection` / `CtaSection` répétés directement | encapsulés dans `RffSectionGroup` / `CtaSectionGroup` |

## Typage renforcé

- **IBAN** : `AccountNumber` passe d'une chaîne libre (`DataStringType_20`) à un type dédié `IbanType` (motif `[A-Z0-9]{14,34}`).
- **Unités de mesure** (`measurementUnit`) : passe d'une chaîne libre de 8 caractères max à une énumération fermée d'environ 90 codes (UNIT, KGM, LTR, MTR, HUR, KWH, etc. — codes proches de la norme UN/ECE Recommendation 20).
- **Identifiant de document** (`DocumentIdentifier`) : motif contraint `[A-Za-z0-9][A-Za-z0-9._-]{0,34}` au lieu d'une simple limite de longueur.
- **Pays** (`Country` dans `FiiType`) : devient obligatoire (`minOccurs` implicite = 1), optionnel en 1.8.9.

## Extensions de codes

- `DocumentType` (`Bgm`) : passe de 6 codes (`I-11` à `I-16`) à 8 (`I-11` à `I-18`).
- `PartnerIdentifierType` : passe de 4 types (`I-01` à `I-04`) à 6 (`I-01` à `I-06`).
- `TaxTypeName` : extension significative de la liste de sous-codes `I-160x` (de 3 à 19 sous-codes), pour un typage plus fin des taxes.
- `DtmDetailType` : nouveaux `functionCode` (`I-39`, `I-310`) et nouveaux formats de date sur 4 chiffres d'année (`ddMMyyyy`, `ddMMyyyyHHmm`, `ddMMyyyy-ddMMyyyy`), avec validation par expression régulière du format déclaré — absent en 1.8.9.

## Compatibilité de version

- Le schéma 1.8.9 accepte encore `1.8.8` comme valeur de l'attribut `version` (rétrocompatibilité assumée par TTN).
- Le schéma 1.9.0 n'accepte **que** `1.9.0` — aucune rétrocompatibilité déclarée avec les versions 1.8.x.

## À vérifier avant toute migration

Cette comparaison est structurelle (lecture du XSD), pas fonctionnelle. Avant de migrer un intégrateur en production vers 1.9.0 quand elle sera officiellement activée : valider avec de vrais cas de test (voir le futur dépôt `teif-test-suite`), et confirmer la date de bascule officielle auprès de TTN — ce dépôt ne fait pas foi sur le calendrier de migration.
