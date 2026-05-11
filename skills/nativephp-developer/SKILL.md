---
name: NativePHP Development
description: >-
  Build native desktop and mobile applications with NativePHP and Laravel.
  Use when creating or working on NativePHP apps — including native API access
  (camera, biometrics, geolocation, secure storage, push notifications),
  EDGE native UI components, deep links, offline-first SQLite databases,
  and deploying to the App Store, Google Play, or as desktop distributables.
compatible_agents:
  - Claude Code
  - Cursor
  - Windsurf
  - GitHub Copilot
tags:
  - nativephp
  - mobile
  - desktop
  - ios
  - android
  - native
  - offline
---

# NativePHP Development

## Context

NativePHP lets Laravel developers build true native desktop and mobile apps
using PHP — no Swift, Kotlin, or JavaScript frameworks required. PHP is
compiled and bundled into the app, with bridges into native OS/device APIs
exposed via familiar Laravel facades.

Two separate packages exist:

- **Mobile** (`nativephp/mobile`) — iOS & Android via Swift/Kotlin shell (v3+)
- **Desktop** (`nativephp/electron`) — macOS, Windows, Linux via Electron

The entire Laravel ecosystem works on-device: Eloquent ORM, routing,
Artisan, events, middleware, queues. No web server is required — the app
runs completely offline-first.

## Rules

### General
- Always check `app()->runningInNativePHP()` before running native-only code
- Use `Storage::disk('local')` for file paths — never hardcode absolute paths
- Run migrations on first boot via `AppServiceProvider::boot()`, not manually
- Prefer `--force` with migrations only in production NativePHP context
- Use standard Laravel routing; the entry point is always the `/` route

### Mobile
- Use **SQLite** as the on-device database (`DB_CONNECTION=sqlite`)
- Never store tokens, passwords, or PII in SQLite — always use `SecureStorage`
- Never assume internet connectivity — always guard with `Network::isOnline()`
- Use token-based auth stored in `SecureStorage`, not Laravel sessions
- Register EDGE components (TopBar, BottomNav) inside middleware, not controllers
- Use `php artisan native:run --watch` for hot reload during development
- Never run `npm run dev` for mobile — NativePHP handles assets via Vite internally

### Desktop
- Use `Window::open()` to manage application windows programmatically
- Define `Menu::create()` in a `NativeAppServiceProvider` to keep it organized
- Use `Notification::send()` for OS-level notifications, not browser alerts

## Examples

### Installation (Mobile)

```bash
composer require nativephp/mobile
php artisan native:install
```

### Configuration (`config/nativephp.php`)

```php
return [
    'app_id'          => env('NATIVEPHP_APP_ID', 'com.example.myapp'),
    'app_name'        => env('APP_NAME', 'My App'),
    'app_version'     => env('NATIVEPHP_APP_VERSION', '1.0.0'),
    'deeplink_scheme' => env('NATIVEPHP_DEEPLINK_SCHEME', 'myapp'),
];
```

### Dev Commands (Mobile)

```bash
php artisan native:run              # default simulator
php artisan native:run ios          # iOS simulator
php artisan native:run android      # Android emulator
php artisan native:run --watch      # hot reload
php artisan native:build ios        # production build
php artisan native:build android
```

### Auto-migrate on Boot

```php
// app/Providers/AppServiceProvider.php
public function boot(): void
{
    if (app()->runningInNativePHP()) {
        Artisan::call('migrate', ['--force' => true]);
    }
}
```

### Offline-First SQLite

```php
// .env
DB_CONNECTION=sqlite
```

Use standard Eloquent — everything works identically to a web Laravel app.

### Native API: Camera

```php
use Native\Mobile\Facades\Camera;

Camera::getPhoto(function (string $path) {
    Storage::disk('local')->put('photo.jpg', file_get_contents($path));
});
```

### Native API: SecureStorage (Keychain / Keystore)

```php
use Native\Mobile\Facades\SecureStorage;

// Store
SecureStorage::set('api_token', $token);

// Retrieve
$token = SecureStorage::get('api_token');

// Delete
SecureStorage::remove('api_token');
```

### Native API: Biometrics

