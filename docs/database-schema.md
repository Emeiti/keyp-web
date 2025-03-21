# Database Schema

This document outlines the database schema for the Keyp.fo platform. We use Firebase Firestore as our primary database, which is a NoSQL document database. The schema is designed to support the platform's core functionality while maintaining good performance characteristics.

## Database Collections

### stores

The central collection for store information.

```typescript
interface Store {
  id: string;                  // Unique identifier
  name: string;                // Store name
  slug: string;                // URL-friendly name
  url: string;                 // Website URL
  active: boolean;             // Whether the store is active
  featured: boolean;           // Whether the store is featured (for promoted stores)
  createdAt: Timestamp;        // Creation timestamp
  updatedAt: Timestamp;        // Last update timestamp
  
  metadata: {
    logo: string;              // URL to store logo
    banner: string;            // URL to banner image (for featured stores)
    description: string;       // Store description
    hasWebshop: boolean;       // Whether the store has an online shop
    hasPhysicalShop: boolean;  // Whether the store has a physical location
    categories: string[];      // Array of category IDs
    tags: string[];            // Additional tags for search
    brands: string[];          // Brands carried by the store
    openingHours: {
      [day: string]: {         // day: 'monday', 'tuesday', etc.
        open: string;          // Opening time (HH:MM)
        close: string;         // Closing time (HH:MM)
      }
    };
  };
  
  contact: {
    email: string;             // Contact email
    phone: string;             // Contact phone
    address: {
      street: string;          // Street address
      city: string;            // City
      postalCode: string;      // Postal code
      coordinates: {           // Geographic coordinates
        latitude: number;
        longitude: number;
      }
    };
    social: {                  // Social media links
      facebook?: string;
      instagram?: string;
      twitter?: string;
      linkedin?: string;
    };
  };
  
  subscription: {
    plan: 'free' | 'basic' | 'premium';  // Subscription level
    status: 'active' | 'expired';        // Subscription status
    features: {                          // Enabled features
      wishlist: boolean;
      giftIdeas: boolean;
      featuredListing: boolean;
      categoryPromotions: boolean;
    };
    limits: {                            // Plan-specific limits
      categoriesCount: number;
      productsCount: number;
      giftIdeasCount: number;
    };
    expiresAt: Timestamp;                // Subscription expiration date
  };
  
  stats: {                               // Analytics data
    views: number;                       // Profile views
    clicks: number;                      // Website clicks
  };
}
```

### categories

Hierarchical collection of store categories.

```typescript
interface Category {
  id: string;                // Unique identifier
  name: string;              // Category name
  slug: string;              // URL-friendly name
  description: string;       // Category description
  icon: string;              // URL to category icon
  image: string;             // URL to category image
  parentId: string | null;   // Parent category ID (null for top-level)
  order: number;             // Display order
  featured: boolean;         // Whether this is a featured category
  createdAt: Timestamp;
  updatedAt: Timestamp;
  
  metadata: {
    forTarget: 'him' | 'her' | 'kids' | 'home' | 'all';  // Target audience
    tags: string[];          // Additional tags for search
  };
}
```

### products

Products offered by stores, including gift ideas.

```typescript
interface Product {
  id: string;                // Unique identifier
  storeId: string;           // Reference to parent store
  name: string;              // Product name
  description: string;       // Product description
  price: number;             // Price
  currency: string;          // Currency code (e.g., 'DKK')
  images: string[];          // Array of image URLs
  url: string;               // Link to product page
  active: boolean;           // Whether the product is active
  createdAt: Timestamp;
  updatedAt: Timestamp;
  
  metadata: {
    categories: string[];    // Array of category IDs
    tags: string[];          // Additional tags for search
    brand: string;           // Product brand
    isGiftIdea: boolean;     // Whether this is a gift idea
    giftIdeaMetadata: {      // Only applicable if isGiftIdea is true
      targetGender: 'male' | 'female' | 'unisex';
      targetAgeMin: number;  // Minimum age
      targetAgeMax: number;  // Maximum age
      occasion: string[];    // E.g., 'birthday', 'christmas'
      priceRange: 'budget' | 'mid' | 'premium';
    };
  };
  
  stats: {                   // Analytics data
    views: number;           // Product views
    clicks: number;          // Product clicks
  };
}
```

### users

User profiles and authentication information.

