# API Reference

This document provides reference information for the Keyp.fo API, which allows you to interact with the platform programmatically.

## API Overview

The Keyp.fo API is a RESTful API that provides endpoints for retrieving and managing stores, categories, products, wishlists, and gift ideas across all Keyp.fo services.

### Base URL

```
https://api.keyp.fo/v1
```

### Authentication

All requests must include a Firebase JWT token in the Authorization header:

```
Authorization: Bearer <token>
```

Most API endpoints require authentication. You can authenticate using Firebase Authentication:

1. **Firebase Authentication Token**: Obtained through the Firebase Authentication SDK

### Response Format

All API responses are in JSON format and include a standard structure:

```json
{
  "success": true,
  "data": { /* Response data */ },
  "error": null
}
```

In case of an error:

```json
{
  "success": false,
  "data": null,
  "error": "Error message"
}
```

### Rate Limiting

The API has rate limits to prevent abuse:

- Maximum 10 wishlists created per user per 24 hours
- Maximum 100 items per wishlist
- Maximum 20 items marked as bought per user per hour

## Stores API

### List Stores

Retrieves a list of stores with optional filtering.

**Endpoint**: `GET /stores`

**Query Parameters**:

| Parameter    | Type    | Description                               |
|--------------|---------|-------------------------------------------|
| `limit`      | number  | Maximum number of stores to return (default: 20, max: 100) |
| `offset`     | number  | Number of stores to skip (for pagination) |
| `category`   | string  | Filter by category ID                     |
| `featured`   | boolean | Filter by featured status                 |
| `webshop`    | boolean | Filter stores with webshops               |
| `physical`   | boolean | Filter stores with physical locations     |
| `search`     | string  | Search term for store name or description |
| `sort`       | string  | Field to sort by (name, created, updated) |
| `order`      | string  | Sort order (asc, desc)                    |

**Example Request**:

```
GET /stores?limit=10&category=clothing&featured=true
```

**Example Response**:

```json
{
  "success": true,
  "data": {
    "stores": [
      {
        "id": "store123",
        "name": "Example Store",
        "slug": "example-store",
        "url": "https://example.com",
        "active": true,
        "featured": true,
        "metadata": {
          "logo": "https://storage.keyp.fo/stores/store123/logo.png",
          "description": "An example store with great products",
          "hasWebshop": true,
          "hasPhysicalShop": true,
          "categories": ["clothing", "accessories"],
          "tags": ["fashion", "shoes"]
        },
        "contact": {
          "email": "contact@example.com",
          "phone": "+298 123456",
          "address": {
            "street": "Example Street 123",
            "city": "Tórshavn",
            "postalCode": "100"
          }
        }
      },
      // More stores...
    ],
    "pagination": {
      "total": 45,
      "limit": 10,
      "offset": 0,
      "hasMore": true
    }
  },
  "error": null
}
```

### Get Store

Retrieves a single store by ID or slug.

**Endpoint**: `GET /stores/{id_or_slug}`

**Path Parameters**:

| Parameter     | Type   | Description                  |
|---------------|--------|------------------------------|
| `id_or_slug`  | string | Store ID or slug             |

**Example Request**:

```
GET /stores/example-store
```

**Example Response**:

