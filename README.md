# 🛒 Superstore Sales — PostgreSQL Data Engineering Project

A complete Data Engineering pipeline that structures and centralizes Superstore sales data into a normalized PostgreSQL database, enabling efficient SQL queries and preparing the data for interactive dashboards.

---

## 📋 Project Overview

This project transforms a flat CSV dataset (`superstore_cleaned.csv`) into a fully normalized relational database (3NF) hosted in PostgreSQL, with the entire infrastructure managed via Docker.

### Goals
- Design a normalized relational schema (3NF)
- Separate entities: Orders, Customers, Products, Categories, Regions
- Create tables with primary and foreign keys
- Load cleaned data via SQLAlchemy
- Write business SQL queries and views
- Prepare data for dashboard visualization

---

## 🗂️ Project Structure

```
superstore_project/
├── docker-compose.yml          # PostgreSQL + pgAdmin + Jupyter
├── Dockerfile.jupyter          # Minimal Python 3.11 Jupyter image
├── data/
│   └── superstore_cleaned.csv  # Source dataset
├── notebooks/
│   └── superstore_pipeline.ipynb  # Main pipeline notebook
└── sql/
    └── views.sql               # SQL views for analysis
```

---

## 🧩 Relational Schema

```
regions ──────────────────────────┐
  region_id (PK)                  │
  region, country, state...       │
                                  ▼
customers ──────────────► orders ◄──────────── order_items (FACT)
  customer_id (PK)          order_id (PK)        item_id (PK)
  customer_name             customer_id (FK)      order_id (FK)
  segment                   region_id (FK)        product_id (FK)
                            order_date            sales
                            ship_mode             delivery_days
                            year, month, quarter

categories ──────────► products
  category_id (PK)       product_id (PK)
  category               product_name
  sub_category           category_id (FK)
```

**6 tables total** — fully normalized to 3NF, no redundancy, referential integrity enforced.

---

## 🐳 Tech Stack

| Tool | Purpose |
|---|---|
| Python 3.11 | Data processing & loading |
| PostgreSQL 15 | Relational database |
| SQLAlchemy | ORM & database connection |
| psycopg2 | PostgreSQL driver |
| pandas | CSV processing & normalization |
| Docker & Docker Compose | Infrastructure management |
| pgAdmin 4 | Database GUI |
| JupyterLab | Interactive development |

---

## 🚀 Getting Started

### Prerequisites
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running

### 1. Clone the repository
```bash
git clone https://github.com/your-username/superstore-postgresql.git
cd superstore-postgresql
```

### 2. Add your dataset
Place your `superstore_cleaned.csv` file in the `data/` folder.

### 3. Start all containers
```bash
docker-compose up --build -d
```

### 4. Verify containers are running
```bash
docker ps
```
You should see 3 containers:
```
superstore_postgres   ✅
superstore_pgadmin    ✅
superstore_jupyter    ✅
```

### 5. Open JupyterLab
Go to **http://localhost:8888** and run the notebook cells in order.

### 6. Open pgAdmin
Go to **http://localhost:8080**

| Field | Value |
|---|---|
| Email | `admin@superstore.com` |
| Password | `admin123` |

**Add a new server with:**
| Field | Value |
|---|---|
| Host | `superstore_postgres` |
| Port | `5432` |
| Database | `superstore_db` |
| Username | `superstore_user` |
| Password | `superstore_pass` |

---

## 📊 Business Queries

### Total Sales by Category
```sql
SELECT c.category, ROUND(SUM(oi.sales)::numeric, 2) AS total_sales
FROM order_items oi
JOIN products p   ON oi.product_id = p.product_id
JOIN categories c ON p.category_id = c.category_id
GROUP BY c.category
ORDER BY total_sales DESC;
```

### Sales by Region
```sql
SELECT r.region, ROUND(SUM(oi.sales)::numeric, 2) AS total_sales
FROM order_items oi
JOIN orders o  ON oi.order_id = o.order_id
JOIN regions r ON o.region_id = r.region_id
GROUP BY r.region
ORDER BY total_sales DESC;
```

### Top 10 Customers by Revenue
```sql
SELECT cu.customer_name, ROUND(SUM(oi.sales)::numeric, 2) AS revenue
FROM order_items oi
JOIN orders o     ON oi.order_id = o.order_id
JOIN customers cu ON o.customer_id = cu.customer_id
GROUP BY cu.customer_name
ORDER BY revenue DESC
LIMIT 10;
```

### Orders per Month
```sql
SELECT year, month, COUNT(DISTINCT order_id) AS nb_orders
FROM orders
GROUP BY year, month
ORDER BY year, month;
```

---

## 👁️ SQL Views

| View | Description |
|---|---|
| `v_sales_by_category` | Sales and item count by category and sub-category |
| `v_sales_by_region` | Sales and order count by region and state |
| `v_top_customers` | Customers ranked by total revenue |
| `v_monthly_sales` | Total sales and orders aggregated by month |

---

## 📦 Dataset Columns

```
row_id, order_id, order_date, ship_date, ship_mode,
customer_id, customer_name, segment, country, city, state,
postal_code, region, product_id, category, sub-category,
product_name, sales, year, month, quarter, delivery_days
```

---

## 🔒 Security Note

The base image uses `python:3.11-slim` instead of `jupyter/scipy-notebook` to minimize vulnerabilities (reduced from 4 critical / 144 high to near zero).

---

## 📁 Environment Variables

All credentials are managed in `docker-compose.yml`:

```yaml
POSTGRES_USER:     superstore_user
POSTGRES_PASSWORD: superstore_pass
POSTGRES_DB:       superstore_db
```

---
