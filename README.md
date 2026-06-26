# LiveMART E-commerce Platform

A comprehensive multi-user e-commerce platform supporting both B2C (Business-to-Consumer) and B2B (Business-to-Business) transactions with role-based access control, secure authentication, and advanced product management.

## 🎯 Platform Overview

LiveMART is a full-stack e-commerce system where:
- **Customers** browse and purchase products from retailers (B2C)
- **Retailers** buy bulk products from wholesalers and sell to customers (B2B)
- **Wholesalers** create products and sell bulk quantities to retailers
- **Admins** approve retailers and wholesalers before they can operate

## 👥 User Roles & Capabilities

### 1. Customer (B2C)
- Browse products from retailers
- Add items to cart with persistent storage
- Place orders online (Razorpay) or Cash on Delivery
- View order history and track status
- Write product reviews
- Auto-login after OTP registration

### 2. Retailer (B2B)
- Browse wholesaler products
- Manage B2B cart (localStorage-backed)
- Purchase bulk from wholesalers with tiered pricing
- Create own products for B2C customers
- Add wholesaler products as proxy products with custom markup
- Manage inventory and view/update order status
- Requires admin approval before activation

### 3. Wholesaler (B2B)
- Create products with wholesale pricing
- Bulk upload products via CSV
- Set quantity-based tiered pricing/discounts
- View B2B orders from retailers
- Manage order status and fulfillment
- Requires admin approval before activation

### 4. Admin
- Approve/reject retailer and wholesaler registrations
- View pending approval requests
- Manage platform users
- Monitor system activity

## 🔐 Authentication & Security

### Two Authentication Options

**Option 1: OTP (Email Verification)**
- User enters email/password
- 6-digit OTP sent via email
- 10-minute expiration
- Works for registration and login

**Option 2: OAuth (Social Login)**
- Google OAuth integration
- Auto-registered as customers
- No OTP needed
- Profile pictures from OAuth providers stored

### Security Features
- JWT tokens for session management
- Password hashing with bcrypt
- Role-based access control (RBAC)
- TTL index for automatic OTP cleanup
- OTP expiration enforcement

## 🛒 Shopping Cart System

### B2C Cart (Customer)
- **Storage**: MongoDB database
- **Persistence**: Across sessions
- **Features**:
  - Real-time stock validation
  - Quantity updates
  - Item removal
  - Automatic price calculation with tax & delivery

### B2B Cart (Retailer)
- **Storage**: Browser localStorage
- **Features**:
  - Client-side only
  - Bulk quantity management
  - Wholesale pricing calculation
  - Tiered pricing support
  - Remove/update items on checkout

## 💳 Payment System

### Razorpay Integration
1. Backend creates Razorpay payment order
2. Frontend opens Razorpay payment modal
3. User completes payment (credit/debit/UPI, netbanking)
4. Signature verification on backend
5. Stock deduction upon verification
6. Order confirmation with email notifications

### Payment Methods
- **Online**: Razorpay (credit/debit card, UPI, netbanking)
- **COD**: Cash on Delivery (stock deducted immediately)

### Payment Flow
```
Order Created → Payment Initiated → User Pays → 
Payment Verified → Stock Deused → Order Confirmed → 
Emails Sent → Inventory Updated (B2B)
```

## 📆 Product Management

### Product Types

**1. Retailer Products (B2C)**
- Created directly by retailer
- Standard retail pricing
- Sold to customers

**2. Wholesaler Products (B2B)**
- Created by wholesaler
- Wholesale pricing with quantity-based tiers
- Bulk upload support via CSV
- Sold to retailers

**3. Proxy Products (Retailer)**
- Retailer adds wholesaler product to their store
- Deducts stock from wholesaler
- Can set custom markup/margin
- Creates new retailer product or updates existing

### Product Schema
```javascript
{
  name, description, category, images,
  retailPrice, stock,
  owner, ownerType: ['retailer', 'wholesaler'],
  wholesalEnabled, wholesalePrice, wholesaleMinQty,
  wholesaleTiers: [{ minQty, discountPercent }],
  sourceWholesaler, // For proxy products
  isLocalProduct,
  region,
  averageRating, numberofReviews
}
```

## 💰 Pricing System

### B2C Pricing
- Retail price from product
- 5% tax applied
- Delivery fee: ₹350 (free above ₹500)

### B2B Pricing (Wholesaler → Retailer)
- Wholesale price from product
- Tiered pricing based on quantity:
  - Min 10 units: 5% discount
  - Min 50 units: 10% discount
  - Min 100+ units: 15% discount
- Minimum order quantity enforcement
- Tax and delivery calculated separately

### Tiered Pricing Example
```javascript
wholesaleTiers: [
  { minQty: 10, discountPercent: 5 },
  { minQty: 50, discountPercent: 10 },
  { minQty: 100, discountPercent: 15 }
]
```

## 📋 Order Management

### Order Types

**B2C Orders (Customer → Retailer)**
- From B2C cart checkout
- Payment: Online (Razorpay) or COD
- Status flow: pending → confirmed → processing → shipped → delivered
- Can be cancelled (restores stock)

**B2B Orders (Retailer → Wholesaler)**
- From B2B cart checkout
- Payment: Online or COD
- Auto-inventory: Products automatically added to retailer inventory after payment
- Order tracking available

### Stock Management
- **Deduction**: Online after payment verification, COD immediately on order
- **Restoration**: On order cancellation (prevents duplicate restoration)
- **Validation**: Stock checked before order creation and before payment verification

## 📈 Location-Based Features

