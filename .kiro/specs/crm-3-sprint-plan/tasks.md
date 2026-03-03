# Implementation Plan: CRM 3-Sprint Development Plan

## Overview

This plan implements a full-stack CRM application across three sprints: Foundation & Authentication (Sprint 1), Product Management (Sprint 2), and Inventory Management & Deployment (Sprint 3). Each task builds incrementally on previous work, with property-based and unit tests integrated throughout to validate correctness early.

**Technology Stack:**
- Frontend: React 19.2.0 + TypeScript + TanStack Router + Vite + Tailwind CSS
- Backend: Node.js + Express 5.2+ + TypeScript + PostgreSQL
- Testing: Vitest + React Testing Library + fast-check (property-based testing)
- Deployment: Docker + Docker Compose + nginx

## Tasks

### Sprint 1: Foundation & Authentication

- [x] 1. Initialize project structure and dependencies
  - Create crm-server directory with package.json and TypeScript configuration
  - Create sleam-crm directory with package.json and TypeScript configuration
  - Install backend dependencies: express, pg, bcrypt, jsonwebtoken, typescript, @types packages, vitest, supertest
  - Install frontend dependencies: react, @tanstack/router, tailwind, vite, vitest, @testing-library/react
  - Configure TypeScript with strict mode for both projects (tsconfig.json)
  - Set up Vite config with TanStack Router plugin and Tailwind
  - Install fast-check for property-based testing in both projects
  - _Requirements: 3.1, 3.2, 3.3, 3.6_

- [x] 2. Set up PostgreSQL database schema
  - [x] 2.1 Create init.sql with all table definitions
    - Users table: id (PK), username (unique), password_hash, email (unique), created_at
    - Products table: id (PK), name, description, price (CHECK > 0), sku (unique), created_at, updated_at
    - Inventory table: id (PK), name, location, created_at
    - Inventory_products junction table: inventory_id (FK), product_id (FK), quantity (CHECK > 0), added_at, PRIMARY KEY (inventory_id, product_id)
    - Add foreign key constraints with ON DELETE CASCADE
    - Add indexes: idx_products_sku, idx_inventory_products_inventory, idx_inventory_products_product
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.6_
  
  - [ ]* 2.2 Write unit tests to verify schema structure
    - Test that all tables exist with correct columns
    - Test that constraints are enforced (unique, check, foreign key)
    - Test that indexes exist
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.6_

- [~] 3. Implement authentication utilities
  - [~] 3.1 Create password hashing utilities (utils/password.ts)
    - Implement hashPassword using bcrypt with cost factor 12
    - Implement comparePassword for verification
    - _Requirements: 2.3_
  
  - [ ]* 3.2 Write property test for password hashing
    - **Property 1: Password hashing consistency**
    - *For any* user registration or password change, the stored password should be a valid bcrypt hash, never plaintext
    - Run 100+ iterations with randomly generated passwords
    - **Validates: Requirements 2.3**
    - **Feature: crm-3-sprint-plan, Property 1**
  
  - [~] 3.3 Create JWT utilities (utils/jwt.ts)
    - Implement generateToken with 24 hour expiration
    - Implement verifyToken with error handling
    - Use JWT_SECRET from environment variables
    - _Requirements: 2.1, 2.4_
  
  - [ ]* 3.4 Write property test for JWT round-trip
    - **Property 2: JWT authentication round-trip**
    - *For any* valid user credentials, logging in should generate a JWT that successfully authenticates protected routes
    - Test token generation, verification, and expiration
    - **Validates: Requirements 2.1, 2.4**
    - **Feature: crm-3-sprint-plan, Property 2**

- [~] 4. Build authentication API
  - [~] 4.1 Create User model (models/User.ts)
    - Implement createUser query with parameterized SQL
    - Implement findUserByUsername query
    - Implement findUserByEmail query
    - Handle unique constraint violations
    - _Requirements: 2.5_
  
  - [~] 4.2 Create auth controller (controllers/authController.ts)
    - Implement register endpoint: validate input, check uniqueness, hash password, create user, generate JWT
    - Implement login endpoint: find user, compare password, generate JWT
    - Return consistent error messages without revealing whether username or password was incorrect
    - _Requirements: 2.1, 2.2, 2.5_
  
  - [~] 4.3 Create auth routes (routes/auth.ts)
    - POST /api/auth/register - accepts username, email, password
    - POST /api/auth/login - accepts username, password
    - Return user object (without password_hash) and JWT token
    - _Requirements: 2.1, 2.5_
  
  - [ ]* 4.4 Write property test for invalid credentials
    - **Property 3: Invalid credentials rejection**
    - *For any* invalid username/password combination, return error without revealing which field was incorrect
    - Test with non-existent users and wrong passwords
    - **Validates: Requirements 2.2**
    - **Feature: crm-3-sprint-plan, Property 3**
  
  - [ ]* 4.5 Write property test for username/email uniqueness
    - **Property 4: Username and email uniqueness**
    - *For any* existing user, attempting to register with same username or email should be rejected
    - Test duplicate username, duplicate email, and both
    - **Validates: Requirements 2.5**
    - **Feature: crm-3-sprint-plan, Property 4**

