# Technical Specification - Origin Alpha Management System
**Agent:** Implementation Agent
**Date:** October 2, 2025
**Status:** In Development

## 1. System Overview

### 1.1 Architecture Pattern
```yaml
Pattern: Modular Monolith with Service Layer
Base: Laravel 11 + React 18
Inspiration: ErpGo Saas (60% reuse) + Caldron Pixer Architecture
Deployment: Multi-tenant SaaS on VPS (8GB RAM, 4 vCPU)
```

### 1.2 Technology Stack

#### Backend Stack
```php
// Core Framework
Laravel: 11.x
PHP: 8.2+
Database: PostgreSQL 15
Cache: Redis 7.0
Queue: Laravel Horizon
Storage: S3-compatible (MinIO/AWS S3)

// Additional Packages
"spatie/laravel-multitenancy": "^3.0"
"spatie/laravel-permission": "^6.0"
"intervention/image": "^3.0"
"maatwebsite/excel": "^3.1"
"barryvdh/laravel-dompdf": "^2.0"
"twilio/sdk": "^7.0" // WhatsApp
"pusher/pusher-php-server": "^7.2" // Real-time
```

#### Frontend Stack
```json
{
  "react": "^18.2.0",
  "typescript": "^5.0.0",
  "react-redux": "^8.0.0",
  "@reduxjs/toolkit": "^1.9.0",
  "react-router-dom": "^6.0.0",
  "axios": "^1.4.0",
  "tailwindcss": "^3.3.0",
  "react-hook-form": "^7.45.0",
  "react-dropzone": "^14.2.0",
  "konva": "^9.2.0", // Canvas annotations
  "react-konva": "^18.2.0"
}
```

## 2. System Architecture

### 2.1 High-Level Architecture
```
┌──────────────────────────────────────────────┐
│             Client Applications              │
├────────────┬────────────┬───────────────────┤
│  React SPA │ Mobile PWA │  WhatsApp Bot    │
└────────────┴────────────┴───────────────────┘
                    │
             ┌──────▼──────┐
             │  Nginx/LB   │
             └──────┬──────┘
                    │
┌──────────────────────────────────────────────┐
│            Laravel API Gateway               │
├──────────────────────────────────────────────┤
│         Service Layer (Business Logic)       │
├────────────┬────────────┬───────────────────┤
│   ErpGo    │   Custom   │    Integration    │
│  Modules   │  Modules   │     Layer         │
└────────────┴────────────┴───────────────────┘
                    │
         ┌──────────┴──────────┐
         │                     │
    ┌────▼────┐         ┌─────▼─────┐
    │Postgres │         │   Redis   │
    │   DBs   │         │   Cache   │
    └─────────┘         └───────────┘
```

### 2.2 Module Architecture

```php
// Directory Structure
app/
├── Core/
│   ├── Tenant/           // Multi-tenancy
│   ├── Auth/             // Authentication (ErpGo 90% reuse)
│   ├── Permission/       // Authorization (ErpGo 85% reuse)
│   └── Common/           // Shared services
│
├── Modules/
│   ├── Order/            // Order management (ErpGo 40% + Custom 60%)
│   ├── Product/          // Product catalog (ErpGo 70% + Custom 30%)
│   ├── Printing/         // Printing workflow (100% Custom)
│   ├── Design/           // Design tools (100% Custom)
│   ├── Production/       // Production queue (100% Custom)
│   ├── Inventory/        // Stock management (ErpGo 50% + Custom 50%)
│   ├── Billing/          // Invoicing (ErpGo 75% + Custom 25%)
│   ├── CRM/              // Customer management (ErpGo 80% + Custom 20%)
│   ├── Communication/    // WhatsApp/Email (100% Custom)
│   └── Reports/          // Analytics (ErpGo 60% + Custom 40%)
│
├── Services/
│   ├── FileProcessor/    // File handling service
│   ├── PricingEngine/    // Dynamic pricing
│   ├── WorkflowEngine/   // State machine
│   └── NotificationService/
│
└── API/
    └── v1/
        ├── Controllers/
        ├── Resources/
        └── Middleware/
```

## 3. Database Design

