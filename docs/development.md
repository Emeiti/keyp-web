# Development Guide

This document outlines the development workflow, coding standards, and best practices for contributing to the Keyp.fo platform.

## Development Workflow

### Git Workflow

We follow a Git Flow inspired workflow:

1. **Main Branch**: The `main` branch contains the production-ready code.
2. **Development Branch**: The `dev` branch is the integration branch for features.
3. **Feature Branches**: Create branch from `dev` with the naming convention `feature/your-feature-name`.
4. **Bug Fix Branches**: For bug fixes, use the naming convention `fix/bug-description`.
5. **Release Branches**: When preparing a release, create a branch `release/vX.Y.Z`.

### Development Process

1. **Planning**:
   - Review requirements in the issue tracker
   - Discuss implementation approach with the team
   - Break down tasks and create subtasks if necessary

2. **Development**:
   - Create a feature branch from `dev`
   - Implement the feature with appropriate tests
   - Follow the coding standards and best practices
   - Commit regularly with meaningful commit messages

3. **Code Review**:
   - Create a pull request to merge into `dev`
   - Ensure the PR description clearly explains the changes
   - Address review comments and make necessary adjustments
   - Get approval from at least one team member

4. **Integration**:
   - Merge the approved PR into `dev`
   - Verify that the feature works as expected in the development environment

5. **Release**:
   - Create a release branch when ready to deploy
   - Perform final testing and verification
   - Merge the release branch into `main` and tag with version number
   - Deploy to production

## Coding Standards

### JavaScript/TypeScript

