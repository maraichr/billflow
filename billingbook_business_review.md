## Billingbook: Business Overview and System Specification

This document provides a clear, business-focused overview of the Billingbook platform. It is intended for product managers, business stakeholders, and other non-technical team members to understand the system's capabilities, workflows, and data structure.

### 1. Executive Summary

#### System Overview
Billingbook is an all-in-one business management platform designed for small businesses and freelancers. It combines a public marketplace for advertising products and services, a powerful Customer Relationship Management (CRM) tool for managing sales leads, and a complete billing system for invoicing and tracking payments. This integrated approach allows users to manage their entire business lifecycle—from attracting new customers to getting paid—within a single application.

#### Target Audience & Needs
The platform serves several key user groups:
*   **Small Business Owners & Freelancers:** Need a simple, cost-effective way to find leads, manage client relationships, and handle financial administration without juggling multiple software tools.
*   **Brokers & Agents (e.g., Real Estate, Automotive):** Require a system to manage leads generated from public listings and track complex transactions through to completion.
*   **Referral Partners & Lead Generators:** Participate in a commission-based ecosystem by sourcing and providing leads to service providers on the platform.
*   **End-Customers:** The clients of our users, who need a simple way to view and pay invoices online.

#### Key Business Capabilities
1.  **Comprehensive Billing & Invoicing:** Create professional invoices, manage a customer database, define a product catalog, apply taxes (VAT) and discounts, record payments, and generate customer statements.
2.  **Lead-to-Cash CRM:** Manage a complete sales pipeline, from capturing a new lead, assigning it to a team member, tracking negotiations with notes and updates, and converting a successful deal into an invoice.
3.  **Multi-Category Marketplace:** Publish classified ads for goods and services across diverse categories like property, vehicles, jobs, and tourism, creating a primary channel for lead generation.
4.  **Collaborative Partner Network:** Supports a referral ecosystem where users can act as `Service Providers`, `Lead Generators`, or `Lead Sources`, enabling commission-based partnerships.
5.  **Integrated Communication:** Built-in messaging, user walls, and social features foster a community and allow for seamless communication between business partners on the platform.
6.  **Customizable User Experience:** Users can enable or disable major modules (e.g., CRM, Social) and customize CRM fields to tailor the platform to their specific business needs.

#### Business Value Proposition
Billingbook eliminates the complexity and cost of using separate tools for marketing, sales, and finance. By providing a single, unified platform, it empowers small businesses to streamline operations, reduce software expenses, and gain a holistic view of their business health—from initial lead to final payment.

---

### 2. Standardized API Specification

The following is a standardized, RESTful API design for the core resources within Billingbook. This specification promotes consistency, predictability, and ease of integration.

**Naming Convention:** `/api/v1/{resource}/{id}/{action}`

#### Customers API
Manages the customers of a platform user.

| Endpoint | Method | Description | Request Body Schema | Response Schema | Permissions |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `/api/v1/customers` | `GET` | Retrieve a list of all customers for the authenticated user. | N/A | Array of Customer objects | User |
| `/api/v1/customers` | `POST` | Create a new customer. | Customer object (name, email, etc.) | The newly created Customer object | User |
| `/api/v1/customers/{id}` | `GET` | Retrieve a single customer by their ID. | N/A | A single Customer object | User (Owner) |
| `/api/v1/customers/{id}` | `PUT` | Update an existing customer's details. | Customer object with updated fields | The updated Customer object | User (Owner) |
| `/api/v1/customers/{id}` | `DELETE` | Deactivate a customer record (soft delete). | N/A | `{ "status": "deactivated" }` | User (Owner) |

#### Invoices API
Manages financial invoices.

| Endpoint | Method | Description | Request Body Schema | Response Schema | Permissions |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `/api/v1/invoices` | `GET` | Retrieve a list of all invoices for the user. | N/A | Array of Invoice objects | User |
| `/api/v1/invoices` | `POST` | Create a new draft invoice. | `{ "customerId": "uuid", "items": [...] }` | The newly created Invoice object | User |
| `/api/v1/invoices/{id}` | `GET` | Retrieve a single invoice by its ID. | N/A | A single Invoice object with items | User (Owner) |
| `/api/v1/invoices/{id}/view` | `GET` | Retrieve the public view of an invoice using its secret key. | N/A | Public Invoice object | Public (with key) |
| `/api/v1/invoices/{id}/payments` | `POST` | Record a payment against an invoice. | `{ "amount": 100.00, "paymentDate": "YYYY-MM-DD" }` | The updated Invoice object | User (Owner) |

#### Leads API
Manages sales leads in the CRM.

