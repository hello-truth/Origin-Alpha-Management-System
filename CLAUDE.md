# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is the **product/design source of truth** for the Caldron Flex Printing Business platform. It contains specifications, architecture documentation, reference projects, and implementation plans - but **no actual application code**. The actual Laravel application should be built separately using "refer project 3" as the base.

## Architecture Overview

### Target Platform
- **Base**: Laravel 11+ (PHP 8.2+) using "refer project 3" structure from `reperence repos/refer project 3/`
- **Frontend**: React components with Inertia.js for server-side rendering
- **Database**: Single MySQL database (`caldron_flex_db`)
- **File Storage**: Local filesystem with CDN-ready paths
- **Queues**: Laravel queue system for background processing (file conversion, notifications)

### Core Systems
- **Pricing Engine**: Area-based calculations (width × height × rate) with modifiers and quantity discounts
- **File Pipeline**: 500MB chunked uploads, TIFF→JPG conversion, watermarking, thumbnails
- **Workflow Engine**: Order statuses (`pending|in_review|approved|revision_requested`), task assignment
- **WhatsApp Integration**: Proxy-based notifications with email fallback (rate limit: 100/hour)
- **Inventory Management**: Product catalog with dynamic pricing

### Package Structure
Follow modular architecture with printing functionality in:
```
packages/caldron/PrintingCore/
├── src/
│   ├── Models/           # PrintingProduct, PrintOrder, DesignFile, PrintTask
│   ├── Services/         # PricingEngineService, FileProcessingService, OrderWorkflowService
│   ├── Http/Controllers/ # API and web controllers
│   ├── Jobs/            # Background processing (file conversion, notifications)
│   ├── Middleware/      # Upload rate limiting, access control
│   └── routes/          # api.php, web.php
├── config/printing.php   # Package configuration
├── database/migrations/  # Printing-specific tables
└── resources/js/Components/Printing/ # React components
```

## Essential Documentation

### Start Here
1. **Architecture**: `.prd/architecture/SYSTEM_ARCHITECTURE.md` - System-level blueprints
2. **Software Spec**: `.prd/architecture/SOFTWARE_SPECIFICATION.md` - Technical requirements
3. **Implementation Plan**: `specs/caldron-flex-printing-system/tasks.md` - Authoritative task list with file paths
4. **Development Setup**: `.prd/setup/DEVELOPMENT_SETUP.md` - Environment configuration

### Reference Projects
- **Primary Base**: `reperence repos/refer project 3/main-file/` - Laravel foundation to build upon
- **Package Examples**: `reperence repos/workdo/` - Modular package structure patterns
- **DO NOT MODIFY** any files under `reperence repos/` - these are read-only references

## Common Development Commands

### Laravel Application (to be created from refer project 3)
```powershell
# Backend setup
composer install
php artisan key:generate
php artisan migrate --seed
php artisan serve --host=127.0.0.1 --port=8000

# Frontend (if using Vite)
npm install
npm run dev        # Development build
npm run build      # Production build

# Testing
php artisan test   # Run all tests
```

### Queue Processing
```powershell
php artisan queue:work    # Start queue worker for background jobs
```

### Package Development
```powershell
# After creating package structure
php artisan package:discover --ansi
```

## Environment Configuration

### Required Environment Variables
```env
# Database
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=caldron_flex_db
DB_USERNAME=caldron_user
DB_PASSWORD=secure_password

# WhatsApp Integration
WHATSAPP_PROXY_URL=https://your-proxy-url.com
WHATSAPP_API_KEY=your_api_key_here

# File Upload Support (500MB)
UPLOAD_MAX_FILESIZE=500M
POST_MAX_SIZE=500M
```

### PHP Extensions Required
- mysqli, curl, mbstring, intl, json, mysqlnd, xml, gd, zlib, pdo_mysql, openssl, fileinfo

## Implementation Workflow

### Task Execution Order
Follow the dependency chain in `specs/caldron-flex-printing-system/tasks.md`:
1. **Setup** (T001-T005): Initialize Laravel base and package structure
2. **Database** (T006-T010): Migrations, factories, seeders
3. **Models** (T017-T021): Core data models with relationships
4. **Services** (T026-T029): Business logic layer
5. **Controllers** (T034-T037): API endpoints and web controllers
6. **Frontend** (T042-T047): React components
7. **Integration** (T051-T065): File processing, notifications, middleware
8. **Testing** (T066-T069): Integration and E2E tests

### Development Patterns
- **Repository + Service Layer**: Use for all business logic (see SYSTEM_ARCHITECTURE.md §15.2)
- **TDD Approach**: Write tests before implementation for each phase
- **Parallel Tasks**: Tasks marked `[P]` can be developed simultaneously
- **File Paths**: Use exact paths specified in tasks.md - do not improvise

## Key Integration Points

### File Processing Pipeline
- Support chunked uploads up to 500MB
- Queue jobs: `ProcessUploadedChunk`, `FinalizeChunkedUpload`, `GenerateOrderPreview`
- File types: TIFF→JPG conversion, thumbnail generation, watermarking
- Storage structure: `orders/designs/temp/proofs` hierarchy

### WhatsApp Notifications
- Rate limit: 100 messages/hour
- Fallback to email if WhatsApp fails
- Queue job: `SendWhatsAppNotification`
- Configure proxy URL and API key in environment

### Pricing Engine
- Formula: `width × height × base_rate`
- Apply variant modifiers and quantity discounts
- Return detailed breakdown for transparency

## Testing Strategy

### Test Structure
- **Unit Tests**: `tests/Unit/Models/`, `tests/Unit/Services/`
- **Feature Tests**: `tests/Feature/Api/`, `tests/Feature/Integration/`
- **Frontend Tests**: `tests/Frontend/Components/`

### Test Commands
```powershell
php artisan test                    # Run all backend tests
php artisan test --filter=Unit      # Unit tests only
php artisan test --filter=Feature   # Feature tests only
```

## Critical Rules

1. **DO NOT** modify anything under `reperence repos/` - these are read-only references
2. **DO NOT** build application code in this repository - use this as specification source only
3. **ALWAYS** follow exact file paths from `specs/caldron-flex-printing-system/tasks.md`
4. **ALWAYS** use Repository + Service Layer patterns per architecture docs
5. **ALWAYS** write tests before implementation (TDD approach)
6. **NEVER** hard-code file storage paths - keep CDN-ready
7. **NEVER** skip WhatsApp rate limiting or email fallback configuration

## Troubleshooting

### Common Issues
- **Package not discovered**: Run `php artisan package:discover --ansi`
- **Queue jobs not processing**: Start worker with `php artisan queue:work`
- **File upload failures**: Check PHP limits and directory permissions
- **WhatsApp integration issues**: Verify proxy URL and API key configuration

### Log Locations
- Laravel: `storage/logs/laravel.log`
- Queue failures: `storage/logs/` and failed_jobs table
- File processing: Custom logs in printing package