# Northline Outfitters — Database Design 

---
**Group Info**

GROUP A4

Colby Broome-@colbybroome- Database & Conceptual Modeler

Angelis Aponte-@AngelisAponte- Data Wrangler & SQL Writer

### **Case Summary**

**Company Overview**
Northline Outfitters is a growing online retail company specializing in student-friendly lifestyle and tech accessories. Their product catalog includes items such as hoodies, water bottles, desk lamps, and computer peripherals. The company operates internationally, sourcing merchandise from various outside vendors and selling directly to consumers across the United States and Canada.

**Data Challenges**
As a rapidly expanding business, Northline Outfitters has reached a critical point where its current reliance on Excel spreadsheets is no longer sustainable. The existing data is "dirty" and unnormalized, presenting several significant challenges for reliable business operations:

* **Inconsistent Formatting:** The records contain a mix of international date formats (e.g., DD-MM-YYYY vs. MM-DD-YYYY) and varying currency indicators (USD and CAD) embedded directly within numeric fields.
* **Embedded Identifiers:** Critical information is often buried within text strings, such as customer loyalty status and country indicators being combined with names or addresses.
* **Data Redundancy and Integrity:** The current files contain partial duplicates, inconsistent naming conventions for categories and vendors, and missing values in key fields like email addresses.
* **Lack of Relational Structure:** Because the data is stored in flat files ("Sales Dump" and "Product Master"), there is no enforced relationship between employees and their managers, or products and their specific suppliers, leading to potential "update anomalies" and reporting errors.

To support future growth, the company requires a transition from these messy spreadsheets to a structured relational database that enforces data integrity and allows for complex SQL-based analysis.

---

### **Conceptual Data Model**

<img width="1054" height="718" alt="gp2final" src="https://github.com/user-attachments/assets/00400dae-c11a-4c5f-826e-9fdd5311e152" />

---

### **Entity Relationships**

**Customers and Orders**
The relationship between **Customers** and **Orders** is **1:M**. A single customer can place multiple orders over time, but each specific order record is linked back to only one unique customer.

**Employees and Orders**
The relationship between **Employees** and **Orders** is **1:M**. One employee can process or be responsible for many different orders, while each order is handled by exactly one primary employee.

**Employees and Employees (Management)**
The relationship between **Employees** and themselves is a **1:M Recursive** relationship. An individual employee can act as a manager for multiple other employees, but each subordinate employee reports to only one manager. This is implemented via the `manager_ref` self-referencing foreign key on the Employees table.

**Vendors and Products**
The relationship between **Vendors** and **Products** is **1:M**. One vendor can supply a variety of different products to Northline Outfitters, but each specific product is sourced from one primary vendor.

**Categories and Products (Primary Category)**
The relationship between **Categories** and **Products** is **1:M**. A single category (e.g., "Tech") can contain many different products, while each product is assigned to one primary category for organizational purposes.

**Categories and Products (Subcategory)**
A product may optionally belong to a secondary category. This is represented by the `sub_category_id` field on the Products table, which references the Categories table with a zero-or-one cardinality on the Products side — meaning a subcategory is not required.

**Orders and Order_Lines**
The relationship between **Orders** and **Order_Lines** is **1:M**. One order can consist of multiple line items (different products or quantities), but each specific line item belongs to only one master order.

**Products and Order_Lines**
The relationship between **Products** and **Order_Lines** is **1:M**. A single product can appear on many different order lines across various customer transactions, but each line item refers to one specific SKU.

**Orders and Payments**
The relationship between **Orders** and **Payments** is **1:M**. While most orders have a single payment, an order can technically be fulfilled through multiple payment installments or methods, though each payment record is tied to one specific order.

---

### **Entity Breakdown**

The database is structured around a central **Orders** hub, which captures the transaction logic by linking three primary business drivers: **Customers**, **Employees** (who facilitate sales and report to a specific **Manager** hierarchy via a recursive self-join on `manager_ref`), and **Payments**. To handle the specifics of what was purchased, the **Orders** entity connects to **Order_Lines**, a bridge table that breaks down individual transaction details such as quantity, unit price, discount, discount type, tax, and line total. These lines point directly to the **Products** entity, which serves as the master record for inventory. Each product is detailed with its own operational attributes — including cost, list price, reorder level, and discontinuation status — and is assigned a primary **Category** via `category_id`, with an optional secondary category tracked through `sub_category_id`, both referencing the **Categories** table. Finally, the **Vendor** entity stores the essential contact and representative information for the vendors that provide these products, ensuring the entire supply chain — from procurement to the final customer receipt — is fully integrated.

---

