# Billingbook - Product Requirements Document (Enhanced)

**Status**: Draft / Reverse-Engineered  
**Version**: 2.0  
**Source**: Automated Legacy Code Analysis  

---

## 1. Executive Summary & Business Overview

### 1.1 System Purpose
"Billingbook" is a multi-tenant **Business Operating System & Community Platform**. Unlike a standalone invoicing tool, this system functions as a "Super App" for Small to Medium Enterprises (SMEs) and freelancers. It integrates three distinct business pillars into a single database:
1.  **ERP/Back-Office**: Invoicing, customer management, and product inventory.
2.  **CRM & Lead Exchange**: A marketplace where leads are generated, distributed, claimed, and tracked through to transaction closure.
3.  **Vertical Marketplace & Social Network**: Specialized listing capabilities for industries (Motoring, Art, Tourism) overlayed with social networking features (Profiles, Friending, Timelines) to drive engagement.

### 1.2 Target Users
*   **Service Providers & SMEs**: Businesses needing to invoice clients, manage inventory, and gain exposure through directory listings.
*   **Lead Generators**: Users or agents who source potential business opportunities and monetize them via the platform.
*   **General Consumers**: Users browsing the social feed, marketplace listings (cars, art, tourism), or looking for service providers.
*   **Administrators**: Staff managing the location hierarchy, vetting listings, and overseeing the lead marketplace.

### 1.3 Core Capabilities
*   **Financial Management**: Full "Quote-to-Cash" cycle including tax/VAT handling, customizable pricing strategies (`pricing_strategy`), statement generation, and payment tracking.
*   **Lead Marketplace Engine**: A sophisticated workflow for creating leads, categorizing them (Physical/Virtual, For Sale/Wanted), and allowing service providers to "claim" or purchase these leads.
*   **Vertical-Specific Directories**: distinct data structures for different industries:
    *   *Motoring*: Detailed vehicle attributes (VIN, Make, Model, Engine specs).
    *   *Tourism*: Attractions, Accommodation details.
    *   *Art*: Exhibitions and gallery profiles.
    *   *Flea Markets*: Vendor management within markets.
*   **Social Connectivity**: A complete internal social graph allowing users to Follow, Friend, Block, and interactive via a "Wall" (Newsfeed) with posts, likes, and shares.
*   **Geo-Spatial Organization**: A strict hierarchical location system (Country → Province/Area → District → Town) used to filter listings and leads relevant to specific users.

### 1.4 Business Value Proposition
The platform solves the fragmentation problem for small business owners. Instead of using Facebook for marketing, Excel for leads, and accounting software for billing, Billingbook allows a user to:
*   Market their services via social profiles and listings.
*   Receive leads directly through the system (via the "Call Center" or Lead Exchange).
*   Convert those leads into Transactions.
*   Seamlessly generate Invoices from those closed transactions.

### 1.5 Key Workflows
*   **The Lead-to-Cash Cycle**: *Lead Creation* (sourced by a Lead Generator) → *Distribution* (Matched via AI or claimed by Service Provider) → *Transaction Management* (Notes, attachments, negotiation) → *Close Deal* → *Generate Invoice*.
*   **Listing Management**: *User Profile Creation* → *Select Vertical* (e.g., Sell a Car) → *Input Domain Specifics* (Mileage, VIN) → *Admin Approval* → *Public Listing*.
*   **Social Engagement**: *User Login* → *Post to Wall* → *Network Reacts/Comments* → *Direct Messaging via Message Center*.

---

### 1.6 Technical Statistics
*   **Total Entities**: 75
*   **PHP Files Analyzed**: 49
*   **SQL Queries Found**: 86
*   **Status Transitions**: 23
*   **Business Rules**: 52
*   **Calculations**: 13
*   **API Endpoints**: 41

---

## 2. Entities

### Core Schema - Identity & Access Management

#### Users Basics Login
*Originally: `000_users_basics_login`*
**Description**: The central identity table ("God Object") storing authentication credentials, basic profile data, KYC status, and configuration flags for every user type (Company, Private, Admin).