### 3.1 Multi-Tenant Strategy
```sql
-- Master Database (shared)
CREATE DATABASE origin_alpha_master;

-- Tenant Databases (isolated)
CREATE DATABASE tenant_{tenant_id};

-- Master Tables
CREATE TABLE tenants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    domain VARCHAR(255) UNIQUE NOT NULL,
    database VARCHAR(255) NOT NULL,
    status ENUM('active', 'suspended', 'cancelled'),
    plan VARCHAR(50),
    created_at TIMESTAMP,
    expires_at TIMESTAMP
);

CREATE TABLE tenant_users (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenants(id),
    email VARCHAR(255) UNIQUE NOT NULL,
    role VARCHAR(50)
);
```

### 3.2 Core Schema (Per Tenant)

#### Users & Auth (From ErpGo)
```sql
-- Reuse ErpGo schema with modifications
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20),
    password VARCHAR(255),
    language VARCHAR(10) DEFAULT 'en',
    organization_id BIGINT,
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

CREATE TABLE roles (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL,
    guard_name VARCHAR(50),
    created_at TIMESTAMP
);

CREATE TABLE permissions (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) UNIQUE NOT NULL,
    guard_name VARCHAR(50),
    module VARCHAR(50),
    created_at TIMESTAMP
);
```

#### Orders & Workflow (Custom + ErpGo Hybrid)
```sql
CREATE TABLE orders (
    id BIGSERIAL PRIMARY KEY,
    order_number VARCHAR(50) UNIQUE NOT NULL,
    customer_id BIGINT REFERENCES customers(id),
    organization_id BIGINT,
    status VARCHAR(50) NOT NULL DEFAULT 'submitted',
    priority INTEGER DEFAULT 5,
    deadline TIMESTAMP,
    total_amount DECIMAL(10,2),
    paid_amount DECIMAL(10,2) DEFAULT 0,
    notes TEXT,
    metadata JSONB,
    created_by BIGINT REFERENCES users(id),
    assigned_to BIGINT REFERENCES users(id),
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

CREATE TABLE order_items (
    id BIGSERIAL PRIMARY KEY,
    order_id BIGINT REFERENCES orders(id),
    product_id BIGINT REFERENCES products(id),
    variant_id BIGINT,
    quantity INTEGER NOT NULL,
    unit_price DECIMAL(10,2),
    discount DECIMAL(10,2) DEFAULT 0,
    tax DECIMAL(10,2) DEFAULT 0,
    total DECIMAL(10,2),
    specifications JSONB,
    created_at TIMESTAMP
);

CREATE TABLE order_workflow_history (
    id BIGSERIAL PRIMARY KEY,
    order_id BIGINT REFERENCES orders(id),
    from_status VARCHAR(50),
    to_status VARCHAR(50),
    user_id BIGINT REFERENCES users(id),
    notes TEXT,
    created_at TIMESTAMP
);
```

#### Products & Pricing (ErpGo Enhanced)
```sql
CREATE TABLE products (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    sku VARCHAR(100) UNIQUE,
    category_id BIGINT,
    type ENUM('standard', 'variant', 'bundle', 'custom'),
    base_price DECIMAL(10,2),
    pricing_type ENUM('fixed', 'per_sqft', 'per_piece', 'custom'),
    description TEXT,
    specifications JSONB,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

CREATE TABLE product_variants (
    id BIGSERIAL PRIMARY KEY,
    product_id BIGINT REFERENCES products(id),
    name VARCHAR(255),
    attributes JSONB,
    price_modifier DECIMAL(10,2),
    sku VARCHAR(100) UNIQUE,
    stock_quantity INTEGER DEFAULT 0,
    created_at TIMESTAMP
);

CREATE TABLE product_bundles (
    id BIGSERIAL PRIMARY KEY,
    bundle_id BIGINT REFERENCES products(id),
    product_id BIGINT REFERENCES products(id),
    quantity INTEGER DEFAULT 1,
    created_at TIMESTAMP
);

CREATE TABLE seasonal_discounts (
    id BIGSERIAL PRIMARY KEY,
    product_id BIGINT REFERENCES products(id),
    discount_percent DECIMAL(5,2),
    start_date DATE,
    end_date DATE,
    code VARCHAR(50),
    created_at TIMESTAMP
);
```

