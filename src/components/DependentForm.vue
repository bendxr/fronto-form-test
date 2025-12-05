<template>
  <div class="form-container">
    <h2>Order Form with Dependencies</h2>
    
    <form @submit.prevent="handleSubmit">
      <!-- Product Selection -->
      <div class="form-group">
        <label for="product">Select Product:</label>
        <select id="product" v-model="formData.product" required>
          <option value="">-- Choose a product --</option>
          <option value="laptop">Laptop ($1000)</option>
          <option value="phone">Phone ($500)</option>
          <option value="tablet">Tablet ($300)</option>
        </select>
      </div>

      <!-- Quantity (depends on product) -->
      <div class="form-group" v-if="formData.product">
        <label for="quantity">Quantity:</label>
        <input 
          id="quantity" 
          type="number" 
          v-model.number="formData.quantity" 
          min="1" 
          max="10"
          required
        />
        <span class="helper-text">Available: {{ maxQuantity }} units</span>
      </div>

      <!-- Shipping Method (depends on quantity and product) -->
      <div class="form-group" v-if="formData.product && formData.quantity > 0">
        <label for="shipping">Shipping Method:</label>
        <select id="shipping" v-model="formData.shipping" required>
          <option value="">-- Choose shipping --</option>
          <option value="standard">Standard (5-7 days) - $10</option>
          <option value="express" :disabled="!isExpressAvailable">
            Express (2-3 days) - $25 {{ !isExpressAvailable ? '(Not available for orders > 5 items)' : '' }}
          </option>
          <option value="overnight" :disabled="!isOvernightAvailable">
            Overnight - $50 {{ !isOvernightAvailable ? '(Only available for orders ≤ 2 items)' : '' }}
          </option>
        </select>
      </div>

      <!-- Insurance (depends on total price) -->
      <div class="form-group" v-if="totalBeforeInsurance > 0">
        <label>
          <input type="checkbox" v-model="formData.insurance" />
          Add Insurance ({{ insuranceCost }})
        </label>
        <span class="helper-text" v-if="totalBeforeInsurance >= 1000">
          Recommended for orders over $1000
        </span>
      </div>

      <!-- Discount Code (depends on product and quantity) -->
      <div class="form-group" v-if="formData.quantity >= 3">
        <label for="discount">Discount Code:</label>
        <input 
          id="discount" 
          type="text" 
          v-model="formData.discountCode" 
          placeholder="Enter code (BULK10 for 10% off)"
        />
        <span class="helper-text success" v-if="isDiscountValid">
          ✓ Valid discount applied!
        </span>
        <span class="helper-text error" v-if="formData.discountCode && !isDiscountValid">
          ✗ Invalid discount code
        </span>
      </div>

      <!-- Summary Section -->
      <div class="summary-section" v-if="formData.product">
        <h3>Order Summary</h3>
        <div class="summary-item">
          <span>Product:</span>
          <span>{{ productName }} x {{ formData.quantity }}</span>
        </div>
        <div class="summary-item">
          <span>Subtotal:</span>
          <span>${{ subtotal }}</span>
        </div>
        <div class="summary-item" v-if="formData.shipping">
          <span>Shipping:</span>
          <span>${{ shippingCost }}</span>
        </div>
        <div class="summary-item" v-if="isDiscountValid">
          <span>Discount ({{ formData.discountCode }}):</span>
          <span class="discount">-${{ discountAmount }}</span>
        </div>
        <div class="summary-item" v-if="formData.insurance">
          <span>Insurance:</span>
          <span>${{ insuranceAmount }}</span>
        </div>
        <div class="summary-item total">
          <span><strong>Total:</strong></span>
          <span><strong>${{ totalPrice }}</strong></span>
        </div>
      </div>

      <!-- Submit Button -->
      <div class="form-group">
        <button 
          type="submit" 
          :disabled="!isFormValid"
          class="submit-btn"
        >
          Place Order
        </button>
      </div>

      <!-- Form Status Message -->
      <div v-if="submitMessage" class="message" :class="submitMessage.type">
        {{ submitMessage.text }}
      </div>
    </form>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const formData = ref({
  product: '',
  quantity: 1,
  shipping: '',
  insurance: false,
  discountCode: ''
})

const submitMessage = ref(null)

// Product prices
const productPrices = {
  laptop: 1000,
  phone: 500,
  tablet: 300
}

// Product names
const productNames = {
  laptop: 'Laptop',
  phone: 'Phone',
  tablet: 'Tablet'
}

// Max quantity per product
const maxQuantity = computed(() => {
  return formData.value.product ? 10 : 0
})

// Computed: Product name
const productName = computed(() => {
  return productNames[formData.value.product] || ''
})

// Computed: Subtotal
const subtotal = computed(() => {
  if (!formData.value.product) return 0
  return productPrices[formData.value.product] * formData.value.quantity
})

