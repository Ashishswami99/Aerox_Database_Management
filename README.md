# ✈️ AeroxDB — Airflight Database Management System

> A comprehensive relational database system designed to manage end-to-end airline operations — from aircraft fleets and crew assignments to passenger bookings, real-time trip tracking, customer reviews, and revenue analytics.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Database Schema](#database-schema)
- [Entity Relationship Summary](#entity-relationship-summary)
- [Tables & Key Fields](#tables--key-fields)
- [Sample Data](#sample-data)
- [Setup & Installation](#setup--installation)
- [Use Cases & Analytics](#use-cases--analytics)
- [Technologies Used](#technologies-used)

---

## Overview

**AeroxDB** (`aeroxDB`) is a fully normalised MySQL database built to power an airline's core operations. It addresses the following domains:

| Domain | Coverage |
|---|---|
| 🛩️ Fleet Management | Aircraft models, capacity, fuel specs, servicing schedules |
| 🌍 Airport Network | Global airports with timezone and status tracking |
| 👤 Passenger Registry | Full passenger profiles with medical and emergency contacts |
| 👨‍✈️ Crew Management | Captains, First Officers, and Cabin Crew with license compliance |
| 🛫 Flight Scheduling | Flight codes, types, scope, and route configurations |
| 🗓️ Trip Operations | Scheduled vs actual departure/arrival, delays, and gate info |
| 🎟️ Booking System | Seat assignment, travel class, meal preferences, payment tracking |
| ⭐ Customer Reviews | Multi-dimensional ratings with airline response capability |
| 💰 Revenue Tracking | Fare breakdown, operating costs, profit, and multi-currency support |

---

## Database Schema

```
aeroxDB
│
├── aircraft          ← Fleet registry
├── airport           ← Global airport network
├── passenger         ← Passenger profiles
├── crew_info         ← Crew & licensing
│
├── flight            ── refs: crew_info, aircraft, airport
├── trip_details      ── refs: flight, airport (origin + destination)
├── booking_info      ── refs: trip_details, passenger
│
├── review            ── refs: booking_info
└── revenue_tracking  ── refs: booking_info
```

---

## Entity Relationship Summary

```
aircraft ──────────────────┐
airport (base) ────────────┤──► flight ──► trip_details ──► booking_info ──► review
crew_info ─────────────────┘                                      │
                                                                   └──► revenue_tracking
airport (origin/dest) ─────────────────────────► trip_details

passenger ─────────────────────────────────────────────────► booking_info
```

---

## Tables & Key Fields

### 1. `aircraft`
Stores the airline's full fleet inventory with maintenance scheduling.

| Column | Type | Description |
|---|---|---|
| `aircraft_id` | INT (PK) | Auto-incremented identifier |
| `model_code` | VARCHAR(10) | Unique model code (e.g. `B737-1`) |
| `model_name` | VARCHAR(50) | Full aircraft name |
| `manufacturer` | VARCHAR(50) | Boeing / Airbus / Embraer etc. |
| `aircraft_capacity` | INT | Total seat count |
| `fuel_capacity` | DECIMAL | Fuel tank capacity in litres |
| `aircraft_status` | ENUM | `active`, `maintenance`, `retired`, `leased_out`, `scrapped` |
| `last_servicing_dt` | DATE | Last maintenance date |
| `next_servicing_dt` | DATE | Scheduled next service date |

---

### 2. `airport`
Global airport directory with timezone support.

| Column | Type | Description |
|---|---|---|
| `airport_id` | INT (PK) | Auto-incremented identifier |
| `airport_name` | VARCHAR(50) | Full airport name |
| `city` / `country` | VARCHAR | Location details |
| `timezone` | VARCHAR(50) | IANA timezone string |
| `is_active` | BOOLEAN | Operational status |

---

### 3. `passenger`
Comprehensive passenger profiles for booking and compliance.

| Column | Type | Description |
|---|---|---|
| `passenger_id` | INT (PK) | Auto-incremented identifier |
| `passport_number` | VARCHAR(20) | Unique passport number |
| `passport_expiry_date` | DATE | For travel document validation |
| `nationality` | VARCHAR(50) | Nationality for immigration |
| `medical_conditions` | TEXT | Flags for special handling |
| `emergency_contact_name/phone` | VARCHAR | Next-of-kin details |
| `is_active` | BOOLEAN | Account status |

---

### 4. `crew_info`
Crew roster with licensing and safety compliance tracking.

| Column | Type | Description |
|---|---|---|
| `crew_id` | INT (PK) | Auto-incremented identifier |
| `staff_code` | VARCHAR(10) | Unique staff identifier |
| `role` | ENUM | `Captain`, `First Officer`, `Lead Cabin Crew`, `Flight Attendant` |
| `license_number` | VARCHAR(20) | Aviation license reference |
| `license_expiry_date` | DATE | Compliance deadline |
| `years_of_experience` | INT | Validated ≥ 0 |
| `last_safety_check` | DATETIME | Most recent safety evaluation |
| `next_safety_check_due` | DATE | Upcoming compliance check |

---

### 5. `flight`
Master flight definitions — reusable route configurations.

| Column | Type | Description |
|---|---|---|
| `flight_id` | INT (PK) | Auto-incremented identifier |
| `flight_code` | VARCHAR(10) | Unique IATA-style code |
| `crew_id` | INT (FK) | Assigned crew from `crew_info` |
| `aircraft_id` | INT (FK) | Assigned aircraft from `aircraft` |
| `base_airport_id` | INT (FK) | Home airport from `airport` |
| `flight_type` | ENUM | `Passenger`, `Cargo`, `Charter`, `Ferry` |
| `flight_scope` | ENUM | `Domestic`, `Regional`, `International`, `Intercontinental` |
| `standard_departure_time` | TIME | Scheduled departure time |
| `total_distance_km` | DECIMAL | Route distance |

---

### 6. `trip_details`
Individual flight instances with real-time operational data.

| Column | Type | Description |
|---|---|---|
| `trip_id` | INT (PK) | Auto-incremented identifier |
| `flight_id` | INT (FK) | References `flight` |
| `origin_airport_id` | INT (FK) | Departure airport |
| `destination_airport_id` | INT (FK) | Arrival airport |
| `scheduled_departure` | DATETIME | Planned departure |
| `actual_departure` | DATETIME | Actual gate departure |
| `trip_status` | ENUM | `Scheduled`, `Boarding`, `Departed`, `In Air`, `Arrived`, `Delayed`, `Cancelled`, `Diverted` |
| `delay_minutes` | INT | Minutes late |
| `delay_reason` | VARCHAR | Cause of delay |
| `gate_departure` / `terminal_departure` | VARCHAR | Boarding gate and terminal |

---

### 7. `booking_info`
Passenger reservations with full seat, class, and payment details.

| Column | Type | Description |
|---|---|---|
| `booking_id` | BIGINT (PK) | Auto-incremented identifier |
| `booking_reference` | VARCHAR(6) | Unique 6-character PNR |
| `trip_id` | INT (FK) | References `trip_details` |
| `passenger_id` | INT (FK) | References `passenger` |
| `travel_class` | ENUM | `Economy`, `Premium Economy`, `Business`, `First` |
| `seat_number` | VARCHAR(5) | Assigned seat |
| `fare` | DECIMAL | Ticket price |
| `extra_baggage_kg` | INT | Additional baggage allowance |
| `meal_preference` | ENUM | `Standard`, `Vegetarian`, `Vegan`, `Gluten-Free`, `Non-Veg` |
| `special_assistance` | ENUM | `None`, `Wheelchair`, `Infant`, `Elderly`, `Medical` |
| `payment_status` | ENUM | `Pending`, `Paid`, `Failed`, `Refunded` |
| `payment_method` | ENUM | Credit Card, UPI, Wallet, etc. |
| `booking_channel` | ENUM | `Website`, `Mobile App`, `Agent`, `API` |

---

### 8. `review`
Post-flight passenger feedback with airline response workflow.

| Column | Type | Description |
|---|---|---|
| `review_id` | INT (PK) | Auto-incremented identifier |
| `booking_id` | BIGINT (FK, UNIQUE) | One review per booking |
| `rating` | INT | Overall rating (1–5) |
| `cleanliness_rating` | INT | Cabin cleanliness (1–5) |
| `service_rating` | INT | Crew service (1–5) |
| `entertainment_rating` | INT | IFE quality (1–5) |
| `food_rating` | INT | Catering quality (1–5) |
| `is_verified` | BOOLEAN | Confirmed traveller flag |
| `is_public` | BOOLEAN | Visibility flag |
| `response_from_airline` | TEXT | Airline's reply |
| `response_date` | TIMESTAMP | When airline responded |

---

### 9. `revenue_tracking`
Financial records per booking for profit and cost analysis.

| Column | Type | Description |
|---|---|---|
| `revenue_id` | INT (PK) | Auto-incremented identifier |
| `booking_id` | BIGINT (FK) | References `booking_info` |
| `base_fare` | DECIMAL | Core ticket price |
| `tax_amount` | DECIMAL | Applicable taxes |
| `ancillary_fees` | DECIMAL | Baggage, upgrades, extras |
| `operating_cost` | DECIMAL | Estimated cost of service |
| `profit` | DECIMAL | Net profit (revenue − cost) |
| `revenue` | DECIMAL | Total collected |
| `currency_code` | CHAR(3) | ISO 4217 currency code |
| `payment_gateway_fee` | DECIMAL | Transaction processing fee |
| `transaction_id` | VARCHAR(50) | Unique payment reference |

---

## Sample Data

The database is pre-loaded with **50 rows per table** covering realistic, globally diverse scenarios:

- **50 aircraft** spanning Boeing, Airbus, Embraer, Bombardier, ATR, and De Havilland — including active, maintenance, retired, leased-out, and scrapped statuses.
- **50 airports** across 6 continents including JFK, Heathrow, Dubai International, Singapore Changi, Mumbai CSIA, and more — each with correct IANA timezones.
- **50 passengers** from 30+ nationalities with full profiles, emergency contacts, and documented medical conditions where applicable.
- **50 crew members** across all roles with license compliance dates and safety check schedules.
- **50 flights** covering Domestic, Regional, International, and Intercontinental routes across all flight types.
- **50 trip instances** with a mix of on-time, delayed, cancelled, and completed statuses.
- **50 bookings** across Economy through First Class, with diverse payment methods and booking channels.
- **50 reviews** with multi-dimensional ratings (1–5) and airline responses for a subset of bookings.
- **50 revenue records** in 20+ currencies (USD, GBP, EUR, AED, JPY, INR, SGD, BRL, QAR, CHF, and more).

---

## Setup & Installation

### Prerequisites
- MySQL 8.0+ (or MariaDB 10.5+)
- MySQL Workbench / DBeaver / CLI access

### Steps

```bash
# 1. Clone the repository
git clone https://github.com/your-username/aeroxdb.git
cd aeroxdb

# 2. Log into MySQL
mysql -u root -p

# 3. Run the SQL script
source AeroxProject.sql;

# 4. Verify the database
USE aeroxDB;
SHOW TABLES;
```

### Expected Output

```
+-------------------------+
| Tables_in_aeroxDB       |
+-------------------------+
| aircraft                |
| airport                 |
| booking_info            |
| crew_info               |
| flight                  |
| passenger               |
| review                  |
| revenue_tracking        |
| trip_details            |
+-------------------------+
```

---

## Use Cases & Analytics

AeroxDB supports a wide range of operational and analytical queries:

**Fleet & Maintenance**
```sql
-- Aircraft due for servicing within 30 days
SELECT model_name, next_servicing_dt
FROM aircraft
WHERE next_servicing_dt <= DATE_ADD(CURDATE(), INTERVAL 30 DAY)
AND aircraft_status = 'active';
```

**Crew Compliance**
```sql
-- Crew with expiring licenses in the next 60 days
SELECT CONCAT(first_name, ' ', last_name) AS crew_name, role, license_expiry_date
FROM crew_info
WHERE license_expiry_date <= DATE_ADD(CURDATE(), INTERVAL 60 DAY)
AND is_active = TRUE;
```

**Revenue Performance**
```sql
-- Total profit by currency
SELECT currency_code, SUM(profit) AS total_profit, SUM(revenue) AS total_revenue
FROM revenue_tracking
GROUP BY currency_code
ORDER BY total_profit DESC;
```

**Passenger Satisfaction**
```sql
-- Average ratings by travel class
SELECT b.travel_class,
       ROUND(AVG(r.rating), 2)               AS avg_overall,
       ROUND(AVG(r.service_rating), 2)        AS avg_service,
       ROUND(AVG(r.food_rating), 2)           AS avg_food
FROM review r
JOIN booking_info b ON r.booking_id = b.booking_id
GROUP BY b.travel_class;
```

**Operational Punctuality**
```sql
-- On-time performance summary
SELECT trip_status, COUNT(*) AS count,
       ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 2) AS percentage
FROM trip_details
GROUP BY trip_status;
```

---

## Technologies Used

| Tool | Purpose |
|---|---|
| **MySQL 8.0** | Core relational database engine |
| **SQL DDL** | Schema creation with constraints, ENUMs, foreign keys |
| **SQL DML** | Bulk data insertion (50 rows per table) |
| **ENUM Types** | Controlled vocabularies for status fields |
| **Referential Integrity** | `ON DELETE RESTRICT` foreign key enforcement |
| **Timestamps** | `DEFAULT CURRENT_TIMESTAMP` and `ON UPDATE` triggers |
| **CHECK Constraints** | Validation for ratings (1–5) and experience (≥ 0) |

---

## Project Structure

```
aeroxdb/
│
└── AeroxProject.sql     ← Full schema + seed data (DDL + DML)
```

---

## Author

Built as a comprehensive database design project demonstrating normalised schema architecture, referential integrity, and real-world airline operations modelling.

---

*AeroxDB — Powering smarter skies through structured data.*
