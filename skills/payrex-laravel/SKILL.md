---
name: payrex-laravel
description: Use when working with the legionhq/laravel-payrex package or when building PayRex paymentintegrations in Laravel. Covers accepting payments (cards, GCash, Maya, BillEase, QR Ph, BDO Installment), Payment Intents, Checkout Sessions, PayRex Elements, webhookhandling, billing statements/invoicing, customer management, refunds, error handling, and testing. Philippine payment gateway.
compatible_agents:
  - Claude Code
  - Cursor
tags:
  - laravel
  - payments
  - payrex
  - gcash
  - maya
  - philippines
  - webhooks
  - checkout
  - billing
---

# PayRex for Laravel

Use this skill when helping developers integrate the `legionhq/laravel-payrex` package — an unofficial Laravel package for the PayRex payment platform (Philippines).

**Package:** `legionhq/laravel-payrex`
**Namespace:** `LegionHQ\LaravelPayrex\`
**Requirements:** PHP 8.3+ · Laravel 11+
**Docs:** https://payrex-laravel.netlify.app

## Installation

```bash
composer require legionhq/laravel-payrex
php artisan vendor:publish --tag="payrex-config"
```

**Required `.env` variables:**

```ini
PAYREX_PUBLIC_KEY=pk_test_...
PAYREX_SECRET_KEY=sk_test_...
PAYREX_WEBHOOK_ENABLED=true
PAYREX_WEBHOOK_SECRET=whsk_...
```

**Optional `.env` variables (with defaults):**

```ini
PAYREX_API_BASE_URL=https://api.payrexhq.com
PAYREX_CURRENCY=PHP
PAYREX_TIMEOUT=30
PAYREX_CONNECT_TIMEOUT=30
PAYREX_RETRIES=0
PAYREX_RETRY_DELAY=100
PAYREX_WEBHOOK_PATH=payrex/webhook
PAYREX_WEBHOOK_TOLERANCE=300
```

For the Billable Customer trait (links User model to PayRex customers):

```bash
php artisan vendor:publish --tag="payrex-migrations"
php artisan migrate
```

## Access Patterns

```php
use LegionHQ\LaravelPayrex\Facades\Payrex;
use LegionHQ\LaravelPayrex\PayrexClient;

// Facade
Payrex::paymentIntents()->create([...]);

// Dependency injection
public function store(Request $request, PayrexClient $client) {
    $client->paymentIntents()->create([...]);
}

