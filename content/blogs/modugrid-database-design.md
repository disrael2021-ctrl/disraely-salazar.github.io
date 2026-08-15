---
title: "Modugrid Database Design: From Business Model to MySQL"
date: 2026-08-14
draft: false
author: "Disraely Salazar"
tags: ["SQL", "Database Design", "MySQL", "Data Analysis"]
categories: ["blog"]
description: "Designing and building a 10-table relational database that models my own e-commerce business, from ER diagram through normalized MySQL schema, sample data, indexing, and analytical queries."
---

One of the best ways to learn database design is to model something you already understand deeply. For this project, I used my own e-commerce business — **Modugrid**, a brand that sells garage floor tiles in a variety of colors and sizes, through multiple online marketplaces, with add-on installation products and a physical warehouse — as the subject for a complete MySQL relational database.

This is a walkthrough of how I went from business model to working schema: identifying entities, normalizing relationships, writing constraint-safe CREATE TABLE scripts, populating the data, adding indexes, and writing analytical queries against it.

<!--more-->

---

## Overview

**Modugrid** sells modular garage floor tiles in multiple colors and configurations. Customers find the product on marketplaces like Amazon and eBay, place orders that may include add-on products (installation kits, edge pieces), and products are fulfilled from a warehouse. The business has multiple suppliers and tracks inventory across product types.

The goal was to translate this business model into a normalized relational database that could answer real questions — things like:

- Which sales channels are generating the most orders?
- What is the total revenue per marketplace?
- Which products sell most frequently together as add-ons?
- Which suppliers provide which products?

---

## Modeling a Real Business

The first step was identifying the core entities and the relationships between them. Starting from what I knew about Modugrid's actual operations:

**Entities identified:**

- `PRODUCTS` — the tile SKUs (sizes, types)
- `COLORS` — color variants for each product
- `SUPPLIER` — vendors providing the products
- `ORDERS` — customer purchase transactions
- `CUSTOMER` — the buyer
- `MARKETPLACE` — sales channels (Amazon, eBay, Shopify, etc.)
- `WAREHOUSE` — where inventory is stored
- `ADD_ONS` — supplementary products (edge trim, installation kits)
- `ORDER_PRODUCTS` — junction table: which products are in which orders
- `ORDER_ADDONS` — junction table: which add-ons are in which orders

**Key relationship decisions:**

The hardest part of schema design is identifying many-to-many relationships that require junction tables. A single order can contain multiple products, and a single product can appear in many orders — that's `ORDER_PRODUCTS`. The same logic applies to add-ons: `ORDER_ADDONS` bridges `ORDERS` and `ADD_ONS`.

This kind of normalization prevents data duplication and keeps the schema consistent as the data grows.

---

## Implementation in MySQL

### Foreign Key Sequencing

A critical discipline in writing CREATE TABLE scripts is sequencing. A table cannot reference another table that doesn't exist yet. The correct build order for this schema:

1. `COLORS` (no dependencies)
2. `SUPPLIER` (no dependencies)
3. `MARKETPLACE` (no dependencies)
4. `WAREHOUSE` (no dependencies)
5. `CUSTOMER` (no dependencies)
6. `ADD_ONS` (no dependencies)
7. `PRODUCTS` (references COLORS, SUPPLIER, WAREHOUSE)
8. `ORDERS` (references CUSTOMER, MARKETPLACE)
9. `ORDER_PRODUCTS` (references ORDERS, PRODUCTS)
10. `ORDER_ADDONS` (references ORDERS, ADD_ONS)

### Sample Table: ORDER_PRODUCTS

The junction table connecting orders and products is a good example of the schema design pattern used throughout:

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

Using a **composite primary key** on `(Order_ID, Product_ID)` ensures each product appears only once per order, while the foreign keys enforce referential integrity — no orphan records.

### Populating Sample Data

After creating all tables, I populated them with referentially consistent sample data. This means inserting in the same order as the CREATE TABLE sequence — you cannot insert an order referencing a customer who doesn't exist yet. I used realistic values: actual marketplace names, product configurations that reflect Modugrid's real catalog, and order amounts consistent with real e-commerce transactions.

### Indexing Strategy

Indexes are added after the data is in place. I selected index targets based on which columns appear in WHERE clauses or JOIN conditions in realistic queries:

- `PRODUCTS(Supplier_ID)` — frequent supplier lookups
- `ORDERS(Marketplace_ID)` — channel performance queries
- `ORDERS(Customer_ID)` — customer order history
- `ORDER_PRODUCTS(Product_ID)` — product frequency analysis

Adding indexes to foreign key columns speeds up JOIN performance significantly on large datasets.

---

## Analytical Queries

The real test of a schema is whether it can answer business questions. Here are examples from this project:

**Which marketplaces generate above-average order volume?**

```sql
SELECT m.Name, COUNT(o.Order_ID) AS Total_Orders
FROM ORDERS o
JOIN MARKETPLACE m ON o.Marketplace_ID = m.Marketplace_ID
GROUP BY m.Name
HAVING COUNT(o.Order_ID) > (
    SELECT AVG(order_count)
    FROM (
        SELECT COUNT(Order_ID) AS order_count
        FROM ORDERS
        GROUP BY Marketplace_ID
    ) AS avg_orders
);
```

This uses a subquery to compute the average and a HAVING clause to filter channels that exceed it — translating a direct business question into SQL.

**Total revenue and units sold per product:**

```sql
SELECT p.Name, SUM(op.Quantity) AS Units_Sold, SUM(op.Quantity * op.Price) AS Revenue
FROM ORDER_PRODUCTS op
JOIN PRODUCTS p ON op.Product_ID = p.Product_ID
GROUP BY p.Name
ORDER BY Revenue DESC;
```

**Which add-ons are purchased most frequently?**

```sql
SELECT a.Name, COUNT(oa.Order_ID) AS Frequency
FROM ORDER_ADDONS oa
JOIN ADD_ONS a ON oa.AddOn_ID = a.AddOn_ID
GROUP BY a.Name
ORDER BY Frequency DESC;
```

---

## Skills Demonstrated

This project covered the full lifecycle of relational database work:

- **Schema design** — entity identification, normalization to 3NF, junction table construction
- **Constraint-safe CREATE TABLE scripts** — correct foreign key sequencing to avoid dependency errors
- **Sample data population** — referentially consistent inserts across 10 tables
- **Index selection** — targeting high-frequency JOIN and WHERE columns
- **SQL querying** — multi-table JOINs, GROUP BY with HAVING, aggregate functions, nested subqueries

The full 10-table schema — PRODUCTS, COLORS, SUPPLIER, ORDERS, ORDER_PRODUCTS, ORDER_ADDONS, ADD_ONS, MARKETPLACE, WAREHOUSE, and CUSTOMER — models a real e-commerce operation with enough complexity to practice every major SQL skill in a meaningful context.

---

*Disraely Salazar is a Data Analytics student at triOS College in Toronto, ON, with a background in business operations and supply chain management.*
