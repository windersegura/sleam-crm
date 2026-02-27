# Design Document: CRM 3-Sprint Development Plan

## Overview

This design outlines a full-stack CRM application built across three sprints. The system uses React (sleam-crm) for the frontend, Express (crm-server) for the backend API, PostgreSQL for data persistence, and Docker for deployment.

**Sprint 1:** Foundation - Authentication, database schema, Docker setup
**Sprint 2:** Product Management - CRUD operations for products
**Sprint 3:** Inventory Management - Inventory tracking and deployment

## Architecture

### System Components

```
┌─────────────────────────────────────────────────────────────┐
│                         Docker Host                          │
│                                                              │
│  ┌──────────────┐      ┌──────────────┐      ┌──────────┐ │
│  │   Frontend   │      │   Backend    │      │ Database │ │
│  │   (nginx)    │─────▶│  (Express)   │─────▶│(Postgres)│ │
│  │   Port 80    │      │  Port 3500   │      │ Port 5432│ │
│  └──────────────┘      └──────────────┘      └──────────┘ │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Frontend (sleam-crm):**
- React 19.2.0 with TypeScript
- TanStack Router for file-based routing
- Vite for build tooling
- Tailwind CSS 4.x for styling
- Vitest + React Testing Library for testing

**Backend (crm-server):**
- Node.js with Express 5.2+
- TypeScript for type safety
- bcrypt for password hashing
- jsonwebtoken for JWT tokens
- pg (node-postgres) for PostgreSQL

**Database:**
- PostgreSQL 15+ with relational schema
- Foreign key constraints for referential integrity

**Deployment:**
- Docker with multi-stage builds
- Docker Compose for orchestration
- nginx for serving frontend static files

## Components and Interfaces

### Database Schema

```sql
-- Users table for authentication
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(50) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Products table
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  price DECIMAL(10, 2) NOT NULL CHECK (price > 0),
  sku VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Inventory locations
