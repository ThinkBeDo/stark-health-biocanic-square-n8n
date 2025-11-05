# **Stark Health - Biocanic Integration | Project Pillar Document**

**Last Updated:** November 5, 2025 - Session 2  
**Project Status:** Phase 4 - Production Deployment (Session Tracking MVP 90% Complete)  
**ClickUp Task:** [86b736dq7](https://app.clickup.com/t/86b736dq7)

---

## **📋 Table of Contents**

1. Project Overview
2. Critical Credentials & Endpoints
3. Project Phase Checklist
4. Technical Architecture
5. Lessons Learned
6. Testing Protocols
7. Emergency Reference

---

## **🎯 Project Overview**

**Business Problem:** Stark Health manually creates/decrements session trackers in Biocanic when clients purchase subscriptions (12x, 16x, 20x sessions/month) or check out for sessions.

**Solution:** n8n middleware that:

- Receives Square webhooks for subscription purchases
- Auto-creates Biocanic session trackers + stores mapping in Supabase
- Receives Square webhooks for session checkouts
- Auto-decrements tracker + updates database
- Alerts when sessions depleted for renewal

**Success Metrics:**

- **Subscription Path:** ✅ 100% Complete (Nov 5, 2025)
- **Order Path:** 🔄 60% Complete (Router logic fixed, testing needed)
- **Overall MVP:** 90% Complete

---

## **🔐 Critical Credentials & Endpoints**

### **System Logins**

```
Biocanic Portal:
  Email: Jcheng@stark.health
  Password: StarkEMR2025!

Square Dashboard:
  Email: Jcheng@stark.health
  Password: JustinSQUARE2025!
```

### **API Credentials**

```
Square Production Token:
  EAAAlwJZtpxkp9IjGK3bS1zzzooJlEAWMc0MWRC34mPuVL6WnSWpAhaZfTpPYcos

Biocanic API:
  Endpoint: https://pxanvhep9a.execute-api.us-west-2.amazonaws.com/default/ghl-dev-api
  Header x-api-key: 32BEN163H02Hfn4mjPeZH341WgwBrYBQ3Yir0yMM
  Body apiKey: e79e20bf-1ed7-4781-a3f8-94fb746a09d0

Supabase Database:
  Connection: postgresql://postgres.ksewrswkhxrpqofzxkqk:password@aws-1-us-east-2.pooler.supabase.com:6543/postgres
  Project ID: ksewrswkhxrpqofzxkqk
  Table: session_trackers
  ⚠️ MUST use Transaction Pooler (port 6543) NOT direct connection (5432)

n8n Workflow:
  Railway URL: https://postgres-production-25f0.up.railway.app
  Workflow ID: XFhRJ33CWSaykaKG
  Webhook: https://postgres-production-25f0.up.railway.app/webhook/square-webhook
```

### **Test Client**

```
Biocanic Test Client:
  Email: tlafleur@hphi.life
  Name: Tyler Lafleur (Tyler L TEST)
```

---

## **✅ Project Phase Checklist**

### **Phase 1: Initial Setup & Discovery ✅ COMPLETE**

- Created n8n workflow on Railway
- Registered Square webhook (subscription.created, order.updated)
- Set up Supabase database + session_trackers table
- Obtained all 10 Square subscription plan variation IDs
- Created test client in Biocanic
- Resolved Biocanic API 403 error (wrong endpoint in docs)

### **Phase 2: Workflow Development ✅ COMPLETE**

- Built Router node to separate subscription vs order paths
- Created "Map Subscription to Sessions" node (10 plan mappings)
- Implemented Square API nodes (customer + subscription details)
- Built "Prepare Subscription Data" consolidation node
- Connected Biocanic tracker creation API
- Connected Supabase INSERT operation
- Fixed Router double-triggering with IF nodes
- Added customer data fetch from Square API
- Added subscription plan details fetch from Catalog API

### **Phase 3: Testing & Validation ✅ COMPLETE**

- Fixed Supabase connection (pooler vs direct)
- Tested subscription purchase flow (execution #7786)
- Verified Biocanic tracker creation in portal
- Verified database record creation with accurate data
- Fixed all 6 field expression issues in Prepare Subscription Data node
- Validated duplicate prevention logic (ON CONFLICT)

### **Phase 4: Production Deployment 🔄 IN PROGRESS (90%)**

- [x] Production credentials configured
- [x] **Subscription path fully tested and validated** ✅ (Nov 5, 2025 - Session 2)
- [x] Database duplicate prevention working
- [x] **catalog_object_id mapping implemented** (workaround for inaccessible subscription API)
- [ ] **Add remaining 9 catalog_object_id mappings** (currently only 1/10 mapped)
- [ ] **URGENT:** Complete order.updated path configuration
- [ ] Test session checkout/decrement flow
- [ ] Final end-to-end testing of both paths
- [ ] Client handoff documentation

**Subtasks:**

- [x] Fixed plan_variation_id null issue using catalog_object_id approach (Session 2)
- [ ] Add remaining catalog mappings for all 10 subscription types
- [ ] Fix "Extract Order Data" node field mappings
- [ ] Implement database lookup by customer email
- [ ] Add Biocanic logToSessionTracker API call
- [ ] Add database UPDATE to decrement remaining_sessions
- [ ] Add IF node to check if sessions = 0
- [ ] Add notification when sessions depleted

### **Phase 5: Additional Automation Triggers ⏸️ PAUSED (Client Requested)**

- [ ] #1: DEXA Initial Appointment → Schedule Review Task (7 days)
- [ ] #2: DEXA File Upload → Create Data Entry + Booking Tasks
- [ ] #3: DEXA Review Completed → Schedule CC Check-In (30 days)
- [ ] #4: CC Check-In Completed → Create Recurring Follow-Up (60 days)
- [ ] #5: DEXA Subsequent Appointment → Schedule Review Task
- [ ] #6: DEXA File Upload Subsequent → Data Entry + Booking
- [ ] #7: ROC Appointment Completed → Schedule Labs (7 days)
- [ ] #8: High Priority Tag → Same-Day Review Task
- [ ] #9: Custom HTC 6-Month Reminder → Check-In Task
- [ ] #10: Missed/No-Show Appointment → Auto Follow-Up Sequence

**Note:** Phase 5 requires completion of Phase 4 first. These are separate workflows extending beyond session tracking.

---

## **🗺️ Technical Architecture**

### **Database Schema**

```sql
CREATE TABLE session_trackers (
    id SERIAL PRIMARY KEY,
    customer_email TEXT NOT NULL,
    customer_name TEXT,
    square_subscription_id TEXT UNIQUE NOT NULL,
    subscription_type TEXT NOT NULL,
    biocanic_tracker_name TEXT NOT NULL,
    initial_sessions INTEGER NOT NULL,
    remaining_sessions INTEGER NOT NULL,
    plan_variation_id TEXT NOT NULL,  -- NOTE: Now stores catalog_object_id
    customer_id TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### **Square Catalog Object IDs (Production)**

**⚠️ CRITICAL:** Use `catalog_object_id` for mapping, NOT `plan_variation_id`

| Service Type | Sessions | Catalog Object ID | Status |
|--------------|----------|-------------------|--------|
| Master Coach Training | 12x | 5LD3NMMI6ZRZD22NRINNDIEH | ✅ Mapped |
| Personal Training | 12x | TBD | ⏳ Need to test |
| Personal Training | 16x | TBD | ⏳ Need to test |
| Personal Training | 20x | TBD | ⏳ Need to test |
| Master Coach Training | 16x | TBD | ⏳ Need to test |
| Master Coach Training | 20x | TBD | ⏳ Need to test |
| Chiro Mobility | 12x | TBD | ⏳ Need to test |
| Chiro Mobility | 16x | TBD | ⏳ Need to test |
| Chiro Mobility | 20x | TBD | ⏳ Need to test |
| Chiro Standard | 8x | TBD | ⏳ Need to test |

**To get catalog_object_id:** Test purchase each subscription type once and check webhook `line_items[0].catalog_object_id`

### **Router Logic (CRITICAL)**

**Correct Routing Logic:**

```javascript
// Check order state + subscription header
order.state === "OPEN" && headers['square-subscription-id'] exists → Subscription Path
order.state === "COMPLETED" && headers['square-subscription-id'] exists → Order Path

// DO NOT route by item name - subscription packages have "subscription" in name
// DO NOT route by item_type alone - both paths may have CREDIT_PACKAGE items
```

### **Working Node Reference Syntax**

```javascript
// Correct syntax for accessing upstream node data:
{{ $('Square: Get Customer Details').item.json.customer.email_address }}
{{ $('Map Subscription to Sessions').item.json.subscription_type }}
{{ $('Square Webhook Receiver').item.json.headers['square-subscription-id'] }}
```

**Router Node (Switch - Rules Mode):**

```javascript
// Rule 1 - Subscription Creation Path
$json.order.line_items.some(item => item.name.toLowerCase().includes('subscription')) && $('Square Webhook Receiver').item.json.headers['square-subscription-id']

// Rule 2 - Session Checkout Path  
$json.order.state === 'COMPLETED' && $('Square Webhook Receiver').item.json.headers['square-subscription-id']\n\n// CRITICAL: No {{ }} brackets in Rules mode!
// Set operator to: Boolean → true
// Enable: Convert types where required (in Settings tab)
```

### **Prepare Subscription Data Node (Working Configuration)**

**Mode:** Manual Mapping (NOT Expression)

```javascript
// Fields to Set:
customer_email: {{ $('Square: Get Customer Details').item.json.customer.email_address }}
customer_name: {{ $('Square: Get Customer Details').item.json.customer.given_name }} {{ $('Square: Get Customer Details').item.json.customer.family_name }}
subscription_type: {{ $('Square: Get Catalog Item').item.json.related_objects.find(obj => obj.type === 'ITEM').item_data.name }}
subscription_id: {{ $('Extract Subscription Data').item.json.subscription_id }}
plan_variation_id: {{ $('Square: Get Order for Customer ID').item.json.order.line_items[0].catalog_object_id }}
customer_id: {{ $('Square: Get Order for Customer ID').item.json.order.customer_id }}

// NOTE: No = prefix in Manual Mapping mode
// NOTE: plan_variation_id actually contains catalog_object_id (field name kept for DB compatibility)
```

### **Map Subscription to Sessions Node (Working Code)**

```javascript
const catalogObjectId = $json.plan_variation_id; // Contains catalog_object_id

const catalogMapping = {
  // Master Coach Training
  '5LD3NMMI6ZRZD22NRINNDIEH': { sessions: 12, name: 'Master Coach Training 12x' },
  
  // TODO: Add remaining 9 subscription types after testing
  // Personal Training 12x: 'CATALOG_ID': { sessions: 12, name: 'Personal Training 12x' },
  // etc...
};

const mapping = catalogMapping[catalogObjectId];
if (!mapping) {
  throw new Error(`Unknown catalog object ID: ${catalogObjectId}`);
}

const trackerName = `${mapping.name} - Sessions`;
const tracker_id = `${$json.customer_email}_${trackerName}`;

return {
  json: {
    ...($json),
    sessionCount: mapping.sessions,
    subscriptionType: mapping.name,
    trackerName: trackerName,
    tracker_id: tracker_id
  }
};
```

---

## **🎓 Lessons Learned - DO NOT REPEAT**

### **❌ WHAT DOESN'T WORK**

1. **Database Connection:**
   - ❌ Direct Supabase: `db.projectid.supabase.co:5432`
   - ✅ Use Transaction Pooler: `aws-1-us-east-2.pooler.supabase.com:6543`
   - **Why:** IPv6 issues cause ENETUNREACH errors

2. **Router Logic:**
   - ❌ Check webhook event type (`$json.body.type`)
   - ❌ Check line item name for word "subscription"
   - ✅ Check order state (OPEN vs COMPLETED) + subscription header
   - **Why:** Square sends order.updated for both subscriptions AND checkouts

3. **Data Mapping:**
   - ❌ Use `$json.field_name` references
   - ✅ Use `$('Node Name').item.json.field_name`
   - **Why:** Generic references break across multiple nodes

4. **Biocanic API:**
   - ❌ Trust documentation endpoint (had copy/paste error)
   - ✅ Verify endpoint with support first
   - **Why:** Wrong endpoint caused weeks of 403 errors

5. **Router Node Configuration in n8n Switch (Rules Mode):**
   - ❌ Use `{{ }}` brackets around expressions in Rules mode
   - ❌ Use string comparison when you need boolean evaluation
   - ✅ Remove `{{ }}` brackets - expressions evaluate directly in Rules mode
   - ✅ Set operator to "Boolean" → "true" for conditional logic
   - ✅ Enable "Convert types where required" in Settings tab if needed
   - **Why:** Template literal syntax `{{ }}` returns strings, not booleans. Switch node Rules mode evaluates expressions natively.

6. **Square Subscription API Access (Session 2 Discovery):**
   - ❌ Try to access subscription details via webhook subscription ID
   - ❌ Try to get `subscription_plan_data` from ITEM_VARIATION objects
   - ❌ Expect related objects to contain subscription plan details
   - ✅ **Solution:** Map `catalog_object_id` directly to sessions
   - **Why:**
     - Webhook header `square-subscription-id` contains webhook ID (`wbhk_*`), not subscription ID
     - Square Subscription API returns 400 error with webhook IDs
     - ITEM_VARIATION objects don't contain `subscription_plan_data`
     - Related objects from catalog fetch don't include subscription details

### **✅ PROVEN PATTERNS**

1. **Biocanic Tracker Creation (Working)**

```json
POST https://pxanvhep9a.execute-api.us-west-2.amazonaws.com/default/ghl-dev-api
Headers: {
  "x-api-key": "32BEN163H02Hfn4mjPeZH341WgwBrYBQ3Yir0yMM"
}
Body: {
  "eventType": "createSessionTracker",
  "trackerName": "Master Coach Training 12x - Sessions",
  "maxValue": 12,
  "email": "tlafleur@hphi.life",
  "apiKey": "e79e20bf-1ed7-4781-a3f8-94fb746a09d0"
}
```

2. **Database Duplicate Prevention**

```sql
INSERT INTO session_trackers (...)
VALUES (...)
ON CONFLICT (square_subscription_id) DO NOTHING;
```

3. **Square Webhook Data Structure**

```json
{
  "headers": {
    "square-subscription-id": "wbhk_abc123..."  // NOTE: This is WEBHOOK ID, not subscription ID!
  },
  "body": {
    "data": {
      "object": {
        "order_updated": {
          "state": "COMPLETED",  // Use for routing: OPEN vs COMPLETED
          "line_items": [{
            "catalog_object_id": "5LD3NMMI6ZRZD22NRINNDIEH"  // USE THIS for mapping!
          }]
        }
      }
    }
  }
}
```

4. **Catalog Object ID Mapping Pattern (Session 2 Solution)**

```javascript
// Build mapping table by testing each subscription type once
// Use catalog_object_id as the unique identifier (not plan_variation_id)
const catalogMapping = {
  'CATALOG_OBJECT_ID': { sessions: X, name: 'Subscription Name' }
};

// Extract from webhook order data
const catalogObjectId = $('Square: Get Order for Customer ID').item.json.order.line_items[0].catalog_object_id;
```

---

## **🧪 Testing Protocols**

### **Current Test Status (Nov 5, 2025 - Session 2)**

```
✅ Router node - Boolean routing working correctly
✅ Subscription path - End-to-end flow complete
✅ Catalog Object ID mapping - Working for Master Coach Training 12x
✅ Biocanic tracker creation - Tested and verified
✅ Database INSERT - Record created successfully
✅ Execution #8551 - Successful test run

⏳ TODO: Add remaining 9 catalog mappings (need to test each subscription type)
🔴 BLOCKER: Order.updated path not yet configured
```

### **Test Subscription Path (WORKING)**

1. Login to Square POS
2. Create order with subscription item (Master Coach Training 12x)
3. Use test card: `4111 1111 1111 1111`
4. Monitor n8n execution
5. **Expected Results:**
   - Webhook received
   - Routed to subscription.created path
   - catalog_object_id extracted: `5LD3NMMI6ZRZD22NRINNDIEH`
   - Mapped to 12 sessions
   - Biocanic tracker created: "Master Coach Training 12x - Sessions"
   - Database record inserted

**Latest Successful Test:** Execution #8551 (Nov 5, 2025 17:37)

### **Test Order Path (When Complete)**

1. Have existing subscription in database
2. Schedule appointment for test client in Square
3. Click "Take Payment" to trigger checkout
4. Verify only order.updated path executes
5. Verify tracker decremented in Biocanic
6. Verify database remaining_sessions decremented

### **Common Test Failures**

| Error | Likely Cause | Solution |
|-------|--------------|----------|
| "Invalid URL" | Expression not resolving | Check node reference syntax |
| "ENETUNREACH" | Wrong connection string | Use Transaction Pooler |
| 403 from Biocanic | Wrong endpoint/API key | Verify credentials |
| NULL values in DB | Field expression error | Check "Prepare Subscription Data" |
| Router double-triggers | IF node missing | Add IF node after Router |
| Unknown catalog ID | Mapping not yet added | Test subscription type to get ID |

---

## **🚨 Emergency Reference**

### **Stakeholders**

- **Client:** Justin Cheng (JCheng@stark.health)
- **Project Lead:** Talia Price (talia@vested.marketing)
- **Technical:** Tyler Lafleur (tlafleur@hphi.life)

### **If Workflow Fails**

1. Check n8n executions for error details
2. Check Railway instance status
3. Check Supabase database (auto-pause?)
4. Contact Tyler: tlafleur@hphi.life
5. If Biocanic API issue: help@biocanic.com

### **Quick Debug Commands**

```sql
-- Check recent database records
SELECT * FROM session_trackers ORDER BY created_at DESC LIMIT 5;

-- Find specific subscription
SELECT * FROM session_trackers WHERE square_subscription_id = '[id]';

-- Check remaining sessions
SELECT customer_email, biocanic_tracker_name, remaining_sessions 
FROM session_trackers 
WHERE remaining_sessions <= 3;
```

---

## **📝 Quick Start for New Claude Sessions**

1. **Load Context:** Read this entire pillar document
2. **Check Status:** Review ClickUp task 86b736dq7
3. **Verify Health:** Check n8n latest execution, Railway status, Supabase active
4. **Establish Goal:** Agree on ONE objective (e.g., "Add remaining catalog mappings")
5. **Work Incrementally:** One change → Test → Update ClickUp → Document
6. **Session Handoff:** Update pillar doc, create ClickUp comment, summarize next steps

---

## **🚀 Next Session Priorities**

### **Priority 1: Complete Catalog Mappings (30 minutes)**
Currently only 1 of 10 subscription types mapped. Need to:
1. Test each subscription type once in Square POS
2. Capture `catalog_object_id` from webhook
3. Add to mapping in "Map Subscription to Sessions" node
4. Document in table above

### **Priority 2: Build Order.Updated Path (1-2 hours)**
Complete session checkout/decrement functionality:
1. Fix "Extract Order Data" node field mappings
2. Add database lookup by customer email
3. Implement Biocanic `logToSessionTracker` API call
4. Add database UPDATE for remaining_sessions
5. Add IF node for sessions = 0 check
6. Add renewal notification

### **Priority 3: Final Testing & Handoff (30 minutes)**
1. End-to-end test both paths
2. Verify all database records accurate
3. Create client handoff documentation

---

**Document Maintenance:** Update after every significant session (2+ hours) or phase completion.

**Last Major Update:** Session 2 (Nov 5, 2025) - Subscription path 100% complete

---

This pillar document is the single source of truth. Reference at start of each session for full context.