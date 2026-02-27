# Tech Stack

## Frontend (sleam-crm)

### Framework
- React 19.2.0 with TypeScript
- TanStack Router for file-based routing and navigation
- Vite as build tool and dev server

### Styling
- Tailwind CSS 4.x with Vite plugin
- Lucide React for icons

### Development Tools
- TypeScript 5.7+ with strict mode enabled
- Vitest for testing with jsdom
- React Testing Library for component tests
- TanStack DevTools for debugging

### Build System
Vite with plugins:
- `@tanstack/router-plugin` - Auto-generates route tree
- `@vitejs/plugin-react` - React support
- `@tailwindcss/vite` - Tailwind integration
- `vite-tsconfig-paths` - Path alias resolution

### Frontend Commands

```bash
# Navigate to frontend
cd sleam-crm

# Install dependencies
npm install

# Start development server (runs on port 3000)
npm run dev

# Run tests
npm test

# Build for production
npm run build

# Preview production build
npm run preview
```

### TypeScript Configuration
- Target: ES2022
- Module resolution: bundler
- Strict mode enabled with additional linting rules
- Path aliases: `@/*` maps to `./src/*`
- No emit (Vite handles bundling)

## Backend (crm-server)

### Framework
- Node.js with Express 5.2+
- TypeScript for type safety

### Development Tools
- ts-node for running TypeScript directly
- nodemon for auto-restart during development
- TypeScript 5.9+

### TypeScript Configuration
- Target: ES6
- Module: CommonJS
- Strict mode enabled
- Output directory: `./dist`
- Source directory: `./src`

### Backend Commands

```bash
# Navigate to backend
cd crm-server

# Install dependencies
npm install

# Start development server (runs on port 3500)
# Note: Add dev script with nodemon for auto-reload
npm run dev

# Build TypeScript to JavaScript
# Note: Add build script with tsc
npm run build
```

### Server Configuration
- Default port: 3500 (configurable via PORT environment variable)
- Entry point: `src/index.ts`
