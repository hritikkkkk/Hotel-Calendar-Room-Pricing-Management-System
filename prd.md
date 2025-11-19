Product Requirements Document (PRD)

Product: Hotel Calendar & Room Pricing Management System
Version: 1.0
Owner: Product Team
Status: Draft

1. Overview

Hotels need a centralized system to manage daily room availability, room types, and dynamic prices.
The product will allow admins to update room inventory and prices for each date, and provide APIs for frontend booking engines.

This PRD defines all functional, non-functional, and API-level requirements for the system.

2. Goals
Primary Goals

Allow hotel staff to manage daily availability of room types (Single, Double).

Allow staff to manage daily prices per room type.

Provide a calendar UI to view room status.

Provide backend services to integrate with booking portals.

Secondary Goals

Bulk update (date range update).

Export calendar data (CSV/Excel).

Audit history for editing changes.

3. Users & Use Cases
User Personas

Hotel Admin / Manager

Updates availability and pricing.

Reviews occupancy trends.

Exports reports.

Booking Engine (API Consumer)

Reads inventory and pricing.

Validates availability.

4. Functional Requirements

4.1 Calendar Management

Admin can view monthly and daily calendar.

Each date must show:

Single rooms available

Double rooms available

Price per room type

Day name(MON  - SUN)

Admin can filter by: month and room type

4.2 Modify Inventory and Pricing

Admin can update: number of room available and price for each room type

Admin can update: a single date or multiple dates (BULK Action)

System must validate: values should be >=0 and Rooms cannot exceed maximum hotel room capacity (Configurable)

4.3 Bulk Operations

Admin can set: Day Wise pattern (e.g, Every Friday set SINGLE_PRICE = X )
Date Range Updates for Promotions and Peak season

4.4 Room Capacity Rules

Admin can configure : Total single and double rooms in hotel
System must prevent Inventory from exceeding these capacities

4.5 API Requirements

4.5.1 Public APIs (for website booking Engines)

GET /calendar/{month} : Returns Complete Inventory and price for the month
GET /availability : Input: Check-In,Check-Out,Room type, quantity Output: available or not
GET /pricing : Input: Room type,date range Output: consolidated price

4.5.2 Admin APIs (protected with auth)
POST /admin/calendar/update
POST /admin/calendar/bulk-update
GET /admin/calendar/history

4.6 Data Export

Admin can export: Monthly calendar data and Date range report


5. Non-functional Requirements

5.1 Security

jwt based authentication for admin panel
role based access control (RBAC) : Admin (full control) and Viewer (read-only)




