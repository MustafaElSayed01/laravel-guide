# Laravel Troubleshooting Guide

## Database Connection Issues

### Connection Refused

**Issue**: Laravel can't connect to the database.

**Solution**:
1. Check database credentials in .env file
2. Ensure database server is running
3. Check if the database exists
4. Verify that the database user has proper permissions

### Migration Issues

**Issue**: Migrations fail to run.

**Solution**:
1. Check database connection
2. Ensure migrations are properly formatted
3. Try running with `--force` flag if in production
4. Check for syntax errors in migration files

## Route Issues

### Route Not Found (404)

**Issue**: Route returns 404 Not Found error.

**Solution**:
1. Check if the route is defined in routes file
2. Run `php artisan route:list` to verify routes
3. Clear route cache: `php artisan route:clear`
4. Check for typos in route definition
5. Ensure the controller and method exist

### Method Not Allowed (405)

**Issue**: Route returns 405 Method Not Allowed error.

**Solution**:
1. Ensure you're using the correct HTTP method (GET, POST, etc.)
2. Check if the route is defined with the correct method
3. For forms, ensure the form method matches the route method

## Artisan Command Issues

### Command Not Found

**Issue**: Artisan command not found.

**Solution**:
1. Check if the command is registered in `app/Console/Kernel.php`
2. Run `php artisan list` to see available commands
3. Clear cache: `php artisan optimize:clear`

## View Issues

### View Not Found

**Issue**: View file not found error.

**Solution**:
1. Check if the view file exists in the resources/views directory
2. Ensure the view name is correct in the controller
3. Check for typos in the view name
4. Clear view cache: `php artisan view:clear`

### Blade Syntax Errors

**Issue**: Blade template syntax errors.

**Solution**:
1. Check for unclosed Blade directives (@if without @endif, etc.)
2. Ensure proper syntax for Blade directives
3. Clear view cache: `php artisan view:clear`

## Authentication Issues

### Login Not Working

**Issue**: Users can't log in.

**Solution**:
1. Check if the user exists in the database
2. Verify password hashing is working correctly
3. Check for middleware issues
4. Ensure session configuration is correct

### Password Reset Not Working

**Issue**: Password reset emails not being sent.

**Solution**:
1. Check mail configuration in .env file
2. Ensure mail driver is set up correctly
3. Check for SMTP issues if using SMTP
4. Verify that the user exists in the database

## Performance Issues

### Slow Application

**Issue**: Laravel application is slow.

**Solution**:
1. Enable caching:
   ```bash
   php artisan config:cache
   php artisan route:cache
   php artisan view:cache
   ```
2. Optimize Composer autoloader:
   ```bash
   composer dump-autoload -o
   ```
3. Use eager loading for Eloquent relationships
4. Implement database indexing
5. Use query caching
6. Consider using Laravel Octane for production

### Memory Limit Exceeded

**Issue**: PHP memory limit exceeded.

**Solution**:
1. Increase memory limit in php.ini
2. Optimize database queries
3. Use chunking for large datasets
4. Implement pagination

## Deployment Issues

### 500 Server Error in Production

**Issue**: Application works locally but shows 500 error in production.

**Solution**:
1. Check server error logs
2. Ensure .env file exists in production
3. Set proper permissions for storage and bootstrap/cache directories
4. Verify that all required PHP extensions are installed
5. Run `php artisan optimize` for production

### Assets Not Loading

**Issue**: CSS, JS, or images not loading in production.

**Solution**:
1. Check if assets are properly compiled: `npm run build`
2. Ensure asset paths are correct
3. Check for mixed content issues (http vs https)
4. Verify that the web server is configured to serve static files

## Package Issues

### Package Conflicts

**Issue**: Composer package conflicts.

**Solution**:
1. Check for version constraints in composer.json
2. Try updating dependencies: `composer update`
3. Use `composer why-not package/name` to see conflicts
4. Consider using a different package or version

### Package Not Working

**Issue**: Installed package not working as expected.

**Solution**:
1. Check if the package is properly installed
2. Verify that the package is compatible with your Laravel version
3. Check if the package requires additional configuration
4. Look for package-specific troubleshooting in its documentation

## Queue Issues

### Queue Worker Not Processing Jobs

**Issue**: Queue jobs not being processed.

**Solution**:
1. Ensure queue worker is running: `php artisan queue:work`
2. Check queue driver configuration in .env file
3. Verify that the queue table exists if using database driver
4. Check for errors in the failed_jobs table

### Failed Jobs

**Issue**: Jobs failing to process.

**Solution**:
1. Check failed_jobs table for error messages
2. Increase timeout if jobs are timing out
3. Ensure job classes are properly defined
4. Check for dependencies required by the job

## Debugging Techniques

### Using Laravel Debugbar

Install Laravel Debugbar for development:
```bash
composer require barryvdh/laravel-debugbar --dev
```

This adds a debug bar to your application with information about queries, request, session, etc.

### Using Laravel Telescope

Install Laravel Telescope for more advanced debugging:
```bash
composer require laravel/telescope --dev
php artisan telescope:install
php artisan migrate
```

Access Telescope at `/telescope` route.

### Logging

Use Laravel's logging system to debug issues:
```php
Log::info('Debug message');
Log::error('Error message', ['context' => $data]);
```

Check logs in `storage/logs/laravel.log`.

### Dump and Die

Use `dd()` or `dump()` helpers to inspect variables:
```php
dd($variable); // Dump and die
dump($variable); // Dump and continue
```

## Common Error Messages and Solutions

### "No application encryption key has been specified"

**Solution**: Generate application key:
```bash
php artisan key:generate
```

### "The only supported ciphers are AES-128-CBC and AES-256-CBC"

**Solution**: Check that the cipher in config/app.php matches the one used for existing encrypted data.

### "Class 'App\Models\User' not found"

**Solution**:
1. Check namespace in the User model
2. Run `composer dump-autoload`
3. Ensure the model exists in the correct directory

### "SQLSTATE[HY000] [1045] Access denied for user"

**Solution**: Check database credentials in .env file.

### "SQLSTATE[42S02]: Base table or view not found"

**Solution**: Run migrations to create the missing table:
```bash
php artisan migrate
```

### "The POST method is not supported for this route"

**Solution**: Ensure the route is defined with the correct HTTP method.

### "Target class controller does not exist"

**Solution**:
1. Check controller namespace
2. Ensure controller exists in the correct directory
3. Run `composer dump-autoload`

## Preventive Measures

1. **Always backup before major changes**
2. **Use version control (Git)**
3. **Write tests for critical functionality**
4. **Keep Laravel and packages updated**
5. **Monitor application logs**
6. **Use Laravel Horizon for queue monitoring in production**
7. **Implement proper error handling**
8. **Set up proper logging and monitoring**