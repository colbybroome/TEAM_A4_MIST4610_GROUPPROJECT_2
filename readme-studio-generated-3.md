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

### **Conceptual Data Model**

<img width="1054" height="718" alt="gp2final" src="https://github.com/user-attachments/assets/00400dae-c11a-4c5f-826e-9fdd5311e152" />


### **Entity Relationships**

**Customers and Orders**
The relationship between **Customers** and **Orders** is **1:M**. A single customer can place multiple orders over time, but each specific order record is linked back to only one unique customer.

**Employees and Orders**
The relationship between **Employees** and **Orders** is **1:M**. One employee can process or be responsible for many different orders, while each order is handled by exactly one primary employee.

**Employees and Employees (Management)**
The relationship between **Employees** and themselves is a **1:M Recursive** relationship. An individual employee can act as a manager for multiple other employees, but each subordinate employee reports to only one manager.

**Vendors and Products**
The relationship between **Vendors** and **Products** is **1:M**. One vendor can supply a variety of different products to Northline Outfitters, but each specific product is sourced from one primary vendor.

**Categories and Products**
The relationship between **Categories** and **Products** is **1:M**. A single category (e.g., "Tech") can contain many different products, while each product is assigned to one primary category for organizational purposes.

**Categories and Products**
The relationship between **Categories** and **Products** is **1:M** where the cardinality is set to zero on the products side to represent that a product may not have a second category. 

**Orders and Order_Lines**
The relationship between **Orders** and **Order_Lines** is **1:M**. One order can consist of multiple line items (different products or quantities), but each specific line item belongs to only one master order.

**Products and Order_Lines**
The relationship between **Products** and **Order_Lines** is **1:M**. A single product can appear on many different order lines across various customer transactions, but each line item refers to one specific SKU.

**Orders and Payments**
The relationship between **Orders** and **Payments** is **1:M**. While most orders have a single payment, an order can technically be fulfilled through multiple payment installments or methods, though each payment record is tied to one specific order.

### **Entity Breakdown**

### **Entity Relationships**

The database is structured around a central **Orders** hub, which captures the transaction logic by linking three primary business drivers: **Customers**, **Employees** (who facilitate sales and report to a specific **Manager** hierarchy), and **Payments**. To handle the specifics of what was purchased, the **Orders** entity connects to **OrderLines**, a bridge table that breaks down individual transaction details such as quantity, price, and discounts. These lines point directly to the **Products** entity, which serves as the master record for inventory. Each product is detailed with its own operational attributes, such as dimensions and reorder levels, and contains the products **Category**, and its possible but optional **Subcategory**. Finally, the **Vendor** entity stores the essential contact and representative information for the vendors that provide these products, ensuring the entire supply chain—from procurement to the final customer receipt—is fully integrated.

### **Data Quality Assessment**

Below is a detailed assessment of the data quality issues found in the `Sales_Dump` and `Product_Supplier_Master` files. These issues prevent the data from being used in a relational database without prior cleaning and transformation.

| Data Category | Specific Issues Identified | Examples from Data |
| :--- | :--- | :--- |
| **Date Formats** | Significant inconsistency in date recording, including U.S. styles, international styles, and written-out months. | `10-11-2025`, `Oct 17 25`, `October 5 25`, `10 Sep 2025` |
| **Customer Info** | Attributes are concatenated into single strings using multiple types of delimiters (`;`, `\|`) and contain non-atomic data. | `Mason Rivera; Loyalty? Y`, `Grace Hall \| Student \| US` |
| **Currency & Numeric** | Numeric fields contain string prefixes (USD/CAD) or currency symbols, which prevent mathematical calculations. | `USD 6.25`, `CAD 31.40`, `$19.43` |
| **Case Sensitivity** | Identical categorical values are recorded with inconsistent capitalization. | `VISA` vs. `visa`, `Debit` vs. `debit` |
| **Category Naming** | Product categories use different separators and inconsistent naming conventions for sub-categories. | `Tech / Student`, `Tech & Student`, `Audio / Student` |
| **Missing Values** | Key contact information is missing or replaced with placeholder text in the middle of data fields. | `liam_patel@school.edu` (Missing emails for others), `Taylor Green / email missing` |
| **Inconsistent Units** | Physical measurements (weight/length) use mixed units (imperial vs. metric) and varying abbreviations. | `8 ounces`, `499 g`, `10.2 in`, `25.4 cm` |
| **Redundancy** | Partial duplicates exist where the same product description appears with slightly different SKU casing or pricing. | `SKU-U-1003` vs. `sku-u-1003`, `Aurora Mechanical Keyboard` (Duplicate desc) |

### **Step 4: Data Transformation Logic (SQL)**

