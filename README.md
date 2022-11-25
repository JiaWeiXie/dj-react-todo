# Django + Vite + React (TypeScript)

## Django

```shell
$ poetry new <project-name> --src
$ cd <project-name>
$ poetry add django
$ poetry shell
$ cd src
$ django-admin startproject core .
$ mv <project-name> app 
$ vi pyproject.toml
remove packages = [{include = "<project-name>", from = "src"}]

```

## Config Django template

```shell
$ mkdir -p src/templates
$ cd src/templates
$ touch index.html
```

index.html
```html
<!DOCTYPE html>
{% load static %}
{% load django_vite %}
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    {% if DEBUG %}
        <script type="module">
            import RefreshRuntime from 'http://localhost:3000/@react-refresh'

            RefreshRuntime.injectIntoGlobalHook(window)
            window.$RefreshReg$ = () => {
            }
            window.$RefreshSig$ = () => (type) => type
            window.__vite_plugin_react_preamble_installed__ = true
        </script>

        {% vite_hmr_client %}
    {% endif %}
</head>
<body>

<div id="root"></div>

{% vite_asset 'src/main.tsx' %}
</body>
</html>
```

```shell
$ touch src/core/views.py
```

src/core/views.py
```python
from django.conf import settings
from django.views.generic import TemplateView


class FrontendView(TemplateView):
    template_name = "index.html"

    def get_context_data(self, **kwargs):
        kwargs["DEBUG"] = settings.DEBUG
        return super().get_context_data(**kwargs)
```

src/core/urls.py
```diff
+from django.conf import settings
+from django.conf.urls.static import static
from django.contrib import admin
from django.urls import path
+from core.views import FrontendView

urlpatterns = [
    path("admin/", admin.site.urls),
+    path("", FrontendView.as_view(), name="index"),
]

+if settings.DEBUG:
+    urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
+    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

```shell
$ poetry add django_vite whitenoise
```

settings.py
```diff
INSTALLED_APPS = [
+    "django_vite",
]


MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
+    "whitenoise.middleware.WhiteNoiseMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.common.CommonMiddleware",
    "django.middleware.csrf.CsrfViewMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
    "django.contrib.messages.middleware.MessageMiddleware",
    "django.middleware.clickjacking.XFrameOptionsMiddleware",
]


TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [
+            BASE_DIR / "templates",
        ],
        "APP_DIRS": True,
        "OPTIONS": {
            "context_processors": [
                "django.template.context_processors.debug",
                "django.template.context_processors.request",
                "django.contrib.auth.context_processors.auth",
                "django.contrib.messages.context_processors.messages",
            ],
        },
    },
]


# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/3.2/howto/static-files/

STATIC_URL = "static/"

+STATICFILES_STORAGE = "whitenoise.storage.CompressedManifestStaticFilesStorage"
+DJANGO_VITE_ASSETS_PATH = BASE_DIR / "static" / "dist"
+DJANGO_VITE_DEV_MODE = DEBUG

+FRONTEND_DIR = BASE_DIR / "frontend"

+STATICFILES_DIRS = [
    BASE_DIR / "static",
    DJANGO_VITE_ASSETS_PATH,
    FRONTEND_DIR / "public",
]

+if DEBUG:
+    STATICFILES_DIRS += [FRONTEND_DIR]

+STATIC_ROOT = BASE_DIR / "assets"
```

## React
```shell
$ mkdir -p src/frontend
$ cd src/frontend
$ yarn create vite . --template react-ts
```

vite.config.ts
```typescript
import {defineConfig} from 'vite'
import react from '@vitejs/plugin-react'

// https://vitejs.dev/config/
export default defineConfig({
    plugins: [react()],
    base: '/static/',
    server: {
        host: 'localhost',
        port: 3000,
        open: false,
        watch: {
            usePolling: true,
            disableGlobbing: false,
        },
        cors: {
            origin: '*',
            methods: 'GET,HEAD,PUT,PATCH,POST,DELETE',
            preflightContinue: false,
            optionsSuccessStatus: 204,
        },
    },
    build: {
        outDir: '../static/dist',
        assetsDir: './src/assets',
        manifest: true,
        target: 'es2015',
        rollupOptions: {
            input: {
                main: './src/main.tsx',
            }
        },
    },
})
```

```shell
$ yarn add -D sass
$ yarn add react-bootstrap bootstrap
```

main.tsx
```diff
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
+import './index.scss'
-import './index.css'
+import 'vite/modulepreload-polyfill';
+import 'bootstrap/dist/css/bootstrap.min.css';

ReactDOM.createRoot(document.getElementById('root') as HTMLElement).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```

# Run development server

```shell
$ cd src/frontend
$ yarn dev

$ cd src
$ python manage.py runserver 
```