```php
use Native\Mobile\Facades\Biometrics;

Biometrics::promptForBiometricID()
    ->on('success', function () {
        // Proceed to authenticated area
    })
    ->on('failure', function (string $reason) {
        // Handle failure
    });
```

### Native API: Geolocation

```php
use Native\Mobile\Facades\Geolocation;

Geolocation::getCurrentPosition(function (float $lat, float $lng) {
    // Use coordinates
});
```

### Native API: Push Notifications (Firebase)

```php
use Native\Mobile\Facades\PushNotifications;
use Native\Mobile\Events\PushNotificationReceived;

// Register device token
PushNotifications::getToken(function (string $token) {
    Http::post('/api/device-tokens', ['token' => $token]);
});

// Handle incoming notifications
Event::listen(PushNotificationReceived::class, function ($event) {
    // $event->data contains the notification payload
});
```

### Native API: QR / Barcode Scanner

```php
use Native\Mobile\Facades\Scanner;

Scanner::scan(function (string $value, string $format) {
    // $format: QR_CODE, EAN_13, CODE_128, etc.
});
```

### Native API: Network Status

```php
use Native\Mobile\Facades\Network;

if (Network::isOnline()) {
    // Sync with remote API
}
```

### EDGE Native UI: TopBar + BottomNav

```php
use Native\Mobile\EDGE\TopBar;
use Native\Mobile\EDGE\BottomNav;

// In middleware
TopBar::make()
    ->title('Dashboard')
    ->rightButton(TopBar::button('settings', 'gear')->onTap('openSettings'));

BottomNav::make()
    ->items([
        BottomNav::item('Home', 'house')->route('/'),
        BottomNav::item('Profile', 'person')->route('/profile'),
        BottomNav::item('Settings', 'gear')->route('/settings'),
    ]);
```

### Events

```php
use Native\Mobile\Events\AppForegrounded;
use Native\Mobile\Events\AppBackgrounded;
use Native\Mobile\Events\DeepLinkReceived;

Event::listen(AppForegrounded::class, function () {
    // Re-sync data, refresh tokens
});

Event::listen(DeepLinkReceived::class, function ($event) {
    $path = str_replace('myapp://', '/', $event->url);
    redirect($path);
});
```

### Token-Based Auth (Mobile Pattern)

```php
// Login
$token = auth()->attempt($credentials)
    ? auth()->user()->createToken('mobile')->plainTextToken
    : null;

SecureStorage::set('auth_token', $token);

// Logout
SecureStorage::remove('auth_token');
auth()->logout();
```

### Desktop: Window, Menu, Notification

```php
use Native\Laravel\Facades\Window;
use Native\Laravel\Facades\Menu;
use Native\Laravel\Facades\Notification;

Window::open()->width(900)->height(600)->route('dashboard');

Menu::create(fn(Menu $menu) =>
    $menu->label('File')->submenu(fn(Menu $sub) =>
        $sub->link('Open', '/open')->separator()->quit()
    )
);

Notification::title('Done!')->message('Build completed.')->send();
```

### Desktop Build Commands

```bash
php artisan native:serve          # dev
php artisan native:build --mac    # .dmg
php artisan native:build --win    # .exe
php artisan native:build --linux  # .AppImage
```

## Anti-Patterns

- **Don't** use server-side sessions for auth in mobile — use `SecureStorage`
- **Don't** assume internet is available — always check `Network::isOnline()`
- **Don't** store sensitive data in SQLite — use `SecureStorage`
- **Don't** hardcode file paths — use `Storage::disk('local')`
- **Don't** run `npm run dev` for mobile — use `php artisan native:run --watch`
- **Don't** call `migrate:fresh` on boot in production — use `--force` with care

## References

- [NativePHP Mobile Docs](https://nativephp.com/docs/mobile/3/getting-started/introduction)
- [NativePHP Desktop Docs](https://nativephp.com/docs/desktop/2/getting-started/introduction)
- [EDGE Components](https://nativephp.com/docs/mobile/3/edge-components/introduction)
- [Core Plugins](https://nativephp.com/plugins)
- [Bifrost Cloud Builds](https://bifrost.nativephp.com)
