---
title: "Russian Glossary"
description: "Russian translation references for key project terminology."
---

# Russian Glossary

This section provides standard Russian translations for key project terminology to ensure consistency and accuracy in Russian localization.

## Core Concepts

- Emojis may be used in translations if they appear in the original text.
- Purely technical terms may remain unchanged if they appear in the original text.
- English technical terms may be used when they are widely accepted in Russian technical contexts, for example API.

| Source Term | Russian | English | Description |
| --- | --- | --- | --- |
| Ratio | Коэффициент | Ratio / Multiplier | Multiplier used for price calculation. In pricing contexts, use "Коэффициент" rather than "Множитель" to maintain terminology consistency. |
| Token | Токен | Token | API credential or text unit. |
| Channel | Канал | Channel | Access channel for an API provider. |
| Group | Группа | Group | Classification of users or tokens. |
| Quota | Квота | Quota | Service quota available to a user. |

## Model Related

| Source Term | Russian | English | Description |
| --- | --- | --- | --- |
| Prompt | Промпт / Ввод | Prompt | Model input content. |
| Completion | Вывод | Completion | Model output content. Do not use "Дополнение" or "Завершение"; use "Вывод" to match technical terminology. |
| Input | Ввод | Input / Prompt | Content sent to the model. |
| Output | Вывод | Output / Completion | Content returned by the model. |
| Model Ratio | Коэффициент модели | Model Ratio | Pricing ratio for different models. |
| Completion Ratio | Коэффициент вывода | Completion Ratio | Additional pricing ratio for output content. |
| Fixed Price | Цена за запрос | Price per call | Price per call. |
| Pay-as-you-go | Оплата по объему | Pay-as-you-go | Usage-based billing. |
| Pay per call | Оплата за запрос | Pay-per-view | Fixed price per call. |

## User Management

| Source Term | Russian | English | Description |
| --- | --- | --- | --- |
| Root User | Суперадминистратор | Root User | Administrator with the highest privileges. |
| Admin User | Администратор | Admin User | System administrator. |
| Normal User | Обычный пользователь | Normal User | User with standard privileges. |

## Top-up and Redemption

| Source Term | Russian | English | Description |
| --- | --- | --- | --- |
| Top Up | Пополнение | Top Up | Adds quota to an account. |
| Redemption Code | Код купона | Redemption Code | Code that can be exchanged for quota. |

## Channel Management

| Source Term | Russian | English | Description |
| --- | --- | --- | --- |
| Channel | Канал | Channel | API provider channel. |
| API Key | API ключ | API Key | API access key. Use "API ключ" instead of "API токен" for precision and to follow common Russian technical terminology. |
| Priority | Приоритет | Priority | Channel selection priority. |
| Weight | Вес | Weight | Load-balancing weight. |
| Proxy | Прокси | Proxy | Proxy server address. |
| Model Mapping | Перенаправление модели | Model Mapping | Replacement of the model name in the request body. |
| Provider | Поставщик | Provider / Vendor | Service or API provider. |

## Security Related

| Source Term | Russian | English | Description |
| --- | --- | --- | --- |
| Two-Factor Authentication | Двухфакторная аутентификация | Two-Factor Authentication | Additional security verification method for accounts. |
| 2FA | 2FA | Two-Factor Authentication | Abbreviation for two-factor authentication. |

## Translation Guidelines

### Contextual Translation Variants

**Промпт / Ввод (Prompt / Input)**

- **Промпт**: Use for LLM interaction, user interfaces, and descriptions of interaction with a model.
- **Ввод**: Use in pricing, technical documentation, and descriptions of data-processing flows.
- **Rule**: If the context is user experience or AI interaction, use "Промпт". If the context is technical processing or calculation, use "Ввод".

**Token**

- API токен доступа (API Token)
- Текстовая единица, обрабатываемая моделью (Text Token)
- Токен доступа к системе (Access Token)

**Квота (Quota)**

- Service quota available to the user.
- Sometimes translated as "Кредит".

### Russian Language Notes

- **Plural forms**: Implement plural forms correctly, such as `_one`, `_few`, `_many`, and `_other`.
- **Case endings**: Pay close attention to case endings in technical terms.
- **Grammatical gender**: Match the grammatical gender of technical terms, for example "модель" is feminine and "канал" is masculine.

### Standardized Terms

- **Вывод (Completion)**: Model output content.
- **Коэффициент (Ratio)**: Multiplier used for price calculation.
- **Код купона (Redemption Code)**: Preferred term for redemption codes.
- **Поставщик (Provider / Vendor)**: Organization or service that provides APIs or AI models.

---

**Contribution Note for Russian:** If you find terminology inconsistencies or have better Russian translation suggestions, please submit an Issue or Pull Request.
