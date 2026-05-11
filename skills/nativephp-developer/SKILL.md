---
name: nativephp-development
description: >-
  Build native desktop and mobile applications with NativePHP and Laravel.
  Use when creating, configuring, or working on NativePHP apps — including
  running on simulators/devices, accessing native APIs (camera, biometrics,
  notifications, geolocation, secure storage), building EDGE native UI
  components, handling deep links, push notifications, and deploying to the
  App Store or Google Play.
---

# NativePHP Development

## Overview

NativePHP lets you build true native desktop (Electron/Tauri) and mobile
(iOS/Android) apps using PHP and Laravel — no Swift, Kotlin, or JavaScript
frameworks required. PHP is bundled directly into the app and bridges into
native OS/device APIs via facades and events.

There are two separate packages:

- **Mobile** (`nativephp/mobile`) — iOS & Android, runs inside Swift/Kotlin shell
- **Desktop** (`nativephp/electron` or `nativephp/tauri`) — macOS, Windows, Linux

## When to Use This Skill

- Creating a new NativePHP mobile or desktop app
- Installing and configuring NativePHP in an existing Laravel project
- Accessing native device APIs (camera, biometrics, GPS, scanner, etc.)
- Building EDGE native UI components (TopBar, BottomNav, SideNav)
- Implementing deep links, push notifications, or offline-first patterns
- Running the app on simulators or real devices
- Preparing and submitting builds to App Store / Google Play

---

## Mobile (iOS & Android)

### Installation

```bash
composer require nativephp/mobile
php artisan native:install
```

Requires: PHP 8.2+, Laravel 11+, Xcode (iOS), Android Studio (Android).

### Dev Commands

```bash
php artisan native:run           # Run on default simulator
php artisan native:run ios       # iOS simulator
php artisan native:run android   # Android emulator
php artisan native:run --watch   # Hot reload on simulator
php artisan native:build ios     # Production build
php artisan native:build android
```

### Configuration

Publish and edit `config/nativephp.php`:

```php
return [
    'app_id'      => env('NATIVEPHP_APP_ID', 'com.example.myapp'),
    'app_name'    => env('APP_NAME', 'My App'),
    'app_version' => env('NATIVEPHP_APP_VERSION', '1.0.0'),
    'deeplink_scheme' => env('NATIVEPHP_DEEPLINK_SCHEME', 'myapp'),
];
```

### Routing

Mobile apps use standard Laravel routing. The entry point is the `/` route.
Keep routes stateless-friendly — sessions persist on-device via SQLite.

```php
// routes/web.php
Route::get('/', [HomeController::class, 'index']);
Route::get('/profile', [ProfileController::class, 'show'])->middleware('auth');
```

### Database (Offline-First)

Mobile apps use **SQLite on-device** via Laravel Eloquent — the full ORM works
exactly as in web apps.

```php
// .env
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database.sqlite  # auto-resolved by NativePHP
```

Run migrations on first boot using the `AppServiceProvider`:

```php
public function boot(): void
{
    if (app()->runningInNativePHP()) {
        Artisan::call('migrate', ['--force' => true]);
    }
}
```

---

## Native API Plugins (Mobile)

Install core plugins as needed:

```bash
composer require nativephp/camera
composer require nativephp/secure-storage
composer require nativephp/geolocation
composer require nativephp/biometrics
composer require nativephp/scanner
composer require nativephp/push-notifications  # Firebase
```

### Camera

```php
use Native\Mobile\Facades\Camera;

Camera::getPhoto(function (string $path) {
    // $path is a temporary local file path
    Storage::disk('local')->put('photo.jpg', file_get_contents($path));
});
```

### Secure Storage (Keychain / Keystore)

```php
use Native\Mobile\Facades\SecureStorage;

SecureStorage::set('api_token', $token);
$token = SecureStorage::get('api_token');
SecureStorage::remove('api_token');
```

Never store sensitive credentials in SQLite or the session. Always use
`SecureStorage` for tokens, passwords, and API keys.

### Biometrics

```php
use Native\Mobile\Facades\Biometrics;

Biometrics::promptForBiometricID()
    ->on('success', function () {
        // Unlock the app or sensitive feature
    })
    ->on('failure', function (string $reason) {
        // Handle failure
    });
```

### Geolocation

```php
use Native\Mobile\Facades\Geolocation;

Geolocation::getCurrentPosition(function (float $lat, float $lng) {
    // Use coordinates
});
```

### Push Notifications (Firebase)

```php
use Native\Mobile\Facades\PushNotifications;

// Get the FCM token to send to your server
PushNotifications::getToken(function (string $token) {
    Http::post('/api/device-tokens', ['token' => $token]);
});
```

Listen for incoming notifications in an event listener:

```php
use Native\Mobile\Events\PushNotificationReceived;

Event::listen(PushNotificationReceived::class, function ($event) {
    // $event->data contains the notification payload
});
```

### QR / Barcode Scanner

```php
use Native\Mobile\Facades\Scanner;

Scanner::scan(function (string $value, string $format) {
    // $format: QR_CODE, EAN_13, etc.
});
```

### Network Status

```php
use Native\Mobile\Facades\Network;

if (Network::isOnline()) {
    // Sync with remote API
}
```

---

## EDGE Native UI Components (Mobile v3+)

EDGE renders truly native UI elements (TopBar, BottomNav, SideNav) via
JSON structures generated from Blade middleware — not web HTML.

