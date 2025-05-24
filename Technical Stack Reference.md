# VF Communicator - Technical Stack Reference

## Overview
This document is the authoritative technical reference for VF Communicator. All development should follow these specs for consistency and maintainability.

## Core Technologies

### 1. Frontend Framework
- **Next.js 14+ (App Router)**
  - App Router (`/app` directory) by default  
  - Server Components and explicit Client Components  
  - Built-in loading states & error boundaries  
  - JavaScript (JSX) only—no TypeScript

### 2. Bundler & Dev Tooling
- **Vite (>= 5.0)**
  - Instant HMR & lightning-fast builds  
  - ESBuild under the hood  
  - Use `@vitejs/plugin-react` for proper integration  
  - `.jsx` file extensions throughout

### 3. Styling
- **Tailwind CSS v4** (strict mode)
  - Enforce JIT and strict class-name checking  
  - Review: docs/tailwindV4Installation.md
  - Utility classes only—no custom CSS outside config  
  - Use `clsx` for dynamic class composition

- **Headless UI v2**
  - Unstyled, accessible components  
  - Style exclusively with Tailwind utilities

### 4. State & Real-time Data
- **react-firebase-hooks v5**
  - `useAuthState()` for auth  
  - `useCollection()` / `useDocument()` for Firestore

### 5. Forms & Validation
- **React Hook Form v7** + **Yup**
  - Fast form state  
  - Runtime schema validation via `@hookform/resolvers/yup`

### 6. Tables & Data Display
- **@tanstack/react-table v8**
  - Filtering, sorting, pagination  
  - Virtualization for large lists

## Backend Services

### Firebase
- **Auth** – custom claims for roles  
- **Firestore** – real-time DB, offline support  
- **Cloud Functions** – audit logs, notifications, batch transitions  
- **Hosting** – CDN + SSL

## Additional Libraries

- **react-hot-toast** – toasts  
- **react-loading-skeleton** – placeholders  
- **date-fns**, **nanoid**, **firebase-admin**

## Quality & CI
- **ESLint**, **Prettier**  
- **Husky** + **lint-staged**  


## Project Structure

```
vf-communicator/
├── app/                          # Next.js App Router
│   ├── (auth)/                   # Auth-required routes
│   │   ├── dashboard/
│   │   │   ├── page.jsx         # Role-based dashboard
│   │   │   └── layout.jsx       # Dashboard layout
│   │   ├── batches/
│   │   │   ├── page.jsx         # Active batches list
│   │   │   ├── [id]/
│   │   │   │   └── page.jsx    # Batch details
│   │   │   └── new/
│   │   │       └── page.jsx    # Create new batch
│   │   ├── qa/
│   │   │   ├── page.jsx         # QA queue
│   │   │   └── layout.jsx       # QA-specific layout
│   │   ├── supervisor/
│   │   │   ├── approvals/       # Approval queue  
│   │   │   └── reports/         # Shift reports
│   │   └── admin/
│   │       ├── users/           # User management
│   │       └── formulas/        # Formula management
│   ├── api/                      # API Routes
│   │   ├── auth/
│   │   │   └── [...nextauth]/
│   │   └── export/
│   │       └── route.js         # PDF export endpoint
│   ├── layout.jsx               # Root layout
│   ├── page.jsx                 # Landing/login page
│   └── globals.css              # Tailwind imports
│
├── components/                   # Reusable components
│   ├── ui/                      # Base UI components
│   │   ├── button.jsx
│   │   ├── dialog.jsx
│   │   ├── menu.jsx
│   │   ├── table.jsx
│   │   └── toast.jsx
│   ├── forms/                   # Form components
│   │   ├── batch-form.jsx
│   │   ├── qa-review-form.jsx
│   │   └── user-form.jsx
│   ├── tables/                  # Table components
│   │   ├── batches-table.jsx
│   │   ├── qa-queue-table.jsx
│   │   └── audit-log-table.jsx
│   ├── layout/                  # Layout components
│   │   ├── header.jsx
│   │   ├── sidebar.jsx
│   │   └── mobile-nav.jsx
│   └── shared/                  # Shared components
│       ├── loading-skeleton.jsx
│       ├── error-boundary.jsx
│       └── role-guard.jsx
│
├── lib/                         # Core utilities
│   ├── firebase/
│   │   ├── config.js           # Firebase initialization
│   │   ├── auth.js             # Auth utilities
│   │   ├── firestore.js        # Firestore helpers
│   │   └── converters.js       # Type converters
│   ├── schemas/                 # Zod schemas
│   │   ├── batch.schema.js
│   │   ├── user.schema.js
│   │   └── formula.schema.js
│   ├── hooks/                   # Custom React hooks
│   │   ├── useAuth.js
│   │   ├── useBatches.js
│   │   ├── useNotifications.js
│   │   └── useRole.js
│   ├── utils/                   # Utility functions
│   │   ├── constants.js        # App constants
│   │   ├── helpers.js          # Helper functions
│   │   └── export.js           # PDF export logic
│   └── types/                   # TypeScript types
│       ├── index.js            # Main type exports
│       ├── batch.types.js
│       └── user.types.js
│
├── firebase/                    # Firebase configuration
│   ├── functions/              # Cloud Functions
│   │   ├── src/
│   │   │   ├── index.js
│   │   │   ├── audit.js        # Audit triggers
│   │   │   ├── notifications.js # Notification system
│   │   │   └── claims.js       # Custom claims
│   │   ├── package.json
│   │   └── tsconfig.json
│   ├── firestore.rules         # Security rules
│   └── firestore.indexes.json  # Composite indexes
│
├── public/                      # Static assets
│   ├── icons/
│   └── images/
│
├── tests/                       # Test files
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
├── .env.local                   # Local environment variables
├── .env.production             # Production environment variables
├── .gitignore
├── next.config.js              # Next.js configuration
├── tailwind.config.js          # Tailwind configuration
├── tsconfig.json               # TypeScript configuration
├── package.json
└── README.md
```

