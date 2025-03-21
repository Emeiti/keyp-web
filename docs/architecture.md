# Architecture

This document outlines the technical architecture of the Keyp.fo platform, describing how the various components work together to deliver the functionality.

## System Architecture Overview

Keyp.fo follows a modern web application architecture that leverages serverless cloud services and integrates with other Keyp services through a shared API:

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│                 │     │                  │     │                 │
│  Client (Web)   │◄────┤  Firebase        │◄────┤  Admin Panel    │
│                 │     │  Backend         │     │                 │
└────────┬────────┘     └────────┬─────────┘     └─────────────────┘
         │                       │
         │                       │
┌────────▼────────┐     ┌────────▼─────────┐     ┌─────────────────┐
│                 │     │                  │     │                 │
│  API Gateway    │◄────┤  Firestore       │◄────┤  Store Panel    │
│  (Functions)    │     │  Database        │     │                 │
└────────┬────────┘     └──────────────────┘     └─────────────────┘
         │
         ├─────────────────────┬─────────────────────┐
         │                     │                     │
┌────────▼────────┐   ┌────────▼────────┐   ┌────────▼────────┐
│                 │   │                 │   │                 │
│  Keyp.fo        │   │  Ynskilisti     │   │  Gavuhugskot    │
│  (Store         │   │  (Wishlist      │   │  (Gift Ideas    │
│  Directory)     │   │  Service)       │   │  Service)       │
└────────┬────────┘   └────────┬────────┘   └────────┬────────┘
         │                     │                     │
         └─────────────────────┼─────────────────────┘
                               │
                     ┌─────────▼─────────┐
                     │                   │
                     │  Storage          │
                     │  (Images)         │
                     └───────────────────┘
```

## Technology Stack

### Frontend

- **Framework**: Next.js 14+ (React)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **State Management**: React Context API
- **Routing**: Next.js App Router
- **Client-Side Fetching**: SWR or React Query

### Backend

- **Database**: Firebase Firestore
- **Authentication**: Firebase Authentication
- **API**: Firebase Cloud Functions
- **Storage**: Firebase Storage
- **Hosting**: Firebase Hosting with custom domains

### Development Tools

- **Version Control**: Git
- **CI/CD**: GitHub Actions
- **Code Quality**: ESLint, Prettier
- **Testing**: Jest, React Testing Library

## Centralized API Architecture

The Keyp ecosystem shares a single centralized API (`api.keyp.fo`) that serves all services:

### Base URL
`https://api.keyp.fo/v1/`

### Core Service Areas

1. **Stores and Products API**
   - Primary service for Keyp.fo store directory
   - Stores information, categories, and product data
   - Provides filtering, search, and management endpoints

2. **Wishlists API**
   - Primary service for Ynskilisti
   - Wishlist management, sharing, and purchase tracking
   - Integrates with store products

3. **Gift Ideas API**
   - Primary service for Gavuhugskot
   - Gift recommendations and filtering
   - Seasonal collections and personalized suggestions

### Shared Authentication

All API endpoints use Firebase Authentication to secure resources and identify users. The authentication system differentiates between user roles:

- **Admin**: Full system access
- **Store Owner**: Can manage store data and products
- **User**: Can create wishlists, mark items as purchased, etc.

### API Rate Limits

- Standard Users: 1000 requests/hour
- Store Owners: 5000 requests/hour
- Admin: Unlimited

### Integrated Image Handling

A shared image handling service manages product images across all services:
- Consistent image processing (WebP format)
- Central storage in Firebase Storage
- Secure access control
- Optimized for performance

## Frontend Architecture

The frontend follows a component-based architecture with clear separation of concerns:

### Directory Structure

