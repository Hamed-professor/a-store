# α Store - Iranian Payment Gateway Integration (Part 4A)

## Overview

Complete Iranian payment gateway integration for α Store luxury e-commerce platform, supporting ZarinPal, Pay.ir, and NextPay with failover capabilities, Persian UI, and comprehensive error handling.

## Features

### 🔐 Payment Gateways
- **ZarinPal** (Primary) - Most popular Iranian gateway
- **Pay.ir** (Secondary) - Fast processing alternative  
- **NextPay** (Tertiary) - Reliable backup option
- **Automatic Failover** - Seamless gateway switching

### 💳 Payment Flow
- Multi-step payment process
- Real-time payment validation
- Secure callback verification
- Payment status tracking
- Receipt generation

### 🎨 Persian UI/UX
- RTL layout support
- Persian payment descriptions
- Iranian Rial/Toman currency
- Culturally appropriate design
- Mobile-optimized interface

### 🛡️ Security Features
- Payment token validation
- Amount tampering prevention
- CSRF protection
- Secure callback verification
- PCI compliance preparation

## Installation

1. **Install Dependencies**
```bash
npm install axios framer-motion lucide-react
```

2. **Environment Setup**
```bash
cp .env.example .env.local
# Fill in your gateway credentials
```

3. **Configure Payment Gateways**
```bash
# Get your credentials from:
# ZarinPal: https://dashboard.zarinpal.com
# Pay.ir: https://pay.ir/panel
# NextPay: https://nextpay.org/panel
```

## Gateway Setup

### ZarinPal Configuration
```env
ZARINPAL_MERCHANT_ID=your-merchant-id
# Sandbox: 36dp7z89-6de4-40d9-ae0b-b8b4w5lrm0p0
```

### Pay.ir Configuration
```env
PAYIR_API_KEY=your-api-key
```

### NextPay Configuration
```env
NEXTPAY_API_KEY=your-api-key
```

## Usage Examples

### Basic Payment Integration
```tsx
import { usePayment } from '../src/hooks/usePayment';
import PaymentMethods from '../src/components/payment/PaymentMethods';

function CheckoutPage() {
  const {
    gateways,
    selectedGateway,
    initiatePayment,
    isProcessing,
    error
  } = usePayment();

  const handlePayment = async (formData) => {
    const success = await initiatePayment(formData, cartItems);
    if (success) {
      // User will be redirected to gateway
    }
  };

  return (
    <PaymentMethods
      gateways={gateways}
      selectedGateway={selectedGateway}
      amount={totalAmount}
      currency="IRT"
      onSubmitPayment={handlePayment}
    />
  );
}
```

### Payment Verification
```tsx
import { useRouter } from 'next/router';
import { parseCallbackParams } from '../src/utils/payment';

function PaymentCallback() {
  const router = useRouter();
  
  useEffect(() => {
    const params = parseCallbackParams(router.query);
    verifyPayment(params);
  }, [router.query]);
}
```

## API Endpoints

### Payment Request
```http
POST /api/payments/request
Content-Type: application/json

{
  "orderId": "AS123456789",
  "amount": 150000,
  "currency": "IRT",
  "gateway": "zarinpal",
  "description": "سفارش از α Store"
}
```

### Payment Verification
```http
POST /api/payments/verify
Content-Type: application/json

{
  "gateway": "zarinpal",
  "authority": "A00000000000000000000000000123456789",
  "orderId": "AS123456789"
}
```

## File Structure

```
alpha-store/
├── src/
│   ├── services/payments/
│   │   ├── zarinpal.ts         # ZarinPal integration
│   │   ├── payir.ts           # Pay.ir integration
│   │   └── nextpay.ts         # NextPay integration
│   ├── components/payment/
│   │   ├── PaymentMethods.tsx  # Gateway selection
│   │   ├── PaymentGateway.tsx  # Redirect interface
│   │   └── PaymentStatus.tsx   # Status display
│   ├── hooks/
│   │   └── usePayment.ts      # Payment hook
│   ├── utils/
│   │   └── payment.ts         # Utilities
│   └── types/
│       └── payment.ts         # TypeScript types
├── pages/
│   ├── api/payments/
│   │   ├── request.ts         # Payment request API
│   │   ├── verify.ts          # Verification API
│   │   └── zarinpal.ts        # ZarinPal callback
│   └── payment/
│       ├── index.tsx          # Payment page
│       └── callback.tsx       # Callback handler
└── .env.example               # Environment variables
```

