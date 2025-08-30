# Laravel Project Development Guide

## Planning Phase

### Database Schema Design

1. Start with brain storming and draw your business schema in [dbdiagram.io](https://dbdiagram.io)
2. Keep your schema up-to-date with any modifications throughout development
3. Consider relationships between entities carefully (one-to-many, many-to-many, etc.)
4. Plan for indexes and performance considerations early

## Environment Setup

### PHP and Composer Installation

1. Install PHP from [php.net/downloads](https://www.php.net/downloads.php)
   - Recommended version: PHP 8.2+ for Laravel 12.x
   - Enable required extensions: BCMath, Ctype, Fileinfo, JSON, Mbstring, OpenSSL, PDO, Tokenizer, XML

2. Install Composer from [getcomposer.org/download](https://getcomposer.org/download/)
   - Verify installation with: `composer --version`

3. Install Laravel Installer globally
   composer global require laravel/installer
   - Ensure Composer's global bin directory is in your PATH

## Project Creation

### Starting a New Laravel Project

Use the Laravel Installer:

```
laravel new project-name
```

Alternatively, use Composer directly:

```
composer create-project laravel/laravel project-name
```

### Popular Laravel Stacks and Packages

- **Livewire**: For reactive frontend components without writing JavaScript
- **Laravel Volt**: Write Livewire components using a single-file component syntax
- **Pest**: Modern testing framework with a focus on simplicity

## Frontend Setup

### Installing Tailwind CSS

1. Install Tailwind CSS and Vite plugin

   ```
   npm install -D tailwindcss postcss autoprefixer @tailwindcss/vite
   ```

2. Initialize Tailwind configuration

   ```
   npx tailwindcss init -p
   ```

3. Configure Vite Plugin in `vite.config.js`

   ```javascript
   import { defineConfig } from 'vite';
   import laravel from 'laravel-vite-plugin';
   import tailwindcss from '@tailwindcss/vite';

   export default defineConfig({
     plugins: [
       laravel({
         input: ['resources/css/app.css', 'resources/js/app.js'],
         refresh: true,
       }),
       tailwindcss(),
     ],
   });
   ```

4. Update your `tailwind.config.js`

   ```javascript
   /** @type {import('tailwindcss').Config} */
   export default {
     content: [
       "./resources/**/*.blade.php",
       "./resources/**/*.js",
       "./resources/**/*.vue",
       "./vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php",
     ],
     theme: {
       extend: {},
     },
     plugins: [],
   }
   ```

5. Update your CSS in `resources/css/app.css`

   ```css
   @tailwind base;
   @tailwind components;
   @tailwind utilities;
   ```

6. Include Vite in your layout file

   ```html
   <head>
       <!-- ... other head elements ... -->
       @vite('resources/css/app.css')
   </head>
   ```

## Development Workflow

### Create a Development Script

Create a `run.bat` file (Windows) or `run.sh` (Linux/Mac) to start both the Laravel server and Vite:

#### run.bat (Windows)

```batch
start cmd /k "php artisan serve"
start cmd /k "npm run dev"
```

### Version Control Setup

```
git init
git add .
git commit -m 'Initial commit'
```

Create a `.gitignore` file (Laravel includes a good default one)

## Database Development

### Creating Migrations

```
php artisan make:migration create_table_name_table
```

Example migration structure:

```php
public function up(): void
{
    Schema::create('table_name', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->text('description')->nullable();
        $table->foreignId('user_id')->constrained();
        $table->timestamps();
    });
}
```

### Running Migrations

```
php artisan migrate
```

Rollback options:

```
php artisan migrate:rollback        # Rollback the last batch
php artisan migrate:reset           # Rollback all migrations
php artisan migrate:refresh         # Reset and re-run all migrations
php artisan migrate:fresh           # Drop all tables and re-run all migrations
```

## Model and Controller Development

### Creating Models

#### Single Components

```
php artisan make:model ModelName                 # Just the model
php artisan make:controller ModelNameController  # Just the controller
```

#### Bundled Generation

```
php artisan make:model ModelName -mcr  # Create model, controller with resource methods, and migration
php artisan make:model ModelName -a    # Create model with all related components (migration, factory, seeder, controller, policy)
```

### Bulk Model Generation

You can create a route to generate multiple models at once:

```php
Route::get('init', function () {
    $models = [
        'Product',
        'Category',
        'Order',
        // Add all your schema tables here
    ];
    
    foreach ($models as $model) {
        Artisan::call('make:model', ['name' => $model, '-a' => true]);
        sleep(1); // Prevent potential issues with rapid generation
    }
    
    return 'Models generated successfully!';
});
```

**Note**: Skip 'User' as it already exists in Laravel by default.

## Relationships and Eloquent

### Common Relationship Types

```php
// One-to-One
public function profile()
{
    return $this->hasOne(Profile::class);
}

// One-to-Many
public function posts()
{
    return $this->hasMany(Post::class);
}

// Many-to-Many
public function roles()
{
    return $this->belongsToMany(Role::class);
}

// Has-Many-Through
public function comments()
{
    return $this->hasManyThrough(Comment::class, Post::class);
}
//Morph-to-Many
public function tags()
{
    return $this->morphMany(Tag::class, 'taggable');
}
```


## Authentication and Authorization

### Setting Up Authentication

```
composer require laravel/ui  # For Laravel 6.0+
php artisan ui bootstrap --auth
```

For newer Laravel versions, consider using:

```
composer require laravel/breeze --dev
php artisan breeze:install blade
```

### Creating Policies

```
php artisan make:policy PostPolicy --model=Post
```

## Testing

### Creating Tests

```
php artisan make:test UserTest       # Feature test
php artisan make:test UserTest --unit # Unit test
```

### Running Tests

```
php artisan test
php artisan test --filter=UserTest
```

## Deployment

### Optimization Commands

```
php artisan config:cache
php artisan route:cache
php artisan view:cache
composer install --optimize-autoloader --no-dev
npm run build
```

## Useful Resources

- [Laravel Artisan Commands](https://artisan.page/12.x/)
- [Laravel Documentation](https://laravel.com/docs/12.x)
- [Laravel News](https://laravel-news.com/)
- [Laracasts](https://laracasts.com/)
- [Laravel Daily](https://laraveldaily.com/)
- [Laravel Forge](https://forge.laravel.com/) (Deployment)
- [Laravel Vapor](https://vapor.laravel.com/) (Serverless Deployment)
- [Laravel Envoyer](https://envoyer.io/) (Zero Downtime Deployment)

## Common Packages

- **spatie/laravel-permission**: Role and permission management
- **laravel/sanctum**: API token authentication
- **barryvdh/laravel-debugbar**: Debugging toolbar
- **laravel/telescope**: Application debugging and monitoring
- **spatie/laravel-backup**: Backup solution
- **intervention/image**: Image manipulation
- **maatwebsite/excel**: Excel import/export

## Best Practices

1. Follow PSR-12 coding standards
2. Use repository pattern for complex data operations
3. Keep controllers thin, move business logic to services
4. Use form requests for validation
5. Implement proper error handling and logging
6. Write tests for critical functionality
7. Use Laravel's built-in security features (CSRF, XSS protection)
8. Optimize database queries with eager loading
9. Use environment variables for configuration
10. Implement proper caching strategies
