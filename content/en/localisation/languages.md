---
title: Languages
description: ''
position: 50
category: Localisation
---

Languages in GetCandy are used to help translate attribute data from resources. Each language's `iso` column must be unique but they can share the same `lang` value.

The API will look for the `'accept-language'` header and set the app language based on this value and what's available in the database. If a language present in the header does not exist in the database, GetCandy will fallback to whatever the default language is.

If the `accept-language` header is not present GetCandy will fallback to the default language automatically.