#### File Management (Custom)
```sql
CREATE TABLE files (
    id BIGSERIAL PRIMARY KEY,
    order_id BIGINT REFERENCES orders(id),
    type ENUM('design', 'reference', 'proof', 'final'),
    original_name VARCHAR(255),
    stored_name VARCHAR(255),
    mime_type VARCHAR(100),
    size_bytes BIGINT,
    path TEXT,
    cdn_url TEXT,
    metadata JSONB,
    uploaded_by BIGINT REFERENCES users(id),
    created_at TIMESTAMP
);

CREATE TABLE file_versions (
    id BIGSERIAL PRIMARY KEY,
    file_id BIGINT REFERENCES files(id),
    version_number INTEGER,
    changes TEXT,
    created_by BIGINT REFERENCES users(id),
    created_at TIMESTAMP
);

CREATE TABLE annotations (
    id BIGSERIAL PRIMARY KEY,
    file_id BIGINT REFERENCES files(id),
    user_id BIGINT REFERENCES users(id),
    type ENUM('comment', 'highlight', 'arrow', 'text'),
    coordinates JSONB,
    content TEXT,
    status ENUM('open', 'resolved'),
    created_at TIMESTAMP
);
```

#### Production Queue (Custom)
```sql
CREATE TABLE production_queue (
    id BIGSERIAL PRIMARY KEY,
    order_id BIGINT REFERENCES orders(id),
    priority INTEGER DEFAULT 5,
    machine_id BIGINT,
    operator_id BIGINT REFERENCES users(id),
    status ENUM('pending', 'in_progress', 'completed', 'on_hold'),
    scheduled_at TIMESTAMP,
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    notes TEXT,
    created_at TIMESTAMP
);

CREATE TABLE production_resources (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255),
    type ENUM('machine', 'material', 'human'),
    capacity JSONB,
    availability JSONB,
    created_at TIMESTAMP
);
```

## 4. API Specification

### 4.1 Authentication Endpoints
```yaml
POST /api/v1/auth/register
POST /api/v1/auth/login
POST /api/v1/auth/logout
POST /api/v1/auth/refresh
POST /api/v1/auth/forgot-password
POST /api/v1/auth/reset-password
GET  /api/v1/auth/me
```

### 4.2 Order Management Endpoints
```yaml
# Orders
GET    /api/v1/orders
POST   /api/v1/orders
GET    /api/v1/orders/{id}
PUT    /api/v1/orders/{id}
DELETE /api/v1/orders/{id}

# Order Workflow
POST   /api/v1/orders/{id}/claim
POST   /api/v1/orders/{id}/status
GET    /api/v1/orders/{id}/history
POST   /api/v1/orders/{id}/priority

# Order Files
POST   /api/v1/orders/{id}/files
GET    /api/v1/orders/{id}/files
DELETE /api/v1/orders/{id}/files/{fileId}
```

### 4.3 Product Management Endpoints
```yaml
# Products
GET    /api/v1/products
POST   /api/v1/products
GET    /api/v1/products/{id}
PUT    /api/v1/products/{id}
DELETE /api/v1/products/{id}

# Variants
GET    /api/v1/products/{id}/variants
POST   /api/v1/products/{id}/variants
PUT    /api/v1/products/{id}/variants/{variantId}

# Pricing
POST   /api/v1/products/{id}/calculate-price
GET    /api/v1/products/{id}/discounts
POST   /api/v1/products/{id}/discounts
```

### 4.4 Design Review Endpoints
```yaml
# Design Files
POST   /api/v1/designs/upload
GET    /api/v1/designs/{id}
GET    /api/v1/designs/{id}/preview
POST   /api/v1/designs/{id}/convert

# Annotations
GET    /api/v1/designs/{id}/annotations
POST   /api/v1/designs/{id}/annotations
PUT    /api/v1/designs/{id}/annotations/{annotationId}
DELETE /api/v1/designs/{id}/annotations/{annotationId}

# Approval
POST   /api/v1/designs/{id}/approve
POST   /api/v1/designs/{id}/reject
GET    /api/v1/designs/{id}/revisions
```

### 4.5 Communication Endpoints
```yaml
# Notifications
POST   /api/v1/notifications/send
GET    /api/v1/notifications/templates
POST   /api/v1/notifications/templates

# WhatsApp
POST   /api/v1/whatsapp/send
POST   /api/v1/whatsapp/webhook
GET    /api/v1/whatsapp/status/{messageId}
```

## 5. Service Layer Implementation