- [~] 5. Create authentication middleware
  - [~] 5.1 Implement JWT validation middleware (middleware/auth.ts)
    - Extract token from Authorization header (Bearer format)
    - Verify token using JWT utilities
    - Attach decoded user to request object (extend Express Request type)
    - Return 401 for missing tokens
    - Return 401 for invalid or expired tokens
    - _Requirements: 2.4, 2.7_
  
  - [ ]* 5.2 Write property test for protected route authentication
    - **Property 5: Protected route authentication**
    - *For any* protected route, requests without valid JWT should be rejected with 401
    - Test missing token, invalid token, expired token, and valid token
    - **Validates: Requirements 2.4, 2.7**
    - **Feature: crm-3-sprint-plan, Property 5**

- [~] 6. Implement error handling
  - [~] 6.1 Create custom error classes (middleware/errorHandler.ts)
    - ValidationError class with details field for field-level errors
    - NotFoundError class with resource name
    - Export consistent ErrorResponse interface
    - _Requirements: 16.1, 16.3, 16.4_
  
  - [~] 6.2 Create global error handler middleware
    - Handle ValidationError → 400 with message and details
    - Handle NotFoundError → 404 with descriptive message
    - Handle duplicate key errors → 409 with generic message
    - Handle authentication errors → 401
    - Handle authorization errors → 403
    - Handle generic errors → 500 without exposing internals
    - Log all errors with timestamp, error type, stack trace, and request context
    - _Requirements: 16.1, 16.2, 16.3, 16.4, 16.5, 16.6, 16.7_
  
  - [ ]* 6.3 Write property test for consistent error format
    - **Property 29: Consistent error response format**
    - *For any* error condition, backend should return JSON with consistent structure (status, message, optional details)
    - Test validation, auth, not found, and server errors
    - **Validates: Requirements 16.1, 16.2, 16.3, 16.4, 16.5, 16.6**
    - **Feature: crm-3-sprint-plan, Property 29**
  
  - [ ]* 6.4 Write property test for error logging
    - **Property 30: Error logging completeness**
    - *For any* backend error, verify it's logged with sufficient detail for debugging
    - Check for timestamp, error type, stack trace, and request context
    - **Validates: Requirements 16.7**
    - **Feature: crm-3-sprint-plan, Property 30**

- [~] 7. Set up Express application
  - [~] 7.1 Create main Express app (index.ts)
    - Initialize Express with TypeScript
    - Add JSON body parser middleware
    - Add CORS middleware with appropriate configuration
    - Mount auth routes at /api/auth
    - Add health check endpoint at /health (checks database connection)
    - Add global error handler middleware (must be last)
    - Start server on port from environment variable (default 3500)
    - Validate required environment variables on startup
    - _Requirements: 3.2, 3.4, 14.4_
  
  - [~] 7.2 Create database connection pool (config/database.ts)
    - Initialize pg.Pool with connection string from environment
    - Export query function that uses the pool
    - Handle connection errors gracefully
    - _Requirements: 3.4, 14.5_

