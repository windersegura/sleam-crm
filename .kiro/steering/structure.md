# Project Structure

## Workspace Organization

This is a multi-root workspace with two separate applications:
- `sleam-crm/` - Frontend React application
- `crm-server/` - Backend Express API

## Frontend (sleam-crm/)

### Root Directory
- `index.html` - Entry HTML file for Vite
- `vite.config.ts` - Vite configuration with plugins
- `tsconfig.json` - TypeScript compiler configuration
- `package.json` - Dependencies and scripts

### Source Directory (`src/`)
- `main.tsx` - Application entry point, initializes React and router
- `router.tsx` - Router configuration and type declarations
- `routeTree.gen.ts` - Auto-generated route tree (do not edit manually)
- `styles.css` - Global styles

### Routes (`src/routes/`)
File-based routing powered by TanStack Router:
- `__root.tsx` - Root layout component with devtools
- `index.tsx` - Home page route (`/`)

Route files export a `Route` object created with TanStack Router's route creation functions.

### Public Assets (`public/`)
Static assets served directly:
- Favicon and logo images
- `manifest.json` for PWA configuration
- `robots.txt` for SEO

### Frontend Conventions

#### Routing
- File-based routing: files in `src/routes/` automatically become routes
- `__root.tsx` defines the root layout with `<Outlet />` for nested routes
- Route components are exported via `createFileRoute()` or `createRootRoute()`

#### Path Aliases
- `@/*` resolves to `src/*` (configured in tsconfig and vite)
- Example: `import { Component } from '@/components/Component'`

#### Component Structure
- Use functional components with TypeScript
- Export route components through TanStack Router's route creation functions
- Co-locate route-specific components within route files when small

#### Styling
- Tailwind utility classes for styling
- Global styles in `src/styles.css`
- Responsive design with Tailwind breakpoints (`md:`, `lg:`, etc.)

## Backend (crm-server/)

### Root Directory
- `package.json` - Dependencies and scripts
- `tsconfig.json` - TypeScript compiler configuration
- `README.md` - Backend documentation

### Source Directory (`src/`)
- `index.ts` - Express server entry point and configuration

### Backend Conventions

#### Server Structure
- Express application initialized in `src/index.ts`
- Server listens on port 3500 by default (configurable via PORT env variable)
- TypeScript compiled to `dist/` directory (when build script is configured)

#### API Design
- RESTful API endpoints for CRM operations
- JSON request/response format
- Express middleware for request processing
