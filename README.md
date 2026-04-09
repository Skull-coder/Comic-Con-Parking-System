# Event Parking System Database Design

This document outlines the database schema for the multi-zone event parking system. It explains the purpose of each table, its role in managing vehicle entry, spot allocation, and payments, as well as how the tables relate to one another.

---

## Database Tables

### `vehicle_type`
* **Why it exists:** To establish a fixed list of allowed vehicle sizes/types (e.g., Bike, Car, SUV).
* **How we use it:** Acts as a lookup table to standardize vehicle classification during entry and ensure data consistency.

### `vehicle`
* **Why it exists:** To uniquely identify every physical vehicle that visits the venue.
* **How we use it:** Stores the unique license plate (`vehicle_no`) and links to `vehicle_type`. Enables tracking of vehicles that return for multiple visits across different days of the event without duplicating data.

### `spot_category`
* **Why it exists:** To define the physical capability or size of a parking space (e.g., Bike space, SUV space, EV charger).
* **How we use it:** Used by the system's logic to match an incoming vehicle's type to a parking spot that can physically accommodate it.

### `reserved_category`
* **Why it exists:** To manage special access tiers (e.g., VIP, Staff, Exhibitor, Cosplayer).
* **How we use it:** Used to tag specific parking spots so they can be securely allocated only to visitors holding special access passes.

### `parking_zone`
* **Why it exists:** To geographically organize the massive convention venue into manageable areas.
* **How we use it:** Groups parking spots by `name` (e.g., "North Wing") and `level` (e.g., "Basement 2") to help visitors locate their vehicles and help staff manage sector capacity.

### `parking_spot`
* **Why it exists:** To map out every individual physical parking space available at the venue.
* **How we use it:** Brings together the zone, spot category, and any reserved restrictions. Queried by the system to check real-time capacity and assign a specific `id` to an incoming vehicle.

### `parking_sessions`
* **Why it exists:** This is the core operational table that records the actual event of a vehicle parking at the venue.
* **How we use it:** Generates the unique `ticket` ID upon entry, records the `entry_time` and `exit_time` to calculate the total duration of the stay, and uses the `is_active` boolean to quickly query exactly which vehicles are currently inside the venue.

### `payment`
* **Why it exists:** To securely separate financial transactions from the physical logistics of parking.
* **How we use it:** Linked directly to a completed `parking_session_id`. Records the final calculated `amount`, the exact `payment_time`, and the `payment_status` (e.g., pending, completed, failed).

---

## Database Relationships

* **`vehicle` & `vehicle_type` (Many-to-One):** Prevents data inconsistency by forcing every vehicle to reference a standardized master category.
* **`parking_sessions` & `vehicle` (Many-to-One):** Allows the system to track repeat visitors across the multi-day event without duplicating the vehicle's details.
* **`parking_sessions` & `parking_spot` (Many-to-One):** Enables the system to check if a physical spot is currently occupied or view its historical usage as cars come and go.
* **`parking_spot` & `spot_category` (Many-to-One):** Allows the system to quickly filter and assign available spots that match an incoming vehicle's physical requirements.
* **`parking_spot` & `parking_zone` (Many-to-One):** Groups spots geographically to easily display sector capacities and direct drivers to available areas.
* **`parking_spot` & `reserved_category` (Many-to-One):** Applies special access rules (like "VIP") to specific spots, while standard spots remain unregulated (nullable FK).
* **`payment` & `parking_sessions` (Many-to-One):** If a visitor fails their first payment (due to some condition), they need to try again, meaning one parking session can have multiple payment records attached to it.
