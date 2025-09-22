# About Mini Card

# HCMS

In the previous example, we saw how to add a collection of **markdown** and metadata inside a folder in our project. This is a convenient way to manage content for small projects.

However, as the project grows, it’s worth considering a **Headless CMS (HCMS)** to manage content in a more scalable way.

In this case, all the information is already available in a **Content Island project**.

> Tour of the model and the data

# Setup

To access this data, Content Island provides a **client API library**. Let’s install it:

```bash
npm install @content-island/api-client
```

To fetch the data, we need to provide an **access token**. In this case, we’ll use a **secret token**, which you can find in your Content Island project settings.

_.env_

```
CONTENT_ISLAND_SECRET_TOKEN=98bfcafff2e649ed1fccec36abde3379
```

Next, we’ll define this variable as a **server environment variable** in Astro:

_./astro.config.mjs_

```diff
// @ts-check
- import { defineConfig } from 'astro/config';
+ import { defineConfig, envField } from "astro/config";
import tailwindcss from '@tailwindcss/vite';

// https://astro.build/config
export default defineConfig({
  vite: {
    plugins: [tailwindcss()],
  },
+  env: {
+    schema: {
+      CONTENT_ISLAND_SECRET_TOKEN: envField.string({
+        context: "server",
+        access: "secret",
+        optional: false,
+        default: "INFORM_VALID_TOKEN",
+      }),
+    },
+  },
});
```

Now let’s create an instance of the client:

_./src/lib/client.ts_

```ts
import { createClient } from "@content-island/api-client";
import { CONTENT_ISLAND_SECRET_TOKEN } from "astro:env/server";

const client = createClient({
  accessToken: CONTENT_ISLAND_SECRET_TOKEN,
});

export default client;
```

# Usage

The author profile section is used in two places:

- A **mini-bio card**
- A **mini-bio hero**, displayed on the _About_ page

We’ll create a **pod** to fetch the author profile data and use it in both places.

Some wiring is already in place:

- Index pages use `pods/mini-bio`.
- The `mini-bio` pod uses two components: **MiniBioCard** and **MiniBioHero**, depending on the props it receives.

Let’s start by extracting the model from the Content Island project:

**COPY FROM CONTENT ISLAND PROJECT**

_./src/pods/mini-bio/mini-bio.model.ts_

```diff
+ import type { Media } from "@content-island/api-client";
+
+ export interface MiniBio {
+  id: string;
+  language: "en";
+  title: string;
+  name: string;
+  role: string;
+  description: string;
+  image: Media;
+  imageAlt: string;
+ }

export type MiniBioType = 'hero' | 'card';
```

Next, let’s add an API to load the author profile data:

_./src/pods/mini-bio/mini-bio.api.ts_

```ts
import client from "#lib/client.ts";
import type { MiniBio } from "./mini-bio.model";

export const getMiniBio = async () =>
  await client.getContent<MiniBio>({
    contentType: "MiniBio",
  });
```

Now let’s use it in the mini-bio pod:

_./src/pods/mini-bio/mini-bio.pod.ts_

```diff
---
import type { MiniBioType } from './mini-bio.model';
+ import { getMiniBio } from './mini-bio.api';
import MiniBioHero from './components/mini-bio-hero.astro';
import MiniBioCard from './components/mini-bio-card.astro';


interface Props {
  type: MiniBioType;
}

const { type } = Astro.props;

+ const miniBio = await getMiniBio();
---

- {type === 'hero' ? <MiniBioHero /> : <MiniBioCard />}
+ {type === 'hero' ? <MiniBioHero /> : <MiniBioCard content={miniBio} />}

```

We’ll start by passing this data to the **MiniBioCard** component:

_./src/pods/mini-bio/mini-bio.pod.ts_

```diff
{type === 'hero' ? <MiniBioHero /> : <MiniBioCard />}
```

And binding the data inside the _MiniBioCard_ component.

First, define the prop:

_./src/pods/mini-bio/components/mini-bio-card.astro_

