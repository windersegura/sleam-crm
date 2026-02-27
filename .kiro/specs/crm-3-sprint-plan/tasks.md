# Implementation Plan: CRM 3-Sprint Development Plan

## Overview

This plan breaks down the CRM application into discrete implementation tasks across 3 sprints. Each task builds on previous work, with testing integrated throughout to catch issues early.

## Tasks

### Sprint 1: Foundation & Authentication

- [x] 1. Initialize project structure and dependencies
  - Create crm-server directory with package.json
  - Create sleam-crm directory with package.json
  - Install backend dependencies: express, pg, bcrypt, jsonwebtoken, typescript, @types packages
  - Install frontend dependencies: react, @tanstack/router, tailwind, vite
  - Configure TypeScript for both projects (tsconfig.json)
  - Set up Vite config with TanStack Router plugin
  - _Requirements: 3.1, 3.2, 3.3, 3.6_

- [~] 2. Set up PostgreSQL database schema
  - [~] 2.1 Create init.sql with all table definitions
    - Users table with constraints
    - Products table with constraints
    - Inventory table
    - Inventory_products junction table with foreign keys
    - Add indexes for performance
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.6_
  
  - [ ]* 2.2 Write unit tests to verify schema structure
    - Test that all tables exist
    - Test that constraints are enforced
    - _Requirements: 1.1-1.6_

- [~] 3. Implement authentication utilities
  - [~] 3.1 Create password hashing utilities (utils/password.ts)
    - Implement hashPassword using bcrypt (cost factor 12)
    - Implement comparePassword
    - _Requirements: 2.3_
  
  - [ ]* 3.2 Write property test for password hashing
    - **Property 1: Password hashing consistency**
    - **Validates: Requirements 2.3**
  
  - [~] 3.3 Create JWT utilities (utils/jwt.ts)
    - Implement generateToken (24 hour expiration)
    - Implement verifyToken
    - _Requirements: 2.1, 2.4_
  
  - [ ]* 3.4 Write property test for JWT round-trip
    - **Property 2: JWT authentication round-trip**
    - **Validates: Requirements 2.1, 2.4**

- [~] 4. Build authentication API
  - [~] 4.1 Create User model (models/User.ts)
    - Implement createUser query
    - Implement findUserByUsername query
    - Implement findUserByEmail query
    - _Requirements: 2.5_
  
  - [~] 4.2 Create auth controller (controllers/authController.ts)
    - Implement register endpoint logic
    - Implement login endpoint logic
    - _Requirements: 2.1, 2.2, 2.5_
  
  - [~] 4.3 Create auth routes (routes/auth.ts)
    - POST /api/auth/register
    - POST /api/auth/login
    - _Requirements: 2.1, 2.5_
  
  - [ ]* 4.4 Write property test for invalid credentials
    - **Property 3: Invalid credentials rejection**
    - **Validates: Requirements 2.2**
  
  - [ ]* 4.5 Write property test for username/email uniqueness
    - **Property 4: Username and email uniqueness**
    - **Validates: Requirements 2.5**

- [~] 5. Create authentication middleware
  - [~] 5.1 Implement JWT validation middleware (middleware/auth.ts)
    - Extract token from Authorization header
    - Verify token and attach user to request
    - Return 401 for missing/invalid tokens
    - _Requirements: 2.4, 2.7_
  
  - [ ]* 5.2 Write property test for protected route authentication
    - **Property 5: Protected route authentication**
    - **Validates: Requirements 2.4, 2.7**

- [~] 6. Implement error handling
  - [~] 6.1 Create custom error classes (middleware/errorHandler.ts)
    - ValidationError class
    - NotFoundError class
    - _Requirements: 16.1, 16.3, 16.4_
  
  - [~] 6.2 Create global error handler middleware
    - Handle ValidationError (400)
    - Handle NotFoundError (404)
    - Handle duplicate key errors (409)
    - Handle generic errors (500)
    - Log all errors with context
    - _Requirements: 16.1, 16.2, 16.3, 16.4, 16.5, 16.6, 16.7_
  
  - [ ]* 6.3 Write property test for consistent error format
    - **Property 29: Consistent error response format**
    - **Validates: Requirements 16.1-16.6**
  
  - [ ]* 6.4 Write property test for error logging
    - **Property 30: Error logging completeness**
    - **Validates: Requirements 16.7**

