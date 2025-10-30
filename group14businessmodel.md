# RC_Models Business Model - Simplified Documentation

## **ENTITIES AND ATTRIBUTES**

### **1. CUSTOMER**
| Attribute | Type | Key | Constraints |
|-----------|------|-----|-------------|
| CUSTOMER_ID | INT | PK | NOT NULL, AUTO_INCREMENT |
| CUSTOMER_NAME | VARCHAR(100) | | NOT NULL |
| CUSTOMER_EMAIL | VARCHAR(100) | | UNIQUE, NOT NULL |
| CUSTOMER_ADDRESS | VARCHAR(255) | | NOT NULL |
| CUSTOMER_PHONE | VARCHAR(20) | | NULL |
| CUSTOMER_SOURCE | VARCHAR(50) | | NOT NULL, DEFAULT 'Direct' |
| DATE_REGISTERED | DATE | | NOT NULL |

**Purpose**: Stores customer information including those from FineScale Modeler magazine list.

---

### **2. ORDER**
| Attribute | Type | Key | Constraints |
|-----------|------|-----|-------------|
| ORDER_ID | INT | PK | NOT NULL, AUTO_INCREMENT |
| ORDER_DATE | DATETIME | | NOT NULL |
| ORDER_STATUS | ENUM | | NOT NULL, DEFAULT 'Pending' |
| CUSTOMER_ID | INT | FK | NOT NULL → CUSTOMER |

**Purpose**: Represents a transaction placed by a customer on the website.

---

### **3. ORDER_LINE**
| Attribute | Type | Key | Constraints |
|-----------|------|-----|-------------|
| ORDER_ID | INT | PK, FK | NOT NULL → ORDER |
| PRODUCT_ID | INT | PK, FK | NOT NULL → PRODUCT |
| QUANTITY | INT | | NOT NULL, CHECK > 0 |
| PRICE_PER_UNIT | DECIMAL(10,2) | | NOT NULL, CHECK > 0 |
| IS_BACKORDERED | BOOLEAN | | DEFAULT FALSE |
| BACKORDER_DATE | DATE | | NULL |

**Purpose**: Links orders to products (junction table). Composite PK prevents duplicate products per order.

---

### **4. INVOICE**
| Attribute | Type | Key | Constraints |
|-----------|------|-----|-------------|
| INVOICE_ID | INT | PK | NOT NULL, AUTO_INCREMENT |
| ORDER_ID | INT | FK | UNIQUE, NOT NULL → ORDER |
| INV_DATE | DATETIME | | NOT NULL |
| INV_TOTAL | DECIMAL(10,2) | | NOT NULL, CHECK >= 0 |
| SHIPPING_CHARGES | DECIMAL(10,2) | | NOT NULL, CHECK >= 0 |
| TAX_AMOUNT | DECIMAL(10,2) | | DEFAULT 0 |
| PAYMENT_STATUS | ENUM | | DEFAULT 'Pending' |
| PAYMENT_DATE | DATETIME | | NULL |

**Purpose**: Bill for an order. UNIQUE on ORDER_ID enforces 1:1 relationship.

---

### **5. PRODUCT**
| Attribute | Type | Key | Constraints |
|-----------|------|-----|-------------|
| PRODUCT_ID | INT | PK | NOT NULL, AUTO_INCREMENT |
| PRODUCT_NAME | VARCHAR(200) | | NOT NULL |
| PRODUCT_CATEGORY | ENUM | | NOT NULL ('Aircraft','Ship','Car','Decal') |
| PRODUCT_SCALE | ENUM | | NOT NULL ('1/144','1/72','1/48','1/32','1/24') |
| PRODUCT_PRICE | DECIMAL(10,2) | | NOT NULL, CHECK > 0 |
| PRODUCT_MIN_QTY | INT | | NOT NULL, CHECK >= 0 |
| PRODUCT_STOCK_LEVEL | INT | | DEFAULT 0, CHECK >= 0 |
| PRODUCT_PROMO | VARCHAR(255) | | NULL |
| IS_ACTIVE | BOOLEAN | | DEFAULT TRUE |
| MFG_ID | INT | FK | NOT NULL → MANUFACTURER |

**Purpose**: Items sold (models/decals). IS_ACTIVE = FALSE after 4 weeks with no sales.

---

### **6. MANUFACTURER**
| Attribute | Type | Key | Constraints |
|-----------|------|-----|-------------|
| MFG_ID | INT | PK | NOT NULL, AUTO_INCREMENT |
| MFG_NAME | VARCHAR(100) | | UNIQUE, NOT NULL |
| MFG_WEBSITE | VARCHAR(255) | | NULL |
| MFG_CONTACT_INFO | VARCHAR(100) | | NULL |
| IS_ACTIVE | BOOLEAN | | DEFAULT TRUE |

**Purpose**: Product suppliers (Tamiya, Revell, etc.).

---

## **RELATIONSHIPS**

### **1. CUSTOMER places ORDER (1:M)**
- **Customer side**: 1 customer → 0 or many orders
- **Order side**: 1 order → exactly 1 customer

