---
title: Search
description: ''
position: 100
category: Miscellaneous
---

Search by default uses Elasticsearch 7.9 but is driver based so this can be changed within the configuration to another implementation including your own such as a basic database driven search.


## Elasticsearch indexing
Indexing updates are carried out when the varying models related to the search are updated, if however, you wish to force a full reindex at any time you can use the below commands.

```
php artisan candy:categories:reindex
php artisan candy:products:reindex
```
