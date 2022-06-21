---
title: Selector de  idiomas
description: 'Cuando **nuxt-i18n** se carga en su aplicación, agrega su configuración `locales` a `this.$i18n` (o `app.i18n`), lo que hace que sea muy fácil mostrar un selector de  idiomas en cualquier lugar de su aplicación.'
position: 10
category: Guía
---

Cuando **nuxt-i18n** se carga en su aplicación, agrega su configuración `locales` a `this.$i18n` (o `app.i18n`), lo que hace que sea muy fácil mostrar un selector de  idiomas en cualquier lugar de su aplicación.

Aquí hay un ejemplo de selector de idiomas donde se ha agregado una tecla `name` a cada objeto de configuración local para mostrar títulos más amigables para cada enlace:
```vue
<nuxt-link
  v-for="locale in availableLocales"
  :key="locale.code"
  :to="switchLocalePath(locale.code)">{{ locale.name }}</nuxt-link>
```

```js
computed: {
  availableLocales () {
    return this.$i18n.locales.filter(i => i.code !== this.$i18n.locale)
  }
}
```

```js {}[nuxt.config.js]
['nuxt-i18n', {
  locales: [
    {
      code: 'en',
      name: 'English'
    },
    {
      code: 'es',
      name: 'Español'
    },
    {
      code: 'fr',
      name: 'Français'
    }
  ]
}]
```

<alert type="info">

When using `detectBrowserLanguage` and wanting to persist locale on a route change, you must call one of the functions that update the stored locale cookie. Call either [`setLocaleCookie(locale)`](/api#setlocalecookie) to persist just the cookie locale or [`setLocale(locale)`](/api#setlocale) to both persist the cookie locale and switch the route to the specified locale. Otherwise, locale might switch back to the saved one during navigation.

</alert>

The template code might look like this, for example:
```vue
<a
  href="#"
  v-for="locale in availableLocales"
  :key="locale.code"
  @click.prevent.stop="$i18n.setLocale(locale.code)">{{ locale.name }}</a>
```

## Parámetros de ruta dinámica

Tratar con parámetros de ruta dinámicos requiere un poco más de trabajo porque necesita proporcionar traducciones de parámetros a **nuxt-i18n**.  Para este propósito, el módulo de tienda de **nuxt-i18n** expone una propiedad de estado `routeParams` que se fusionará con los parámetros de ruta al generar rutas del selector de idiomas con `switchLocalePath()`.

<alert type="warning">

Asegúrese de que Vuex [esté habilitado](https://nuxtjs.org/guides/directory-structure/store) en su aplicación y que no haya configurado la opción  `vuex` en `false` en opciones de **nuxt-i18n**.

</alert>

To provide dynamic parameters translations, dispatch the `i18n/setRouteParams` mutation as early as possible when loading a page. The passed in object must contain the mappings from the locale `code` to an object with a mapping from slug name to expected slug value for given locale.

An example (replace `postId` with the appropriate route parameter):

```vue
<template>
  <!-- pages/post/_postId.vue -->
</template>

<script>
export default {
  async asyncData ({ store }) {
    await store.dispatch('i18n/setRouteParams', {
      en: { postId: 'my-post' },
      fr: { postId: 'mon-article' }
    })
    return {
      // your data
    }
  }
}
</script>
```

<alert type="info">

**nuxt-i18n** no restablecerá las traducciones de parámetros por usted, esto significa que si utiliza parámetros idénticos para diferentes rutas, navegar entre esas rutas podría generar parámetros conflictivos. Asegúrese de establecer siempre traducciones de parámetros en tales casos.

</alert>

## Wait for page transition

By default, the locale will be changed right away when navigating to a route with a different locale which means that if you have a page transition, it will fade out the page with the text already switched to the new language and fade back in with the same content.

To work around the issue, you can set the option [`skipSettingLocaleOnNavigate`](./options-reference#skipsettinglocaleonnavigate) to `true` and handle setting the locale yourself from a `beforeEnter` transition hook defined in a plugin.

```js {}[nuxt.config.js]
export default {
  plugins: ['~/plugins/router'],

  i18n: {
    // ... your other options
    skipSettingLocaleOnNavigate: true,
  }
}
```

```js {}[~/plugins/router.js]
export default ({ app }) => {
  app.nuxt.defaultTransition.beforeEnter = () => {
    app.i18n.finalizePendingLocaleChange()
  }

  // Optional: wait for locale before scrolling for a smoother transition
  app.router.options.scrollBehavior = async (to, from, savedPosition) => {
    // Make sure the route has changed
    if (to.name !== from.name) {
      await app.i18n.waitForPendingLocaleChange()
    }
    return savedPosition || { x: 0, y: 0 }
  }
}
```

If you have a specific transition defined in a page component, you would also need to call `finalizePendingLocaleChange` from there.

```js {}[~/pages/foo.vue]
export default {
  transition: {
    beforeEnter() {
      this.$i18n.finalizePendingLocaleChange()
    }
  }
}
```
