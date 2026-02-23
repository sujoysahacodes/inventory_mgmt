# Domain Model

## Overview
This document defines the core domain entities, their relationships, and business rules for the inventory management system. The domain model serves as the foundation for data architecture, API design, and business logic implementation.

## Core Entities

### Inventory Item
The central entity representing any trackable item in the inventory system.

**Attributes:**
- `itemId` (String): Unique identifier (SKU)
- `description` (String): Human-readable item description
- `category` (String): Item classification
- `manufacturer` (String): Item manufacturer/brand
- `unitOfMeasure` (String): Base unit (EA, LB, GAL, etc.)
- `serialized` (Boolean): Requires serial number tracking
- `lotControlled` (Boolean): Requires lot/batch tracking
- `expirationControlled` (Boolean): Has expiration dates
- `hazardous` (Boolean): Requires special handling
- `dimensions` (Object): Physical measurements (L×W×H)
- `weight` (Number): Item weight in base unit
- `value` (Number): Unit value for accounting
- `status` (Enum): Active, Discontinued, Obsolete
- `createdDate` (DateTime): Entity creation timestamp
- `modifiedDate` (DateTime): Last modification timestamp

**Business Rules:**
- SKU must be unique across the system
- Serialized items require individual tracking records
- Hazardous items must have safety data sheets
- Discontinued items cannot have new receipts

### Location
Represents physical or logical locations where inventory is stored.

**Attributes:**
- `locationId` (String): Unique location identifier
- `name` (String): Display name for the location
- `type` (Enum): Warehouse, Store, Transit, Virtual
- `parentLocationId` (String): Hierarchical parent reference
- `address` (Object): Physical address details
- `capacity` (Object): Storage capacity constraints
- `restrictions` (Array): Item type restrictions
- `climate` (Object): Environmental conditions
- `active` (Boolean): Location operational status
- `defaultReceiving` (Boolean): Default inbound location
- `defaultShipping` (Boolean): Default outbound location

**Business Rules:**
- Location hierarchy cannot create circular references
- Capacity constraints must be enforced for physical locations
- Climate restrictions must be validated for sensitive items
- One default receiving location per facility

### Stock Record
Represents the current inventory position for an item at a location.

**Attributes:**
- `stockId` (String): Unique stock record identifier
- `itemId` (String): Reference to inventory item
- `locationId` (String): Reference to storage location
- `quantityOnHand` (Number): Current available quantity
- `quantityReserved` (Number): Allocated but not picked quantity
- `quantityInTransit` (Number): Goods in movement
- `quantityBackordered` (Number): Committed but not fulfilled
- `averageCost` (Number): Weighted average unit cost
- `lastMovement` (DateTime): Most recent transaction
- `cycleCountDate` (DateTime): Last physical count date
- `frozen` (Boolean): Temporarily locked for adjustments

**Business Rules:**
- Available quantity = On Hand - Reserved
- Negative on-hand quantities trigger alerts
- Frozen stock cannot be allocated to orders
- Cost updates must maintain audit trail

### Stock Movement
Records all inventory transactions and provides complete audit trail.

**Attributes:**
- `movementId` (String): Unique transaction identifier
- `itemId` (String): Item being moved
- `fromLocationId` (String): Source location (null for receipts)
- `toLocationId` (String): Destination location (null for shipments)
- `movementType` (Enum): Receipt, Shipment, Transfer, Adjustment
- `quantity` (Number): Quantity moved (positive/negative)
- `unitCost` (Number): Unit cost at time of movement
- `reasonCode` (String): Movement reason classification
- `referenceDocument` (String): Source document reference
- `transactionDate` (DateTime): When movement occurred
- `userId` (String): User who initiated movement
- `batchNumber` (String): Processing batch for bulk operations
- `serialNumbers` (Array): Serial numbers for tracked items
- `lotNumber` (String): Lot/batch information
- `expirationDate` (DateTime): Expiration for perishables

**Business Rules:**
- All movements must balance (total in = total out)
- Serial numbers must be unique and tracked individually
- Expired items cannot be shipped (configurable grace period)
- Negative adjustments require approval workflow

### Purchase Order
Represents procurement requests to suppliers.

**Attributes:**
- `orderNumber` (String): Unique PO identifier
- `supplierId` (String): Vendor reference
- `orderDate` (DateTime): When order was created
- `requestedDate` (DateTime): Requested delivery date
- `status` (Enum): Draft, Sent, Acknowledged, Received, Closed
- `deliveryLocation` (String): Receiving location
- `terms` (Object): Payment and delivery terms
- `totalValue` (Number): Total order value
- `currency` (String): Order currency
- `approvalStatus` (Enum): Pending, Approved, Rejected
- `approvedBy` (String): Approving authority
- `approvedDate` (DateTime): Approval timestamp