CREATE TABLE inventory (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  location VARCHAR(255),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Junction table for products in inventory
CREATE TABLE inventory_products (
  inventory_id INTEGER REFERENCES inventory(id) ON DELETE CASCADE,
  product_id INTEGER REFERENCES products(id) ON DELETE CASCADE,
  quantity INTEGER NOT NULL CHECK (quantity > 0),
  added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (inventory_id, product_id)
);

-- Indexes for performance
CREATE INDEX idx_products_sku ON products(sku);
CREATE INDEX idx_inventory_products_inventory ON inventory_products(inventory_id);
CREATE INDEX idx_inventory_products_product ON inventory_products(product_id);
```

### Backend API Structure

**Authentication Endpoints:**

```typescript
POST /api/auth/register
  Body: { username: string, email: string, password: string }
  Response: { user: User, token: string }
  Status: 201 | 400 | 409

POST /api/auth/login
  Body: { username: string, password: string }
  Response: { user: User, token: string }
  Status: 200 | 401
```

**Product Endpoints (require authentication):**

```typescript
GET /api/products
  Response: { products: Product[] }
  Status: 200 | 401

POST /api/products
  Body: { name: string, description?: string, price: number, sku: string }
  Response: { product: Product }
  Status: 201 | 400 | 401 | 409

GET /api/products/:id
  Response: { product: Product }
  Status: 200 | 401 | 404

PUT /api/products/:id
  Body: Partial<Product>
  Response: { product: Product }
  Status: 200 | 400 | 401 | 404 | 409

DELETE /api/products/:id
  Status: 204 | 401 | 404
```

**Inventory Endpoints (require authentication):**

```typescript
GET /api/inventory
  Response: { inventories: Inventory[] }
  Status: 200 | 401

POST /api/inventory
  Body: { name: string, location?: string }
  Response: { inventory: Inventory }
  Status: 201 | 400 | 401

GET /api/inventory/:id/products
  Response: { inventory: InventoryWithProducts }
  Status: 200 | 401 | 404

POST /api/inventory/:id/products
  Body: { productId: number, quantity: number }
  Response: { inventory: InventoryWithProducts }
  Status: 200 | 400 | 401 | 404
```

### Backend Implementation Structure

**Directory Structure:**
```
crm-server/src/
├── index.ts              # Express app setup
├── config/
│   └── database.ts       # PostgreSQL connection pool
├── middleware/
│   ├── auth.ts           # JWT validation middleware
│   └── errorHandler.ts   # Global error handler
├── routes/
│   ├── auth.ts           # Authentication routes
│   ├── products.ts       # Product routes
│   └── inventory.ts      # Inventory routes
├── controllers/
│   ├── authController.ts
│   ├── productController.ts
│   └── inventoryController.ts
├── models/
│   ├── User.ts
│   ├── Product.ts
│   └── Inventory.ts
├── utils/
│   ├── jwt.ts            # JWT generation/validation
│   ├── password.ts       # bcrypt hashing
│   └── validation.ts     # Input validation
└── types/
    └── express.d.ts      # Extended Express types
```

**Authentication Middleware:**
```typescript
// middleware/auth.ts
export async function authenticate(req: Request, res: Response, next: NextFunction) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({ message: 'Authentication required' });
  }
  
  try {
    const decoded = verifyToken(token);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ message: 'Invalid or expired token' });
  }
}
```

**Error Handler:**
```typescript
// middleware/errorHandler.ts
export function errorHandler(err: Error, req: Request, res: Response, next: NextFunction) {
  console.error(err);
  
  if (err instanceof ValidationError) {
    return res.status(400).json({ message: err.message, details: err.details });
  }
  
  if (err.message.includes('duplicate key')) {
    return res.status(409).json({ message: 'Resource already exists' });
  }
  
  res.status(500).json({ message: 'Internal server error' });
}
```

### Frontend Implementation Structure

**Directory Structure:**
```
sleam-crm/src/
├── main.tsx              # App entry point
├── router.tsx            # Router config
├── styles.css            # Global styles
├── routes/
│   ├── __root.tsx        # Root layout
│   ├── index.tsx         # Home page
│   ├── login.tsx         # Login page
│   ├── products/
│   │   ├── index.tsx     # Product list
│   │   ├── new.tsx       # Create product
│   │   ├── $id.tsx       # View product
│   │   └── $id.edit.tsx  # Edit product
│   └── inventory/
│       ├── index.tsx     # Inventory list
│       ├── new.tsx       # Create inventory
│       └── $id.tsx       # View inventory + add products
├── components/
│   ├── ProtectedRoute.tsx
│   ├── ProductForm.tsx
│   └── InventoryForm.tsx
├── lib/
│   ├── api.ts            # API client
│   └── auth.ts           # Auth utilities
└── types/
    └── index.ts          # Shared types
```

**API Client:**
```typescript
// lib/api.ts
class ApiClient {
  private baseURL = import.meta.env.VITE_API_URL || 'http://localhost:3500';
  
  private async request<T>(endpoint: string, options?: RequestInit): Promise<T> {
    const token = localStorage.getItem('token');
    const headers = {
      'Content-Type': 'application/json',
      ...(token && { Authorization: `Bearer ${token}` }),
      ...options?.headers,
    };
    
    const response = await fetch(`${this.baseURL}${endpoint}`, {
      ...options,
      headers,
    });
    
    if (response.status === 401) {
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    
    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.message);
    }
    
    return response.json();
  }
  
  async get<T>(endpoint: string) {
    return this.request<T>(endpoint);
  }
  
  async post<T>(endpoint: string, data: any) {
    return this.request<T>(endpoint, {
      method: 'POST',
      body: JSON.stringify(data),
    });
  }
  
  async put<T>(endpoint: string, data: any) {
    return this.request<T>(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data),
    });
  }
  
  async delete(endpoint: string) {
    return this.request(endpoint, { method: 'DELETE' });
  }
}

export const api = new ApiClient();
```

**Protected Route Component:**
```typescript
// components/ProtectedRoute.tsx
export function ProtectedRoute({ children }: { children: React.ReactNode }) {
  const token = localStorage.getItem('token');
  
  if (!token) {
    return <Navigate to="/login" />;
  }
  
  return <>{children}</>;
}
```

## Data Models

**TypeScript Types (shared between frontend and backend):**

```typescript
// User (password_hash excluded from client-side type)
interface User {
  id: number;
  username: string;
  email: string;
  created_at: string;
}

