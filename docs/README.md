# Keyp.fo Platform Documentation

This documentation provides an overview of the Keyp.fo platform architecture, features, and implementation details.

## Table of Contents

- [Project Overview](./project-overview.md)
- [Getting Started](./getting-started.md)
- [Architecture](./architecture.md)
- [Database Schema](./database-schema.md)
- [API Reference](./api-reference.md)
- [Development Guidelines](./development.md)
- [Security](./security.md)
- [Integration with Other Keyp Services](./integration.md)

## Keyp Ecosystem

The Keyp ecosystem consists of three primary services:

1. **Keyp.fo** - The main platform for store listings and product discovery
2. **Ynskilisti** - The wishlist service allowing users to create wishlists of products
3. **Gavuhugskot** - The gift ideas service providing gift recommendations

All services are integrated through a centralized API (`api.keyp.fo`) and share:

- Common authentication system (Firebase Auth)
- Unified data models and API endpoints
- Centralized image handling service
- Consistent design elements and user experience

## Key Features

- Store directory with detailed information
- Product browsing and search
- Wishlist integration with Ynskilisti
- Gift ideas integration with Gavuhugskot
- Admin and store owner dashboards
- Mobile-friendly responsive design

## API Documentation

For detailed API information, see the [API Reference](./api-reference.md) documentation.
The central API (`https://api.keyp.fo/v1/`) serves all Keyp services with unified endpoints for:

- Stores and products
- Wishlists and wishlist items
- Gift ideas and recommendations

## Installation and Setup

Please refer to the [Getting Started](./getting-started.md) guide for setup instructions.

## Contributing

Please follow the [Development Guidelines](./development.md) when contributing to this project.

## License

Proprietary - All rights reserved.

## Development Environment

The Keyp.fo platform is built using:

- [Next.js](https://nextjs.org/) - React framework for server-side rendering and static generation
- [TypeScript](https://www.typescriptlang.org/) - Typed JavaScript
- [Tailwind CSS](https://tailwindcss.com/) - Utility-first CSS framework
- [Firebase](https://firebase.google.com/) - Backend services (Authentication, Firestore, Functions)

## Contact

For questions or support, please contact:
- Development Team: dev@keyp.fo
- Support: support@keyp.fo 