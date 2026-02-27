# Product Overview

Full-stack CRM application for tracking inventory and sales. Users can manage products, inventory, and authenticate to access the system.

## Core Features
- User authentication and login
- Product management (create, edit, delete, view, list)
- Inventory tracking and management
- Product-to-inventory association

## Architecture
The system consists of two separate applications:

### Frontend (sleam-crm)
React-based SPA that provides the user interface for the CRM system.

### Backend (crm-server)
REST API built with Express that handles data operations and business logic. The frontend communicates with this API for all data operations.