// Service container
$client = app(PayrexClient::class);
```

## Resources and Methods

All amounts are in **cents** (₱100.00 = `10000`). Currency defaults to `PAYREX_CURRENCY` config and is sent automatically.

### Payment Intents

```php
Payrex::paymentIntents()->create(array $params, ?string $idempotencyKey = null): PaymentIntent
Payrex::paymentIntents()->retrieve(string $id): PaymentIntent
Payrex::paymentIntents()->cancel(string $id, ?string $idempotencyKey = null): PaymentIntent
Payrex::paymentIntents()->capture(string $id, array $params, ?string $idempotencyKey = null): PaymentIntent
```

**Create params:** `amount` (required, int, min 2000, max 5999999999), `payment_methods` (optional, array — defaults to account's enabled methods), `currency`, `description`, `statement_descriptor`, `metadata`, `return_url`, `customer_id`, `payment_method_options`

**PaymentIntent properties (camelCase):** `id`, `amount`, `amountReceived`, `amountCapturable`, `clientSecret`, `currency`, `description`, `lastPaymentError` (array), `latestPayment` (string|Payment), `nextAction` (array), `paymentMethodOptions` (array), `paymentMethods` (array), `statementDescriptor`, `status` (PaymentIntentStatus enum), `paymentMethodId`, `captureBeforeAt` (int timestamp), `customer` (string|Customer), `returnUrl`, `metadata`, `livemode`, `createdAt`, `updatedAt`

### Checkout Sessions

```php
Payrex::checkoutSessions()->create(array $params, ?string $idempotencyKey = null): CheckoutSession
Payrex::checkoutSessions()->retrieve(string $id): CheckoutSession
Payrex::checkoutSessions()->expire(string $id, ?string $idempotencyKey = null): CheckoutSession
```

**Create params:** `line_items` (required, array of `{name, amount, quantity, description?, image?}`), `success_url` (required), `cancel_url` (required), `payment_methods`, `currency`, `customer_reference_id`, `description`, `expires_at`, `billing_details_collection`, `submit_type`, `statement_descriptor`, `payment_method_options`, `metadata`

**CheckoutSession properties:** `id`, `amount`, `clientSecret`, `currency`, `customerReferenceId`, `description`, `status` (CheckoutSessionStatus enum), `url` (redirect customer here), `lineItems` (array), `successUrl`, `cancelUrl`, `paymentIntent` (string|PaymentIntent), `paymentMethods` (array), `paymentMethodOptions` (array), `billingDetailsCollection`, `submitType`, `statementDescriptor`, `expiresAt`, `metadata`, `livemode`, `createdAt`, `updatedAt`

### Payments

```php
Payrex::payments()->retrieve(string $id): Payment
Payrex::payments()->update(string $id, array $params): Payment
```

**Update params:** `description`, `metadata`

**Payment properties:** `id`, `amount`, `amountRefunded`, `billing` (array — `name`, `email`, `phone`, `address`), `currency`, `description`, `fee`, `netAmount`, `paymentIntentId`, `paymentMethod` (array — `type`, then method-specific nested object), `status` (PaymentStatus enum), `customer` (string|Customer), `pageSession` (array), `refunded` (bool), `metadata`, `livemode`, `createdAt`, `updatedAt`

### Refunds

```php
Payrex::refunds()->create(array $params, ?string $idempotencyKey = null): Refund
Payrex::refunds()->update(string $id, array $params): Refund
```

**Create params:** `payment_id` (required), `amount` (required, int), `reason` (required — use RefundReason enum values), `currency`, `description`, `remarks`, `metadata`

**Refund properties:** `id`, `amount`, `currency`, `status` (RefundStatus enum), `description`, `reason` (RefundReason enum), `remarks`, `paymentId`, `metadata`, `livemode`, `createdAt`, `updatedAt`

> QR Ph (`qrph`) payments do NOT support refunds.

### Customers

```php
Payrex::customers()->create(array $params, ?string $idempotencyKey = null): Customer
Payrex::customers()->list(array $params = []): PayrexCollection
Payrex::customers()->retrieve(string $id): Customer
Payrex::customers()->update(string $id, array $params): Customer
Payrex::customers()->delete(string $id): DeletedResource
```

**Create params:** `name` (required), `email` (required), `currency`, `billing_details` (`phone`, `address`), `billing_statement_prefix`, `next_billing_statement_sequence_number`, `metadata`

**List filters:** `name`, `email`, `metadata` (in addition to pagination params `limit`, `before`, `after`)

**Customer properties:** `id`, `name`, `email`, `currency`, `billingStatementPrefix`, `nextBillingStatementSequenceNumber`, `billing` (array), `metadata`, `livemode`, `createdAt`, `updatedAt`

### Billing Statements

```php
Payrex::billingStatements()->create(array $params, ?string $idempotencyKey = null): BillingStatement
Payrex::billingStatements()->list(array $params = []): PayrexCollection
Payrex::billingStatements()->retrieve(string $id): BillingStatement
Payrex::billingStatements()->update(string $id, array $params): BillingStatement
Payrex::billingStatements()->delete(string $id): DeletedResource
Payrex::billingStatements()->finalize(string $id, ?string $idempotencyKey = null): BillingStatement
Payrex::billingStatements()->void(string $id, ?string $idempotencyKey = null): BillingStatement
Payrex::billingStatements()->markUncollectible(string $id, ?string $idempotencyKey = null): BillingStatement
Payrex::billingStatements()->send(string $id, ?string $idempotencyKey = null): BillingStatement
```

**Create params:** `customer_id` (required), `currency`, `description`, `billing_details_collection`, `metadata`, `payment_settings` (`payment_methods`, `payment_method_options`)

**Update params:** `payment_settings` (required by PayRex API on every update), `customer_id`, `description`, `due_at`, `billing_details_collection`, `metadata`

**Lifecycle:** `draft` → `open` (finalize) → `paid` | `void` | `uncollectible`

**BillingStatement properties:** `id`, `amount`, `currency`, `customerId`, `status` (BillingStatementStatus enum), `description`, `url` (available when open), `billingDetailsCollection`, `dueAt`, `lineItems` (array), `customer` (string|Customer), `paymentIntent` (string|PaymentIntent), `paymentSettings` (array), `metadata`, `livemode`, `createdAt`, `updatedAt`

### Billing Statement Line Items

```php
Payrex::billingStatementLineItems()->create(array $params, ?string $idempotencyKey = null): BillingStatementLineItem
Payrex::billingStatementLineItems()->update(string $id, array $params): BillingStatementLineItem
Payrex::billingStatementLineItems()->delete(string $id): DeletedResource
```

**Create params:** `billing_statement_id` (required), `description` (required), `unit_price` (required, int), `quantity` (required, int)

Line items can only be modified while the billing statement is in `draft` status.

### Payout Transactions

```php
Payrex::payoutTransactions()->list(string $payoutId, array $params = []): PayrexCollection
```

Read-only. First argument is the payout ID (`po_` prefix), unlike other list methods.

**PayoutTransaction properties:** `id`, `amount`, `netAmount`, `transactionId`, `transactionType` (`payment`, `refund`, `adjustment`)

### Webhooks (API Resource)

```php
Payrex::webhooks()->create(array $params, ?string $idempotencyKey = null): WebhookEndpoint
Payrex::webhooks()->list(array $params = []): PayrexCollection
Payrex::webhooks()->retrieve(string $id): WebhookEndpoint
Payrex::webhooks()->update(string $id, array $params): WebhookEndpoint
Payrex::webhooks()->delete(string $id): DeletedResource
Payrex::webhooks()->enable(string $id, ?string $idempotencyKey = null): WebhookEndpoint
Payrex::webhooks()->disable(string $id, ?string $idempotencyKey = null): WebhookEndpoint
```

**Create params:** `url` (required, HTTPS), `events` (required, array), `description`

**List filters:** `url`, `description`

**WebhookEndpoint properties:** `id`, `secretKey`, `url`, `events` (array), `description`, `status` (WebhookEndpointStatus enum), `livemode`, `createdAt`, `updatedAt`

## Enums

All in `LegionHQ\LaravelPayrex\Enums\`:

```
PaymentMethod:        Card, GCash, Maya, BillEase, QrPh, BdoInstallment
PaymentIntentStatus:  AwaitingPaymentMethod, AwaitingNextAction, AwaitingCapture, Processing, Succeeded, Canceled
PaymentStatus:        Paid, Failed
CheckoutSessionStatus: Active, Completed, Expired
RefundStatus:         Succeeded, Failed, Pending
RefundReason:         Fraudulent, RequestedByCustomer, ProductOutOfStock, ServiceNotProvided, ProductWasDamaged, ServiceMisaligned, WrongProductReceived, Others
BillingStatementStatus: Draft, Open, Paid, Void, Uncollectible
PayoutStatus:         Pending, InTransit, Failed, Successful
WebhookEndpointStatus: Enabled, Disabled
WebhookEventType:     PaymentIntentSucceeded, PaymentIntentAmountCapturable, CashBalanceFundsAvailable, CheckoutSessionExpired, PayoutDeposited, RefundCreated, RefundUpdated, BillingStatementCreated, BillingStatementUpdated, BillingStatementDeleted, BillingStatementFinalized, BillingStatementSent, BillingStatementMarkedUncollectible, BillingStatementVoided, BillingStatementPaid, BillingStatementWillBeDue, BillingStatementOverdue, BillingStatementLineItemCreated, BillingStatementLineItemUpdated, BillingStatementLineItemDeleted
```

Pass enum values in API calls with `->value`:

```php
'reason' => RefundReason::RequestedByCustomer->value,
```

Typed properties return enum instances for comparison:

```php
if ($paymentIntent->status === PaymentIntentStatus::Succeeded) { ... }
```

## Webhook Handling

### Built-in Route (Recommended)

Set `PAYREX_WEBHOOK_ENABLED=true` and `PAYREX_WEBHOOK_SECRET` in `.env`. The package registers a POST route at the configured path (default: `/payrex/webhook`) with HMAC-SHA256 signature verification.

Listen for events in `AppServiceProvider::boot()`:

```php
use Illuminate\Support\Facades\Event;
use LegionHQ\LaravelPayrex\Events\PaymentIntentSucceeded;

