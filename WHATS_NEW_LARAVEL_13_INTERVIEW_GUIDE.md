# Laravel 13 — Interview Knowledge-Upgrade Guide

> Short, precise reference for interviews. Verified against the official release notes
> and the freshly-cloned `laravel/laravel` skeleton (this folder) on **2026-07-20**.

---

## 1. Fast facts (memorize these)

| Thing | Value |
|---|---|
| **Latest version** | **Laravel 13** |
| **Released** | **17 March 2026** |
| **Release cadence** | One major per year, ~Q1. Minor/patch weekly — **never** breaking |
| **PHP required** | **8.3 – 8.5** (v12 was 8.2–8.5, v11 was 8.2–8.4) |
| **Support policy** | Bug fixes **18 months**, security fixes **2 years** |
| **v13 bug fixes until** | Q3 2027 · **security until 17 Mar 2028** |
| Skeleton this folder | `laravel/framework ^13.8`, `tinker ^3.0`, PHPUnit **12.5** |

**Theme of the release:** *AI-native, minimal breaking changes.* Laravel now brands itself
"the clean stack for Artisans **and agents**". Most apps upgrade 12 → 13 with little/no code change.

> ⚠️ **Your machine:** XAMPP here runs **PHP 8.2.12**. This clone is fine to *read/browse*, but
> `composer install` / `php artisan serve` will fail until you install **PHP 8.3+**. (v12 is the
> newest version that runs on your 8.2.)

---

## 2. What's genuinely new in Laravel 13

### 2.1 First-party **Laravel AI SDK** ⭐ (the headline)
Provider-agnostic, Laravel-native API for text, tool-calling **agents**, embeddings, audio, images.
```php
$response = SalesCoach::make()->prompt('Analyze this sales transcript...');   // agent
$image    = Image::of('A donut on the counter')->generate();                   // image
$audio    = Audio::of('I love coding with Laravel.')->generate();              // TTS
$vec      = Str::of('Napa Valley has great wine.')->toEmbeddings();            // embeddings
```
*Talking point:* swap providers without rewriting app code — same idea as the DB/queue/cache
driver abstraction, now for LLMs.

### 2.2 Semantic / **Vector Search** in the query builder
Native vector queries on **PostgreSQL + pgvector** (and **MariaDB** vector indexes via schema builder).
```php
$docs = DB::table('documents')
    ->whereVectorSimilarTo('embedding', 'Best wineries in Napa Valley')
    ->limit(10)->get();
```
You can define vector indexes in migrations — no raw SQL. Combines with the AI SDK's embeddings.

### 2.3 **JSON:API resources** (first-party)
New resource classes emit spec-compliant responses: resource-object serialization, relationship
**includes**, **sparse fieldsets**, links, and correct JSON:API headers — no more hand-rolling it.

### 2.4 Expanded **PHP 8 Attributes** (non-breaking, opt-in)
Declarative config colocated with the class instead of properties/boilerplate.
```php
#[Middleware('auth')]
class CommentController {
    #[Middleware('subscribed')]
    #[Authorize('create', [Comment::class, 'post'])]
    public function store(Post $post) { /* ... */ }
}
```
Also for jobs: `#[Tries]`, `#[Backoff]`, `#[Timeout]`, `#[FailOnTimeout]`; plus new attributes across
Eloquent, events, notifications, validation, testing. Attributes can now replace `$table`,
`$fillable`, `$hidden` on models. **Old property style still works** — attributes are an alternative.

### 2.5 **Queue routing by class** — `Queue::route()`
Centralize which connection/queue a job uses; keeps infra config out of the job's business logic.
```php
Queue::route(ProcessPodcast::class, connection: 'redis', queue: 'podcasts');
```

### 2.6 **PreventRequestForgery** middleware (CSRF, hardened)
CSRF protection formalized as `PreventRequestForgery` with **origin-aware** verification, while
staying compatible with token-based CSRF.

### 2.7 `Cache::touch()`
Extend an existing cache item's TTL **without** fetching + re-storing the value.

### 2.8 **Route metadata**
`->metadata()` attaches arbitrary structured data that rides the whole routing pipeline
(SEO tags, feature flags, permissions).

