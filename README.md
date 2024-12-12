# WordPress Theme with Vite Integration

Modern WordPress theme development environment using Vite for hot module replacement, SCSS processing, and optimized production builds.

## Setup

1. Install Dependencies
```bash
cd wp-content/themes/your-theme
npm init -y
npm install vite @vitejs/plugin-react sass --save-dev


2. Add to wp-config.php
define('IS_VITE_DEVELOPMENT', true); // Set to false in production

3. Create Directory Structure
mkdir -p src/scss src/js
touch src/main.js src/scss/main.scss src/js/app.js

4. Create Configuration Files
vite.config.js:

import { defineConfig } from 'vite'

export default defineConfig({
    build: {
        outDir: 'dist',
        manifest: true,
        rollupOptions: {
            input: 'src/main.js'
        }
    },
    server: {
        cors: true,
        port: 3000,
        strictPort: true,
        hmr: {
            host: 'localhost'
        }
    },
    css: {
        preprocessorOptions: {
            scss: {
                additionalData: `
                    @import "./src/scss/variables";
                    @import "./src/scss/mixins";
                `
            }
        }
    }
})


5. Add to package.json:
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}


6. Add to functions.php:

function theme_enqueue_assets() {
    if (defined('IS_VITE_DEVELOPMENT') && IS_VITE_DEVELOPMENT === true) {
        add_filter('script_loader_tag', function($tag, $handle, $src) {
            if (in_array($handle, ['vite-client', 'theme-scripts'])) {
                return '<script type="module" src="' . esc_url($src) . '"></script>';
            }
            return $tag;
        }, 10, 3);

        wp_enqueue_script('vite-client', 'http://localhost:3000/@vite/client', [], null);
        wp_enqueue_script('theme-scripts', 'http://localhost:3000/src/main.js', [], null);
    } else {
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


Development
1. Start Vite development server:
npm run dev

2. Start WordPress development server

php -S localhost:8000

Your development environment now has:

Hot Module Replacement
SCSS processing
ES6+ JavaScript support
Asset optimization
Source maps


Production
1. Build optimized assets:
npm run build

2. Set IS_VITE_DEVELOPMENT to false in wp-config.php

3. Deploy theme with the dist/ directory


File Structure
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


Features
Hot Module Replacement (HMR)
SCSS support
Modern JavaScript (ES6+)
Asset optimization
Production builds
Source maps in development
WordPress integration