Event::listen(PaymentIntentSucceeded::class, function ($event) {
    $paymentIntent = $event->data();  // Typed DTO
    $type = $event->eventType();       // ?WebhookEventType enum
    $isLive = $event->isLiveMode();    // bool
    $raw = $event->payload;            // array (full raw payload)
});
```

### Available Event Classes

All in `LegionHQ\LaravelPayrex\Events\`:

`PaymentIntentSucceeded`, `PaymentIntentAmountCapturable`, `CashBalanceFundsAvailable`, `CheckoutSessionExpired`, `PayoutDeposited`, `RefundCreated`, `RefundUpdated`, `BillingStatementCreated`, `BillingStatementUpdated`, `BillingStatementDeleted`, `BillingStatementFinalized`, `BillingStatementSent`, `BillingStatementMarkedUncollectible`, `BillingStatementVoided`, `BillingStatementPaid`, `BillingStatementWillBeDue`, `BillingStatementOverdue`, `BillingStatementLineItemCreated`, `BillingStatementLineItemUpdated`, `BillingStatementLineItemDeleted`

`WebhookReceived` — Catch-all dispatched for every webhook in addition to the typed event.

### Queued Listeners

```php
use Illuminate\Contracts\Queue\ShouldQueue;
use LegionHQ\LaravelPayrex\Events\PaymentIntentSucceeded;