- [~] 8. Build frontend login page
  - [~] 8.1 Create API client (lib/api.ts)
    - Implement ApiClient class with baseURL from environment (VITE_API_URL)
    - Implement request method with automatic auth header injection from localStorage
    - Implement get, post, put, delete methods
    - Handle 401 responses by clearing token and redirecting to /login
    - Handle network errors with appropriate error messages
    - _Requirements: 2.6, 8.6, 8.7_
  
  - [ ]* 8.2 Write property test for token storage and inclusion
    - **Property 6: Token storage and inclusion**
    - *For any* successful login, frontend should store JWT and include it in Authorization header of subsequent requests
    - Test token persistence and header injection
    - **Validates: Requirements 2.6, 8.6**
    - **Feature: crm-3-sprint-plan, Property 6**
  
  - [ ]* 8.3 Write property test for auth failure redirect
    - **Property 31: Authentication failure redirection**
    - *For any* 401 response from backend, frontend should redirect to login page
    - Test with expired tokens and invalid tokens
    - **Validates: Requirements 8.7**
    - **Feature: crm-3-sprint-plan, Property 31**
  
  - [~] 8.4 Create login route (routes/login.tsx)
    - Create form with username and password fields
    - Implement form validation (required fields)
    - Submit to POST /api/auth/login on form submit
    - Store JWT token in localStorage on success
    - Redirect to /products page after successful login
    - Display error messages from API
    - Show loading state during submission
    - _Requirements: 2.1, 2.6_
  
  - [~] 8.5 Create ProtectedRoute component (components/ProtectedRoute.tsx)
    - Check for JWT token in localStorage
    - Redirect to /login if token is missing
    - Render children if token is present
    - Use with TanStack Router's beforeLoad hook
    - _Requirements: 2.7_
  
  - [ ]* 8.6 Write unit tests for login page
    - Test form submission with valid credentials
    - Test error display for invalid credentials
    - Test redirect on successful login
    - Test loading state during submission
    - _Requirements: 2.1, 2.6_

- [~] 9. Create Docker configuration
  - [~] 9.1 Create backend Dockerfile
    - Use node:20-alpine as base image
    - Copy package files and run npm ci --only=production
    - Copy source code and build TypeScript
    - Expose port 3500
    - Add health check using Node.js http module to check /health endpoint
    - Set CMD to run compiled JavaScript
    - _Requirements: 4.2, 12.2_
  
  - [~] 9.2 Create frontend Dockerfile with multi-stage build
    - Build stage: node:20-alpine, install deps, run vite build
    - Production stage: nginx:alpine, copy built files to /usr/share/nginx/html
    - Copy nginx configuration
    - Expose port 80
    - _Requirements: 4.1, 12.1_
  
  - [~] 9.3 Create nginx configuration (nginx.conf)
    - Serve static files from /usr/share/nginx/html
    - Configure SPA routing (try_files with fallback to index.html)
    - Proxy /api requests to backend:3500
    - Set appropriate headers for proxied requests
    - _Requirements: 12.5_
  
  - [~] 9.4 Create docker-compose.yml
    - Define db service: postgres:15-alpine with health check (pg_isready)
    - Define backend service: build from crm-server, depends on db health check
    - Define frontend service: build from sleam-crm, expose port 80
    - Configure named volume for postgres data persistence (postgres_data)
    - Configure bridge network (crm_network)
    - Load environment variables from .env file
    - Add restart policies (unless-stopped)
    - Configure resource limits for production
    - Add logging configuration
    - _Requirements: 4.3, 4.4, 4.5, 4.6, 12.3, 12.7, 12.8, 13.1, 13.2, 13.3, 13.4_
  
  - [~] 9.5 Create .env.example file
    - Document DB_NAME, DB_USER, DB_PASSWORD
    - Document JWT_SECRET with note about 256-bit random string
    - Document PORT (default 3500)
    - Document VITE_API_URL for frontend
    - Add comments explaining each variable
    - _Requirements: 3.4, 14.1, 14.2_
  
  - [ ]* 9.6 Write property test for environment validation
    - **Property 28: Environment variable validation**
    - *For any* required environment variable, if not set, system should fail with clear error message
    - Test missing DATABASE_URL, JWT_SECRET, and other required vars
    - **Validates: Requirements 14.4**
    - **Feature: crm-3-sprint-plan, Property 28**

- [~] 10. Checkpoint - Test authentication flow end-to-end
  - Start Docker Compose with `docker-compose up -d`
  - Verify all containers are healthy
  - Register a new user via API or UI
  - Login with credentials and verify JWT token is returned
  - Verify JWT token is stored in localStorage
  - Access a protected route and verify authentication works
  - Try accessing protected route without token and verify 401 response
  - Ensure all tests pass, ask the user if questions arise.

### Sprint 2: Product Management

- [~] 11. Implement product data model
  - [~] 11.1 Create Product model (models/Product.ts)
    - Implement createProduct query with parameterized SQL, auto-set created_at and updated_at
    - Implement getAllProducts query returning all products ordered by created_at DESC
    - Implement getProductById query with error handling for not found
    - Implement updateProduct query with parameterized SQL, auto-update updated_at
    - Implement deleteProduct query
    - Handle unique constraint violations for SKU
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 6.4, 6.5_
  
  - [~] 11.2 Create product validation utilities (utils/validation.ts)
    - Validate product name: non-empty string, 1-255 characters
    - Validate price: positive number, max 2 decimal places, range 0.01-999999.99
    - Validate SKU: non-empty, 1-100 characters, alphanumeric + hyphens only
    - Validate description: optional, max 5000 characters
    - Return detailed field-level error messages
    - _Requirements: 6.1, 6.2, 6.3, 6.6, 6.7_