// Product
interface Product {
  id: number;
  name: string;
  description: string | null;
  price: number;
  sku: string;
  created_at: string;
  updated_at: string;
}

// Inventory
interface Inventory {
  id: number;
  name: string;
  location: string | null;
  created_at: string;
}

// Inventory with products
interface InventoryWithProducts extends Inventory {
  products: Array<{
    product: Product;
    quantity: number;
    added_at: string;
  }>;
}
```

**Validation Rules:**

```typescript
// User validation
const userSchema = {
  username: {
    minLength: 3,
    maxLength: 50,
    pattern: /^[a-zA-Z0-9_]+$/,
  },
  email: {
    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
  },
  password: {
    minLength: 8,
    pattern: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/,
  },
};

// Product validation
const productSchema = {
  name: {
    minLength: 1,
    maxLength: 255,
  },
  description: {
    maxLength: 5000,
    optional: true,
  },
  price: {
    min: 0.01,
    max: 999999.99,
  },
  sku: {
    minLength: 1,
    maxLength: 100,
    pattern: /^[a-zA-Z0-9-]+$/,
  },
};

// Inventory validation
const inventorySchema = {
  name: {
    minLength: 1,
    maxLength: 255,
  },
  location: {
    maxLength: 255,
    optional: true,
  },
};

