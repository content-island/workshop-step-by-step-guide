# Post list

If we navigate to the homepage, we can see an empty place holder where the list of posts will be displayed.

Where can we get the list of posts? From Content Island

**Quick Check Content Island list of posts**

Let's get the list of posts from Content Island

We are going to create a new Pod and we will call it post-collection.

Let's add the post model.

**Copy from Content Island**

_./src/pods/post-collection/post-collection.model.ts_

```ts
import type { Media } from '@content-island/api-client';

export interface Post {
  id: string;
  language: 'en';
  title: string;
  slug: string;
  date: string; // Stores the date in ISO 8601 format. For example: 2021-09-10T19:30:00.000Z
  summary: string;
  image: Media;
  content: string;
  readTime: number;
}
```

And the api to load the posts.

_./src/pods/post-collection/post-collection.api.ts_

```ts
import client from '#lib/client.ts';
import type { Post } from './post-collection.model';

export const getAllPosts = async () =>
  await client.getContentList<Post>({
    contentType: 'Post',
    sort: { 'fields.date': 'desc' },
    pagination: { take: 6 }
  });
```

This time we will load the posts directly the post pod, but we could make the call in the index page and pass the posts as props to the post list component.

Let's first check that data is being loaded correctly.

_./src/pods/post-collection/post-collection.pod.astro_

```astro
---
import { getAllPosts } from './post-collection.api.ts';

const posts = await getAllPosts();
---
<section class="flex flex-1 flex-col gap-9">
{posts.map((post) => (
      <h2>{post.title}</h2>
))}
</section>
```

Let's make use of this post:

_./src/pages/index.astro_

```diff
---
import Layout from '#layouts/layout.astro';
import { getCollection } from 'astro:content';
import Hero from '#components/hero.astro';
import MiniBioPod from '#pods/mini-bio/mini-bio.pod.astro';
import NewsletterPod from '#pods/newsletter/newsletter.pod.astro';
import PopularPosts from '#pods/popular-posts/popular-posts.astro';
+ import PostCollectionPod from '#pods/post-collection/post-collection.pod.astro';

const homeContent = {
  hero: {
    title: "John's Web Dev Blog",
    description: 'Here you can find various articles on web application development.',
  },
};
---
```

```diff
  </Hero>

-  <div>Place holder for post preview collection</div>
+  <PostCollectionPod />

  <Fragment slot="aside">
```

If we ran the project, we would see the list of posts (just the titles for now).

```bash
npm run dev
```

Let's give it some style, we will create a component that will show a post card.

First we will define the code in the fences:

_./src/components/post-card.astro_

```astro
---
import HeartIcon from '#assets/icons/heart.svg';
import type { Post } from '../post-collection.model';

interface Props {
  post: Post;
}
const { post } = Astro.props;
const readTimeLabel = 'min read';
---
```

And Let's go for the markup:

```astro
<a
  href={`/posts/${post.slug}`}
  class="group @container cursor-pointer rounded-4xl transition-shadow duration-300 hover:shadow-lg"
>
  <article class="flex h-full flex-col @lg:flex-row">
    <div class="aspect-[16/9] overflow-hidden rounded-4xl bg-gray-300 @lg:flex-1">
      <img
        src={post.image.url}
        alt={post.title}
        class="h-full w-full rounded-4xl object-cover transition-transform duration-300 group-hover:scale-[1.06]"
        aria-hidden="true"
      />
    </div>
    <div class="flex flex-1 flex-col justify-between gap-6 p-4 @lg:flex-2">
      <div>
        <time datetime={post.date} class="mb-1 block text-xs">
          {
            new Date(post.date).toLocaleDateString('en-US', {
              year: 'numeric',
              month: 'long',
              day: 'numeric',
            })
          }
        </time>
        <h3
          class="group-hover:text-primary-700 text-tbase-500/90 mb-2 text-xl font-bold transition-colors duration-300"
        >
          {post.title}
        </h3>
        <p class="text-sm">{post.summary}</p>
      </div>

      <div class="flex items-center gap-4">
        <span class="text-xs">{post.readTime} {readTimeLabel}</span>
        <div class="flex items-center gap-1">
          <HeartIcon class="h-5 w-5" />
          <span class="text-xs">{6}</span>
        </div>
      </div>
    </div>
  </article>
</a>
```

And let's use it in our post collection pod:

_./src/pods/post-collection/post-collection.pod.astro_

```diff
---
import { getAllPosts } from './post-collection.api.ts';
+ import PostCard from './components/post-card.astro';

const posts = await getAllPosts();
---

<section class="flex flex-1 flex-col gap-9">
-  {posts.map(post => <h2>{post.title}</h2>)}
+  {posts.map(post => <PostCard post={post} />)}
</section>
```

Not bad, let's give some extra styling...

_./src/pods/post-collection/post-collection.pod.astro_

```diff
<section class="flex flex-1 flex-col gap-9">
+  <div class="grid max-w-[895px] gap-8 xl:grid-cols-2">

  {posts.map(post => <PostCard post={post} />)}
+  </div>

</section>
```

And there we go :), now if we click on a post, we will get a 404, but we will fix that in the next step, let's display a single post.