- [~] 7. Set up Express application
  - [~] 7.1 Create main Express app (index.ts)
    - Initialize Express
    - Add JSON body parser
    - Add CORS middleware
    - Mount auth routes
    - Add error handler middleware
    - Add health check endpoint
    - Start server on port 3500
    - _Requirements: 3.2, 3.4_
  
  - [~] 7.2 Create database connection pool (config/database.ts)
    - Initialize pg.Pool with environment variables
    - Export query function
    - _Requirements: 3.4_

- [~] 8. Build frontend login page
  - [~] 8.1 Create API client (lib/api.ts)
    - Implement request method with auth header injection
    - Implement get, post, put, delete methods
    - Handle 401 responses with redirect to login
    - _Requirements: 2.6, 8.6, 8.7_
  
  - [ ]* 8.2 Write property test for token storage and inclusion
    - **Property 6: Token storage and inclusion**
    - **Validates: Requirements 2.6, 8.6**
  
  - [ ]* 8.3 Write property test for auth failure redirect
    - **Property 31: Authentication failure redirection**
    - **Validates: Requirements 8.7**
  
  - [~] 8.4 Create login route (routes/login.tsx)
    - Form with username and password fields
    - Submit to POST /api/auth/login
    - Store token in localStorage on success
    - Redirect to products page
    - Display errors
    - _Requirements: 2.1, 2.6_
  
  - [~] 8.5 Create ProtectedRoute component (components/ProtectedRoute.tsx)
    - Check for token in localStorage
    - Redirect to /login if missing
    - Render children if present
    - _Requirements: 2.7_
  
  - [ ]* 8.6 Write unit tests for login page
    - Test form submission
    - Test error display
    - Test redirect on success
    - _Requirements: 2.1, 2.6_

- [~] 9. Create Docker configuration
  - [~] 9.1 Create backend Dockerfile
    - Multi-stage build not needed for backend
    - Install production dependencies
    - Build TypeScript
    - Add health check
    - _Requirements: 4.2, 12.2_
  
  - [~] 9.2 Create frontend Dockerfile
    - Multi-stage build (builder + nginx)
    - Build Vite app in builder stage
    - Serve with nginx in production stage
    - _Requirements: 4.1, 12.1_
  
  - [~] 9.3 Create nginx configuration
    - Serve static files
    - Proxy /api requests to backend
    - _Requirements: 12.5_
  
  - [~] 9.4 Create docker-compose.yml
    - Define db service (postgres:15-alpine)
    - Define backend service with health check dependency
    - Define frontend service
    - Configure volumes for database persistence
    - Configure network
    - Load environment variables
    - _Requirements: 4.3, 4.4, 4.5, 4.6, 12.3, 12.7, 12.8, 13.1, 13.2, 13.3, 13.4_
  
  - [~] 9.5 Create .env.example file
    - Document all required environment variables
    - _Requirements: 3.4, 14.1, 14.2_
  
  - [ ]* 9.6 Write property test for environment validation
    - **Property 28: Environment variable validation**
    - **Validates: Requirements 14.4**

- [~] 10. Checkpoint - Test authentication flow
  - Start Docker Compose
  - Register a new user
  - Login with credentials
  - Verify JWT token is stored
  - Verify protected routes require authentication
  - Ensure all tests pass, ask the user if questions arise.

### Sprint 2: Product Management

- [~] 11. Implement product data model
  - [~] 11.1 Create Product model (models/Product.ts)
    - Implement createProduct query
    - Implement getAllProducts query
    - Implement getProductById query
    - Implement updateProduct query
    - Implement deleteProduct query
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5_
  
  - [~] 11.2 Create product validation utilities (utils/validation.ts)
    - Validate product name (1-255 chars)
    - Validate price (positive, max 2 decimals)
    - Validate SKU (1-100 chars, alphanumeric + hyphens)
    - Validate description (optional, max 5000 chars)
    - _Requirements: 6.1, 6.2, 6.3, 6.6_

