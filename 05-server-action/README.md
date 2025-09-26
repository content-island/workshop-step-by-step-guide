# Server Actions

In this section, we’ll store the number of likes on the server while still interacting from the client. In other words, we’ll have a button that, when clicked, increments the number of likes and reflects the new value on screen.

For simplicity, we’ll keep this value in memory (ideally, you’d save it in a database and maintain a mapping of likes by lesson/post ID).

First, let’s set up server actions by updating the Astro adapter (we’ll modify `astro.config.mjs`). Here you can choose Node.js, Vercel, Netlify, or Deno. Note that once you enable an adapter like these, Astro will no longer generate a 100% static site—you’ll need to deploy to a platform that supports the chosen runtime.

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

Now let’s define our server actions. Astro favors convention over configuration, so actions must live inside the `src/actions` folder.

Start with a model:

_./src/actions/model.ts_

```ts
export type LikesResponse = {
  likes: number;
};
```

Then, add an in‑memory repository (in a real app you’d connect this to a database or external API).

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

Now define the action itself:

_src/actions/index.ts_

```ts
import { defineAction } from 'astro:actions';
import { addLike, getLikes } from './repository';
import type { LikesResponse } from './model';

export const server = {
  addLike: defineAction<LikesResponse>({
    async handler(slug) {
      return { likes: await addLike(slug) };
    }
  }),
  getLikes: defineAction<LikesResponse>({
    async handler(slug) {
      return { likes: await getLikes(slug) };
    }
  })
};
```

Next, update the component to use the action:

> **Important:** Don’t forget to build the project so the actions become available.

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
+    actions.getLikes(slug).then(response => {
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

Finally, pass the `slug` prop when using the component:

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