### Top Bar

```php
use Native\Mobile\EDGE\TopBar;

TopBar::make()
    ->title('Dashboard')
    ->leftButton(TopBar::backButton())
    ->rightButton(
        TopBar::button('settings', 'gear')
            ->onTap('openSettings')
    );
```

### Bottom Navigation

```php
use Native\Mobile\EDGE\BottomNav;

BottomNav::make()
    ->items([
        BottomNav::item('Home', 'house')->route('/'),
        BottomNav::item('Profile', 'person')->route('/profile'),
        BottomNav::item('Settings', 'gear')->route('/settings'),
    ]);
```

### Side Navigation (Drawer)

```php
use Native\Mobile\EDGE\SideNav;

SideNav::make()
    ->header('My App')
    ->items([
        SideNav::item('Dashboard', 'chart.bar')->route('/'),
        SideNav::item('Logout', 'arrow.right.square')->action('logout'),
    ]);
```

Register EDGE components inside a middleware and apply it to relevant routes:

```php
// app/Http/Middleware/NativeUiMiddleware.php
public function handle(Request $request, Closure $next): Response
{
    TopBar::make()->title(config('app.name'));
    BottomNav::make()->items([...]);

    return $next($request);
}
```

---

## Events (Mobile)

NativePHP fires Laravel events when native actions occur. Listen in
`EventServiceProvider` or route event listeners:

```php
use Native\Mobile\Events\AppForegrounded;
use Native\Mobile\Events\AppBackgrounded;
use Native\Mobile\Events\DeepLinkReceived;

Event::listen(AppForegrounded::class, function () {
    // Re-sync data, refresh token, etc.
});

Event::listen(DeepLinkReceived::class, function ($event) {
    // $event->url — parse and redirect accordingly
    $path = parse_url($event->url, PHP_URL_PATH);
    redirect($path);
});
```

---

## Deep Links

Configure the scheme in `config/nativephp.php`:

```php
'deeplink_scheme' => 'myapp', // myapp://path/to/screen
```

Handle in a route or event listener:

```php
Event::listen(DeepLinkReceived::class, function ($event) {
    $uri = str_replace('myapp://', '', $event->url);
    redirect('/' . ltrim($uri, '/'));
});
```

---

## Authentication (Mobile)

Use standard Laravel Auth. For mobile, prefer token-based auth stored in
`SecureStorage` rather than sessions:

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

---

## Desktop (Electron)

### Installation

```bash
composer require nativephp/electron
php artisan native:install
```

### Dev & Build Commands

```bash
php artisan native:serve           # Start dev app
php artisan native:build           # Build distributable
php artisan native:build --win     # Windows
php artisan native:build --mac     # macOS
php artisan native:build --linux   # Linux
```

### Desktop Native APIs

```php
use Native\Laravel\Facades\Notification;
use Native\Laravel\Facades\Menu;
use Native\Laravel\Facades\Window;
use Native\Laravel\Facades\Clipboard;
use Native\Laravel\Facades\System;

// System notification
Notification::title('Hello!')
    ->message('Your task is complete.')
    ->send();

// Menu bar
Menu::create(function (Menu $menu) {
    $menu->label('File')
        ->submenu(function (Menu $sub) {
            $sub->link('Open', 'https://myapp.test/open');
            $sub->separator();
            $sub->quit();
        });
});

// Open/manage windows
Window::open()
    ->width(800)
    ->height(600)
    ->route('dashboard')
    ->title('Dashboard');

// Clipboard
Clipboard::text('Copied to clipboard!');
```

### Desktop Events

```php
use Native\Laravel\Events\App\ApplicationBooted;
use Native\Laravel\Events\Windows\WindowClosed;

Event::listen(ApplicationBooted::class, fn() => /* bootstrap */);
Event::listen(WindowClosed::class, fn($e) => /* cleanup $e->id */);
```

---

## Anti-Patterns

- **Do not** use server-side sessions for auth in mobile — use `SecureStorage`
- **Do not** assume internet connectivity — always guard with `Network::isOnline()`
- **Do not** store tokens, passwords, or PII in SQLite — use `SecureStorage`
- **Do not** call `php artisan migrate:fresh` in production boot — use `--step`
  or check migration status first
- **Do not** use absolute file paths for storage — use `Storage::disk('local')`
- **Do not** run `npm run dev` for mobile — use `php artisan native:run --watch`

---

## Deployment

### Mobile — App Store / Google Play

1. Set `NATIVEPHP_APP_ID`, `NATIVEPHP_APP_VERSION` in `.env`
2. Build: `php artisan native:build ios` / `native:build android`
3. Upload via Xcode Organizer (iOS) or Google Play Console (Android)
4. Use **Bifrost** (`bifrost.nativephp.com`) for cloud builds if macOS is unavailable

### Desktop — Distribution

```bash
php artisan native:build --mac    # .dmg
php artisan native:build --win    # .exe installer
php artisan native:build --linux  # .AppImage / .deb
```

Enable auto-updates by configuring `updater` in `config/nativephp.php`.

---

## Reference

- Mobile docs: https://nativephp.com/docs/mobile/3/getting-started/introduction
- Desktop docs: https://nativephp.com/docs/desktop/2/getting-started/introduction
- Cloud builds: https://bifrost.nativephp.com
- Plugins: https://nativephp.com/plugins