**Fields:**
| Field | Type | Nullable | Description |
|-------|------|----------|-------------|
| users_uid | UUID | No | Primary Key |
| users_email | VARCHAR(100) | Yes | Login Email |
| users_is_comp_priva_both | SMALLINT | Yes | **Type**: 1=Company, 2=Private, 3=Both |
| user_active | BOOLEAN | Yes | Account status (Active/Suspended) |
| user_verified | BOOLEAN | Yes | KYC/Email Verification status |
| users_vat_registered | BOOLEAN | Yes | Is the user a registered tax entity? |
| users_vat_tax_rate | NUMERIC(3,1) | Yes | Default tax rate for this user's invoices |
| users_invoice_start | INTEGER | Yes | Seed number for invoice generation |
| users_admin_level | ENUM(1, 2, 3) | Yes | 1=Super Admin, 2=Moderator, 3=User |
| access_level_id | INTEGER | Yes | FK → access_level (RBAC Role) |
| ... | ... | ... | *[Technical fields retained from original]* |

**Relationships:**
- **Belongs to Location**: Links to `country`, `area`, `district`, `town` for geo-targeting.
- **Belongs to Telegram**: Links to `user_telegram_chat` for notifications.

---

#### Users Setting
*Originally: `000_users_settings`*
**Description**: Toggles user-specific modules. Allows a user to turn on/off specific verticals (e.g., a user might want CRM but not Social).

**Fields:**
| Field | Type | Nullable | Description |
|-------|------|----------|-------------|
| users_uid | INTEGER | No | PK |
| users_is_crm_billing | BOOLEAN | Yes | Enable Invoicing/CRM Module |
| users_is_a_service_provider | BOOLEAN | Yes | Enable "Pro" Marketplace features |
| users_is_social | BOOLEAN | Yes | Enable Social Network features |
| users_is_motoring | BOOLEAN | Yes | Enable Car Trading features |

---

### Business Schema - Domain Entities

#### Crm Lead
*Originally: `010_crm_leads`*
**Description**: Represents a potential business opportunity. Leads are commodities that can be bought, sold, or assigned.

**Fields:**
| Field | Type | Nullable | Description |
|-------|------|----------|-------------|
| crm_leads_id | UUID | No | Primary Key |
| crm_leads_value | NUMERIC(20,2) | No | Estimated monetary value of the lead |
| crm_leads_target_open_exclusive | ENUM(O, E) | Yes | **O**=Open to all, **E**=Exclusive to one claiming user |
| crm_leads_type_wanted_for_sale | ENUM(W, S) | Yes | **W**=Buyer Request (Wanted), **S**=Seller Offering (For Sale) |
| crm_leads_type_physical_virtual | ENUM(P, V) | Yes | **P**=Physical Good, **V**=Virtual Service |
| crm_leads_status_what_is_progress | ENUM(N, A, I, F) | Yes | **N**=New, **A**=Accepted/Active, **I**=In Progress, **F**=Finalized |
| crm_leads_status_assigned_by_ai | BOOLEAN | Yes | Was this auto-matched by system logic? |

---

#### Crm Leads Claim
*Originally: `010_crm_leads_claimes`*
**Description**: Tracks the ownership transfer of a lead. When a Service Provider "claims" a lead, a record is created here.

**Fields:**
| Field | Type | Nullable | Description |
|-------|------|----------|-------------|
| 1_who_created_the_lead | INTEGER | No | User ID of source |
| 2_who_claimed_the_lead | INTEGER | No | User ID of new owner (Service Provider) |
| crm_leads_date_claim_passed_to_transactions | TIMESTAMPTZ | No | Conversion timestamp |

---

#### Crm Transaction Note
*Originally: `010_crm_transaction_notes`*
**Description**: The "Deal Room". Once a lead is claimed, it becomes a transaction. This table stores the history, negotiation, and state changes of the deal.

**Fields:**
| Field | Type | Nullable | Description |
|-------|------|----------|-------------|
| crm_leads_status | ENUM(N, A, I, F) | Yes | **N**=Negotiating, **A**=Agreement, **I**=Invoicing, **F**=Failed/Finished |
| crm_transaction_notes_type | ENUM | Yes | **update**, **question**, **offer**, **system** notification |
| lead_seller_approve | BOOLEAN | Yes | Digital handshake (Seller side) |
| lead_buyer_approve | BOOLEAN | Yes | Digital handshake (Buyer side) |
| lead_closed_for_billing | BOOLEAN | Yes | Trigger to generate invoice |

---

#### Customer
*Originally: `605_customers`*
**Description**: The paying entity in the system. Can be an organization or individual. Linked to the user but stores billing-specific metadata.

