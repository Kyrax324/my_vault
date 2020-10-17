# Develop Vue with Hot-reload [ Laravel-Mix ]

Run :
```bash
npm run hot
```

### # Pre-sequence && Configuration

* make sure your webpack's versioning is not enabled in dev mode.
* using mix() function in your SPA blade.php
* adding *'--disable-host-check'* into *'scripts.hot'* in 'package.json'

*****webpack.min.js*****
```js
mix.js('resources/js/app.js', 'public/js')
    .sass('resources/sass/app.scss', 'public/css');

if (mix.inProduction()) {
    mix.version();
}

// uncomment this section if wish use different port
// mix.options({
//     hmrOptions: {
//         host: 'localhost',
//         port: '1214' // any port (default port 8080)
//     }
// });
````

*****welcome.blade.php*****
```html
<script src="{{ mix('js/app.js') }}" defer></script>
<link href="{{ mix('css/app.css') }}" rel="stylesheet">
```

*****package.json*****
```json
"hot": "cross-env NODE_ENV=development node_modules/webpack-dev-server/bin/webpack-dev-server.js --inline --hot --disable-host-check --config=node_modules/laravel-mix/setup/webpack.config.js",

```

### # Reference

[Hot Module Replacement | Laravel Mix Documentation](https://laravel-mix.com/docs/5.0/hot-module-replacement)



