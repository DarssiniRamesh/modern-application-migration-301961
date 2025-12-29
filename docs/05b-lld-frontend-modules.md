# Module-wise LLD: React Frontend SPA

## Architecture Overview
- React SPA (Create React App base) consuming FastAPI REST APIs.
- Routing via react-router-dom.
- State via React Context + Reducer or React Query; minimal global state for auth and cart badges.
- Theming aligns with existing KAVIA template; reuse Font Awesome via @fortawesome/react-fontawesome; Swiper-like via swiper/react.

## Shared Layout and Components
- App shell with Header, Footer, and main content.
- Header shows navigation: home, about, orders, shop, contact; search input; wishlist and cart icons with counts; profile menu mirroring PHP header behavior.
- Footer contains quick links and contact information.

## Routes and Pages
- Public: 
  - / (Home with latest products slider)
  - /about
  - /shop
  - /search
  - /product/:id (Quick View)
  - /contact
  - /login
  - /register
- Auth-required:
  - /cart
  - /wishlist
  - /checkout
  - /orders
- Admin:
  - /admin/login
  - /admin/dashboard
  - /admin/products
  - /admin/orders
  - /admin/users
  - /admin/admins
  - /admin/messages

## Data Fetching and DTOs
- Auth:
  - POST /auth/login -> {access_token}
  - POST /auth/register
  - GET /auth/me -> {id, name, email, roles}
- Products:
  - GET /products -> [{id, name, price, image_01}]
  - GET /products/{id} -> {id, name, details, price, image_01..03}
  - GET /products/search?q=
- Wishlist:
  - GET /wishlist -> [{id, pid, name, price, image}]
  - POST /wishlist {pid}
  - DELETE /wishlist/{id}
- Cart:
  - GET /cart -> {items:[{id, pid, name, price, quantity, image}], grand_total}
  - POST /cart {pid, qty}
  - PATCH /cart/{id} {qty}
  - DELETE /cart/{id} and DELETE /cart
- Orders:
  - POST /orders {name, number, email, method, address}
  - GET /orders -> [{... order summary incl. placed_on and payment_status}]
- Admin Products:
  - POST /products (multipart for images)
  - PUT /products/{id}, DELETE /products/{id}

## Error Handling and UX
- Display server validation messages.
- Redirect unauthenticated users to login on protected routes.
- Keep cart/wishlist counts in header synced after mutations.

## Component Sketches
- Header: uses context to show counts from /wishlist and /cart.
- Home: fetch /products?limit=6 and render Swiper slider.
- Shop: fetch /products and render grid with add-to-cart/wishlist.
- Search: debounce query to /products/search.
- Quick View: fetch /products/{id}, support add to cart/wishlist.
- Cart: list items, editable quantity, delete/clear, proceed to checkout disabled if total <= 0.
- Checkout: form fields paralleling PHP version, submit to /orders.
- Orders: fetch history and render payment status badges.

Sources:
- E-commerce-PHP-Application-301945 header/footer and page flows
- modern-application-migration-301961/frontend_app/src/App.js
- Backend API outline in 04-migration-architecture-hld.md