```json
{
  "success": true,
  "data": {
    "id": "store123",
    "name": "Example Store",
    "slug": "example-store",
    "url": "https://example.com",
    "active": true,
    "featured": true,
    "metadata": {
      "logo": "https://storage.keyp.fo/stores/store123/logo.png",
      "banner": "https://storage.keyp.fo/stores/store123/banner.png",
      "description": "An example store with great products",
      "hasWebshop": true,
      "hasPhysicalShop": true,
      "categories": ["clothing", "accessories"],
      "tags": ["fashion", "shoes"],
      "brands": ["Brand A", "Brand B"],
      "openingHours": {
        "monday": { "open": "09:00", "close": "18:00" },
        "tuesday": { "open": "09:00", "close": "18:00" },
        "wednesday": { "open": "09:00", "close": "18:00" },
        "thursday": { "open": "09:00", "close": "18:00" },
        "friday": { "open": "09:00", "close": "18:00" },
        "saturday": { "open": "10:00", "close": "16:00" },
        "sunday": { "open": null, "close": null }
      }
    },
    "contact": {
      "email": "contact@example.com",
      "phone": "+298 123456",
      "address": {
        "street": "Example Street 123",
        "city": "Tórshavn",
        "postalCode": "100",
        "coordinates": {
          "latitude": 62.007864,
          "longitude": -6.790982
        }
      },
      "social": {
        "facebook": "https://facebook.com/examplestore",
        "instagram": "https://instagram.com/examplestore"
      }
    },
    "subscription": {
      "plan": "premium",
      "status": "active"
    }
  },
  "error": null
}
```

### Create Store

Creates a new store. Requires administrator or store owner authentication.

**Endpoint**: `POST /stores`

**Request Body**:

```json
{
  "name": "New Store",
  "url": "https://newstore.com",
  "metadata": {
    "description": "A new store with amazing products",
    "hasWebshop": true,
    "hasPhysicalShop": true,
    "categories": ["electronics"],
    "tags": ["tech", "gadgets"]
  },
  "contact": {
    "email": "contact@newstore.com",
    "phone": "+298 654321",
    "address": {
      "street": "New Street 456",
      "city": "Tórshavn",
      "postalCode": "100"
    }
  }
}
```

**Example Response**:

```json
{
  "success": true,
  "data": {
    "id": "store456",
    "slug": "new-store",
    "createdAt": "2025-03-21T12:00:00Z"
  },
  "error": null
}
```

### Update Store

Updates an existing store. Requires administrator or store owner authentication.

**Endpoint**: `PUT /stores/{id}`

**Path Parameters**:

| Parameter | Type   | Description |
|-----------|--------|-------------|
| `id`      | string | Store ID    |

**Request Body**: (partial update is supported)

```json
{
  "metadata": {
    "description": "Updated description",
    "categories": ["electronics", "gadgets"]
  },
  "contact": {
    "phone": "+298 999888"
  }
}
```

**Example Response**:

```json
{
  "success": true,
  "data": {
    "id": "store123",
    "updatedAt": "2025-03-21T14:30:00Z"
  },
  "error": null
}
```

### Delete Store

Deletes a store. Requires administrator authentication.

**Endpoint**: `DELETE /stores/{id}`

**Path Parameters**:

| Parameter | Type   | Description |
|-----------|--------|-------------|
| `id`      | string | Store ID    |

**Example Response**:

```json
{
  "success": true,
  "data": {
    "id": "store123",
    "deleted": true
  },
  "error": null
}
```

## Categories API

### List Categories

Retrieves a list of categories with optional filtering.

**Endpoint**: `GET /categories`

**Query Parameters**:

| Parameter    | Type    | Description                                  |
|--------------|---------|----------------------------------------------|
| `parent`     | string  | Filter by parent category ID (null for top-level) |
| `featured`   | boolean | Filter by featured status                    |
| `target`     | string  | Filter by target audience (him, her, kids, home) |

**Example Request**:

```
GET /categories?featured=true
```

**Example Response**:

```json
{
  "success": true,
  "data": {
    "categories": [
      {
        "id": "clothing",
        "name": "Clothing",
        "slug": "clothing",
        "description": "Apparel and fashion items",
        "icon": "https://storage.keyp.fo/categories/clothing/icon.png",
        "image": "https://storage.keyp.fo/categories/clothing/image.jpg",
        "parentId": null,
        "order": 1,
        "featured": true,
        "metadata": {
          "forTarget": "all",
          "tags": ["fashion", "apparel"]
        }
      },
      // More categories...
    ]
  },
  "error": null
}
```

### Get Category