class FulfillOrder implements ShouldQueue
{
    public function handle(PaymentIntentSucceeded $event): void
    {
        $paymentIntent = $event->data();
        // ...
    }
}
```

### Custom Controller with constructEvent()

For multi-tenant or custom routing:

```php
use LegionHQ\LaravelPayrex\Exceptions\WebhookVerificationException;
use LegionHQ\LaravelPayrex\Facades\Payrex;

try {
    $event = Payrex::constructEvent(
        payload: $request->getContent(),
        signatureHeader: $request->header('Payrex-Signature'),
        secret: $tenantSecret,     // optional, defaults to config
        tolerance: 300,            // optional, seconds
    );
} catch (WebhookVerificationException $e) {
    return response('Invalid signature', 403);
}
```

### Custom Route with Built-in Controller

```php
use LegionHQ\LaravelPayrex\Http\Controllers\WebhookController;
use LegionHQ\LaravelPayrex\Middleware\VerifyWebhookSignature;

Route::post('my/webhook', WebhookController::class)
    ->middleware(VerifyWebhookSignature::class);
```

## Billable Customer Trait

Add to your User model (requires published migration):

```php
use LegionHQ\LaravelPayrex\Concerns\HasPayrexCustomer;

class User extends Authenticatable
{
    use HasPayrexCustomer;
}
```

**Methods:**

```php
$user->createAsPayrexCustomer(array $params = []): Customer    // Creates in PayRex + stores ID
$user->asPayrexCustomer(): Customer                            // Retrieves from PayRex
$user->updatePayrexCustomer(array $params = []): Customer      // Updates in PayRex
$user->deleteAsPayrexCustomer(): DeletedResource               // Deletes + clears ID
$user->payrexCustomerId(): ?string
$user->hasPayrexCustomerId(): bool
```

**Override-friendly methods:** `payrexCustomerName()`, `payrexCustomerEmail()`, `payrexCustomerIdColumn()`

Throws `LogicException` if `createAsPayrexCustomer()` is called on a model that already has a PayRex customer ID.

## Error Handling

```
PayrexException (base)
├── PayrexApiException (any API error)
│   ├── AuthenticationException     (401)
│   ├── InvalidRequestException     (400)
│   ├── RateLimitException          (429)
│   └── ResourceNotFoundException   (404)
└── WebhookVerificationException    (signature failure)
```

All in `LegionHQ\LaravelPayrex\Exceptions\`.

**PayrexApiException properties:** `$e->errors` (array), `$e->statusCode` (int), `$e->body` (array), `$e->getMessage()` (first error detail)

```php
use LegionHQ\LaravelPayrex\Exceptions\InvalidRequestException;
use LegionHQ\LaravelPayrex\Exceptions\PayrexApiException;

