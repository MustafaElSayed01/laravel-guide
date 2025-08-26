# Laravel Best Practices

## Code Organization

### Directory Structure

- Follow Laravel's default directory structure
- Create subdirectories for complex features
- Group related files together

### Naming Conventions

- **Controllers**: Singular, PascalCase, suffixed with `Controller` (e.g., `UserController`)
- **Models**: Singular, PascalCase (e.g., `User`, `Product`)
- **Tables**: Plural, snake_case (e.g., `users`, `product_categories`)
- **Migrations**: Snake_case, descriptive (e.g., `create_users_table`, `add_role_to_users`)
- **Routes**: Kebab-case for URLs (e.g., `/user-profile`)
- **Methods**: camelCase (e.g., `getUsers()`, `createOrder()`)
- **Variables**: camelCase (e.g., `$userCount`, `$orderItems`)

## Database

### Migrations

- Use migrations for all database changes
- Keep migrations small and focused
- Include both up() and down() methods
- Use foreign key constraints
- Add indexes for frequently queried columns

```php
public function up(): void
{
    Schema::create('posts', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->text('content');
        $table->foreignId('user_id')->constrained()->onDelete('cascade');
        $table->timestamp('published_at')->nullable();
        $table->timestamps();
        
        $table->index('published_at');
    });
}
```

### Eloquent Models

- Define relationships in models
- Use model factories for testing
- Define fillable or guarded properties
- Use accessors and mutators for data transformation
- Use scopes for common queries

```php
class Post extends Model
{
    protected $fillable = ['title', 'content', 'user_id', 'published_at'];
    
    protected $casts = [
        'published_at' => 'datetime',
    ];
    
    public function user()
    {
        return $this->belongsTo(User::class);
    }
    
    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
    
    public function getTitleAttribute($value)
    {
        return ucfirst($value);
    }
    
    public function scopePublished($query)
    {
        return $query->whereNotNull('published_at')->where('published_at', '<=', now());
    }
}
```

## Controllers

### Keep Controllers Thin

- Controllers should only handle HTTP requests and responses
- Move business logic to services or models
- Use resource controllers for CRUD operations
- Use form requests for validation

```php
class PostController extends Controller
{
    protected $postService;
    
    public function __construct(PostService $postService)
    {
        $this->postService = $postService;
    }
    
    public function store(StorePostRequest $request)
    {
        $post = $this->postService->createPost($request->validated());
        
        return redirect()->route('posts.show', $post)
            ->with('success', 'Post created successfully.');
    }
}
```

## Routes

### Route Organization

- Group related routes
- Use route names
- Use route model binding
- Apply middleware at the group level when possible

```php
Route::middleware(['auth'])->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');
    
    Route::resource('posts', PostController::class);
    
    Route::prefix('admin')->middleware(['admin'])->group(function () {
        Route::resource('users', UserController::class);
    });
});
```

## Validation

### Form Requests

- Use form requests for complex validation
- Reuse validation rules when possible
- Add custom error messages

```php
class StorePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }
    
    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:255'],
            'content' => ['required', 'string'],
            'category_id' => ['required', 'exists:categories,id'],
            'tags' => ['array'],
            'tags.*' => ['exists:tags,id'],
        ];
    }
    
    public function messages(): array
    {
        return [
            'category_id.exists' => 'The selected category does not exist.',
            'tags.*.exists' => 'One or more selected tags do not exist.',
        ];
    }
}
```

## Authentication and Authorization

### Policies

- Use policies for authorization
- Register policies in the AuthServiceProvider
- Use Gates for simple authorization rules

```php
class PostPolicy
{
    public function viewAny(User $user): bool
    {
        return true;
    }
    
    public function view(User $user, Post $post): bool
    {
        return $post->published_at !== null || $user->id === $post->user_id;
    }
    
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id || $user->hasRole('editor');
    }
    
    public function delete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id || $user->hasRole('admin');
    }
}
```

## Services

### Service Layer

- Create services for complex business logic
- Inject dependencies via constructor
- Keep methods focused on a single responsibility

```php
class PostService
{
    protected $tagRepository;
    
    public function __construct(TagRepository $tagRepository)
    {
        $this->tagRepository = $tagRepository;
    }
    
    public function createPost(array $data): Post
    {
        $post = Post::create([
            'title' => $data['title'],
            'content' => $data['content'],
            'user_id' => auth()->id(),
            'category_id' => $data['category_id'],
        ]);
        
        if (isset($data['tags'])) {
            $post->tags()->attach($data['tags']);
        }
        
        return $post;
    }
}
```

## Repositories

### Repository Pattern

- Use repositories for complex database queries
- Create interfaces for repositories
- Bind interfaces to implementations in a service provider

