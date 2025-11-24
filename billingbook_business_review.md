# Billingbook: Business Operating System
**Document Type:** Functional Specification & Business Requirements  
**Version:** 2.0 (Standardized)  
**Date:** October 26, 2023

---

## 1. Executive Summary

### 1.1 System Overview
**Billingbook** is a comprehensive **Business Operating System** designed for Small to Medium Enterprises (SMEs) and freelancers. It unifies three traditionally fragmented business functions—Customer Relationship Management (CRM), Financial Operations (ERP), and Social Networking—into a single "Super App."

Instead of juggling separate tools for marketing (Facebook), lead tracking (Excel), and billing (Accounting Software), Billingbook offers a unified "Lead-to-Cash" workflow. It allows users to market services, acquire leads through a marketplace, manage client negotiations, and seamlessly convert closed deals into paid invoices.

### 1.2 Target Audience
*   **Service Providers (SMEs):** Businesses requiring an all-in-one tool to find work, manage clients, and handle billing.
*   **Lead Generators:** Agents who source business opportunities and monetize them by selling leads on the platform.
*   **Marketplace Consumers:** Individuals browsing specific vertical directories (Motoring, Art, Tourism) or looking for services.
*   **Platform Administrators:** Staff responsible for content moderation, user verification, and marketplace oversight.

### 1.3 Key Business Capabilities
1.  **Unified Financial Suite:** A complete cycle for managing products, generating quotes, and issuing invoices with automated tax/VAT calculations and payment tracking.
2.  **Lead Exchange Marketplace:** A dynamic engine where business opportunities are created, categorized (e.g., "For Sale" vs. "Wanted"), and distributed to service providers via AI matching or manual claiming.
3.  **Vertical-Specific Directories:** Specialized listing capabilities for distinct industries (e.g., detailed VIN/Engine specs for Motoring, exhibition details for Art) rather than generic listing fields.
4.  **Social Commerce Graph:** A built-in social network allowing businesses to build brand equity through profiles, newsfeeds, and direct messaging, fostering trust before transactions occur.
5.  **Geo-Spatial Targeting:** A strict location hierarchy ensures users only see leads, listings, and services relevant to their specific region (Country down to Town level).

### 1.4 Value Proposition
Billingbook reduces operational friction for small businesses. By integrating the **source of revenue** (leads/marketplace) directly with the **management of revenue** (invoicing/CRM), the platform significantly reduces administrative time and increases the speed at which service providers can find work and get paid.

---

## 2. API Standardization

The following specification standardizes the legacy backend into a modern, RESTful API architecture.

**Base URL:** `https://api.billingbook.com/api/v1`

### 2.1 Core Resources

| Resource | Description |
| :--- | :--- |
| `/users` | Identity management and profile settings |
| `/customers` | Client data management for billing purposes |
| `/leads` | The marketplace of business opportunities |
| `/invoices` | Financial records and billing documents |
| `/listings` | Vertical-specific marketplace items (Cars, Art, etc.) |

### 2.2 Endpoint Specification

#### Customers (Billing Entities)
*Manage the clients that users are invoicing.*

*   **List Customers**
    *   `GET /customers`
    *   *Permissions:* User (Own records only)
*   **Create Customer**
    *   `POST /customers`
    *   *Body:* `{ "name": "Acme Corp", "email": "contact@acme.com", "vat_number": "12345", "address": {...} }`
*   **Update Customer**
    *   `PUT /customers/{id}`
    *   *Body:* `{ "status": "active", "pricing_tier_id": "tier_02" }`
*   **Delete Customer**
    *   `DELETE /customers/{id}`

#### Invoices (Financials)
*Manage the full billing lifecycle.*

*   **Create Invoice**
    *   `POST /invoices`
    *   *Body:* `{ "customer_id": "uuid", "due_date": "2023-12-01", "items": [ { "product_id": "uuid", "qty": 2 } ] }`
*   **Get Invoice Details**
    *   `GET /invoices/{id}`
    *   *Response:* Returns full invoice object including calculated tax totals and line items.
*   **Finalize Invoice**
    *   `POST /invoices/{id}/finalize`
    *   *Action:* Locks the invoice from editing and assigns a final invoice number.
*   **Public View**
    *   `GET /public/invoices/{view_key}`
    *   *Permissions:* Public (No Auth required - for client viewing).

#### Leads (CRM & Marketplace)
*Manage the acquisition and processing of leads.*

*   **Browse Leads**
    *   `GET /leads`
    *   *Filters:* `?type=physical&category=motoring&location=ny`
*   **Claim Lead**
    *   `POST /leads/{id}/claim`
    *   *Action:* Assigns the lead to the current user and moves it to the "Transaction" phase.
*   **Update Transaction Status**
    *   `PUT /leads/{id}/transaction`
    *   *Body:* `{ "status": "negotiating", "notes": "Client requested 10% discount" }`

---

## 3. Data Model Simplification

The data structure has been reorganized into logical business domains, removing technical prefixes.