try {
    Payrex::paymentIntents()->create([...]);
} catch (InvalidRequestException $e) {
    // 400 — $e->errors has validation details
} catch (PayrexApiException $e) {
    // Any other API error
    $requestId = Payrex::getLastResponse()?->header('X-Request-Id');
}
```

## Pagination

Resources with `list()`: Customers, Billing Statements, Payout Transactions, Webhooks.

```php
// Basic
$customers = Payrex::customers()->list(['limit' => 10]);
$customers->data;     // array of Customer DTOs
$customers->hasMore;  // bool

// Auto-pagination (lazy — fetches pages on demand)
$all = Payrex::customers()->list(['limit' => 100])->autoPaginate();
foreach ($all as $customer) { ... }

// With safety limit
$all = Payrex::customers()->list()->autoPaginate(maxPages: 50);

// Use list() as a lookup
$customers = Payrex::customers()->list(['email' => 'juan@example.com']);
$customer = $customers->data[0] ?? null;
```

**Filter support:** Customers (`name`, `email`, `metadata`), Webhooks (`url`, `description`).

## Response Metadata

```php
$paymentIntent = Payrex::paymentIntents()->create([...]);
$metadata = Payrex::getLastResponse();    // ?ApiResponseMetadata
$metadata->statusCode;                     // 200
$metadata->header('X-Request-Id');        // case-insensitive
```

Available after both successful and failed requests (captured before exceptions are thrown).

## Common Integration Flows

### Accept a Payment (Checkout Session)

```php
$session = Payrex::checkoutSessions()->create([
    'line_items' => [
        ['name' => 'Pro Plan', 'amount' => 99900, 'quantity' => 1],
    ],
    'payment_methods' => ['card', 'gcash', 'maya'],
    'success_url' => route('checkout.success'),
    'cancel_url' => route('checkout.cancel'),
]);

return redirect()->away($session->url);
```

### Accept a Payment (Payment Intent + Elements)

```php
// Backend
$paymentIntent = Payrex::paymentIntents()->create([
    'amount' => 10000,
    'payment_methods' => ['card', 'gcash', 'maya'],
]);

return response()->json([
    'client_secret' => $paymentIntent->clientSecret,
]);
```

Frontend uses PayRex JS SDK (`https://js.payrexhq.com`) or `payrex-js` npm package with `PAYREX_PUBLIC_KEY`.

### Hold then Capture (Card Only)