### Purchase Order Line
Individual line items within purchase orders.

**Attributes:**
- `lineNumber` (Integer): Line sequence number
- `itemId` (String): Item being ordered
- `quantityOrdered` (Number): Ordered quantity
- `quantityReceived` (Number): Received quantity
- `unitPrice` (Number): Negotiated price per unit
- `deliveryDate` (DateTime): Expected delivery
- `lineStatus` (Enum): Open, Partial, Complete, Cancelled
- `receiptTolerance` (Object): Over/under receipt limits

### Sales Order
Customer orders for inventory items.

**Attributes:**
- `orderNumber` (String): Unique sales order identifier
- `customerId` (String): Customer reference
- `orderDate` (DateTime): Order creation date
- `requestedDate` (DateTime): Customer requested delivery
- `promisedDate` (DateTime): Promised delivery date
- `status` (Enum): New, Allocated, Picked, Shipped, Delivered
- `shipToAddress` (Object): Delivery address
- `priority` (Enum): Normal, High, Rush, Emergency
- `totalValue` (Number): Order total value
- `paymentStatus` (Enum): Pending, Authorized, Paid

### Sales Order Line  
Individual items within sales orders.

**Attributes:**
- `lineNumber` (Integer): Line sequence
- `itemId` (String): Item ordered
- `quantityOrdered` (Number): Requested quantity
- `quantityAllocated` (Number): Reserved quantity
- `quantityPicked` (Number): Physically picked
- `quantityShipped` (Number): Shipped quantity
- `unitPrice` (Number): Selling price per unit
- `lineStatus` (Enum): Open, Allocated, Picked, Shipped

## Aggregate Relationships

### Inventory Aggregate  
- **Root**: Inventory Item
- **Children**: Stock Records, Stock Movements
- **Invariants**: 
  - Stock levels must equal sum of movements
  - Reserved quantities cannot exceed on-hand
  - Serialized items require individual movement records

### Order Aggregate
- **Root**: Sales/Purchase Order
- **Children**: Order Lines, Allocations, Shipments
- **Invariants**:
  - Line totals must equal order total
  - Allocated quantities cannot exceed stock
  - Shipped quantities cannot exceed allocated

## Domain Events

### Inventory Events
- `StockLevelChanged`: When quantities are updated
- `StockReserved`: When inventory is allocated
- `StockReleased`: When reservations are freed
- `LowStockAlert`: When levels fall below thresholds
- `ExpirationWarning`: When items approach expiration

### Order Events
- `OrderCreated`: New order entered
- `OrderAllocated`: Inventory reserved for order
- `OrderPicked`: Items physically selected
- `OrderShipped`: Goods in transit to customer
- `OrderDelivered`: Customer receipt confirmed

### Movement Events
- `ItemReceived`: Goods received from supplier
- `ItemShipped`: Goods sent to customer
- `ItemTransferred`: Movement between locations
- `InventoryAdjusted`: Cycle count corrections

## Business Rules Engine

### Allocation Rules
1. **FIFO (First In, First Out)**: Default allocation strategy
2. **LIFO (Last In, First Out)**: For specific item types
3. **FEFO (First Expired, First Out)**: For perishable goods
4. **Location Priority**: Preferred picking locations
5. **Customer Priority**: VIP customer allocation preferences

### Replenishment Rules
1. **Reorder Point**: Min/max inventory levels
2. **Economic Order Quantity**: Optimal order sizes
3. **Seasonal Adjustments**: Demand pattern modifications
4. **Vendor Minimums**: Supplier MOQ requirements
5. **Safety Stock**: Buffer inventory calculations

### Validation Rules
1. **Negative Inventory**: Prevent overselling (configurable)
2. **Expired Items**: Block shipment of expired goods
3. **Hazmat Restrictions**: Special handling requirements
4. **Credit Limits**: Customer credit validation
5. **Approval Workflows**: High-value transaction approvals

## Data Quality Standards

### Master Data Governance
- **Item Master**: Single source of truth for item attributes
- **Location Master**: Standardized location hierarchies
- **Supplier Master**: Vendor information management
- **Customer Master**: Customer data consistency

### Data Validation Rules
- **Format Standards**: Consistent data formats and patterns
- **Completeness Checks**: Required field validation
- **Referential Integrity**: Valid foreign key relationships
- **Business Logic**: Domain-specific validation rules

### Data Cleansing Procedures
- **Duplicate Detection**: Automated duplicate identification
- **Data Standardization**: Format normalization processes  
- **Quality Scoring**: Data quality metrics and monitoring
- **Correction Workflows**: Standardized data fix procedures