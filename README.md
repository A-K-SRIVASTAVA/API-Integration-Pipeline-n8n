# 🔗 API Integration Pipeline — n8n

**An automated API integration pipeline built with n8n for retrieving orders, enriching them with customer data, handling pagination, routing orders by priority and region, processing batches, and handling API failures gracefully.**

---

# 📌 Project Overview

## Project Name

**API Integration Pipeline**

### Workflow Name

```text
Section 1 - API Integration Pipeline
```

### Platform

**n8n**

### Course

**n8n Foundations — Section 1**

### Estimated Time

```text
60 minutes
```

### Tag

```text
n8n102
```

---

# 🎯 Project Objective

The objective of this project is to build a reliable API integration pipeline using n8n.

The workflow retrieves order data from the n8n Academy API, retrieves customer information independently, enriches orders by matching `customer_id`, sends the enriched orders to a processing queue, routes orders based on customer subscription level and region, processes priority orders in controlled batches, and finalizes the pipeline.

The complete process is:

```text
Fetch
   ↓
Paginate
   ↓
Enrich
   ↓
Aggregate
   ↓
Queue
   ↓
Route
   ↓
Filter
   ↓
Batch
   ↓
Process
   ↓
Finalize
```

The project demonstrates how n8n can connect multiple APIs and apply conditional logic, data enrichment, batching, pagination, and error handling.

---

# 🏗️ Architecture

The workflow uses two independent API requests to retrieve orders and customer data. These datasets are then merged using `customer_id`.

```text
                              ┌─────────────────────┐
                              │    TriggerManual    │
                              │    Manual Trigger   │
                              └──────────┬──────────┘
                                         │
                         ┌───────────────┴────────────────┐
                         │                                │
                         ▼                                ▼
              ┌─────────────────────┐          ┌─────────────────────┐
              │      GetOrders      │          │    GetCustomers     │
              │    HTTP Request     │          │    HTTP Request     │
              │    Paginated API    │          │     Customer API    │
              └──────────┬──────────┘          └──────────┬──────────┘
                         │                                │
                         │                                │
                         └───────────────┬────────────────┘
                                         │
                                         ▼
                              ┌─────────────────────┐
                              │ MergeOrdersCustomers│
                              │        Merge        │
                              │ Match customer_id   │
                              └──────────┬──────────┘
                                         │
                         ┌───────────────┴────────────────┐
                         │                                │
                         ▼                                ▼
              ┌─────────────────────┐          ┌─────────────────────┐
              │   AggregateOrders   │          │ CheckSubscriptionTier│
              │      Aggregate      │          │         IF          │
              └──────────┬──────────┘          └──────────┬──────────┘
                         │                                │
                         ▼                         ┌──────┴──────┐
              ┌─────────────────────┐              │             │
              │ SendToOrdersQueue   │           TRUE          FALSE
              │     HTTP POST       │              │             │
              └─────────────────────┘              │             ▼
                                                   │      ┌─────────────────┐
                                                   │      │  RouteByRegion  │
                                                   │      │     Switch      │
                                                   │      └───────┬─────────┘
                                                   │              │
                                                   │      ┌───────┼────────┐
                                                   │      │       │        │
                                                   │      ▼       ▼        ▼
                                                   │    North   South    East/West
                                                   │
                                                   ▼
                                          ┌─────────────────────┐
                                          │   FilterDelivered   │
                                          │        Filter        │
                                          └──────────┬──────────┘
                                                     │
                                                     ▼
                                          ┌─────────────────────┐
                                          │  BatchPriorityOrders│
                                          │   Loop Over Items   │
                                          │      Batch = 5      │
                                          └──────────┬──────────┘
                                                     │
                                                     ▼
                                          ┌─────────────────────┐
                                          │ SendToPriorityQueue │
                                          │      HTTP POST      │
                                          └──────────┬──────────┘
                                                     │
                                                     │ Loop
                                                     └──────────┐
                                                                │
                                                     ┌──────────▼──────────┐
                                                     │        Done          │
                                                     └──────────┬──────────┘
                                                                │
                                                                ▼
                                                     ┌─────────────────────┐
                                                     │   FinalizePipeline  │
                                                     │      HTTP POST      │
                                                     └─────────────────────┘
```

---

# 🔄 Business Flow

The pipeline has two major processing paths after customer enrichment:

```text
                         MergeOrdersCustomers
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
                    ▼                           ▼
             Order Queue                  Priority Routing
                    │                           │
                    ▼                           ▼
           SendToOrdersQueue            CheckSubscriptionTier
                                                │
                                    ┌───────────┴───────────┐
                                    │                       │
                                 Enterprise              Other
                                    │                       │
                                    ▼                       ▼
                            FilterDelivered          RouteByRegion
                                    │
                                    ▼
                            BatchPriorityOrders
                                    │
                                    ▼
                           SendToPriorityQueue
                                    │
                                    ▼
                             FinalizePipeline
```

The non-enterprise branch is additionally routed by:

```text
North
South
East
West
Fallback
```

---

# 🏷️ Node Naming Convention

Every node follows a clear and descriptive naming convention.

| Node Type       | Node Name               |
| --------------- | ----------------------- |
| Manual Trigger  | `TriggerManual`         |
| HTTP Request    | `GetOrders`             |
| HTTP Request    | `GetCustomers`          |
| Merge           | `MergeOrdersCustomers`  |
| Aggregate       | `AggregateOrders`       |
| HTTP Request    | `SendToOrdersQueue`     |
| IF              | `CheckSubscriptionTier` |
| Switch          | `RouteByRegion`         |
| Filter          | `FilterDelivered`       |
| Loop Over Items | `BatchPriorityOrders`   |
| HTTP Request    | `SendToPriorityQueue`   |
| HTTP Request    | `FinalizePipeline`      |
| Edit Fields     | `SetFallbackCustomer`   |
| No Operation    | `ProcessNorth`          |
| No Operation    | `ProcessSouth`          |
| No Operation    | `ProcessEast`           |
| No Operation    | `ProcessWest`           |
| No Operation    | `ProcessFallback`       |

Clear node names make the workflow easier to understand, debug, maintain, and hand over to another developer.

---

# 🔐 Authentication

The Academy API requires authentication.

The workflow uses an n8n **Header Auth credential** for the Academy API key.

### Credential

```text
Credential Type:
Header Auth
```

```text
Header Name:
X-API-Key
```

The workflow also sends the assessment identifier using a separate HTTP header.

### Assessment Header

```text
Name:
X-Assessment-ID

Value:
[Your Assessment ID]
```

The authentication configuration is applied to the Academy HTTP Request nodes:

```text
GetOrders
GetCustomers
SendToOrdersQueue
SendToPriorityQueue
FinalizePipeline
```

---

# 📥 Step 1 — Fetch Orders

## Learning Objectives

This step demonstrates:

* HTTP Request configuration
* Header authentication
* API integration
* Pagination
* n8n expressions
* Multiple API requests

---

## 1.1 Create the Workflow

Create a new n8n workflow named:

```text
Section 1 - API Integration Pipeline
```

Add the tag:

```text
n8n102
```

Add a **Manual Trigger** node.

Rename it:

```text
TriggerManual
```

---

# 1.2 Configure GetOrders

Add an HTTP Request node.

Rename it:

```text
GetOrders
```

### Configuration

| Property        | Value                                                      |
| --------------- | ---------------------------------------------------------- |
| Method          | `GET`                                                      |
| URL             | `https://learn.app.n8n.cloud/webhook/course/n8n102/orders` |
| Authentication  | Generic Credential Type                                    |
| Credential Type | Header Auth                                                |
| Credential      | n8n Academy API Key                                        |

Enable:

```text
Send Headers
```

Add:

```text
X-Assessment-ID: [Your Assessment ID]
```

Initially, the endpoint returns:

```text
10 orders
```

---

# 🔄 Step 1.3 — Enable Pagination

The API returns orders in pages of 10.

Configure pagination on `GetOrders`.

### Pagination

```text
Pagination Mode:
Update a Parameter in Each Request
```

### Parameter

```text
Type:
Query
```

```text
Name:
page
```

```text
Value:
{{ $pageCount + 1 }}
```

### Pagination Complete Condition

```text
Complete When:
Response Is Empty
```

n8n will automatically make multiple requests:

```text
/orders?page=1 → 10 orders
/orders?page=2 → 10 orders
/orders?page=3 → 10 orders
/orders?page=4 → 10 orders
/orders?page=5 → 10 orders
/orders?page=6 → empty → STOP
```

The results are combined by n8n:

```text
10 + 10 + 10 + 10 + 10
             ↓
          50 orders
```

Therefore:

```text
GetOrders output = 50 items
```

---

# 📋 Step 2 — Fetch Customer Data

Add another HTTP Request node.

Rename it:

```text
GetCustomers
```

### Configuration

```text
Method:
GET
```

```text
URL:
https://learn.app.n8n.cloud/webhook/course/n8n102/customers
```

Use the same Header Auth credential.

Add:

```text
X-Assessment-ID:
[Your Assessment ID]
```

---

# ⚠️ Important Workflow Execution Concept

Do **not** connect:

```text
GetOrders → GetCustomers
```

because n8n executes nodes once for each incoming item.

If `GetOrders` produces 10 items and `GetCustomers` is connected after it, the customer request could execute once per order.

Conceptually:

```text
10 order items
      ↓
GetCustomers
      ↓
10 API calls
      ↓
10 × 10 customer records
      ↓
100 items
```

Instead, both API requests should connect directly to the trigger:

```text
                  TriggerManual
                  /           \
                 /             \
                ▼               ▼
          GetOrders       GetCustomers
                \               /
                 \             /
                  ▼           ▼
                 MergeOrdersCustomers
```

This allows both datasets to be fetched independently.

---

# 🔗 Step 3 — Enrich Orders with Customer Data

Add a **Merge** node.

Rename it:

```text
MergeOrdersCustomers
```

Connect:

```text
GetOrders → MergeOrdersCustomers
GetCustomers → MergeOrdersCustomers
```

### Configuration

```text
Mode:
Combine
```

```text
Combine By:
Matching Fields
```

### Fields to Match

```text
customer_id
```

### Output Type

```text
Enrich Input 1
```

The resulting order records contain customer information such as:

```text
customer_id
customer_name
customer_email
subscription
account_manager
```

The enriched structure conceptually becomes:

```text
Order
 │
 ├── order_id
 ├── status
 ├── region
 ├── customer_id
 │
 └── Customer Information
      ├── customer_name
      ├── customer_email
      ├── subscription
      └── account_manager
```

---

# 📦 Step 4 — Send Enriched Orders to the Order Queue

Add an **Aggregate** node.

Rename it:

```text
AggregateOrders
```

Configure:

```text
Aggregate:
All Item Data
```

Output field:

```text
enriched_orders
```

This converts the individual enriched order items into one collection.

---

# 📤 SendToOrdersQueue

Add an HTTP Request node after `AggregateOrders`.

Rename it:

```text
SendToOrdersQueue
```

### Configuration

```text
Method:
POST
```

```text
URL:
https://learn.app.n8n.cloud/webhook/course/n8n102/orders-queue
```

Use the Academy Header Auth credential.

Add:

```text
X-Assessment-ID:
[Your Assessment ID]
```

### JSON Body

```text
Name:
enriched_orders
```

```text
Value:
{{ $json.enriched_orders }}
```

The endpoint should confirm that:

```text
orders_queued = 50
```

---

# 🌳 Step 5 — Route Orders by Subscription

The queue response contains a summary rather than the original order records.

Therefore, routing must use the output from:

```text
MergeOrdersCustomers
```

not:

```text
SendToOrdersQueue
```

Connect:

```text
MergeOrdersCustomers
        ↓
CheckSubscriptionTier
```

The workflow now has:

```text
MergeOrdersCustomers
        │
        ├──────────────► AggregateOrders → SendToOrdersQueue
        │
        └──────────────► CheckSubscriptionTier
```

---

# 5.1 Check Subscription Tier

Add an **IF** node.

Rename it:

```text
CheckSubscriptionTier
```

### Condition

Value 1:

```text
{{ $json.subscription }}
```

Operation:

```text
equals
```

Value 2:

```text
Enterprise
```

The outputs are:

```text
TRUE
 ↓
Enterprise Orders
```

and:

```text
FALSE
 ↓
Non-Enterprise Orders
```

---

# 🗺️ Step 6 — Route Non-Enterprise Orders by Region

Connect the `FALSE` output of:

```text
CheckSubscriptionTier
```

to a **Switch** node.

Rename it:

```text
RouteByRegion
```

### Configuration

```text
Mode:
Rules
```

Add the following rules:

| Rule | Expression                        | Output |
| ---- | --------------------------------- | ------ |
| 1    | `{{ $json.region }} equals north` | North  |
| 2    | `{{ $json.region }} equals south` | South  |
| 3    | `{{ $json.region }} equals east`  | East   |
| 4    | `{{ $json.region }} equals west`  | West   |

Configure:

```text
Fallback Output:
Extra Output
```

This creates five possible destinations:

```text
North
South
East
West
Fallback
```

---

# 🏢 Step 7 — Process Enterprise Orders

Enterprise orders require priority processing.

Connect the `TRUE` output of:

```text
CheckSubscriptionTier
```

to:

```text
FilterDelivered
```

---

# 7.1 Filter Delivered Orders

Add a **Filter** node.

Rename it:

```text
FilterDelivered
```

Configure:

### Value 1

```text
{{ $json.status }}
```

### Operation

```text
is not equal to
```

### Value 2

```text
delivered
```

Only enterprise orders that have **not** been delivered continue to the priority queue.

The expected assessment result is:

```text
20 enterprise orders
        ↓
FilterDelivered
        ↓
12 orders
```

---

# 🔁 Step 8 — Batch Priority Orders

Sending every order to an external system simultaneously can create unnecessary load.

Use **Loop Over Items** to process priority orders in controlled batches.

Add:

```text
Loop Over Items
```

Rename it:

```text
BatchPriorityOrders
```

### Configuration

```text
Batch Size:
5
```

The processing flow becomes:

```text
FilterDelivered
      ↓
BatchPriorityOrders
      ↓
5 orders
      ↓
SendToPriorityQueue
      ↓
Back to BatchPriorityOrders
      ↓
Next 5 orders
```

The `Done` output fires after all batches have completed.

---

# 📤 Step 9 — Send Priority Orders

Add an HTTP Request node connected to the loop output.

Rename it:

```text
SendToPriorityQueue
```

### Configuration

```text
Method:
POST
```

```text
URL:
https://learn.app.n8n.cloud/webhook/course/n8n102/priority-queue
```

Use the Academy Header Auth credential.

Add:

```text
X-Assessment-ID:
[Your Assessment ID]
```

---

# 📦 Request Body

Use:

```text
JSON
```

Add:

### Order ID

```text
Name:
order_id
```

```text
Value:
{{ $json.order_id }}
```

### Customer ID

```text
Name:
customer_id
```

```text
Value:
{{ $json.customer_id }}
```

### Company Name

```text
Name:
company_name
```

```text
Value:
{{ $json.customer_name }}
```

### Subscription

```text
Name:
subscription
```

```text
Value:
{{ $json.subscription }}
```

Each successful request should return a response similar to:

```json
{
  "status": "priority_queued",
  "order_id": "ORD-005",
  "priority_level": "high",
  "processing_queue_time": "15 minutes"
}
```

---

# 🔄 Loop Connection

Connect:

```text
SendToPriorityQueue
        │
        └──────────────► BatchPriorityOrders
```

This creates the processing loop.

The loop continues until all priority orders have been processed.

The:

```text
Done
```

output fires only after all batches are complete.

---

# 🏁 Step 10 — Finalize the Pipeline

Connect the `Done` output of:

```text
BatchPriorityOrders
```

to an HTTP Request node.

Rename it:

```text
FinalizePipeline
```

### Configuration

```text
Method:
POST
```

```text
URL:
https://learn.app.n8n.cloud/webhook/course/n8n102/finalize
```

Use the Academy Header Auth credential.

Add:

```text
X-Assessment-ID:
[Your Assessment ID]
```

---

# 📊 Finalization Request

Use:

```text
JSON
```

Add:

### Total Orders

```text
Name:
total_orders
```

```text
Value:
{{ $('MergeOrdersCustomers').all().length }}
```

### Enterprise Count

```text
Name:
enterprise_count
```

```text
Value:
{{ $('FilterDelivered').all().length }}
```

### Batch Size

```text
Name:
batch_size
```

```text
Value:
{{ $('BatchPriorityOrders').params.batchSize }}
```

### Retry Enabled

```text
Name:
retry_enabled
```

```text
Value:
{{ true }}
```

Enable:

```text
Settings → Execute Once
```

---

# 🛡️ Step 11 — Error Handling

A reliable integration should not fail completely because of a temporary API problem.

The pipeline supports two levels of resilience:

```text
Retry
  +
Fallback
```

---

# 🔁 Retry On Fail

