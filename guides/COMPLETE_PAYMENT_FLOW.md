# Complete Payment Flow Guide

This document explains the complete flow from when an admin visits your website to purchase a plan, including card collection, payment processing, and card management.

## 📋 Table of Contents
1. [Step-by-Step Flow](#step-by-step-flow)
2. [Frontend Implementation](#frontend-implementation)
3. [API Endpoints](#api-endpoints)
4. [Card Management](#card-management)
5. [Recurring Payments](#recurring-payments)

---

## 🚀 Step-by-Step Flow

### **Step 1: Admin Visits Website & Views Plans**

Admin navigates to your pricing page.

**Frontend:**
```javascript
// Load Stripe.js
const stripe = Stripe('pk_your_publishable_key');

// Fetch available plans
const response = await fetch('/v1/payments/plans', {
  headers: {
    'Authorization': `Bearer ${adminToken}`,
  },
});

const { data: plans } = await response.json();
// Display plans: Basic ($29/month), Prime ($49/month), etc.
```

**API Call:**
```
GET /v1/payments/plans
Authorization: Bearer <admin_token>
```

**Response:**
```json
{
  "status": "success",
  "data": [
    {
      "id": "basic-plan-id",
      "name": "Basic",
      "description": "Perfect for small teams",
      "monthly": {
        "price": 29,
        "currency": "usd",
        "priceId": "price_monthly_basic"
      },
      "annually": {
        "price": 290,
        "currency": "usd",
        "priceId": "price_annual_basic"
      },
      "maxUsers": 1000,
      "features": [...]
    },
    // ... more plans
  ]
}
```

---

### **Step 2: Admin Clicks on a Plan**

Admin selects a plan and billing cycle (monthly or annually).

**Frontend:**
```javascript
function onPlanSelected(planId, billingCycle = 'monthly') {
  // Show payment form
  showPaymentForm(planId, billingCycle);
}

function showPaymentForm(planId, billingCycle) {
  // Initialize Stripe Elements
  const elements = stripe.elements();
  const cardElement = elements.create('card', {
    style: {
      base: {
        fontSize: '16px',
        color: '#424770',
      },
    },
  });
  
  cardElement.mount('#card-element');
  
  // Store plan info for later
  window.selectedPlan = { planId, billingCycle, cardElement };
}
```

---

### **Step 3: Admin Enters Card Details**

Admin enters card information using Stripe Elements (secure card form).

**Frontend HTML:**
```html
<div id="card-element">
  <!-- Stripe Elements will mount here -->
</div>
<div id="card-errors" role="alert"></div>
<button id="submit-button">Subscribe Now</button>
```

**Frontend JavaScript:**
```javascript
// Listen for card errors
cardElement.on('change', ({error}) => {
  const displayError = document.getElementById('card-errors');
  if (error) {
    displayError.textContent = error.message;
  } else {
    displayError.textContent = '';
  }
});

// Handle form submission
document.getElementById('submit-button').addEventListener('click', async (e) => {
  e.preventDefault();
  
  const { planId, billingCycle } = window.selectedPlan;
  
  try {
    // Step 1: Create Payment Method from card
    const { paymentMethod, error: pmError } = await stripe.createPaymentMethod({
      type: 'card',
      card: cardElement,
      billing_details: {
        name: adminName,
        email: adminEmail,
      },
    });

    if (pmError) {
      // Show error to user
      showError(pmError.message);
      return;
    }

    // Step 2: Create subscription with payment method
    const response = await fetch('/v1/payments/subscriptions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${adminToken}`,
      },
      body: JSON.stringify({
        organizationId: adminOrganizationId,
        planId: planId,
        billingCycle: billingCycle, // 'monthly' or 'annually'
        paymentMethodId: paymentMethod.id, // e.g., "pm_1234567890"
      }),
    });

    const result = await response.json();

    if (!response.ok) {
      throw new Error(result.message || 'Subscription failed');
    }

    // Step 3: Handle payment confirmation if needed (3D Secure)
    if (result.data.paymentIntent?.requires_action) {
      // Payment requires 3D Secure confirmation
      const { error: confirmError, paymentIntent } = await stripe.confirmCardPayment(
        result.data.paymentIntent.client_secret
      );

      if (confirmError) {
        showError(confirmError.message);
        return;
      }

      if (paymentIntent.status === 'succeeded') {
        // Payment confirmed! Show success
        showSuccess('Subscription activated successfully!');
      }
    } else if (result.data.status === 'active') {
      // Payment succeeded immediately
      showSuccess('Subscription activated successfully!');
    } else {
      // Payment is processing
      showMessage('Subscription created. Payment is being processed...');
    }

  } catch (error) {
    showError(error.message);
  }
});
```

**API Call:**
```
POST /v1/payments/subscriptions
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "organizationId": "507f1f77bcf86cd799439011",
  "planId": "basic-plan-id",
  "billingCycle": "monthly",
  "paymentMethodId": "pm_1234567890"
}
```

**What Happens Backend:**
1. ✅ Creates/retrieves Stripe customer
2. ✅ Attaches payment method to customer
3. ✅ Sets payment method as default for future payments
4. ✅ Creates Stripe subscription
5. ✅ **SAVES CARD DETAILS** in database (card brand, last4, expiry)
6. ✅ Processes payment
7. ✅ Returns subscription with payment intent if needed

**Response:**
```json
{
  "status": "success",
  "data": {
    "id": "subscription-id",
    "planId": "basic-plan-id",
    "planName": "Basic",
    "status": "active",
    "billingCycle": "monthly",
    "price": 29,
    "currency": "usd",
    "paymentMethods": [
      {
        "id": "pm_1234567890",
        "type": "card",
        "card": {
          "brand": "visa",
          "last4": "4242",
          "expMonth": 12,
          "expYear": 2025
        },
        "isDefault": true,
        "addedAt": "2024-01-15T10:00:00Z"
      }
    ],
    "paymentIntent": {
      // Only if payment requires confirmation
      "id": "pi_1234567890",
      "status": "succeeded",
      "requires_action": false
    }
  },
  "message": "Subscription created and activated successfully."
}
```

---

### **Step 4: Card Details Saved for Future Payments**

✅ **Card is automatically saved!**

When you create a subscription with `paymentMethodId`, the backend:
1. Attaches payment method to Stripe customer
2. Sets it as default payment method
3. Saves card details in database:
   - Card brand (Visa, Mastercard, etc.)
   - Last 4 digits
   - Expiry month/year
   - Payment method ID

**Saved Card Information:**
- Stored in `subscription.paymentMethods[]` array
- First card becomes default automatically
- Used for all future recurring payments

---

### **Step 5: Recurring Payments (Automatic)**

Stripe automatically charges the saved card on each billing cycle:

- **Monthly Plan**: Charged every month on the same date
- **Annual Plan**: Charged once per year

**No action needed from admin!**

**Webhook Events Handled:**
- `invoice.payment_succeeded` → Subscription stays active
- `invoice.payment_failed` → System records error, subscription goes to `past_due`
- `customer.subscription.updated` → Subscription status synced

---

### **Step 6: Admin Changes Card for Next Payment**

Admin can change/update their payment method anytime.

#### **Option A: Add New Card & Set as Default**

**Frontend:**
```javascript
// Step 1: Collect new card
const { paymentMethod, error } = await stripe.createPaymentMethod({
  type: 'card',
  card: newCardElement,
  billing_details: {
    name: adminName,
    email: adminEmail,
  },
});

