# Stripe Sandbox Setup Guide

This guide explains how to set up and use Stripe's sandbox environment for testing the SaaS application.

## Prerequisites

1. A Stripe account with access to the test environment
2. Stripe CLI installed locally
3. Node.js and pnpm installed

## Initial Setup

### 1. Stripe CLI Installation

```bash
# macOS
brew install stripe/stripe-cli/stripe

# Other platforms
# Visit: https://stripe.com/docs/stripe-cli
```

### 2. Authentication

```bash
# Login to Stripe CLI
stripe login
```

### 3. Environment Variables

Create a `.env` file with your Stripe test keys:

```env
STRIPE_SECRET_KEY=sk_test_your_key_here
STRIPE_WEBHOOK_SECRET=whsec_your_webhook_secret_here
BASE_URL=http://localhost:3000
AUTH_SECRET=your_auth_secret_here
```

## Product Configuration

### 1. Create Products and Prices

```bash
# Create Base Plan ($8/month)
stripe products create -d name="Base" -d description="Base subscription plan"
stripe prices create -d product=PRODUCT_ID -d unit_amount=800 -d currency=usd -d recurring[interval]=month -d recurring[trial_period_days]=14

# Create Plus Plan ($12/month)
stripe products create -d name="Plus" -d description="Plus subscription plan"
stripe prices create -d product=PRODUCT_ID -d unit_amount=1200 -d currency=usd -d recurring[interval]=month -d recurring[trial_period_days]=14
```

### 2. Verify Setup

```bash
# List active products with prices
stripe products list --active=true --expand=data.default_price

# Clean up duplicate/test products
stripe products update prod_XXX -d active=false
```

## Customer Portal Configuration

1. Create portal configuration:

```bash
stripe post "/v1/billing_portal/configurations" \
  -d "business_profile[headline]=Manage your subscription" \
  -d "features[customer_update][enabled]=true" \
  -d "features[customer_update][allowed_updates][]=email" \
  -d "features[customer_update][allowed_updates][]=address" \
  -d "features[invoice_history][enabled]=true" \
  -d "features[payment_method_update][enabled]=true" \
  -d "features[subscription_cancel][enabled]=true" \
  -d "features[subscription_update][enabled]=true" \
  -d "features[subscription_update][default_allowed_updates][]=price" \
  -d "features[subscription_update][default_allowed_updates][]=quantity" \
  -d "features[subscription_update][proration_behavior]=create_prorations"
```

2. Add products to portal configuration:

```bash
stripe post "/v1/billing_portal/configurations/CONFIG_ID" \
  -d "features[subscription_update][products][0][product]=PRODUCT_ID" \
  -d "features[subscription_update][products][0][prices][]=PRICE_ID"
```

## Webhook Events Flow

### 1. Initial Subscription

```plaintext
1. customer.created
2. payment_method.attached
3. plan.created
4. price.created
5. charge.succeeded
6. customer.updated
7. payment_intent.created
8. customer.subscription.created
9. payment_intent.succeeded
10. invoice.created
11. invoice.finalized
12. invoice.paid
13. invoice.payment_succeeded
```

### 2. Plan Changes (Upgrade/Downgrade)

```plaintext
1. customer.subscription.updated
2. invoiceitem.created (for prorations)
3. invoice.created
4. invoice.finalized
5. invoice.paid
6. invoice.payment_succeeded
```

### 3. Cancellation and Reactivation

```plaintext
1. customer.subscription.updated (cancel)
2. customer.subscription.updated (reactivate)
```

### 4. Trial Management

```plaintext
1. customer.subscription.trial_will_end (3 days before)
2. customer.subscription.updated
3. charge.succeeded
4. payment_intent.succeeded
5. invoice.created
6. invoice.paid
```

### 5. Portal Sessions

```plaintext
1. billing_portal.session.created
2. billing_portal.configuration.updated
```

## Testing

### 1. Start Webhook Listener

```bash
stripe listen --forward-to localhost:3000/api/stripe/webhook
```

### 2. Test Cards

```plaintext
Success: 4242 4242 4242 4242
Decline: 4000 0000 0000 0002
3D Secure: 4000 0000 0000 3220
Expiry: Any future date
CVC: Any 3 digits
```

## Common Issues

### 1. Webhook Connection Errors

If you see "Failed to POST: connection refused":

- Ensure Next.js server is running
- Check webhook URL is correct
- Verify webhook secret in .env

### 2. Product Not Found

If you get "Error: No such product":

- Check product exists and is active
- Verify default price is set
- Update portal configuration with correct product

### 3. Portal Access Issues

- Verify customer has active subscription
- Check portal configuration includes products
- Ensure all required features are enabled

## Branding Limitations

Note: In test mode, branding customization is limited:

- Logo customization unavailable
- Color schemes locked
- Email templates use default design
- Custom domain not available

Full branding customization requires live mode.

## API Version

Current API Version: 2022-11-15

## Additional Resources

- [Stripe API Documentation](https://stripe.com/docs/api)
- [Stripe CLI Documentation](https://stripe.com/docs/stripe-cli)
- [Testing Webhooks](https://stripe.com/docs/webhooks/test)
- [Customer Portal Guide](https://stripe.com/docs/billing/subscriptions/customer-portal)
- [Test Clock Guide](https://stripe.com/docs/billing/testing/test-clock)