Now that the specific issues have been identified, the following plan outlines the logic required to transform the "dirty" source data into a normalized format.
| Objective | Target Attribute | SQL Transformation Logic / Function |
| :--- | :--- | :--- |
| **Currency Normalization** | `All Financials` | `CASE WHEN cost LIKE 'CAD%' THEN CAST(REGEXP_REPLACE(cost, '[^0-9.]', '') AS DECIMAL(10,2)) * 0.73 ELSE CAST(REGEXP_REPLACE(cost, '[^0-9.]', '') AS DECIMAL(10,2)) END` *(Assumes 0.73 exchange rate for CAD to USD)* |
| **Standardize Dates** | `Orders.sale_date` | `STR_TO_DATE(sale_date, '%m-%d-%Y')` with `CASE` statements to handle variations like 'Oct 17 25' or '10 Sep 2025'. |
| **Clean Numeric Cost** | `Products.cost` | `CAST(REPLACE(REPLACE(cost, 'USD ', ''), 'CAD ', '') AS DECIMAL(10,2))` |
| **Atomic Customer Name** | `Customers.customer_name` | `SUBSTRING_INDEX(customer_info, ';', 1)` or `SUBSTRING_INDEX(customer_info, ' |', 1)` |
| **Parse Loyalty Status**| `Customers.loyalty_status`| `CASE WHEN customer_info LIKE '%Loyalty? Y%' THEN 'Y' ELSE 'N' END` |
| **Standardize Payments**| `Payments.payment_method`| `UPPER(CASE WHEN payment_method IN ('MC', 'Mastercard') THEN 'MASTERCARD' ELSE payment_method END)` |
| **Recursive Manager Link**| `Employees.manager_id` | `UPDATE Employees e1 SET manager_id = (SELECT e2.employee_id FROM Employees e2 WHERE e1.manager_ref = e2.employee_ref)` |
| **Normalize Category** | `Products.category_id` | `INSERT INTO Categories (category_name) SELECT DISTINCT SUBSTRING_INDEX(category, ' /', 1) FROM Staging_Table` |

### **Step 5: SQL Implementation (DDL)**

The following SQL script generates the relational schema for Northline Outfitters. It includes all 8 entities, defines primary and foreign keys, and implements the recursive relationship for the employee management structure.

```sql
-- 1. Categories Table
CREATE TABLE Categories (
    category_id INT AUTO_INCREMENT PRIMARY KEY,
    category_name VARCHAR(50) NOT NULL
);

-- 2. Vendors Table
CREATE TABLE Vendors (
    vendor_id INT AUTO_INCREMENT PRIMARY KEY,
    vendor_name VARCHAR(100) NOT NULL,
    vendor_phone VARCHAR(20),
    vendor_rep VARCHAR(100)
);

-- 3. Customers Table
CREATE TABLE Customers (
    customer_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    loyalty_status CHAR(1) DEFAULT 'N',
    customer_type VARCHAR(50)
);

-- 4. Employees Table (Includes Recursive Relationship)
CREATE TABLE Employees (
    employee_id INT AUTO_INCREMENT PRIMARY KEY,
    employee_ref VARCHAR(10) UNIQUE NOT NULL,
    employee_name VARCHAR(100),
    manager_id INT,
    CONSTRAINT fk_manager FOREIGN KEY (manager_id) 
        REFERENCES Employees(employee_id) ON DELETE SET NULL
);

-- 5. Products Table (Includes Self-Reference for Variants)
CREATE TABLE Products (
    sku VARCHAR(20) PRIMARY KEY,
    alt_sku VARCHAR(20),
    product_description VARCHAR(255),
    category_id INT,
    vendor_id INT,
    cost DECIMAL(10,2),
    list_price DECIMAL(10,2),
    reorder_level INT,
    pack_size VARCHAR(50),
    weight VARCHAR(50),
    length VARCHAR(50),
    discontinued CHAR(1) DEFAULT 'N',
    parent_sku VARCHAR(20),
    notes TEXT,
    CONSTRAINT fk_category FOREIGN KEY (category_id) REFERENCES Categories(category_id),
    CONSTRAINT fk_vendor FOREIGN KEY (vendor_id) REFERENCES Vendors(vendor_id),
    CONSTRAINT fk_parent_sku FOREIGN KEY (parent_sku) REFERENCES Products(sku)
);

-- 6. Orders Table
CREATE TABLE Orders (
    order_id VARCHAR(20) PRIMARY KEY,
    sale_date DATE,
    customer_id INT,
    employee_id INT,
    ship_country VARCHAR(5),
    ship_to VARCHAR(255),
    return_flag CHAR(1) DEFAULT 'N',
    order_notes TEXT,
    CONSTRAINT fk_customer FOREIGN KEY (customer_id) REFERENCES Customers(customer_id),
    CONSTRAINT fk_employee FOREIGN KEY (employee_id) REFERENCES Employees(employee_id)
);

-- 7. Order_Lines Table
CREATE TABLE Order_Lines (
    line_id VARCHAR(20) PRIMARY KEY,
    order_id VARCHAR(20),
    sku VARCHAR(20),
    quantity INT,
    unit_price DECIMAL(10,2),
    discount VARCHAR(20),
    tax VARCHAR(20),
    line_total DECIMAL(10,2),
    size_or_weight VARCHAR(50),
    CONSTRAINT fk_order FOREIGN KEY (order_id) REFERENCES Orders(order_id),
    CONSTRAINT fk_sku FOREIGN KEY (sku) REFERENCES Products(sku)
);

-- 8. Payments Table
CREATE TABLE Payments (
    payment_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id VARCHAR(20),
    payment_method VARCHAR(50),
    amount_paid DECIMAL(10,2),
    CONSTRAINT fk_payment_order FOREIGN KEY (order_id) REFERENCES Orders(order_id)
);
