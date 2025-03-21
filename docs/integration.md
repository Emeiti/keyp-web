# Integration with Other Keyp Services

This document outlines how to integrate the Keyp.fo platform with other Keyp services, specifically Ynskilisti (wishlist service) and Gavuhugskot (gift ideas service).

## Overview

The Keyp ecosystem consists of three main services:

1. **Keyp.fo** - The main store directory platform
2. **Ynskilisti.keyp.fo** - Wishlist service
3. **Gavuhugskot.keyp.fo** - Gift ideas service

These services are designed to work together seamlessly, providing a unified user experience while maintaining separation of concerns.

## Common Integration Components

### Shared Authentication

All three services share a common authentication system based on Firebase Authentication. This allows users to sign in once and access all services without having to authenticate separately.

```typescript
// Example of setting up shared authentication
import { getAuth, onAuthStateChanged } from 'firebase/auth';
import { firebaseApp } from '@/lib/firebase/config';

const auth = getAuth(firebaseApp);

export function useAuthState() {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, (user) => {
      setUser(user);
    });
    
    return () => unsubscribe();
  }, []);
  
  return user;
}
```

### Shared Image Handler Service

All services use a common Image Handler service for consistency in image processing, storage, and retrieval. This ensures that product images are consistently formatted and accessible across all Keyp services.

```typescript
// Example of using the shared Image Handler service
import { ImageHandler } from '@keyp/shared/image-handler';

async function uploadProductImage(imageData: string, productName: string, storeName: string) {
  const imageHandler = new ImageHandler();
  
  // Generate a URL-safe product ID from the name
  const productId = btoa(productName).replace(/[^a-zA-Z0-9]/g, '');
  
  try {
    // Upload to the centralized image service
    const imageUrl = await imageHandler.uploadToKeypApi({
      imageData,     // Base64 WebP image data
      storeName,     // Store name (must be consistent)
      productId
    });
    
    return imageUrl;
  } catch (error) {
    console.error('Failed to upload image:', error);
    throw error;
  }
}
```

**Key requirements for Image Handler integration:**
- Images must be converted to WebP format before uploading
- Images should be resized to appropriate dimensions (max 800px width)
- Image data must be base64 encoded
- User must be authenticated with Firebase Auth
- Store names must be consistent across all services

### Shared Design System

To maintain visual consistency across all services, we use a shared design system with common components, colors, and typography.

Key elements to maintain across services:

- Color palette
- Typography
- Component styling
- Layout patterns
- Navigation elements

### Cross-Service Navigation

Each service includes a common header with navigation links to other services:

```tsx
// Example of cross-service navigation component
const ServiceNav = () => (
  <nav className="flex space-x-4">
    <a href="https://keyp.fo" className="text-blue-600 hover:text-blue-800">
      Keyp.fo
    </a>
    <a href="https://ynskilisti.keyp.fo" className="text-blue-600 hover:text-blue-800">
      Ynskilisti
    </a>
    <a href="https://gavuhugskot.keyp.fo" className="text-blue-600 hover:text-blue-800">
      Gávuhugskot
    </a>
  </nav>
);
```

## Integration with Ynskilisti (Wishlist Service)

### Overview

Ynskilisti allows users to create and manage wishlists of products from various stores. Integration with Keyp.fo enables:

1. Adding store products to wishlists
2. Displaying wishlist items with store information
3. Sharing wishlists with others

### API Integration

#### Adding Products to Wishlists