- [~] 12. Build product API endpoints
  - [~] 12.1 Create product controller (controllers/productController.ts)
    - Implement create product: validate input, create product, return 201 with created product
    - Implement list products: return all products with 200
    - Implement get product by ID: return product with 200, or 404 if not found
    - Implement update product: validate input, update product, return 200 with updated product, or 404 if not found
    - Implement delete product: delete product, return 204, or 404 if not found
    - Handle validation errors with 400 and field-level details
    - Handle duplicate SKU with 409
    - Automatic timestamps handled by database (created_at on insert, updated_at on update)
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 5.6, 5.7, 6.4, 6.5, 6.7_
  
  - [~] 12.2 Create product routes (routes/products.ts)
    - GET /api/products (protected with auth middleware)
    - POST /api/products (protected with auth middleware)
    - GET /api/products/:id (protected with auth middleware)
    - PUT /api/products/:id (protected with auth middleware)
    - DELETE /api/products/:id (protected with auth middleware)
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 5.8_
  
  - [~] 12.3 Mount product routes in main app (index.ts)
    - Add product routes at /api/products
    - Ensure routes are mounted before error handler
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 5.8_
  
  - [ ]* 12.4 Write property test for product creation and retrieval
    - **Property 8: Product creation and retrieval**
    - *For any* valid product data, creating via POST should result in retrievable product via GET with matching fields
    - Use fast-check to generate random valid products (100+ iterations)
    - **Validates: Requirements 5.1, 5.3**
    - **Feature: crm-3-sprint-plan, Property 8**
  
  - [ ]* 12.5 Write property test for product list completeness
    - **Property 9: Product list completeness**
    - *For any* set of created products, GET /api/products should return all non-deleted products
    - Create multiple products, verify all appear in list
    - **Validates: Requirements 5.2**
    - **Feature: crm-3-sprint-plan, Property 9**
  
  - [ ]* 12.6 Write property test for product update persistence
    - **Property 10: Product update persistence**
    - *For any* existing product and valid update data, updating via PUT should persist new values
    - Create product, update with random data, verify changes persisted
    - **Validates: Requirements 5.4**
    - **Feature: crm-3-sprint-plan, Property 10**
  
  - [ ]* 12.7 Write property test for product deletion
    - **Property 11: Product deletion removal**
    - *For any* existing product, deleting via DELETE should remove it from list and return 404 on GET
    - Create product, delete it, verify it's gone
    - **Validates: Requirements 5.5**
    - **Feature: crm-3-sprint-plan, Property 11**
  
  - [ ]* 12.8 Write property test for product validation
    - **Property 12: Product validation enforcement**
    - *For any* invalid product data (empty name, non-positive price, duplicate SKU, invalid SKU format), backend should return 400 with field errors
    - Test all validation rules with randomly generated invalid data
    - **Validates: Requirements 5.6, 6.1, 6.2, 6.3, 6.6, 6.7**
    - **Feature: crm-3-sprint-plan, Property 12**
  
  - [ ]* 12.9 Write property test for automatic timestamps
    - **Property 13: Automatic timestamp management**
    - *For any* product creation, created_at should be set; for any update, updated_at should be updated
    - Verify timestamps are present and updated_at changes on update
    - **Validates: Requirements 6.4, 6.5**
    - **Feature: crm-3-sprint-plan, Property 13**
  
  - [ ]* 12.10 Write property test for product endpoint authentication
    - **Property 14: Product endpoint authentication requirement**
    - *For any* product endpoint, requests without valid auth should return 401
    - Test all endpoints (GET, POST, PUT, DELETE) without token
    - **Validates: Requirements 5.8**
    - **Feature: crm-3-sprint-plan, Property 14**

