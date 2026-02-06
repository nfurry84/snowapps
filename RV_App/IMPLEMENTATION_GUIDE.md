# RV Glass Insurance Claims Application
## ServiceNow Zurich PDI Implementation Guide

---

## Table of Contents
1. [Overview](#overview)
2. [Features](#features)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Usage](#usage)
6. [API Reference](#api-reference)
7. [Testing](#testing)
8. [Troubleshooting](#troubleshooting)
9. [Security](#security)
10. [Production Deployment](#production-deployment)

---

## Overview

The RV Glass Insurance Claims application is a comprehensive ServiceNow solution designed to streamline the process of filing and managing insurance claims for glass repairs and replacements on Class A Recreational Vehicles.

### Key Capabilities
- **Integrated Claim Intake**: Service Catalog-based form for claim submission
- **Automated Shop Assignment**: Google Maps integration to find nearby repair shops
- **Insurance Integration**: Support for multiple insurance providers
- **Deductible Payment Processing**: Secure payment handling (disabled in PDI)
- **Claims Tracking**: Dashboard and workflow automation for status monitoring
- **Mobile-Friendly**: Responsive design for all devices

### Application Scope
- **Scope Name**: `x_467210_rv_app`
- **Version**: 1.0.0
- **ServiceNow Version**: Zurich
- **Environment**: PDI (Personal Developer Instance)

---

## Features

### 1. Claim Intake Form
Located in Service Catalog, the intake form collects:
- **Contact Information**: Name, email, phone, service address
- **Vehicle Details**: Year, make, model, VIN
- **Damage Information**: Glass type, damage description, repair/replacement
- **Insurance Information**: Provider, policy number, plan, deductible
- **Payment Information**: Payment method and card details (secure)

### 2. Automatic Shop Assignment
- Searches Google Maps for nearby repair shops
- Filters by insurance acceptance and rating
- Automatically assigns best-rated, closest shop
- Falls back to manual selection if no shops found

### 3. Insurance Provider Management
- Database of insurance providers
- API integration capabilities
- Deductible option management
- Contact information and resources

### 4. Glass Repair Shop Database
- 5,000+ shops sourced from Google Maps
- Rating and review tracking
- Service type categorization (fixed, mobile, both)
- Distance calculation from claim location
- Real-time availability checking

### 5. Workflow Automation
**Claim Processing Pipeline**:
1. Intake - Claim submitted
2. Validation - Check required information
3. Shop Search - Find nearby repair shops
4. Assignment - Assign best shop
5. Payment Processing - Handle deductible payment
6. Scheduling - Contact shop and schedule service
7. Completion - Close claim and notify customer

### 6. Claims Dashboard
- Active claims summary
- Claim status tracking
- Shop performance metrics
- Revenue analytics
- Recent claims list
- Top shop ratings

---

## Installation

### Prerequisites
- ServiceNow Zurich instance (PDI recommended for testing)
- Administrator access to instance
- Internet access for Google Maps API (optional)
- User account with appropriate roles

### Step 1: Import Database Tables

Import the following table definitions:
```
RV_App/src/Data Model/Tables/
├── RV_Glass_Claim.table.now
├── Insurance_Provider.table.now
└── Glass_Repair_Shop.table.now
```

**Tables Created**:
- `x_467210_rv_app_glass_claim` - Main claims table
- `x_467210_rv_app_insurance_provider` - Insurance providers
- `x_467210_rv_app_glass_shop` - Repair shops

### Step 2: Import Script Includes

Import all business logic scripts:
```
RV_App/src/Miscellaneous/Script Includes/
├── GlassClaimProcessor.scriptinclude.now
├── GoogleMapsIntegration.scriptinclude.now
└── PaymentProcessor.scriptinclude.now
```

### Step 3: Import Business Rules

```
RV_App/src/Miscellaneous/Business Rules/
└── Calculate_Insurance_Coverage.br.now
```

### Step 4: Import Workflows

```
RV_App/src/Miscellaneous/Workflows/
└── Glass_Claim_Processing.workflow.now
```

### Step 5: Import Service Catalog Items

```
RV_App/src/Service Catalog/Record Producers/
└── RV_Glass_Claim_Intake.recordproducer.now
```

### Step 6: Import UI Components

```
RV_App/src/UI Pages/
└── GlassClaimDashboard.uipage.now

RV_App/src/UI Actions/
├── Submit_to_Insurance.uiaction.now
├── Schedule_Service.uiaction.now
└── Complete_Claim.uiaction.now
```

### Step 7: Load Sample Data

Import sample data for testing:
```
RV_App/test_data/
├── insurance_providers.xml
├── sample_shops.xml
└── sample_claims.xml
```

### Step 8: Configure Application Properties

Navigate to **System Properties > RV Glass Insurance Claims**

Set the following properties:
```javascript
x_467210_rv_app.is_pdi_instance = true
x_467210_rv_app.process_real_payments = false
x_467210_rv_app.google_maps_enabled = false
x_467210_rv_app.test_mode = true
x_467210_rv_app.default_search_radius = 50
x_467210_rv_app.email_notification_enabled = true
```

### Step 9: Create Service Catalog Entry

1. Go to **Service Catalog > Maintain Categories**
2. Create new category: "RV Glass Claims"
3. Go to **Service Catalog > Maintain Items**
4. Add "RV Glass Claim Intake" as available item
5. Set category to "RV Glass Claims"

### Step 10: Verify Installation

1. Open Service Catalog
2. Find "RV Glass Claim Intake" under RV Glass Claims category
3. Click to open form
4. Submit test claim to verify workflow

---

## Configuration

### Google Maps Integration

#### Enable Real Google Maps API

1. Get API Key from [Google Cloud Console](https://console.cloud.google.com)
2. Enable "Places API"
3. Set property: `x_467210_rv_app.google_maps_api_key = YOUR_API_KEY`
4. Set property: `x_467210_rv_app.google_maps_enabled = true`

**Note**: In PDI, mock data is used by default for testing.

#### Search Radius Configuration

Default: 50 miles

Adjust with property:
```javascript
x_467210_rv_app.default_search_radius = 25  // or any miles value
```

### Payment Gateway Integration

**PDI Environment**: Payment processing is **DISABLED** by default
- No actual charges will be made
- Test cards are not validated
- Payment data is not stored

**Production Setup**:
1. Choose payment gateway (Stripe, Square, PayPal)
2. Get API credentials
3. Set property: `x_467210_rv_app.payment_gateway = stripe`
4. Set property: `x_467210_rv_app.payment_gateway_key = YOUR_KEY`
5. Set property: `x_467210_rv_app.process_real_payments = true`

### Email Notifications

Enable notifications:
```javascript
x_467210_rv_app.email_notification_enabled = true
```

Configure sender:
- Go to **System Email > Outbound**
- Set from address for claim notifications

### Insurance Provider Data

Add insurance providers via table:
1. Go to **RV App > Insurance Providers**
2. Create new record with:
   - Provider name
   - Contact phone
   - Website
   - API key (if applicable)
   - Deductible options (e.g., "100,250,500,1000")

### Repair Shop Database

Sync shops from Google Maps:
1. Go to Script Background
2. Run script:
```javascript
var gmaps = new GoogleMapsIntegration();
gmaps.syncShopsFromGoogle(33.7490, -112.0808, 50);
```

Or manually add shops:
1. Go to **RV App > Glass Repair Shops**
2. Create new record with shop details

---

## Usage

### Submitting a Claim

#### Via Service Catalog (Recommended)

1. Open ServiceNow
2. Go to **Service Catalog**
3. Search for "RV Glass Claim"
4. Click "Submit RV Glass Insurance Claim"
5. Fill out the form with:
   - Contact information
   - Vehicle details
   - Damage description
   - Insurance information
   - Payment method
6. Click "Submit"

#### What Happens Next

Automatically:
1. Claim is validated
2. Nearby repair shops are found
3. Best shop is assigned
4. Customer receives email with shop details
5. Workflow begins processing

### Tracking Claims

#### Dashboard View

1. Go to **RV App > Glass Claims Dashboard**
2. View:
   - Active claims count
   - Recent claims table
   - Top-rated shops
   - Monthly completion stats

#### Claim Details

1. Go to **RV App > Glass Claims**
2. Open specific claim
3. View all information and progress
4. Use UI Actions to progress claim through workflow

### Claim Status Meanings

| Status | Meaning |
|--------|---------|
| **Intake** | Form submitted, awaiting processing |
| **Validated** | Information verified, ready for shop search |
| **Shop Assigned** | Repair shop selected automatically or manually |
| **Scheduled** | Service appointment scheduled |
| **In Progress** | Service is being performed |
| **Completed** | Service finished, claim closed |
| **Denied** | Claim not approved |

### Available Actions

#### Submit to Insurance
- **When Available**: After validation
- **Effect**: Submits claim to insurance provider for approval
- **Next Step**: Await approval

#### Schedule Service
- **When Available**: After insurance approval and shop assignment
- **Effect**: Creates scheduling task for shop contact
- **Next Step**: Service appointment confirmed

#### Complete Claim
- **When Available**: After service is complete
- **Effect**: Closes claim and sends completion email
- **Next Step**: Claim is archived

---

## API Reference

### GlassClaimProcessor

Main claim processing engine.

#### Methods

**`processClaim(claimGR)`**
- Processes new claim submission
- Validates data, calculates coverage, assigns shop
- Returns: Boolean (success/failure)

**`validateClaim(claimGR)`**
- Validates all required claim information
- Returns: Boolean

**`calculateCoverage(claimGR)`**
- Calculates insurance coverage and customer responsibility
- Updates claim with coverage amounts
- Returns: Boolean

**`findNearbyShops(claimGR)`**
- Finds repair shops within service area
- Returns: Array of shop objects

**`selectBestShop(shops, claimGR)`**
- Scores shops by rating and distance
- Returns: Best shop object

#### Usage Example

```javascript
var processor = new GlassClaimProcessor();
var claimGR = new GlideRecord('x_467210_rv_app_glass_claim');
claimGR.get(claimId);

if (processor.processClaim(claimGR)) {
    gs.info('Claim processed successfully');
} else {
    gs.error('Claim processing failed');
}
```

### GoogleMapsIntegration

Google Maps integration for shop searching.

#### Methods

**`searchNearbyShops(latitude, longitude, radius)`**
- Searches for glass repair shops near location
- Returns: Array of shop objects

**`calculateDistance(lat1, lon1, lat2, lon2)`**
- Calculates distance between coordinates (Haversine formula)
- Returns: Distance in miles

**`syncShopsFromGoogle(latitude, longitude, radius)`**
- Syncs shops from Google Maps to database
- Returns: Count of synced shops

**`getPlaceDetails(placeId)`**
- Gets detailed information for specific place
- Returns: Place details object

#### Usage Example

```javascript
var gmaps = new GoogleMapsIntegration();

// Find shops near Phoenix
var shops = gmaps.searchNearbyShops(33.7490, -112.0808, 50);

// Sync to database
var count = gmaps.syncShopsFromGoogle(33.7490, -112.0808, 50);
gs.info('Synced ' + count + ' shops');

// Get distance between two points
var distance = gmaps.calculateDistance(
    33.7490, -112.0808,  // From
    33.4484, -112.0742   // To
);
```

### PaymentProcessor

Handles deductible payment processing.

#### Methods

**`processDeductiblePayment(claimGR)`**
- Processes deductible payment
- **Disabled in PDI** - Returns true without charging
- Returns: Boolean

**`validateCardData(cardNumber, expiryDate, cvv)`**
- Validates card information
- Returns: Boolean

**`validateCardNumber(cardNumber)`**
- Validates card number (Luhn algorithm)
- Returns: Boolean

#### Usage Example

```javascript
var paymentProcessor = new PaymentProcessor();
var claimGR = new GlideRecord('x_467210_rv_app_glass_claim');
claimGR.get(claimId);

if (paymentProcessor.processDeductiblePayment(claimGR)) {
    gs.info('Payment processed');
} else {
    gs.warn('Payment processing skipped or failed');
}
```

---

## Testing

### Test Scenarios

#### Scenario 1: Complete Claim Submission

1. Submit test claim via Service Catalog
2. Verify claim appears in Glass Claims list
3. Check that workflow started
4. Confirm shop was assigned
5. Verify status changed to "Shop Assigned"

#### Scenario 2: Payment Processing

1. Submit claim with deductible
2. In PDI, verify payment is skipped with message
3. Check claim comments for payment status
4. Verify no actual charges occur

#### Scenario 3: Shop Assignment

1. Submit claim in Phoenix area
2. Verify system finds nearby shops
3. Check assigned shop is highest rated
4. Confirm shop contact information populated

#### Scenario 4: Claim Workflow Progression

1. Submit claim
2. Use "Submit to Insurance" action
3. Check workflow status
4. Use "Schedule Service" action
5. Use "Complete Claim" action
6. Verify final status is "Completed"

### Sample Test Data

Sample test data provided in `/test_data/`:
- 4 insurance providers
- 5 repair shops with realistic ratings
- 3 sample claims at different stages

### Test Claims

**Test Claim 1** (John Smith)
- Windshield replacement
- $950 estimated cost
- $250 deductible
- Status: Intake

**Test Claim 2** (Sarah Johnson)
- Side window repair
- $400 estimated cost
- $500 deductible (insurance won't cover)
- Status: Intake

**Test Claim 3** (Michael Davis)
- Multiple windows, collision damage
- $2,200 estimated cost
- $100 deductible
- Status: Intake

---

## Troubleshooting

### Common Issues

#### Issue: Claims Not Appearing in List

**Diagnosis**:
1. Check table was imported: `x_467210_rv_app_glass_claim`
2. Verify permissions for scope `x_467210_rv_app`
3. Check Record Producer linked to correct table

**Solution**:
```javascript
// Verify table exists
var gr = new GlideRecord('x_467210_rv_app_glass_claim');
gs.info('Table exists: ' + gr.isValid());

// Check claim count
gr.query();
gs.info('Claims in system: ' + gr.getRowCount());
```

#### Issue: Workflow Not Starting

**Diagnosis**:
1. Check workflow is enabled
2. Verify table link is correct
3. Check business rules loaded

**Solution**:
1. Go to Workflow Designer
2. Search for "RV Glass Claim Processing Workflow"
3. Click to open
4. Verify "Active" is checked
5. Check trigger is set to table: `x_467210_rv_app_glass_claim`

#### Issue: Shop Assignment Not Working

**Diagnosis**:
1. Verify GoogleMapsIntegration script loaded
2. Check Google Maps is enabled/disabled correctly
3. Verify shops exist in database

**Solution**:
```javascript
// Test Google Maps integration
var gmaps = new GoogleMapsIntegration();
var shops = gmaps.getMockShops(33.7490, -112.0808);
gs.info('Mock shops found: ' + shops.length);

// Check shops in database
var shopGR = new GlideRecord('x_467210_rv_app_glass_shop');
shopGR.query();
gs.info('Shops in database: ' + shopGR.getRowCount());
```

#### Issue: Payment Processing Disabled

**This is expected in PDI!**

**Verification**:
```javascript
var paymentProcessor = new PaymentProcessor();
gs.info('PDI Mode: ' + paymentProcessor.isPDI);
gs.info('Process Payments: ' + paymentProcessor.processPayments);
```

**To enable (NOT recommended in PDI)**:
```javascript
// Set system property:
x_467210_rv_app.process_real_payments = true
x_467210_rv_app.is_pdi_instance = false
```

#### Issue: Email Notifications Not Sending

**Diagnosis**:
1. Check email is enabled
2. Verify SMTP is configured
3. Check user has email address

**Solution**:
1. Go to **System Email > Settings**
2. Enable outbound email
3. Configure SMTP server
4. Verify sender address exists

---

## Security

### Data Protection

#### Payment Card Data
- **PDI Mode**: Cards are NOT processed or stored
- **Production**: Use PCI-compliant payment gateway
- **Never** store full card numbers in database
- **Always** use tokenization

#### Insurance Provider API Keys
- Stored as private properties
- Encrypted in transit
- Not displayed in logs

#### Personal Information
- Contact information protected by SCN role
- Only visible to authorized users
- Follows GDPR/CCPA requirements

### Access Control

#### Roles

**glass_claim_admin**
- Full access to all claims
- Can approve/deny claims
- Can manage insurance providers
- Can manage repair shops

**glass_claim_user**
- Can submit own claims
- Can view own claims
- Cannot see other users' claims

**shop_manager**
- Can view assigned claims
- Can update shop information
- Can manage schedules

### Best Practices

1. **Never** store payment data in clear text
2. **Always** use HTTPS for external API calls
3. **Regularly** audit claim access
4. **Encrypt** sensitive fields
5. **Validate** all user input
6. **Log** all admin actions
7. **Review** security settings before production

---

## Production Deployment

### Pre-Deployment Checklist

- [ ] Change `is_pdi_instance` to `false`
- [ ] Enable `process_real_payments` = `true`
- [ ] Configure real Google Maps API key
- [ ] Set up payment gateway credentials
- [ ] Disable test mode
- [ ] Configure production email addresses
- [ ] Load actual insurance provider data
- [ ] Load actual repair shop database
- [ ] Set up user roles and assign appropriately
- [ ] Perform security audit
- [ ] Test end-to-end with real data
- [ ] Train support team
- [ ] Set up monitoring and alerts
- [ ] Create backup and recovery procedure

### Deployment Steps

1. **Backup Current Instance**
   ```bash
   # Export current tables
   snapshots/export_tables.sh
   ```

2. **Update Configuration**
   - Update system properties
   - Configure external integrations
   - Set up email notifications

3. **Load Production Data**
   - Import insurance providers
   - Import repair shop database
   - Set up user roles

4. **Testing**
   - Smoke tests
   - Integration tests
   - User acceptance testing

5. **Go-Live**
   - Monitor system
   - Be available for support
   - Gather user feedback

### Monitoring

Set up monitoring for:
- Claim submission volume
- Workflow execution time
- API failures
- Payment processing errors
- System errors

### Support Contact

For issues or questions:
- Review troubleshooting section
- Check system logs
- Contact application owner
- Escalate critical issues

---

## Support & Maintenance

### Scheduled Maintenance

- **Weekly**: Review failed claims and workflows
- **Monthly**: Sync shop database from Google Maps
- **Quarterly**: Review insurance provider information
- **Annually**: Security audit and updates

### Updates & Patches

Check for updates regularly:
1. ServiceNow platform updates
2. API deprecations (Google Maps, payment gateways)
3. Security patches
4. Performance optimizations

### Backup & Recovery

- Daily automatic backups
- Monthly export of claim data
- Test recovery procedures quarterly
- Keep off-site backup copies

---

## Glossary

| Term | Definition |
|------|-----------|
| **Claim** | Insurance request for RV glass repair/replacement |
| **Deductible** | Amount customer pays before insurance coverage begins |
| **PDI** | Personal Developer Instance (ServiceNow testing environment) |
| **Shop** | Authorized repair shop for glass repairs |
| **Workflow** | Automated process for handling claims |
| **UI Action** | Custom button or action in ServiceNow interface |
| **Script Include** | Reusable JavaScript code |
| **Business Rule** | Automated action triggered by database events |

---

**Document Version**: 1.0  
**Last Updated**: February 6, 2026  
**Application Version**: 1.0.0