---

## 3. Structural context interviewers love (the 11 → 13 "slim skeleton")

Laravel 11 restructured the app skeleton and **13 keeps it**. Know this cold — it's a common question:

- **No `app/Http/Kernel.php`, no `app/Console/Kernel.php`.** Config moved to **`bootstrap/app.php`**
  (see this clone) via a fluent builder: `->withRouting() ->withMiddleware() ->withExceptions()`.
- **Middleware** is registered in `bootstrap/app.php`, not a Kernel array. Default middleware classes
  are gone from `app/Http/Middleware` — customize via the `Middleware` object.
- **Providers** live in **`bootstrap/providers.php`** (a returned array), not `config/app.php`.
- **Slimmer `config/`** — many config files are optional; publish only what you override.
- **Routes:** default files are `routes/web.php` + `routes/console.php`. **`api.php` is opt-in**
  (`php artisan install:api`, which also wires **Sanctum**). `channels.php` opt-in too.
- **Health check** built in: `->withRouting(health: '/up')` exposes a `/up` endpoint.
- Default DB in new apps is **SQLite**; queue/cache/session default to the `database` driver.

Actual `bootstrap/app.php` from this clone:
```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(web: __DIR__.'/../routes/web.php',
                  commands: __DIR__.'/../routes/console.php', health: '/up')
    ->withMiddleware(function (Middleware $middleware): void { /* ... */ })
    ->withExceptions(function (Exceptions $exceptions): void {
        $exceptions->shouldRenderJsonWhen(
            fn (Request $r) => $r->is('api/*') || $r->expectsJson());
    })->create();
```

---

## 4. Adjacent ecosystem worth name-dropping
- **Starter kits** rebuilt around **React / Vue / Livewire** (Breeze/Jetstream deprecated in favor of the new kits).
- **Pest** is the default test runner in new kits (PHPUnit still supported; this skeleton ships PHPUnit 12.5).
- **First-party infra/monitoring:** Cloud, Forge, **Nightwatch** (monitoring), Vapor, **Reverb** (WebSockets), **Pulse**, **Horizon**, **Pennant** (feature flags), **Pint** (formatter), **Sail**.

---

## 5. Likely interview Q&A (quick answers)

- **"What's new in the latest Laravel?"** → v13 (Mar 2026): first-party **AI SDK** + **vector search**,
  **JSON:API** resources, expanded **PHP attributes**, `Queue::route()`, hardened CSRF
  (`PreventRequestForgery`), `Cache::touch()`, route metadata. Requires **PHP 8.3+**. Near-zero breaking changes.
- **"Where did the HTTP Kernel go?"** → Removed in 11, still gone in 13. Middleware/exceptions/routing
  now configured fluently in **`bootstrap/app.php`**; providers in **`bootstrap/providers.php`**.
- **"How do you add an API in a new Laravel app?"** → `php artisan install:api` — creates `routes/api.php`
  and installs **Sanctum**. It's no longer scaffolded by default.
- **"Attributes vs. properties on a model?"** → Both valid in 13; attributes are declarative/colocated,
  properties are classic. Attributes are additive and **non-breaking**.
- **"How would you do semantic search?"** → Postgres + pgvector, generate embeddings via the AI SDK /
  `Str::toEmbeddings()`, query with `whereVectorSimilarTo(...)`, vector index defined in the migration.
- **"Upgrade effort 12 → 13?"** → Low. Main gate is **PHP 8.3 minimum**; otherwise mostly additive.

---

## 6. To actually run this clone
```bash
# Needs PHP 8.3+  (XAMPP here is 8.2 — install 8.3+ first, or use Laravel Herd/Sail)
composer install
cp .env.example .env       # Windows: copy .env.example .env
php artisan key:generate
php artisan migrate         # SQLite by default
php artisan serve           # http://127.0.0.1:8000  ·  health: /up
```

---

### Sources
- [Laravel 13 Release Notes (official)](https://laravel.com/docs/13.x/releases)
- [Laravel AI SDK](https://laravel.com/ai)
- [What We Know About Laravel 13 — Laravel News](https://laravel-news.com/laravel-13)