- [~] 13. Create product UI components
  - [~] 13.1 Create product list page (routes/products/index.tsx)
    - Fetch products from GET /api/products on component mount
    - Display products in responsive table/grid with columns: name, SKU, price, actions
    - Add action buttons for each product: View, Edit, Delete
    - Add "Create New Product" button linking to /products/new
    - Handle loading state with spinner
    - Handle error state with error message display
    - Handle empty state with helpful message
    - _Requirements: 7.1, 8.1_
  
  - [~] 13.2 Create product form component (components/ProductForm.tsx)
    - Form fields: name (text), description (textarea), price (number), SKU (text)
    - Implement client-side validation matching backend rules
    - Display server validation errors inline with fields
    - Disable submit button during API request (loading state)
    - Accept onSubmit callback and initialValues prop for reuse
    - Show success feedback after submission
    - _Requirements: 7.2, 7.3, 7.7, 7.8_
  
  - [~] 13.3 Create new product page (routes/products/new.tsx)
    - Use ProductForm component with empty initial values
    - Submit to POST /api/products via API client
    - Redirect to /products on success
    - Display error toast on failure
    - _Requirements: 7.2, 7.3, 8.2_
  
  - [~] 13.4 Create product detail page (routes/products/$id.tsx)
    - Fetch product from GET /api/products/:id using route params
    - Display all product information in readable format
    - Add Edit button linking to /products/$id/edit
    - Add Delete button with confirmation
    - Handle loading and error states
    - _Requirements: 7.6_
  
  - [~] 13.5 Create edit product page (routes/products/$id.edit.tsx)
    - Fetch product to get current values
    - Use ProductForm component with product as initialValues
    - Submit to PUT /api/products/:id via API client
    - Redirect to /products/$id on success
    - Display error toast on failure
    - _Requirements: 7.4, 8.3_
  
  - [~] 13.6 Implement delete confirmation dialog
    - Show confirmation dialog when delete button clicked
    - Display product name in confirmation message
    - Submit to DELETE /api/products/:id on confirm
    - Redirect to /products list on success
    - Show error toast on failure
    - _Requirements: 7.5, 8.4_
  
  - [ ]* 13.7 Write property test for frontend CRUD integration
    - **Property 15: Frontend product CRUD integration**
    - *For any* product operation via UI (create, update, delete), frontend should send correct API request and update UI
    - Mock API responses and verify UI updates correctly
    - **Validates: Requirements 8.1, 8.2, 8.3, 8.4**
    - **Feature: crm-3-sprint-plan, Property 15**
  
  - [ ]* 13.8 Write property test for error display
    - **Property 16: Frontend error display**
    - *For any* API error response, frontend should display error in user-friendly format
    - Test validation errors, auth errors, not found errors
    - **Validates: Requirements 7.7, 8.5**
    - **Feature: crm-3-sprint-plan, Property 16**
  
  - [ ]* 13.9 Write property test for form submission state
    - **Property 17: Form submission state management**
    - *For any* form submission, submit button should be disabled during request and re-enabled after
    - Test with successful and failed submissions
    - **Validates: Requirements 7.8**
    - **Feature: crm-3-sprint-plan, Property 17**
  
  - [ ]* 13.10 Write property test for edit form pre-population
    - **Property 18: Edit form pre-population**
    - *For any* product being edited, form should be pre-filled with current values
    - Verify all fields (name, description, price, SKU) are populated
    - **Validates: Requirements 7.4, 7.6**
    - **Feature: crm-3-sprint-plan, Property 18**
  
  - [ ]* 13.11 Write unit tests for product components
    - Test ProductForm validation logic
    - Test product list rendering with mock data
    - Test delete confirmation dialog flow
    - Test error handling in components
    - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, 7.7, 7.8_

- [~] 14. Checkpoint - Test product management end-to-end
  - Create a new product via UI with valid data
  - View product details page
  - Edit product and verify changes persist
  - Delete product with confirmation
  - Verify all CRUD operations work correctly
  - Test validation by submitting invalid data
  - Verify error messages display properly
  - Ensure all tests pass, ask the user if questions arise.

### Sprint 3: Inventory Management & Deployment

- [~] 15. Implement inventory data model
  - [~] 15.1 Create Inventory model (models/Inventory.ts)
    - Implement createInventory query with parameterized SQL, auto-set created_at
    - Implement getAllInventories query returning all inventories
    - Implement getInventoryById query with error handling for not found
    - Implement getInventoryWithProducts query using JOIN to include products with quantities
    - Implement addProductToInventory query with INSERT ON CONFLICT UPDATE for idempotence
    - Handle foreign key constraint violations
    - Auto-set added_at timestamp when adding products
    - _Requirements: 9.1, 9.2, 10.1, 10.5, 10.6, 10.7_
  
  - [~] 15.2 Add inventory validation to validation utilities
    - Validate inventory name: non-empty string, 1-255 characters
    - Validate location: optional, max 255 characters
    - Validate quantity: positive integer (min 1)
    - Validate product ID and inventory ID existence
    - Return detailed field-level error messages
    - _Requirements: 9.3, 10.2, 10.3, 10.4_
  
  - [ ]* 15.3 Write property test for referential integrity
    - **Property 7: Referential integrity enforcement**
    - *For any* attempt to insert inventory-product association with non-existent IDs, database should reject
    - Test with invalid inventory_id and invalid product_id
    - **Validates: Requirements 1.5**
    - **Feature: crm-3-sprint-plan, Property 7**

