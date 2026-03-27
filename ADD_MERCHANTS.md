# APASS Merchant Onboarding Specification (v1)

## 1. Purpose

This document defines the minimum off-chain merchant onboarding structure for Africa Tourism Pass (APASS).

It is designed to:

- onboard merchants quickly
- avoid unnecessary blockchain friction
- support real-world rollout immediately
- provide a clean path toward future backend and dashboard integration
- establish consistent merchant records and transaction logging
- create an operational standard for the first APASS participants

This is the recommended v1 architecture for onboarding the first APASS merchants.

---

## 2. Core Principle

Merchants should be added through the application and operations layer, not through Hardhat or direct smart contract interaction.

Hardhat is for:

- smart contract development
- testing
- deployment
- verification

Merchant onboarding belongs to:

- backend
- database
- admin workflows
- merchant operations
- customer support processes
- reporting and analytics

---

## 3. Scope

This specification covers:

- merchant data structure
- merchant status management
- discount policy recording
- transaction logging
- onboarding workflow
- storage options
- future API design
- operational rules for first merchant activation

This specification does not cover:

- smart contract deployment
- on-chain merchant registry implementation
- automated payment settlement
- full production admin dashboard design
- full compliance program implementation

---

## 4. Merchant Eligibility

The following merchant categories are eligible for APASS onboarding in v1:

- hotel
- lodge
- guesthouse
- campsite
- accommodation_provider
- tour_operator
- tourism_service_provider

A merchant should only be onboarded if it meets all of the following minimum conditions:

- has a valid operating business
- has a clear point of contact
- agrees to honor a defined APASS holder benefit
- agrees to participate in transaction verification
- agrees to have its merchant information recorded in the APASS system
- agrees to operate under the APASS onboarding and participation framework

---

## 5. Recommended v1 Data Model

Each merchant should have a structured record.

### Merchant Object

```json
{
  "merchant_id": "APASS-MERCHANT-0001",
  "business_name": "Example Lodge Namibia",
  "merchant_type": "lodge",
  "country": "Namibia",
  "city": "Oshakati",
  "physical_address": "123 Example Road, Oshakati, Namibia",
  "contact_person": "Jane Doe",
  "contact_email": "info@examplelodge.com",
  "contact_phone": "+264811234567",
  "website": "https://examplelodge.com",
  "wallet_address": "",
  "discount_policy": {
    "apass_holder_discount_percent": 10,
    "notes": "Valid for direct accommodation bookings only"
  },
  "status": "active",
  "properties": [
    {
      "property_id": "PROP-0001",
      "property_name": "Example Lodge Main Property",
      "location": "Oshakati, Namibia"
    }
  ],
  "date_onboarded": "2026-03-27",
  "approved_by": "APASS-ADMIN-001",
  "notes": "Initial pilot merchant"
}
```

## 6. Required Fields

The following fields should be mandatory for v1:

- merchant_id
- business_name
- merchant_type
- country
- city
- physical_address
- contact_person
- contact_email or contact_phone
- discount_policy
- status
- date_onboarded

The following fields are optional but recommended:

- website
- wallet_address
- properties
- approved_by
- notes

---

## 7. Merchant ID Standard

Each merchant should receive a unique APASS merchant identifier.

Recommended format:

APASS-MERCHANT-0001

This format should be:

- sequential
- unique
- permanent
- never reused after assignment

Property identifiers should follow a similar format:

PROP-0001

---

## 8. Merchant Status Values

Use the following status values:

- draft
- pending_review
- approved
- active
- suspended
- inactive
- rejected

Definitions:

- draft: merchant record created but incomplete
- pending_review: merchant submitted and awaiting approval
- approved: merchant accepted but not yet live
- active: merchant is live in APASS
- suspended: merchant temporarily paused
- inactive: merchant no longer participating
- rejected: merchant not approved

For the first merchant, status can move directly to active if the merchant has agreed and is ready to participate.

---

## 9. Discount Policy Structure

Each merchant should define its APASS holder benefit clearly.

Example:

apass_holder_discount_percent: 15  
notes: Applies to accommodation only. Food, beverages, and third-party services excluded.