// Step 2: Add to subscription and set as default
const response = await fetch(
  `/v1/payments/subscriptions/${subscriptionId}/payment-methods`,
  {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${adminToken}`,
    },
    body: JSON.stringify({
      paymentMethodId: paymentMethod.id,
      setAsDefault: true, // This makes it the default for future payments
    }),
  }
);

// Step 3: Make it default (if not done above)
await fetch(
  `/v1/payments/subscriptions/${subscriptionId}/payment-methods/default`,
  {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${adminToken}`,
    },
    body: JSON.stringify({
      paymentMethodId: paymentMethod.id,
    }),
  }
);
```

**API Calls:**
```
POST /v1/payments/subscriptions/{subscriptionId}/payment-methods
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "paymentMethodId": "pm_new_card_123",
  "setAsDefault": true
}
```

**Or set existing card as default:**
```
POST /v1/payments/subscriptions/{subscriptionId}/payment-methods/default
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "paymentMethodId": "pm_existing_456"
}
```

#### **Option B: Remove Old Card**

**Frontend:**
```javascript
await fetch(
  `/v1/payments/subscriptions/${subscriptionId}/payment-methods/${oldCardId}`,
  {
    method: 'DELETE',
    headers: {
      'Authorization': `Bearer ${adminToken}`,
    },
  }
);
```

**API Call:**
```
DELETE /v1/payments/subscriptions/{subscriptionId}/payment-methods/{paymentMethodId}
Authorization: Bearer <admin_token>
```

---

### **Step 7: View Saved Cards**

Admin can view all saved payment methods.

**Frontend:**
```javascript
// Get billing details (includes payment methods)
const response = await fetch(
  `/v1/payments/billing/${organizationId}`,
  {
    headers: {
      'Authorization': `Bearer ${adminToken}`,
    },
  }
);

