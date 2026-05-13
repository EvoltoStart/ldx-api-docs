---
title: "French Glossary"
description: "French translation references for key project terminology."
---

# French Glossary

This document provides standard French translations for key project terminology to ensure consistency and accuracy in French localization.

## Core Concepts

- Emojis may be used in translations if they appear in the original text.
- Pure technical terms may remain unchanged if they appear in the original text.
- English technical terms may be used when they are widely accepted in French technical contexts, for example API.

| Source Term | French | English | Description |
| --- | --- | --- | --- |
| Ratio | Ratio | Ratio / Multiplier | Multiplier used for price calculation. Always use "Ratio" rather than "Multiplicateur" in pricing contexts to maintain terminology consistency. |
| Token | Jeton | Token | API access credential or text unit processed by a model. |
| Channel | Canal | Channel | Access channel for API providers. |
| Group | Groupe | Group | Classification of users or tokens. |
| Quota | Quota | Quota | Service quota available to a user. |

## Model Related

| Source Term | French | English | Description |
| --- | --- | --- | --- |
| Prompt | Invite | Prompt | Model input content. |
| Completion | Complétion | Completion | Model output content. Do not use "Achèvement" or "Finalisation"; use "Complétion" to match technical terminology. |
| Input | Entrée | Input / Prompt | Content sent to the model. |
| Output | Sortie | Output / Completion | Content returned by the model. |
| Model Ratio | Ratio du modèle | Model Ratio | Pricing ratio for different models. |
| Completion Ratio | Ratio de complétion | Completion Ratio | Additional pricing ratio for output content. |
| Fixed Price | Prix fixe | Price per call | Price per call. |
| Pay-as-you-go | Paiement à l'utilisation | Pay-as-you-go | Usage-based billing. |
| Pay per call | Paiement par appel | Pay-per-view | Fixed price per call. |

## User Management

| Source Term | French | English | Description |
| --- | --- | --- | --- |
| Root User | Super-administrateur | Root User | Administrator with the highest privileges. |
| Admin User | Administrateur | Admin User | System administrator. |
| Normal User | Utilisateur normal | Normal User | User with standard privileges. |

## Top-up and Redemption

| Source Term | French | English | Description |
| --- | --- | --- | --- |
| Top Up | Recharge | Top Up | Adds quota to an account. |
| Redemption Code | Code d'échange | Redemption Code | Code that can be exchanged for quota. |

## Channel Management

| Source Term | French | English | Description |
| --- | --- | --- | --- |
| Channel | Canal | Channel | API provider channel. |
| API Key | Clé API | API Key | API access key. Use "Clé API" instead of "Jeton API" for precision and to follow common French technical terminology. |
| Priority | Priorité | Priority | Channel selection priority. |
| Weight | Poids | Weight | Load-balancing weight. |
| Proxy | Proxy | Proxy | Proxy server address. |
| Model Mapping | Redirection de modèle | Model Mapping | Replacement of the model name in the request body. |
| Provider | Fournisseur | Provider / Vendor | Service or API provider. |

## Security Related

| Source Term | French | English | Description |
| --- | --- | --- | --- |
| Two-Factor Authentication | Authentification à deux facteurs | Two-Factor Authentication | Additional security verification method for accounts. |
| 2FA | 2FA | Two-Factor Authentication | Abbreviation for two-factor authentication. |

## Translation Guidelines

### Contextual Translation Variants

**Invite / Entrée (Prompt / Input)**

- **Invite**: Use for LLM interaction, user interfaces, and descriptions of interaction with a model.
- **Entrée**: Use in pricing, technical documentation, and descriptions of data-processing flows.
- **Rule**: If the context is user experience or AI interaction, use "Invite". If the context is technical processing or calculation, use "Entrée".

**Jeton (Token)**

- Jeton d'accès API (API Token)
- Unité de texte traitée par le modèle (Text Token)
- Jeton d'accès système (Access Token)

**Quota**

- Service quota available to the user.
- Sometimes translated as "Crédit".

### French Language Notes

- **Plural forms**: Implement plural forms correctly, such as `_one` and `_other`.
- **Agreement**: Pay attention to grammatical agreement in technical terms.
- **Gender**: Match the grammatical gender of technical terms, for example "modèle" is masculine and "canal" is masculine.

### Standardized Terms

- **Complétion (Completion)**: Model output content.
- **Ratio (Ratio)**: Multiplier used for price calculation.
- **Code d'échange (Redemption Code)**: Preferred term for redemption codes.
- **Fournisseur (Provider / Vendor)**: Organization or service that provides APIs or AI models.

---

**Contribution Note for French:** If you find terminology inconsistencies or have better French translation suggestions, please submit an Issue or Pull Request.