## Component Documentation

### PaymentMethods
Primary payment interface component.

**Props:**
- `gateways`: Available payment gateways
- `selectedGateway`: Currently selected gateway
- `amount`: Payment amount
- `currency`: 'IRR' or 'IRT'
- `onSubmitPayment`: Payment submission handler

### PaymentStatus
Payment result display component.

**Props:**
- `status`: Payment status enum
- `verification`: Verification details
- `onRetry`: Retry payment handler
- `onDownloadReceipt`: Receipt download handler

### usePayment Hook
Complete payment management hook.

**Returns:**
- `gateways`: Available gateways
- `initiatePayment`: Start payment process
- `verifyPayment`: Verify payment result
- `resetPayment`: Reset payment state

## Error Handling

### Error Types
```typescript
interface PaymentError {
  code: string;
  message: string;
  messagePersian: string;
  retryable: boolean;
}
```

### Common Error Codes
- `NETWORK_ERROR`: Connection issues
- `INVALID_AMOUNT`: Amount validation failed
- `GATEWAY_ERROR`: Gateway-specific error
- `VERIFICATION_FAILED`: Payment verification failed

### Error Recovery
```typescript
const handleError = (error: PaymentError) => {
  if (error.retryable) {
    // Show retry button
    setShowRetry(true);
  } else {
    // Show permanent error message
    showErrorDialog(error.messagePersian);
  }
};
```

## Security Considerations

### Payment Validation
- Amount integrity checking
- Gateway signature verification
- User session validation
- CSRF token validation

### Data Protection
- Sensitive data sanitization
- Payment logs anonymization
- Secure callback URLs
- PCI compliance preparation

## Testing

### Development Testing
```bash
# Use sandbox credentials
ZARINPAL_MERCHANT_ID=36dp7z89-6de4-40d9-ae0b-b8b4w5lrm0p0
NODE_ENV=development
```

### Test Scenarios
1. Successful payment flow
2. Payment cancellation
3. Network failure recovery
4. Gateway timeout handling
5. Invalid amount rejection

## Production Deployment

### Environment Setup
```env
NODE_ENV=production
ZARINPAL_MERCHANT_ID=your-production-merchant-id
PAYIR_API_KEY=your-production-api-key
NEXTPAY_API_KEY=your-production-api-key
```

### SSL Requirements
- HTTPS required for production
- Valid SSL certificate
- Secure callback URLs

### Monitoring
- Payment success/failure rates
- Gateway response times
- Error rate monitoring
- Transaction volume tracking

## Persian Localization

### Currency Support
- Iranian Rial (IRR)
- Toman (IRT) - 1 Toman = 10 Rials
- Automatic conversion

### Text Direction
- RTL layout support
- Persian number formatting
- Culturally appropriate messaging

## Performance Optimization

### Caching Strategy
- Gateway availability caching
- Payment method caching
- Static asset optimization

### Bundle Optimization
- Code splitting by gateway
- Lazy loading components
- Tree shaking unused code

## Troubleshooting

### Common Issues
1. **Payment Not Processing**
   - Check gateway credentials
   - Verify network connectivity
   - Check amount limits

2. **Verification Failing**
   - Confirm callback URL accessibility
   - Check SSL certificate
   - Verify merchant account status

3. **UI Display Issues**
   - Check RTL CSS support
   - Verify Persian font loading
   - Test mobile responsiveness

### Debug Mode
```env
DEBUG_PAYMENT_LOGS=true
```

## Contributing

### Development Workflow
1. Clone repository
2. Install dependencies
3. Set up environment variables
4. Run development server
5. Test payment flows

### Code Standards
- TypeScript strict mode
- Persian UI text
- Comprehensive error handling
- Mobile-first design

## License

This payment integration is part of α Store and follows the same licensing terms.

## Support

For payment integration support:
- Email: dev@alphastore.ir
- Documentation: https://docs.alphastore.ir
- Issues: GitHub Issues

---

**α Store Payment Integration v4A**  
*Secure, Persian, Professional*