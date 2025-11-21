# Simple Payment Flow Guide

**How admin purchases a plan and manages cards for recurring payments**

---

## 🎯 Complete Flow (Step-by-Step)

### **Step 1: Admin Views Plans**

Admin visits your pricing page.

**Frontend:**
```javascript
// Fetch available plans
const response = await fetch('/v1/payments/plans', {
  headers: { 'Authorization': `Bearer ${adminToken}` }
});

const { data: plans } = await response.json();
// Display: Basic ($29/month), Prime ($49/month), Premium ($99/month)
```

**API:**
```
GET /v1/payments/plans
Authorization: Bearer <admin_token>
```

---

### **Step 2: Admin Clicks on Plan**

Admin selects a plan (e.g., Basic Plan - $29/month).

**Frontend shows payment form with Stripe Elements:**

```html
<div id="card-element">
  <!-- Stripe Elements card form -->
</div>
<div id="card-errors"></div>
<button id="subscribe-btn">Subscribe Now</button>
```

```javascript
// Initialize Stripe Elements
const stripe = Stripe('pk_your_publishable_key');
const elements = stripe.elements();
const cardElement = elements.create('card');
cardElement.mount('#card-element');
```

---

### **Step 3: Admin Enters Card Details**

Admin enters card number, expiry, CVV using Stripe Elements (secure form).

---

### **Step 4: Admin Subscribes - API Called**

Admin clicks "Subscribe" button.

**Frontend Code:**
```javascript
document.getElementById('subscribe-btn').addEventListener('click', async (e) => {
  e.preventDefault();
  
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
    alert(pmError.message);
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
      organizationId: '507f1f77bcf86cd799439011',
      planId: 'basic-plan-id',
      billingCycle: 'monthly', // or 'annually'
      paymentMethodId: paymentMethod.id, // e.g., "pm_1234567890"
    }),
  });

  const result = await response.json();

  // Step 3: Handle 3D Secure if needed
  if (result.data.paymentIntent?.requires_action) {
    const { error } = await stripe.confirmCardPayment(
      result.data.paymentIntent.client_secret
    );
    
    if (error) {
      alert(error.message);
      return;
    }
  }

  // Success!
  alert('Subscription created! Card saved for future payments.');
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
1. ✅ Admin gets charged immediately ($29)
2. ✅ Card details are saved automatically (Visa •••• 4242)
3. ✅ Card is set as default for future payments
4. ✅ Subscription created and activated
5. ✅ Card saved in database for display

**Response:**
```json
{
  "status": "success",
  "data": {
    "id": "subscription-id",
    "status": "active",
    "price": 29,
    "billingCycle": "monthly",
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
        "isDefault": true
      }
    ]
  },
  "message": "Subscription created and activated successfully."
}
```

---

### **Step 5: Future Payments (Automatic)**

**No action needed!** Stripe automatically charges the saved default card every month (or year if annual).

- **Monthly plan**: Charged every month on the same date
- **Annual plan**: Charged once per year

The default payment method is used automatically.

---

### **Step 6: Admin Adds Another Card**

Admin can add more cards anytime.

**Frontend:**
```javascript
// Collect new card using Stripe Elements
const { paymentMethod, error } = await stripe.createPaymentMethod({
  type: 'card',
  card: newCardElement,
});

// Add to subscription
await fetch(`/v1/payments/subscriptions/${subscriptionId}/payment-methods`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${adminToken}`,
  },
  body: JSON.stringify({
    paymentMethodId: paymentMethod.id,
    setAsDefault: false, // Keep current card as default
  }),
});
```

**API:**
```
POST /v1/payments/subscriptions/{subscriptionId}/payment-methods
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "paymentMethodId": "pm_new_card_123",
  "setAsDefault": false
}
```

**Response:** Card added to subscription, ready to use.

---

### **Step 7: Admin Sets Different Card as Default**

Admin can change which card is used for future payments.

**Frontend:**
```javascript
await fetch(
  `/v1/payments/subscriptions/${subscriptionId}/payment-methods/default`,
  {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${adminToken}`,
    },
    body: JSON.stringify({
      paymentMethodId: 'pm_existing_card_456', // Different card
    }),
  }
);
```

**API:**
```
POST /v1/payments/subscriptions/{subscriptionId}/payment-methods/default
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "paymentMethodId": "pm_existing_card_456"
}
```

**Result:** This card becomes the default and will be used for ALL future recurring payments.

---

### **Step 8: Admin Views Saved Cards**

Admin can see all saved payment methods.

**Frontend:**
```javascript
const response = await fetch(
  `/v1/payments/billing/${organizationId}`,
  {
    headers: { 'Authorization': `Bearer ${adminToken}` }
  }
);

