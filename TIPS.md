# LARAVEL TIPS

## tinker

```php
DB::listen( fn($q) => dump($q->sql, $q->bindings))
```