- [~] 16. Build inventory API endpoints
  - [~] 16.1 Create inventory controller (controllers/inventoryController.ts)
    - Implement create inventory: validate input, create inventory, return 201 with created inventory
    - Implement list inventories: return all inventories with 200
    - Implement get inventory with products: return inventory with product list and quantities, or 404 if not found
    - Implement add product to inventory: validate inventory exists, validate product exists, validate quantity, upsert association, return 200 with updated inventory
    - Handle validation errors with 400 and field-level details
    - Handle not found errors with 404
    - Automatic timestamps handled by database (created_at, added_at)
    - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.6, 10.1, 10.2, 10.3, 10.4, 10.5, 10.6, 10.7_
  
  - [~] 16.2 Create inventory routes (routes/inventory.ts)
    - GET /api/inventory (protected with auth middleware)
    - POST /api/inventory (protected with auth middleware)
    - GET /api/inventory/:id/products (protected with auth middleware)
    - POST /api/inventory/:id/products (protected with auth middleware, body: { productId, quantity })
    - _Requirements: 9.1, 9.2, 9.5, 10.1_
  
  - [~] 16.3 Mount inventory routes in main app (index.ts)
    - Add inventory routes at /api/inventory
    - Ensure routes are mounted before error handler
    - _Requirements: 9.1, 9.2, 9.5, 10.1_
  
  - [ ]* 16.4 Write property test for inventory creation and retrieval
    - **Property 19: Inventory creation and retrieval**
    - *For any* valid inventory data, creating via POST should result in retrievable inventory via GET with matching fields
    - Use fast-check to generate random valid inventories (100+ iterations)
    - **Validates: Requirements 9.1, 9.2**
    - **Feature: crm-3-sprint-plan, Property 19**
  
  - [ ]* 16.5 Write property test for inventory validation
    - **Property 20: Inventory validation enforcement**
    - *For any* invalid inventory data (empty name), backend should return 400 with validation errors
    - Test with empty and whitespace-only names
    - **Validates: Requirements 9.3, 9.6**
    - **Feature: crm-3-sprint-plan, Property 20**
  
  - [ ]* 16.6 Write property test for inventory timestamps
    - **Property 21: Inventory automatic timestamps**
    - *For any* inventory creation, created_at should be automatically set to current timestamp
    - Verify timestamp is present and recent
    - **Validates: Requirements 9.4**
    - **Feature: crm-3-sprint-plan, Property 21**
  
  - [ ]* 16.7 Write property test for inventory authentication
    - **Property 22: Inventory endpoint authentication**
    - *For any* inventory endpoint, requests without valid auth should return 401
    - Test all endpoints without token
    - **Validates: Requirements 9.5**
    - **Feature: crm-3-sprint-plan, Property 22**
  
  - [ ]* 16.8 Write property test for product-to-inventory association
    - **Property 23: Product-to-inventory association**
    - *For any* valid inventory ID, product ID, and quantity, adding product should create/update association
    - Verify product appears in inventory with correct quantity
    - **Validates: Requirements 10.1, 10.7**
    - **Feature: crm-3-sprint-plan, Property 23**
  
  - [ ]* 16.9 Write property test for association validation
    - **Property 24: Inventory-product validation**
    - *For any* attempt with non-existent inventory/product ID or non-positive quantity, backend should return 400
    - Test all invalid cases
    - **Validates: Requirements 10.2, 10.3, 10.4**
    - **Feature: crm-3-sprint-plan, Property 24**
  
  - [ ]* 16.10 Write property test for idempotence
    - **Property 25: Inventory-product idempotence**
    - *For any* product already in inventory, adding again should update quantity not create duplicate
    - Add product twice, verify single entry with updated quantity
    - **Validates: Requirements 10.5**
    - **Feature: crm-3-sprint-plan, Property 25**
  
  - [ ]* 16.11 Write property test for association timestamps
    - **Property 26: Inventory-product automatic timestamps**
    - *For any* product added to inventory, added_at should be automatically set
    - Verify timestamp is present and recent
    - **Validates: Requirements 10.6**
    - **Feature: crm-3-sprint-plan, Property 26**