```diff
---
import ContactButtons from '#components/contact-buttons.astro';
import ShapeOval from '#assets/shapes/oval.svg';
import ShapeRectangle from '#assets/shapes/rectangle.svg';
+ import type { MiniBio } from '../mini-bio.model';

+ interface Props {
+  content: MiniBio | null;
+ }
+ const { content } = Astro.props;
---
```

Let’s confirm the data is being received correctly:

````diff
_./src/pods/mini-bio/components/mini-bio-card.astro_

```diff
<section
  class="bg-primary-50 dark:bg-primary-100 relative hidden overflow-clip rounded-2xl p-6 lg:block"
  aria-labelledby="about-me-aside-heading"
>
+  <div class="relative z-10 flex flex-col items-center justify-center gap-6">
+    <h2 class="font-geist w-full font-bold" id="about-me-aside-heading">{content.title}</h2>
+  </div>

  <!-- Decorative shapes -->
  <div class="text-primary-500/25 absolute top-[-48%] left-[-30%] h-[243px] w-[420px]" aria-hidden="true">
    <ShapeRectangle />
  </div>
  <div class="text-primary-500/25 absolute top-[91.5%] left-[-8%] h-[243px] w-[420px] overflow-clip" aria-hidden="true">
    <ShapeOval />
  </div>
</section>
````

Check the result in the browser:

```bash
npm run dev
```

Now let’s move on and display the rest of the content:

_./src/pods/mini-bio/components/mini-bio-card.astro_

```diff
    <h2 class="font-geist w-full font-bold" id="about-me-aside-heading">{content.title}</h2>

+    <div class="flex flex-col items-center justify-center gap-2 text-center">
+      <div class="h-24 w-24">
+        <img
+          alt={content.imageAlt}
+          class="block h-full w-full rounded-full object-cover"
+          height="80"
+          src={content.image.url}
+          width="80"
+        />
+      </div>
+
+      <span class="text-tbase-500/90 font-bold">{content.name}</span>
+      <span class="text-primary-500 font-semibold">{content.role}</span>
+      <p class="text-sm">
+        {content.description}
+      </p>
+    </div>
+
+    <ContactButtons />
  </div>

  <!-- Decorative shapes -->
```

That works nicely 🎉 — now let’s do the same for the **MiniBioHero** component.

If you navigate to the _About_ page right now, you’ll see it’s empty. Since we already have the data fetching in place, we just need to pass the data to the **MiniBioHero** component:

_./src/pods/mini-bio/mini-bio.pod.astro_

```diff
- {type === 'hero' ? <MiniBioHero /> : <MiniBioCard content={miniBio} />}
+ {type === 'hero' ? <MiniBioHero content={miniBio} /> : <MiniBioCard content={miniBio} />}
```

And update the _mini-bio-hero.astro_ component:

_./src/pods/mini-bio/components/mini-bio-hero.astro_

```diff
---
import Hero from '#components/hero.astro';
+ import type { MiniBio } from '../mini-bio.model';

+ interface Props {
+  content: MiniBio;
+ }
+ const { content } = Astro.props;
---

- <Hero ariaLabelledby="about-hero-heading" />
+ <Hero ariaLabelledby="about-hero-heading">
+  <div class="flex flex-col justify-center gap-6 md:flex-row md:gap-10">
+    <div class="h-[120px] w-[120px] md:h-[240px] md:w-[240px]">
+      <img src={content.image.url} alt={content.imageAlt} class="h-full w-full rounded-full object-cover" />
+    </div>
+    <div class="flex max-w-[55ch] flex-1 flex-col justify-center gap-4">
+      <div>
+        <h1 class="text-primary-700 text-5xl font-bold" id="about-hero-heading">{content.name}</h1>
+        <p class="text-primary-500 font-semibold">{content.role}</p>
+      </div>
+      <p>
+        {content.description}
+      </p>
+    </div>
+  </div>
</Hero>
```

Finally, check the result:

```bash
npm run dev
```

And that’s it 🎉 — now we have both the **mini-bio card** and the **mini-bio hero** fully working with live data.