### 5.1 File Processing Service
```php
namespace App\Services;

class FileProcessorService
{
    private ImageManager $imageManager;
    private StorageService $storage;
    
    public function processUpload(UploadedFile $file, Order $order): File
    {
        // 1. Validate file
        $this->validateFile($file);
        
        // 2. Store original
        $path = $this->storage->store($file, "orders/{$order->id}/originals");
        
        // 3. Generate preview
        if ($this->isImageFile($file)) {
            $preview = $this->generatePreview($file);
            $previewPath = $this->storage->store($preview, "orders/{$order->id}/previews");
        }
        
        // 4. Apply watermark
        if ($preview) {
            $watermarked = $this->applyWatermark($preview);
        }
        
        // 5. Upload to CDN
        $cdnUrl = $this->uploadToCdn($watermarked ?? $file);
        
        // 6. Create database record
        return File::create([
            'order_id' => $order->id,
            'original_name' => $file->getClientOriginalName(),
            'stored_name' => $path,
            'mime_type' => $file->getMimeType(),
            'size_bytes' => $file->getSize(),
            'cdn_url' => $cdnUrl,
            'metadata' => $this->extractMetadata($file)
        ]);
    }
    
    public function convertTiffToJpg(string $tiffPath): string
    {
        $image = $this->imageManager->make($tiffPath);
        $jpgPath = str_replace('.tiff', '.jpg', $tiffPath);
        
        $image->encode('jpg', 90)->save($jpgPath);
        
        return $jpgPath;
    }
}
```

### 5.2 Workflow Engine Service
```php
namespace App\Services;

use App\Models\Order;
use App\Events\OrderStatusChanged;

class WorkflowEngineService
{
    private array $transitions = [
        'submitted' => ['queued'],
        'queued' => ['claimed'],
        'claimed' => ['designing'],
        'designing' => ['review'],
        'review' => ['approved', 'corrections'],
        'corrections' => ['designing'],
        'approved' => ['production'],
        'production' => ['completed'],
        'completed' => ['delivered'],
        'delivered' => ['invoiced'],
        'invoiced' => ['paid']
    ];
    
    public function transitionTo(Order $order, string $newStatus, User $user): bool
    {
        if (!$this->canTransition($order->status, $newStatus)) {
            throw new InvalidTransitionException();
        }
        
        DB::transaction(function () use ($order, $newStatus, $user) {
            // Record history
            OrderWorkflowHistory::create([
                'order_id' => $order->id,
                'from_status' => $order->status,
                'to_status' => $newStatus,
                'user_id' => $user->id,
                'created_at' => now()
            ]);
            
            // Update order
            $order->update(['status' => $newStatus]);
            
            // Trigger events
            event(new OrderStatusChanged($order, $newStatus));
            
            // Send notifications
            $this->sendStatusNotification($order, $newStatus);
        });
        
        return true;
    }
    
    public function canTransition(string $from, string $to): bool
    {
        return in_array($to, $this->transitions[$from] ?? []);
    }
}
```

### 5.3 Pricing Engine Service
```php
namespace App\Services;

class PricingEngineService
{
    public function calculatePrice(Product $product, array $specifications): float
    {
        $basePrice = $product->base_price;
        
        switch ($product->pricing_type) {
            case 'per_sqft':
                $area = $specifications['width'] * $specifications['height'];
                $price = $basePrice * $area;
                break;
                
            case 'per_piece':
                $quantity = $specifications['quantity'] ?? 1;
                $price = $basePrice * $quantity;
                break;
                
            case 'custom':
                $price = $this->calculateCustomPrice($product, $specifications);
                break;
                
            default:
                $price = $basePrice;
        }
        
        // Apply variant modifiers
        if (isset($specifications['variant_id'])) {
            $variant = ProductVariant::find($specifications['variant_id']);
            $price += $variant->price_modifier;
        }
        
        // Apply seasonal discounts
        $discount = $this->getApplicableDiscount($product);
        if ($discount) {
            $price *= (1 - $discount->discount_percent / 100);
        }
        
        return round($price, 2);
    }
}
```

### 5.4 Notification Service
```php
namespace App\Services;

use Twilio\Rest\Client as TwilioClient;

class NotificationService
{
    private TwilioClient $twilio;
    private MailService $mail;
    
    public function sendOrderNotification(Order $order, string $event): void
    {
        $customer = $order->customer;
        $template = $this->getTemplate($event);
        
        // Try WhatsApp first
        if ($customer->whatsapp_enabled) {
            try {
                $this->sendWhatsApp($customer->phone, $template, $order);
            } catch (\Exception $e) {
                Log::error('WhatsApp failed: ' . $e->getMessage());
                $this->sendEmail($customer->email, $template, $order);
            }
        } else {
            $this->sendEmail($customer->email, $template, $order);
        }
    }
    
    private function sendWhatsApp(string $phone, Template $template, Order $order): void
    {
        $message = $this->twilio->messages->create(
            "whatsapp:$phone",
            [
                'from' => config('services.twilio.whatsapp_from'),
                'body' => $this->parseTemplate($template, $order)
            ]
        );
    }
}
```