- [~] 17. Create inventory UI components
  - [~] 17.1 Create inventory list page (routes/inventory/index.tsx)
    - Fetch inventories from GET /api/inventory on component mount
    - Display inventories in responsive table/grid with columns: name, location, actions
    - Add "Create New Inventory" button linking to /inventory/new
    - Add View button for each inventory linking to /inventory/$id
    - Handle loading, error, and empty states
    - _Requirements: 11.1_
  
  - [~] 17.2 Create inventory form component (components/InventoryForm.tsx)
    - Form fields: name (text, required), location (text, optional)
    - Implement client-side validation matching backend rules
    - Display server validation errors inline with fields
    - Disable submit button during API request
    - Accept onSubmit callback and initialValues prop for reuse
    - _Requirements: 11.2, 11.7_
  
  - [~] 17.3 Create new inventory page (routes/inventory/new.tsx)
    - Use InventoryForm component with empty initial values
    - Submit to POST /api/inventory via API client
    - Redirect to /inventory on success
    - Display error toast on failure
    - _Requirements: 11.2, 11.3_
  
  - [~] 17.4 Create inventory detail page (routes/inventory/$id.tsx)
    - Fetch inventory with products from GET /api/inventory/:id/products
    - Display inventory information (name, location)
    - Display product list table with columns: product name, SKU, quantity
    - Add "Add Product" button to show add product form
    - Handle loading, error, and empty product list states
    - _Requirements: 11.4_
  
  - [~] 17.5 Add product-to-inventory form on detail page
    - Product selection dropdown populated from GET /api/products
    - Quantity input field (number, min 1)
    - Submit to POST /api/inventory/:id/products with { productId, quantity }
    - Refresh inventory view on success to show updated product list
    - Display validation errors
    - Show success feedback after adding product
    - _Requirements: 11.5, 11.6_
  
  - [ ]* 17.6 Write property test for frontend inventory integration
    - **Property 27: Frontend inventory integration**
    - *For any* inventory operation via UI (create inventory, add product), frontend should send correct API request and update UI
    - Mock API responses and verify UI updates
    - **Validates: Requirements 11.3, 11.4, 11.6**
    - **Feature: crm-3-sprint-plan, Property 27**
  
  - [ ]* 17.7 Write unit tests for inventory components
    - Test InventoryForm validation logic
    - Test inventory list rendering with mock data
    - Test add product to inventory form
    - Test error handling in components
    - _Requirements: 11.1, 11.2, 11.3, 11.4, 11.5, 11.6, 11.7_

- [~] 18. Optimize Docker for production
  - [~] 18.1 Update frontend Dockerfile for production optimization
    - Verify multi-stage build is properly configured
    - Minimize image size by using alpine base images
    - Remove development dependencies from final image
    - Optimize layer caching by copying package files first
    - _Requirements: 12.1_
  
  - [~] 18.2 Update backend Dockerfile for production optimization
    - Add health check if not already present
    - Use --only=production flag for npm ci
    - Remove source TypeScript files from final image (only keep compiled JS)
    - Use alpine base image for smaller size
    - _Requirements: 12.2_
  
  - [~] 18.3 Update docker-compose.yml for production
    - Add restart policies: unless-stopped for all services
    - Configure resource limits (memory, CPU) for each service
    - Add logging configuration with log rotation
    - Configure proper depends_on with health check conditions
    - _Requirements: 12.3_
  
  - [~] 18.4 Configure Docker secrets for sensitive data
    - Use Docker secrets or .env files for sensitive configuration
    - Update docker-compose to use secrets for DB_PASSWORD and JWT_SECRET
    - Document secret management in README
    - Never commit actual secrets to version control
    - _Requirements: 12.4_
  
  - [~] 18.5 Verify database initialization and persistence
    - Ensure init.sql runs on first database container startup
    - Test database persistence by stopping and restarting containers
    - Verify data survives container restarts
    - Test with fresh database (remove volume and recreate)
    - _Requirements: 12.6, 13.5_
  
  - [~] 18.6 Configure backend to wait for database readiness
    - Verify depends_on with health check condition in docker-compose
    - Add retry logic in backend database connection
    - Test that backend starts only after database is ready
    - Handle database connection failures gracefully
    - _Requirements: 13.6_