const { data } = await response.json();

// Display payment methods
data.paymentMethods.forEach(card => {
  console.log(`${card.card.brand} •••• ${card.card.last4}`);
  console.log(`Expires: ${card.card.expMonth}/${card.card.expYear}`);
  console.log(`Default: ${card.isDefault}`);
});
```

**API Call:**
```
GET /v1/payments/billing/{organizationId}
Authorization: Bearer <admin_token>
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "subscription": {
      "id": "...",
      "planName": "Basic",
      "status": "active",
      "billingCycle": "monthly",
      "price": 29,
      "nextBillingDate": "2024-02-15T00:00:00Z",
      ...
    },
    "paymentMethods": [
      {
        "id": "pm_1234567890",
        "type": "card",
        "card": {
          "brand": "visa",
          "last4": "4242",
          "expMonth": 12,
          "expYear": 2025
        },
        "isDefault": true,
        "addedAt": "2024-01-15T10:00:00Z"
      },
      {
        "id": "pm_0987654321",
        "type": "card",
        "card": {
          "brand": "mastercard",
          "last4": "5555",
          "expMonth": 6,
          "expYear": 2026
        },
        "isDefault": false,
        "addedAt": "2024-01-20T14:30:00Z"
      }
    ],
    "defaultPaymentMethodId": "pm_1234567890"
  }
}
```

---

## 📝 Complete API Reference

### **Get Available Plans**
```
GET /v1/payments/plans
Authorization: Bearer <admin_token>
```

### **Create Subscription**
```
POST /v1/payments/subscriptions
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "organizationId": "org-id",
  "planId": "plan-id",
  "billingCycle": "monthly" | "annually",
  "paymentMethodId": "pm_xxx" // Optional - if not provided, payment required later
}
```

### **Get Billing Details (View Cards)**
```
GET /v1/payments/billing/{organizationId}
Authorization: Bearer <admin_token>
```

### **Add Payment Method**
```
POST /v1/payments/subscriptions/{subscriptionId}/payment-methods
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "paymentMethodId": "pm_new_card",
  "setAsDefault": true // Optional, default: false
}
```

### **Set Default Payment Method**
```
POST /v1/payments/subscriptions/{subscriptionId}/payment-methods/default
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "paymentMethodId": "pm_card_id"
}
```

### **Remove Payment Method**
```
DELETE /v1/payments/subscriptions/{subscriptionId}/payment-methods/{paymentMethodId}
Authorization: Bearer <admin_token>
```

### **Get Subscription Details**
```
GET /v1/payments/subscriptions/{subscriptionId}
Authorization: Bearer <admin_token>

OR