| Endpoint | Method | Description | Request Body Schema | Response Schema | Permissions |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `/api/v1/leads` | `GET` | Retrieve a list of leads (e.g., open, or owned by the user). | N/A | Array of Lead objects | User |
| `/api/v1/leads` | `POST` | Create a new lead and post it to the marketplace. | Lead object (title, value, description) | The newly created Lead object | User |
| `/api/v1/leads/{id}` | `GET` | Retrieve a single lead by its ID. | N/A | A single Lead object | User (Owner/Claimant) |
| `/api/v1/leads/{id}/claim` | `POST` | Claim an open lead to begin working on it. | N/A | The updated Lead object | User (Service Provider) |
| `/api/v1/leads/{id}/notes` | `POST` | Add a note or update to a lead's transaction log. | `{ "type": "update", "content": "..." }` | The newly created Note object | User (Owner/Claimant) |

---

### 3. Simplified Data Model

The system's data is organized into logical business domains. Technical prefixes have been removed for clarity.

#### Identity & Access Management
This domain manages who can access the platform and what they can do.
*   **User:** The central account for an individual or business using Billingbook. It stores login credentials, profile information, and company details.
*   **User Setting:** Contains preferences and feature flags for a **User**, such as which modules (Billing, CRM) are active and custom field names.
*   **Relationships:** A **User** has one **User Setting** profile that controls their experience.

#### CRM & Leads
This domain covers the entire sales and customer relationship lifecycle.
*   **Lead:** A potential business opportunity, with a value, description, and status. Leads can be "Open" for anyone to claim or "Exclusive" to a specific user.
*   **Transaction Note:** A message, update, or offer related to a **Lead**. These notes form the history of a deal's negotiation.
*   **Relationships:** A **Lead** can have many **Transaction Notes**. A **User** can create a **Lead**, and another **User** (a Service Provider) can claim it.

#### Billing & Finance
This domain handles all financial transactions.
*   **Customer:** A client of a platform **User**. This is the entity that receives invoices, not a user of the platform itself.
*   **Product:** A reusable service or item with a standard price, which a **User** can add to their invoices.
*   **Invoice:** A financial document sent to a **Customer**. It contains line items, totals, tax, and payment status.
*   **Invoice Item:** A single line on an **Invoice**, detailing the product, quantity, and price.
*   **Payment:** A record of money received from a **Customer** for a specific **Invoice**.
*   **Relationships:** A **User** owns many **Customers**. An **Invoice** is sent to one **Customer** and contains many **Invoice Items**. An **Invoice** can have multiple **Payments** recorded against it until it is fully paid.

#### Marketplace & Listings
This domain manages the public classifieds portal.
*   **Listing:** A public advertisement for an item or service (e.g., a car for sale, a property for rent). It contains all the details, images, and pricing for the ad.
*   **Listing Type / Category:** The classification system that organizes **Listings** (e.g., Type: Motoring, Category: Cars).
*   **Relationships:** A **User** creates a **Listing** to advertise their offerings. These **Listings** are the primary source for generating new **Leads** in the CRM.

---

### 4. Key Business Rules & Logic

This section translates technical rules into plain business logic.

#### Workflow States (Lifecycles)

*   **Lead Status:**
    1.  **New:** A lead is created and is available on the marketplace.
    2.  **Active:** A Service Provider has claimed the lead and is reviewing it.
    3.  **In Progress:** The provider is actively negotiating with the potential customer.
    4.  **Finalized:** The deal is closed (either won or lost).

*   **Invoice Status:**
    1.  **Pending:** The invoice has been created and sent to the customer but is not yet paid.
    2.  **Paid:** The sum of all recorded payments equals the total invoice amount. The invoice is considered closed.

*   **Customer CRM Status:**
    1.  **New/Open:** A potential customer or active prospect.
    2.  **In Progress:** The user is actively engaged in a sales process with the customer.
    3.  **Completed/Declined:** The sales process has concluded, either successfully or unsuccessfully.

#### Financial Calculations

*   **Invoice Line Item Total:**
    *   `Line Item Total = (Quantity × Price per Unit) - Discount`
*   **Invoice Subtotal:**
    *   `Subtotal = Sum of all Line Item Totals`
*   **Invoice Total (Amount Due):**
    *   `Total = Subtotal + VAT Amount`
*   **Invoice Amount Due (Balance):**
    *   `Amount Due = Invoice Total - Sum of all Payments`

---

### 5. Core Business Workflows

#### Workflow 1: Lead-to-Invoice Process
This workflow describes the journey from a new business opportunity to a paid invoice.

