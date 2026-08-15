---
title: "Modugrid Database Design: From Business Model to MySQL"
date: 2026-08-14
draft: false
author: "Disraely Salazar"
tags: ["SQL", "Database Design", "MySQL", "Data Analysis"]
categories: ["blog"]
description: "Designing and building a 10-table relational database that models my own e-commerce business, from ER diagram through normalized MySQL schema, sample data, indexing, and analytical queries."
---

For the final project of my Data Analysis program, I designed and built a complete relational database from scratch — not a textbook example, but a real model of **Modugrid**, the garage floor tile business I founded. The project covers the full lifecycle of a database: requirements, schema design, normalization, implementation in MySQL, sample data, performance indexing, and analytical querying.

<!--more-->

---

## Overview

For the final project of my Data Analysis program, I designed and built a complete relational database from scratch — not a textbook example, but a real model of Modugrid, the garage floor tile business I founded. The project covers the full lifecycle of a database: requirements, schema design, normalization, implementation in MySQL, sample data, performance indexing, and analytical querying.

---

## Modeling a Real Business

Rather than pick a generic example, I modeled my own operation: a product catalog of interlocking tiles sold in 10 colors and multiple pack sizes, sourced from two suppliers, sold across four channels, shipped from two warehouses, with optional add-ons (lighting, edging, installation) available per order.

The core design challenge was recognizing where a many-to-many relationship needed its own table. A single order can contain multiple different products and multiple add-ons at once — no single column can represent that. I resolved this with two junction tables, `ORDER_PRODUCTS` and `ORDER_ADDONS`, each using a composite primary key to link its two parent tables.

The final schema totals 10 tables:

- **Catalog**: `PRODUCTS`, `COLORS`, `SUPPLIER`
- **Transactions**: `ORDERS`, `ORDER_PRODUCTS`, `ORDER_ADDONS`
- **Supporting entities**: `ADD_ONS`, `MARKETPLACE`, `WAREHOUSE`, `CUSTOMER`

Every foreign key relationship across the design uses consistent color-coding between the primary and foreign key sides, making the full relational structure readable at a glance before a single line of SQL was written.

---

## Implementation in MySQL

With the design finalized, I wrote the full schema as a `CREATE TABLE` script — sequencing table creation so that every lookup table exists before anything references it, and every junction table is created last, after both of its parent tables:

```sql
CREATE TABLE ORDER_PRODUCTS (
    Order_ID    INT,
    Product_ID  INT,
    Quantity    INT          NOT NULL,
    Price       DECIMAL(7,2),
    PRIMARY KEY (Order_ID, Product_ID),
    FOREIGN KEY (Order_ID)   REFERENCES ORDERS(Order_ID),
    FOREIGN KEY (Product_ID) REFERENCES PRODUCTS(Product_ID)
);
```

From there, I populated every table with realistic sample data, added indexes on the columns most likely to be filtered or joined in practice — product color, supplier, customer order history, order date — and wrote a set of analytical queries covering basic selects, multi-table joins, aggregate functions, `WHERE`/`HAVING` filtering, and nested subqueries.

One query identifies which sales channels are generating more orders than the channel average — the kind of question a real business owner would actually ask of their own data.

---

## Skills Demonstrated

- Relational schema design and normalization, including correctly identifying when a many-to-many relationship requires a junction table
- Writing constraint-safe `CREATE TABLE` scripts with proper foreign key sequencing
- Populating a multi-table schema with referentially consistent sample data
- Index selection based on realistic query patterns
- SQL querying: joins, aggregate functions, `GROUP BY`/`HAVING`, and nested subqueries
- Debugging real MySQL errors by reading server error messages precisely rather than guessing at fixes

---

*Disraely Salazar is a Data Analytics student at triOS College in Toronto, ON, with a background in business operations and supply chain management.*