// Inventory product validation
const inventoryProductSchema = {
  quantity: {
    min: 1,
    type: 'integer',
  },
};
```


## Correctness Properties

A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.

### Sprint 1 Properties: Foundation & Authentication

**Property 1: Password hashing consistency**
*For any* user registration or password change, the stored password in the database should be a valid bcrypt hash, never the plaintext password.
**Validates: Requirements 2.3**

**Property 2: JWT authentication round-trip**
*For any* valid user credentials, logging in should generate a JWT token that, when used to access protected routes, successfully authenticates the user.
**Validates: Requirements 2.1, 2.4**

**Property 3: Invalid credentials rejection**
*For any* invalid username/password combination, the authentication system should return an error without revealing whether the username or password was incorrect.
**Validates: Requirements 2.2**

**Property 4: Username and email uniqueness**
*For any* existing user, attempting to register a new user with the same username or email should be rejected with an appropriate error.
**Validates: Requirements 2.5**

**Property 5: Protected route authentication**
*For any* protected route, requests without a valid JWT token should be rejected with 401 status, and the frontend should redirect to the login page.
**Validates: Requirements 2.4, 2.7**

**Property 6: Token storage and inclusion**
*For any* successful login, the frontend should store the JWT token and include it in the Authorization header of all subsequent API requests.
**Validates: Requirements 2.6, 8.6**

**Property 7: Referential integrity enforcement**
*For any* attempt to insert an inventory-product association with non-existent inventory_id or product_id, the database should reject the operation.
**Validates: Requirements 1.5**

### Sprint 2 Properties: Product Management

**Property 8: Product creation and retrieval**
*For any* valid product data, creating a product via POST /api/products should result in that product being retrievable via GET /api/products/:id with all fields matching.
**Validates: Requirements 5.1, 5.3**

**Property 9: Product list completeness**
*For any* set of created products, GET /api/products should return all products that have been created and not deleted.
**Validates: Requirements 5.2**

**Property 10: Product update persistence**
*For any* existing product and valid update data, updating via PUT /api/products/:id should result in the product having the new values when subsequently retrieved.
**Validates: Requirements 5.4**

**Property 11: Product deletion removal**
*For any* existing product, deleting via DELETE /api/products/:id should result in that product no longer appearing in the product list and returning 404 when accessed by ID.
**Validates: Requirements 5.5**

**Property 12: Product validation enforcement**
*For any* product creation or update request with invalid data (empty name, non-positive price, duplicate SKU, or invalid SKU format), the backend should return 400 status with detailed field-level error messages.
**Validates: Requirements 5.6, 6.1, 6.2, 6.3, 6.6, 6.7**

**Property 13: Automatic timestamp management**
*For any* product creation, created_at should be automatically set to the current timestamp, and for any product update, updated_at should be automatically updated to the current timestamp.
**Validates: Requirements 6.4, 6.5**

**Property 14: Product endpoint authentication requirement**
*For any* product API endpoint (GET, POST, PUT, DELETE), requests without valid authentication should be rejected with 401 status.
**Validates: Requirements 5.8**

**Property 15: Frontend product CRUD integration**
*For any* product operation performed through the UI (create, update, delete), the frontend should send the appropriate API request and update the UI to reflect the change.
**Validates: Requirements 8.1, 8.2, 8.3, 8.4**

**Property 16: Frontend error display**
*For any* API error response (validation errors, authentication errors, not found errors), the frontend should display the error message in a user-friendly format.
**Validates: Requirements 7.7, 8.5**

**Property 17: Form submission state management**
*For any* form submission in the frontend, the submit button should be disabled while the API request is in progress and re-enabled when the request completes.
**Validates: Requirements 7.8**

**Property 18: Edit form pre-population**
*For any* product being edited, the edit form should be pre-filled with the product's current values (name, description, price, SKU).
**Validates: Requirements 7.4, 7.6**

### Sprint 3 Properties: Inventory Management & Deployment

**Property 19: Inventory creation and retrieval**
*For any* valid inventory data, creating an inventory via POST /api/inventory should result in that inventory being retrievable via GET /api/inventory with all fields matching.
**Validates: Requirements 9.1, 9.2**

**Property 20: Inventory validation enforcement**
*For any* inventory creation request with invalid data (empty name), the backend should return 400 status with validation error messages.
**Validates: Requirements 9.3, 9.6**

**Property 21: Inventory automatic timestamps**
*For any* inventory creation, created_at should be automatically set to the current timestamp.
**Validates: Requirements 9.4**

**Property 22: Inventory endpoint authentication**
*For any* inventory API endpoint, requests without valid authentication should be rejected with 401 status.
**Validates: Requirements 9.5**

**Property 23: Product-to-inventory association**
*For any* valid inventory ID, product ID, and positive quantity, adding the product to the inventory should create or update the association, and the inventory should include that product with the specified quantity when retrieved.
**Validates: Requirements 10.1, 10.7**

**Property 24: Inventory-product validation**
*For any* attempt to add a product to inventory with non-existent inventory ID, non-existent product ID, or non-positive quantity, the backend should return 400 status with appropriate error messages.
**Validates: Requirements 10.2, 10.3, 10.4**

**Property 25: Inventory-product idempotence**
*For any* product already in an inventory, adding the same product again should update the quantity rather than creating a duplicate entry.
**Validates: Requirements 10.5**

**Property 26: Inventory-product automatic timestamps**
*For any* product added to inventory, added_at should be automatically set to the current timestamp.
**Validates: Requirements 10.6**

**Property 27: Frontend inventory integration**
*For any* inventory operation performed through the UI (create inventory, add product to inventory), the frontend should send the appropriate API request and update the UI to reflect the change.
**Validates: Requirements 11.3, 11.4, 11.6**

**Property 28: Environment variable validation**
*For any* required environment variable (database credentials, JWT secret, API URLs), if it is not set when services start, the system should fail gracefully with a clear error message.
**Validates: Requirements 14.4**

### Cross-Sprint Properties: Error Handling & API Consistency

**Property 29: Consistent error response format**
*For any* error condition (validation, authentication, authorization, not found, server error), the backend should return a JSON response with consistent structure containing status code, message, and optional details.
**Validates: Requirements 16.1, 16.2, 16.3, 16.4, 16.5, 16.6**

**Property 30: Error logging completeness**
*For any* error that occurs in the backend, the error should be logged with sufficient detail (timestamp, error type, stack trace, request context) for debugging purposes.
**Validates: Requirements 16.7**

**Property 31: Authentication failure redirection**
*For any* 401 response from the backend, the frontend should redirect the user to the login page.
**Validates: Requirements 8.7**

## Error Handling

**Consistent Error Response Format:**

```typescript
interface ErrorResponse {
  message: string;
  details?: Record<string, string>; // Field-level validation errors
}
```

**HTTP Status Codes:**
- 200: Success (GET, PUT)
- 201: Created (POST)
- 204: No Content (DELETE)
- 400: Bad Request (validation errors)
- 401: Unauthorized (missing/invalid token)
- 403: Forbidden (insufficient permissions)
- 404: Not Found (resource doesn't exist)
- 409: Conflict (duplicate unique values)
- 500: Internal Server Error

**Backend Error Handling:**

```typescript
// Custom error classes
class ValidationError extends Error {
  constructor(public details: Record<string, string>) {
    super('Validation failed');
  }
}