For API requests such as:

```text
GetOrders
GetCustomers
```

enable:

```text
Settings
   ↓
Retry On Fail
```

Recommended configuration:

```text
Max Tries:
3
```

```text
Wait Between Tries:
1000 ms
```

This allows transient failures to recover automatically.

Example:

```text
Attempt 1 → Failed
     ↓
Wait 1 second
     ↓
Attempt 2 → Failed
     ↓
Wait 1 second
     ↓
Attempt 3 → Success
```

---

# 🧯 Customer API Fallback

If the customer API remains unavailable, the workflow can use fallback customer information.

Open:

```text
GetCustomers
```

Go to:

```text
Settings → On Error
```

Select:

```text
Continue (using error output)
```

This exposes a red error output.

Connect the error output to an **Edit Fields** node.

Rename it:

```text
SetFallbackCustomer
```

Configure:

```text
customer_id = unknown
company_name = Unknown Company
subscription = Starter
account_manager = Unassigned
```

Connect:

```text
SetFallbackCustomer
        ↓
MergeOrdersCustomers
```

This allows the pipeline to continue with fallback information when customer enrichment fails.

---

# 🗺️ Regional Processing

The regional outputs are placeholders for region-specific processing in a real-world workflow.

```text
RouteByRegion
      │
      ├── North   → ProcessNorth
      ├── South   → ProcessSouth
      ├── East    → ProcessEast
      ├── West    → ProcessWest
      └── Fallback → ProcessFallback
```

These can later be connected to:

```text
Regional queues
Regional APIs
Databases
Notification systems
Business logic
```

For the prototype, connect each output to a **No Operation** node.

Use:

```text
ProcessNorth
ProcessSouth
ProcessEast
ProcessWest
ProcessFallback
```

---

# 📝 Workflow Documentation

Add a Sticky Note to the workflow.

Suggested documentation:

```text
# API Integration Pipeline

Owned By: Anubhav Kumar Srivastava
Last Updated: 27-Sept-2026

Fetches orders from the Academy API, enriches them with customer
data, routes by priority and region, and sends enterprise orders
to a priority queue in batches.

## Flow

1. GetOrders: Fetches paginated orders from the Academy API
2. GetCustomers: Fetches customer records in parallel with orders
3. MergeOrdersCustomers: Joins orders + customers on customer_id
4. AggregateOrders + SendToOrdersQueue: Aggregates and sends enriched orders
5. CheckSubscriptionTier: Splits enterprise vs other customers
6. FilterDelivered + BatchPriorityOrders: Filters and batches enterprise orders
7. SendToPriorityQueue: Posts priority orders to the priority queue
8. RouteByRegion: Routes non-enterprise orders by region
9. FinalizePipeline: Posts the final pipeline summary

## Error Handling

- Retry enabled for transient API failures
- Customer API fallback data available
- Regional fallback available for unsupported regions

## Dependencies

- n8n Academy API Key credential
- X-Assessment-ID

Part of: n8n Foundations Program Course N8N102, Section 1
```

---

# 🧪 Testing Checklist

## Data Retrieval

```text
☐ GetOrders executes successfully
☐ Pagination is enabled
☐ 50 orders are retrieved
☐ GetCustomers executes independently
☐ Customer data is available
```

## Data Enrichment

```text
☐ MergeOrdersCustomers executes successfully
☐ customer_id is used as the matching field
☐ Orders contain customer information
☐ customer_name is available
☐ customer_email is available
☐ subscription is available
☐ account_manager is available
```

## Order Queue

```text
☐ AggregateOrders creates enriched_orders
☐ SendToOrdersQueue executes successfully
☐ orders_queued = 50
```

## Priority Routing

```text
☐ CheckSubscriptionTier checks subscription
☐ Enterprise orders go to TRUE
☐ Non-enterprise orders go to FALSE
☐ FilterDelivered excludes delivered orders
☐ 12 priority orders remain
☐ BatchPriorityOrders uses batch size 5
☐ SendToPriorityQueue processes priority orders
```

## Regional Routing

```text
☐ North route configured
☐ South route configured
☐ East route configured
☐ West route configured
☐ Fallback route configured
```

## Finalization

```text
☐ Done output is connected
☐ FinalizePipeline executes once
☐ Total order count is correct
☐ Enterprise count is correct
☐ Batch size is correct
☐ Retry status is included
☐ Confirmation code received
```

