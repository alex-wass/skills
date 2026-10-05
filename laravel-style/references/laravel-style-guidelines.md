# Laravel Style Guidelines (Reference)

A portable style guide for Laravel projects. Where a rule here differs from a widely-assumed default, **this guide wins**; those points are listed under [Intentional Departures](#intentional-departures).

## Precedence

When sources disagree, the first one that covers the topic wins:

1. **`testing-best-practices`** skill — for anything about tests, when available.
2. **This guide** (`laravel-style`).
3. **`laravel-best-practices`** skill — the fallback for anything this guide doesn't cover, when available.
4. Laravel's documented conventions, then PSR-1, PSR-2 and PSR-12. The one exception is trait imports, where Laravel's own convention is followed over PSR-12 — see [Intentional Departures](#intentional-departures).
5. What sibling files in the codebase already do, rather than inventing a second pattern.

A skill that isn't installed is skipped.

## Table of Contents
- [Laravel Style Guidelines (Reference)](#laravel-style-guidelines-reference)
  - [Precedence](#precedence)
  - [Table of Contents](#table-of-contents)
  - [Core Principle](#core-principle)
  - [PHP Standards](#php-standards)
  - [Formatting Baseline (Pint)](#formatting-baseline-pint)
  - [Imports](#imports)
  - [Class Structure](#class-structure)
  - [Class Member Ordering](#class-member-ordering)
    - [Models](#models)
    - [Enums](#enums)
    - [Jobs](#jobs)
    - [Services and data classes](#services-and-data-classes)
    - [Controllers](#controllers)
    - [Policies](#policies)
    - [Observers](#observers)
    - [Form Requests](#form-requests)
  - [Documentation](#documentation)
    - [Annotations](#annotations)
  - [Comments](#comments)
  - [Whitespace](#whitespace)
  - [Local Variables](#local-variables)
  - [Control Flow](#control-flow)
  - [Ternaries and match](#ternaries-and-match)
  - [Closures](#closures)
  - [Multi-line Calls and Trailing Commas](#multi-line-calls-and-trailing-commas)
  - [Strings](#strings)
  - [Enums](#enums-1)
  - [Laravel Conventions](#laravel-conventions)
    - [Routes](#routes)
    - [Controllers](#controllers-1)
    - [Views](#views)
    - [Blade](#blade)
    - [Translations](#translations)
    - [Configuration](#configuration)
    - [Config File Comments](#config-file-comments)
    - [Artisan Commands](#artisan-commands)
    - [Migrations](#migrations)
    - [API Routing](#api-routing)
  - [Validation](#validation)
  - [Authorization](#authorization)
  - [Auth Resolution](#auth-resolution)
  - [Livewire](#livewire)
    - [Component member order](#component-member-order)
    - [Form object member order (`Livewire\Form`)](#form-object-member-order-livewireform)
    - [Notes](#notes)
  - [Testing](#testing)
    - [General](#general)
    - [Suite setup (Pest)](#suite-setup-pest)
    - [Layout and naming](#layout-and-naming)
    - [Filament (v4+)](#filament-v4)
    - [Assertions and fakes](#assertions-and-fakes)
    - [Test value](#test-value)
  - [Intentional Departures](#intentional-departures)
  - [Quick Reference](#quick-reference)
    - [Naming](#naming)
    - [File structure](#file-structure)

---

## Core Principle

**Follow Laravel conventions first. If Laravel has a documented way to do something, use it. Only deviate when you have a clear justification.**

Supporting principle — **these rules win; bring the code to them.** This guide is prescriptive. It describes how code should look, not how any particular codebase currently looks, so where existing code disagrees, align the code. Where this guide is silent, work down the [Precedence](#precedence) list.

---

## PHP Standards

- Follow PSR-1, PSR-2, and PSR-12.
- `camelCase` for non-public-facing strings.
- Short nullable notation: `?string`, never `string|null`.
- Always declare return types, including `void`.
- Always type method parameters and properties.
- Don't use `final` by default.
- **Do** use `readonly` for immutable value objects and injected dependencies (see [Class Structure](#class-structure)).

---

## Formatting Baseline (Pint)

If the project uses Laravel Pint, it owns the mechanical layer — never hand-fix what Pint handles: import ordering, operator spacing, array syntax, blank-line normalisation.

Run `vendor/bin/pint` after editing and make sure it passes. Everything in this document is the layer Pint **cannot** express.

A representative `pint.json` for this style (Laravel preset plus):

```json
{
    "preset": "laravel",
    "rules": {
        "not_operator_with_successor_space": false,
        "ordered_imports": { "sort_algorithm": "length", "imports_order": ["const", "class", "function"] },
        "concat_space": { "spacing": "one" },
        "blank_line_before_statement": true,
        "array_syntax": { "syntax": "short" }
    }
}
```

That config implies `!$user` (no space), spaced concatenation (`'key:' . $id`), and length-sorted imports. Match whatever the project's own `pint.json` says.

---

## Imports

Import every class, facade, enum, and exception. Never reference by FQN inline.

```php
// Good
use Illuminate\Support\Facades\DB;

DB::raw('price * quantity');

// Bad — FQN inline
\DB::raw('price * quantity');
\App\Enums\OrderStatus::Shipped;
```

**This applies inside docblocks too** — import the class and use the short name:

```php
// Good
use Illuminate\Support\Collection;

/** @return Collection<int, User> */

// Bad
/** @return \Illuminate\Support\Collection<int, \App\Models\User> */
```

Let Pint sort the `use` block; just add the import and re-run it.

---

## Class Structure

- **Typed properties**, never a docblock standing in for a type.
- **Constructor property promotion** when every property can be promoted.
- **Traits on a single `use` statement**, comma-separated.
- **`readonly`** for immutable value objects and injected dependencies — not for models or components.

```php
class OrderTotal
{
    use HasFactory, HasUuids;

    public function __construct(
        private readonly int $subtotalCents,
        private readonly int $taxCents,
    ) {}
}

// Immutable value object: mark the class readonly
readonly class DateRange
{
    public function __construct(
        public CarbonImmutable $start,
        public CarbonImmutable $end,
    ) {}
}
```

Promoted constructors with multiple parameters put each on its own line with a trailing comma. An empty body is `{}` on the same line as the closing paren.

---

## Class Member Ordering

Member placement is part of the style. Each artifact type has a fixed order. The universal rule: **public API first, then `protected`/`private` helpers after it.**

### Models
1. `use` traits
2. `$fillable`
3. `$hidden`
4. `casts()`
5. Accessors and mutators — `Attribute` methods
6. Relationships — `belongsTo` / `hasMany` / `hasOneThrough`
7. Query scopes — `scopeActive()`, `scopePublished()`
8. Instance helpers

### Enums
1. `case`s
2. Instance methods — `label()` first, then others
3. `static` methods last

### Jobs
1. `use Queueable;`
2. Config properties — `$tries`, `$backoff`, `$timeout`
3. `__construct`
4. `uniqueId()` when `ShouldBeUnique`
5. `handle()`
6. `private` helpers
7. `failed()` — last

### Services and data classes
1. Constants
2. Memoization properties
3. `__construct`
4. Public methods
5. `private` helpers last

### Controllers
CRUD order: `index` → `create` → `store` → `show` → `edit` → `update` → `destroy`.

### Policies
Laravel's canonical ability order, then custom abilities: `viewAny` → `view` → `create` → `update` → `delete` → `restore` → `forceDelete` → custom.

### Observers
Eloquent lifecycle order, then `private` helpers: `saving` → `creating` → `created` → `updating` → `updated` → `deleting` → `deleted` → helpers.

### Form Requests
`authorize()` → `rules()` → `messages()` / `attributes()` → helpers.

Livewire components and form objects have their own ordering — see [Livewire](#livewire).

---

## Documentation

**Never put a docblock or comment above a class declaration.** The class name and its namespace say what it is; a class comment restates them and goes stale. This applies to every artifact — models, controllers, jobs, services, enums, policies, components.

```php
// Bad
/**
 * Handles processing of customer orders.
 */
class OrderProcessor

// Good
class OrderProcessor
```

**Every method carries a one-line `/** … */` docblock**, including trivial ones — `render()`, `handle()`, `__construct`, `label()`, relationships, and scopes.

```php
/**
 * Get the user that owns this order.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class);
}

/**
 * Execute the job.
 */
public function handle(): void
```

Documented properties too:

```php
/**
 * The number of times the job may be attempted.
 */
public int $tries = 3;
```

Reuse the framework's own wording when a vendor stub provides one:

```php
/**
 * The attributes that are mass assignable.
 *
 * @var list<string>
 */
```

> This is a [deliberate deviation](#deliberate-deviations) from guides that say "omit docblocks when full type hints exist." Do not strip these.

### Annotations
- Document iterables with generics, always both key and value:  `@return Collection<int, User>`, `@param array<int, Order> $orders`
- Use array shape notation for fixed keys, one key per line:
  ```php
  /**
   * @return array{
   *     total: int,
   *     currency: string,
   * }
   */
  ```
- Most common type first in a union: `@var Collection|SomeVendor\Collection`
- If one parameter needs a docblock, document them all.
- Import classnames in docblocks; never fully qualified (see [Imports](#imports)).

Multi-line docblocks add a blank `*` line, then prose explaining a non-obvious rationale:

```php
/**
 * Ensure the user has a valid access token, refreshing if necessary.
 *
 * Uses a per-user cache lock to prevent concurrent refreshes.
 */
```

Config-file comments follow the cascading style — see [Config File Comments](#config-file-comments).

---

## Comments

Comments **inside method bodies** explain **why**, never **what**. Prefer expressive code and descriptive method names over a comment. If the code already says it, delete the comment.

```php
// Bad — restates the expression
// cycleStart = the date immediately before the next due date.
$cycleStart = $dates->last(fn (CarbonImmutable $date) => $date->lt($nextDueOn));

// Bad — restates the method call
// Generate occurrences for the next 18 months.
$dates = $schedule->occurrencesBetween($today, $today->addMonths(18));

// Good — captures a rationale the code cannot express
// Dates strictly after creation: the creation date represents the initial
// state, not a future target.
$dates = $schedule->occurrencesBetween($createdAt->addDay(), $end);
```

Section comments labelling the halves of a genuinely multi-part calculation are acceptable.

Formatting: a space after `//`; multi-line blocks use a single leading `*`.

```php
// Single line with a space after the slashes

/*
 * Multi-line blocks start with a single asterisk.
 */
```

---

## Whitespace

Let code breathe. Add blank lines between statements, except in sequences of equivalent single-line operations. Never leave an empty line directly inside `{}`.

```php
public function findPage(string $slug): ?Page
{
    $page = $this->pages()->where('slug', $slug)->first();

    if (!$page) {
        return null;
    }

    if ($page->isPrivate() && !auth()->check()) {
        return null;
    }

    return $page;
}
```

A blank line before `return` when the method has more than one statement.

---

## Local Variables

**No single-use variables.** Assigned once and read once → inline it.

```php
// Bad
$isPublished = $post->is_published;
// ... used exactly once, later
'published' => $isPublished,

// Good
'published' => $post->is_published,
```

Keep a variable when it is genuinely reused, or when naming a complex expression aids reading.

---

## Control Flow

- **Happy path last** — handle error and guard conditions first.
- **Avoid `else`** — use early returns.
- **Separate conditions** — prefer multiple `if` statements over one compound condition.
- **Always use curly braces**, even for single statements.

```php
public function handle(): void
{
    $order = $this->order->fresh();

    if (!$order) {
        return;
    }

    if ($order->isCancelled()) {
        return;
    }

    // happy path
}
```

---

## Ternaries and match

**Ternary as an array value or call argument → three lines:**

```php
'status' => $date->isPast()
    ? OrderStatus::Overdue
    : OrderStatus::Scheduled,
```

**Scalar assignment and `match` arms → one line:**

```php
$from = $start->lessThan($cycleStart) ? $cycleStart : $start;

return match ($this) {
    Frequency::Daily => $interval === 1 ? 'Daily' : "Every {$interval} days",
    Frequency::Yearly => $interval === 1 ? 'Yearly' : "Every {$interval} years",
};
```

Anything longer than a short expression breaks onto its own lines:

```php
$result = $object instanceof Model
    ? $object->name
    : 'A default value';
```

**Exhaustive `match` over an enum takes no `default` arm** — an unhandled case should raise `UnhandledMatchError` rather than fall through silently:

```php
return match ($this) {
    self::Daily, self::Weekly => false,
    self::Monthly => $interval > 1,
    self::Yearly, self::OneOff => true,
};
```

Where a case is genuinely impossible, assert it rather than defaulting:

```php
Frequency::OneOff => throw new LogicException('A detected pattern cannot be one-off.'),
```

A `default` arm is fine when it legitimately means "everything else".

---

## Closures

**Type every closure parameter:**

```php
// Good
fn (Order $order) => $order->placed_at->toDateString()
fn (Builder $query) => $query->where('placed_at', '>=', $start)
function (Order $order): int { }

// Bad
fn ($q) => ...
fn ($o) => ...
```

Add `: void` to closures that return nothing:

```php
DB::transaction(function () use ($order): void {
    // ...
});
```

---

## Multi-line Calls and Trailing Commas

Multi-line calls and array literals end with a trailing comma on the final argument.

Use **named arguments** for value-object constructors and any call with several optional or same-typed parameters.

```php
$schedule = new Schedule(
    frequency: $order->frequency,
    interval: $order->interval,
    startsOn: $startsOn,
);
```

---

## Strings

- Curly interpolation over concatenation: `"Hi, I am {$name}."`
- Spaced concatenation for simple joins: `'cache-key:' . auth()->id()`
- Wrap user-facing strings in `__()`; no trailing whitespace inside the string — `__('Sync started.')`, not `__('Sync started. ')`

---

## Enums

- **Enum cases: `PascalCase`.** Backed values are `snake_case`.

Class constants are not covered here — PSR-1 governs them.

```php
enum OrderStatus: string
{
    case AwaitingPayment = 'awaiting_payment';
    case Shipped = 'shipped';

    /**
     * Get the human-readable label for the status.
     */
    public function label(): string
    {
        return match ($this) {
            self::AwaitingPayment => 'Awaiting payment',
            self::Shipped => 'Shipped',
        };
    }
}
```

Pass backed enums directly to query builders — never `->value`:

```php
->where('status', OrderStatus::Shipped)
```

---

## Laravel Conventions

### Routes
- URLs: `kebab-case` (`/order-history`)
- Route names: dot-notation, lowercase (`orders.index`, `auth.callback`)
- Parameters: `camelCase` (`{orderId}`)
- Tuple notation: `[OrdersController::class, 'index']`
- HTTP verb first; no leading slash unless the URL is empty

```php
Route::get('/', [HomeController::class, 'index'])->name('home');
Route::get('order-history', [OrdersController::class, 'index'])->name('orders.index');
Route::post('orders', [OrdersController::class, 'store'])->name('orders.store');
```

### Controllers
- Plural resource name + `Controller` (`OrdersController`)
- Stick to CRUD methods; extract a new controller for non-CRUD actions (`FavouritePostsController@store` rather than `PostsController@favourite`)
- Single-action controllers use `__invoke`

### Views
- View files: `kebab-case` (`order-history.blade.php`, `delete-user-form.blade.php`)
- Directories mirror the component/route namespace

### Blade
- Indent with 4 spaces
- **Add a space after control structures**, matching the Laravel documentation: `@if ($condition)`, `@foreach ($orders as $order)`
- `__()` over `@lang`

```blade
@if ($orders->isNotEmpty())
    @foreach ($orders as $order)
        <h2>{{ __('orders.history.title') }}</h2>
    @endforeach
@endif
```

### Translations
- Use the `__()` function, not `@lang`
- Key by dot-notation namespace: `__('orders.history.title')`
- No trailing whitespace inside a translatable string

```blade
<h2>{{ __('newsletter.form.title') }}</h2>
```

### Configuration
- Files: `kebab-case` (`pdf-generator.php`)
- Keys: `snake_case` (`chrome_path`)
- Add third-party service credentials to `config/services.php` rather than a new file
- `env()` **only** inside config files; everywhere else use `config()`

### Config File Comments
Laravel's section headers use a three-line **cascading** body: each line is slightly shorter than the one above, which gives configs their characteristic polish.

| Line | Target content length |
|---|---|
| 1st | 72–76 characters |
| 2nd | 68–72 characters (~4 fewer than the 1st) |
| 3rd | 65–69 characters (~3 fewer than the 2nd) |

Count characters excluding the leading `| `.

```php
/*
|--------------------------------------------------------------------------
| Application Timezone
|--------------------------------------------------------------------------
|
| Here you may specify the default timezone for your application, which
| will be used by the PHP date and date-time functions. We have gone
| ahead and set this to a sensible default for you out of the box.
|
*/
```

To hit the cascade, reword rather than pad: drop filler ("in order to" → "to") or shorten synonyms when a line runs long; add a clarifying adjective or expand a contraction when it runs short. The prose must still read naturally.

### Artisan Commands
- Names: `kebab-case` (`php artisan delete-old-records`)
- Always give feedback; show progress in loops and a summary at the end
- Print the line **before** processing an item, so a crash names the culprit

```php
$orders->each(function (Order $order): void {
    $this->info("Processing order `{$order->id}`...");

    $this->processOrder($order);
});

$this->comment("Processed {$orders->count()} orders.");
```

### Migrations
- Generate with `php artisan make:migration`
- **Write a reversible `down()`** — do not omit it
- Use `constrained()` for foreign keys; add indexes in the same migration
- One concern per migration

**Editing an existing migration.** While a migration is still new on the current branch — part of the feature being built and not yet merged or shipped — edit it in place rather than stacking a follow-up migration on top. A feature should land as the migrations it actually needs, not as a history of how it was developed. Once it has shipped, never edit it; add a new migration.

**Data changes.** A data change may live in the migration that introduces the schema it belongs to, so the two stay colocated:

```php
public function up(): void
{
    Schema::table('orders', function (Blueprint $table): void {
        $table->string('status')->default('pending');
    });

    Order::query()->whereNotNull('shipped_at')->update(['status' => 'shipped']);
}
```

### API Routing
- Plural resource names: `/errors`
- `kebab-case` segments: `/error-occurrences`
- Limit nesting depth: prefer `/error-occurrences/1` over deep chains

---

## Validation

Use **array notation** for rules — it composes with custom rule objects:

```php
public function rules(): array
{
    return [
        'email' => ['required', 'email'],
        'status' => ['required', Rule::enum(OrderStatus::class)],
    ];
}
```

Custom validation rule names are `snake_case`.

Scope `exists`/`unique` rules to the current user when the value is user-supplied:

```php
'account_id' => ['nullable', Rule::exists('accounts', 'id')->where('user_id', auth()->id())],
```

Always consume `validated()` output — never raw request or property values.

---

## Authorization

- Ability names: `camelCase` (`Gate::define('editPost', ...)`)
- Use CRUD words, but `view` instead of `show`
- Authorize every action that mutates or reveals a model

```blade
@can('update', $post)
    <a href="{{ route('posts.edit', $post) }}">{{ __('Edit') }}</a>
@endcan
```

---

## Auth Resolution

Use the `auth()` helper. Never the `Auth` facade.

```php
// Good
auth()->user()->orders()->create($data);
auth()->id();
auth()->guard('web')->logout();

// Bad
Auth::user();
Auth::id();
```

Scope user-supplied IDs through the relationship, never a bare model lookup:

```php
// Good
$order = auth()->user()->orders()->find($id);

// Bad — resolves any row in the table
$order = Order::find($id);
```

---

## Livewire

Skip this section in projects that don't use Livewire.

### Component member order
1. `use` traits
2. Bound model properties first, then scalar state
3. `#[Computed]` properties — **grouped together**, never scattered
4. Lifecycle hooks — `mount()`, `boot()`
5. Update hooks and listeners — `updatedFoo()`, `#[On(...)]`
6. Public action methods
7. `protected` / `private` helpers
8. `render()` — **always last**

```php
class Show extends Component
{
    public Customer $customer;        // 2. bound model
    public string $filter = 'all';    // 2. scalar state

    #[Computed]
    public function orders(): Collection { }         // 3.

    public function mount(): void { }                // 4.

    public function updatedFilter(): void { }        // 5.

    public function deleteOrder(string $id): void { }        // 6.

    private function resolveCacheKey(): string { }   // 7.

    public function render(): View { }               // 8. always last
}
```

### Form object member order (`Livewire\Form`)
1. The bound model — `public ?Order $order = null;`
2. Validated properties with `#[Validate]`
3. Properties without validation attributes
4. `rules()`
5. Hydration — `setOrder()`
6. Update hooks — `updatedStatus()`
7. Persistence — `save()`, `update()`
8. `protected` helpers

### Notes
- Expose the authenticated user once as a `#[Computed]` property and reuse it.
- Validate and authorize in actions exactly as you would in an HTTP request.
- Livewire autowires action-method parameters — inject dependencies there rather than reaching for `app()`.

---

## Testing

> **Precedence:** the `testing-best-practices` skill always wins on testing. If a rule here conflicts with it, follow `testing-best-practices`. This section only adds Laravel-specific conventions on top of it.

### General
- Arrange–act–assert.
- Use factories and their states to build data.
- Test the slice, not the change. A behaviour change is covered by the test of the whole flow it belongs to (the request, page or command a user actually runs). Update that existing test's arrange and assertions to cover the new behaviour. Don't add a separate micro-test for each change.
- Add a new test only when a flow has no test yet, or the change is a real new path through it (a new outcome, error or permission branch).

### Suite setup (Pest)
- Bind every suite in one place in `tests/Pest.php`: `pest()->extend(TestCase::class)->use(LazilyRefreshDatabase::class)->beforeEach(...)->in('Feature', 'Unit')`. Never call `pest()->extend()` in an individual test file.
- The global `beforeEach` holds the safety nets every test needs: `Http::preventStrayRequests()`, `Sleep::fake(syncWithCarbon: true)`, `Exceptions::fake()` and `$this->withoutVite()`.
- On Laravel versions that ship them, `TestCase` uses `WithCachedConfig` and `WithCachedRoutes`.
- A helper goes in `Pest.php` only once two or more test files use it. Until then it stays in the file that uses it. Name helpers so two of them can't be mistaken for each other.

### Layout and naming
- Test paths mirror `app/`: one test file per class, named after it (`app/Actions/PublishPost.php` → `tests/Feature/Actions/PublishPostTest.php`). Don't split one class's tests across several files.
- Tests that span many classes (a whole admin panel, seeders) get a folder named for the area, e.g. `tests/Feature/Panel/`, `tests/Feature/Seeders/`.
- Use `it()` for behaviour and `test()` for declarative facts (policies, enums). Keep to one style per file.
- Names describe the behaviour and lead with the result: `it('cannot link another users record')`. API error tests include the status code: `it('returns 403 when the user does not own the post')`. Never name a factory state or a method.

### Filament (v4+)
Applies only when the project uses Filament.
- Use Filament's documented testing assertions. Don't reach into `getTable()` internals. Build a check by hand only when no assertion exists, and give the helper a docblock saying why.
- Check for deprecations in `vendor/filament/*/.stubs.php`. The runtime traits don't carry the `@deprecated` tags, so the IDE won't warn you.
- Table actions: `TestAction::make(...)->table($record)`, and `->table()->bulk()` for bulk actions. Never use `assertTableAction*` or `callTableAction`.
- Action forms: `fillForm` (not `setActionData`), `assertSchemaStateSet` (not `assertFormSet`), and `assertHasFormErrors` / `assertHasNoFormErrors` for validation.
- Refer to actions by class, not by string name: custom actions through their `#[ActionName]` attribute, built-ins as `ViewAction::class`.
- Don't test table or menu configuration: column lists, which columns are toggleable or sortable, character limits, action order. Do test what the table computes from a record (counts, colours, tooltips, placeholders, formatted state) and what interacting with it does (filter, sort, call an action).
- Assert that an action is absent with `assertActionDoesNotExist`. `assertTableActionsExistInOrder([])` passes for any table, so it proves nothing.
- For columns hidden by default, use `assertCanNotRenderTableColumn`. `isHidden()` doesn't report toggled-off columns.
- Always pass a direction to `sortTable()`. Without one it toggles, which moves away from the table's default sort.

### Assertions and fakes
- Assert database state with Laravel's assertions (`assertDatabaseHas`, `assertModelExists`, …), imported as Pest functions (`use function Pest\Laravel\assertDatabaseHas;`), never through `$this->`.
- Use `expect()` for values. A count with conditions also uses `expect(Post::where(...)->count())`, since `assertDatabaseCount()` only counts a whole table.
- Test a query scope by comparing the ids it returns to a known list: `expect(Post::published()->pluck('id')->all())->toEqualCanonicalizing([$a->id, $b->id])`. Don't use `->exists()`, which passes if any single row matches.
- Name the classes in every fake: `Queue::fake([SendInvoice::class])`. Use a bare `Queue::fake()` only together with `assertNothingPushed()`.
- If a fake is there only to stop a background side effect (not to be asserted on), wrap it in a named helper such as `withoutSearchIndexing()`, so it doesn't look like a forgotten assertion.
- Don't fake or assert against something the code never does.

### Test value
- Test the full permission matrix at the lowest layer only: policy tests run as a dataset over every role (`Role::cases()`), plus an explicit denial for a user from another tenant or owner. Controller, page and API tests keep one allowed and one denied case to prove the policy is wired in.
- Don't test factory defaults or the exact contents of a seeder. Test a seeder by its contract: it fills the tables, running it twice is safe, and (in multi-tenant apps) its records don't cross tenants.
- Every assertion must be able to fail. After writing a test for a bug, break the code, watch the test fail, then restore it.
- When two nullable fields depend on each other, test every combination that can actually occur.
- Validation tests assert the rule that failed by name (`['field' => 'required']`), not the message text, which is framework copy and breaks when it changes. Assert the field alone only when the rule has no name, such as a closure rule.
- For jobs using `WithoutOverlapping` middleware, assert the lock key, not just that the middleware is present.

---

## Intentional Departures

These rules differ from what is widely assumed to be idiomatic Laravel, so tooling — or an assistant working from general training — may try to "correct" them. **Don't.** They are chosen deliberately.

| Topic | Widely-assumed default | This guide |
|---|---|---|
| Docblocks | Omit when fully type-hinted | **Every method gets a one-line docblock** |
| Class docblocks | Commonly added, and some tooling inserts them | **Never** — methods are documented, classes are not |
| Traits | One `use` statement per line (PSR-12 §4.2) | **Single comma-separated `use` statement** — matches Laravel's own model stubs |
| `readonly` | Don't use by default | **Use for value objects and injected dependencies** |
| View files | `camelCase` | **`kebab-case`** |
| Migrations | Only write `up()` | **Always write a reversible `down()`** |
| Editing migrations | Never modify an existing migration | **Edit in place while still new on the branch; never once shipped** |
| Data in migrations | Keep schema and data changes separate | **Colocate a data change with the schema change it belongs to** |
| Ternaries | "Own line unless short" | **Array/argument values split; scalar assignments and `match` arms inline** |
| Route names | `camelCase` | **Dot-notation, lowercase** |

Everything else here is standard Laravel and PSR practice — `PascalCase` enum cases, early returns, avoiding `else`, curly braces always, string interpolation, array validation notation, tuple route notation, plural controller names, `config()` over `env()`, service credentials in `config/services.php`, `__()` over `@lang`, and no fully qualified classnames in docblocks.

---

## Quick Reference

### Naming
| Thing | Convention |
|---|---|
| Classes | `PascalCase` (`OrdersController`, `OrderStatus`) |
| Methods / variables | `camelCase`, descriptive — `isRegisteredForDiscounts`, not `discount` |
| Enum cases | `PascalCase`, backed values `snake_case` |
| Accessors | `protected function nextDueOn(): Attribute` → read as `$model->next_due_on` |
| Scopes | `scopePublished(Builder $query): Builder` |
| URLs / config files / commands | `kebab-case` |
| Config keys | `snake_case` |
| Route names | dot-notation (`orders.index`) |
| View files | `kebab-case` |

### File structure
- Controllers: plural + `Controller` (`OrdersController`)
- Jobs: action-based (`CreateUser`, `SendEmailNotification`)
- Events: tense-based (`UserRegistering`, `UserRegistered`)
- Listeners: action + `Listener` (`SendInvitationMailListener`)
- Commands: action + `Command` (`PublishScheduledPostsCommand`)
- Mailables: purpose + `Mail` (`AccountActivatedMail`)
- Resources: plural + `Resource` (`UsersResource`)
- Enums: descriptive, no prefix (`OrderStatus`, `BookingType`)