- [~] 12. Build product API endpoints
  - [~] 12.1 Create product controller (controllers/productController.ts)
    - Implement create product logic with validation
    - Implement list products logic
    - Implement get product by ID logic
    - Implement update product logic with validation
    - Implement delete product logic
    - Handle automatic timestamps (created_at, updated_at)
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 6.4, 6.5, 6.7_
  
  - [~] 12.2 Create product routes (routes/products.ts)
    - GET /api/products (with auth middleware)
    - POST /api/products (with auth middleware)
    - GET /api/products/:id (with auth middleware)
    - PUT /api/products/:id (with auth middleware)
    - DELETE /api/products/:id (with auth middleware)
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 5.8_
  
  - [~] 12.3 Mount product routes in main app
    - Add product routes to Express app
    - _Requirements: 5.1-5.8_
  
  - [ ]* 12.4 Write property test for product creation and retrieval
    - **Property 8: Product creation and retrieval**
    - **Validates: Requirements 5.1, 5.3**
  
  - [ ]* 12.5 Write property test for product list completeness
    - **Property 9: Product list completeness**
    - **Validates: Requirements 5.2**
  
  - [ ]* 12.6 Write property test for product update persistence
    - **Property 10: Product update persistence**
    - **Validates: Requirements 5.4**
  
  - [ ]* 12.7 Write property test for product deletion
    - **Property 11: Product deletion removal**
    - **Validates: Requirements 5.5**
  
  - [ ]* 12.8 Write property test for product validation
    - **Property 12: Product validation enforcement**
    - **Validates: Requirements 5.6, 6.1, 6.2, 6.3, 6.6, 6.7**
  
  - [ ]* 12.9 Write property test for automatic timestamps
    - **Property 13: Automatic timestamp management**
    - **Validates: Requirements 6.4, 6.5**
  
  - [ ]* 12.10 Write property test for product endpoint authentication
    - **Property 14: Product endpoint authentication requirement**
    - **Validates: Requirements 5.8**

- [~] 13. Create product UI components
  - [~] 13.1 Create product list page (routes/products/index.tsx)
    - Fetch products from GET /api/products on mount
    - Display products in table/grid with name, SKU, price
    - Add action buttons (View, Edit, Delete)
    - Add link to create new product
    - Handle loading and error states
    - _Requirements: 7.1, 8.1_
  
  - [~] 13.2 Create product form component (components/ProductForm.tsx)
    - Form fields: name, description, price, SKU
    - Client-side validation
    - Display server validation errors
    - Disable submit button during request
    - _Requirements: 7.2, 7.3, 7.7, 7.8_
  
  - [~] 13.3 Create new product page (routes/products/new.tsx)
    - Use ProductForm component
    - Submit to POST /api/products
    - Redirect to product list on success
    - _Requirements: 7.2, 7.3, 8.2_
  
  - [~] 13.4 Create product detail page (routes/products/$id.tsx)
    - Fetch product from GET /api/products/:id
    - Display all product information
    - Add Edit and Delete buttons
    - _Requirements: 7.6_
  
  - [~] 13.5 Create edit product page (routes/products/$id.edit.tsx)
    - Fetch product to pre-fill form
    - Use ProductForm component with initial values
    - Submit to PUT /api/products/:id
    - Redirect to product detail on success
    - _Requirements: 7.4, 8.3_
  
  - [~] 13.6 Implement delete confirmation
    - Show confirmation dialog on delete click
    - Submit to DELETE /api/products/:id
    - Redirect to product list on success
    - _Requirements: 7.5, 8.4_
  
  - [ ]* 13.7 Write property test for frontend CRUD integration
    - **Property 15: Frontend product CRUD integration**
    - **Validates: Requirements 8.1, 8.2, 8.3, 8.4**
  
  - [ ]* 13.8 Write property test for error display
    - **Property 16: Frontend error display**
    - **Validates: Requirements 7.7, 8.5**
  
  - [ ]* 13.9 Write property test for form submission state
    - **Property 17: Form submission state management**
    - **Validates: Requirements 7.8**
  
  - [ ]* 13.10 Write property test for edit form pre-population
    - **Property 18: Edit form pre-population**
    - **Validates: Requirements 7.4, 7.6**
  
  - [ ]* 13.11 Write unit tests for product components
    - Test ProductForm validation
    - Test product list rendering
    - Test delete confirmation
    - _Requirements: 7.1-7.8_

