## RC Model — Draft 1 (cleaned and expanded)

Purpose: clean up the draft business rules, normalize entity names, fix PK/FK issues, correct cardinalities, and suggest missing high-value entities (address, payments, shipments, etc.).

---

## Conventions used in this document
- PK: primary key
- FK: foreign key (target in parentheses)
- Cardinalities use standard notation: 0..1 (zero or one), 0..* (zero or many), 1 (exactly one), 1..* (one or many)

---

## Entities and suggested attributes

1) Customer
 - PK: Customer_ID
 - Customer_Name
 - Customer_Email
 - Customer_Phone
 - Customer_Customer_Type (Individual / Company)
 - Customer_Default_Billing_Address_ID (FK: Address.Address_ID)
 - Customer_Default_Shipping_Address_ID (FK: Address.Address_ID)
 - Customer_Customer_Revenue
 - Customer_Date_Added
 - Customer_Status (Active/Inactive)
 - Customer_Created_By, Customer_Created_Date, Customer_Updated_By, Customer_Updated_Date

2) Address
 - PK: Address_ID
 - FK: Customer_ID (nullable) — allows addresses shared/reused or linked to other entities
 - Address_Address_Type (Billing / Shipping / Other)
 - Address_Street, Address_City, Address_State, Address_Postal_Code, Address_Country
 - Address_Contact_Name, Address_Phone

3) Manufacturer (Supplier)
 - PK: Manufacturer_ID
 - Manufacturer_Manufacturer_Name
 - Manufacturer_Country
 - Manufacturer_Active_Flag
 - Manufacturer_Website
 - Manufacturer_Created_By, Manufacturer_Created_Date, Manufacturer_Updated_By, Manufacturer_Updated_Date

4) Product
 - PK: Product_ID (or SKU)
 - Product_Product_Name
 - Product_Product_Model
 - Product_Manufacturer_Product_Code
 - Product_Manufacturer_ID (FK: Manufacturer.Manufacturer_ID)
 - Product_Product_Category_ID (FK: ProductCategory.Product_Category_ID)
 - Product_Description
 - Product_Unit_Price (list price)
 - Product_Cost
 - Product_Quantity_On_Hand (if single-location) — otherwise track per location
 - Product_Min_Quantity / Product_Reorder_Level
 - Product_Product_Status (Active / Discontinued)
 - Product_Weight, Product_Dimensions
 - Product_Created_By, Product_Created_Date, Product_Updated_By, Product_Updated_Date

5) ProductCategory (optional but recommended)
 - PK: Product_Category_ID
 - ProductCategory_Category_Name, ProductCategory_Description

6) SalesOrder (optional — separate order from invoice)
 - PK: SalesOrder_ID
 - SalesOrder_Customer_ID (FK)
 - SalesOrder_Order_Date, SalesOrder_Requested_Ship_Date
 - SalesOrder_Salesperson_ID (FK: Employee)
 - SalesOrder_Status (New / Confirmed / Shipped / Invoiced / Cancelled)

7) Invoice
 - PK: Invoice_ID
 - Invoice_Customer_ID (FK)
 - Invoice_Invoice_Date, Invoice_Due_Date
 - Invoice_Invoice_Status (Draft / Issued / Paid / Cancelled)
 - Invoice_Subtotal, Invoice_Tax_Amount, Invoice_Shipping_Charge, Invoice_Discount_Amount, Invoice_Total_Amount
 - Invoice_Billing_Address_ID (FK: Address.Address_ID)
 - Invoice_Shipping_Address_ID (FK: Address.Address_ID)
 - Invoice_SalesOrder_ID (FK, nullable)
 - Invoice_Created_By, Invoice_Created_Date, Invoice_Updated_By, Invoice_Updated_Date

8) LineItem (invoice line)
 - PK: LineItem_ID (or composite key: Invoice_ID + Line_Number)
 - LineItem_Invoice_ID (FK)
 - LineItem_Product_ID (FK)
 - LineItem_Quantity
 - LineItem_Unit_Price
 - LineItem_Line_Discount, LineItem_Line_Tax, LineItem_Line_Total

9) PurchaseOrder
 - PK: PurchaseOrder_ID
 - PurchaseOrder_Manufacturer_ID (FK)
 - PurchaseOrder_PO_Date, PurchaseOrder_Expected_Delivery_Date
 - PurchaseOrder_PO_Total, PurchaseOrder_Status (Draft / Ordered / Received / Closed / Cancelled)
 - PurchaseOrder_Created_By (FK: Employee)

10) POLine (purchase order line)
 - PK: POLine_ID (or composite: PurchaseOrder_ID + Line_Number)
 - POLine_PurchaseOrder_ID (FK)
 - POLine_Product_ID (FK)
 - POLine_Quantity_Ordered
 - POLine_Quantity_Received
 - POLine_Unit_Cost
 - POLine_Line_Total

11) GoodsReceipt (receipt against a PO)
 - PK: GRN_ID
 - GoodsReceipt_PurchaseOrder_ID (FK)
 - GoodsReceipt_Received_Date, GoodsReceipt_Received_By
 - GoodsReceipt_GRN_Status
 - GoodsReceipt_Link_to_GRN_lines (received quantities per POLine)

