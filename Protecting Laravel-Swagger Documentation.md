# DarkaOnLine/L5-Swagger

`#Laravel`

## Introduction

The [`DarkaOnLine/L5-Swagger`](https://github.com/DarkaOnLine/L5-Swagger#disclaimer) package is a wrapper of Swagger-php and swagger-ui adapted to work with Laravel. 

## Installation

Due to the supported versions of different Laravel versions, please visit the [installation & configuration documentation](https://github.com/DarkaOnLine/L5-Swagger/wiki/Installation-&-Configuration) for better information.

## Usage

To be continue ...

## 🚧 Protect your documentation

By default, our documentation are open to the public, therefore we can protect our documentation by adding middleware into the `l5-swagger.php` config file.

#### 1. Publish the config file
```bash
php artisan vendor:publish --provider "L5Swagger\L5SwaggerServiceProvider"
```

#### 2. Adding your own middleware into `config('l5-swagger.defaults.middleware.api')`

#### 3. Setting up the Web Auth Page (login & logout function). 

For the Web Auth Page, you can:
- use the [`laravel/ui`](https://github.com/laravel/ui#supported-versions) package based on your Laravel Version.
- setup by your own

#### 4. Done.

---

## Example

\*This example is based on my previous project with Larvel-Vue SPA.

#### Config

Modify config/l5-swagger.php

```diff config/l5-swagger.php
'defaults'=> [
	// ...,
    'middleware' => [
-       'api' => [],
+       'api' => [
+			'web',
+			'auth:web',
+			'checkRoles:99' // some extra middleware
+		],
        'asset' => [],
        'docs' => [],
        'oauth2_callback' => [],
    ],
    // ...,

```

#### Web Route

Modify routes/web.php

```php routes/web.php
// without auth
Route::get('/watcher',[App\Http\Controllers\HomeController::class, 'showLoginForm'])->name('login');
Route::post('/watcher',[App\Http\Controllers\HomeController::class, 'login']);
Route::post('/logout',[App\Http\Controllers\HomeController::class, 'logout'])->name('logout');

// with auth 
Route::view('/watcher/home','home')->middleware(['auth:web']);

// SPA
Route::get('/{any?}',[App\Http\Controllers\HomeController::class, 'index'])->where('any', '^(?!api\/)[\/\w\.-]*');
```

#### Controller

Modified HomeController.php

```diff HomeController.php
namespace App\Http\Controllers;

+ use Illuminate\Http\Request;
+ use Illuminate\Foundation\Auth\AuthenticatesUsers;

class HomeController extends Controller
{
+	use AuthenticatesUsers;

+    protected $redirectTo = 'watcher/home';

+	// @override
+    protected function loggedOut(Request $request)
+    {
+        return redirect('/watcher');
+    }

	// some other codes
```

#### Views

Create layouts/app.blade.php & home.blade.php

```php layouts/app.blade.php
<!doctype html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <!-- CSRF Token -->
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <title>{{ config('app.name', 'Laravel') }}</title>

    <!-- Scripts -->
    <!-- <script src="{{ asset('js/app.js') }}" defer></script> -->

    <!-- Fonts -->
    <link rel="dns-prefetch" href="//fonts.gstatic.com">
    <link href="https://fonts.googleapis.com/css?family=Nunito" rel="stylesheet">

    <!-- Styles -->
    <link href="{{ asset('css/laravel.css') }}" rel="stylesheet">
</head>
<body>
    <div id="app">
        <nav class="navbar navbar-expand-md navbar-light bg-white shadow-sm">
            <div class="container">
                <a class="navbar-brand" href="{{ url('/') }}">
                    {{ config('app.name', 'Laravel') }}
                </a>
                <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="{{ __('Toggle navigation') }}">
                    <span class="navbar-toggler-icon"></span>
                </button>
            </div>
        </nav>

        <main class="py-4">
            @yield('content')
        </main>
    </div>
</body>
</html>
```

```php home.blade.php
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">Dashboard</div>

                <div class="card-body">
                    @if (session('status'))
                        <div class="alert alert-danger" role="alert">
                            {{ session('status') }}
                        </div>
                    @endif

                    @auth
                    <ul>
                        <li><a href="/api/documentation">Swagger Documentation</a></li>
                        <li>
                            <a
                            href="{{ url('/logout') }}" 
                            onclick="event.preventDefault();document.getElementById('logout-form').submit();"
                            >
                                Logout
                            </a>
                        </li>
                    </ul>
                    <form id="logout-form" action="{{ route('logout') }}" method="POST" style="display: none;">@csrf</form>
                    @endAuth
                </div>
            </div>
        </div>
    </div>
</div>
@endsection

```

#### Extra - CheckRole Middleware

Modified Middleware/CheckRole.php

```diff
-    return response()->json([
-        'status' => 403,
-        'message' => 'Unauthorized'
-    ], 403);
+    if($request->wantsJson()){
+        return response()->json([
+            'status' => 403,
+            'message' => 'Unauthorized'
+        ], 403);
+    }else{
+        $request->session()->flash('status', 'Unauthorized.');
+        return back();
+    }

```