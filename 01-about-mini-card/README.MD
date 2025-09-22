# About Mini Card

# HCMS

As we saw in our previous example, we can add a collection of markdown and metadata inside a folder in our project. This is a great way to manage content for a small project, but as our project grows, we might want to consider using a Headless CMS (HCMS) to manage our content.

In this case we have all the info available in a Content Islan Project.

> Tour on the Model and the data

# Setup

In order to access this data Content Island already provides a client api librar,y let's install it:

```bash
npm install @content-island/api-client
```

To be able to access the data we need to provide an access token, in this case we will use a secret token that you can find in your Content Island project settings.

_.env_

```
CONTENT_ISLAND_SECRET_TOKEN=98bfcafff2e649ed1fccec36abde3379
```

We are going to type this variable as a server environment variable in Astro:

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
+       default: "INFORM_VALID_TOKEN",
+      }),
+    },
+  },
});
```

Let's create an instance of the client:

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

- A minibio card.
- A minibio hero, that is used in the about page.

So let's create a pod that will fetch the author profile data and we will use it in both places.

There's already some plumbing in place:

- Index pages uses pods/mini-bio.

- mini-bio pod uses two components, MiniBioCard and MiniBioHero depending on the props type it receives.

Let's extract the model from the Content Island project:

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

Now let's add an api to load the author profile data:

_./src/pods/mini-bio/mini-bio.api.ts_

```ts
import client from "#lib/client.ts";
import type { MiniBio } from "./mini-bio.model";

export const getMiniBio = async () =>
  await client.getContent<MiniBio>({
    contentType: "MiniBio",
  });
```

Let's use it in the mini-bio pod:

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

{type === 'hero' ? <MiniBioHero /> : <MiniBioCard />}
```

We will start by passig this info to the MiniBioCard component:

_./src/pods/mini-bio/mini-bio.pod.ts_

```diff
{type === 'hero' ? <MiniBioHero /> : <MiniBioCard />}
```

And bindg the info to the _MiniBioCard_ component:

First the prop:

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

And then the bindings:

_./src/pods/mini-bio/components/mini-bio-card.astro_

```diff
<section
  class="bg-primary-50 dark:bg-primary-100 relative hidden overflow-clip rounded-2xl p-6 lg:block"
  aria-labelledby="about-me-aside-heading"
>
  <!-- Decorative shapes -->
  <div class="text-primary-500/25 absolute top-[-48%] left-[-30%] h-[243px] w-[420px]" aria-hidden="true">
    <ShapeRectangle />
  </div>
  <div class="text-primary-500/25 absolute top-[91.5%] left-[-8%] h-[243px] w-[420px] overflow-clip" aria-hidden="true">
    <ShapeOval />
  </div>
</section>
```


Let's check the result in the browser:

```bash
npm run dev
```