### **Data Quality Assessment**

Below is a detailed assessment of the data quality issues found in the `Sales_Dump` and `Product_Supplier_Master` files. These issues prevent the data from being used in a relational database without prior cleaning and transformation.

| Data Category | Specific Issues Identified | Examples from Data |
| :--- | :--- | :--- |
| **Date Formats** | Significant inconsistency in date recording, including U.S. styles, international styles, and written-out months. | `10-11-2025`, `Oct 17 25`, `October 5 25`, `10 Sep 2025` |
| **Customer Info** | Attributes are concatenated into single strings using multiple types of delimiters (`;`, `\|`) and contain non-atomic data. First and last name must be parsed into separate fields. | `Mason Rivera; Loyalty? Y`, `Grace Hall \| Student \| US` |
| **Currency & Numeric** | Numeric fields contain string prefixes (USD/CAD) or currency symbols, which prevent mathematical calculations. | `USD 6.25`, `CAD 31.40`, `$19.43` |
| **Case Sensitivity** | Identical categorical values are recorded with inconsistent capitalization. | `VISA` vs. `visa`, `Debit` vs. `debit` |
| **Category Naming** | Product categories use different separators and inconsistent naming conventions for sub-categories. | `Tech / Student`, `Tech & Student`, `Audio / Student` |
| **Missing Values** | Key contact information is missing or replaced with placeholder text in the middle of data fields. | `liam_patel@school.edu` (Missing emails for others), `Taylor Green / email missing` |
| **Inconsistent Units** | Physical measurements (weight/length) use mixed units (imperial vs. metric) and varying abbreviations. | `8 ounces`, `499 g`, `10.2 in`, `25.4 cm` |
| **Redundancy** | Partial duplicates exist where the same product description appears with slightly different SKU casing or pricing. | `SKU-U-1003` vs. `sku-u-1003`, `Aurora Mechanical Keyboard` (Duplicate desc) |

---

### **Data Transformation**


| Objective | Target Attribute | SQL Transformation Logic / Function |
| :--- | :--- | :--- |
| **Currency Normalization** | `All Financials` | `CASE WHEN cost LIKE 'CAD%' THEN CAST(REGEXP_REPLACE(cost, '[^0-9.]', '') AS DECIMAL(10,2)) * 0.73 ELSE CAST(REGEXP_REPLACE(cost, '[^0-9.]', '') AS DECIMAL(10,2)) END` *(Assumes 0.73 exchange rate for CAD to USD)* |
| **Standardize Dates** | `Orders.sale_date` | `STR_TO_DATE(sale_date, '%m-%d-%Y')` with `CASE` statements to handle variations like `'Oct 17 25'` or `'10 Sep 2025'`. |
| **Clean Numeric Cost** | `Products.cost` | `CAST(REPLACE(REPLACE(cost, 'USD ', ''), 'CAD ', '') AS DECIMAL(10,2))` |
| **Parse First Name** | `Customers.first_name` | `TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(customer_info, ';', 1), ' ', 1))` |
| **Parse Last Name** | `Customers.last_name` | `TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(customer_info, ';', 1), ' ', -1))` |
| **Parse Loyalty Status** | `Customers.loyalty_status` | `CASE WHEN customer_info LIKE '%Loyalty? Y%' THEN 'Y' ELSE 'N' END` |
| **Standardize Payments** | `Payments.payment_method` | `UPPER(CASE WHEN payment_method IN ('MC', 'Mastercard') THEN 'MASTERCARD' ELSE payment_method END)` |
| **Recursive Manager Link** | `Employees.manager_ref` | `UPDATE Employees e1 SET manager_ref = (SELECT e2.employee_ref FROM Employees e2 WHERE e1.manager_ref = e2.employee_ref)` |
| **Normalize Category** | `Products.category_id` | `INSERT INTO Categories (category_name) SELECT DISTINCT SUBSTRING_INDEX(category, ' /', 1) FROM Staging_Table` |
| **Normalize Subcategory** | `Products.sub_category_id` | `INSERT INTO Categories (category_name) SELECT DISTINCT SUBSTRING_INDEX(category, '/ ', -1) FROM Staging_Table WHERE category LIKE '%/%'` |

---

### **Data Cleaning Process**

#### **Overview**

The cleaning process was completed in three phases: AI-assisted pre-cleaning, CSV export and import, and SQL fine-tuning.

---

#### **Phase 1 — AI-Assisted Pre-Cleaning**

The raw source files (`Sales_Dump` and `Product_Supplier_Master`) contained too many inconsistencies to address manually row by row. AI tools were used to assist in bulk-identifying and resolving the most pervasive issues across both spreadsheets before any database import occurred. This included:

- **Standardizing date formats** — All date values across the `sale_date` column were reviewed and reformatted into a consistent `MM-DD-YYYY` pattern, resolving variations such as `Oct 17 25`, `October 5 25`, `10 Sep 2025`, and `31/10/2025`.
- **Splitting the `customer_info` field** — The concatenated customer strings (e.g., `Mason Rivera; Loyalty? Y`, `Grace Hall | Student | US`) were parsed to extract `first_name`, `last_name`, and `loyalty_status` into separate columns.
- **Stripping currency prefixes** — Numeric fields containing `USD`, `CAD`, or `$` prefixes were cleaned so that only numeric values remained, allowing them to be stored as `DECIMAL` types.
- **Normalizing case** — Payment methods (`VISA`, `visa`, `Debit`, `debit`) and SKU casing (`SKU-U-1003` vs. `sku-u-1003`) were standardized to a consistent format.
- **Resolving category separators** — Category values using mixed delimiters (`Tech / Student`, `Tech & Student`, `Lifestyle , Student`) were normalized to a single `Category / Subcategory` format.
- **Flagging duplicate and variant SKUs** — Rows with duplicate SKU descriptions or near-duplicate product entries (e.g., `Aurora Mechanical Keyboard` appearing under both `SKU-C-1002` and `sku-c-1002`) were identified and consolidated or flagged for removal.
- **Handling missing values** — Fields containing placeholder text such as `Taylor Green / email missing` were cleaned; the name was preserved and the email field was left NULL rather than populated with invalid text.

---

#### **Phase 2 — CSV Export and Import**

Once the bulk of the cleaning was completed, the data was exported from Excel into structured CSV files — one per entity — aligned to the final schema defined in the DDL. These CSVs were then imported into the corresponding MySQL tables using MySQL Workbench's table import wizard. Tables were loaded in dependency order to respect foreign key constraints:

1. `Categories`
2. `Vendors`
3. `Customers`
4. `Employees`
5. `Products`
6. `Orders`
7. `Order_Lines`
8. `Payments`

---

#### **Phase 3 — SQL Fine-Tuning**

After import, a final round of SQL statements was used to correct any remaining inconsistencies that survived the pre-cleaning phase or emerged during the import process. The table below documents each specific transformation applied at this stage.

---

### **Data Transformation**

| Objective | Target Attribute | SQL Transformation Logic / Function |
| :--- | :--- | :--- |
| **Currency Normalization** | `All Financials` | `CASE WHEN cost LIKE 'CAD%' THEN CAST(REGEXP_REPLACE(cost, '[^0-9.]', '') AS DECIMAL(10,2)) * 0.73 ELSE CAST(REGEXP_REPLACE(cost, '[^0-9.]', '') AS DECIMAL(10,2)) END` *(Assumes 0.73 exchange rate for CAD to USD)* |
| **Standardize Dates** | `Orders.sale_date` | `STR_TO_DATE(sale_date, '%m-%d-%Y')` with `CASE` statements to handle variations like `'Oct 17 25'` or `'10 Sep 2025'`. |
| **Clean Numeric Cost** | `Products.cost` | `CAST(REPLACE(REPLACE(cost, 'USD ', ''), 'CAD ', '') AS DECIMAL(10,2))` |
| **Parse First Name** | `Customers.first_name` | `TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(customer_info, ';', 1), ' ', 1))` |
| **Parse Last Name** | `Customers.last_name` | `TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(customer_info, ';', 1), ' ', -1))` |
| **Parse Loyalty Status** | `Customers.loyalty_status` | `CASE WHEN customer_info LIKE '%Loyalty? Y%' THEN 'Y' ELSE 'N' END` |
| **Standardize Payments** | `Payments.payment_method` | `UPPER(CASE WHEN payment_method IN ('MC', 'Mastercard') THEN 'MASTERCARD' ELSE payment_method END)` |
| **Recursive Manager Link** | `Employees.manager_ref` | `UPDATE Employees e1 SET manager_ref = (SELECT e2.employee_ref FROM Employees e2 WHERE e1.manager_ref = e2.employee_ref)` |
| **Normalize Category** | `Products.category_id` | `INSERT INTO Categories (category_name) SELECT DISTINCT SUBSTRING_INDEX(category, ' /', 1) FROM Staging_Table` |
| **Normalize Subcategory** | `Products.sub_category_id` | `INSERT INTO Categories (category_name) SELECT DISTINCT SUBSTRING_INDEX(category, '/ ', -1) FROM Staging_Table WHERE category LIKE '%/%'` |