### Domain 1: Identity & Access
*   **User:** The central login entity. Stores authentication, KYC status (Verified/Unverified), and tax settings (VAT Registered Y/N).
*   **User Settings:** Controls feature toggles. Allows a user to enable/disable specific modules (e.g., "Enable CRM," "Enable Motoring Vertical").
*   **Location:** Hierarchical data (Country > Province > District > Town) used for filtering content.

### Domain 2: CRM & Sales
*   **Lead:** A raw business opportunity containing a value, type (Goods vs. Services), and exclusivity status (Open vs. Exclusive).
*   **Lead Claim:** A record tracking the transfer of a lead from a "Generator" to a "Service Provider."
*   **Transaction:** The "Deal Room." Created once a lead is claimed. Stores negotiation history, digital handshakes (approvals from both sides), and conversion status.

### Domain 3: Finance (Billing)
*   **Customer:** The entity being billed. Distinct from a system "User." Contains specific billing preferences and pricing strategies.
*   **Product:** Inventory items or services available for sale. Tracks stock levels for physical goods.
*   **Invoice:** The legal financial request. Links a Customer to Products.
*   **Invoice Item:** Individual lines on an invoice. Supports temporary session storage (shopping cart functionality).

### Domain 4: Vertical Marketplaces
*   **Vehicle Listing:** Specialized inventory for the Motoring sector (VIN, Mileage, Maintenance Plans).
*   **Attraction Listing:** Specialized inventory for Tourism (Map coordinates, WhatsApp contact integration).
*   **Social Post:** User-generated content for the newsfeed.

---

## 4. Business Rules Translation

### Financial Rules
1.  **Tax Compliance:** If a user is flagged as `VAT Registered`, the system must automatically calculate tax on all generated invoices based on the `users_vat_tax_rate`.
2.  **Invoice Locking:** Invoices in `Draft` status can be edited. Once moved to `Finalized` or `Sent`, the invoice is locked to preserve audit trails.
3.  **Pricing Strategies:** Customers can be assigned specific pricing tiers (Retail, Wholesale, Volume). The system automatically applies the correct price for that customer when items are added to an invoice.

### Marketplace Rules
1.  **Lead Exclusivity:**
    *   *Open Leads:* Visible to all service providers; first to claim gets it.
    *   *Exclusive Leads:* Only visible to specific providers matched by the system.
2.  **The "Handshake" Protocol:** A transaction cannot be closed until both the Buyer and the Seller have digitally approved the terms (`lead_seller_approve` AND `lead_buyer_approve` are both True).
3.  **Automated Billing Trigger:** When a transaction is marked `Closed/Won`, the system automatically generates a draft Invoice populated with the transaction details.

---

## 5. Workflow Documentation

### Workflow A: The Lead-to-Cash Cycle
*The primary revenue journey for a Service Provider.*

1.  **Lead Generation:** A Lead Generator posts a request (e.g., "I need a web designer").
    *   *System:* Creates a Lead record with status `New`.
2.  **Distribution:** The system matches the lead to relevant providers based on Location and Category.
3.  **Claiming:** A Service Provider views the lead and clicks "Claim."
    *   *System:* Changes status to `In Progress`, creates a Transaction record, and hides the lead from other providers.
4.  **Negotiation:** Provider and Client exchange notes and files via the Transaction dashboard.
5.  **Agreement:** Both parties click "Approve" (Digital Handshake).
6.  **Conversion:** Provider clicks "Generate Invoice."
    *   *System:* Converts Transaction data into an Invoice Draft.
7.  **Payment:** Provider finalizes the invoice and sends the secure link to the Client.

### Workflow B: Vertical Listing Creation (e.g., Selling a Car)
1.  **Initiation:** User selects "Create Listing" -> "Motoring."
2.  **Data Entry:** User inputs standard data (Price, Title) and Domain Data (VIN, Engine Specs).
3.  **Validation:** System checks required fields.
4.  **Moderation:** Listing enters `Pending Approval` queue.
5.  **Publication:** Admin approves listing; it becomes visible in the Motoring Directory and on the User's Social Wall.

---

## 6. Recommendations

Based on the analysis of the legacy system, the following improvements are recommended for the roadmap:

### 6.1 Functional Gaps
*   **Payment Gateway Integration:** The current system tracks "Paid" status but does not appear to process credit cards. **Recommendation:** Integrate Stripe or PayPal to allow clients to pay invoices directly via the "Public View" link.
*   **Dispute Resolution:** There is no workflow for when a "Handshake" goes wrong. **Recommendation:** Add a "Raise Dispute" status to Transactions that alerts Admins.

### 6.2 User Experience (UX)
*   **Mobile App/PWA:** The heavy reliance on "WhatsApp" integration implies users are mobile. **Recommendation:** Ensure the frontend is a Progressive Web App (PWA) optimized for mobile usage.
*   **Onboarding Wizard:** The "User Settings" are complex. **Recommendation:** Create a "Getting Started" wizard that asks "What is your business?" and auto-configures the toggles (CRM, Social, etc.).

### 6.3 Security & Data Integrity
*   **Authentication:** The current "Secret Public View Key" for invoices is a security risk if guessed. **Recommendation:** Implement signed URLs with expiration times for public invoice viewing.
*   **Audit Logging:** While transaction notes exist, a dedicated system audit log (who changed what and when) is missing and critical for financial compliance.
