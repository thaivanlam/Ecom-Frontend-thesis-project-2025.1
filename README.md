# TechZone Laptop — E-commerce Frontend

A modern, full-featured e-commerce frontend for a laptop retail platform, built with **React 19**, **Redux Toolkit**, and **Tailwind CSS v4**. This application is the client-facing layer of a microservices-based e-commerce platform designed for the graduation thesis: *"Laptop E-commerce Platform Using Microservices Architecture"*.

---

## Table of Contents

1. [Application Flow](#application-flow)
2. [Technology Choices](#technology-choices)
3. [Project Structure](#project-structure)
4. [Key Features](#key-features)
5. [Trade-offs & Design Decisions](#trade-offs--design-decisions)
6. [Getting Started](#getting-started)
7. [Environment Variables](#environment-variables)
8. [License](#license)

---

## Application Flow

### Customer Journey

```
Home Page → Browse Products (with filters) → View Product Details
  → Add to Cart → Cart Review → Checkout Flow:
      Step 1: Select/Add Delivery Address
      Step 2: Choose Payment Method (Stripe)
      Step 3: Review Order Summary
      Step 4: Complete Stripe Payment → Order Confirmation
```

After placing an order, customers can track and manage orders from **Profile → My Orders**, where they can view order status and cancel pending orders.

### Authentication Flow

```
Register (select role: Customer / Seller / Admin)
  → Login (JWT cookie-based auth via withCredentials)
    → Role-based redirect:
        Customer → Home page (shop, cart, orders)
        Seller   → Admin panel (own products & orders only)
        Admin    → Admin panel (full access: dashboard, products,
                   categories, sellers, customers, all orders)
```

Authentication state is persisted in `localStorage` and rehydrated into the Redux store on page load. The API layer uses `axios` with `withCredentials: true` to send HTTP-only cookies for session management.

### Admin/Seller Panel Flow

```
Admin Layout (sidebar navigation)
  ├── Dashboard — analytics overview with Chart.js visualizations
  ├── Products  — CRUD with image upload, SKU auto-generation, specifications
  ├── Orders    — view all orders, update status (Pending → Shipped → Delivered)
  ├── Categories — CRUD for product categories
  ├── Sellers   — manage seller accounts (admin only)
  └── Customers — manage customer accounts (admin only)
```

Sellers see a restricted view: only **Products** (their own) and **Orders** (their own).

### Data Flow (Frontend ↔ Backend)

```
React Component
  → dispatches Redux action (thunk)
    → axios call to API Gateway (port 8080)
      → Gateway routes to microservice:
           /product-manager/** → Product Service
           /user-manager/**    → User Service
           /order-manager/**   → Order Service
      → Response flows back through Redux → Component re-renders
```

All API calls pass through a single gateway endpoint. The frontend is unaware of individual microservice addresses — it only knows the gateway URL configured in `.env`.

---

## Technology Choices

### Core Stack

| Technology | Version | Purpose | Why chosen |
|---|---|---|---|
| **React** | 19.1 | UI library | Component-based architecture, large ecosystem, hooks API for clean state logic |
| **Redux Toolkit** | 2.9 | Global state management | Centralized store for auth, cart, products, and orders; thunks for async API calls; predictable state updates across deeply nested components |
| **React Router DOM** | 7.9 | Client-side routing | Declarative routing with nested layouts, protected routes, and URL-based filtering |
| **Tailwind CSS** | 4.1 (via Vite plugin) | Styling | Utility-first approach enables rapid UI development without context-switching to CSS files; v4's native Vite integration eliminates PostCSS configuration |
| **Vite** | 7.1 | Build tool | Near-instant HMR, fast cold starts, native ESM support — significantly faster than Create React App / Webpack |

### UI & Component Libraries

| Library | Purpose | Why chosen |
|---|---|---|
| **MUI (Material UI)** | DataGrid tables, form controls, dialogs | Production-ready data tables with server-side pagination, sorting; consistent form components |
| **Headless UI** | Modals, listboxes, dialogs | Unstyled, accessible primitives that integrate cleanly with Tailwind (no style conflicts with MUI) |
| **React Icons** | Icon library | Single import for Font Awesome, Material, Heroicons — avoids installing multiple icon packages |
| **React Hook Form** | Form validation | Minimal re-renders, built-in validation rules, cleaner code than controlled inputs with useState |
| **React Hot Toast** | Notifications | Lightweight toast system with zero config — simpler than MUI Snackbar for most use cases |
| **Chart.js + react-chartjs-2** | Dashboard charts | Lightweight charting with Line, Bar, and Doughnut charts for admin analytics |
| **Swiper** | Image carousel/slider | Touch-friendly, performant slider for the home page banner |

### Payment & Utilities

| Library | Purpose |
|---|---|
| **@stripe/react-stripe-js + @stripe/stripe-js** | Stripe Elements integration for secure card payments |
| **axios** | HTTP client with interceptors, cookie support (`withCredentials`), and clean error handling |
| **classnames** | Conditional CSS class composition (used in Sidebar active state logic) |
| **react-loader-spinner** | Loading animations (Vortex spinner) |

---

## Project Structure

```
src/
├── api/
│   └── api.js                  # Axios instance with base URL & credentials
├── components/
│   ├── admin/                  # Admin panel pages
│   │   ├── dashboard/          # Analytics dashboard with charts
│   │   ├── products/           # Product CRUD, image upload, specifications
│   │   ├── orders/             # Order management & status updates
│   │   ├── categories/         # Category CRUD
│   │   ├── sellers/            # Seller management
│   │   ├── customers/          # Customer management
│   │   └── AdminLayout.jsx     # Sidebar + top nav layout wrapper
│   ├── auth/                   # Login & Register pages
│   ├── cart/                   # Cart page, item content, quantity controls
│   ├── checkout/               # Multi-step checkout flow (address, payment, summary)
│   ├── home/                   # Home page with banner slider
│   ├── modal/                  # Product specification modal
│   ├── order/                  # Customer order history & management
│   ├── products/               # Product listing with advanced filters
│   ├── profile/                # User profile & address management
│   ├── shared/                 # Reusable components (Navbar, Sidebar, Loader, etc.)
│   └── helper/
│       └── tableColumn.jsx     # DataGrid column definitions for all tables
├── hooks/                      # Custom hooks for URL-based filtering
│   ├── useProductFilter.js     # Product page filter → URL params → API call
│   ├── useOrderFilter.js       # Admin order filter
│   ├── useCustomerOrderFilter.js
│   ├── useCategoryFilter.js
│   ├── useSellerFilter.js
│   └── useCustomerFilter.js
├── store/
│   ├── action/index.js         # All Redux thunk actions (API calls)
│   └── reducers/
│       ├── store.js            # Redux store configuration with preloaded state
│       ├── ProductReducer.js   # Products, categories, brands, pagination
│       ├── authReducer.js      # User, addresses, sellers, customers, client secret
│       ├── cartReducer.js      # Cart items, total price, cart ID
│       ├── orderReducer.js     # Admin orders, customer orders
│       ├── adminReducer.js     # Analytics data
│       ├── errorReducer.js     # Loading states, error messages
│       └── paymentMethodReducer.js
└── utils/
    ├── index.js                # Navigation configs, banner data
    ├── formatPrice.js          # Currency formatting & revenue abbreviation
    ├── truncateText.js         # Text truncation utility
    └── constant.js             # Asset imports
```

---

## Key Features

### Product Browsing & Filtering
- **Category filter** via dropdown (server-side)
- **Price range filter** with slider and manual input
- **Advanced filters**: Brand, Processor (CPU), RAM, Storage — all rendered as checkbox groups inside collapsible accordions
- **Keyword search** with 700ms debounce to avoid excessive API calls
- **Sort by price** (ascending/descending)
- **Server-side pagination** — all filter state is synced to URL query params, enabling shareable/bookmarkable URLs

### Shopping Cart
- Cart state stored in Redux and persisted to `localStorage`
- Quantity controls with stock validation (checks against product inventory)
- Real-time price calculation with savings display
- Cart survives page refreshes; cleared on successful order placement

### Checkout
- 4-step wizard: Address → Payment Method → Order Summary → Payment
- Address CRUD with selection UI
- Stripe Elements integration (test mode) for secure card input
- Payment confirmation page handles Stripe redirect and places order via backend API

### Admin Dashboard
- Revenue, orders, products, and customer count cards
- Revenue trend (Line chart), order status distribution (Doughnut), top products (Bar chart)
- All charts use mock data — in production, these would come from dedicated analytics endpoints

### Role-Based Access Control (RBAC)
- `PrivateRoute` component handles three modes: `publicPage` (redirect logged-in users away), `adminOnly` (restrict to admin/seller), and default (require authentication)
- Sellers are restricted to `/admin/orders` and `/admin/products` paths
- Conditional rendering throughout the UI based on `user.roles`

### Product Specifications
- Admin/seller can add technical specs (CPU, RAM, Storage, Display, GPU) per product
- Specs are fetched and displayed in the product detail modal for customers
- Spec modal uses select dropdowns for standardized values (enables filtering)

---

## Trade-offs & Design Decisions

### 1. Redux with Plain Reducers vs. RTK Slices

**Decision**: Used Redux Toolkit's `configureStore` but wrote reducers with manual `switch/case` statements and string action types instead of `createSlice`.

**Why**: The project was built incrementally — starting with basic Redux patterns and adding features over time. `createSlice` would reduce boilerplate (auto-generated action types, Immer-based immutability), but the current structure was sufficient for a thesis-scale project.

**Trade-off**: More verbose code (~600 lines in `action/index.js`), risk of typos in action type strings. For a larger team or longer-lived project, migrating to `createSlice` + `createAsyncThunk` would be beneficial.

### 2. MUI + Tailwind CSS (Dual Styling Systems)

**Decision**: Used MUI for complex interactive components (DataGrid, Select, Radio, Stepper) and Tailwind for everything else (layout, cards, buttons, responsive design).

**Why**: MUI's DataGrid provides server-side pagination, column resizing, and sorting out of the box — rebuilding this in pure Tailwind would take significant effort. Meanwhile, Tailwind enables rapid custom styling without vendor-specific component APIs.

**Trade-off**: Two styling paradigms in one project increases bundle size and cognitive overhead. MUI's default styles occasionally conflict with Tailwind (e.g., button colors, font sizes). The `!important` overrides in Tailwind config mitigate most conflicts.

### 3. Client-Side Cart with Server Sync

**Decision**: Cart is managed entirely in Redux + localStorage for guest users. When a logged-in user proceeds to checkout, the cart is synced to the backend via `POST /cart/create`.

**Why**: Allows browsing and adding to cart without authentication — a common e-commerce pattern that reduces friction. Server-side cart is only needed at checkout time for order processing.

**Trade-off**: Cart state can become stale if product prices or stock change between adding to cart and checkout. A more robust solution would validate cart items against current inventory at checkout time.

### 4. URL-Based Filter State (Search Params)

**Decision**: All product filters (category, sort order, price range, brand, processor, RAM, storage, keyword, page number) are stored as URL query parameters, not just in component state.

**Why**: Enables shareable/bookmarkable URLs (e.g., `/products?category=Gaming&brands=ASUS,MSI&minPrice=1000`). Custom hooks (`useProductFilter`, etc.) read URL params and dispatch the corresponding API calls automatically.

**Trade-off**: More complex state management — every filter change requires URL navigation, which triggers a re-render cycle. The 700ms debounce on search input mitigates excessive API calls, but a global filter cache would further improve performance.

### 5. Single Action File vs. Feature-Based Splitting

**Decision**: All Redux actions (~90 thunks) live in a single `store/action/index.js` file.

**Why**: Started small and grew organically. For a thesis project with a single developer, having everything in one file makes global search easy.

**Trade-off**: The file is ~500+ lines and mixes concerns (auth, products, cart, orders, admin). For a production app, splitting into `authActions.js`, `productActions.js`, `cartActions.js`, etc. would improve maintainability.

### 6. Hardcoded Dashboard Analytics

**Decision**: Dashboard cards and charts use a mix of real API data (product count, total revenue, total orders) and mock/hardcoded values (today's orders, conversion rate, shipped today).

**Why**: The backend analytics endpoints return aggregate counts. Building real-time, time-series analytics would require significant backend work (event sourcing, time-bucketed queries) beyond the scope of this thesis.

**Trade-off**: The dashboard looks complete visually but isn't fully functional. This is documented as a known limitation and potential future improvement.

### 7. Role Selection at Registration

**Decision**: Users can self-select their role (Customer, Seller, or Admin) during registration.

**Why**: Simplifies development and testing — no separate admin provisioning flow needed. In a production system, seller/admin roles would require approval workflows.

**Trade-off**: Security concern — anyone can register as an admin. This is acceptable for a thesis demo but would need role approval or invitation-based registration in production.

### 8. Axios Instance with Cookie Auth

**Decision**: Created a single axios instance (`api/api.js`) with `withCredentials: true` and the gateway base URL. All API calls use this instance.

**Why**: Centralizes auth configuration. The backend uses HTTP-only cookies for JWT tokens, so `withCredentials` must be set on every request. A single instance avoids repeating this config.

**Trade-off**: No request/response interceptors for automatic token refresh or 401 handling. If the session expires, users see raw error messages instead of being redirected to login. Adding an interceptor chain would improve UX significantly.

---

## Getting Started

### Prerequisites
- Node.js >= 20.19.0
- npm or yarn
- Backend microservices running (API Gateway on port 8080)

### Installation

```bash
# Clone the repository
git clone <repository-url>
cd ecom-frontend

# Install dependencies
npm install

# Start development server
npm run dev
```

The app runs on `http://localhost:5173` by default.

### Build for Production

```bash
npm run build
npm run preview  # Preview the production build locally
```

---

## Environment Variables

Create a `.env` file in the project root:

```env
# Backend API Gateway URL
VITE_BACK_END_URL=http://localhost:8080

# Stripe publishable key (test mode)
VITE_STRIPE_PUBLISHABLE_KEY=pk_test_xxx

# Frontend URL (used for Stripe payment redirect)
VITE_FRONTEND_URL=http://localhost:5173
```

---

## Related Services

This frontend connects to the following backend microservices through the API Gateway:

| Service | Gateway Path | Responsibility |
|---|---|---|
| **User Service** | `/user-manager/**` | Authentication, user management, addresses |
| **Product Service** | `/product-manager/**` | Products, categories, brands, specifications |
| **Order Service** | `/order-manager/**` | Cart, orders, Stripe payments, analytics |
| **Config Server** | — | Centralized configuration (not exposed to frontend) |
| **Discovery Service** | — | Eureka service registry (not exposed to frontend) |

---

## License

This project is licensed under the **GNU Affero General Public License v3.0 (AGPL-3.0)**.

### What This Means

- ✅ **You can**: Use, modify, and distribute this code freely
- ✅ **You must**: Include a copy of the license and state changes made
- ✅ **You must**: Disclose the source code if you deploy modified versions as a network service
- ❌ **You cannot**: Use this code in proprietary/closed-source projects without permission

### Copyright

**Copyright © 2025 Thái Văn Lâm**

This program is free software: you can redistribute it and/or modify it under the terms of the GNU Affero General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but **WITHOUT ANY WARRANTY**; without even the implied warranty of **MERCHANTABILITY** or **FITNESS FOR A PARTICULAR PURPOSE**. See the [LICENSE](./LICENSE) file for more details.

For more information about AGPL-3.0, visit: [https://www.gnu.org/licenses/agpl-3.0.html](https://www.gnu.org/licenses/agpl-3.0.html)

---

*Built as part of the graduation thesis: "Laptop E-commerce Platform Using Microservices Architecture"*
