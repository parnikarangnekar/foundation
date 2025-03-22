# createOrderItemInventoryReservation

*   **Purpose:** This service reserves inventory for a specific order item within a shipment group at a designated facility in HotWax Commerce (HC).
      The brokering logic or pre-allocation location should be assigned before calling the reservation service. This service works only if a facility has already been assigned to the order item.
*   **Scope:** The service handles finished goods, bundle product and non-serialized inventory items.
  * Finished Goods
  * Non serialized Inventory
  * One InventoryItem per Facility per Product
  * statusId is not used
  * Do negative reservation if not enough in stock.
  * Do reservation for bundle product components

## Service Definition

*   **Service Name:** `createOrderItemInventoryReservation`
*   **Input Parameters:**
    *   `orderId` (required): The ID of the order.
    *   `orderItemSeqId` (required): The sequence ID of the order item.
    *   `quantity` (required): The quantity to reserve.

## Data Model

*   **Entity Relationships:**
    *   `OrderItem` has a one-to-one relationship with `OrderItemShipGroup` in HC.
    *   `OrderItem` has a one-to-many relationship with `OrderItemShipGrpInvRes`.
    *   `OrderItemShipGroup` has a one-to-many relationship with `OrderItemShipGrpInvRes`.
    *   `InventoryItem` has a one-to-many relationship with `InventoryItemDetail`.
    *   `OrderItemShipGrpInvRes` has a many-to-one relationship with `InventoryItem`.

## Service Logic

*   **Detailed Steps:**
    1. **Input Validation:**
    2. **Inventory Item Retrieval:**
        *   [Query](findOrCreateFacilityInventoryItem.md) the `InventoryItemId` for the `productId` and `facilityId`.
    3. **Reservation Creation:**
        *   Creating an `OrderItemShipGrpInvRes` record to link the reservation to the order item, shipment group, and inventory item.
        *   Create an `InventoryItemDetail` record for the reservation to track the change in inventory. ECA on `InventoryItemDetail` updates `InventoryItem` on `availableToPromise`.
    4. **Bundle product reservation**
        * Creating an `OrderItemShipGrpInvRes` record to link the reservation to the bundle prdouct, shipment group, and inventory item. 
        * Creating an `OrderItemShipGrpInvRes` record for all bundle product components, the shipment group, and the inventory item. The `orderItemSeqId` will be the same as the bundle product's order item, and the inventory item ID will be from the component product.

*   **HC-Specific Considerations:**
  *   The `OrderItem` entity in HC includes the `shipGroupSeqId` field, resulting in a one-to-one relationship between `OrderItem` and `OrderItemShipGroup`.
  *   In HC we have one InventoryItem per facility per product.
  *   This implementation focuses solely on non-serialized inventory, as that's the only type supported in HC.


## Scenario:

A customer places an order for 1 unit of product "P001" to be shipped from facility "F001".

### Initial State:

| Entity             | Field               | Value        |
|----------------------|----------------------|--------------|
| **OrderItem**       | `orderId`           | "O001"       |
|                    | `orderItemSeqId`    | "00001"      |
|                    | `productId`         | "P001"       |
|                    | `quantity`          | 1            |
|                    | `shipGroupSeqId`   | "00001"      |
| **OrderItemShipGroup** | `orderId`           | "O001"       |
|                    | `shipGroupSeqId`   | "00001"      |
|                    | `facilityId`        | "F001"       |
| **InventoryItem**    | `inventoryItemId`   | "I001"       |
|                    | `productId`         | "P001"       |
|                    | `facilityId`        | "F001"       |
|                    | `availableToPromiseTotal` | 10           |
|                    | `datetimeReceived`  | 2024-11-10   |


### Service Execution:

The `createOrderItemInventoryReservation` service is called with the following input parameters:

| Parameter         | Value        |
|------------------|--------------|
| `orderId`        | "O001"       |
| `orderItemSeqId` | "00001"      |
| `productId`      | "P001"       |
| `quantity`       | 1            |
| `facilityId`     | "F001"       |


### Final State:

| Entity                 | Field               | Value        |
|--------------------------|----------------------|--------------|
| **OrderItemShipGrpInvRes** | `orderId`           | "O001"       |
|                          | `orderItemSeqId`    | "00001"      |
|                          | `shipGroupSeqId`   | "00001"      |
|                          | `inventoryItemId`   | "I001"       |
|                          | `quantity`          | 1            |
|                          | `quantityNotAvailable` | 0            |
| **InventoryItem**        | `inventoryItemId`   | "I001"       |
|                          | `productId`         | "P001"       |
|                          | `facilityId`        | "F001"       |
|                          | `availableToPromiseTotal` | 9            |
|                          | `datetimeReceived`  | 2024-11-10   |
| **InventoryItemDetail**  | `inventoryItemId`   | "I001"       |
|                          | `orderId`           | "O001"       |
|                          | `orderItemSeqId`    | "00001"      |
|                          | `shipGroupSeqId`   | "00001"      |
|                          | `availableToPromiseDiff` | -1           |
