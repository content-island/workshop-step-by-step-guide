# Server Actions

In this case, we’re going to store the number of Likes on the server, but we’ll still be interacting with the client. That is, we’ll have a button that, when clicked, increments the number of likes and displays it on the screen.

For simplicity, we’re going to store this value in a variable in memory (ideally, it should be saved in a database, along with a mapping of likes for each lesson ID).

The first thing we do is define the server action setup (we’ll modify the astro.config file). Here we can choose whether to use Node.js, Vercel, Netlify, or Deno... In other words, once we make this choice, Astro will no longer generate a 100% static site, and we’ll need to deploy to a server that supports Node.js or whichever option we selected.

```bash
npm install @astrojs/node
```

_./astro.config.mjs_

```diff
// @ts-check
import { defineConfig, envField } from 'astro/config';
import tailwindcss from '@tailwindcss/vite';
import react from '@astrojs/react';
+ import node from '@astrojs/node';

// https://astro.build/config
export default defineConfig({
  vite: {
    plugins: [tailwindcss()],
  },
+  adapter: node({
+    mode: 'standalone',
+  }),
  integrations: [react()],
  env: {
    schema: {
      CONTENT_ISLAND_SECRET_TOKEN: envField.string({
        context: 'server',
        access: 'secret',
        optional: false,
        default: 'INFORM_VALID_TOKEN',
      }),
    },
  },
});
```

Let's define the server actions. Astro uses convention over configuration, so they must be placed inside the actions folder.

Let's first define a model:

_./src/actions/model.ts_

```ts
export type LikesResponse = {
  likes: number;
};
```

Then let's define an inmemory repository (in real life we would connect this to a database / api).

_src/actions/repository.ts_

```ts
// This is just an in-memory store for demonstration purposes.
// Ideally we could connect to a database or an external API.
const likeStore: Map<string, number> = new Map();

export const getLikes = async (slug: string): Promise<number> => {
  return likeStore.get(slug) ?? 0;
};

export const addLike = async (slug: string): Promise<number> => {
  const current = likeStore.get(slug) ?? 0;
  const updated = current + 1;
  likeStore.set(slug, updated);
  return updated;
};
```

And now let's define the action itself:

_src/actions/index.ts_

```ts
import { defineAction } from "astro:actions";
import { addLike, getLikes } from "./repository";
import type { LikesResponse } from "./model";

export const server = {
  addLike: defineAction<LikesResponse>({
    async handler(slug) {
      return { likes: await addLike(slug) };
    },
  }),
  getLikes: defineAction<LikesResponse>({
    async handler(slug) {
      return { likes: await getLikes(slug) };
    },
  }),
};
```

Now let's update the component to use this action:

> IMPORTANT: do not forget to build the project, to get the actions available.

_./src/pods/post/components/like-button.component.tsx_

```diff
+ import { actions } from 'astro:actions';
import { useState, useEffect } from 'react';

+ interface Props {
+  slug: string;
+ }

- export const LikeButton: React.FC = () => {
+ export const LikeButton: React.FC<Props> = ({ slug }) => {
  const [likes, setLikes] = useState<number>(0);

  useEffect(() => {
-    const storedLikes = localStorage.getItem('likes');
-    if (storedLikes) {
-      setLikes(parseInt(storedLikes, 10));
-    }
+    actions.getLikes().then(response => {
+      setLikes(response?.data?.likes || 0);
+    });
  }, []);

  const handleLike = () => {
    const newLikes = likes + 1;
    setLikes(newLikes);
-    localStorage.setItem('likes', newLikes.toString());
+    actions.addLike(slug);
  };
```

And let's inform the _slug_ property when we use the component:

_./src/pods/post/components/body.astro_

```diff
<div class="flex flex-col gap-6">
  <h1 class="text-tbase-500/90 text-5xl leading-[1.1] font-bold" id="article-section-heading">
    {entry.title}
  </h1>

  <div class="border-tbase-500/40 mb-2 flex items-center justify-between gap-4 border-y py-2">
    <p class="text-xs">{entry.readTime} {minReadText}</p>
-    <LikeButton client:load />
+    <LikeButton client:load slug={entry.slug} />
  </div>
  <MarkdownRenderer content={entry.content} />
</div>
```