## 6. Frontend Architecture

### 6.1 React Component Structure
```typescript
// src/components/
components/
├── Layout/
│   ├── Header.tsx
│   ├── Sidebar.tsx
│   └── Footer.tsx
├── Order/
│   ├── OrderList.tsx
│   ├── OrderDetails.tsx
│   ├── OrderForm.tsx
│   └── OrderWorkflow.tsx
├── Product/
│   ├── ProductCatalog.tsx
│   ├── ProductForm.tsx
│   └── PricingCalculator.tsx
├── Design/
│   ├── DesignCanvas.tsx
│   ├── AnnotationTool.tsx
│   └── VersionHistory.tsx
├── Common/
│   ├── DataTable.tsx
│   ├── FileUploader.tsx
│   └── NotificationToast.tsx
└── hooks/
    ├── useAuth.ts
    ├── useApi.ts
    └── useTenant.ts
```

### 6.2 State Management (Redux Toolkit)
```typescript
// src/store/slices/orderSlice.ts
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

export const fetchOrders = createAsyncThunk(
    'orders/fetch',
    async (params: OrderParams) => {
        const response = await api.get('/orders', { params });
        return response.data;
    }
);

const orderSlice = createSlice({
    name: 'orders',
    initialState: {
        items: [],
        loading: false,
        error: null,
        filters: {},
        pagination: {}
    },
    reducers: {
        setFilter: (state, action) => {
            state.filters = action.payload;
        },
        clearFilters: (state) => {
            state.filters = {};
        }
    },
    extraReducers: (builder) => {
        builder
            .addCase(fetchOrders.pending, (state) => {
                state.loading = true;
            })
            .addCase(fetchOrders.fulfilled, (state, action) => {
                state.items = action.payload.data;
                state.pagination = action.payload.meta;
                state.loading = false;
            })
            .addCase(fetchOrders.rejected, (state, action) => {
                state.error = action.error.message;
                state.loading = false;
            });
    }
});
```

### 6.3 Design Canvas Component
```typescript
// src/components/Design/DesignCanvas.tsx
import React, { useState, useRef } from 'react';
import { Stage, Layer, Image, Circle, Text } from 'react-konva';

interface Annotation {
    id: string;
    type: 'comment' | 'highlight';
    x: number;
    y: number;
    content: string;
}

export const DesignCanvas: React.FC<{ fileUrl: string }> = ({ fileUrl }) => {
    const [annotations, setAnnotations] = useState<Annotation[]>([]);
    const [selectedTool, setSelectedTool] = useState<'comment' | 'highlight'>('comment');
    
    const handleStageClick = (e: any) => {
        const pos = e.target.getStage().getPointerPosition();
        
        if (selectedTool === 'comment') {
            const newAnnotation: Annotation = {
                id: Date.now().toString(),
                type: 'comment',
                x: pos.x,
                y: pos.y,
                content: ''
            };
            
            setAnnotations([...annotations, newAnnotation]);
        }
    };
    
    return (
        <div className="design-canvas">
            <Stage width={800} height={600} onClick={handleStageClick}>
                <Layer>
                    <Image src={fileUrl} />
                    {annotations.map((ann) => (
                        <Circle
                            key={ann.id}
                            x={ann.x}
                            y={ann.y}
                            radius={10}
                            fill="red"
                        />
                    ))}
                </Layer>
            </Stage>
            
            <div className="annotation-tools">
                <button onClick={() => setSelectedTool('comment')}>
                    Comment
                </button>
                <button onClick={() => setSelectedTool('highlight')}>
                    Highlight
                </button>
            </div>
        </div>
    );
};
```

## 7. Security Implementation

### 7.1 Authentication & Authorization
```php
// Multi-tenant authentication middleware
namespace App\Http\Middleware;

class TenantAuthentication
{
    public function handle(Request $request, Closure $next)
    {
        $tenant = $this->identifyTenant($request);
        
        if (!$tenant) {
            return response()->json(['error' => 'Tenant not found'], 404);
        }
        
        // Switch database connection
        config(['database.connections.tenant.database' => $tenant->database]);
        DB::purge('tenant');
        DB::reconnect('tenant');
        
        // Set tenant in container
        app()->instance('tenant', $tenant);
        
        return $next($request);
    }
    
    private function identifyTenant(Request $request): ?Tenant
    {
        // Try subdomain
        $host = $request->getHost();
        $subdomain = explode('.', $host)[0];
        
        return Tenant::where('domain', $subdomain)->first();
    }
}
```

