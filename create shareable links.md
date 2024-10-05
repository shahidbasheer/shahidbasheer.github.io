To implement a feature in Laravel that allows users to share their projects with others for editing—similar to Google Sheets' editable links—you can create **shareable links** using unique tokens. This approach enables project owners to grant edit access without manually adding each user as a member. Here's a step-by-step guide to achieve this:

### 1. **Database Schema Update**

First, you'll need to create a new table to store the shareable links associated with each project.

```php
// Create a migration for project_shares table
php artisan make:migration create_project_shares_table --create=project_shares
```

In the migration file:

```php
public function up()
{
    Schema::create('project_shares', function (Blueprint $table) {
        $table->id();
        $table->foreignId('project_id')->constrained()->onDelete('cascade');
        $table->string('token')->unique();
        $table->timestamp('expires_at')->nullable(); // Optional: to set expiration
        $table->timestamps();
    });
}

public function down()
{
    Schema::dropIfExists('project_shares');
}
```

Run the migration:

```bash
php artisan migrate
```

### 2. **Model Setup**

Create a `ProjectShare` model to interact with the `project_shares` table.

```php
php artisan make:model ProjectShare
```

In `app/Models/ProjectShare.php`:

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class ProjectShare extends Model
{
    use HasFactory;

    protected $fillable = [
        'project_id',
        'token',
        'expires_at',
    ];

    public function project()
    {
        return $this->belongsTo(Project::class);
    }

    // Optionally, add a scope to check for valid tokens
    public function scopeValid($query)
    {
        return $query->where(function ($q) {
            $q->whereNull('expires_at')
              ->orWhere('expires_at', '>', now());
        });
    }
}
```

### 3. **Generating Shareable Links**

Create a method to generate a unique token and store it in the `project_shares` table.

```php
use App\Models\ProjectShare;
use Illuminate\Support\Str;

// In your ProjectController or a dedicated controller
public function generateShareableLink(Request $request, $projectId)
{
    $project = Project::findOrFail($projectId);

    // Ensure the authenticated user owns the project
    if ($project->user_id !== auth()->id()) {
        abort(403, 'Unauthorized action.');
    }

    // Generate a unique token
    $token = Str::random(32);

    // Optionally set an expiration time, e.g., 7 days from now
    $expiresAt = now()->addDays(7);

    // Create the share record
    $share = ProjectShare::create([
        'project_id' => $project->id,
        'token' => $token,
        'expires_at' => $expiresAt,
    ]);

    // Generate the shareable URL
    $shareableUrl = route('projects.shared.edit', ['token' => $token]);

    return response()->json([
        'shareable_url' => $shareableUrl,
        'expires_at' => $expiresAt,
    ]);
}
```

**Route Example:**

```php
// In routes/web.php or routes/api.php
Route::get('/projects/shared/edit/{token}', [ProjectController::class, 'sharedEdit'])
    ->name('projects.shared.edit');
```

### 4. **Handling Shared Access**

Create a method to handle editing via the shareable link.

```php
public function sharedEdit($token)
{
    $share = ProjectShare::valid()->where('token', $token)->firstOrFail();

    $project = $share->project;

    // Optionally, you can log the access or perform additional checks

    // Render the edit view, passing the project
    return view('projects.edit', compact('project', 'share'));
}
```

### 5. **Authorization in Edit Actions**

Ensure that both the project owner and anyone with a valid shareable link can edit the project. Update your authorization logic accordingly.

```php
public function update(Request $request, $projectId, $token = null)
{
    if ($token) {
        // Editing via shareable link
        $share = ProjectShare::valid()->where('token', $token)->firstOrFail();
        $project = $share->project;
    } else {
        // Standard editing by authenticated user
        $project = Project::findOrFail($projectId);
        if ($project->user_id !== auth()->id()) {
            abort(403, 'Unauthorized action.');
        }
    }

    // Validate and update the project
    $validated = $request->validate([
        'name' => 'required|string|max:255',
        // Add other fields as necessary
    ]);

    $project->update($validated);

    return redirect()->route('projects.show', $project->id)
                     ->with('success', 'Project updated successfully.');
}
```

**Update Route to Handle Token:**

```php
Route::post('/projects/{project}/update/{token?}', [ProjectController::class, 'update'])
    ->name('projects.update');
```

### 6. **Security Considerations**

- **Token Security:** Ensure that tokens are sufficiently random and unique. Using `Str::random(32)` provides a good level of entropy.
  
- **Expiration:** Implement token expiration to limit the window of access. This reduces the risk if a token is inadvertently shared.
  
- **Revocation:** Provide functionality for project owners to revoke shareable links, deleting the corresponding records from `project_shares`.
  
- **Access Limits:** Optionally, limit the number of edits or views a shareable link can perform.

### 7. **Optional Enhancements**

- **Email Invitations:** Instead of generating a public link, send the shareable link via email to specific users.
  
- **Role-Based Access:** Extend the functionality to allow different levels of access (e.g., view-only, comment, edit).
  
- **Audit Logs:** Keep track of who accessed the project via the shareable link and what changes were made.

### 8. **Example View Implementation**

In your `projects/edit.blade.php` view, you might want to handle both authenticated users and those accessing via a shareable link.

```blade
@extends('layouts.app')

@section('content')
<div class="container">
    <h1>Edit Project: {{ $project->name }}</h1>

    <form action="{{ route('projects.update', ['project' => $project->id, 'token' => $share->token ?? null]) }}" method="POST">
        @csrf
        @method('POST')

        <div class="form-group">
            <label for="name">Project Name</label>
            <input type="text" name="name" id="name" value="{{ old('name', $project->name) }}" class="form-control" required>
        </div>

        <!-- Add other fields as necessary -->

        <button type="submit" class="btn btn-primary">Update Project</button>
    </form>
</div>
@endsection
```

### 9. **Testing the Implementation**

Ensure you thoroughly test the feature:

- **Generate a Shareable Link:** As a project owner, generate a shareable link and verify it works for editing.
  
- **Access Control:** Attempt to access the edit page without a token or with an invalid/expired token to ensure access is denied.
  
- **Revocation:** Revoke a shareable link and confirm it no longer grants access.

### 10. **Final Thoughts**

Implementing shareable links provides a flexible way to grant temporary or specific access to your projects without the overhead of managing individual user permissions. Always prioritize security by ensuring tokens are secure, have limited lifespans, and can be revoked as needed.

If you prefer using packages to streamline this process, consider exploring Laravel packages like [spatie/laravel-permission](https://github.com/spatie/laravel-permission) for advanced permission handling, although for simple shareable links, a custom implementation as outlined above is often sufficient and more straightforward.
