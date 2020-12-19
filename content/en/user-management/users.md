---
title: Users
description: ''
position: 60
category: User Management
---

GetCandy has two concepts customers and users, users are fundamentally people who can log into your storefront and make purchases, customers on the other hand contain information about the customer or company, they can also have multiple users assigned to them, for information on customers please see the [Customers](doc:customers) section of the guides.

When creating a user a customer will be created for them if a `customer_id` has not been provided during the call, if a `customer_id` is provided and either the requester can manage the customer or they have an invite they will then be assigned to the customer supplied.