// Computed: Shipping availability
const isExpressAvailable = computed(() => {
  return formData.value.quantity <= 5
})

const isOvernightAvailable = computed(() => {
  return formData.value.quantity <= 2
})

// Computed: Shipping cost
const shippingCost = computed(() => {
  const costs = {
    standard: 10,
    express: 25,
    overnight: 50
  }
  return costs[formData.value.shipping] || 0
})

// Computed: Discount validation and amount
const isDiscountValid = computed(() => {
  const code = formData.value.discountCode.toUpperCase()
  return code === 'BULK10' && formData.value.quantity >= 3
})

const discountAmount = computed(() => {
  if (!isDiscountValid.value) return 0
  return Math.round(subtotal.value * 0.1)
})

// Computed: Total before insurance
const totalBeforeInsurance = computed(() => {
  return subtotal.value + shippingCost.value - discountAmount.value
})

// Computed: Insurance cost
const insuranceAmount = computed(() => {
  if (!formData.value.insurance) return 0
  return Math.round(totalBeforeInsurance.value * 0.05)
})

const insuranceCost = computed(() => {
  const cost = Math.round(totalBeforeInsurance.value * 0.05)
  return `$${cost} (5% of order)`
})

// Computed: Total price
const totalPrice = computed(() => {
  return totalBeforeInsurance.value + insuranceAmount.value
})

// Computed: Form validation
const isFormValid = computed(() => {
  return formData.value.product && 
         formData.value.quantity > 0 && 
         formData.value.shipping
})

// Watch for product changes - reset dependent fields
watch(() => formData.value.product, (newProduct, oldProduct) => {
  if (newProduct !== oldProduct) {
    formData.value.quantity = 1
    formData.value.shipping = ''
    formData.value.insurance = false
    formData.value.discountCode = ''
  }
})

// Watch for quantity changes - adjust shipping if needed
watch(() => formData.value.quantity, (newQuantity) => {
  if (formData.value.shipping === 'express' && !isExpressAvailable.value) {
    formData.value.shipping = ''
  }
  if (formData.value.shipping === 'overnight' && !isOvernightAvailable.value) {
    formData.value.shipping = ''
  }
})

const handleSubmit = () => {
  if (isFormValid.value) {
    submitMessage.value = {
      type: 'success',
      text: `Order placed successfully! Total: $${totalPrice.value}`
    }
    
    // Reset form after 3 seconds
    setTimeout(() => {
      formData.value = {
        product: '',
        quantity: 1,
        shipping: '',
        insurance: false,
        discountCode: ''
      }
      submitMessage.value = null
    }, 3000)
  }
}
</script>

<style scoped>
.form-container {
  background: white;
  border-radius: 8px;
  padding: 2rem;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

h2 {
  color: #2c3e50;
  margin-bottom: 1.5rem;
  border-bottom: 2px solid #42b983;
  padding-bottom: 0.5rem;
}

h3 {
  color: #2c3e50;
  margin-bottom: 1rem;
}

.form-group {
  margin-bottom: 1.5rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
  color: #2c3e50;
  font-weight: 500;
}

input[type="text"],
input[type="number"],
select {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
  transition: border-color 0.3s;
}

input[type="text"]:focus,
input[type="number"]:focus,
select:focus {
  outline: none;
  border-color: #42b983;
}

select option:disabled {
  color: #999;
}

input[type="checkbox"] {
  margin-right: 0.5rem;
}

.helper-text {
  display: block;
  margin-top: 0.25rem;
  font-size: 0.875rem;
  color: #666;
}

.helper-text.success {
  color: #42b983;
  font-weight: 500;
}

.helper-text.error {
  color: #e74c3c;
}

.summary-section {
  background: #f8f9fa;
  border-radius: 6px;
  padding: 1.5rem;
  margin: 1.5rem 0;
}

.summary-item {
  display: flex;
  justify-content: space-between;
  padding: 0.5rem 0;
  border-bottom: 1px solid #e0e0e0;
}

.summary-item:last-child {
  border-bottom: none;
}

.summary-item.total {
  font-size: 1.25rem;
  padding-top: 1rem;
  margin-top: 0.5rem;
  border-top: 2px solid #2c3e50;
}

.summary-item .discount {
  color: #42b983;
  font-weight: 500;
}

.submit-btn {
  width: 100%;
  padding: 1rem;
  background: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 1.1rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.3s;
}

.submit-btn:hover:not(:disabled) {
  background: #36a372;
}

.submit-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.message {
  padding: 1rem;
  border-radius: 4px;
  margin-top: 1rem;
  text-align: center;
  font-weight: 500;
}

.message.success {
  background: #d4edda;
  color: #155724;
  border: 1px solid #c3e6cb;
}

.message.error {
  background: #f8d7da;
  color: #721c24;
  border: 1px solid #f5c6cb;
}
</style>
