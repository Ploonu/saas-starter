# Stripe CLI Commands Cheatsheet

Current API Version: 2022-11-15

## Authentication & Setup

```bash
# Login to Stripe CLI
stripe login

# Check version
stripe --version

# View webhook logs
stripe logs tail

# List all commands
stripe help
```

## Products & Prices

```bash
# List active products with default prices
stripe products list --active=true --expand=data.default_price

# List active prices
stripe prices list --active=true

# Create a product
stripe products create -d name="Product Name" -d description="Description"

# Update product's default price
stripe products update prod_XXXXX -d default_price=price_XXXXX

# Create a price
stripe prices create \
  -d unit_amount=1000 \
  -d currency=usd \
  -d recurring[interval]=month \
  -d product=prod_XXXXX
```

## Customer Portal

```bash
# Create portal configuration
stripe post "/v1/billing_portal/configurations" \
  -d "business_profile[headline]=Manage your subscription" \
  -d "features[customer_update][enabled]=true" \
  -d "features[customer_update][allowed_updates][]=email" \
  -d "features[customer_update][allowed_updates][]=address" \
  -d "features[invoice_history][enabled]=true" \
  -d "features[payment_method_update][enabled]=true" \
  -d "features[subscription_cancel][enabled]=true"

# Update portal with subscription updates
stripe post "/v1/billing_portal/configurations/CONFIG_ID" \
  -d "features[subscription_update][enabled]=true" \
  -d "features[subscription_update][default_allowed_updates][]=price" \
  -d "features[subscription_update][default_allowed_updates][]=quantity" \
  -d "features[subscription_update][proration_behavior]=create_prorations" \
  -d "features[subscription_update][products][0][product]=PRODUCT_ID" \
  -d "features[subscription_update][products][0][prices][]=PRICE_ID"

# List portal configurations
stripe get "/v1/billing_portal/configurations"
```

## Webhooks

```bash
# Start webhook listener
stripe listen --forward-to localhost:3000/api/stripe/webhook

# List webhook endpoints
stripe webhooks list

# Create webhook endpoint
stripe webhooks create --url https://your-domain.com/api/stripe/webhook \
  --events customer.subscription.updated,customer.subscription.deleted

# Delete webhook endpoint
stripe webhooks delete we_XXXXX
```

## Testing

```bash
# Trigger test webhook event
stripe trigger customer.subscription.updated

# List test webhook events
stripe events list

# Get specific event details
stripe events retrieve evt_XXXXX
```

## Customers & Subscriptions

```bash
# List customers
stripe customers list

# Get customer details
stripe customers retrieve cus_XXXXX

# List subscriptions
stripe subscriptions list

# Get subscription details
stripe subscriptions retrieve sub_XXXXX

# Cancel subscription
stripe subscriptions cancel sub_XXXXX
```

## Common Flags

```bash
--api-key=sk_test_XXX  # Use specific API key
--api-version=2022-11-15  # Use specific API version
--expand=data.default_price  # Expand nested objects
--limit=3  # Limit number of results
--starting-after=obj_XXXXX  # Pagination
```

## Environment Variables

```bash
# Test mode
STRIPE_SECRET_KEY=sk_test_XXX
STRIPE_WEBHOOK_SECRET=whsec_XXX

# Live mode
STRIPE_SECRET_KEY=sk_live_XXX
STRIPE_WEBHOOK_SECRET=whsec_XXX
```

## Useful Debugging Commands

```bash
# Check API connection
stripe config

# View request logs
stripe logs tail

# Test webhook signature
stripe webhook-verify --secret whsec_XXX

# Get rate limit info
stripe rate-limit-info
```
