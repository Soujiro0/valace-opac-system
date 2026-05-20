# ValACE OPAC - Architecture & System Design

> **Note:** The full source code for this system is closed-source as it belongs to the Valenzuela City Academic Center of Excellence (ValACE). This repository serves as a technical showcase of the system architecture and stack I engineered as a Full-Stack Web Developer Intern.

# ValACE New OPAC - Library Resource Management System

A comprehensive Online Public Access Catalog (OPAC) system for managing library resources, integrating external APIs, and providing advanced book search capabilities. Built with Laravel backend and React frontend.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Backend Documentation](#backend-documentation)
- [Frontend Documentation](#frontend-documentation)
- [API Endpoints](#api-endpoints)
- [Capabilities](#capabilities)
- [Deployment](#deployment)

## 🎯 Overview

ValACE New OPAC is a modern library management system that enables:

- **Resource Management**: Create and manage API resources and redirect links to external library systems
- **Book Search**: Advanced search functionality across multiple integrated resources
- **Featured Books**: Curate and showcase featured book collections
- **API Integration**: Connect to external academic databases (EBSCOhost, JSTOR, ProQuest, etc.)
- **Sync Management**: Granular per-endpoint synchronization with external APIs
- **Admin Dashboard**: Comprehensive administrative interface for resource management

## ✨ Features

### Core Features
- 📚 **Multi-resource Book Search** - Search across internal and external book databases
- 🔗 **API Resource Integration** - Connect to external library APIs with authentication
- 🔄 **Automated Sync** - Scheduled synchronization of external resources
- ⭐ **Featured Books** - Curate and display featured book collections
- 📊 **Dashboard Analytics** - Monitor system statistics and resource status
- 📝 **Activity Logging** - Comprehensive logging system for debugging and monitoring
- 🔐 **Admin Authentication** - Secure admin access with token-based auth

### Advanced Features
- **Per-Endpoint Sync Control** - Granular synchronization at endpoint level
- **Real-time Status Updates** - Live sync status monitoring with adaptive polling
- **Rate Limiting** - Built-in rate limiting for external API calls
- **Chunked Processing** - Efficient data processing in configurable chunks
- **Retry Logic** - Automatic retry with exponential backoff
- **Search Highlighting** - Full-text search with Meilisearch integration

## 🏗️ Architecture

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Frontend (React)                        │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  Pages: Search, Admin Dashboard, Resources, Logs      │ │
│  │  Components: UI, Layout, Page-specific                │ │
│  │  State: React Query, Context API                      │ │
│  └────────────────────────────────────────────────────────┘ │
└────────────────────┬────────────────────────────────────────┘
                     │ REST API (JSON)
┌────────────────────▼────────────────────────────────────────┐
│                   Backend (Laravel)                          │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  Controllers: Resource, Book, Auth, Sync, Log         │ │
│  │  Services: Fetch, Store, Update, Search, Sync         │ │
│  │  Jobs: Sync, Process, Update (Queue-based)            │ │
│  │  Models: Resource, ApiResource, ApiEndpoint, Book     │ │
│  └────────────────────────────────────────────────────────┘ │
└────────────────────┬────────────────────────────────────────┘
                     │
        ┌────────────┼────────────┐
        │            │            │
┌───────▼──────┐ ┌──▼───────┐ ┌─▼──────────┐
│   MySQL DB   │ │ Queue    │ │ Meilisearch│
│   Resources  │ │ Worker   │ │   Search   │
│   Books      │ │ Jobs     │ │   Index    │
└──────────────┘ └──────────┘ └────────────┘
```

### Technology Stack

**Backend:**
- Laravel 10.x (PHP 8.1+)
- MySQL 8.0+
- Meilisearch (Full-text search)
- Queue Workers (Background jobs)
- Composer (Dependency management)

**Frontend:**
- React 18.x
- Vite (Build tool)
- TailwindCSS (Styling)
- React Query (Data fetching)
- React Router (Routing)
- Axios (HTTP client)

## 📚 Backend Documentation

### Directory Structure

```
backend/
├── app/
│   ├── Console/           # Artisan commands
│   │   └── Commands/      # Custom commands (SyncApiEndpointsCommand)
│   ├── Exceptions/        # Exception handlers
│   ├── Http/
│   │   ├── Controllers/   # API controllers
│   │   ├── Middleware/    # Custom middleware (BasicAuthMiddleware)
│   │   └── Resources/     # API resources (transformers)
│   ├── Jobs/              # Queue jobs
│   │   ├── SyncApiEndpointJob.php
│   │   ├── ProcessExternalResourceChunk.php
│   │   └── FetchAndDispatchExternalResource.php
│   ├── Logging/           # Custom logging
│   │   └── CustomLogger.php
│   ├── Models/            # Eloquent models
│   │   ├── Resource.php
│   │   ├── ApiResource.php
│   │   ├── ApiEndpoint.php
│   │   ├── CacheBook.php
│   │   └── FeaturedBook.php
│   └── Services/          # Business logic services
│       ├── ResourceDataFetchService.php
│       ├── ResourceDataStoreService.php
│       ├── ResourceDataUpdateService.php
│       ├── ResourceDataDestroyService.php
│       ├── BookSearchService.php
│       ├── BookFetchService.php
│       ├── FeaturedBookService.php
│       └── ExternalResourceFetchService.php
├── config/                # Configuration files
├── database/
│   ├── migrations/        # Database migrations
│   └── seeders/          # Database seeders
├── routes/
│   └── api.php           # API route definitions
├── storage/
│   └── logs/             # Application logs
└── tests/                # PHPUnit tests
```

### Core Services

#### ResourceDataFetchService
Handles fetching and filtering of resources:
- `fetchAllResources()` - Retrieve resources with filtering, sorting, pagination
- `fetchResourceById()` - Get single resource with relationships
- `fetchActiveResources()` - Get only active resources
- Supports filtering by type (API/REDIRECT), status (ACTIVE/INACTIVE)

#### ResourceDataStoreService
Creates new resources:
- `storeResource()` - Create resource with validation
- Handles both API and REDIRECT resource types
- Creates associated ApiResource, ApiEndpoints, and RedirectLink records

#### ResourceDataUpdateService
Updates existing resources:
- `updateResource()` - Full resource update
- `updateStatus()` - Update resource status only
- `updateApiConfig()` - Update API configuration
- `updateEndpoints()` - Update API endpoints
- `updateRedirect()` - Update redirect link

#### ResourceDataDestroyService
Manages resource deletion:
- `deleteResource()` - Soft delete resource
- `destroyMultipleResources()` - Batch delete
- `softDeleteResource()` - Explicit soft delete
- `restoreResource()` - Restore soft-deleted resource
- `forceDeleteResource()` - Permanent deletion

#### BookSearchService
Advanced book search functionality:
- `searchBooks()` - Multi-criteria search
- Full-text search using Meilisearch
- Filtering by title, author, ISBN, date range
- Sorting and pagination support

#### ExternalResourceFetchService
Fetches data from external APIs:
- `fetchExternalResource()` - Fetch from single endpoint
- Handles authentication (Bearer, API Key, Basic Auth)
- Rate limiting and retry logic
- Field mapping and data transformation

#### FeaturedBookService
Manages featured book collections:
- `fetchBooksForSelection()` - Get books available for featuring
- `fetchCuratedBooks()` - Get current featured books
- `storeFeaturedBooks()` - Add books to featured collection
- `deleteFeaturedBook()` - Remove from featured collection

#### DashboardStatsService
Provides dashboard statistics:
- Total resources count
- Active/inactive resource breakdown
- API vs Redirect resource counts
- System health metrics

### Controllers

| Controller | Purpose | Key Methods |
|------------|---------|-------------|
| **ResourceController** | Resource CRUD operations | index, store, show, update, destroy, updateStatus, updateApiConfig |
| **BookManagerController** | Book search and management | searchBooks, fetchBooksForFeaturedSelection, storeCuratedFeaturedBooks |
| **ApiSyncController** | Sync management | syncResource, syncEndpoint, getEndpointStatus, getResourceStatus |
| **AuthController** | Authentication | login, verify, logout |
| **DashboardController** | Dashboard data | stats |
| **LogController** | Log retrieval | getLaravelLog, getInfoLog, getErrorLog, clearAllLogs |

### Background Jobs

#### SyncApiEndpointJob
- Syncs individual API endpoint
- Updates sync status (IDLE → SYNCING → COMPLETED/FAILED)
- Handles authentication and rate limiting
- Processes data in chunks
- Automatic retry with exponential backoff

#### ProcessExternalResourceChunk
- Processes fetched data chunks
- Stores/updates book records in cache
- Maps fields from external format to internal format
- Handles duplicate detection

#### FetchAndDispatchExternalResource
- Dispatches sync jobs for all endpoints of a resource
- Orchestrates resource-level synchronization

### Models

| Model | Description | Key Relationships |
|-------|-------------|-------------------|
| **Resource** | Base resource model | hasOne(ApiResource), hasOne(RedirectLink) |
| **ApiResource** | API configuration | belongsTo(Resource), hasMany(ApiEndpoint) |
| **ApiEndpoint** | API endpoint details | belongsTo(ApiResource) |
| **CacheBook** | Cached book data | belongsTo(ApiResource), searchable with Meilisearch |
| **FeaturedBook** | Featured book references | belongsTo(CacheBook) |
| **RedirectLink** | External redirect URLs | belongsTo(Resource) |

### Commands

```bash
# Sync API endpoints
php artisan api:sync-endpoints

# Sync specific endpoint
php artisan api:sync-endpoints --endpoint-id=123

# Sync all endpoints for a resource
php artisan api:sync-endpoints --resource-id=456

# Force sync even if already syncing
php artisan api:sync-endpoints --force
```

## 🎨 Frontend Documentation

### Directory Structure

```
frontend/
├── src/
│   ├── api/               # API service layer
│   │   ├── ApiService.js
│   │   ├── ApiSyncService.js
│   │   ├── ResourceService.js
│   │   ├── AuthService.js
│   │   └── LogService.js
│   ├── components/
│   │   ├── layout/        # Layout components
│   │   ├── page_components/ # Page-specific components
│   │   └── ui/            # Reusable UI components
│   ├── contexts/          # React contexts
│   │   └── AuthContext.jsx
│   ├── hooks/             # Custom React hooks
│   │   ├── auth/          # Auth hooks
│   │   ├── books/         # Book hooks
│   │   ├── dashboard/     # Dashboard hooks
│   │   ├── resources/     # Resource hooks
│   │   └── useNotification.jsx
│   ├── pages/             # Page components
│   │   ├── admin/         # Admin pages
│   │   ├── SearchPage.jsx
│   │   └── SourcePage.jsx
│   ├── routes/            # Routing configuration
│   │   ├── AppRoute.jsx
│   │   └── ProtectedRoute.jsx
│   └── utils/             # Utility functions
├── public/                # Static assets
└── package.json
```

### Pages

#### Public Pages
- **SearchPage** (`/`) - Main book search interface
  - Multi-field search form
  - Results with pagination
  - Filter by source/resource
  - Real-time search with debouncing

- **SourcePage** (`/source/:id`) - Individual resource detail page
  - Display resource information
  - Redirect to external resource

#### Admin Pages
- **AdminLoginPage** (`/admin/login`) - Admin authentication
  - Basic auth login form
  - Redirect to dashboard on success

- **AdminDashboard** (`/admin/dashboard`) - Dashboard overview
  - System statistics
  - Quick action buttons
  - Recent activity

- **ResourcePage** (`/admin/resources`) - Resource management
  - List all resources with filtering
  - Create/edit/delete resources
  - Sync controls for API resources
  - Real-time sync status

- **FeaturedBooksPage** (`/admin/featured-books`) - Featured books management
  - Select books for featuring
  - Manage featured collection
  - Drag-and-drop reordering

- **LogsPage** (`/admin/logs`) - System logs viewer
  - Laravel system logs
  - Application info logs
  - Error logs
  - Log clearing functionality

- **CreateApiResourcePage** (`/admin/resources/create/api`) - Create API resource
  - API configuration form
  - Endpoint management
  - Authentication setup

- **EditApiResourcePage** (`/admin/resources/:id/edit`) - Edit resource
  - Update resource details
  - Modify endpoints
  - Change authentication

- **CreateRedirectResourcePage** (`/admin/resources/create/redirect`) - Create redirect link
  - Simple redirect URL setup

### Key Components

#### Layout Components
- **AdminLayout** - Admin page wrapper with navigation
- **LayoutWrapper** - Root layout provider

#### Resource Page Components
- **ResourceTable** - Displays resources in table format
- **ResourceCard** - Card view for resources
- **SyncStatusBadge** - Shows sync status with animations
- **ResourceFilters** - Filter controls for resource list

#### UI Components
- **Button** - Styled button component
- **Input** - Form input component
- **Modal** - Modal dialog component
- **Toast** - Notification toast
- **LogoLoading** - Loading spinner with logo

### API Services

#### ResourceService.js
```javascript
// Get all resources with filtering
getActiveApiResources(params, auth)

// Get single resource
getResourceById(id, auth)

// Create resource
createResource(data, auth)

// Update resource
updateResource(id, data, auth)

// Delete resource
deleteResource(id, auth)
```

#### ApiSyncService.js
```javascript
// Sync all endpoints of a resource
syncResource(resourceId, auth)

// Sync single endpoint
syncEndpoint(endpointId, auth)

// Get endpoint sync status
getEndpointStatus(endpointId, auth)

// Get resource sync status
getResourceStatus(resourceId, auth)
```

#### AuthService.js
```javascript
// Login
login(username, password)

// Verify token
verify(token)

// Logout
logout(token)
```

### Custom Hooks

#### Resource Hooks
- `useResources()` - Fetch resources with React Query
- `useCreateResource()` - Create resource mutation
- `useUpdateResource()` - Update resource mutation
- `useDeleteResource()` - Delete resource mutation
- `useSyncResourceMutation()` - Sync resource
- `useSyncEndpointMutation()` - Sync endpoint
- `useResourceSyncStatus()` - Poll sync status

#### Book Hooks
- `useSearchBooks()` - Search books with filters
- `useFeaturedBooks()` - Get featured books
- `useStoreFeaturedBooks()` - Add featured books
- `useDeleteFeaturedBook()` - Remove featured book

#### Auth Hooks
- `useAuth()` - Access auth context
- `useLogin()` - Login mutation
- `useLogout()` - Logout mutation

#### Utility Hooks
- `useNotification()` - Toast notification system
- `useDebounce()` - Debounce value changes
- `useKeyRedirect()` - Keyboard shortcut navigation

### State Management

The application uses:
- **React Query** for server state (data fetching, caching, synchronization)
- **React Context** for global state (authentication)
- **Local State** for component-specific state

## 🔌 API Endpoints

### Base URL
```
http://localhost:8000/api/v1
```

### Authentication
Most endpoints require Basic Auth:
```
Authorization: Basic base64(username:password)
```

### Resource Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/resources` | List all resources |
| GET | `/resources/{id}` | Get single resource |
| POST | `/resources` | Create resource |
| PUT | `/resources/{id}` | Update resource |
| PATCH | `/resources/{id}` | Partial update |
| DELETE | `/resources/{id}` | Delete resource |
| DELETE | `/resources/multiple` | Delete multiple resources |
| DELETE | `/resources/{id}/soft` | Soft delete resource |
| POST | `/resources/{id}/restore` | Restore soft-deleted resource |
| DELETE | `/resources/{id}/force` | Force delete resource |
| PATCH | `/resources/{id}/status` | Update status only |
| PATCH | `/resources/{id}/api-config` | Update API config |
| PATCH | `/resources/{id}/endpoints` | Update endpoints |
| PATCH | `/resources/{id}/redirect` | Update redirect URL |

#### Query Parameters (GET /resources)
```
?type=API|REDIRECT          - Filter by type
&status=ACTIVE|INACTIVE     - Filter by status
&per_page=20                - Results per page
&page=1                     - Page number
&sort_by=name|created_at    - Sort field
&sort_order=asc|desc        - Sort direction
```

### Sync Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/sync/resources/{id}` | Sync all endpoints of resource |
| POST | `/sync/endpoints/{id}` | Sync single endpoint |
| GET | `/sync/endpoints/{id}/status` | Get endpoint sync status |
| GET | `/sync/resources/{id}/status` | Get resource sync status |

### Book Search

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/books/search` | Search books |
| GET | `/books/featured/selection` | Get books for featuring |
| GET | `/books/featured/curated` | Get current featured books |
| POST | `/books/featured/store` | Add featured books |
| DELETE | `/books/featured/delete/{id}` | Remove featured book |

#### Search Parameters
```
?q=search_term              - Search query
&title=book_title           - Filter by title
&author=author_name         - Filter by author
&isbn=isbn_number           - Filter by ISBN
&api_resource_id=123        - Filter by resource
&date_from=2020-01-01       - Date range start
&date_to=2024-12-31         - Date range end
&per_page=20                - Results per page
&page=1                     - Page number
```

### Dashboard

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/dashboard/stats` | Get dashboard statistics |

### Logs

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/logs/laravel` | Get Laravel log (text/plain) |
| GET | `/logs/info` | Get info log (text/plain) |
| GET | `/logs/error` | Get error log (text/plain) |
| POST | `/logs/laravel/clear` | Clear Laravel log |
| POST | `/logs/info/clear` | Clear info log |
| POST | `/logs/error/clear` | Clear error log |
| POST | `/logs/all/clear` | Clear all logs |

### Authentication

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/auth/login` | Admin login |
| POST | `/auth/auth/verify` | Verify token |
| POST | `/auth/auth/logout` | Logout |

### Response Format

All API responses follow this format:

**Success Response:**
```json
{
  "data": { ... },
  "message": "Success message",
  "success": true,
  "status_code": 200
}
```

**Error Response:**
```json
{
  "data": null,
  "message": "Error message",
  "success": false,
  "status_code": 400|404|500,
  "error": "Detailed error information"
}
```

**Paginated Response:**
```json
{
  "data": {
    "results": [ ... ],
    "total_results": 100,
    "per_page": 20,
    "current_page": 1,
    "last_page": 5,
    "from": 1,
    "to": 20
  },
  "message": "Resources fetched successfully",
  "success": true,
  "status_code": 200
}
```

For detailed API documentation with request/response examples, see [backend/API_DOCUMENTATION.md](backend/API_DOCUMENTATION.md)

## 🎯 Capabilities

### 1. Multi-source Book Search
- Search across internal cache and external APIs
- Full-text search with Meilisearch
- Advanced filtering (title, author, ISBN, date range)
- Real-time results with pagination
- Source attribution for each result

### 2. External API Integration
- Connect to academic databases (EBSCOhost, JSTOR, ProQuest, ScienceDirect)
- Multiple authentication methods:
  - Bearer Token
  - API Key (Header/Query Parameter)
  - Basic Authentication
- Custom field mapping for data transformation
- Rate limiting and retry logic

### 3. Resource Synchronization
- **Granular Control**: Per-endpoint sync management
- **Scheduled Sync**: Automatic sync every 30 minutes
- **Manual Sync**: On-demand sync via UI or CLI
- **Status Tracking**: Real-time sync status (IDLE, SYNCING, COMPLETED, FAILED)
- **Chunked Processing**: Process large datasets in manageable chunks
- **Error Recovery**: Automatic retry with exponential backoff

### 4. Featured Books Management
- Curate featured book collections
- Select from available books
- Display on public-facing pages
- Easy management interface

### 5. Admin Dashboard
- System statistics overview
- Resource management interface
- Featured books curation
- Log viewing and management
- Sync status monitoring

### 6. Logging and Monitoring
- **Laravel System Logs**: Framework-level logging
- **Application Info Logs**: Success and information events
- **Error Logs**: Error tracking and debugging
- **Activity Logs**: User actions and system events
- **Real-time Log Viewer**: View logs in admin interface
- **Log Clearing**: Clean up old log files

### 7. Authentication and Security
- Admin authentication with token-based auth
- Basic Auth for API endpoints
- Protected routes with middleware
- Session management
- Secure password handling

### 8. Performance Optimization
- Database query optimization
- Eager loading of relationships
- Response caching
- Queue-based background processing
- Efficient pagination
- Meilisearch for fast full-text search

### 9. Developer-Friendly
- Comprehensive API documentation
- Consistent response format
- Detailed error messages
- Artisan commands for common tasks
- Environment-based configuration
- Database migrations and seeders

## 📦 Deployment

### Production Checklist

**Backend:**
```bash
# 1. Install dependencies
composer install --optimize-autoloader --no-dev

# 2. Configure environment
cp .env.example .env
# Edit .env with production settings

# 3. Generate key and optimize
php artisan key:generate
php artisan config:cache
php artisan route:cache
php artisan view:cache

# 4. Run migrations
php artisan migrate --force

# 5. Set permissions
chmod -R 775 storage bootstrap/cache
chown -R www-data:www-data storage bootstrap/cache

# 6. Start queue workers (use supervisor)
php artisan queue:work --daemon

# 7. Setup scheduler (add to crontab)
* * * * * cd /path-to-project && php artisan schedule:run >> /dev/null 2>&1
```

**Frontend:**
```bash
# 1. Install dependencies
npm install

# 2. Build for production
npm run build

# 3. Deploy dist/ folder to web server
# Configure web server to serve index.html for all routes
```

### Web Server Configuration

**Nginx:**
```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /path/to/ValACE-New-OPAC/backend/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

### Environment Variables

**Backend (.env):**
```env
APP_NAME=ValACE
APP_ENV=production
APP_DEBUG=false
APP_URL=https://your-domain.com

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=valace_db
DB_USERNAME=your_username
DB_PASSWORD=your_password

CACHE_DRIVER=redis
QUEUE_CONNECTION=redis
SESSION_DRIVER=redis

SCOUT_DRIVER=meilisearch
MEILISEARCH_HOST=http://127.0.0.1:7700
MEILISEARCH_KEY=your_master_key
```

**Frontend (.env):**
```env
VITE_API_BASE_URL=https://your-domain.com/api/v1
```

## 👤 Author

**Jerry Castrudes**
- GitHub: [@Soujiro0](https://github.com/Soujiro0)

## 📂 Architectural Exhibits
*(Link directly to the safe code snippets you extracted)*
* 📄 [Sanitized Database Schema](./assets/database-schema.sql)
* 📄 [Meilisearch Integration Snippet](./assets/search-service.php)
* 📄 [Queue Worker Job Example](./assets/sync-job.php)
