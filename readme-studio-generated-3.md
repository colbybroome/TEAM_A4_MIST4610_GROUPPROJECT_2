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

**Orders and Order_Lines**
The relationship between **Orders** and **Order_Lines** is **1:M**. One order can consist of multiple line items (different products or quantities), but each specific line item belongs to only one master order.

**Products and Order_Lines**
The relationship between **Products** and **Order_Lines** is **1:M**. A single product can appear on many different order lines across various customer transactions, but each line item refers to one specific SKU.

**Orders and Payments**
The relationship between **Orders** and **Payments** is **1:M**. While most orders have a single payment, an order can technically be fulfilled through multiple payment installments or methods, though each payment record is tied to one specific order.

### **Table: Customers**
| Attribute | Type | Key | Description |
| :--- | :--- | :--- | :--- |
| customer_id | INT | PK | Unique identifier for each customer. |
| customer_name | VARCHAR(100) | | Full name of the customer. |
| customer_email | VARCHAR(100) | | Contact email address. |
| loyalty_status | CHAR(1) | | Indicates if the customer is a loyalty member (Y/N). |
| customer_type | VARCHAR(50) | | Category of customer (e.g., Student, Standard). |

### **Table: Employees**
| Attribute | Type | Key | Description |
| :--- | :--- | :--- | :--- |
| employee_id | INT | PK | Unique identifier for each staff member. |
| employee_ref | VARCHAR(10) | | Internal company reference code (e.g., EMU-202). |
| employee_name | VARCHAR(100) | | Full name of the employee. |
| manager_id | INT | FK | References employee_id for the reporting manager. |

### **Table: Products**
| Attribute | Type | Key | Description |
| :--- | :--- | :--- | :--- |
| sku | VARCHAR(20) | PK | Unique internal stock keeping unit code. |
| alt_sku | VARCHAR(20) | | Secondary or legacy product code. |
| product_description | VARCHAR(255) | | Full name and details of the product. |
| category_id | INT | FK | Links to the product category classification. |
| vendor_id | INT | FK | Links to the supplier of the product. |
| cost | DECIMAL(10,2) | | Wholesale price paid to vendor. |
| list_price | DECIMAL(10,2) | | Standard retail selling price. |
| reorder_level | INT | | Minimum stock threshold for replenishment. |
| pack_size | VARCHAR(50) | | Unit of measure for ordering (e.g., Case of 6). |
| weight | VARCHAR(50) | | Physical weight of the item. |
| length | VARCHAR(50) | | Dimension or size measurement. |
| discontinued | CHAR(1) | | Status flag for inactive items (Y/N). |
| parent_sku | VARCHAR(20) | FK | Self-reference to link variants to a main product. |
| notes | TEXT | | General remarks or variant details. |

### **Table: Vendors**
| Attribute | Type | Key | Description |
| :--- | :--- | :--- | :--- |
| vendor_id | INT | PK | Unique identifier for the supplier. |
| vendor_name | VARCHAR(100) | | Legal name of the supplying company. |
| vendor_phone | VARCHAR(20) | | Primary contact phone number. |
| vendor_rep | VARCHAR(100) | | Primary point of contact at the vendor. |

### **Table: Categories**
| Attribute | Type | Key | Description |
| :--- | :--- | :--- | :--- |
| category_id | INT | PK | Unique identifier for the product group. |
| category_name | VARCHAR(50) | | Name of the group (e.g., Tech, Apparel). |

### **Table: Orders**
| Attribute | Type | Key | Description |
| :--- | :--- | :--- | :--- |
| order_id | VARCHAR(20) | PK | Unique transaction identifier from sales records. |
| sale_date | DATE | | Date the transaction occurred. |
| customer_id | INT | FK | Links the order to the purchasing customer. |
| employee_id | INT | FK | Links the order to the salesperson. |
| ship_country | VARCHAR(5) | | Destination country code (US/CA). |
| ship_to | VARCHAR(255) | | Specific shipping address or city/state info. |
| return_flag | CHAR(1) | | Indicates if the order was returned (Y/N). |
| order_notes | TEXT | | Comments regarding shipping or order status. |

### **Table: Order_Lines**
| Attribute | Type | Key | Description |
| :--- | :--- | :--- | :--- |
| line_id | VARCHAR(20) | PK | Unique identifier for each line in an order. |
| order_id | VARCHAR(20) | FK | Links to the main order header. |
| sku | VARCHAR(20) | FK | Links to the specific product purchased. |
| quantity | INT | | Number of units purchased. |
| unit_price | DECIMAL(10,2) | | Price per unit at time of sale. |
| discount | VARCHAR(20) | | Discount applied (percentage or code). |
| tax | VARCHAR(20) | | Tax amount or percentage applied. |
| line_total | DECIMAL(10,2) | | Final