**Notation**: `CUSTOMER ||----o< ORDER`

**Why?** Customers from magazine list may not have ordered yet (0). Each order belongs to one customer only.

---

### **2. ORDER contains ORDER_LINE (1:M)**
- **Order side**: 1 order → at least 1 order line
- **Order_Line side**: 1 order line → exactly 1 order

**Notation**: `ORDER ||----|| ORDER_LINE`

**Why?** Can't have empty orders. Must have at least one product.

---

### **3. PRODUCT appears in ORDER_LINE (1:M)**
- **Product side**: 1 product → 0 or many order lines
- **Order_Line side**: 1 order line → exactly 1 product

**Notation**: `PRODUCT ||----o< ORDER_LINE`

**Why?** New products may not be ordered yet (0). Each line references one product.

---

### **4. ORDER generates INVOICE (1:1)**
- **Order side**: 1 order → exactly 1 invoice
- **Invoice side**: 1 invoice → exactly 1 order

**Notation**: `ORDER ||----|| INVOICE`

**Why?** Every completed order generates one invoice. No sharing.

---

### **5. MANUFACTURER supplies PRODUCT (1:M)**
- **Manufacturer side**: 1 manufacturer → 0 or many products
- **Product side**: 1 product → exactly 1 manufacturer

**Notation**: `MANUFACTURER ||----o< PRODUCT`

**Why?** Some manufacturers added but not supplying yet (0). Each product from one manufacturer.

---

## **KEY CONSTRAINTS**

### **Business Logic Constraints**

1. **Invoice Total Must Equal Line Items + Shipping + Tax**
   ```
   INV_TOTAL = SUM(QUANTITY × PRICE_PER_UNIT) + SHIPPING_CHARGES + TAX_AMOUNT
   ```

2. **Price Lock at Order Time**
   - ORDER_LINE.PRICE_PER_UNIT captured when order placed
   - Cannot be changed even if PRODUCT.PRODUCT_PRICE changes later

3. **Auto-Reorder When Low Stock**
   ```
   IF PRODUCT_STOCK_LEVEL < PRODUCT_MIN_QTY
   THEN trigger automatic purchase order to manufacturer
   ```

4. **Scrap Unsold Products (4 Weeks)**
   ```
   IF no sales in 28 days THEN SET IS_ACTIVE = FALSE
   ```

5. **Backorder Charging Rule**
   - Backordered items (IS_BACKORDERED = TRUE) not charged until shipped
   - BACKORDER_DATE set when item ships

6. **Stock Reduction**
   - When order placed: PRODUCT_STOCK_LEVEL -= QUANTITY (if not backordered)
   - Backorders don't reduce stock until shipped

7. **Order Must Have Products**
   - At least 1 ORDER_LINE required per ORDER
   - Validated before order submission

8. **Composite Key Prevents Duplicates**
   - (ORDER_ID, PRODUCT_ID) ensures same product appears only once per order

---

### **Referential Integrity**

| Child Table | Foreign Key | Parent | Delete Rule | Update Rule |
|-------------|-------------|--------|-------------|-------------|
| ORDER | CUSTOMER_ID | CUSTOMER | RESTRICT | CASCADE |
| ORDER_LINE | ORDER_ID | ORDER | CASCADE | CASCADE |
| ORDER_LINE | PRODUCT_ID | PRODUCT | RESTRICT | CASCADE |
| INVOICE | ORDER_ID | ORDER | RESTRICT | CASCADE |
| PRODUCT | MFG_ID | MANUFACTURER | RESTRICT | CASCADE |

**Delete Rules**:
- **RESTRICT**: Can't delete if child records exist
- **CASCADE**: Auto-delete children (ORDER_LINE when ORDER deleted)

---

### **Data Validation**

- **UNIQUE**: CUSTOMER_EMAIL, INVOICE.ORDER_ID, MFG_NAME
- **CHECK > 0**: QUANTITY, PRICE_PER_UNIT, PRODUCT_PRICE
- **CHECK >= 0**: INV_TOTAL, SHIPPING_CHARGES, TAX_AMOUNT, PRODUCT_STOCK_LEVEL
- **ENUMS**: ORDER_STATUS, PAYMENT_STATUS, PRODUCT_CATEGORY, PRODUCT_SCALE
- **NOT NULL**: All PKs, FKs, critical business fields

---

## **ER DIAGRAM**---

## **Quick Reference Summary**

**6 Entities**: CUSTOMER, ORDER, ORDER_LINE, INVOICE, PRODUCT, MANUFACTURER

**5 Relationships**:
1. CUSTOMER (1) → (0..M) ORDER
2. ORDER (1) → (1..M) ORDER_LINE  
3. PRODUCT (1) → (0..M) ORDER_LINE
4. ORDER (1) → (1) INVOICE
5. MANUFACTURER (1) → (0..M) PRODUCT

**Top 5 Constraints**:
1. Price frozen at order time
2. Invoice total = lines + shipping + tax
3. Auto-reorder at min quantity
4. Scrap products after 4 weeks unsold
5. Backorders charged only when shipped