- Geospatial indexing for retailer location queries
- Location-based product filtering
- Delivery availability by region
- Enables local shopping experience

## 🔄 Workflow

### Customer Journey
1. Register/Login (OTP or OAuth) → Browse products → Add to cart → Checkout → Pay → Order placed → Track status → Receive product → Write review

### Retailer Journey
1. Register (needs approval) → Get approved → Browse wholesalers → Add to B2B cart → Order → Products added to inventory → Create own products → Sell to customers

### Wholesaler Journey
1. Register (needs approval) → Get approved → Create products → Set pricing/tiers → Bulk upload CSV → Receive orders from retailers → Manage fulfillment

## 🏷️ Tech Stack

### Backend
- **Runtime**: Node.js
- **Framework**: Express.js
- **Database**: MongoDB (Mongoose ODM)
- **Authentication**: JWT tokens
- **Email**: Nodemailer (OTP and notifications)
- **Payment**: Razorpay API
- **File Upload**: Multer
- **CSV Parsing**: CSV parsing for bulk uploads

### Frontend
- **Framework**: React
- **State Management**: React Context API
- **Routing**: React Router
- **HTTP**: Axios
- **UI Components**: Custom CSS styling
- **Payment Modal**: Razorpay integration

### Infrastructure
- **Deployment**: [Your deployment platform]
- **Database**: MongoDB Atlas (or local)
- **Email Service**: Nodemailer with SMTP

## 📁 Project Structure

```
backend/
├── controllers/       # Request handlers for each resource
│   ├── adminController.js
│   ├── authController.js
│   ├── productController.js
│   ├── orderController.js
│   ├── cartController.js
│   ├── paymentController.js
│   └── ...
├── models/           # Database schemas
│   ├── userModel.js
│   ├── productModel.js
│   ├── orderModel.js
│   └── ...
├── routes/           # API route definitions
│   ├── authRoutes.js
│   ├── productRoutes.js
│   ├── orderRoutes.js
│   └── ...
├── middleware/       # Authentication, validation, error handling
├── services/         # Business logic (email, OTP, etc.)
├── utils/           # Utility functions
├── config/          # Configuration files
└── server.js        # Entry point

frontend/
├── src/
│   ├── components/     # Reusable React components
│   │   ├── Navbar.js
│   │   ├── ConfirmDialog.js
│   │   ├── LocationPicker.js
│   │   └── Toast.js
│   ├── pages/         # Page components
│   │   ├── Home.js
│   │   ├── Login.js
│   │   ├── Cart.js
│   │   ├── Checkout.js
│   │   ├── AdminDashboard.js
│   │   ├── RetailerDashboard.js
│   │   └── WholesalerDashboard.js
│   ├── context/       # React Context for state
│   ├── services/      # API calls and business logic
│   ├── utils/         # Utility functions
│   ├── assets/        # Images and static files
│   ├── App.js
│   └── index.js
└── public/           # Static files
```

## 🚀 Getting Started

### Prerequisites
- Node.js (v14+)
- MongoDB
- npm or yarn

### Backend Setup
```bash
cd backend
npm install
cp .env.example .env
# Configure environment variables
npm start
```

### Frontend Setup
```bash
cd frontend
npm install
npm start
```

### Environment Variables (.env)
```
MONGODB_URI=mongodb://localhost:27017/ecommerce
JWT_SECRET=your_jwt_secret
RAZORPAY_KEY_ID=your_razorpay_key
RAZORPAY_KEY_SECRET=your_razorpay_secret
GMAIL_USER=your_gmail@gmail.com
GMAIL_PASSWORD=your_app_password
GOOGLE_CLIENT_ID=your_google_oauth_id
GOOGLE_CLIENT_SECRET=your_google_oauth_secret
```

## 📄 API Endpoints

### Authentication
- `POST /api/auth/register` - Register new user
- `POST /api/auth/login` - Login user
- `POST /api/auth/verify-otp` - Verify OTP
- `POST /api/auth/oauth-login` - OAuth login

### Products
- `GET /api/products` - Get all products (with filters)
- `GET /api/products/:id` - Get single product
- `POST /api/products` - Create product (authenticated)
- `PUT /api/products/:id` - Update product (authenticated)
- `DELETE /api/products/:id` - Delete product (authenticated)

### Orders
- `GET /api/orders` - Get user orders
- `POST /api/orders` - Create order
- `PUT /api/orders/:id/status` - Update order status
- `POST /api/orders/:id/cancel` - Cancel order

### Cart
- `GET /api/cart` - Get cart (B2C only)
- `POST /api/cart` - Add to cart
- `PUT /api/cart/:itemId` - Update cart item
- `DELETE /api/cart/:itemId` - Remove from cart

### Admin
- `GET /api/admin/pending-approvals` - Get pending retailer/wholesaler approvals
- `POST /api/admin/approve/:userId` - Approve user
- `POST /api/admin/reject/:userId` - Reject user

## 🪨 Testing

```bash
# Backend tests
cd backend
npm test

# Frontend tests
cd frontend
npm test
``
### Using Cloud Platforms
- Deploy backend to Heroku, AWS, or DigitalOcean
- Deploy frontend to Vercel, Netlify, or AWS S3
- Use MongoDB Atlas for database

## 👨‍💻 Contributors

- Mark nikhil pandiri ID:2024A7PS0069
- Visakh Menon ID: 2024A7PS008H
- Aarav Bhattacharya ID: 2024A7PS0115H
- Ishan Kumar ID: 2024A7PS0159H
#   E c o m m e r c e w e b s i t e  
 