- [~] 14. Checkpoint - Test product management
  - Create a new product via UI
  - View product details
  - Edit product
  - Delete product
  - Verify all operations work end-to-end
  - Ensure all tests pass, ask the user if questions arise.

### Sprint 3: Inventory Management & Deployment

- [~] 15. Implement inventory data model
  - [~] 15.1 Create Inventory model (models/Inventory.ts)
    - Implement createInventory query
    - Implement getAllInventories query
    - Implement getInventoryById query
    - Implement getInventoryWithProducts query (with JOIN)
    - Implement addProductToInventory query (INSERT or UPDATE)
    - _Requirements: 9.1, 9.2, 10.1, 10.5, 10.7_
  
  - [~] 15.2 Add inventory validation to validation utilities
    - Validate inventory name (1-255 chars)
    - Validate location (optional, max 255 chars)
    - Validate quantity (positive integer)
    - _Requirements: 9.3, 10.4_
  
  - [ ]* 15.3 Write property test for referential integrity
    - **Property 7: Referential integrity enforcement**
    - **Validates: Requirements 1.5**

- [~] 16. Build inventory API endpoints
  - [~] 16.1 Create inventory controller (controllers/inventoryController.ts)
    - Implement create inventory logic with validation
    - Implement list inventories logic
    - Implement get inventory with products logic
    - Implement add product to inventory logic with validation
    - Handle automatic timestamps (created_at, added_at)
    - _Requirements: 9.1, 9.2, 9.3, 9.4, 10.1, 10.2, 10.3, 10.4, 10.5, 10.6, 10.7_
  
  - [~] 16.2 Create inventory routes (routes/inventory.ts)
    - GET /api/inventory (with auth middleware)
    - POST /api/inventory (with auth middleware)
    - GET /api/inventory/:id/products (with auth middleware)
    - POST /api/inventory/:id/products (with auth middleware)
    - _Requirements: 9.1, 9.2, 9.5, 10.1_
  
  - [~] 16.3 Mount inventory routes in main app
    - Add inventory routes to Express app
    - _Requirements: 9.1-9.6, 10.1-10.7_
  
  - [ ]* 16.4 Write property test for inventory creation and retrieval
    - **Property 19: Inventory creation and retrieval**
    - **Validates: Requirements 9.1, 9.2**
  
  - [ ]* 16.5 Write property test for inventory validation
    - **Property 20: Inventory validation enforcement**
    - **Validates: Requirements 9.3, 9.6**
  
  - [ ]* 16.6 Write property test for inventory timestamps
    - **Property 21: Inventory automatic timestamps**
    - **Validates: Requirements 9.4**
  
  - [ ]* 16.7 Write property test for inventory authentication
    - **Property 22: Inventory endpoint authentication**
    - **Validates: Requirements 9.5**
  
  - [ ]* 16.8 Write property test for product-to-inventory association
    - **Property 23: Product-to-inventory association**
    - **Validates: Requirements 10.1, 10.7**
  
  - [ ]* 16.9 Write property test for association validation
    - **Property 24: Inventory-product validation**
    - **Validates: Requirements 10.2, 10.3, 10.4**
  
  - [ ]* 16.10 Write property test for idempotence
    - **Property 25: Inventory-product idempotence**
    - **Validates: Requirements 10.5**
  
  - [ ]* 16.11 Write property test for association timestamps
    - **Property 26: Inventory-product automatic timestamps**
    - **Validates: Requirements 10.6**

