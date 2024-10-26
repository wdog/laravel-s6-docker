# LARAVEL TIPS

## tinker

```php
DB::listen( fn($q) => dump($q->sql, $q->bindings))
```


```php
DB::enableQueryLog();
DB::getQueryLog();
```


## jetstream dark mode swticher

app.js

```javascript
window.themeSwitcher = function () {
    return {
        switchOn: JSON.parse(localStorage.getItem("isDark")) || false,
        switchTheme() {
            if (this.switchOn) {
                document.documentElement.classList.add("dark");
            } else {
                document.documentElement.classList.remove("dark");
            }
            localStorage.setItem("isDark", this.switchOn);
        },
    };
};

```

tailwindcss

```css
darkMode: 'class', // or 'media' if you want to use the OS setting
```

app.blade.php

```html
<body x-data="themeSwitcher()" :class="{ 'dark': switchOn }">
```

navigation.blade.php

```html
<div x-data="window.themeSwitcher()" x-init="switchTheme()" @keydown.window.tab="switchOn = false"
     class="flex items-center justify-center space-x-2">
    <input id="thisId" type="checkbox" name="switch" class="hidden" :checked="switchOn">

    <button
            x-ref="switchButton"
            type="button"
            @click="switchOn = ! switchOn; switchTheme()"
            :class="switchOn ? 'bg-blue-600' : 'bg-neutral-200'"
            class="relative ml-4 inline-flex h-6 w-10 rounded-full py-0.5 focus:outline-none">
        <span :class="switchOn ? 'translate-x-[18px]' : 'translate-x-0.5'"
              class="w-5 h-5 duration-200 ease-in-out bg-white rounded-full shadow-md"></span>
    </button>

    <label @click="$refs.switchButton.click(); $refs.switchButton.focus()" :id="$id('switch')"
           :class="{ 'text-blue-600': switchOn, 'text-gray-400': !switchOn }"
           class="text-sm select-none">
        Dark Mode
    </label>
</div>
```
