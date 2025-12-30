# Frontend to Backend API Mapping

## Table of Contents
- [Overview](#overview)
- [Conventions](#conventions)
- [1. Authentication (User and Admin)](#1-authentication-user-and-admin)
- [2. Product Catalog](#2-product-catalog)
- [3. Wishlist](#3-wishlist)
- [4. Cart](#4-cart)
- [5. Checkout and Orders (User)](#5-checkout-and-orders-user)
- [6. User Profile and Addresses](#6-user-profile-and-addresses)
- [7. Contact and Support](#7-contact-and-support)
- [8. Uploads (Image Handling)](#8-uploads-image-handling)
- [9. Admin Area](#9-admin-area)
  - [9.1 Admin Dashboard KPIs](#91-admin-dashboard-kpis)
  - [9.2 Admin: Products CRUD](#92-admin-products-crud)
  - [9.3 Admin: Orders Management](#93-admin-orders-management)
  - [9.4 Admin: Users/Admins Management](#94-admin-usersadmins-management)
  - [9.5 Admin: Contact Messages](#95-admin-contact-messages)
- [Appendix: Schemas Referenced](#appendix-schemas-referenced)
- [Appendix: Notes and Gaps](#appendix-notes-and-gaps)

## Overview
This document maps planned frontend pages and components for the React SPA to the FastAPI backend endpoints defined in the OpenAPI specification. It covers user-facing and admin flows. For each page, we outline the purpose, key UI components, backend routes and methods, primary request/response schemas, and authentication requirements.

Source of truth for endpoints and schemas is modern-application-migration-301960/backend_api/interfaces/openapi.json.

## Conventions
- Auth: Bearer JWT tokens in Authorization header, using “Bearer <token>”
- User vs Admin tokens: Admin tokens encode subject as admin:<id>
- Pagination and filtering are via query parameters as documented
- All protected endpoints show “Auth: User” or “Auth: Admin” below

## 1. Authentication (User and Admin)
### Pages/Components
- Login (user): Purpose is to authenticate a shopper and obtain a JWT for protected flows (wishlist, cart, checkout, orders, profile).
- Register (user): Create an account.
- Admin Login: Authenticate an administrator to access admin dashboard and management screens.

### Endpoint Table
| Page/Component | Method | Path | Auth | Request Schema | Response Schema | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| User Register | POST | /auth/register | Public | UserRegister | UserOut | Registers a new user by email/password |
| User Login | POST | /auth/login | Public | UserLogin | Token | Returns access_token (bearer) |
| Who Am I (user) | GET | /auth/me | User | — | UserOut | Profile of current authenticated user |
| Admin Login | POST | /admin/auth/login | Public | AdminLogin | Token | Returns access_token (bearer) |
| Admin Me | GET | /admin/me | Admin | — | AdminUserOut | Current admin profile |
| Admin Update Self | PUT | /admin/me | Admin | AdminUserUpdate | AdminUserOut | Update admin username |
| List Admins | GET | /admin/admins | Admin | — | AdminUserOut[] | Admin-only listing of admin accounts |
| Admin Register | POST | /admin/auth/register | Admin (per note) | AdminRegister | AdminUserOut | In production, may require existing admin |

## 2. Product Catalog
### Pages/Components
- Home and Shop (browse): Display product lists with pagination, search, category filter, and sort.
- Search: Dedicated search with query and sort.
- Product Details: View details for a product by id or slug.
- Category menu: Pull category list for filters.

### Endpoint Table
| Page/Component | Method | Path | Auth | Request | Response | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| Categories | GET | /products/categories | Public | — | CategoryOut[] | Load filter chips/menus |
| Product List | GET | /products | Public | q, category, sort, page, size | PaginatedProducts | Sort options latest, price_asc, price_desc |
| Product Details | GET | /products/{id_or_slug} | Public | id_or_slug (path) | ProductOut | Supports numeric ID or slug |

## 3. Wishlist
### Pages/Components
- Wishlist page: List products user saved, with remove and clear.
- Add to wishlist: Buttons on product grid/detail.

### Endpoint Table
| Page/Component | Method | Path | Auth | Request Schema/Params | Response Schema | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| Get Wishlist | GET | /wishlist | User | — | WishlistOut | Current user’s wishlist |
| Add Item | POST | /wishlist | User | WishlistItemIn | WishlistOut | Ignores duplicates |
| Remove Item | DELETE | /wishlist/{product_id} | User | product_id (path) | WishlistOut | Removes a single item |
| Clear All | DELETE | /wishlist | User | — | 204 No Content | Idempotent clear |

## 4. Cart
### Pages/Components
- Cart page: List items, adjust quantities, remove items, clear cart.
- Add to cart: Buttons on product grid/detail.

### Endpoint Table
| Page/Component | Method | Path | Auth | Request Schema/Params | Response Schema | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| Get Cart | GET | /cart | User | — | CartOut | Includes items and total |
| Add/Update Item | POST | /cart | User | CartItemIn | CartOut | Adds or updates quantity |
| Update Quantity | PATCH | /cart/{product_id} | User | product_id (path), CartItemUpdate | CartOut | Quantity 1..99 |
| Remove Item | DELETE | /cart/{product_id} | User | product_id (path) | CartOut | Removes one item |
| Clear Cart | DELETE | /cart | User | — | CartOut | Clears all items |

## 5. Checkout and Orders (User)
### Pages/Components
- Checkout: Collect address and payment method, place order.
- Orders list: Order history for the user.
- Order details: Single order view.

### Endpoint Table
| Page/Component | Method | Path | Auth | Request Schema/Params | Response Schema | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| Checkout | POST | /orders/checkout | User | CheckoutIn | OrderOut | Validates stock, clears cart |
| My Orders | GET | /orders | User | — | OrderOut[] | List of user’s orders |
| Order Details | GET | /orders/{order_id} | User | order_id (path) | OrderOut | Details for one order |

## 6. User Profile and Addresses
### Pages/Components
- Profile page: View and update profile (name) and address.
- Address form in profile or as part of checkout.

### Endpoint Table
| Page/Component | Method | Path | Auth | Request Schema | Response Schema | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| Get My Profile | GET | /users/me | User | — | UserOut | Includes addresses[] |
| Update My Profile | PATCH | /users/me | User | UserUpdate | UserOut | Can upsert one address via AddressIn |

## 7. Contact and Support
### Pages/Components
- Contact form (public): Visitors submit messages to support.
- Admin messages: List and delete messages.

### Endpoint Table
| Page/Component | Method | Path | Auth | Request Schema/Params | Response Schema | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| Submit Message | POST | /contact/messages | Public | MessageCreate | MessageOut (201) | Public, consider rate-limiting in prod |
| List Messages | GET | /contact/admin/messages | Admin | page, size | MessageOut[] | Reverse chronological, paginated |
| Delete Message | DELETE | /contact/admin/messages/{message_id} | Admin | message_id (path) | 204 No Content | Permanent delete |

## 8. Uploads (Image Handling)
### Pages/Components
- Admin image upload for product images (exposed via admin product forms).

### Endpoint Table
| Page/Component | Method | Path | Auth | Request | Response | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| Upload Image | POST | /admin/upload/admin/upload | Admin | multipart/form-data (file) | 201 with URL | Validates MIME (jpeg/png/webp) and size (<=5MB) |

## 9. Admin Area
Admin screens are accessed with an admin token. The SPA should use a separate auth flow and token storage for admins.

### 9.1 Admin Dashboard KPIs
#### Page Purpose
Provide an overview of key metrics, including totals for users/orders/revenue, today’s orders, top products, low stock, and recent orders.

#### Endpoint Table
| Page/Component | Method | Path | Auth | Request | Response | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| Dashboard | GET | /admin/dashboard | Admin | — | DashboardKPIs | Aggregated metrics for dashboards |

### 9.2 Admin: Products CRUD
#### Pages/Components
- Product list table
- Create product form
- Edit product form
- Delete product action
- Upload image (see Uploads section)

#### Endpoint Table
| Page/Component | Method | Path | Auth | Request Schema/Params | Response Schema | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| List Products | GET | /admin/products | Admin | — | ProductOut[] | Admin-scoped list |
| Create Product | POST | /admin/products | Admin | ProductCreate | ProductOut (201) | Enforces unique title |
| Get Product | GET | /admin/products/{product_id} | Admin | product_id (path) | ProductOut |  |
| Update Product | PATCH | /admin/products/{product_id} | Admin | product_id (path), ProductUpdate | ProductOut | Unique title enforced |
| Delete Product | DELETE | /admin/products/{product_id} | Admin | product_id (path) | 204 No Content |  |

### 9.3 Admin: Orders Management
#### Pages/Components
- Orders table (all orders)
- Order detail view
- Update status action
- Delete order action (allowed for pending/cancelled)

#### Endpoint Table
| Page/Component | Method | Path | Auth | Request Schema/Params | Response Schema | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| List Orders | GET | /admin/orders | Admin | — | OrderOut[] |  |
| Get Order | GET | /admin/orders/{order_id} | Admin | order_id (path) | OrderOut |  |
| Update Status | PATCH | /admin/orders/{order_id} | Admin | order_id (path), status_value (query) | OrderOut | Supply new status via query |
| Delete Order | DELETE | /admin/orders/{order_id} | Admin | order_id (path) | 204 No Content | Only pending/cancelled |

### 9.4 Admin: Users/Admins Management
#### Pages/Components
- Users table
- User detail view with activate/deactivate
- Admins list
- Admin profile/self-update

#### Endpoint Table
| Page/Component | Method | Path | Auth | Request Schema/Params | Response Schema | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| List Users | GET | /admin/users | Admin | — | UserOut[] |  |
| Get User | GET | /admin/users/{user_id} | Admin | user_id (path) | UserOut |  |
| Toggle User Active | PATCH | /admin/users/{user_id} | Admin | user_id (path), active (query) | UserOut | Activate/deactivate |
| List Admins | GET | /admin/admins | Admin | — | AdminUserOut[] |  |
| Admin Self (me) | GET | /admin/me | Admin | — | AdminUserOut |  |
| Admin Update Self | PUT | /admin/me | Admin | AdminUserUpdate | AdminUserOut | Update username |
| Admin Register | POST | /admin/auth/register | Admin (per note) | AdminRegister | AdminUserOut | Seeding/controlled environments |

### 9.5 Admin: Contact Messages
#### Pages/Components
- Contact messages table with pagination
- Delete message action

#### Endpoint Table
| Page/Component | Method | Path | Auth | Request Schema/Params | Response Schema | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| List Messages | GET | /contact/admin/messages | Admin | page, size | MessageOut[] | Newest first |
| Delete Message | DELETE | /contact/admin/messages/{message_id} | Admin | message_id (path) | 204 No Content |  |

## Appendix: Schemas Referenced
- Auth: UserRegister, UserLogin, Token, UserOut
- Users: UserOut, UserUpdate, AddressIn, AddressOut
- Products: ProductOut, PaginatedProducts, CategoryOut, ProductCreate, ProductUpdate, ProductImageOut
- Wishlist: WishlistOut, WishlistItemIn, WishlistItemOut
- Cart: CartOut, CartItemIn, CartItemUpdate, CartItemOut
- Orders: OrderOut, OrderItemOut, CheckoutIn, CheckoutAddress
- Contact: MessageCreate, MessageOut
- Admin: AdminLogin, AdminRegister, AdminUserOut, AdminUserUpdate, DashboardKPIs
- Common: HTTPValidationError, PageMeta

## Appendix: Notes and Gaps
- The frontend LLD mentions a separate “search” route; search is supported by GET /products with the q parameter and sort options; no distinct /products/search endpoint exists in the backend.
- Image upload is admin-only and returns a URL under /static/uploads; production deployments should consider external storage.
- Admin register endpoint may be limited in production and is often used during bootstrap or by a super-admin flow.
- The OpenAPI does not expose public product image gallery arrays beyond ProductOut.images (which is present); ensure UI binds to ProductOut.image_url and ProductOut.images if needed.
- Ensure the SPA stores user and admin tokens separately and guards routes accordingly.