- [~] 17. Create inventory UI components
  - [~] 17.1 Create inventory list page (routes/inventory/index.tsx)
    - Fetch inventories from GET /api/inventory
    - Display inventories with name and location
    - Add link to create new inventory
    - _Requirements: 11.1_
  
  - [~] 17.2 Create inventory form component (components/InventoryForm.tsx)
    - Form fields: name, location
    - Client-side validation
    - Display server validation errors
    - _Requirements: 11.2, 11.7_
  
  - [~] 17.3 Create new inventory page (routes/inventory/new.tsx)
    - Use InventoryForm component
    - Submit to POST /api/inventory
    - Redirect to inventory list on success
    - _Requirements: 11.2, 11.3_
  
  - [~] 17.4 Create inventory detail page (routes/inventory/$id.tsx)
    - Fetch inventory with products from GET /api/inventory/:id/products
    - Display inventory info and product list with quantities
    - Add "Add Product" button
    - _Requirements: 11.4_
  
  - [~] 17.5 Add product-to-inventory form
    - Product selection dropdown (fetch from GET /api/products)
    - Quantity input field
    - Submit to POST /api/inventory/:id/products
    - Refresh inventory view on success
    - _Requirements: 11.5, 11.6_
  
  - [ ]* 17.6 Write property test for frontend inventory integration
    - **Property 27: Frontend inventory integration**
    - **Validates: Requirements 11.3, 11.4, 11.6**
  
  - [ ]* 17.7 Write unit tests for inventory components
    - Test InventoryForm validation
    - Test inventory list rendering
    - Test add product to inventory
    - _Requirements: 11.1-11.7_

- [~] 18. Optimize Docker for production
  - [~] 18.1 Update frontend Dockerfile for production
    - Verify multi-stage build is optimized
    - Minimize image size
    - _Requirements: 12.1_
  
  - [~] 18.2 Update backend Dockerfile for production
    - Add health check if not present
    - Use production dependencies only
    - _Requirements: 12.2_
  
  - [~] 18.3 Update docker-compose.yml for production
    - Add restart policies
    - Configure resource limits
    - Add logging configuration
    - _Requirements: 12.3_
  
  - [~] 18.4 Configure Docker secrets
    - Use Docker secrets or env files for sensitive data
    - Update docker-compose to use secrets
    - _Requirements: 12.4_
  
  - [~] 18.5 Verify database initialization
    - Ensure init.sql runs on first startup
    - Test database persistence across restarts
    - _Requirements: 12.6, 13.5_
  
  - [~] 18.6 Configure backend to wait for database
    - Add health check dependency in docker-compose
    - Verify backend starts only after database is ready
    - _Requirements: 13.6_

- [~] 19. Write comprehensive test suite
  - [ ]* 19.1 Write backend unit tests
    - Test all API endpoints with valid inputs
    - Test all API endpoints with invalid inputs
    - Test authentication middleware
    - Test error handling middleware
    - _Requirements: 15.1, 15.2_
  
  - [ ]* 19.2 Write frontend component tests
    - Test all product components
    - Test all inventory components
    - Test authentication components
    - Test protected route behavior
    - _Requirements: 15.3, 15.4_
  
  - [ ]* 19.3 Write integration tests
    - Test complete authentication flow
    - Test complete product CRUD flow
    - Test complete inventory flow
    - _Requirements: 15.5, 15.6_
  
  - [ ]* 19.4 Verify code coverage
    - Run coverage reports for backend
    - Ensure minimum 70% coverage for API routes
    - _Requirements: 15.7_
  
  - [~] 19.5 Add test scripts to package.json
    - Add test, test:watch, test:coverage scripts
    - _Requirements: 15.8_

- [~] 20. Create API documentation
  - [~] 20.1 Document all API endpoints
    - List all endpoints with methods
    - Document request formats
    - Document response formats
    - Include example requests and responses
    - _Requirements: 17.1, 17.2_
  
  - [~] 20.2 Document authentication requirements
    - Explain JWT token usage
    - Document how to include token in requests
    - _Requirements: 17.3_
  
  - [~] 20.3 Document error responses
    - List all possible error status codes
    - Explain error response format
    - Provide examples of each error type
    - _Requirements: 17.4_
  
  - [~] 20.4 Create README or API documentation file
    - Consolidate all API documentation
    - Add setup instructions
    - Add deployment instructions
    - _Requirements: 17.5_

- [~] 21. Final checkpoint - Complete system test
  - Start complete Docker Compose stack
  - Test authentication flow
  - Test product management flow
  - Test inventory management flow
  - Verify all containers are healthy
  - Verify data persists across restarts
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional testing tasks that can be skipped for faster MVP
- Each task references specific requirements for traceability
- Property tests validate universal correctness properties (run 100+ iterations each)
- Unit tests validate specific examples and edge cases
- Checkpoints ensure incremental validation at key milestones
- All code should be production-ready with proper error handling and validation