GET /v1/payments/subscriptions?organizationId={orgId}
Authorization: Bearer <admin_token>
```

---

## 🔒 Security Notes

1. **Never send card numbers to your backend** - Always use Stripe.js/Elements
2. **Payment method IDs are safe** - `pm_xxx` IDs can be sent to backend
3. **Card details stored securely** - Only last4, brand, expiry saved (no full card number)
4. **Stripe handles PCI compliance** - Your backend never touches sensitive card data

---

## 🎯 Key Points

✅ **Card is saved automatically** when creating subscription with `paymentMethodId`  
✅ **Default card is used** for all recurring payments automatically  
✅ **Admin can add multiple cards** and choose which one to use  
✅ **Changing default card** updates future payments immediately  
✅ **All payment processing** happens automatically via Stripe webhooks  
✅ **No manual intervention needed** for recurring payments

---

## 📚 Frontend Code Example (Complete)

```html
<!DOCTYPE html>
<html>
<head>
  <title>Subscribe to Plan</title>
  <script src="https://js.stripe.com/v3/"></script>
</head>
<body>
  <div id="plans-container"></div>
  
  <div id="payment-form" style="display: none;">
    <div id="card-element"></div>
    <div id="card-errors"></div>
    <button id="submit-btn">Subscribe</button>
  </div>

  <script>
    const stripe = Stripe('pk_your_publishable_key');
    const elements = stripe.elements();
    let cardElement;
    let selectedPlan = null;

    // Load plans
    async function loadPlans() {
      const response = await fetch('/v1/payments/plans', {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      const { data: plans } = await response.json();
      
      plans.forEach(plan => {
        const planCard = document.createElement('div');
        planCard.innerHTML = `
          <h3>${plan.name}</h3>
          <p>$${plan.monthly.price}/month or $${plan.annually.price}/year</p>
          <button onclick="selectPlan('${plan.id}')">Select Plan</button>
        `;
        document.getElementById('plans-container').appendChild(planCard);
      });
    }

    function selectPlan(planId) {
      selectedPlan = { planId, billingCycle: 'monthly' };
      
      // Show payment form
      document.getElementById('payment-form').style.display = 'block';
      
      // Mount card element
      cardElement = elements.create('card');
      cardElement.mount('#card-element');
      
      cardElement.on('change', ({error}) => {
        const displayError = document.getElementById('card-errors');
        displayError.textContent = error ? error.message : '';
      });
    }

    document.getElementById('submit-btn').addEventListener('click', async () => {
      // Create payment method
      const { paymentMethod, error: pmError } = await stripe.createPaymentMethod({
        type: 'card',
        card: cardElement,
      });

      if (pmError) {
        alert(pmError.message);
        return;
      }

      // Create subscription
      const response = await fetch('/v1/payments/subscriptions', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`,
        },
        body: JSON.stringify({
          organizationId: orgId,
          planId: selectedPlan.planId,
          billingCycle: selectedPlan.billingCycle,
          paymentMethodId: paymentMethod.id,
        }),
      });

      const result = await response.json();

      // Handle 3D Secure if needed
      if (result.data.paymentIntent?.requires_action) {
        const { error } = await stripe.confirmCardPayment(
          result.data.paymentIntent.client_secret
        );
        if (error) {
          alert(error.message);
          return;
        }
      }

      alert('Subscription created! Your card is saved for future payments.');
    });

    // Load plans on page load
    loadPlans();
  </script>
</body>
</html>
```

---

## ✅ Summary

**Complete Flow:**
1. Admin views plans → `GET /v1/payments/plans`
2. Admin selects plan → Shows Stripe Elements card form
3. Admin enters card → `stripe.createPaymentMethod()` (frontend)
4. Admin subscribes → `POST /v1/payments/subscriptions` with `paymentMethodId`
5. Backend saves card → Automatically saved in database
6. Payment processed → Stripe charges card
7. Future payments → Automatic (Stripe charges saved card)
8. Change card → `POST /v1/payments/subscriptions/{id}/payment-methods` with `setAsDefault: true`

**Card Management:**
- View cards → `GET /v1/payments/billing/{orgId}`
- Add card → `POST /v1/payments/subscriptions/{id}/payment-methods`
- Set default → `POST /v1/payments/subscriptions/{id}/payment-methods/default`
- Remove card → `DELETE /v1/payments/subscriptions/{id}/payment-methods/{pmId}`

All implemented and ready to use! 🎉