12) Warehouse / Location
 - PK: Location_ID
 - Location_Name, Location_Address_ID (FK)
 - Location_Type (Warehouse / Store / Transit)

13) Inventory (per-location quantities)
 - PK: Inventory_ID (or composite Product_ID + Location_ID)
 - Inventory_Product_ID (FK)
 - Inventory_Location_ID (FK)
 - Inventory_Quantity_On_Hand
 - Inventory_Reorder_Level

14) Payment
 - PK: Payment_ID
 - Payment_Invoice_ID (FK)
 - Payment_Payment_Date, Payment_Amount, Payment_Payment_Method (Card / Bank / Cash), Payment_Reference
 - Payment_Payment_Status (Pending / Completed / Failed)

15) Shipment
 - PK: Shipment_ID
 - Shipment_Invoice_ID or Shipment_SalesOrder_ID (FK)
 - Shipment_Carrier, Shipment_Tracking_Number, Shipment_Ship_Date, Shipment_Ship_Cost
 - Shipment_Ship_From_Location_ID, Shipment_Ship_To_Address_ID

16) Employee (optional)
 - PK: Employee_ID
 - Employee_Name, Employee_Email, Employee_Role

17) SupplierContact (optional)
 - PK: Contact_ID
 - SupplierContact_Manufacturer_ID (FK)
 - SupplierContact_Contact_Name, SupplierContact_Phone, SupplierContact_Email, SupplierContact_Role

18) Return / CreditMemo (optional)
 - PK: CreditMemo_ID
 - CreditMemo_Invoice_ID (FK), CreditMemo_LineItem_references, CreditMemo_Amount, CreditMemo_Reason, CreditMemo_Status

---

## Corrected / canonical relationships (high level)

- Customer (0..*) — (1) Invoice
	- Each invoice must have exactly one customer; a customer may have zero or many invoices.

- Invoice (1) — (1..*) LineItem
	- An invoice must have one or more line items; each line item belongs to exactly one invoice.

- Product (0..*) — (1..*) LineItem
	- A product may appear in many line items; a line item references exactly one product.

- Manufacturer (0..*) — (0..*) Product
	- A manufacturer may supply many products; a product is supplied by one manufacturer (if known). You may allow Manufacturer_ID nullable for products of unknown source.

- PurchaseOrder (1) — (1..*) POLine
	- A PO must include one or more lines; each POLine belongs to exactly one PO.

- Product (0..*) — (0..*) POLine
	- Product may appear on many PO lines; a PO line references a product.

- PurchaseOrder (1) — (0..*) GoodsReceipt
	- A PO can have zero or more GRNs (partial receipts); each GRN references a PO.

- Warehouse/Location (1) — (0..*) Inventory
	- Inventory quantities are tracked per product per location.

- Invoice (0..1) — (0..1) Shipment (or Shipment may reference SalesOrder)

- Invoice (0..*) — (0..*) Payment
	- An invoice may have one or more payments (partial payments allowed).

Notes on cardinalities: choose 0..* vs 1..* depending on whether you allow orphaned records (e.g., a Product may be created before it appears on any invoice or PO).

---

## Modeling recommendations and small fixes applied
- Use singular entity names (Invoice, PurchaseOrder, LineItem, POLine).
- Avoid storing Product_ID or Product_Name as FK directly on Invoice — keep product references inside LineItem.
- Prefer surrogate PKs (Invoice_ID, LineItem_ID) but consider composite natural keys for line ordering (Invoice_ID + Line_Number) if you need ordered lines.
- Add audit fields to most tables (Created_By, Created_Date, Updated_By, Updated_Date).
- Normalize addresses into an `Address` table if customers or manufacturers can have multiple addresses.
- Track inventory per location using Inventory (Product_ID + Location_ID).

---

## Questions for you (to refine the model)
1. Do you separate Sales Orders from Invoices (i.e., create sales orders that later become invoices)?
2. Do customers have multiple addresses (billing vs shipping) or only one address per customer?
3. Do you need to track payments and partial payments, refunds, or payment reconciliation?
4. Do you need to track shipments, carriers, and tracking numbers (or is a simple Shipping_Charge on the invoice enough)?
5. Do you use multiple warehouses/locations for inventory tracking?
6. Is the `Manufacturer` table representing vendors/suppliers only, or do you want to distinguish suppliers vs manufacturers?
7. Who will use this system — do you need Employee / User tables for approvals and audit?

---

## Next steps (pick one)
- I can produce a cleaned `RC Model - Draft 1 Rule.md` (done) and optionally:
	- A) Generate a PlantUML or Mermaid source file to visualize the ERD.
	- B) Generate SQL DDL (Postgres or MySQL) for the core tables (Customer, Address, Product, Invoice, LineItem, PurchaseOrder, POLine).
	- C) Update your `Narinda_Tanvilai.erd` or create a new diagram file with the changes.

Please tell me which of the next steps you want, and answer any of the questions above that apply — I will iterate and then produce the diagram or DDL.

---

Edited on: 2025-10-30

```