---

### **SQL Implementation (DDL)**

The following SQL script generates the relational schema for Northline Outfitters. It includes all 8 entities, defines primary and foreign keys, and implements the recursive relationship for the employee management structure.

```sql
-- ==============================================
-- DATABASE SCHEMA - CREATE STATEMENTS
-- ==============================================

-- 1. Categories Table
-- Used for both primary categories and subcategories via sub_category_id in Products
CREATE TABLE Categories (
    category_id INT AUTO_INCREMENT PRIMARY KEY,
    category_name VARCHAR(100) NOT NULL
);

-- 2. Vendors Table
CREATE TABLE Vendors (
    vendor_id INT AUTO_INCREMENT PRIMARY KEY,
    vendor_name VARCHAR(100) NOT NULL,
    vendor_phone VARCHAR(25),
    vendor_rep VARCHAR(100)
);

-- 3. Customers Table
CREATE TABLE Customers (
    customer_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    customer_email VARCHAR(100),
    loyalty_status CHAR(1) DEFAULT 'N'
);

-- 4. Employees Table (Recursive Relationship)
-- manager_ref self-references employee_ref to implement the 1:M management hierarchy
CREATE TABLE Employees (
    employee_ref VARCHAR(50) PRIMARY KEY,
    manager_ref VARCHAR(50),
    CONSTRAINT fk_manager FOREIGN KEY (manager_ref)
        REFERENCES Employees(employee_ref) ON DELETE SET NULL
);

-- 5. Products Table
-- sub_category_id is optional (nullable) and references Categories for a secondary classification
CREATE TABLE Products (
    sku VARCHAR(50) PRIMARY KEY,
    alt_sku VARCHAR(50),
    product_description VARCHAR(255),
    category_id INT,
    vendor_id INT,
    sub_category_id INT,
    cost DECIMAL(10,2),
    list_price DECIMAL(10,2),
    reorder_level VARCHAR(20),
    discontinued VARCHAR(5),
    CONSTRAINT fk_category FOREIGN KEY (category_id)
        REFERENCES Categories(category_id),
    CONSTRAINT fk_sub_category FOREIGN KEY (sub_category_id)
        REFERENCES Categories(category_id),
    CONSTRAINT fk_vendor FOREIGN KEY (vendor_id)
        REFERENCES Vendors(vendor_id)
);

-- 6. Orders Table
CREATE TABLE Orders (
    order_id VARCHAR(50) PRIMARY KEY,
    sale_date DATE,
    customer_id INT,
    employee_ref VARCHAR(50),
    ship_country VARCHAR(50),
    ship_to VARCHAR(100),
    return_flag CHAR(1) DEFAULT 'N',
    CONSTRAINT fk_customer FOREIGN KEY (customer_id)
        REFERENCES Customers(customer_id),
    CONSTRAINT fk_employee FOREIGN KEY (employee_ref)
        REFERENCES Employees(employee_ref)
);

-- 7. Order_Lines Table
CREATE TABLE Order_Lines (
    line_id VARCHAR(50) PRIMARY KEY,
    order_id VARCHAR(50),
    sku VARCHAR(50),
    quantity INT,
    unit_price DECIMAL(10,2),
    discount VARCHAR(50),
    discount_type VARCHAR(20),
    tax VARCHAR(50),
    line_total DECIMAL(10,2),
    CONSTRAINT fk_order FOREIGN KEY (order_id)
        REFERENCES Orders(order_id),
    CONSTRAINT fk_sku FOREIGN KEY (sku)
        REFERENCES Products(sku)
);

-- 8. Payments Table
CREATE TABLE Payments (
    payment_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id VARCHAR(50),
    payment_method VARCHAR(50),
    amount_paid DECIMAL(10,2),
    CONSTRAINT fk_payment_order FOREIGN KEY (order_id)
        REFERENCES Orders(order_id)
);
```

---

```
### **SQL Queries**
**1.** 

Which products generated the highest total sales revenue, by country?
<img width="620" height="297" alt="image" src="https://github.com/user-attachments/assets/11a0571e-f3b5-4ccd-a8cb-540c3edf24ba" />

**2.**

Which employees handled the largest number of orders, and how do their results compare with other
employees under the same manager?
<img width="624" height="470" alt="image" src="https://github.com/user-attachments/assets/47cc974a-e6d5-4e2e-9992-370b72d9601a" />

**3.**

Which vendors supply products that appear in more than one category?
<img width="1072" height="614" alt="image" src="https://github.com/user-attachments/assets/6d15536a-8d3a-4de2-a944-1693f176d366" />