## Error Handling

```text
☐ Retry On Fail configured
☐ Maximum retries configured
☐ Customer API error output configured
☐ Fallback customer data configured
☐ Regional fallback configured
```

---

# 🛠️ Troubleshooting

## Merge Node Shows 0 Items

Check:

```text
☐ GetOrders is connected to MergeOrdersCustomers
☐ GetCustomers is connected to MergeOrdersCustomers
☐ Both APIs execute successfully
☐ customer_id exists in both datasets
☐ Matching field name is exactly customer_id
```

Field names are case-sensitive.

---

## Pagination Returns Only 10 Orders

Verify:

```text
Pagination Mode:
Update a Parameter in Each Request
```

and:

```text
Type:
Query
```

and:

```text
Name:
page
```

and:

```text
Value:
{{ $pageCount + 1 }}
```

Also verify:

```text
Complete When:
Response Is Empty
```

---

## GetCustomers Returns 100 Items

This usually means:

```text
GetOrders
    ↓
GetCustomers
```

was used.

Because `GetOrders` produces multiple input items, `GetCustomers` can execute once for each input item.

Correct structure:

```text
          TriggerManual
          /           \
         ▼             ▼
   GetOrders      GetCustomers
         \             /
          ▼           ▼
       Merge
```

---

## Loop Never Completes

Check:

```text
☐ SendToPriorityQueue connects back to BatchPriorityOrders
☐ Done output is connected to FinalizePipeline
☐ Batch Size is greater than 0
☐ SendToPriorityQueue does not accidentally create additional items
```

---

## Error Output Does Not Appear

Verify:

```text
GetCustomers
      ↓
Settings
      ↓
On Error
      ↓
Continue (using error output)
```

The red error connector should then become available.

---

## Node Reference Not Found

n8n node references are case-sensitive.

Correct:

```text
$('GetOrders')
```

Incorrect:

```text
$('getOrders')
```

Also verify the node name has not been changed.

---

# 📊 Evaluation Criteria

| Criteria             | Requirement                                             |
| -------------------- | ------------------------------------------------------- |
| Orders Retrieved     | 50 orders fetched through pagination                    |
| Customer Data        | Customer API fetched independently                      |
| Data Enrichment      | Orders and customers merged using `customer_id`         |
| Order Queue          | 50 enriched orders queued                               |
| Subscription Routing | Enterprise and non-enterprise orders separated          |
| Regional Routing     | North, South, East, West and fallback routes configured |
| Priority Filtering   | Delivered enterprise orders excluded                    |
| Batch Processing     | Loop Over Items configured                              |
| Priority Queue       | Priority orders sent successfully                       |
| Error Handling       | Retry and customer fallback configured                  |
| Finalization         | Pipeline summary submitted                              |
| Confirmation         | Final confirmation code received                        |

---

# 🎯 Success Criteria

The pipeline is successfully completed when:

```text
✓ 50 orders are retrieved through pagination
✓ Customer data is retrieved independently
✓ Orders are enriched using customer_id
✓ 50 enriched orders are sent to the order queue
✓ Enterprise orders are separated from other customers
✓ Non-enterprise orders are routed by region
✓ Delivered enterprise orders are filtered out
✓ Priority orders are processed in controlled batches
✓ Priority orders reach the priority queue
✓ API retry handling is configured
✓ Customer API fallback is configured
✓ Regional fallback is configured
✓ FinalizePipeline executes after all batches complete
✓ Final confirmation code is received
```

---

# 📚 Skills Demonstrated

This project demonstrates practical experience with:

* n8n workflow automation
* HTTP Request nodes
* Header authentication
* API integration
* Pagination
* n8n expressions
* Parallel data retrieval
* Merge nodes
* Matching fields
* Data enrichment
* Aggregate
* IF conditions
* Switch routing
* Filtering
* Loop Over Items
* Batch processing
* API queue integration
* Retry On Fail
* Error outputs
* Fallback data
* Regional routing
* Workflow documentation
* Execution flow
* Multi-branch workflows

---

# 💡 Key Concepts Learned

## Pagination

Pagination allows a workflow to retrieve all records when an API limits the number of records returned per request.

```text
Page 1 → 10
Page 2 → 10
Page 3 → 10
Page 4 → 10
Page 5 → 10
Page 6 → Empty

Total = 50
```