```php
interface PostRepositoryInterface
{
    public function getPublishedPosts();
    public function getPostsByUser(User $user);
}

class PostRepository implements PostRepositoryInterface
{
    public function getPublishedPosts()
    {
        return Post::published()
            ->with(['user', 'category'])
            ->latest('published_at')
            ->paginate(15);
    }
    
    public function getPostsByUser(User $user)
    {
        return Post::where('user_id', $user->id)
            ->with(['category'])
            ->latest()
            ->paginate(15);
    }
}
```

## Views

### Blade Templates

- Use components for reusable UI elements
- Use layouts for consistent page structure
- Keep logic out of views
- Use view composers for common data

```php
// AppServiceProvider.php
public function boot(): void
{
    View::composer('layouts.sidebar', function ($view) {
        $view->with('categories', Category::all());
    });
}
```

### Blade Components

- Create components for reusable UI elements
- Use slots for flexible content
- Pass attributes to components

```php
// resources/views/components/alert.blade.php
<div class="alert alert-{{ $type ?? 'info' }}">
    <div class="alert-title">{{ $title }}</div>
    <div class="alert-body">{{ $slot }}</div>
</div>

// Usage
<x-alert type="success" title="Success!">
    Your post has been created.
</x-alert>
```

## API Development

### API Resources

- Use API resources for response transformation
- Create resource collections for lists
- Include relationships when needed

```php
class PostResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'content' => $this->content,
            'published_at' => $this->published_at,
            'created_at' => $this->created_at,
            'user' => new UserResource($this->whenLoaded('user')),
            'category' => new CategoryResource($this->whenLoaded('category')),
            'comments_count' => $this->whenCounted('comments'),
        ];
    }
}
```

### API Authentication

- Use Laravel Sanctum for API authentication
- Implement rate limiting
- Use API versioning

```php
// routes/api.php
Route::middleware('auth:sanctum')->prefix('v1')->group(function () {
    Route::apiResource('posts', PostController::class);
});
```

## Testing

### Unit Tests

- Test individual components in isolation
- Use mocks and stubs for dependencies
- Focus on testing business logic

```php
public function test_post_service_creates_post(): void
{
    $tagRepository = $this->mock(TagRepository::class);
    $postService = new PostService($tagRepository);
    
    $data = [
        'title' => 'Test Post',
        'content' => 'Test content',
        'category_id' => 1,
    ];
    
    $post = $postService->createPost($data);
    
    $this->assertEquals('Test Post', $post->title);
    $this->assertEquals('Test content', $post->content);
}
```

### Feature Tests

- Test complete features
- Test HTTP requests and responses
- Use database transactions

```php
public function test_user_can_create_post(): void
{
    $user = User::factory()->create();
    $category = Category::factory()->create();
    
    $response = $this->actingAs($user)
        ->post(route('posts.store'), [
            'title' => 'Test Post',
            'content' => 'Test content',
            'category_id' => $category->id,
        ]);
    
    $response->assertRedirect(route('posts.show', Post::latest()->first()));
    $this->assertDatabaseHas('posts', [
        'title' => 'Test Post',
        'user_id' => $user->id,
    ]);
}
```

## Performance Optimization

### Database Optimization

- Use eager loading to avoid N+1 queries
- Use query caching for expensive queries
- Use database indexes
- Use chunking for large datasets

```php
// Bad: N+1 problem
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->user->name;
}

// Good: Eager loading
$posts = Post::with('user')->get();
foreach ($posts as $post) {
    echo $post->user->name;
}

// Chunking for large datasets
Post::chunk(100, function ($posts) {
    foreach ($posts as $post) {
        // Process post
    }
});
```

### Caching

- Cache expensive operations
- Use cache tags for related items
- Set appropriate cache expiration times

```php
$posts = Cache::remember('homepage.posts', now()->addHours(3), function () {
    return Post::with(['user', 'category'])
        ->published()
        ->latest('published_at')
        ->take(10)
        ->get();
});
```

## Security

### CSRF Protection

- Use CSRF tokens for all forms
- Exclude API routes from CSRF protection

```html
<form method="POST" action="/posts">
    @csrf
    <!-- Form fields -->
</form>
```

### Mass Assignment Protection

- Define fillable or guarded properties on models
- Use form requests for validation

```php
class Post extends Model
{
    protected $fillable = [
        'title',
        'content',
        'category_id',
        'published_at',
    ];
    
    // Or use $guarded to blacklist attributes
    // protected $guarded = ['id', 'user_id'];
}
```

### SQL Injection Prevention

- Use Eloquent or query builder for database queries
- Never use raw queries with user input

