# WordPress Theme with Vite Integration

A modern WordPress theme development environment using **Vite** for hot module replacement, SCSS processing, and optimized production builds.

---

## Features

- **Hot Module Replacement (HMR)**
- **SCSS support**
- **Modern JavaScript (ES6+)**
- **Asset optimization**
- **Production builds**
- **Source maps in development**
- **Seamless WordPress integration**

---

## Setup

### 1. Install Dependencies

```bash
cd wp-content/themes/your-theme
npm init -y
npm install vite @vitejs/plugin-react sass --save-dev
```

### 2. Add to `wp-config.php`

```php
define('IS_VITE_DEVELOPMENT', true); // Set to false in production
```

### 3. Create Directory Structure

```bash
mkdir -p src/scss src/js
touch src/main.js src/scss/main.scss src/js/app.js src/scss/variables.scss src/scss/mixins.scss
```

Add the following to `src/main.js`:

```javascript
import "./scss/main.scss";
import "./js/app.js";
```

### 4. Create Configuration Files

#### `vite.config.js`
```bash
touch vite.config.js
```

```javascript
import { defineConfig } from "vite";

export default defineConfig({
  build: {
    outDir: "dist",
    manifest: true,
    rollupOptions: {
      input: "src/main.js",
    },
  },
  server: {
    cors: true,
    port: 3000,
    strictPort: true,
    hmr: {
      host: "localhost",
    },
    css: {
      preprocessorOptions: {
        scss: {
          additionalData: `
          @import "./src/scss/variables";
          @import "./src/scss/mixins";
        `,
        },
      },
    },
  },
});

```

### 5. Add to `package.json`

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

### 6. Add to `functions.php`

Copy code if functions.php don't exists
```bash
touch functions.php
```
```php
<?php

function theme_enqueue_assets() {
    // Development mode
    if (defined('IS_VITE_DEVELOPMENT') && IS_VITE_DEVELOPMENT === true) {
       add_action('wp_footer', function() {
            echo '<script type="importmap">';
            echo json_encode([
                'imports' => [
                    'vite' => 'http://localhost:3000/@vite/client',
                ]
            ]);
            echo '</script>';
            echo '<script type="module" src="http://localhost:3000/@vite/client"></script>';
            echo '<script type="module" src="http://localhost:3000/src/main.js"></script>';
        });
    } 
    // Production mode
    else {
        if (file_exists(get_template_directory() . '/dist/manifest.json')) {
            $manifest = json_decode(file_get_contents(get_template_directory() . '/dist/manifest.json'), true);
            
            if (isset($manifest['src/main.js']['css'])) {
                foreach ($manifest['src/main.js']['css'] as $cssFile) {
                    wp_enqueue_style('theme-styles', get_template_directory_uri() . '/dist/' . $cssFile, [], null);
                }
            }
            
            add_action('wp_footer', function() use ($manifest) {
                $js_file = $manifest['src/main.js']['file'];
                echo '<script type="module" src="' . get_template_directory_uri() . '/dist/' . $js_file . '"></script>';
            });
        }
    }
}
add_action('wp_enqueue_scripts', 'theme_enqueue_assets');
```

---

## Development

### 1. Start Vite Development Server

```bash
npm run dev
```

### 2. Start WordPress Development Server

```bash
php -S localhost:8000
```

Now you have:

- Hot Module Replacement
- SCSS processing
- ES6+ JavaScript support
- Asset optimization
- Source maps

---

## Production

### 1. Build Optimized Assets

```bash
npm run build
```

### 2. Update `wp-config.php`

Set `IS_VITE_DEVELOPMENT` to `false`:

```php
define('IS_VITE_DEVELOPMENT', false);
```

### 3. Deploy Theme with `dist/` Directory

Ensure the `dist/` directory is included when deploying the theme.

---

## File Structure

```plaintext
theme/
├── src/
│   ├── scss/
│   │   ├── main.scss
│   │   ├── variables.scss
│   │   └── mixins.scss
│   ├── js/
│   │   └── app.js
│   └── main.js
├── dist/
├── vite.config.js
├── package.json
└── functions.php
```