- Use TypeScript for all new code
- Follow the [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
- Use meaningful variable and function names
- Add appropriate TypeScript interfaces for all data structures
- Use async/await for asynchronous code
- Avoid any types unless absolutely necessary

```typescript
// Good
interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'store';
}

async function getUserById(id: string): Promise<User | null> {
  try {
    const userDoc = await db.collection('users').doc(id).get();
    if (!userDoc.exists) return null;
    return userDoc.data() as User;
  } catch (error) {
    console.error('Error fetching user:', error);
    return null;
  }
}

// Bad
const getUser = async (id) => {
  const user = await db.collection('users').doc(id).get();
  return user.data();
};
```

### React Components

- Use functional components with hooks
- Follow component naming conventions:
  - Use PascalCase for component names (e.g., `StoreList.tsx`)
  - Use camelCase for instance names (e.g., `const storeList = ...`)
- Separate concerns:
  - UI components should be presentation-focused
  - Container components handle data fetching and state
- Organize props with TypeScript interfaces
- Use destructuring for props
- Use React.memo() for performance optimization when appropriate

```tsx
// Good
interface ButtonProps {
  onClick: () => void;
  children: React.ReactNode;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
}

const Button: React.FC<ButtonProps> = ({
  onClick,
  children,
  variant = 'primary',
  disabled = false,
}) => {
  const classes = `btn btn-${variant} ${disabled ? 'disabled' : ''}`;
  
  return (
    <button className={classes} onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
};

export default React.memo(Button);
```

### CSS and Styling

- Use Tailwind CSS for styling
- Follow the utility-first approach
- Create custom components for repeated patterns
- Use CSS modules for complex components
- Maintain responsive design for all screen sizes
- Follow a mobile-first approach

```tsx
// Using Tailwind utility classes
const Card = ({ title, description }) => (
  <div className="bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow duration-200">
    <h3 className="text-xl font-semibold text-gray-800 mb-2">{title}</h3>
    <p className="text-gray-600">{description}</p>
  </div>
);
```

### Project Structure

Maintain the following project structure:

```
src/
├── app/                # Next.js App Router pages
│   ├── (main)/         # Main website routes
│   ├── admin/          # Admin panel routes
│   └── api/            # API routes
├── components/         # React components
│   ├── common/         # Shared UI components
│   ├── layout/         # Layout components
│   └── [feature]/      # Feature-specific components
├── hooks/              # Custom React hooks
├── lib/                # Utility libraries
│   ├── api/            # API client functions
│   ├── firebase/       # Firebase configuration
│   └── utils/          # Utility functions
├── contexts/           # React context providers
└── types/              # TypeScript type definitions
```

## Testing Guidelines

### Unit Testing

- Write unit tests for utility functions, hooks, and complex logic
- Use Jest for testing framework
- Test both success and failure cases
- Mock external dependencies

```typescript
// Example unit test for a utility function
import { formatCurrency } from '../lib/utils/formatters';

describe('formatCurrency', () => {
  it('formats currency with correct symbol', () => {
    expect(formatCurrency(100, 'DKK')).toBe('100,00 kr.');
    expect(formatCurrency(99.99, 'EUR')).toBe('99,99 €');
  });
  
  it('handles zero values correctly', () => {
    expect(formatCurrency(0, 'DKK')).toBe('0,00 kr.');
  });
  
  it('handles undefined values correctly', () => {
    expect(formatCurrency(undefined, 'DKK')).toBe('');
  });
});
```

### Component Testing

- Test components with React Testing Library
- Focus on testing behavior, not implementation
- Test user interactions and rendering
- Use mock data for API responses

```tsx
// Example component test
import { render, screen, fireEvent } from '@testing-library/react';
import SearchBar from '../components/search/SearchBar';

describe('SearchBar', () => {
  const mockOnSearch = jest.fn();
  
  beforeEach(() => {
    jest.clearAllMocks();
  });
  
  it('renders correctly', () => {
    render(<SearchBar onSearch={mockOnSearch} />);
    expect(screen.getByPlaceholderText('Search stores...')).toBeInTheDocument();
  });
  
  it('calls onSearch when input changes', () => {
    render(<SearchBar onSearch={mockOnSearch} />);
    const input = screen.getByPlaceholderText('Search stores...');
    
    fireEvent.change(input, { target: { value: 'clothing' } });
    
    expect(mockOnSearch).toHaveBeenCalledWith('clothing');
  });
});
```

### Integration Testing

- Test interactions between components
- Test data flow through the application
- Ensure features work end-to-end

### Running Tests

Run tests using the following commands:

```bash
# Run all tests
npm run test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage
```

## Performance Considerations

### Frontend Performance

- Optimize component rendering:
  - Use React.memo() for pure components
  - Use useMemo() and useCallback() to prevent unnecessary re-renders
  - Implement virtualization for long lists
- Optimize images:
  - Use Next.js Image component
  - Choose appropriate formats (WebP, AVIF)
  - Implement responsive images
- Implement code splitting:
  - Use dynamic imports for large components
  - Lazy load non-critical components
- Minimize bundle size:
  - Use tree-shaking
  - Import only what's needed

### Firebase Performance

- Structure your data for efficient reads:
  - Denormalize when necessary
  - Create indexes for common queries
- Batch writes when updating multiple documents
- Use server timestamps for ordering
- Implement pagination for large collections
- Cache frequently accessed data

```typescript
// Good - Batched writes
const batch = db.batch();
stores.forEach(store => {
  const storeRef = db.collection('stores').doc(store.id);
  batch.set(storeRef, store);
});
await batch.commit();

// Bad - Individual writes
for (const store of stores) {
  await db.collection('stores').doc(store.id).set(store);
}
```

## Accessibility (A11y)

- Use semantic HTML elements
- Include proper ARIA attributes when necessary
- Ensure sufficient color contrast
- Implement keyboard navigation
- Make forms accessible
- Test with screen readers

```tsx
// Good - Accessible form
<form onSubmit={handleSubmit}>
  <div className="mb-4">
    <label htmlFor="email" className="block text-gray-700 mb-2">
      Email
    </label>
    <input
      id="email"
      type="email"
      value={email}
      onChange={(e) => setEmail(e.target.value)}
      aria-required="true"
      className="w-full px-3 py-2 border rounded"
    />
  </div>
  <button 
    type="submit" 
    className="bg-blue-500 text-white py-2 px-4 rounded"
    aria-label="Sign In"
  >
    Sign In
  </button>
</form>
```

## Security Best Practices

- Use Firebase Authentication for user management
- Implement proper Firestore security rules
- Validate all input data
- Sanitize user-generated content
- Use environment variables for secrets
- Implement rate limiting for API endpoints
- Follow OWASP security guidelines

```typescript
// Good - Firestore security rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users can only read and write their own data
    match /users/{userId} {
      allow read, update, delete: if request.auth != null && request.auth.uid == userId;
      allow create: if request.auth != null;
    }
    
    // Only store owners can update their store
    match /stores/{storeId} {
      allow read: if true;
      allow create: if request.auth != null && request.auth.token.role == 'admin';
      allow update, delete: if request.auth != null && 
        (request.auth.token.role == 'admin' || 
         resource.data.ownerId == request.auth.uid);
    }
  }
}
```

## Error Handling

- Implement consistent error handling across the application
- Use try/catch blocks for async operations
- Log errors appropriately
- Display user-friendly error messages
- Create a central error handling service

```typescript
// Good - Error handling with user-friendly messages
async function fetchStores() {
  setLoading(true);
  setError(null);
  
  try {
    const stores = await api.getStores();
    setStores(stores);
    return stores;
  } catch (error) {
    const message = error.code === 'NETWORK_ERROR' 
      ? 'Unable to connect. Please check your internet connection.'
      : 'Failed to load stores. Please try again.';
    
    setError(message);
    console.error('Error fetching stores:', error);
    return [];
  } finally {
    setLoading(false);
  }
}
```

## Internationalization (i18n)

- Use a translation library (e.g., next-intl, react-i18next)
- Extract all user-facing strings to translation files
- Support Faroese and English languages
- Consider right-to-left (RTL) layout for the future

```tsx
// Using next-intl for translations
import { useTranslations } from 'next-intl';

function WelcomeMessage() {
  const t = useTranslations('Home');
  
  return (
    <div>
      <h1>{t('welcome')}</h1>
      <p>{t('description')}</p>
    </div>
  );
}
```

## Documentation

- Document all components, hooks, and utilities
- Use TypeScript JSDoc comments for inline documentation
- Keep the documentation up to date
- Create diagrams for complex workflows

```typescript
/**
 * Fetches stores based on the provided filters
 * 
 * @param filters - The filters to apply to the query
 * @param pagination - Pagination options
 * @returns A promise that resolves to the stores and pagination info
 * 
 * @example
 * ```ts
 * const { stores, pagination } = await getStores({ 
 *   category: 'clothing',
 *   featured: true 
 * }, { limit: 10, offset: 0 });
 * ```
 */
async function getStores(
  filters: StoreFilters,
  pagination: PaginationOptions
): Promise<StoreQueryResult> {
  // Implementation...
}
```

## Code Review Checklist

When reviewing code, check for the following:

1. **Functionality**: Does the code work as expected?
2. **Code Quality**: Is the code clean, readable, and maintainable?
3. **Performance**: Are there any performance concerns?
4. **Security**: Are there any security vulnerabilities?
5. **Accessibility**: Does the code follow accessibility best practices?
6. **Tests**: Are there appropriate tests?
7. **Documentation**: Is the code properly documented?
8. **Responsiveness**: Does the UI work on all screen sizes?
9. **Error Handling**: Is error handling implemented correctly?
10. **Type Safety**: Are TypeScript types used correctly?

## Deployment

- Use GitHub Actions for CI/CD
- Implement automated testing in the pipeline
- Set up staging and production environments
- Use feature flags for controlled rollouts
- Monitor deployments and be prepared to roll back if needed

## Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [React Documentation](https://reactjs.org/docs)
- [TypeScript Documentation](https://www.typescriptlang.org/docs)
- [Firebase Documentation](https://firebase.google.com/docs)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [Jest Documentation](https://jestjs.io/docs)
- [React Testing Library Documentation](https://testing-library.com/docs/react-testing-library/intro) 