const { data } = await response.json();

// Display cards
data.paymentMethods.forEach(card => {
  console.log(`${card.card.brand} •••• ${card.card.last4}`);
  console.log(`Expires: ${card.card.expMonth}/${card.card.expYear}`);
  console.log(`Default: ${card.isDefault ? 'Yes' : 'No'}`);
});
```

**API:**
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
      "price": 29,
      "billingCycle": "monthly",
      "nextBillingDate": "2024-02-15T00:00:00Z"
    },
    "paymentMethods": [
      {
        "id": "pm_1234567890",
        "card": {
          "brand": "visa",
          "last4": "4242",
          "expMonth": 12,
          "expYear": 2025
        },
        "isDefault": true
      },
      {
        "id": "pm_0987654321",
        "card": {
          "brand": "mastercard",
          "last4": "5555",
          "expMonth": 6,
          "expYear": 2026
        },
        "isDefault": false
      }
    ]
  }
}
```

---

## 📋 Complete API Reference

### **1. View Plans**
```
GET /v1/payments/plans
Authorization: Bearer <admin_token>
```

### **2. Create Subscription (with card)**
```
POST /v1/payments/subscriptions
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "organizationId": "org-id",
  "planId": "plan-id",
  "billingCycle": "monthly" | "annually",
  "paymentMethodId": "pm_xxx"  // REQUIRED - card is saved automatically
}
```

**Result:**
- ✅ Admin gets charged immediately
- ✅ Card saved automatically
- ✅ Card set as default
- ✅ Subscription activated

### **3. View Billing & Saved Cards**
```
GET /v1/payments/billing/{organizationId}
Authorization: Bearer <admin_token>
```

### **4. Add New Card**
```
POST /v1/payments/subscriptions/{subscriptionId}/payment-methods
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "paymentMethodId": "pm_new_card",
  "setAsDefault": false  // Optional, default: false
}
```

### **5. Set Default Card**
```
POST /v1/payments/subscriptions/{subscriptionId}/payment-methods/default
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "paymentMethodId": "pm_card_id"
}
```

**Result:** This card will be used for ALL future recurring payments.

### **6. Remove Card**
```
DELETE /v1/payments/subscriptions/{subscriptionId}/payment-methods/{paymentMethodId}
Authorization: Bearer <admin_token>
```

---

## ✅ Key Points

1. **Card saved automatically** when creating subscription with `paymentMethodId`
2. **Admin gets charged immediately** on subscription creation
3. **Default card is set automatically** (first card becomes default)
4. **Future payments use default card** automatically (no action needed)
5. **Admin can add multiple cards** and switch default anytime
6. **Setting new default** updates it immediately for next payment
7. **All recurring payments** happen automatically via Stripe

---

## 🎯 Summary Flow

1. Admin clicks plan → Shows card form (Stripe Elements)
2. Admin enters card → `stripe.createPaymentMethod()` (frontend)
3. Admin subscribes → `POST /v1/payments/subscriptions` with `paymentMethodId`
4. **Backend charges immediately** → Card saved → Set as default
5. **Future payments automatic** → Uses default card every billing cycle
6. Admin can add more cards → `POST /subscriptions/{id}/payment-methods`
7. Admin can change default → `POST /subscriptions/{id}/payment-methods/default`

**Everything is automatic after initial setup!** 🎉
