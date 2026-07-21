---
name: laravel-style
description: Apply these Laravel coding conventions for any task that creates, edits, reviews, refactors, or formats Laravel or Blade code. Covers PSR standards, class structure and member ordering, docblock and comment policy, control flow, ternary and closure formatting, imports, naming, routes, config, validation, authorization, Blade, Livewire, migrations, and testing conventions. Activate for controllers, models, migrations, jobs, services, enums, policies, observers, form objects, and Livewire components. Sits on top of Laravel Pint, which handles mechanical formatting.
license: MIT
metadata:
  author: alex-wass
  credit: https://github.com/spatie/guidelines-skills/
---

# Laravel Style

## Overview
Applies a custom style guide for Laravel projects to keep code style consistent and Laravel-native.

Anything not covered here is governed by Laravel's own convention, then PSR-1, PSR-2, and PSR-12.

## When to Activate
- Activate for any Laravel coding work, even if style is not mentioned.
- Activate when generating, editing, formatting, refactoring, or reviewing Laravel/Blade code.
- Activate when adding a method, class, or enum case — member **placement** and the **docblock**
  are part of the style, not an afterthought.

## Scope
- In scope: `.php`, `.blade.php` — formatting, naming, member ordering, documentation, comments, control flow, and Laravel's own conventions (routes, config, validation, migrations, tests).
- Out of scope: behaviour, performance, N+1s, authorization gaps, queue reliability. Do not fix bugs under the banner of a style pass — report them separately.
- Out of scope: JS/TS, CSS, infrastructure, non-Laravel frameworks.

## Workflow
1. Identify the artifact (model, controller, job, service, enum, policy, Livewire component).
2. Read `references/laravel-style-guidelines.md` and focus on the relevant sections — especially **Class Member Ordering**, which differs per artifact type.
3. Apply the core principle first, then the PHP standards, then the artifact-specific rules.
4. Run `vendor/bin/pint` and confirm it passes.

## Core Principle
**Follow Laravel conventions first. If Laravel has a documented way to do something, use it. Only deviate when you have a clear justification.**

Second principle: **these rules win — bring the code to them.** This guide is prescriptive, not a description of what a codebase currently does; where existing code disagrees, align the code. Where this guide is silent, follow what sibling files already do rather than inventing a second pattern.

## Core Rules (Summary)
- Follow Laravel conventions first.
- Follow PSR-1, PSR-2, and PSR-12.
- Prefer typed properties and explicit return types (including `void`).
- Use short nullable syntax like `?string`.
- Use constructor property promotion when all properties can be promoted.
- Put traits on a single comma-separated `use` statement.
- Use `readonly` for value objects and injected dependencies.
- Give every method a one-line `/** … */` docblock, including trivial ones.
- Never put a docblock or comment above a class declaration.
- Explain **why** in body comments, never **what**.
- Inline single-use variables.
- Import every class, facade, and exception.
- Never reference a class by an inline fully qualified name.
- Split ternaries used as array values or call arguments across three lines.
- Keep scalar-assignment ternaries and `match` arms on one line.
- Omit the `default` arm from an exhaustive `match` over an enum.
- Type every closure parameter.
- Add `: void` to closures that return nothing.
- End multi-line calls with a trailing comma.
- Use named arguments for value-object constructors.
- Prefer early returns and avoid `else` when possible.
- Happy path last: handle error conditions first.
- Always use curly braces for control structures.
- Use string interpolation over concatenation.
- Use the `auth()` helper, never the `Auth` facade.
- Scope user-supplied IDs through the authenticated user's relationships.
- Order class members by artifact type.
- Put `render()` last in a Livewire component.

## Do and Don't

Do:
- Document every method, even `render()` and `handle()`.
- Group all Livewire `#[Computed]` properties together, before lifecycle hooks.
- Put `protected`/`private` helpers after the public API.
- Pass backed enums directly to `where()` — not `->value`.
- Use PascalCase for enum cases.
- Use kebab-case URLs, config files, Artisan commands, and view files.
- Use snake_case config keys.
- Use dot-notation route names.
- Use camelCase route parameters.
- Use tuple notation for routes: `[Controller::class, 'method']`.
- Use plural resource names for controllers (`OrdersController`).
- Use array notation for validation rules.
- Scope `exists` and `unique` rules to the current user.
- Use the `config()` helper and avoid `env()` outside config files.
- Add service configs to `config/services.php`, not new files.
- Use `__()` for translations instead of `@lang`.
- Add a space after Blade control structures: `@if ($condition)`, per the Laravel docs.
- Write a reversible `down()` in every migration.
- Edit a migration in place while it is still new on the current branch.
- Colocate a data change in the migration that introduces the schema it belongs to.
- Let code breathe — add blank lines between statements.
- Add a blank line before `return` in multi-statement methods.

Don't:
- Don't add a class-level docblock or comment.
- Don't write a comment that restates the line beneath it.
- Don't add a `default` arm to a `match` that covers every enum case.
- Don't put `#[Computed]` methods at the bottom of a Livewire component.
- Don't leave closure parameters untyped, like `fn ($q)`.
- Don't use `Auth::user()`.
- Don't use `env()` outside config files.
- Don't use `final` by default.
- Don't use fully qualified classnames in docblocks.
- Don't edit a migration that has already shipped — add a new one instead.
- Don't hand-fix anything Pint already handles.

## Examples
```php
/**
 * Get the orders belonging to this user.
 */
public function orders(): HasMany
{
    return $this->hasMany(Order::class);
}
```

```php
// Guard clauses, happy path last, breathing room.
public function handle(): void
{
    $order = $this->order->fresh();

    if (!$order) {
        return;
    }

    if ($order->isCancelled()) {
        return;
    }

    $this->dispatchInvoice($order);
}
```

```php
// Ternary as an array value: split across three lines.
$record->update([
    'status' => $date->isPast()
        ? OrderStatus::Overdue
        : OrderStatus::Scheduled,
]);

// Scalar assignment and match arms: stay inline.
$from = $start->lessThan($cycleStart) ? $cycleStart : $start;

return match ($this) {
    Frequency::Daily => $interval === 1 ? 'Daily' : "Every {$interval} days",
    Frequency::Yearly => $interval === 1 ? 'Yearly' : "Every {$interval} years",
};
```

```php
// Typed closures, trailing comma on multi-line calls, named arguments.
->groupBy(fn (Order $order) => $order->placed_at->toDateString());

$schedule = new Schedule(
    frequency: $order->frequency,
    interval: $order->interval,
    startsOn: $startsOn,
);
```

## References
- `references/laravel-style-guidelines.md`
