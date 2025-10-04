# System Documentation - Origin Alpha Management System
**Agent:** Documentation Agent  
**Date:** October 2, 2025
**Status:** Living Document

## Table of Contents
1. [System Overview](#system-overview)
2. [Getting Started](#getting-started)
3. [Architecture Guide](#architecture-guide)
4. [Module Documentation](#module-documentation)
5. [API Reference](#api-reference)
6. [Development Guide](#development-guide)
7. [Deployment Guide](#deployment-guide)
8. [Troubleshooting](#troubleshooting)

## 1. System Overview

### 1.1 Introduction
Origin Alpha is a comprehensive printing business management system designed as a multi-tenant SaaS platform. It streamlines the entire printing workflow from order submission through design, production, and billing.

### 1.2 Key Features
- **Multi-tenant Architecture**: Database isolation per tenant
- **Order Management**: Complete workflow automation
- **File Processing**: TIFF to JPG conversion, watermarking
- **Design Review**: Canvas-based annotation system
- **Dynamic Pricing**: Flexible pricing engine
- **Communication**: WhatsApp and email integration
- **Billing**: Partial payments, credit management

### 1.3 Technology Stack
| Layer | Technology | Version |
|-------|------------|---------|
| Backend | Laravel | 11.x |
| Frontend | React | 18.x |
| Database | PostgreSQL | 15 |
| Cache | Redis | 7.0 |
| Queue | Laravel Horizon | Latest |
| Storage | S3/MinIO | Latest |

## 2. Getting Started

### 2.1 Prerequisites
```bash
# System Requirements
- PHP >= 8.2
- Node.js >= 18
- PostgreSQL >= 15
- Redis >= 7.0
- Composer >= 2.0
```

### 2.2 Installation

#### Backend Setup
```bash
# Clone repository
git clone https://github.com/origin-alpha/backend.git
cd backend

# Install dependencies
composer install

# Copy environment file
cp .env.example .env

# Generate application key
php artisan key:generate

# Run migrations
php artisan migrate --seed

# Start queue worker
php artisan horizon

# Start development server
php artisan serve
```

#### Frontend Setup
```bash
# Clone frontend repository
git clone https://github.com/origin-alpha/frontend.git
cd frontend

# Install dependencies
npm install

# Copy environment file
cp .env.example .env.local

# Start development server
npm run dev
```

### 2.3 Configuration

#### Multi-tenancy Setup
```php
// config/multitenancy.php
return [
    'tenant_finder' => \App\Tenancy\TenantFinder::class,
    'tenant_db_prefix' => 'tenant_',
    'master_db' => env('DB_DATABASE', 'origin_alpha_master'),
    'tenant_migrations_path' => database_path('migrations/tenant'),
];
```

#### File Storage Configuration
```php
// config/filesystems.php
'disks' => [
    's3' => [
        'driver' => 's3',
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION'),
        'bucket' => env('AWS_BUCKET'),
        'url' => env('AWS_URL'),
        'endpoint' => env('AWS_ENDPOINT'), // For MinIO
    ],
],
```

## 3. Architecture Guide

### 3.1 System Architecture
```
┌─────────────────────────────────────────────┐
│            Client Applications              │
├──────────┬───────────┬────────────────────┤
│  React   │  Mobile   │   WhatsApp Bot     │
│   SPA    │   PWA     │                    │
└──────────┴───────────┴────────────────────┘
                 │
          ┌──────▼──────┐
          │   Nginx LB  │
          └──────┬──────┘
                 │
┌────────────────────────────────────────────┐
│         Laravel API Gateway                │
├────────────────────────────────────────────┤
│      Business Logic Service Layer          │
├──────────┬───────────┬────────────────────┤
│  ErpGo   │  Custom   │   Integration      │
│ Modules  │  Modules  │      Layer         │
└──────────┴───────────┴────────────────────┘
                 │
      ┌──────────┴──────────┐
      │                     │
┌─────▼─────┐        ┌─────▼─────┐
│PostgreSQL │        │   Redis   │
└───────────┘        └───────────┘
```

### 3.2 Module Architecture
- **Core Modules**: Authentication, Authorization, Tenancy
- **Business Modules**: Orders, Products, Printing, Design, Production
- **Support Modules**: Billing, CRM, Reports, Communication
- **Service Layer**: File Processing, Pricing Engine, Workflow Engine

### 3.3 Database Design
- **Master Database**: Tenant management, global settings
- **Tenant Databases**: Isolated data per tenant
- **Caching Layer**: Redis for performance optimization
- **File Storage**: S3/MinIO for documents and images

## 4. Module Documentation

### 4.1 Order Management Module

#### Overview
Handles complete order lifecycle from submission to payment.

#### Key Components
```php
// Models
App\Models\Order
App\Models\OrderItem
App\Models\OrderWorkflowHistory

// Services
App\Services\OrderService
App\Services\WorkflowEngineService

// Controllers
App\Http\Controllers\Api\OrderController
```

#### Workflow States
1. **Submitted**: Initial order placement
2. **Queued**: In task queue for staff
3. **Claimed**: Staff member assigned
4. **Designing**: Design work in progress
5. **Review**: Client review phase
6. **Corrections**: Revisions needed
7. **Approved**: Design approved
8. **Production**: Being printed
9. **Completed**: Ready for collection
10. **Delivered**: Order delivered
11. **Invoiced**: Invoice generated
12. **Paid**: Payment received

#### Usage Example
```php
// Create new order
$order = OrderService::create([
    'customer_id' => $customerId,
    'items' => $orderItems,
    'deadline' => $deadline,
    'priority' => 5
]);

// Transition status
WorkflowEngineService::transitionTo($order, 'claimed', $user);

// Add file to order
FileService::attachToOrder($file, $order);
```

### 4.2 Product Management Module

#### Overview
Manages product catalog with dynamic pricing and variants.

#### Pricing Types
- **Fixed**: Standard fixed price
- **Per Square Foot**: Area-based calculation
- **Per Piece**: Quantity-based pricing
- **Custom**: Manual quotation

#### Configuration Example
```php
// Create product with variants
$product = Product::create([
    'name' => 'Business Card',
    'pricing_type' => 'per_piece',
    'base_price' => 0.10
]);

$product->variants()->create([
    'name' => 'Premium Paper',
    'price_modifier' => 0.05,
    'attributes' => ['paper_quality' => 'premium']
]);

// Calculate price
$price = PricingEngine::calculate($product, [
    'quantity' => 1000,
    'variant_id' => $variant->id
]);
```

### 4.3 File Processing Module

#### Overview
Handles file uploads, format conversion, and CDN management.

#### Supported Formats
- Images: JPEG, PNG, TIFF, PSD, SVG
- Documents: PDF
- Maximum Size: 500MB

#### Processing Pipeline
1. **Upload**: Chunked upload support
2. **Validation**: Format and size checks
3. **Conversion**: TIFF to JPG conversion
4. **Watermarking**: Apply watermark
5. **CDN Upload**: Store in S3/CDN
6. **Database**: Create file record

#### Usage Example
```php
// Process uploaded file
$processor = new FileProcessorService();
$file = $processor->processUpload($uploadedFile, $order);

// Generate preview
$preview = $processor->generatePreview($file);

// Convert TIFF to JPG
$jpgPath = $processor->convertTiffToJpg($tiffPath);
```

### 4.4 Design Review Module

#### Overview
Provides canvas-based annotation tools for design feedback.

#### Features
- Visual annotations (comments, highlights)
- Version tracking
- Revision history
- Approval workflow

#### Frontend Implementation
```typescript
// React component usage
<DesignCanvas 
    fileUrl={design.previewUrl}
    annotations={annotations}
    onAnnotationAdd={handleAddAnnotation}
    onApprove={handleApprove}
    onReject={handleReject}
/>
```

### 4.5 Communication Module

#### Overview
Handles multi-channel notifications via WhatsApp and email.

#### WhatsApp Integration
```php
// Send WhatsApp message
NotificationService::sendWhatsApp(
    $customer->phone,
    'order_status_update',
    ['order' => $order, 'status' => 'approved']
);
```

#### Email Templates
- Order confirmation
- Status updates
- Design approval request
- Invoice generation
- Payment reminder

## 5. API Reference

### 5.1 Authentication

#### Login
```http
POST /api/v1/auth/login
Content-Type: application/json

{
    "email": "user@example.com",
    "password": "password"
}

Response:
{
    "token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
    "user": {
        "id": 1,
        "name": "John Doe",
        "email": "user@example.com"
    }
}
```

#### Refresh Token
```http
POST /api/v1/auth/refresh
Authorization: Bearer {token}

Response:
{
    "token": "eyJ0eXAiOiJKV1QiLCJhbGc..."
}
```

### 5.2 Orders

#### List Orders
```http
GET /api/v1/orders?status=queued&page=1&per_page=20
Authorization: Bearer {token}

Response:
{
    "data": [...],
    "meta": {
        "current_page": 1,
        "total": 100,
        "per_page": 20
    }
}
```

#### Create Order
```http
POST /api/v1/orders
Authorization: Bearer {token}
Content-Type: application/json

{
    "customer_id": 123,
    "deadline": "2025-10-15",
    "priority": 5,
    "items": [
        {
            "product_id": 1,
            "quantity": 100,
            "specifications": {
                "width": 10,
                "height": 5
            }
        }
    ]
}
```

#### Update Order Status
```http
POST /api/v1/orders/{id}/status
Authorization: Bearer {token}
Content-Type: application/json

{
    "status": "approved",
    "notes": "Design approved by client"
}
```

### 5.3 Products

#### Get Product Catalog
```http
GET /api/v1/products
Authorization: Bearer {token}

Response:
{
    "data": [
        {
            "id": 1,
            "name": "Business Card",
            "pricing_type": "per_piece",
            "base_price": 0.10,
            "variants": [...]
        }
    ]
}
```

#### Calculate Price
```http
POST /api/v1/products/{id}/calculate-price
Authorization: Bearer {token}
Content-Type: application/json

{
    "specifications": {
        "quantity": 1000,
        "variant_id": 2
    }
}

Response:
{
    "price": 150.00,
    "breakdown": {
        "base": 100.00,
        "variant_modifier": 50.00,
        "discount": 0
    }
}
```

### 5.4 Files

#### Upload File
```http
POST /api/v1/orders/{orderId}/files
Authorization: Bearer {token}
Content-Type: multipart/form-data

file: (binary)
type: "design"

Response:
{
    "id": 123,
    "original_name": "design.tiff",
    "cdn_url": "https://cdn.example.com/files/123.jpg",
    "preview_url": "https://cdn.example.com/previews/123.jpg"
}
```

## 6. Development Guide

### 6.1 Code Standards

#### PHP/Laravel
```php
// Follow PSR-12 coding standard
namespace App\Services;

class OrderService
{
    public function __construct(
        private OrderRepository $repository,
        private EventDispatcher $events
    ) {}
    
    public function create(array $data): Order
    {
        // Implementation
    }
}
```

#### TypeScript/React
```typescript
// Use functional components with hooks
import React, { useState, useEffect } from 'react';

interface OrderListProps {
    status?: string;
    onOrderSelect: (order: Order) => void;
}

export const OrderList: React.FC<OrderListProps> = ({ 
    status, 
    onOrderSelect 
}) => {
    const [orders, setOrders] = useState<Order[]>([]);
    
    // Implementation
};
```

### 6.2 Testing

#### Unit Tests
```php
// tests/Unit/Services/PricingEngineTest.php
class PricingEngineTest extends TestCase
{
    public function test_calculates_pricing_correctly(): void
    {
        $engine = new PricingEngine();
        $price = $engine->calculate($product, $specifications);
        
        $this->assertEquals(100.00, $price);
    }
}
```

#### Integration Tests
```php
// tests/Feature/OrderApiTest.php
class OrderApiTest extends TestCase
{
    public function test_can_create_order(): void
    {
        $response = $this->postJson('/api/v1/orders', [
            'customer_id' => 1,
            'items' => [...]
        ]);
        
        $response->assertStatus(201)
                 ->assertJsonStructure(['id', 'order_number']);
    }
}
```

### 6.3 Database Migrations

#### Creating Migrations
```bash
# Create migration for tenant database
php artisan make:migration create_orders_table --path=database/migrations/tenant

# Run tenant migrations
php artisan tenants:migrate
```

#### Migration Example
```php
// database/migrations/tenant/create_orders_table.php
public function up(): void
{
    Schema::create('orders', function (Blueprint $table) {
        $table->id();
        $table->string('order_number')->unique();
        $table->foreignId('customer_id')->constrained();
        $table->string('status')->default('submitted');
        $table->integer('priority')->default(5);
        $table->timestamp('deadline')->nullable();
        $table->timestamps();
        
        $table->index(['status', 'priority']);
        $table->index('deadline');
    });
}
```

## 7. Deployment Guide

### 7.1 Production Setup

#### Server Requirements
```yaml
Minimum:
  CPU: 4 vCPU
  RAM: 8GB
  Storage: 100GB SSD
  OS: Ubuntu 22.04 LTS
  
Recommended:
  CPU: 8 vCPU
  RAM: 16GB
  Storage: 500GB SSD
  Database: Managed PostgreSQL
  Cache: Managed Redis
```

#### Nginx Configuration
```nginx
server {
    listen 80;
    server_name app.originalpha.com;
    root /var/www/origin-alpha/public;
    
    index index.php;
    
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
    
    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

### 7.2 Docker Deployment

#### Dockerfile
```dockerfile
FROM php:8.2-fpm

RUN apt-get update && apt-get install -y \
    libpq-dev \
    libzip-dev \
    unzip \
    && docker-php-ext-install pdo pdo_pgsql zip

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www
COPY . .

RUN composer install --no-dev --optimize-autoloader

CMD ["php-fpm"]
```

#### Docker Compose
```yaml
version: '3.8'

services:
  app:
    build: .
    volumes:
      - .:/var/www
    depends_on:
      - postgres
      - redis
      
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - .:/var/www
      
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: origin_alpha
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      
  redis:
    image: redis:7-alpine
    
  horizon:
    build: .
    command: php artisan horizon
    depends_on:
      - redis
      
volumes:
  postgres_data:
```

### 7.3 CI/CD Pipeline

#### GitHub Actions
```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
      - name: Install dependencies
        run: composer install
      - name: Run tests
        run: php artisan test
        
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Deploy to server
        uses: appleboy/ssh-action@v0.1.5
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /var/www/origin-alpha
            git pull origin main
            composer install --no-dev
            php artisan migrate --force
            php artisan config:cache
            php artisan route:cache
            php artisan horizon:terminate
```

## 8. Troubleshooting

### 8.1 Common Issues

#### Issue: File upload fails for large files
```bash
# Solution: Increase PHP limits
# php.ini
upload_max_filesize = 500M
post_max_size = 500M
max_execution_time = 300

# Nginx
client_max_body_size 500M;
```

#### Issue: Queue jobs not processing
```bash
# Check Horizon status
php artisan horizon:status

# Restart Horizon
php artisan horizon:terminate
php artisan horizon
```

#### Issue: Tenant database connection errors
```php
// Clear and reconnect
DB::purge('tenant');
DB::reconnect('tenant');

// Check tenant configuration
php artisan tenant:list
```

### 8.2 Performance Optimization

#### Database Optimization
```sql
-- Analyze query performance
EXPLAIN ANALYZE SELECT * FROM orders WHERE status = 'pending';

-- Create missing indexes
CREATE INDEX CONCURRENTLY idx_orders_status ON orders(status);

-- Vacuum and analyze tables
VACUUM ANALYZE orders;
```

#### Redis Optimization
```bash
# Monitor Redis performance
redis-cli monitor

# Check memory usage
redis-cli info memory

# Clear cache if needed
php artisan cache:clear
```

### 8.3 Monitoring

#### Application Monitoring
```php
// config/logging.php
'channels' => [
    'stack' => [
        'driver' => 'stack',
        'channels' => ['daily', 'slack'],
    ],
    'slack' => [
        'driver' => 'slack',
        'url' => env('LOG_SLACK_WEBHOOK_URL'),
        'level' => 'error',
    ],
],
```

#### Health Checks
```php
// routes/api.php
Route::get('/health', function () {
    return [
        'status' => 'ok',
        'database' => DB::connection()->getPdo() ? 'connected' : 'disconnected',
        'redis' => Redis::ping() ? 'connected' : 'disconnected',
        'queue' => Queue::size() < 1000 ? 'healthy' : 'overloaded'
    ];
});
```

---

## Appendix A: Environment Variables

```env
# Application
APP_NAME="Origin Alpha"
APP_ENV=production
APP_DEBUG=false
APP_URL=https://app.originalpha.com

# Database
DB_CONNECTION=pgsql
DB_HOST=localhost
DB_PORT=5432
DB_DATABASE=origin_alpha_master
DB_USERNAME=origin
DB_PASSWORD=secret

# Redis
REDIS_HOST=localhost
REDIS_PASSWORD=null
REDIS_PORT=6379

# Storage
FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=xxx
AWS_SECRET_ACCESS_KEY=xxx
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=origin-alpha

# WhatsApp
TWILIO_ACCOUNT_SID=xxx
TWILIO_AUTH_TOKEN=xxx
TWILIO_WHATSAPP_FROM=whatsapp:+xxx

# Mail
MAIL_MAILER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=xxx
MAIL_PASSWORD=xxx
```

## Appendix B: Useful Commands

```bash
# Artisan Commands
php artisan tenant:create {name} {domain}
php artisan tenant:migrate
php artisan tenant:seed
php artisan horizon:snapshot
php artisan queue:retry all

# Maintenance
php artisan down --message="Maintenance in progress"
php artisan up
php artisan optimize
php artisan config:cache
php artisan route:cache

# Development
php artisan ide-helper:generate
php artisan make:service OrderService
php artisan make:repository OrderRepository
```

---

*Documentation maintained by the Origin Alpha Development Team*
*Last Updated: October 2, 2025*
*Version: 1.0.0*