**Fields:**
| Field | Type | Nullable | Description |
|-------|------|----------|-------------|
| customers_uid | UUID | No | Primary Key |
| customers_role | ENUM | No | **organization**, **branch**, **agent**, **client** |
| customers_crm_status | ENUM | Yes | **New**, **Open**, **In Progress**, **Completed**, **Declined**, **Paused** |
| customer_pricing_strategy_id | INTEGER | Yes | Link to custom rate card/pricing tier |
| customers_vat_tax_inc | BOOLEAN | Yes | Are prices VAT inclusive? |

---

#### Invoice
*Originally: `620_invoices`*
**Description**: The financial record requesting payment. Generated from interactions or manually created.

**Fields:**
| Field | Type | Nullable | Description |
|-------|------|----------|-------------|
| invoices_status | ENUM | Yes | **pending**, **paid** |
| invoices_amount_due | NUMERIC(10,2) | Yes | Outstanding balance |
| invoices_secret_public_view_key | VARCHAR(51) | Yes | Token for external "View Invoice" link (no login req) |
| pricing_strategy_id | UUID | Yes | Applied pricing tier |

---

#### Invoice Item
*Originally: `621_invoice_items`*
**Description**: Line items belonging to an invoice. Supports inventory linking and session-based temporary storage (shopping cart behavior).

**Fields:**
| Field | Type | Nullable | Description |
|-------|------|----------|-------------|
| product_id | UUID | Yes | FK to Inventory |
| temp_session_key | VARCHAR(50) | Yes | Used for drafting invoices before saving |
| invoice_items_vat_tax_amount | NUMERIC | Yes | Cached calculated tax value |

---

#### Product
*Originally: `606_products`*
**Description**: Inventory items or services available for billing.

**Fields:**
| Field | Type | Nullable | Description |
|-------|------|----------|-------------|
| product_stock_item_yes_no | BOOLEAN | Yes | **Yes**=Physical Stock tracked, **No**=Service/Digital |
| product_stock_reorder_level | INTEGER | Yes | Low stock alert threshold |
| customer_vat_tax_inc | ENUM(I, E) | Yes | Price is **I**nclusive or **E**xclusive of Tax |

---

### Marketplace & Specialized Verticals

#### Motoring All Vehicle
*Originally: `235_motoring_all_vechicles`*
**Description**: specialized table for vehicle listings with deep technical specifications.

**Fields:**
| Field | Type | Nullable | Description |
|-------|------|----------|-------------|
| car_all_data_vin | CHAR(17) | Yes | Vehicle Identification Number |
| car_all_data_0_100_kms | VARCHAR | Yes | Performance stats |
| car_all_data_maintenance_plan | VARCHAR | Yes | Service history details |

#### Tourism Attraction
*Originally: `053_tourism_attractions`*
**Description**: Listings for venues, hotels, or activities.

**Fields:**
| Field | Type | Nullable | Description |
|-------|------|----------|-------------|
| listing_tel_is_wassup | ENUM(1, 0) | Yes | Is the mobile number WhatsApp enabled? (1=Yes) |
| listing_renewal_month | TEXT | Yes | Subscription management for the listing |
| listing_open_street_maps | VARCHAR | Yes | Co-ordinates for map visualization |

---

### Social & Storage Schema

#### Social Friendship
*Originally: `020_social_friendships`*
**Description**: bidirectional relationship between two users.
**Status**: `pending` → `accepted` or `rejected`.

#### Social Wall Post
*Originally: `030_social_wall_posts`*
**Description**: The "Newsfeed" content generated by users. Supports text, and likely attachments via related tables.

---

### Integration & System Schema

#### Notifications Center
*Originally: `002_notifications_center`*
**Description**: Central inbox for system alerts.
**Types**: `alert`, `info`, `request` (e.g., friend request), `open_lead` (new lead matching user criteria).

#### Location Hierarchy (Country/Area/District/Town)
*Originally: `900` - `903` series*
**Description**: Rigid address database used for filtering leads and listings closer to the user.

---

## 3. Business Flows & State Machines

### 3.1 The "Lead" Lifecycle
Based on `crm_leads` and `crm_leads_transactions`:
1.  **Creation**: Status `New` (N). Created by a "Lead Source".
2.  **Marketplace**: Status `Open` (O) or `Exclusive` (E). Visible to Service Providers.
3.  **Claiming**: A Service Provider claims the lead via `crm_leads_claimes`.
4.  **Transaction**: Moves to "Transaction" table.
    *   *Negotiation*: Notes exchanged. Status `In Progress` (I).
    *   *Handshake*: `lead_seller_approve` = True AND `lead_buyer_approve` = True.