This is important because:

- APASS holders need a clear benefit
- merchants need a clear commitment
- support and disputes need a reference point
- transaction logging must be measurable
- promotional claims must be auditable

Discount policy should always define:

- discount percentage
- what the discount applies to
- what the discount excludes
- whether the benefit is ongoing or promotional

---

## 10. Multi-Property Merchant Support

A merchant may operate one or many properties.

The merchant record should support:

- one central business identity
- one or more properties
- one shared APASS onboarding
- one or more location-specific benefits if needed later

This is important for operators such as:

- hotel groups
- lodge groups
- guesthouse groups
- tourism operators with multiple destinations

A single onboarding can apply across multiple properties, provided each property is clearly recorded.

---

## 11. Wallet Address Handling

Merchant wallet address is optional in v1.

Use cases for a merchant wallet address may include:

- future token-based incentives
- future treasury-linked flows
- future identity linkage
- future reporting and settlement structures

If not required operationally, this field may remain blank in v1.

Wallet addresses should only be stored if:

- provided voluntarily
- validated for BEP-20 compatibility
- stored correctly in merchant records

---

## 12. Suggested Storage Options

Option A: JSON File

Use a file such as:

/merchants/merchants.json

Best for:

- first merchant
- fast pilots
- manual control
- low-complexity testing

Option B: Database Table

Use a table structure such as:

merchants  
merchant_properties  
merchant_discount_policies  
merchant_contacts  

Best for:

- scale
- admin dashboard
- search and filtering
- reporting
- role-based workflows
- audit history

Recommended path:

- start with JSON if speed is required
- move to database once multiple merchants are active

---

## 13. Recommended File Structure for v1

/APASS  
  /merchants  
    merchants.json  
    merchant_template.json  
    transactions.json  
  /docs  
    MERCHANT_ONBOARDING.md  

If preferred, merchants.json and transactions.json may also be stored under a backend data path rather than directly in the repo.

---

## 14. Example merchant_template.json

merchant_id:  
business_name:  
merchant_type:  
country:  
city:  
physical_address:  
contact_person:  
contact_email:  
contact_phone:  
website:  
wallet_address:  
discount_policy:  
  apass_holder_discount_percent: 0  
  notes:  
status: draft  
properties: []  
date_onboarded:  
approved_by:  
notes:  

---

## 15. Merchant Acceptance Checklist

Before activating a merchant, confirm:

- business name recorded
- merchant type recorded
- physical address recorded
- contact details recorded
- discount policy agreed
- participation approved
- first property recorded
- status changed to active
- internal note added
- merchant informed that onboarding is complete

A merchant should not be announced as active until all minimum details are complete.

---

## 16. First Merchant Operational Workflow

For the first APASS merchant, use this flow:

1. Receive merchant agreement to participate  
2. Collect business details  
3. Define APASS holder discount  
4. Record merchant data in merchants.json or database  
5. Add initial property details  
6. Set status to active  
7. Log approving admin or operator  
8. Confirm merchant is live internally  
9. Test first real or simulated APASS holder interaction  
10. Log first discount usage and points issuance  

---

## 17. First Transaction Logging Structure

Early transaction logging is mandatory.

Each first-phase transaction should record:

- merchant identity
- traveler wallet
- gross booking value
- discount applied
- final amount paid
- points earned
- date
- verification status

Example:

transaction_id: APASS-TXN-0001  
merchant_id: APASS-MERCHANT-0001  
traveler_wallet: 0x1234567890abcdef1234567890abcdef12345678  
booking_value_usd: 400  
discount_percent: 10  
discount_value_usd: 40  
final_amount_paid_usd: 360  
apass_points_earned: 400  
date: 2026-03-27  
status: verified  
notes: Pilot transaction  

---

## 18. Transaction ID Standard

Each transaction should receive a unique identifier.

Recommended format:

APASS-TXN-0001

This should be:

- sequential
- unique
- permanent
- never reused

---

## 19. Transaction Verification Standard

A transaction should only be marked as verified if:

- the merchant is active
- the traveler interaction actually occurred
- the transaction value is known
- the discount was actually applied
- the points to be issued are recorded correctly