class NotFoundError extends Error {
  constructor(resource: string) {
    super(`${resource} not found`);
  }
}

// Global error handler
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error(err);
  
  if (err instanceof ValidationError) {
    return res.status(400).json({ message: err.message, details: err.details });
  }
  
  if (err instanceof NotFoundError) {
    return res.status(404).json({ message: err.message });
  }
  
  if (err.message.includes('duplicate key')) {
    return res.status(409).json({ message: 'Resource already exists' });
  }
  
  res.status(500).json({ message: 'Internal server error' });
});
```

**Frontend Error Handling:**

```typescript
// Display errors to users
function handleApiError(error: Error) {
  // Show toast notification or inline error message
  toast.error(error.message);
}

// Automatic retry for network errors
async function fetchWithRetry<T>(fn: () => Promise<T>, retries = 3): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    if (retries > 0 && error instanceof NetworkError) {
      await delay(1000);
      return fetchWithRetry(fn, retries - 1);
    }
    throw error;
  }
}
```

## Testing Strategy

### Testing Approach

This project uses both unit tests and property-based tests for comprehensive coverage:

- **Unit tests**: Specific examples, edge cases, error conditions
- **Property-based tests**: Universal properties across many generated inputs

Both are necessary and complementary. Unit tests document expected behavior through concrete examples. Property-based tests verify correctness across a wide input space.

### Backend Testing

**Test Framework:** Vitest with supertest for API testing

**Unit Tests:**
```typescript
// Example: Product API endpoint test
describe('POST /api/products', () => {
  it('creates a product with valid data', async () => {
    const response = await request(app)
      .post('/api/products')
      .set('Authorization', `Bearer ${token}`)
      .send({
        name: 'Test Product',
        price: 29.99,
        sku: 'TEST-001',
      });
    
    expect(response.status).toBe(201);
    expect(response.body.product).toMatchObject({
      name: 'Test Product',
      price: 29.99,
      sku: 'TEST-001',
    });
  });
  
  it('rejects product with negative price', async () => {
    const response = await request(app)
      .post('/api/products')
      .set('Authorization', `Bearer ${token}`)
      .send({
        name: 'Invalid Product',
        price: -10,
        sku: 'TEST-002',
      });
    
    expect(response.status).toBe(400);
  });
});
```

**Property-Based Tests (using fast-check):**

Each property test runs 100+ iterations with randomly generated data:

```typescript
import fc from 'fast-check';

// Feature: crm-3-sprint-plan, Property 8: Product creation and retrieval
test('created products are retrievable with matching fields', async () => {
  await fc.assert(
    fc.asyncProperty(
      fc.record({
        name: fc.string({ minLength: 1, maxLength: 255 }),
        description: fc.option(fc.string({ maxLength: 5000 })),
        price: fc.float({ min: 0.01, max: 999999.99, noNaN: true }),
        sku: fc.string({ minLength: 1, maxLength: 100 })
          .map(s => s.replace(/[^a-zA-Z0-9-]/g, '') || 'SKU-001'),
      }),
      async (productData) => {
        const created = await createProduct(productData, token);
        const retrieved = await getProduct(created.id, token);
        
        expect(retrieved.name).toBe(productData.name);
        expect(retrieved.price).toBeCloseTo(productData.price, 2);
        expect(retrieved.sku).toBe(productData.sku);
      }
    ),
    { numRuns: 100 }
  );
});

