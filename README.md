#  Bike Store — Big Data Pipeline

A full end-to-end big data pipeline built with **Apache Spark** and **Apache HBase**,
applied to a real-world bike store dataset. The project covers data ingestion,
cleaning, exploratory data analysis, cross-table analytics, and finally loading
the processed data into HBase as a NoSQL store.

---

##  Project Overview

This project simulates a realistic data engineering workflow:

1. **Ingest** raw CSV files into Apache Spark
2. **Clean & Transform** each table (handle nulls, fix types, engineer new features)
3. **Explore** the data with statistics and visualizations
4. **Analyze** relationships across tables (revenue, customers, stores, staff)
5. **Load** the final clean data into Apache HBase

---

##  Dataset
[Kaggle DataSet](https://www.kaggle.com/datasets/dillonmyrick/bike-store-sample-database)


The dataset represents a bike store chain with 9 related tables:

| Table | Description | Rows |
|---|---|---|
| `customers` | Customer personal and location info | 1,445 |
| `orders` | Order headers with dates and status | 1,615 |
| `order_items` | Line items per order with prices | 4,722 |
| `products` | Product catalog with pricing | 321 |
| `categories` | Product categories | 7 |
| `brands` | Product brands | 9 |
| `stores` | Store locations | 3 |
| `staffs` | Staff members and their managers | 10 |
| `stocks` | Inventory levels per store | 939 |

---

##  Tech Stack

| Tool | Version | Purpose |
|---|---|---|
| Apache Spark | 3.x | Data processing and EDA |
| PySpark | 3.x | Python API for Spark |
| Apache HBase | 2.5.x | NoSQL storage |
| happybase | latest | Python client for HBase (via Thrift) |
| Matplotlib | latest | Data visualization |
| Python | 3.11 | Main language |
| Ubuntu 24.04 (WSL) | 24.04 | Runtime environment |
| Java | 21 | Required by Spark and HBase |

---

## 📁 Project Structure

```
bike-store-pipeline/
│
├── data/                        # Raw CSV files
│   ├── customers.csv
│   ├── orders.csv
│   ├── order_items.csv
│   ├── products.csv
│   ├── categories.csv
│   ├── brands.csv
│   ├── stores.csv
│   ├── staffs.csv
│   └── stocks.csv
│
├── notebook/
│   └── pipeline.ipynb           # Main Jupyter notebook
│
└── README.md
```

---

##  Setup & Installation

### Prerequisites

- Ubuntu 24.04 (or WSL2 on Windows)
- Java 21
- Python 3.11 with Conda or virtualenv
- Apache Spark installed
- Apache HBase installed

---

### 1. Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/bike-store-pipeline.git
cd bike-store-pipeline
```

### 2. Install Python Dependencies

```bash
pip install pyspark happybase matplotlib
```

### 3. Set Environment Variables

Add these to your `~/.bashrc`:

```bash
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
export HBASE_HOME=/home/$USER/hbase
export PATH=$PATH:$HBASE_HOME/bin
export HBASE_OPTS="-Djava.net.preferIPv4Stack=true"
```

Then reload:

```bash
source ~/.bashrc
```

### 4. Configure HBase

Edit `~/hbase/conf/hbase-site.xml`:

```xml
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>file:///home/YOUR_USER/hbase-data</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/home/YOUR_USER/zookeeper-data</value>
  </property>
  <property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>localhost</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>false</value>
  </property>
</configuration>
```

### 5. Start HBase and Thrift Server

```bash
start-hbase.sh
hbase thrift start &
sleep 5
jps   # should show HMaster, ThriftServer
```

### 6. Create HBase Tables

```bash
hbase shell
```

```ruby
create 'customers',  {NAME => 'personal'},  {NAME => 'location'}
create 'orders',     {NAME => 'info'},       {NAME => 'dates'}
create 'order_items',{NAME => 'details'}
create 'products',   {NAME => 'info'},       {NAME => 'pricing'}
create 'categories', {NAME => 'info'}
create 'brands',     {NAME => 'info'}
create 'stores',     {NAME => 'info'},       {NAME => 'contact'}
create 'staffs',     {NAME => 'personal'},   {NAME => 'work'}
create 'stocks',     {NAME => 'inventory'}
list
exit
```

### 7. Run the Notebook

```bash
jupyter notebook notebook/pipeline.ipynb
```

---

##  What the Pipeline Does

### Data Cleaning (per table)
- Replaced string `"NULL"` values with actual `null`
- Cast columns to correct data types
- Filled missing values with sensible defaults
- Removed duplicate rows
- Standardized text (uppercase cities/states, title case names)

### Feature Engineering
- `full_name` — combined first and last name for customers and staff
- `total_price` — `quantity × list_price × (1 - discount)` for order items
- `price_category` — LOW / MEDIUM / HIGH based on list price
- `stock_level` — OUT_OF_STOCK / LOW / SUFFICIENT based on quantity

### Cross-Table Analysis

| Analysis | Tables Used | Key Finding |
|---|---|---|
| Revenue by Category | order_items + products + categories | Which bike type earns the most |
| Top Customers | order_items + orders + customers | Who spends the most |
| Store Performance | order_items + orders + stores | Which store has highest revenue |
| Staff vs Store | staffs + stores + revenue | Revenue per active staff member |

### HBase Loading
- All 9 tables loaded using `happybase` batch writes
- Row keys designed for efficient lookups
- Composite keys used for `order_items` (`order_id_item_id`) and `stocks` (`store_id_product_id`)
- Final verification confirms Spark row counts match HBase row counts

---

## 🗃️ HBase Table Design

| Table | Row Key | Column Families |
|---|---|---|
| customers | `customer_id` | `personal`, `location` |
| orders | `order_id` | `info`, `dates` |
| order_items | `order_id_item_id` | `details` |
| products | `product_id` | `info`, `pricing` |
| categories | `category_id` | `info` |
| brands | `brand_id` | `info` |
| stores | `store_id` | `info`, `contact` |
| staffs | `staff_id` | `personal`, `work` |
| stocks | `store_id_product_id` | `inventory` |

---

##  Key Insights

- **Total Revenue:** ~$7.69 million across all stores
- **Top Category:** Road Bikes and Electric Bikes generate the highest average revenue per order
- **Best Store:** Store performance varies — revenue per active staff member reveals true efficiency
- **Customer Segments:** Majority of customers fall in the Medium spending range ($1k–$5k)
- **Inventory:** 25 product-store combinations are completely out of stock

---

##  Restart HBase After WSL Reboot

Every time you restart WSL, run:

```bash
start-hbase.sh
hbase thrift start &
sleep 5
jps
```

---

##  Author

**Dina**
