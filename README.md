# Fronto Form Test

A Vue.js test project that features a form with inter-dependent input fields, demonstrating reactive form validation and dynamic field interactions.

## Features

This project showcases an order form with the following inter-dependent fields:

### Form Dependencies

1. **Product Selection** → Triggers:
   - Quantity field (becomes visible)
   - Resets all dependent fields when changed

2. **Quantity** → Affects:
   - Shipping options availability (Express disabled if > 5 items, Overnight disabled if > 2 items)
   - Discount code field visibility (appears when quantity ≥ 3)
   - Order total calculations

3. **Shipping Method** → Impacts:
   - Total price calculation
   - Available based on quantity constraints

4. **Discount Code** → Provides:
   - 10% discount validation (BULK10 code for orders with 3+ items)
   - Real-time validation feedback
   - Updates total price

5. **Insurance Option** → Calculates:
   - 5% of order total dynamically
   - Recommended for orders over $1000

### Real-time Features

- Dynamic price calculations
- Form validation with visual feedback
- Conditional field visibility
- Disabled states for unavailable options
- Order summary that updates in real-time
- Success message on form submission
- Auto-reset form after successful submission

## Tech Stack

- **Vue 3** - Progressive JavaScript framework
- **Vite** - Next-generation frontend tooling
- **Composition API** - Vue's modern reactive API with `<script setup>`

## Getting Started

### Prerequisites

- Node.js (v14 or higher)
- npm or yarn

### Installation

```bash
npm install
```

### Development

Run the development server:

```bash
npm run dev
```

The application will be available at `http://localhost:5173/`

### Build

Build for production:

```bash
npm run build
```

### Preview Production Build

```bash
npm run preview
```

## Project Structure

```
fronto-form-test/
├── src/
│   ├── components/
│   │   └── DependentForm.vue    # Main form component with dependencies
│   ├── App.vue                   # Root component
│   ├── main.js                   # Application entry point
│   └── style.css                 # Global styles
├── index.html                    # HTML template
├── vite.config.js               # Vite configuration
└── package.json                 # Project dependencies
```

## Usage Examples

### Test the Dependencies

1. Select a product (e.g., Laptop)
2. Change quantity to 3+ to see the discount code field appear
3. Change quantity to 6+ to see Express shipping become disabled
4. Change quantity to 3+ to see Overnight shipping become disabled (Overnight only available for ≤ 2 items)
5. Enter discount code "BULK10" to see 10% discount applied
6. Select shipping method to enable insurance option
7. Toggle insurance to see total price update
8. Submit the form to see success message

## License

This project is a test/demonstration project.