Suggested transaction statuses:

- pending
- verified
- rejected
- disputed

---

## 20. APASS Points Calculation Rule

For v1, use a simple and transparent rule.

Example:

1 USD verified spend = 1 APASS Point

Example transaction:

- booking value: 400 USD
- points earned: 400 APASS Points

This should remain configurable later, but v1 should prioritize clarity over complexity.

---

## 21. Support and Dispute Readiness

Even in v1, merchant operations should be support-aware.

Basic operational dispute types may include:

- merchant did not apply agreed discount
- traveler did not receive expected points
- transaction was logged incorrectly
- merchant benefit scope was unclear

For each issue, record:

- dispute date
- merchant_id
- transaction_id
- issue summary
- resolution status
- internal notes

This can begin in a simple log file or spreadsheet.

---

## 22. Suggested API Endpoints (Future Backend)

When moving to backend implementation, use endpoints such as:

Merchant Endpoints:

POST /api/merchants  
GET /api/merchants  
GET /api/merchants/:merchant_id  
PATCH /api/merchants/:merchant_id  
POST /api/merchants/:merchant_id/activate  
POST /api/merchants/:merchant_id/suspend  

Property Endpoints:

POST /api/merchants/:merchant_id/properties  
GET /api/merchants/:merchant_id/properties  

Transaction Endpoints:

POST /api/transactions  
GET /api/transactions  
GET /api/transactions/:transaction_id  

Dispute Endpoints:

POST /api/disputes  
GET /api/disputes  
PATCH /api/disputes/:dispute_id  

---

## 23. Example Merchant Create Payload

business_name: Example Lodge Namibia  
merchant_type: lodge  
country: Namibia  
city: Oshakati  
physical_address: 123 Example Road, Oshakati, Namibia  
contact_person: Jane Doe  
contact_email: info@examplelodge.com  
contact_phone: +264811234567  
website: https://examplelodge.com  
discount_policy:  
  apass_holder_discount_percent: 10  
  notes: Valid for accommodation bookings only  

---

## 24. v1 Operating Rule

Do not wait for full automation.

For the first merchants:

- collect details manually
- register manually
- verify discount policy manually
- log transactions manually if needed
- issue points according to recorded rules
- prove the model works in practice

This is the correct operational approach for early deployment.

---

## 25. Security and Data Handling

Even in v1, merchant records should be handled carefully.

Requirements:

- do not expose private contact data publicly without approval
- limit editing access to approved operators
- maintain backups of merchant records
- avoid deleting records silently
- record changes in notes or audit fields where possible

If using JSON files:

- restrict write access
- maintain version control carefully
- keep sensitive merchant data out of public-facing files where necessary

---

## 26. Recommended Public vs Internal Data Separation

Not all merchant data should be public.

Public data may include:

- business_name
- merchant_type
- city
- country
- website
- discount summary
- property names
- status if active

Internal-only data may include:

- contact_person
- contact_email
- contact_phone
- internal notes
- approved_by
- dispute history

This separation should be maintained from the beginning.

---

## 27. Success Criteria for the First Merchant

The first merchant onboarding should be considered successful only when:

- merchant is recorded correctly
- merchant is active
- discount policy is clear
- first transaction is logged
- points are issued or recorded
- merchant understands APASS participation
- APASS can reference the merchant as a live participant

The real milestone is not registration alone. It is verified operational participation.

---

## 28. Future Upgrade Path

Once 5 to 20 merchants are active, evolve into:

- backend-managed merchant records
- admin dashboard
- merchant self-service onboarding
- traveler-facing merchant directory
- automated points issuance
- dispute management workflows
- analytics and reporting
- optional on-chain merchant registry if justified

Do not move on-chain unless there is a clear operational reason.

---

## 29. Bottom Line

Use Hardhat for contracts.

Use backend or structured records for merchants.

For APASS v1, the correct path is:

- off-chain merchant onboarding
- clear merchant record structure
- clear discount policy
- clear property structure
- real transaction logging
- early support readiness
- fast rollout

That is how to onboard the first APASS merchants correctly.