---

## Parallel Data Fetching

Independent APIs should not unnecessarily be chained.

```text
              Trigger
              /     \
             ▼       ▼
         Orders   Customers
             \       /
              ▼     ▼
                Merge
```

This prevents the customer API from being executed once for every order item.

---

## Data Enrichment

The order dataset is enriched using customer information.

```text
Order Data
     +
Customer Data
     ↓
Match customer_id
     ↓
Enriched Order
```

---

## Conditional Routing

The IF node creates two possible paths:

```text
                Subscription
                     │
             ┌───────┴───────┐
             │               │
        Enterprise        Other
             │               │
             ▼               ▼
       Priority Flow     Region Flow
```

---

## Multi-Path Routing

The Switch node allows multiple processing destinations:

```text
                Region
                  │
        ┌─────────┼─────────┐
        │         │         │
       North    South     East
                             
        ┌─────────┴─────────┐
        │                   │
       West              Fallback
```

---

## Batch Processing

Loop Over Items prevents all records from being processed at once.

```text
12 Priority Orders
       ↓
Batch Size = 5
       ↓
5 + 5 + 2
       ↓
Priority Queue
```

---

## Retry Handling

Transient failures can be retried automatically.

```text
API Request
     ↓
  Failure
     ↓
   Retry
     ↓
  Success
```

This is useful for temporary network or service failures.

---

## Graceful Degradation

Fallback data prevents the entire workflow from stopping when customer enrichment fails.

```text
Customer API
     │
     ├── Success → Customer Data
     │
     └── Failure → Fallback Customer
                         │
                         ▼
                    Continue Flow
```

---

# 🔮 Future Improvements

A production implementation could replace the Academy APIs with real business systems.

## CRM Integration

```text
Orders API
    ↓
n8n
    ↓
CRM API
    ↓
Enriched Orders
```

## Message Queue

The priority queue could be replaced with:

```text
RabbitMQ
Kafka
AWS SQS
Azure Service Bus
```

## Database Storage

Enriched orders could be persisted in:

```text
MySQL
PostgreSQL
MongoDB
```

## Monitoring

Add:

```text
Metrics
Logs
Alerts
Execution Monitoring
```

to track API failures and workflow performance.

## Scheduled Execution

Replace the manual trigger with:

```text
Schedule Trigger
        ↓
Periodic Pipeline
```

For example:

```text
Every 15 minutes
       ↓
Fetch Orders
       ↓
Enrich
       ↓
Process
```

---

# 🏁 Final Result

The completed workflow creates an end-to-end API integration pipeline:

```text
┌──────────────────────┐
│      Orders API      │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│      GetOrders       │
│      Pagination      │
└──────────┬───────────┘
           │
           │
           │       ┌──────────────────────┐
           └──────►│    GetCustomers      │
                   │    Customer API       │
                   └──────────┬───────────┘
                              │
                              ▼
                   ┌──────────────────────┐
                   │ MergeOrdersCustomers │
                   │    customer_id       │
                   └──────────┬───────────┘
                              │
                 ┌────────────┴────────────┐
                 │                         │
                 ▼                         ▼
          AggregateOrders          CheckSubscriptionTier
                 │                         │
                 ▼                  ┌──────┴──────┐
        SendToOrdersQueue           │             │
                                  Enterprise     Other
                                      │             │
                                      ▼             ▼
                               FilterDelivered RouteByRegion
                                      │
                                      ▼
                               BatchPriorityOrders
                                      │
                                      ▼
                              SendToPriorityQueue
                                      │
                                      ▼
                               FinalizePipeline
```

The pipeline follows the complete API integration lifecycle:

```text
FETCH
  ↓
PAGINATE
  ↓
ENRICH
  ↓
AGGREGATE
  ↓
QUEUE
  ↓
ROUTE
  ↓
FILTER
  ↓
BATCH
  ↓
PROCESS
  ↓
FINALIZE
```

---

# 📁 Repository Structure

```text
api-integration-pipeline/
│
├── README.md
├─ section-1-api-integration-pipeline.json
├─workflow.png
```

---

# 📜 Project Notes

This project is intended for learning, experimentation, and demonstration of n8n API integration and workflow automation concepts.

The n8n Academy endpoints and associated course resources belong to their respective owners.

**Do not commit real API credentials, API keys, or assessment identifiers to a public repository.**