### 7.2 API Rate Limiting
```php
// config/rate-limiting.php
RateLimiter::for('api', function (Request $request) {
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});

RateLimiter::for('uploads', function (Request $request) {
    return Limit::perMinute(10)->by($request->user()->id);
});
```

### 7.3 Data Encryption
```php
// Encrypt sensitive data
class EncryptedCast implements CastsAttributes
{
    public function get($model, $key, $value, $attributes)
    {
        return $value ? decrypt($value) : null;
    }
    
    public function set($model, $key, $value, $attributes)
    {
        return $value ? encrypt($value) : null;
    }
}

// Usage in model
protected $casts = [
    'payment_details' => EncryptedCast::class,
    'api_keys' => EncryptedCast::class,
];
```

## 8. Performance Optimization

### 8.1 Database Optimization
```sql
-- Indexes for common queries
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_customer ON orders(customer_id);
CREATE INDEX idx_orders_deadline ON orders(deadline);
CREATE INDEX idx_files_order ON files(order_id);
CREATE INDEX idx_workflow_order ON order_workflow_history(order_id);

-- Composite indexes
CREATE INDEX idx_orders_status_priority ON orders(status, priority);
CREATE INDEX idx_products_active_category ON products(is_active, category_id);
```

### 8.2 Caching Strategy
```php
// Cache commonly accessed data
Cache::remember('products:catalog', 3600, function () {
    return Product::active()->with('variants')->get();
});

Cache::tags(['orders', "user:{$userId}"])->remember(
    "user:{$userId}:orders",
    600,
    function () use ($userId) {
        return Order::where('customer_id', $userId)->latest()->take(10)->get();
    }
);
```

### 8.3 Queue Processing
```php
// Async job for file processing
class ProcessUploadedFile implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    
    public function __construct(
        private File $file,
        private Order $order
    ) {}
    
    public function handle(FileProcessorService $processor): void
    {
        $processor->process($this->file, $this->order);
    }
}
```

## 9. Testing Strategy

### 9.1 Unit Tests
```php
// tests/Unit/Services/PricingEngineTest.php
class PricingEngineTest extends TestCase
{
    public function test_calculates_per_sqft_pricing()
    {
        $product = Product::factory()->create([
            'pricing_type' => 'per_sqft',
            'base_price' => 50
        ]);
        
        $service = new PricingEngineService();
        $price = $service->calculatePrice($product, [
            'width' => 10,
            'height' => 5
        ]);
        
        $this->assertEquals(2500, $price);
    }
}
```

### 9.2 Integration Tests
```php
// tests/Feature/OrderWorkflowTest.php
class OrderWorkflowTest extends TestCase
{
    public function test_order_workflow_transitions()
    {
        $order = Order::factory()->create(['status' => 'submitted']);
        
        $response = $this->actingAs($this->staff)
            ->post("/api/v1/orders/{$order->id}/claim");
        
        $response->assertOk();
        $this->assertEquals('claimed', $order->fresh()->status);
    }
}
```

## 10. Deployment Configuration

### 10.1 Docker Compose
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:80"
    environment:
      - DB_CONNECTION=pgsql
      - DB_HOST=postgres
      - REDIS_HOST=redis
    depends_on:
      - postgres
      - redis
      
  postgres:
    image: postgres:15
    environment:
      - POSTGRES_PASSWORD=secret
    volumes:
      - postgres_data:/var/lib/postgresql/data
      
  redis:
    image: redis:7
    
  horizon:
    build: .
    command: php artisan horizon
    depends_on:
      - redis
      
volumes:
  postgres_data:
```

### 10.2 Environment Configuration
```env
# .env.production
APP_NAME="Origin Alpha"
APP_ENV=production
APP_DEBUG=false
APP_URL=https://app.originalpha.com

DB_CONNECTION=pgsql
DB_HOST=localhost
DB_PORT=5432
DB_DATABASE=origin_alpha_master

REDIS_HOST=localhost
REDIS_PORT=6379

FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=xxx
AWS_SECRET_ACCESS_KEY=xxx
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=origin-alpha

TWILIO_ACCOUNT_SID=xxx
TWILIO_AUTH_TOKEN=xxx
TWILIO_WHATSAPP_FROM=whatsapp:+xxx

MAIL_MAILER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
```

---

*Technical Specification Complete. Ready for development implementation.*
