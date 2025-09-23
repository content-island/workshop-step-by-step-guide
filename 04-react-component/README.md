# React Integration

We said that Astro can easily integrate with your favorite frameworks. Let's see how to do that with React.

Just for the sake of practicing we are going to implement a _cool_ like button using React (this could be implemented using plain vanilla javascript but we want to practice React here).

## Step 1: Install React

Let's install React integration for Astro:

```bash
npm install @astrojs/react
```

Let's update the `astro.config.mjs` file to add the React integration:

```diff
// @ts-check
import { defineConfig, envField } from 'astro/config';
import tailwindcss from '@tailwindcss/vite';
+ import react from '@astrojs/react';

// https://astro.build/config
export default defineConfig({
+  integrations: [react()],
  vite: {
    plugins: [tailwindcss()],
  },
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

## Step 2: Create the React component

Now let's create the like button React component.

For the sake of simplicity, we will just store the likes in memory in a variable. In a real world scenario you would probably want to store this in a database.

_./src/pods/post/components/like-button.component.tsx_

```tsx
import { useState, useEffect } from "react";

export const LikeButton: React.FC = () => {
  const [likes, setLikes] = useState<number>(0);

  useEffect(() => {
    const storedLikes = localStorage.getItem("likes");
    if (storedLikes) {
      setLikes(parseInt(storedLikes, 10));
    }
  }, []);

  const handleLike = () => {
    const newLikes = likes + 1;
    setLikes(newLikes);
    localStorage.setItem("likes", newLikes.toString());
  };

  return (
    <div className="flex items-center">
      <button
        type="button"
        className="group w-fit cursor-pointer rounded-full bg-white p-1 transition-colors duration-300"
        aria-label="Like"
        title="Like this post"
        onClick={handleLike}
      >
        <svg
          aria-hidden="true"
          className="flex h-5.5 w-5.5 items-center justify-center transition-colors duration-300 group-hover:text-red-500"
          xmlns="http://www.w3.org/2000/svg"
          width="1em"
          height="1em"
          viewBox="0 0 24 24"
        >
          <path
            fill="currentColor"
            d="M12 19.75a.75.75 0 0 1-.53-.22L4.7 12.74a5 5 0 0 1 0-7a4.95 4.95 0 0 1 7 0L12 6l.28-.28a4.92 4.92 0 0 1 3.51-1.46a4.92 4.92 0 0 1 3.51 1.45a5 5 0 0 1 0 7l-6.77 6.79a.75.75 0 0 1-.53.25m-3.79-14a3.44 3.44 0 0 0-2.45 1a3.48 3.48 0 0 0 0 4.91L12 17.94l6.23-6.26a3.47 3.47 0 0 0 0-4.91a3.4 3.4 0 0 0-2.44-1a3.44 3.44 0 0 0-2.45 1l-.81.81a.77.77 0 0 1-1.06 0l-.81-.81a3.44 3.44 0 0 0-2.45-1.02"
          />
        </svg>
      </button>
      <span className="text-xs">{likes}</span>
    </div>
  );
};

export default LikeButton;
```

## Step 3: Use the React component in an Astro page

_src/pods/post/components/body.astro_

```diff

```

```diff
---
- import HeartIcon from '#assets/icons/heart.svg';
+ import LikeButton from './like-button.component.tsx';
import type { Post } from '#pods/post-collection/post-collection.model';
import MarkdownRenderer from '#components/markdown-renderer.astro';

interface Props {
  entry: Post;
  likeCount: number;
  minReadText: string;
}

const { entry, likeCount, minReadText } = Astro.props;
---
```

```diff
  <div class="border-tbase-500/40 mb-2 flex items-center justify-between gap-4 border-y py-2">
    <p class="text-xs">{entry.readTime} {minReadText}</p>
-    <div class="flex items-center">
-      <button
-        type="button"
-        class="group w-fit cursor-pointer rounded-full bg-white p-1 transition-colors duration-300"
-        aria-label="Like"
-      >
-        <HeartIcon class="h-5.5 w-5.5 transition-colors duration-300 group-hover:text-red-500" />
-      </button>
-      <span class="text-xs">{likeCount}</span>
-    </div>
+    <LikeButton client:load />
  </div>
  <MarkdownRenderer content={entry.content} />
```

And let's give a try...

```bash
npm run dev
```

Even we can debug it...