```php
// 1. Create with manual capture
$pi = Payrex::paymentIntents()->create([
    'amount' => 10000,
    'payment_methods' => ['card'],
    'payment_method_options' => [
        'card' => ['capture_type' => 'manual'],
    ],
]);

// 2. After customer authorizes (via webhook PaymentIntentAmountCapturable or retrieve):
$pi = Payrex::paymentIntents()->retrieve('pi_xxxxx');
// $pi->status === PaymentIntentStatus::AwaitingCapture
// $pi->amountCapturable === 10000

// 3. Capture (full or partial, one-time only)
$pi = Payrex::paymentIntents()->capture('pi_xxxxx', [
    'amount' => 10000,  // Must be ≤ amountCapturable
]);
```

Authorization expires in 7 days (`captureBeforeAt` timestamp).

### Refund a Payment

```php
use LegionHQ\LaravelPayrex\Enums\RefundReason;

$refund = Payrex::refunds()->create([
    'payment_id' => 'pay_xxxxx',
    'amount' => 5000,
    'reason' => RefundReason::RequestedByCustomer->value,
]);
```

### Billing Statement Flow

```php
// 1. Create draft
$stmt = Payrex::billingStatements()->create([
    'customer_id' => $user->payrexCustomerId(),
    'payment_settings' => ['payment_methods' => ['card', 'gcash']],
]);

// 2. Add line items
Payrex::billingStatementLineItems()->create([
    'billing_statement_id' => $stmt->id,
    'description' => 'Consulting — March 2026',
    'unit_price' => 500000,
    'quantity' => 2,
]);

// 3. Set due date
Payrex::billingStatements()->update($stmt->id, [
    'due_at' => now()->addDays(30)->timestamp,
    'payment_settings' => ['payment_methods' => ['card', 'gcash']],
]);

// 4. Finalize (generates payment URL)
$stmt = Payrex::billingStatements()->finalize($stmt->id);
// $stmt->url — send this to customer

// 5. Send via email
Payrex::billingStatements()->send($stmt->id);
```

## Testing

```php
use Illuminate\Support\Facades\Http;
use LegionHQ\LaravelPayrex\Facades\Payrex;

// Mock API responses
Http::fake([
    'api.payrexhq.com/payment_intents' => Http::response([
        'id' => 'pi_test_123',
        'resource' => 'payment_intent',
        'amount' => 10000,
        'currency' => 'PHP',
        'status' => 'awaiting_payment_method',
        'payment_methods' => ['card'],
        'livemode' => false,
        'created_at' => now()->timestamp,
        'updated_at' => now()->timestamp,
    ]),
]);

$pi = Payrex::paymentIntents()->create([
    'amount' => 10000,
    'payment_methods' => ['card'],
]);

// Test webhook listeners
use LegionHQ\LaravelPayrex\Events\PaymentIntentSucceeded;

PaymentIntentSucceeded::dispatch([
    'id' => 'evt_test_123',
    'type' => 'payment_intent.succeeded',
    'livemode' => false,
    'data' => [
        'resource' => [
            'id' => 'pi_test_456',
            'resource' => 'payment_intent',
            'amount' => 10000,
        ],
    ],
    'created_at' => now()->timestamp,
    'updated_at' => now()->timestamp,
]);
```

**Artisan command for testing webhooks locally:**

```bash
php artisan payrex:webhook-test payment_intent.succeeded
```

## Key Rules

- All monetary amounts are in **cents** (₱100.00 = `10000`, min ₱20.00 = `2000`, max ₱59,999,999.99 = `5999999999`)
- DTO properties are **camelCase** (`clientSecret`, `amountReceived`). Array access uses **snake_case** (`$pi['client_secret']`)
- All DTOs are **immutable** — setting/unsetting via array access throws `LogicException`
- Expandable fields (e.g. `latestPayment`, `customer`) can be a string ID or a typed DTO depending on the API response
- `currency` is auto-applied from config — you don't need to pass it unless overriding
- Always use **webhooks** for order fulfillment, not return/success URLs
- `payment_methods` is optional — defaults to all enabled methods on the merchant's PayRex account
- Idempotency keys are supported on all `create()` and action methods via `idempotencyKey:` named parameter