```typescript
// Example function to add a product to a wishlist
async function addToWishlist(userId: string, productData: Product) {
  try {
    const response = await fetch('https://api.keyp.fo/v1/wishlistItems?wishlistId=' + wishlistId, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${await getAuthToken()}`
      },
      body: JSON.stringify({
        name: productData.name,
        description: productData.description,
        priority: 2, // Default priority
        price: productData.price,
        url: productData.url,
        note: "" // Optional note
      })
    });
    
    const result = await response.json();
    return result.success;
  } catch (error) {
    console.error('Error adding to wishlist:', error);
    return false;
  }
}
```

#### Retrieving Wishlist Data

```typescript
// Example function to get items from a wishlist
async function getWishlistItems(wishlistId: string) {
  try {
    const response = await fetch(`https://api.keyp.fo/v1/wishlistItems?wishlistId=${wishlistId}`, {
      headers: {
        'Authorization': `Bearer ${await getAuthToken()}`
      }
    });
    
    const result = await response.json();
    return result.data;
  } catch (error) {
    console.error('Error fetching wishlist items:', error);
    return [];
  }
}
```

### UI Integration

#### Wishlist Button Component

Add a "Add to Wishlist" button to product cards or pages:

```tsx
// Wishlist button component
interface WishlistButtonProps {
  product: Product;
  userId: string;
}