## Implementation Guidelines

### 1. Firebase Setup
```typescript
// lib/firebase/config.js
import { initializeApp, getApps } from 'firebase/app';
import { getAuth } from 'firebase/auth';
import { getFirestore } from 'firebase/firestore';

const firebaseConfig = {
  // Your config
};

// Initialize Firebase
const app = getApps().length ? getApps()[0] : initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const db = getFirestore(app);
```

### 2. Type Safety
Define all data models with Zod schemas and derive TypeScript types:

```typescript
// lib/schemas/batch.schema.js
import { z } from 'zod';

export const batchSchema = z.object({
  id: z.string(),
  deckNumber: z.string(),
  formulaNumber: z.string(),
  batchNumber: z.string(),
  lab: z.enum(['front', 'back']),
  status: z.enum([
    'mixing',
    'qAwaiting',
    'testing',
    'approved',
    'denied',
    'resampleAwaiting',
    'rejected',
    'archived'
  ]),
  // ... rest of schema
});

export type Batch = z.infer<typeof batchSchema>;
```

### 3. Real-time Hooks
Create custom hooks for Firebase operations:

```typescript
// lib/hooks/useBatches.js
import { useCollection } from 'react-firebase-hooks/firestore';
import { collection, query, where, orderBy } from 'firebase/firestore';
import { db } from '@/lib/firebase/config';

export function useActiveBatches(deckFilter?: string) {
  const constraints = [
    where('status', 'not-in', ['archived', 'rejected']),
    orderBy('createdAt', 'desc')
  ];
  
  if (deckFilter) {
    constraints.push(where('deckNumber', '==', deckFilter));
  }
  
  const q = query(collection(db, 'batches'), ...constraints);
  return useCollection(q);
}
```

### 4. Component Organization
- Keep components small and focused
- Use TypeScript for all components
- Implement proper loading and error states
- Follow accessibility best practices

### 5. Security Rules
Implement strict Firestore security rules based on roles:

```javascript
// firebase/firestore.rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function hasRole(role) {
      return request.auth != null && 
        request.auth.token.role == role;
    }
    
    function isTrainedOnDeck(deckNumber) {
      return request.auth != null &&
        deckNumber in request.auth.token.trainedDecks;
    }
    
    match /batches/{batchId} {
      allow read: if request.auth != null;
      allow create: if hasRole('processor') && 
        isTrainedOnDeck(request.resource.data.deckNumber);
      allow update: if hasRole('processor') && 
        resource.data.activeProcessor == request.auth.uid;
      // ... more rules
    }
  }
}
```

## Performance Considerations

1. **Use React.memo** for expensive components
2. **Implement virtual scrolling** for large lists
3. **Enable Firestore offline persistence**
4. **Lazy load route components**
5. **Optimize images with Next.js Image component**
6. **Use Firebase compound queries** to minimize data transfer

## Development Workflow

1. **Branch Strategy**: Use feature branches off `develop`
2. **Commit Convention**: Use conventional commits
3. **Code Review**: All PRs require review
4. **Testing**: Unit tests for utilities, integration tests for workflows
5. **Deployment**: Auto-deploy to staging on merge to `develop`