```typescript
interface User {
  id: string;                // Unique identifier (Firebase Auth UID)
  email: string;             // User email
  displayName: string;       // Display name
  role: 'admin' | 'store' | 'user';  // User role
  storeId?: string;          // Associated store ID (for store owners)
  createdAt: Timestamp;
  updatedAt: Timestamp;
  lastLogin: Timestamp;
  
  metadata: {
    phone?: string;          // Phone number
    address?: {              // User address
      street: string;
      city: string;
      postalCode: string;
    };
  };
  
  preferences: {
    notifications: boolean;  // Email notification preferences
    language: 'fo' | 'en';   // Preferred language
  };
}
```

### subscriptions

Store subscription details.

```typescript
interface Subscription {
  id: string;                // Unique identifier
  storeId: string;           // Associated store ID
  plan: 'free' | 'basic' | 'premium';  // Subscription plan
  status: 'active' | 'trial' | 'expired' | 'cancelled';
  startDate: Timestamp;      // Subscription start date
  endDate: Timestamp;        // Subscription end date
  autoRenew: boolean;        // Whether to auto-renew
  
  paymentMethod?: {          // Payment information
    type: 'card' | 'invoice' | 'other';
    lastFour?: string;       // Last four digits of card
    expiryDate?: string;     // Card expiry date
  };
  
  billingHistory: {          // Billing records
    id: string;              // Invoice ID
    date: Timestamp;         // Billing date
    amount: number;          // Amount
    currency: string;        // Currency code
    status: 'paid' | 'pending' | 'failed';
  }[];
}
```

### featured

Promotional content for the homepage and category pages.

```typescript
interface Featured {
  id: string;                // Unique identifier
  type: 'store' | 'category' | 'banner';  // Type of featured content
  targetId: string;          // ID of the featured item (store/category)
  position: 'homepage' | 'category' | 'search';  // Where it's displayed
  categoryId?: string;       // Only for category-specific features
  startDate: Timestamp;      // Start of promotional period
  endDate: Timestamp;        // End of promotional period
  active: boolean;           // Whether the promotion is active
  
  content: {
    title: string;           // Promotional title
    description: string;     // Promotional description
    imageUrl: string;        // Promotional image
    buttonText?: string;     // Call-to-action text
    buttonUrl?: string;      // Call-to-action URL
  };
  
  stats: {                   // Analytics data
    impressions: number;     // Times shown
    clicks: number;          // Times clicked
  };
}
```

## Relationships

The database follows these key relationships:

1. **Stores to Categories**: Many-to-many relationship via the `categories` array in the store metadata
2. **Stores to Products**: One-to-many relationship via the `storeId` field in products
3. **Products to Categories**: Many-to-many relationship via the `categories` array in product metadata
4. **Users to Stores**: One-to-one relationship for store owners via the `storeId` field in user profiles
5. **Stores to Subscriptions**: One-to-one relationship via the `storeId` field in subscriptions

## Indexing Strategy

For optimal query performance, we maintain the following indexes:

1. **Stores collection**:
   - `active` ASC, `featured` DESC (for finding active and featured stores)
   - `metadata.categories` ASC, `active` ASC (for category-based queries)
   - `metadata.hasWebshop` ASC, `active` ASC (for filtering online shops)
   - `metadata.hasPhysicalShop` ASC, `active` ASC (for filtering physical shops)

2. **Products collection**:
   - `storeId` ASC, `active` ASC (for store product listings)
   - `metadata.isGiftIdea` ASC, `active` ASC (for gift idea listings)
   - `metadata.categories` ASC, `active` ASC (for category-based product queries)

3. **Users collection**:
   - `role` ASC, `createdAt` DESC (for admin queries)
   - `storeId` ASC (for finding store owners)

## Data Consistency

Since Firestore is a NoSQL database without transactions across collections, we maintain data consistency through careful application logic and cloud functions:

1. **Store Updates**: When a store is updated, check if category relationships need to be updated
2. **User Deletion**: When a user is deleted, check if associated store ownership needs to be transferred
3. **Subscription Changes**: When a subscription changes, update the store's subscription status

## Data Migration Strategy

As the database schema evolves, we'll use Firebase Cloud Functions to handle migrations:

1. **Add New Fields**: Add new fields with default values to existing documents
2. **Rename Fields**: Create new fields and copy data from old fields, then remove old fields
3. **Change Structure**: Use batch operations to update document structure while maintaining atomicity

## Backup and Recovery

The database is backed up regularly using Firestore's export functionality:

1. **Daily Exports**: Full database exports stored in Google Cloud Storage
2. **Retention Policy**: Backups are retained for 30 days
3. **Recovery Procedure**: In case of data corruption, the latest backup can be imported to restore the database 