// Feature: crm-3-sprint-plan, Property 12: Product validation enforcement
test('invalid product data is rejected with 400', async () => {
  await fc.assert(
    fc.asyncProperty(
      fc.record({
        name: fc.constantFrom('', '   '), // Invalid: empty or whitespace
        price: fc.constantFrom(-1, 0, -100), // Invalid: non-positive
        sku: fc.string({ minLength: 1, maxLength: 100 }),
      }),
      async (invalidData) => {
        const response = await request(app)
          .post('/api/products')
          .set('Authorization', `Bearer ${token}`)
          .send(invalidData);
        
        expect(response.status).toBe(400);
        expect(response.body.message).toBeTruthy();
      }
    ),
    { numRuns: 100 }
  );
});
```

**Coverage Goal:** Minimum 70% code coverage for API routes

### Frontend Testing

**Test Framework:** Vitest + React Testing Library

**Component Tests:**
```typescript
// Example: Product form component test
describe('ProductForm', () => {
  it('submits valid product data', async () => {
    const onSubmit = vi.fn();
    render(<ProductForm onSubmit={onSubmit} />);
    
    await userEvent.type(screen.getByLabelText('Name'), 'Test Product');
    await userEvent.type(screen.getByLabelText('Price'), '29.99');
    await userEvent.type(screen.getByLabelText('SKU'), 'TEST-001');
    await userEvent.click(screen.getByRole('button', { name: 'Submit' }));
    
    expect(onSubmit).toHaveBeenCalledWith({
      name: 'Test Product',
      price: 29.99,
      sku: 'TEST-001',
    });
  });
  
  it('displays validation errors', async () => {
    render(<ProductForm onSubmit={vi.fn()} />);
    
    await userEvent.click(screen.getByRole('button', { name: 'Submit' }));
    
    expect(screen.getByText(/name is required/i)).toBeInTheDocument();
  });
});
```

**Integration Tests:**
```typescript
// Test complete flow with mocked API
describe('Product Management Flow', () => {
  it('creates, views, and deletes a product', async () => {
    // Mock API responses
    server.use(
      http.post('/api/products', () => {
        return HttpResponse.json({ product: mockProduct });
      })
    );
    
    render(<App />);
    
    // Navigate to create product
    await userEvent.click(screen.getByText('New Product'));
    
    // Fill form
    await userEvent.type(screen.getByLabelText('Name'), 'Test Product');
    await userEvent.type(screen.getByLabelText('Price'), '29.99');
    await userEvent.type(screen.getByLabelText('SKU'), 'TEST-001');
    await userEvent.click(screen.getByRole('button', { name: 'Create' }));
    
    // Verify product appears in list
    expect(await screen.findByText('Test Product')).toBeInTheDocument();
  });
});
```

### Docker Testing

**Container Health Checks:**
```yaml
# docker-compose.yml
services:
  backend:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3500/health"]
      interval: 30s
      timeout: 10s
      retries: 3
  
  db:
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

**Integration Test:**
```bash
# Test script to verify Docker deployment
#!/bin/bash
docker-compose up -d
sleep 10

# Test backend health
curl -f http://localhost:3500/health || exit 1

# Test frontend serves
curl -f http://localhost:80 || exit 1

# Test database connection
docker-compose exec backend npm run db:test || exit 1

echo "All Docker tests passed"
```

## Docker Configuration

### Frontend Dockerfile (Multi-stage)

```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Backend Dockerfile

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build
EXPOSE 3500
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3500/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"
CMD ["node", "dist/index.js"]
```

### Docker Compose

```yaml
version: '3.8'

services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - crm_network

  backend:
    build:
      context: ./crm-server
      dockerfile: Dockerfile
    environment:
      DATABASE_URL: postgresql://${DB_USER}:${DB_PASSWORD}@db:5432/${DB_NAME}
      JWT_SECRET: ${JWT_SECRET}
      PORT: 3500
    depends_on:
      db:
        condition: service_healthy
    networks:
      - crm_network
    restart: unless-stopped

  frontend:
    build:
      context: ./sleam-crm
      dockerfile: Dockerfile
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - crm_network
    restart: unless-stopped

volumes:
  postgres_data:

networks:
  crm_network:
    driver: bridge
```

