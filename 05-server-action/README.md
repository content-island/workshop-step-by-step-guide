# Server Actions

En este caso vamos a almacenar el número de Likes en el servidor, pero seguimos interactuando con el cliente, es decir, vamos a tener un botón que al pulsarlo incrementa el número de likes y lo muestra en pantalla.

Por simplicidad vamos a almacenar ese valor en una variable en memoria (lo suyo sería guardarlo en base de datos y además guardar un mapeo de like por cada id de lección).

Lo primero que hacemos es definir el setup de la acción de servidor (tocaremos el archivo astro.config), aquí podemos elegir si tirar de Node.js, Vercel, Netifly o Deno, ... es decir en cuanto elegimos está opción Astro va a dejar de generar un sitio 100% estático, y nos hará falta poder desplegar en un servidor que soporte nodejs o lo que hayamos elegido.

```bash
npm install @astrojs/node
```


```diff
import { defineConfig, envField } from "astro/config";
import react from "@astrojs/react";
+ import node from '@astrojs/node';

export default defineConfig({
  integrations: [react()],
+  adapter: node({ mode: 'standalone' }), // Configuración para Node.js
  env: {
    schema: {
      CONTENT_ISLAND_SECRET_TOKEN: envField.string({
        context: "server",
        access: "secret",
        optional: false,
        default: "INFORM_VALID_TOKEN",
      }),
    },
  },
});
```