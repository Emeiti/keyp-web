# Getting Started

This guide will help you set up the Keyp.fo development environment and get started with contributing to the project.

## Prerequisites

Before you begin, ensure you have the following installed:

- [Node.js](https://nodejs.org/) (v18.0.0 or later)
- [npm](https://www.npmjs.com/) (v9.0.0 or later)
- [Git](https://git-scm.com/)
- A code editor (we recommend [Visual Studio Code](https://code.visualstudio.com/))
- [Firebase CLI](https://firebase.google.com/docs/cli) (for backend development)

## Setting Up the Development Environment

### 1. Clone the Repository

```bash
git clone https://github.com/Emeiti/keyp-web.git
cd keyp-web
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Set Up Environment Variables

Create a `.env.local` file in the root directory with the following content:

```
# Firebase Configuration
NEXT_PUBLIC_FIREBASE_API_KEY=your_api_key
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=your_auth_domain
NEXT_PUBLIC_FIREBASE_PROJECT_ID=your_project_id
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=your_storage_bucket
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=your_messaging_sender_id
NEXT_PUBLIC_FIREBASE_APP_ID=your_app_id

# Environment
NEXT_PUBLIC_BASE_URL=http://localhost:3000
```

Ask a project administrator for the actual values for these environment variables.

### 4. Start the Development Server

```bash
npm run dev
```

This will start the development server at `http://localhost:3000`.

## Project Structure

The project follows a standard Next.js structure with some custom organization:

```
keyp-web/
├── docs/                # Documentation
├── public/              # Static assets
├── src/                 # Source code
│   ├── app/             # Next.js App Router pages
│   ├── components/      # Reusable React components
│   ├── contexts/        # React Context providers
│   ├── hooks/           # Custom React hooks
│   ├── lib/             # Utility libraries
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── .env.local           # Environment variables (not in repo)
├── firebase.json        # Firebase configuration
├── firestore.rules      # Firestore security rules
├── package.json         # Project dependencies
└── tsconfig.json        # TypeScript configuration
```

## Development Workflow

### Feature Development Process

1. **Create a Branch**: Always create a new branch for each feature or fix
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Develop and Test**: Make your changes and test thoroughly

3. **Commit Changes**: Write clear, concise commit messages
   ```bash
   git add .
   git commit -m "Add feature: brief description of changes"
   ```

4. **Push Changes**: Push your branch to GitHub
   ```bash
   git push origin feature/your-feature-name
   ```

5. **Create a Pull Request**: Open a PR against the main branch for review

### Code Style and Conventions

We follow these code standards:

- TypeScript for type safety
- Prettier for code formatting
- ESLint for code quality
- Component-based architecture
- CSS modules with Tailwind CSS

The project includes configurations for these tools. Run `npm run lint` to check for code style issues.

## Firebase Development

For backend development, you'll need to set up Firebase locally:

### 1. Install the Firebase Emulator Suite

```bash
npm install -g firebase-tools
firebase login
```

### 2. Start the Firebase Emulators

```bash
firebase emulators:start
```

This will start local emulators for Firestore, Authentication, and Cloud Functions.

### 3. Connect to the Emulators

Update the Firebase configuration in your app to connect to the local emulators:

```typescript
// src/lib/firebase/config.ts
if (process.env.NODE_ENV === 'development') {
  connectFirestoreEmulator(db, 'localhost', 8080);
  connectAuthEmulator(auth, 'http://localhost:9099');
  connectFunctionsEmulator(functions, 'localhost', 5001);
}
```

## Common Tasks

### Creating a New Component

1. Create a new file in the appropriate subdirectory of `src/components/`
2. Follow the component template structure:

```tsx
// src/components/common/Button.tsx
import React from 'react';

interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
}

export default function Button({
  children,
  onClick,
  variant = 'primary',
  size = 'md',
  disabled = false,
}: ButtonProps) {
  const baseClasses = 'rounded font-medium transition-colors focus:outline-none focus:ring-2';
  
  const variantClasses = {
    primary: 'bg-blue-500 text-white hover:bg-blue-600 focus:ring-blue-300',
    secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300 focus:ring-gray-200',
    outline: 'bg-transparent border border-gray-300 text-gray-700 hover:bg-gray-50 focus:ring-gray-200',
  };
  
  const sizeClasses = {
    sm: 'px-3 py-1 text-sm',
    md: 'px-4 py-2',
    lg: 'px-6 py-3 text-lg',
  };
  
  const classes = `
    ${baseClasses}
    ${variantClasses[variant]}
    ${sizeClasses[size]}
    ${disabled ? 'opacity-50 cursor-not-allowed' : ''}
  `;
  
  return (
    <button 
      className={classes} 
      onClick={onClick} 
      disabled={disabled}
    >
      {children}
    </button>
  );
}
```

### Adding a New Page

1. Create a new directory or file in `src/app/` following the Next.js App Router conventions
2. Create a page component:

```tsx
// src/app/stores/[id]/page.tsx
import { Metadata } from 'next';
import { getStoreById } from '@/lib/api/stores';
import StoreDetails from '@/components/stores/StoreDetails';

export async function generateMetadata({ params }: { params: { id: string } }): Promise<Metadata> {
  const store = await getStoreById(params.id);
  
  return {
    title: `${store.name} | Keyp.fo`,
    description: store.metadata.description || `Discover ${store.name} on Keyp.fo`,
  };
}

export default async function StorePage({ params }: { params: { id: string } }) {
  const store = await getStoreById(params.id);
  
  return <StoreDetails store={store} />;
}
```

### Working with Firebase

#### Reading Data

```typescript
// src/lib/api/stores.ts
import { db } from '@/lib/firebase/config';
import { collection, getDocs, query, where, orderBy, limit } from 'firebase/firestore';
import { Store } from '@/types/store';

export async function getFeaturedStores(count = 4): Promise<Store[]> {
  const storesRef = collection(db, 'stores');
  const q = query(
    storesRef,
    where('active', '==', true),
    where('featured', '==', true),
    orderBy('updatedAt', 'desc'),
    limit(count)
  );
  
  const snapshot = await getDocs(q);
  return snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() } as Store));
}
```

#### Writing Data

```typescript
// src/lib/api/stores.ts
import { db } from '@/lib/firebase/config';
import { doc, setDoc, serverTimestamp } from 'firebase/firestore';
import { Store } from '@/types/store';

export async function updateStore(storeId: string, data: Partial<Store>): Promise<void> {
  const storeRef = doc(db, 'stores', storeId);
  
  await setDoc(storeRef, {
    ...data,
    updatedAt: serverTimestamp(),
  }, { merge: true });
}
```

## Testing

### Running Tests

```bash
# Run all tests
npm run test

# Run tests in watch mode during development
npm run test:watch
```

### Writing Tests

We use Jest and React Testing Library for testing. Place test files next to the files they test with a `.test.tsx` extension.

```tsx
// src/components/common/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import Button from './Button';

describe('Button', () => {
  it('renders correctly', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });
  
  it('calls onClick when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>);
    expect(screen.getByText('Click me')).toBeDisabled();
  });
});
```

## Deployment

The project is automatically deployed when changes are merged to the main branch. However, you can also deploy manually:

```bash
# Build the project
npm run build

# Deploy to Firebase hosting
firebase deploy
```

## Troubleshooting

### Common Issues

1. **Firebase Authentication Issues**
   - Check if the Firebase emulator is running
   - Verify that your `.env.local` file has the correct Firebase configuration

2. **Next.js Build Errors**
   - Run `npm run lint` to check for code issues
   - Check for type errors with `npm run type-check`

3. **Layout Shifts or CSS Issues**
   - Make sure you're following the Tailwind CSS conventions
   - Check for missing responsive design considerations

### Getting Help

If you're stuck, you can:

1. Check the [Next.js documentation](https://nextjs.org/docs)
2. Check the [Firebase documentation](https://firebase.google.com/docs)
3. Open an issue on the GitHub repository
4. Contact the project maintainers

## Next Steps

Now that you have the development environment set up, you can:

1. Explore the codebase to understand the existing functionality
2. Check the open issues on GitHub for tasks to work on
3. Read the [Architecture](./architecture.md) and [Database Schema](./database-schema.md) documentation to understand the system design 