- [~] 19. Write comprehensive test suite
  - [ ]* 19.1 Write backend unit tests
    - Test all authentication endpoints (register, login) with valid and invalid inputs
    - Test all product endpoints (GET, POST, PUT, DELETE) with valid and invalid inputs
    - Test all inventory endpoints with valid and invalid inputs
    - Test authentication middleware with valid, invalid, and missing tokens
    - Test error handling middleware with different error types
    - Test password hashing and JWT utilities
    - Use supertest for API endpoint testing
    - _Requirements: 15.1, 15.2_
  
  - [ ]* 19.2 Write frontend component tests
    - Test all product components (list, form, detail, edit)
    - Test all inventory components (list, form, detail, add product)
    - Test authentication components (login, protected route)
    - Test protected route behavior (redirect when not authenticated)
    - Test form validation and error display
    - Use React Testing Library and Vitest
    - _Requirements: 15.3, 15.4_
  
  - [ ]* 19.3 Write integration tests
    - Test complete authentication flow: register → login → access protected route
    - Test complete product CRUD flow: create → read → update → delete
    - Test complete inventory flow: create inventory → add product → view inventory with products
    - Mock API responses for frontend integration tests
    - Use test database for backend integration tests
    - _Requirements: 15.5, 15.6_
  
  - [ ]* 19.4 Verify code coverage
    - Run coverage reports for backend using Vitest coverage
    - Ensure minimum 70% coverage for API routes
    - Identify untested code paths
    - Add tests for critical uncovered areas
    - _Requirements: 15.7_
  
  - [~] 19.5 Add test scripts to package.json
    - Add "test" script: vitest run
    - Add "test:watch" script: vitest watch
    - Add "test:coverage" script: vitest run --coverage
    - Add scripts to both frontend and backend package.json
    - Document test commands in README
    - _Requirements: 15.8_

- [~] 20. Create API documentation
  - [~] 20.1 Document all API endpoints
    - List all endpoints with HTTP methods and paths
    - Document request formats with example JSON payloads
    - Document response formats with example JSON responses
    - Include status codes for success and error cases
    - Document query parameters and path parameters
    - Include example requests and responses for each endpoint
    - Organize by resource (Auth, Products, Inventory)
    - _Requirements: 17.1, 17.2_
  
  - [~] 20.2 Document authentication requirements
    - Explain JWT token-based authentication
    - Document how to obtain a token (register/login)
    - Document how to include token in requests (Authorization: Bearer <token>)
    - Explain token expiration (24 hours)
    - List which endpoints require authentication
    - _Requirements: 17.3_
  
  - [~] 20.3 Document error responses
    - List all possible HTTP status codes (200, 201, 204, 400, 401, 403, 404, 409, 500)
    - Explain error response format: { message: string, details?: object }
    - Provide examples of each error type
    - Document validation error format with field-level details
    - Explain when each status code is returned
    - _Requirements: 17.4_
  
  - [~] 20.4 Create comprehensive API documentation file
    - Consolidate all API documentation into API.md or README.md
    - Add table of contents for easy navigation
    - Include setup instructions for running the application
    - Add deployment instructions with Docker Compose
    - Document environment variables and configuration
    - Add troubleshooting section
    - _Requirements: 17.5_

- [~] 21. Final checkpoint - Complete system test
  - Start complete Docker Compose stack with `docker-compose up -d`
  - Verify all containers are healthy (db, backend, frontend)
  - Test complete authentication flow: register new user → login → receive token
  - Test complete product management flow: create → list → view → edit → delete
  - Test complete inventory management flow: create inventory → add products → view inventory with products
  - Verify data persists across container restarts (stop and restart containers)
  - Test error handling: invalid inputs, authentication failures, not found errors
  - Verify all API endpoints return correct status codes and error messages
  - Test frontend UI: all pages load, forms work, navigation works
  - Run all automated tests (unit, property, integration) and verify they pass
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional testing tasks that can be skipped for faster MVP delivery
- Each task references specific requirements for full traceability back to requirements.md
- Property tests validate universal correctness properties using fast-check (minimum 100 iterations each)
- Unit tests validate specific examples, edge cases, and error conditions
- Both property tests and unit tests are necessary and complementary for comprehensive coverage
- Checkpoints ensure incremental validation at key milestones (end of each sprint)
- All code should be production-ready with proper error handling, validation, and security
- Property test format: **Feature: crm-3-sprint-plan, Property N: [property description]**
- Each property test must reference its corresponding property from the design document
- Testing approach: property tests for universal properties, unit tests for specific cases
- Fast-check configuration: minimum 100 iterations per property test due to randomization
- Environment variables must be validated on startup to fail fast with clear error messages
- Docker health checks ensure services start in correct order and are ready before dependent services start
- All sensitive data (passwords, JWT secrets) must use environment variables, never hardcoded
- Database uses parameterized queries to prevent SQL injection
- All API endpoints (except auth) require JWT authentication
- Frontend stores JWT in localStorage and includes it in Authorization header
- Error responses follow consistent format: { message: string, details?: object }
- Timestamps (created_at, updated_at, added_at) are automatically managed by database