```php
// Bad: SQL injection vulnerability
$results = DB::select("SELECT * FROM users WHERE email = '{$request->email}'");

// Good: Using query builder
$results = DB::table('users')->where('email', $request->email)->get();

// Better: Using Eloquent
$results = User::where('email', $request->email)->get();
```

### XSS Prevention

- Use {{ }} for escaping output in Blade templates
- Use {!! !!} only when necessary and with sanitized data

```blade
<!-- Good: Escaped output -->
<div>{{ $post->title }}</div>

<!-- Risky: Unescaped output, only use with sanitized data -->
<div>{!! $sanitizedHtml !!}</div>
```

## Deployment

### Environment Configuration

- Use .env files for environment-specific configuration
- Never commit .env files to version control
- Use environment variables for sensitive information

### Optimization Commands

- Run optimization commands before deployment

```bash
php artisan optimize
php artisan config:cache
php artisan route:cache
php artisan view:cache
composer install --optimize-autoloader --no-dev
npm run build
```

### Deployment Checklist

1. Run tests
2. Backup database
3. Put application in maintenance mode
4. Deploy code
5. Run migrations
6. Clear and rebuild cache
7. Take application out of maintenance mode

```bash
# Put application in maintenance mode
php artisan down

# Deploy code and run migrations
git pull origin main
composer install --optimize-autoloader --no-dev
npm run build
php artisan migrate --force

# Optimize application
php artisan optimize
php artisan config:cache
php artisan route:cache
php artisan view:cache

# Take application out of maintenance mode
php artisan up
```

## Logging and Monitoring

### Logging

- Use Laravel's logging system
- Configure appropriate log channels
- Log important events and errors

```php
Log::info('User logged in', ['user_id' => $user->id]);
Log::error('Payment failed', ['order_id' => $order->id, 'error' => $exception->getMessage()]);
```

### Exception Handling

- Customize exception handling in App\Exceptions\Handler
- Create custom exceptions for specific error cases
- Log exceptions with context

```php
class PaymentFailedException extends Exception
{
    protected $order;
    
    public function __construct(Order $order, $message = null)
    {
        $this->order = $order;
        parent::__construct($message ?? 'Payment processing failed');
    }
    
    public function context(): array
    {
        return ['order_id' => $this->order->id];
    }
}
```

## Queues and Background Jobs

### Queue Configuration

- Use queues for time-consuming tasks
- Configure queue workers appropriately
- Use supervisor for production queue workers

```php
// Dispatch a job to the queue
ProcessPodcast::dispatch($podcast);

// Dispatch with delay
ProcessPodcast::dispatch($podcast)->delay(now()->addMinutes(10));

// Specify queue
ProcessPodcast::dispatch($podcast)->onQueue('podcasts');
```

### Job Classes

- Create job classes for background processing
- Make jobs self-contained
- Handle failures gracefully

```php
class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    
    protected $podcast;
    
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast;
    }
    
    public function handle(AudioProcessor $processor): void
    {
        $processor->process($this->podcast);
    }
    
    public function failed(Exception $exception): void
    {
        Log::error('Podcast processing failed', [
            'podcast_id' => $this->podcast->id,
            'error' => $exception->getMessage(),
        ]);
        
        // Notify admin or retry
    }
}
```

## Maintenance

### Regular Updates

- Keep Laravel and packages updated
- Follow semantic versioning
- Read release notes before updating

### Code Quality Tools

- Use Laravel Pint for code style
- Use PHPStan or Larastan for static analysis
- Use PHP CS Fixer for code formatting

```bash
# Install Laravel Pint
composer require laravel/pint --dev

# Run Pint
./vendor/bin/pint

# Install Larastan
composer require nunomaduro/larastan:^2.0 --dev

# Run Larastan
./vendor/bin/phpstan analyse
```

## Documentation

### Code Documentation

- Document complex methods and classes
- Use PHPDoc blocks for type hinting
- Document API endpoints

```php
/**
 * Process a podcast for publishing.
 *
 * @param Podcast $podcast The podcast to process
 * @param bool $generateTranscript Whether to generate a transcript
 * @return bool True if processing was successful
 * @throws ProcessingException If processing fails
 */
public function processPodcast(Podcast $podcast, bool $generateTranscript = false): bool
{
    // Implementation
}
```

### Project Documentation

- Create a README.md file
- Document installation and setup steps
- Document API endpoints
- Keep documentation up-to-date

## Team Collaboration

### Git Workflow

- Use feature branches
- Write descriptive commit messages
- Use pull requests for code review
- Keep commits focused and atomic

### Code Review

- Review code for functionality
- Check for security issues
- Ensure code follows project standards
- Provide constructive feedback

## Conclusion

Following these best practices will help you build maintainable, secure, and performant Laravel applications. Remember that best practices evolve over time, so stay updated with the Laravel community and official documentation.