Retrieves a single category by ID or slug.

**Endpoint**: `GET /categories/{id_or_slug}`

**Path Parameters**:

| Parameter     | Type   | Description                    |
|---------------|--------|--------------------------------|
| `id_or_slug`  | string | Category ID or slug            |

**Example Request**:

```
GET /categories/clothing
```

**Example Response**:

```json
{
  "success": true,
  "data": {
    "id": "clothing",
    "name": "Clothing",
    "slug": "clothing",
    "description": "Apparel and fashion items",
    "icon": "https://storage.keyp.fo/categories/clothing/icon.png",
    "image": "https://storage.keyp.fo/categories/clothing/image.jpg",
    "parentId": null,
    "order": 1,
    "featured": true,
    "metadata": {
      "forTarget": "all",
      "tags": ["fashion", "apparel"]
    },
    "subcategories": [
      {
        "id": "mens-clothing",
        "name": "Men's Clothing",
        "slug": "mens-clothing",
        "parentId": "clothing",
        "order": 1
      },
      {
        "id": "womens-clothing",
        "name": "Women's Clothing",
        "slug": "womens-clothing",
        "parentId": "clothing",
        "order": 2
      }
    ]
  },
  "error": null
}
```

## Products API

### List Products

Retrieves a list of products with optional filtering.

**Endpoint**: `GET /products`

**Query Parameters**:

| Parameter    | Type    | Description                                  |
|--------------|---------|----------------------------------------------|
| `limit`      | number  | Maximum number of products to return         |
| `offset`     | number  | Number of products to skip (for pagination)  |
| `store`      | string  | Filter by store ID                           |
| `category`   | string  | Filter by category ID                        |
| `giftIdea`   | boolean | Filter gift ideas only                       |
| `search`     | string  | Search term for product name or description  |
| `minPrice`   | number  | Minimum price filter                         |
| `maxPrice`   | number  | Maximum price filter                         |

**Example Request**:

```
GET /products?category=mens-clothing&limit=10
```

**Example Response**:

```json
{
  "success": true,
  "data": {
    "products": [
      {
        "id": "product123",
        "storeId": "store123",
        "name": "Men's T-Shirt",
        "description": "Comfortable cotton t-shirt",
        "price": 199,
        "currency": "DKK",
        "images": ["https://storage.keyp.fo/products/product123/1.jpg"],
        "url": "https://example.com/products/mens-tshirt",
        "active": true,
        "metadata": {
          "categories": ["mens-clothing"],
          "brand": "Example Brand",
          "isGiftIdea": false
        }
      },
      // More products...
    ],
    "pagination": {
      "total": 45,
      "limit": 10,
      "offset": 0,
      "hasMore": true
    }
  },
  "error": null
}
```

### Gift Ideas API

### Search Gift Ideas

Searches for gift ideas based on recipient characteristics.

**Endpoint**: `GET /gift-ideas/search`

**Query Parameters**:

| Parameter      | Type    | Description                              |
|----------------|---------|------------------------------------------|
| `gender`       | string  | Target gender (male, female, unisex)     |
| `minAge`       | number  | Minimum age of recipient                 |
| `maxAge`       | number  | Maximum age of recipient                 |
| `occasion`     | string  | Gift occasion (birthday, christmas, etc.) |
| `priceRange`   | string  | Price range (budget, mid, premium)       |
| `limit`        | number  | Maximum number of results                |

**Example Request**:

```
GET /gift-ideas/search?gender=female&minAge=10&maxAge=15&occasion=birthday
```

**Example Response**:

```json
{
  "success": true,
  "data": {
    "giftIdeas": [
      {
        "id": "product789",
        "storeId": "store456",
        "storeName": "Gift Shop",
        "name": "Friendship Bracelet Set",
        "description": "Colorful friendship bracelet making kit",
        "price": 149,
        "currency": "DKK",
        "images": ["https://storage.keyp.fo/products/product789/1.jpg"],
        "url": "https://giftshop.fo/products/friendship-bracelet",
        "metadata": {
          "categories": ["crafts", "jewelry"],
          "brand": "Craft World",
          "isGiftIdea": true,
          "giftIdeaMetadata": {
            "targetGender": "female",
            "targetAgeMin": 8,
            "targetAgeMax": 16,
            "occasion": ["birthday", "christmas"],
            "priceRange": "budget"
          }
        }
      },
      // More gift ideas...
    ],
    "pagination": {
      "total": 12,
      "limit": 10,
      "offset": 0,
      "hasMore": true
    }
  },
  "error": null
}
```

## User Management API

### Get Current User

Retrieves the profile of the currently authenticated user.

**Endpoint**: `GET /user`

**Example Response**:

```json
{
  "success": true,
  "data": {
    "id": "user123",
    "email": "user@example.com",
    "displayName": "John Doe",
    "role": "store",
    "storeId": "store123",
    "lastLogin": "2025-03-20T18:45:00Z",
    "preferences": {
      "notifications": true,
      "language": "fo"
    }
  },
  "error": null
}
```

### Update User

Updates the current user's profile.

**Endpoint**: `PUT /user`

**Request Body**:

```json
{
  "displayName": "John Smith",
  "preferences": {
    "notifications": false
  }
}
```

**Example Response**:

```json
{
  "success": true,
  "data": {
    "id": "user123",
    "updatedAt": "2025-03-21T15:00:00Z"
  },
  "error": null
}
```

## Error Codes

| Code                    | Description                                          |
|-------------------------|------------------------------------------------------|
| `AUTHENTICATION_REQUIRED` | Authentication is required for this endpoint         |
| `INVALID_CREDENTIALS`   | The provided authentication credentials are invalid  |
| `PERMISSION_DENIED`     | User doesn't have permission to perform this action  |
| `RESOURCE_NOT_FOUND`    | The requested resource was not found                 |
| `VALIDATION_ERROR`      | The request data failed validation                   |
| `RATE_LIMIT_EXCEEDED`   | Rate limit exceeded, try again later                 |
| `SERVER_ERROR`          | An unexpected server error occurred                  |

## Webhooks

Keyp.fo supports webhooks for real-time event notifications. Contact administrators to set up webhook integrations.

### Available Events

- `store.created` - A new store has been created
- `store.updated` - A store has been updated
- `store.deleted` - A store has been deleted
- `product.created` - A new product has been created
- `product.updated` - A product has been updated
- `product.deleted` - A product has been deleted
- `subscription.created` - A new subscription has been created
- `subscription.updated` - A subscription has been updated
- `subscription.expired` - A subscription has expired

### Webhook Payload

```json
{
  "event": "store.updated",
  "timestamp": "2025-03-21T16:00:00Z",
  "data": {
    "id": "store123",
    "name": "Example Store",
    "updatedBy": "user456"
  }
}
```

## SDK Libraries

Official SDKs are available for the following platforms:

- [JavaScript/TypeScript](https://github.com/keyp-fo/keyp-js)
- [Python](https://github.com/keyp-fo/keyp-python)

## Additional Resources

- [Postman Collection](https://www.postman.com/keyp-fo/workspace/keyp-api)
- [OpenAPI Specification](https://api.keyp.fo/openapi.json)
- [API Changelog](./api-changelog.md)

## Wishlists API

### List Items in Wishlist

Retrieves all items in a specific wishlist.

**Endpoint**: `GET /wishlistItems?wishlistId={id}`

**Query Parameters**:

| Parameter    | Type    | Description                               |
|--------------|---------|-------------------------------------------|
| `wishlistId` | string  | ID of the wishlist to retrieve items from |

**Example Request**:

```
GET /wishlistItems?wishlistId=abc123
```

**Example Response**:

```json
{
  "success": true,
  "data": [
    {
      "itemId": "item456",
      "name": "Bluetooth Speaker",
      "description": "Portable wireless speaker with amazing sound quality",
      "price": 499,
      "url": "https://example.com/products/speaker",
      "priority": 2,
      "note": "I prefer black color",
      "createdAt": "2023-11-15T12:00:00Z",
      "updatedAt": "2023-11-15T12:00:00Z"
    }
  ]
}
```

### Get Single Item

Retrieves a specific item from a wishlist.

**Endpoint**: `GET /wishlistItems?wishlistId={id}&itemId={id}`

**Query Parameters**:

| Parameter    | Type    | Description                               |
|--------------|---------|-------------------------------------------|
| `wishlistId` | string  | ID of the wishlist                        |
| `itemId`     | string  | ID of the item to retrieve                |

**Example Request**:

```
GET /wishlistItems?wishlistId=abc123&itemId=item456
```

**Example Response**:

```json
{
  "success": true,
  "data": {
    "itemId": "item456",
    "name": "Bluetooth Speaker",
    "description": "Portable wireless speaker with amazing sound quality",
    "price": 499,
    "url": "https://example.com/products/speaker",
    "priority": 2,
    "note": "I prefer black color",
    "createdAt": "2023-11-15T12:00:00Z",
    "updatedAt": "2023-11-15T12:00:00Z"
  }
}
```

### Add Item to Wishlist

Adds a new item to a wishlist.

**Endpoint**: `POST /wishlistItems?wishlistId={id}`

**Query Parameters**:

| Parameter    | Type    | Description                               |
|--------------|---------|-------------------------------------------|
| `wishlistId` | string  | ID of the wishlist to add the item to     |

**Request Body**:

```json
{
  "name": "Item Name",
  "description": "Item description",
  "priority": 2,
  "price": 99.99,
  "url": "https://example.com/item",
  "note": "Color preference: blue"
}
```

**Example Response**:

```json
{
  "success": true,
  "data": {
    "itemId": "newItem123",
    "name": "Item Name",
    "description": "Item description",
    "priority": 2,
    "price": 99.99,
    "url": "https://example.com/item",
    "note": "Color preference: blue",
    "createdAt": "2023-11-15T14:30:00Z",
    "updatedAt": "2023-11-15T14:30:00Z"
  }
}
```

### Update Item

Updates an existing item in a wishlist.

**Endpoint**: `PUT /wishlistItems?wishlistId={id}&itemId={id}`

**Query Parameters**:

| Parameter    | Type    | Description                               |
|--------------|---------|-------------------------------------------|
| `wishlistId` | string  | ID of the wishlist                        |
| `itemId`     | string  | ID of the item to update                  |

**Request Body**:

```json
{
  "name": "Updated Item Name",
  "priority": 1,
  "note": "Changed color preference"
}
```

**Example Response**:

```json
{
  "success": true,
  "data": {
    "itemId": "item456",
    "name": "Updated Item Name",
    "description": "Portable wireless speaker with amazing sound quality",
    "price": 499,
    "url": "https://example.com/products/speaker",
    "priority": 1,
    "note": "Changed color preference",
    "createdAt": "2023-11-15T12:00:00Z",
    "updatedAt": "2023-11-15T15:45:00Z"
  }
}
```

### Delete Item

Deletes an item from a wishlist.

**Endpoint**: `DELETE /wishlistItems?wishlistId={id}&itemId={id}`

**Query Parameters**:

| Parameter    | Type    | Description                               |
|--------------|---------|-------------------------------------------|
| `wishlistId` | string  | ID of the wishlist                        |
| `itemId`     | string  | ID of the item to delete                  |

**Example Response**:

```json
{
  "success": true,
  "data": {
    "message": "Item successfully deleted"
  }
}
```

## Validation Rules

### Item Properties

- name: Required, 1-200 characters
- description: Optional, max 500 characters
- priority: Required, number 1-5
- price: Optional, number
- url: Optional, valid URL
- note: Optional, max 1000 characters 