*   **Roles Involved:** Lead Generator (optional), Service Provider (User), End-Customer.
*   **Process:**
    1.  A **Lead Generator** or **Service Provider** creates a new **Lead** (e.g., "Looking for a web designer").
    2.  The Lead appears on the internal marketplace. If it's "Open," any registered **Service Provider** can see it.
    3.  A **Service Provider** claims the Lead, changing its status to "Active." The lead is now assigned to them.
    4.  The provider engages with the prospect, logging all communication as **Transaction Notes** (e.g., "Sent proposal," "Customer offered $500"). The status moves to "In Progress."
    5.  An agreement is reached. The provider marks the Lead as "Finalized."
    6.  The provider navigates to the Billing module and creates a new **Invoice** for their **Customer**.
    7.  The system generates the invoice, which is sent to the End-Customer for payment.
*   **Decision Points:**
    *   Should a Service Provider claim an open lead?
    *   Is the offer in a transaction note acceptable?
*   **Expected Outcome:** A successful deal is converted into a payable invoice, and commissions (if any) are calculated for the Lead Generator/Source.

#### Workflow 2: Standard Billing Process
This workflow describes how a user bills a customer for products or services.

*   **Roles Involved:** User, End-Customer.
*   **Process:**
    1.  The **User** adds a new client to their **Customer** list.
    2.  The User creates a new **Invoice**, selecting the Customer from their list.
    3.  The User adds **Invoice Items** by selecting from their predefined **Product** catalog or by entering ad-hoc items with custom descriptions and prices.
    4.  The system automatically calculates the subtotal, applies the User's default VAT rate, and computes the final total.
    5.  The User saves and sends the invoice. Its status is now **Pending**.
    6.  The **End-Customer** makes a payment. The User records this **Payment** against the invoice.
    7.  Once payments cover the full invoice amount, the system automatically updates the invoice status to **Paid**.
*   **Decision Points:**
    *   Apply a line-item or overall discount?
    *   Mark the invoice as paid manually or record a partial payment?
*   **Expected Outcome:** The User's financial records are updated, the invoice is marked as paid, and the customer's account statement reflects the transaction.

---

### 6. Recommendations

Based on the analysis of the system architecture, we recommend the following enhancements to improve functionality, user experience, and security.

#### Missing Functionality
*   **Payment Gateway Integration:** The current system relies on manually recording payments. Integrating with gateways like Stripe or PayPal would automate payment collection and reconciliation, significantly improving efficiency.
*   **Recurring Invoices:** Many businesses use subscription or retainer models. The ability to automatically generate and send invoices on a recurring schedule (e.g., monthly) is a critical missing feature.
*   **Reporting & Dashboards:** The system collects valuable data but lacks a dedicated reporting module. Dashboards showing revenue over time, outstanding payments, and lead conversion rates would provide immense value to users.
*   **Quote/Estimate Generation:** A workflow to create and send quotes that can be approved by a client and converted into an invoice with one click would streamline the sales process.

#### User Experience (UX) Improvements
*   **Simplified Listing Creation:** The listing creation process appears to be a multi-step wizard tracked by several boolean flags (`listing_loaded_step_...`). This could be simplified into a more fluid, single-page form with better user guidance to reduce friction.
*   **Consolidate User Profile:** The `User` and `User Setting` entities contain overlapping information (e.g., VAT settings). Merging these would simplify account management for the user and reduce data redundancy.
*   **Contextual Social Features:** The social features (friends, followers) feel disconnected from the core business workflows. They should be integrated more contextually, for example, by highlighting trusted partners or showing transaction history with a connection.

#### Data Integrity Enhancements
*   **Remove Redundant Fields:** Fields like `users_vat_registered` exist in both the `User` and `User Setting` tables. This creates a risk of data inconsistency. The data model should be refactored to have a single source of truth for such settings.
*   **Enforce Data Constraints:** Many critical fields are nullable (can be empty), which can lead to incomplete records. Fields like `users_email`, `customers_name`, and `invoice_amount` should be made mandatory at the database level.

#### Security Considerations
*   **CRITICAL - Secure Storage of Sensitive Data:** The `users_banking_details` field appears to store sensitive financial information as plain text. This is a major security vulnerability. All such data **must** be encrypted at rest, and access must be strictly controlled. It is highly recommended to integrate with a compliant third-party provider for handling banking data.
*   **CRITICAL - Password Management:** The presence of a `users_secret_md5` field suggests a legacy, insecure password hashing algorithm (MD5) may be in use. All passwords must be hashed using a modern, strong algorithm like Argon2 or bcrypt.
*   **Secure Public Invoice Links:** The `invoices_secret_public_view_key` is a good feature but must be implemented with a cryptographically secure random token to prevent unauthorized users from guessing URLs and accessing invoice data.
