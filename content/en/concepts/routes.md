---
title: Routes
description: ''
position: 40
category: Concepts
---

Not to be confused with the routing in your Laravel app, GetCandy routes are database entries that allow you to specify different routes for each resources that the API can react to.

This is helpful as normally an API will generally require an ID  to be passed when you want a specific resource, this isn't too helpful if you just have a url on your storefront called `some-product`. How do we let the API know which resource we want to fetch? What if we want to support slugs in different languages or redirects?

This is where routes come in, they allow you to have vanity URL's for your products/categories etc whilst still having a clear path with how to retrieve the resource from the API without running circles around yourself.

### Validation

Routes have both a `path` and `slug` field available, each route must be unique across both of these fields. For example if you have a route with a path `foo` and slug `bar`, you can create an additional route with the slug `bar` providing the path is different to `foo`