```
src/
├── app/                    # Next.js App Router
│   ├── (main)/             # Main website routes
│   ├── admin/              # Admin panel routes
│   ├── store-admin/        # Store owner panel routes
│   ├── gavuhugskot/        # Gift ideas routes
│   ├── api/                # API routes
├── components/             # Reusable UI components
│   ├── common/             # Shared components
│   ├── stores/             # Store-related components
│   ├── categories/         # Category-related components
│   ├── search/             # Search components
│   ├── admin/              # Admin panel components
├── contexts/               # React Context for state management
├── hooks/                  # Custom React hooks
├── lib/                    # Utility libraries
│   ├── firebase/           # Firebase configuration & helpers
│   ├── api/                # API client functions
│   ├── helpers/            # Helper utilities
├── types/                  # TypeScript type definitions
├── styles/                 # CSS and style utilities
├── utils/                  # Utility functions
```

### Component Hierarchy

- **Layout Components**: Define the overall structure (header, footer, etc.)
- **Page Components**: Correspond to routes in the application
- **Feature Components**: Implement specific features (store list, search, etc.)
- **UI Components**: Reusable UI elements (buttons, cards, etc.)

## Backend Architecture

The backend is built on Firebase services, providing a serverless architecture:

### Firebase Firestore

Serves as the main database for the application, with the following collections:

- `stores`: Store information
- `categories`: Category hierarchy
- `products`: Product information for stores
- `users`: User profiles and authentication information
- `subscriptions`: Store subscription details

### Firebase Cloud Functions

Implements server-side logic, including:

- API endpoints for data access
- Subscription management
- Image processing
- Scheduled tasks and maintenance
- Integration with external services

### Firebase Authentication

Manages user authentication with support for:

- Email/password authentication
- Social login (Google, Facebook)
- Role-based access control (admin, store owner, user)

### Firebase Storage

Stores binary assets such as:

- Store logos and banners
- Product images
- Category icons

## Data Flow Architecture

```
┌─────────────┐     ┌───────────────┐     ┌────────────────┐
│             │     │               │     │                │
│  User Input ├────►│ React Component─────►  Data Fetching │
│             │     │               │     │                │
└─────────────┘     └───────────────┘     └────────┬───────┘
                                                   │
                                                   ▼
┌─────────────┐     ┌───────────────┐     ┌────────────────┐
│             │     │               │     │                │
│  UI Update  │◄────┤ State Update  │◄────┤  Firebase SDK  │
│             │     │               │     │                │
└─────────────┘     └───────────────┘     └────────┬───────┘
                                                   │
                                                   ▼
                                          ┌────────────────┐
                                          │                │
                                          │  Firestore DB  │
                                          │                │
                                          └────────────────┘
```

## Authentication & Authorization

### User Types

- **Anonymous Users**: Can browse stores and search
- **Registered Users**: Can save favorites and get personalized recommendations
- **Store Owners**: Can manage their store profiles and products
- **Administrators**: Have full system access and management capabilities

### Authorization Flow

1. User authenticates via Firebase Authentication
2. JWT token is provided to the client
3. Token includes user role and permissions
4. Backend functions verify token and permissions for protected operations

## Integration Architecture

The platform integrates with other Keyp services:

### Ynskilisti.keyp.fo Integration

- Shared authentication system
- API endpoints for wishlist functionality
- Common design elements and navigation

### Gavuhugskot.keyp.fo Integration

- Product database shared with gift ideas service
- Cross-service search capabilities
- Unified store management interface

## Deployment Architecture

### Environments

- **Development**: For active development work
- **Staging**: For testing before production
- **Production**: Live environment for end users

### Deployment Process

1. Code is pushed to the appropriate branch
2. CI/CD pipeline runs tests and builds
3. Firebase deployment to the corresponding environment
4. Post-deployment verification

## Monitoring and Analytics

- **Error Tracking**: Firebase Crashlytics
- **Performance Monitoring**: Firebase Performance
- **Usage Analytics**: Google Analytics
- **Log Management**: Firebase Debug Logs

## Security Architecture

- **Authentication**: Firebase Authentication
- **Data Security**: Firestore Security Rules
- **Storage Security**: Firebase Storage Rules
- **API Security**: Firebase Functions with authentication
- **SSL/TLS**: Enforced for all communication 