const WishlistButton: React.FC<WishlistButtonProps> = ({ product, userId }) => {
  const [isAdding, setIsAdding] = useState(false);
  const [isAdded, setIsAdded] = useState(false);
  
  const handleAddToWishlist = async () => {
    if (!userId) {
      // Redirect to login
      window.location.href = '/login?redirect=' + encodeURIComponent(window.location.pathname);
      return;
    }
    
    setIsAdding(true);
    
    try {
      const success = await addToWishlist(userId, product);
      setIsAdded(success);
    } catch (error) {
      console.error('Error adding to wishlist:', error);
    } finally {
      setIsAdding(false);
    }
  };
  
  return (
    <button
      onClick={handleAddToWishlist}
      disabled={isAdding || isAdded}
      className={`px-4 py-2 rounded transition ${
        isAdded 
          ? 'bg-green-100 text-green-800' 
          : 'bg-blue-100 text-blue-800 hover:bg-blue-200'
      }`}
    >
      {isAdding ? 'Adding...' : isAdded ? 'Added to Wishlist' : 'Add to Wishlist'}
    </button>
  );
};
```

## Integration with Gavuhugskot (Gift Ideas Service)

### Overview

Gavuhugskot helps users find gift ideas for specific target demographics. Integration with Keyp.fo enables:

1. Featuring store products as gift ideas
2. Filtering gift ideas by various criteria
3. Directing users to stores for purchasing

### Data Integration

#### Product Data Structure

When adding products to Keyp.fo that should also be featured as gift ideas, include the gift idea metadata:

```typescript
// Example product with gift idea metadata
const product = {
  id: 'product123',
  name: 'Bluetooth Speaker',
  description: 'Portable wireless speaker with amazing sound quality',
  price: 499,
  currency: 'DKK',
  images: ['https://example.com/images/speaker.jpg'],
  storeId: 'store456',
  storeName: 'Electronics Store',
  url: 'https://electronicsstore.fo/products/bluetooth-speaker',
  metadata: {
    categories: ['electronics', 'audio'],
    brand: 'SoundMaster',
    isGiftIdea: true,
    giftIdeaMetadata: {
      targetGender: 'unisex',
      targetAgeMin: 12,
      targetAgeMax: 99,
      occasion: ['birthday', 'christmas'],
      priceRange: 'mid'
    }
  }
};
```

### API Integration

#### Fetching Gift Ideas

```typescript
// Example function to fetch gift ideas
async function getGiftIdeas(filters: GiftIdeaFilters) {
  try {
    const queryParams = new URLSearchParams({
      gender: filters.gender,
      minAge: filters.minAge.toString(),
      maxAge: filters.maxAge.toString(),
      occasion: filters.occasion,
      priceRange: filters.priceRange,
      limit: '20'
    }).toString();
    
    const response = await fetch(`https://api.keyp.fo/v1/gift-ideas/search?${queryParams}`);
    const result = await response.json();
    
    return result.data.giftIdeas;
  } catch (error) {
    console.error('Error fetching gift ideas:', error);
    return [];
  }
}
```

### UI Integration

#### Gift Idea Search Widget

Embed a gift idea search widget in relevant parts of the Keyp.fo website:

```tsx
// Gift Idea Search Widget
const GiftIdeaWidget: React.FC = () => {
  const [gender, setGender] = useState('unisex');
  const [ageRange, setAgeRange] = useState([5, 15]);
  const [occasion, setOccasion] = useState('birthday');
  
  const handleSearch = () => {
    // Redirect to the gift ideas service with search parameters
    const queryParams = new URLSearchParams({
      gender,
      minAge: ageRange[0].toString(),
      maxAge: ageRange[1].toString(),
      occasion
    }).toString();
    
    window.location.href = `https://gavuhugskot.keyp.fo/search?${queryParams}`;
  };
  
  return (
    <div className="bg-white p-6 rounded-lg shadow-md">
      <h3 className="text-xl font-semibold mb-4">Find the Perfect Gift</h3>
      
      <div className="space-y-4">
        <div>
          <label className="block text-gray-700 mb-2">For who?</label>
          <select 
            value={gender} 
            onChange={(e) => setGender(e.target.value)}
            className="w-full border rounded px-3 py-2"
          >
            <option value="male">Male</option>
            <option value="female">Female</option>
            <option value="unisex">Anyone</option>
          </select>
        </div>
        
        <div>
          <label className="block text-gray-700 mb-2">Age Range</label>
          <div className="flex items-center space-x-2">
            <input 
              type="number" 
              min="0" 
              max="99"
              value={ageRange[0]} 
              onChange={(e) => setAgeRange([parseInt(e.target.value), ageRange[1]])}
              className="w-20 border rounded px-3 py-2"
            />
            <span>to</span>
            <input 
              type="number" 
              min="0" 
              max="99"
              value={ageRange[1]} 
              onChange={(e) => setAgeRange([ageRange[0], parseInt(e.target.value)])}
              className="w-20 border rounded px-3 py-2"
            />
          </div>
        </div>
        
        <div>
          <label className="block text-gray-700 mb-2">Occasion</label>
          <select 
            value={occasion} 
            onChange={(e) => setOccasion(e.target.value)}
            className="w-full border rounded px-3 py-2"
          >
            <option value="birthday">Birthday</option>
            <option value="christmas">Christmas</option>
            <option value="wedding">Wedding</option>
            <option value="graduation">Graduation</option>
          </select>
        </div>
        
        <button 
          onClick={handleSearch}
          className="w-full bg-blue-500 text-white py-2 px-4 rounded hover:bg-blue-600 transition"
        >
          Find Gift Ideas
        </button>
      </div>
    </div>
  );
};
```

## Store Admin Integration

For store owners, provide a unified dashboard to manage their presence across all Keyp services:

### Combined Management Dashboard

Create a dashboard that allows store owners to:

1. Update their store details on Keyp.fo
2. Manage which products are featured as gift ideas on Gavuhugskot
3. View how many times their products have been added to wishlists on Ynskilisti

```tsx
// Example store dashboard section for cross-service integration
const StoreServiceIntegration: React.FC<{ storeId: string }> = ({ storeId }) => {
  const [giftIdeasCount, setGiftIdeasCount] = useState(0);
  const [wishlistsCount, setWishlistsCount] = useState(0);
  
  useEffect(() => {
    // Fetch integration stats
    async function fetchStats() {
      try {
        const [giftIdeasStats, wishlistsStats] = await Promise.all([
          fetch(`https://api.keyp.fo/v1/stores/${storeId}/stats/gift-ideas`).then(r => r.json()),
          fetch(`https://api.keyp.fo/v1/stores/${storeId}/stats/wishlists`).then(r => r.json())
        ]);
        
        setGiftIdeasCount(giftIdeasStats.data.count);
        setWishlistsCount(wishlistsStats.data.count);
      } catch (error) {
        console.error('Error fetching integration stats:', error);
      }
    }
    
    fetchStats();
  }, [storeId]);
  
  return (
    <div className="bg-white p-6 rounded-lg shadow-md">
      <h2 className="text-xl font-semibold mb-4">Service Integration</h2>
      
      <div className="grid grid-cols-2 gap-4">
        <div className="border rounded-lg p-4">
          <h3 className="font-medium text-gray-700">Gávuhugskot</h3>
          <p className="text-2xl font-bold">{giftIdeasCount}</p>
          <p className="text-sm text-gray-500">Active gift ideas</p>
          <a 
            href={`https://admin.keyp.fo/store/${storeId}/gift-ideas`}
            className="text-blue-600 hover:text-blue-800 text-sm mt-2 inline-block"
          >
            Manage Gift Ideas →
          </a>
        </div>
        
        <div className="border rounded-lg p-4">
          <h3 className="font-medium text-gray-700">Ynskilisti</h3>
          <p className="text-2xl font-bold">{wishlistsCount}</p>
          <p className="text-sm text-gray-500">Wishlist additions</p>
          <a 
            href={`https://admin.keyp.fo/store/${storeId}/wishlists`}
            className="text-blue-600 hover:text-blue-800 text-sm mt-2 inline-block"
          >
            View Wishlist Stats →
          </a>
        </div>
      </div>
    </div>
  );
};
```

## Technical Integration Details

### Shared Firebase Project

All three services operate within the same Firebase project, sharing:

- Authentication
- Firestore database (with appropriate collection namespacing)
- Storage bucket
- Cloud Functions

### Domain Configuration

Each service has its own subdomain:

- keyp.fo - Main service
- ynskilisti.keyp.fo - Wishlist service
- gavuhugskot.keyp.fo - Gift ideas service

Configure Firebase Hosting with multiple sites for each domain.

### Deployment Coordination

Coordinate deployments across all services to ensure compatibility:

1. Use semantic versioning for all services
2. Document API changes and ensure backward compatibility
3. Test integration points before deployment
4. Deploy services in the correct order to avoid breaking dependencies

## Best Practices for Integration

1. **Keep Services Loosely Coupled**: While integrated, each service should be able to function independently.

2. **Use Well-Defined APIs**: Document and version all APIs used for cross-service communication.

3. **Consistent Error Handling**: Implement consistent error handling across services.

4. **Shared Authentication**: Use a single authentication system across all services.

5. **Unified Design Language**: Maintain visual consistency with shared components.

6. **Feature Toggles**: Use feature toggles to enable/disable integration points during deployment.

7. **Monitoring**: Set up monitoring for integration points to detect failures.

## Integration Testing

Create end-to-end tests that verify the integration between services:

```typescript
// Example integration test
describe('Cross-service integration', () => {
  it('should add product to wishlist from Keyp.fo', async () => {
    // Set up test user
    const user = await createTestUser();
    
    // Add product to wishlist via Keyp.fo API
    const result = await addToWishlist(user.id, testProduct);
    expect(result.success).toBe(true);
    
    // Verify product was added to wishlist in Ynskilisti
    const wishlists = await getUserWishlists(user.id);
    const wishlist = wishlists[0];
    
    expect(wishlist.items).toContainEqual(expect.objectContaining({
      productId: testProduct.id,
      storeId: testProduct.storeId
    }));
  });
});
```

## Troubleshooting Common Integration Issues

### Authentication Issues

**Problem**: User is logged in on one service but not authenticated on another.

**Solution**: Ensure cookies and tokens are shared across subdomains. Check that Firebase Authentication is configured correctly for all domains.

### Cross-Origin Requests

**Problem**: API requests fail due to CORS policies.

**Solution**: Configure CORS headers correctly on all API endpoints to allow requests from all Keyp subdomains.

### Data Consistency

**Problem**: Data inconsistency across services.

**Solution**: Implement data validation and synchronization mechanisms. Consider using Firebase Cloud Functions to maintain consistency across services.

### User Experience Breaks

**Problem**: Inconsistent UI or UX when moving between services.

**Solution**: Implement shared components and design tokens. Test user journeys that span multiple services.

## Future Integration Plans

1. **Single Sign-On Enhancement**: Improve the authentication flow between services.
2. **Unified Notifications**: Create a centralized notification system across all services.
3. **Shared Analytics**: Implement cross-service analytics for better understanding of user journeys.
4. **Mobile App Integration**: Prepare services for integration into a future mobile application. 