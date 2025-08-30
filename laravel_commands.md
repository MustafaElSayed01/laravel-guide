# Laravel Commands Cheat Sheet

## Artisan Commands

### Project Setup

```bash
# Create a new Laravel project
laravel new project-name

# Create a new Laravel project via Composer
composer create-project laravel/laravel project-name

# Start the development server
php artisan serve

# Clear application cache
php artisan cache:clear

# Clear configuration cache
php artisan config:clear

# Clear route cache
php artisan route:clear

# Clear view cache
php artisan view:clear

# Clear all caches at once
php artisan optimize:clear
```

### Database Commands

```bash
# Run migrations
php artisan migrate

# Rollback the last migration batch
php artisan migrate:rollback

# Rollback all migrations
php artisan migrate:reset

# Rollback all migrations and run them again
php artisan migrate:refresh

# Drop all tables and run migrations
php artisan migrate:fresh

# Create a new migration
php artisan make:migration create_table_name_table

# Run database seeds
php artisan db:seed

# Run a specific seeder
php artisan db:seed --class=UserSeeder

# Create a new seeder
php artisan make:seeder UserSeeder
```

### Model Commands

```bash
# Create a model
php artisan make:model ModelName

# Create a model with a migration
php artisan make:model ModelName -m

# Create a model with a controller
php artisan make:model ModelName -c

# Create a model with a resource controller
php artisan make:model ModelName -cr

# Create a model with migration, controller, factory, and seeder
php artisan make:model ModelName -a
```

### Controller Commands

```bash
# Create a controller
php artisan make:controller UserController

# Create a resource controller
php artisan make:controller UserController --resource

# Create an API controller
php artisan make:controller API/UserController --api

# Create a controller with model binding
php artisan make:controller UserController --model=User
```

### Middleware Commands

```bash
# Create a middleware
php artisan make:middleware CheckAge
```

### Policy Commands

```bash
# Create a policy
php artisan make:policy PostPolicy

# Create a policy for a model
php artisan make:policy PostPolicy --model=Post
```

### Request Commands

```bash
# Create a form request
php artisan make:request StorePostRequest
```

### Resource Commands

```bash
# Create a resource
php artisan make:resource UserResource

# Create a resource collection
php artisan make:resource Users --collection
```

### Event Commands

```bash
# Create an event
php artisan make:event UserRegistered

# Create a listener
php artisan make:listener SendWelcomeEmail --event=UserRegistered
```

### Job Commands

```bash
# Create a job
php artisan make:job ProcessPodcast

# Create a queued job
php artisan make:job ProcessPodcast --queued
```

### Mail Commands

```bash
# Create a mailable
php artisan make:mail OrderShipped

# Create a markdown mailable
php artisan make:mail OrderShipped --markdown=emails.orders.shipped
```

### Notification Commands

```bash
# Create a notification
php artisan make:notification InvoicePaid

# Create a markdown notification
php artisan make:notification InvoicePaid --markdown=notifications.invoice
```

### Provider Commands

```bash
# Create a service provider
php artisan make:provider RiakServiceProvider
```

### Testing Commands

```bash
# Create a test
php artisan make:test UserTest

# Create a unit test
php artisan make:test UserTest --unit

# Run tests
php artisan test

# Run a specific test
php artisan test --filter=UserTest
```

### Queue Commands

```bash
# Run the queue worker
php artisan queue:work

# Run the queue worker once
php artisan queue:work --once

# Run the queue worker and stop when the queue is empty
php artisan queue:work --stop-when-empty

# Restart the queue workers
php artisan queue:restart
```

### Tinker Commands

```bash
# Start Tinker (REPL)
php artisan tinker
```

### Storage Commands

```bash
# Create a symbolic link from public/storage to storage/app/public
php artisan storage:link
```

### Vendor Commands

```bash
# Publish vendor files/resources
php artisan vendor:publish

# Publish vendor files/resources for a specific provider
php artisan vendor:publish --provider="Vendor\Package\ServiceProvider"

# Publish vendor files/resources for a specific tag
php artisan vendor:publish --tag=config
```

## Composer Commands

```bash
# Install dependencies
composer install

# Update dependencies
composer update

# Install a package
composer require vendor/package

# Install a development package
composer require --dev vendor/package

# Remove a package
composer remove vendor/package

# Show installed packages
composer show

# Create autoload files
composer dump-autoload

# Create optimized autoload files
composer dump-autoload -o
```

## NPM Commands

```bash
# Install dependencies
npm install

# Update dependencies
npm update

# Install a package
npm install package-name

# Install a development package
npm install package-name --save-dev

# Run development build
npm run dev

# Run production build
npm run build

# Run development server with hot reloading
npm run hot
```

## Git Commands

```bash
# Initialize a Git repository
git init

# Add all files to staging
git add .

# Commit changes
git commit -m "Commit message"

# Add remote repository
git remote add origin https://github.com/username/repository.git

# Push to remote repository
git push -u origin main

# Create a new branch
git checkout -b branch-name

# Switch to a branch
git checkout branch-name

# Merge a branch
git merge branch-name

# Pull changes from remote repository
git pull origin main
```

## Deployment Commands

```bash
# Optimize for production
php artisan optimize

# Cache configuration
php artisan config:cache

# Cache routes
php artisan route:cache

# Cache views
php artisan view:cache

# Install production dependencies only
composer install --no-dev --optimize-autoloader

# Build assets for production
npm run build