5.  **Closure*: Moves to Status `Finalized` (F).
6.  **Billing**: Boolean `lead_closed_for_billing` triggers Invoice draft.

### 3.2 Invoice Lifecycle
Based on `620_invoices`:
1.  **Draft**: Created via UI or converted from Lead. Items held in `621_invoice_items` with `temp_session_key`.
2.  **Pending**: Saved to database. Status `pending`. `invoices_due_date` is set.
3.  **Viewed**: Accessed via `invoices_secret_public_view_key` (Client view).
4.  **Paid**: Payment recorded in `623_payments` matches total. Status transition to `paid`.

---

## 4. API & Interface Overview

The system uses a mixed architecture of direct PHP actions and AJAX endpoints.

*   **Auth**: Custom session-based auth (`/inc/navbar/left/section/99/everyone/logout`).
*   **Billing Actions**:
    *   `POST /billing/invoice/add/new`: Creates invoice header.
    *   `GET /billing/invoice/add/new/line`: AJAX call to add line items dynamically.
    *   `POST /billing/invoice/finalize`: Commits the invoice (generates tax totals, locks editing).
*   **Client Management**:
    *   `GET /billing/clients/get/locations`: Cascading dropdowns for the geo-hierarchy.
*   **Dashboards**:
    *   Specific dashboard endpoints exist for different user roles (Billing, CRM, Social).

---

## 5. Appendix: Key Data Dictionaries

### 5.1 Pricing Strategies
The field `pricing_strategy_id` appears in Customers and Invoices. This suggests the system supports:
*   **Standard Retail Pricing**
*   **Wholesale/Trade Levels**
*   **Volume-based Discounts**
*   **Rollover Contracts** (enum: 30, 60, 180, 360 days)

### 5.2 User Roles (Inferred)
1.  **Master Admin**: Full access to System Settings and Location Data.
2.  **Service Provider**: paid tier? Access to Claim Leads, CRM, and Billing.
3.  **Lead Generator**: Earns commission (fields usually found in `crm_relationships`).
4.  **Listing Owner**: Manages specific vertical listings (Art, Cars).
5.  **Basic User**: Social features only.

---

## Technical Statistics

- **Total Entities**: 75
- **PHP Files Analyzed**: 49
- **SQL Queries Found**: 86
- **Status Transitions**: 23
- **Business Rules**: 52
- **Calculations**: 13
- **API Endpoints**: 41

## Appendix: PHP Files Analyzed

The following 49 PHP files were analyzed to extract business rules and patterns:

- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/000_other_scripts/billing_dashboard.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/00_inc_system/inc_db_conn.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/00_inc_system/inc_stats.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/01_inc_variables/inc_user_variables.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/01_inc_variables/inc_variables.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/05_css_js/inc_css.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/05_css_js/inc_js.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/07_theme_elements/inc_footer.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/07_theme_elements/inc_meta.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/09_navigation/inc_navbar_left.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/09_navigation/inc_navbar_left_section_01_everyone_basic.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/09_navigation/inc_navbar_left_section_40_members_billing.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/09_navigation/inc_navbar_left_section_50_members_crm.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/09_navigation/inc_navbar_left_section_50_members_listings.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/09_navigation/inc_navbar_left_section_50_members_modules_all.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/09_navigation/inc_navbar_left_section_50_members_modules_properties.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/09_navigation/inc_navbar_left_section_50_members_social_all.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/09_navigation/inc_navbar_left_section_99_everyone_logout.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/09_navigation/inc_navbar_left_section_bottom.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/09_navigation/inc_navbar_left_section_top.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/09_navigation/inc_navbar_right.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/09_navigation/inc_navbar_top.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_clients_add_new.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_clients_add_new_action.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_clients_delete_record.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_clients_delete_record_permanently.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_clients_edit.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_clients_get_locations.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_clients_list.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_clients_list_deleted.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_clients_tagging_select_client.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_clients_tagging_select_products.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_clients_undelete_record.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_clients_update.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_dashboard.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_invoice_add_new.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_invoice_add_new_action.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_invoice_add_new_line_action.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_invoice_delete_action.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_invoice_edit.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_invoice_edit_process.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_invoice_finalize.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_invoice_finalize_action.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_invoice_list.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_invoice_view.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_products_add_new.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_products_add_new_action.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_products_edit.php`
- `/home/maraichr/buildit/billingbook/Billing_Book_2024_4-main/billing_products_list.php`
