# Prueba Técnica – Data Engineer

## Descripción del Proyecto
Este proyecto implementa un **pipeline ETL** que consume datos desde una **API REST**, los transforma y los carga en un **Data Warehouse relacional** (MySQL) y, en una versión alternativa, hacia **BigQuery** usando **PySpark**.


---

## Stack Tecnológico
- **Lenguaje:** Python 3 y PySpark  
- **Librerías principales:** `requests`, `pandas`, `mysql.connector`, `pyspark`  
- **Data Warehouse:** MySQL y BigQuery (para la versión PySpark)  

---

## Flujo del Pipeline ETL

### 1️⃣ Extract
- Se consumen los endpoints `/products` y `/purchases` desde la API con autenticación mediante header `x-api-key`.
- Los datos son almacenados en memoria en formato JSON.

### 2️⃣ Transform
- Normalización de precios y fechas.
- Cálculo del **total por compra** aplicando descuentos.
- Creación de la tabla intermedia **purchase_products** que asocia productos con compras y su cantidad.
- Limpieza y ajuste de tipos de datos.

### 3️⃣ Load
- **Versión 1:** Inserción en base de datos **MySQL**, por lotes de 1000 registros (`executemany`).
- **Versión 2:** Carga de los DataFrames en **BigQuery** usando el conector oficial de Spark.

---

## Esquema Relacional

### Tabla: `products`
| Campo | Tipo | Descripción |
|--------|------|-------------|
| id | VARCHAR(100) PK | ID del producto |
| name | VARCHAR(255) | Nombre |
| description | TEXT | Descripción |
| price | DECIMAL(10,2) | Precio |
| category | VARCHAR(100) | Categoría |
| created_at | DATE | Fecha de creación |

### Tabla: `purchases`
| Campo | Tipo | Descripción |
|--------|------|-------------|
| id | VARCHAR(100) PK | ID de la compra |
| status | VARCHAR(50) | Estado |
| credit_card_type | VARCHAR(50) | Tipo de tarjeta |
| purchase_date | DATE | Fecha |
| total | DECIMAL(12,2) | Total calculado |

### Tabla: `purchase_products`
| Campo | Tipo | Descripción |
|--------|------|-------------|
| purchase_id | VARCHAR(100) FK | ID de la compra |
| product_id | VARCHAR(100) FK | ID del producto |
| quantity | INT | Cantidad |
**Clave primaria compuesta:** `(purchase_id, product_id)`

---

## Scripts SQL de Creación

```sql
CREATE TABLE products (
  id VARCHAR(100) NOT NULL,
  name VARCHAR(255),
  description TEXT,
  price DECIMAL(10,2),
  category VARCHAR(100),
  created_at DATE,
  PRIMARY KEY (id)
);

CREATE TABLE purchases (
  id VARCHAR(100) NOT NULL,
  status VARCHAR(50),
  credit_card_type VARCHAR(50),
  purchase_date DATE,
  total DECIMAL(12,2),
  PRIMARY KEY (id)
);

CREATE TABLE purchase_products (
  purchase_id VARCHAR(100) NOT NULL,
  product_id VARCHAR(100) NOT NULL,
  quantity INT NOT NULL,
  PRIMARY KEY (purchase_id, product_id),
  FOREIGN KEY (purchase_id) REFERENCES purchases(id),
  FOREIGN KEY (product_id) REFERENCES products(id)
);
```

---

## Ejecución

### 1️⃣ Versión Python + MySQL
```bash
pip install requests pandas mysql-connector-python
Jupyter Prueba_tecnica_Windmar_Home.ipynb
```

### 2️⃣ Versión PySpark + BigQuery
#### Requisitos
- Proyecto en GCP con BigQuery habilitado.
- Credenciales del Service Account con permisos de escritura.

#### Variables de entorno
```bash
export GOOGLE_CLOUD_PROJECT="tu_proyecto"
export GOOGLE_APPLICATION_CREDENTIALS="/ruta/credenciales.json"
export BQ_DATASET="etl_windmar"
```

#### Ejecución
```bash
spark-submit   --packages com.google.cloud.spark:spark-bigquery-with-dependencies_2.12:0.36.1   pyspark_etl_bigquery.py
```

---

## 🧭 Arquitectura

```
        +----------------+
        |  REST API      |
        | (Products &    |
        |  Purchases)    |
        +--------+-------+
                 |
                 v
        +----------------+
        |   Extract      |
        | (requests)     |
        +--------+-------+
                 |
                 v
        +----------------+
        |   Transform    |
        | (pandas /      |
        |  pyspark)      |
        +--------+-------+
                 |
                 v
        +----------------+
        |     Load       |
        | (MySQL /       |
        |  BigQuery)     |
        +----------------+
```

---