### nginx Configuration

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    # Serve static files
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Proxy API requests to backend
    location /api {
        proxy_pass http://backend:3500;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Environment Variables

**.env.example:**
```bash
# Database
DB_NAME=crm_db
DB_USER=crm_user
DB_PASSWORD=change_me_in_production

# Backend
JWT_SECRET=change_me_to_random_256_bit_string
PORT=3500

# Frontend
VITE_API_URL=http://localhost:3500
```


## Implementation Notes

### Sprint Execution Order

**Sprint 1: Foundation (Week 1)**
1. Initialize both projects with TypeScript configs
2. Set up PostgreSQL schema with init.sql
3. Implement bcrypt password hashing utility
4. Implement JWT generation/validation utilities
5. Create auth endpoints (register, login)
6. Create auth middleware for protected routes
7. Build frontend login page with TanStack Router
8. Create ProtectedRoute component
9. Write Dockerfiles and docker-compose.yml
10. Test end-to-end authentication flow

**Sprint 2: Products (Week 2)**
1. Create product model and database queries
2. Implement product validation
3. Build product CRUD API endpoints
4. Add auth middleware to product routes
5. Create product list page (routes/products/index.tsx)
6. Create product form page (routes/products/new.tsx, $id.edit.tsx)
7. Create product detail page (routes/products/$id.tsx)
8. Wire up API client to components
9. Write unit tests for product endpoints
10. Write property-based tests for product operations

**Sprint 3: Inventory & Deployment (Week 3)**
1. Create inventory model and queries
2. Build inventory API endpoints
3. Implement inventory-product association logic
4. Create inventory UI pages
5. Optimize Docker images (multi-stage builds)
6. Configure nginx for frontend
7. Add health checks to containers
8. Set up environment variable management
9. Write comprehensive test suite
10. Document API endpoints

### Key Technical Decisions

**Why PostgreSQL over MySQL:**
- Better JSON support for future extensibility
- Stronger constraint enforcement
- Superior performance for complex queries
- More robust ACID compliance

**Why fast-check for property-based testing:**
- Native TypeScript support
- Rich built-in generators (arbitraries)
- Excellent Vitest integration
- Active community and maintenance

**Why localStorage for JWT storage:**
- Simple implementation for MVP
- Sufficient security for this use case
- Consider httpOnly cookies for production if XSS is a concern

**Why fetch over Axios:**
- Native browser API, no extra dependency
- Sufficient for this project's needs
- TypeScript support out of the box

### Security Best Practices

**Authentication:**
- bcrypt cost factor: 12 (good balance of security and performance)
- JWT expiration: 24 hours (adjust based on requirements)
- Password requirements: min 8 chars, uppercase, lowercase, number
- Never log passwords or tokens

**API Security:**
- Use parameterized queries (pg library handles this)
- Validate all inputs before processing
- Return generic error messages (don't leak implementation details)
- Implement rate limiting in production (express-rate-limit)

**Docker Security:**
- Run containers as non-root user
- Use Docker secrets for sensitive data in production
- Keep base images updated
- Scan images for vulnerabilities (docker scan)

**Environment Variables:**
- Never commit .env to version control
- Use strong random values for JWT_SECRET (256+ bits)
- Rotate secrets regularly in production
- Validate required env vars on startup

### Performance Considerations

**Database:**
- Add indexes on frequently queried columns (sku, username, email)
- Use connection pooling (pg.Pool)
- Consider read replicas for scaling

**API:**
- Implement pagination for list endpoints
- Add caching headers for static resources
- Consider Redis for session storage at scale

**Frontend:**
- Code splitting with React.lazy()
- Optimize images and assets
- Use Vite's build optimization
- Implement virtual scrolling for large lists

### Monitoring and Logging

**Backend Logging:**
```typescript
// Use structured logging
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
});

// Log all requests
app.use((req, res, next) => {
  logger.info({
    method: req.method,
    path: req.path,
    userId: req.user?.id,
  });
  next();
});
```

**Health Check Endpoint:**
```typescript
app.get('/health', async (req, res) => {
  try {
    await db.query('SELECT 1');
    res.status(200).json({ status: 'healthy', database: 'connected' });
  } catch (error) {
    res.status(503).json({ status: 'unhealthy', database: 'disconnected' });
  }
});
```

### Future Enhancements

**Phase 2 Features:**
- Refresh tokens for extended sessions
- Role-based access control (admin, user roles)
- Product categories and tags
- Inventory alerts (low stock notifications)
- Audit logs for all operations

**Scalability:**
- Horizontal scaling with load balancer
- Database read replicas
- Redis for caching and sessions
- Message queue for async operations

**Observability:**
- Application performance monitoring (APM)
- Distributed tracing
- Metrics dashboard (Grafana)
- Error tracking (Sentry)
