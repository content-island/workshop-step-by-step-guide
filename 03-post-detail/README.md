# 03 Post Detail

Let's display the post detail page.

Right now, if we try to navigate to a specific post detail page, we get a 404 error.

Let's start by creating the post detail page.

We could try something like:

_./src/pages/posts/index.astro_

```astro
<h1>Hey I'm the post detail</h1>
```

But this won’t work. We need one route per post. In Content Island we have a field called `slug` that we can use to create a dynamic route, and then link each post with it.

So, what can we do? Astro provides a way to create dynamic routes using square brackets:

- You can add one or more segments to the file name using square brackets to indicate a dynamic segment.

- The name inside the brackets becomes the property you can use to access that segment’s value.
- With _getStaticPaths_, you can tell Astro which pages to generate at build time.

If you’re a Marvel fan, think of _Infinity War_ when Dr. Strange looked into millions of futures and saw only one where they won. That’s pretty much what we’re doing here: telling Astro exactly which pages to generate at build time.

So let’s rename the file from _index.astro_ to:

_./src/pages/posts/[slug].astro_

And now let’s calculate all the paths we need to generate using _getStaticPaths_.

To do this, we need to:

- Fetch the list of posts from Content Island.
- For each post, return an object with a `params` property containing the post’s slug.
- Then use the props to render the post detail.

We already have an API and model available in the post-collection pod. At this point, we could:

1. Reuse that API and model on the page.
2. Copy the API and model into the page.
3. Promote it to shared code that can be reused.

For simplicity, let’s just reuse the API and model directly on the page.

_./src/pages/posts/[slug].astro_

```astro
---
import { getAllPosts } from '#pods/post-collection/post-collection.api';
import type { Post } from '#pods/post-collection/post-collection.model';

// Generate all possible paths
export async function getStaticPaths() {
  const postEntries = await getAllPosts();
  return postEntries.map(entry => ({
    params: { slug: entry.slug },
    props: { entry },
  }));
}

// Per path let's generate the page
const { entry } = Astro.props;
---
```

Let's check that the post detail is being shown.

_./src/pages/posts/[slug].astro_

```diff
---
+ <Layout title={entry.title}>
+  <h2>{entry.title}</h2>
+ </Layout>
```

Let's run this and check that everything is working as expected.

```bash
npm run dev
```

Time to add some design :).

Let’s add a _hero_ component to display the post title.

_./src/pages/posts/[slug].astro_

```diff
---
import Layout from '#layouts/layout.astro';
+ import Hero from '#components/hero.astro';
import { getAllPosts } from '#pods/post-collection/post-collection.api';
```

_./src/pages/posts/[slug].astro_

```diff
---
 <Layout title={entry.title}>
+  <Hero url={entry.image.url} slot="hero" />
  <h2>{entry.title}</h2>
 </Layout>
```

On the aside we have different elements, so let’s define them here:

_./src/pages/posts/[slug].astro_

```diff
---
import Layout from '#layouts/layout.astro';
import Hero from '#components/hero.astro';
+ import MiniBioPod from '#pods/mini-bio/mini-bio.pod.astro';
+ import NewsletterPod from '#pods/newsletter/newsletter.pod.astro';
+ import PopularPostsPod from '#pods/popular-posts/popular-posts.astro';
import { getAllPosts } from '#pods/post-collection/post-collection.api';
```

_./src/pages/posts/[slug].astro_

```diff
---
 <Layout title={entry.title}>
  <Hero url={entry.image.url} slot="hero" />
  <h2>{entry.title}</h2>
+  <Fragment slot="aside">
+    <MiniBioPod type="card" />
+    <NewsletterPod type="mini" />
+    <PopularPostsPod />
+  </Fragment>

 </Layout>
```

Let's give it a try:

```bash
npm run dev
```

Now let’s dive into the post content. We’ll create a separate component for this.

_src/pods/post/post.pod.astro_

```astro
---
import type { Post } from '#pods/post-collection/post-collection.model';

interface Props {
  postEntry: Post;
}

const { postEntry } = Astro.props;

const likeCount = 6;

const postContent = {
  backButton: 'Go back',
  published: 'Published',
  minRead: 'min read',
};
---

<section class="flex shrink-1 grow flex-col gap-12 px-6 py-4">
  <div>{postEntry.title}</div>
  <div>{postEntry.content}</div>
</section>
```

Let's use it on the post detail page.

_./src/pages/posts/[slug].astro_

```diff
---
import Layout from '#layouts/layout.astro';
import Hero from '#components/hero.astro';
import MiniBioPod from '#pods/mini-bio/mini-bio.pod.astro';
import NewsletterPod from '#pods/newsletter/newsletter.pod.astro';
import PopularPostsPod from '#pods/popular-posts/popular-posts.astro';
import { getAllPosts } from '#pods/post-collection/post-collection.api';
+ import PostPod from '#pods/post/post.pod.astro';

// Generate all possible paths
export async function getStaticPaths() {
  const postEntries = await getAllPosts();
  return postEntries.map(entry => ({
    params: { slug: entry.slug },
    props: { entry },
  }));
}

// Per path let's generate the page
const { entry } = Astro.props;
---

<Layout title={entry.title}>
  <Hero url={entry.image.url} slot="hero" />
-  <h2>{entry.title}</h2>
+  <PostPod postEntry={entry} />
  <Fragment slot="aside">
    <MiniBioPod type="card" />
    <NewsletterPod type="mini" />
    <PopularPostsPod />
  </Fragment>
</Layout>
```

It works, but it doesn’t look great yet. If we compare it with the original post, we see that we need both a header and a body. Let’s create two new components.

**src/pods/post/components/header.astro**

```astro
---
import ArrowLeftIcon from '#assets/icons/arrow-left.svg';

interface Props {
  gobackText: string;
  publishedText: string;
  date: string;
}
const { gobackText, publishedText, date } = Astro.props;
---

<header class="flex items-start justify-between">
  <a href="/" class="hover:text-primary-600 flex items-center gap-2 font-semibold transition-colors">
    <ArrowLeftIcon />
    {gobackText}
  </a>

  <p class="text-sm">
    {publishedText}{' '}
    <time datetime={date}>
      {
        new Date(date).toLocaleDateString('en-US', {
          year: 'numeric',
          month: 'long',
          day: 'numeric',
        })
      }
    </time>
  </p>
</header>
```

_src/pods/post/post.pod.astro_

```diff
---
import type { Post } from '#pods/post-collection/post-collection.model';
+ import Header from '#pods/post/components/header.astro';

// (...)
---

<section class="flex shrink-1 grow flex-col gap-12 px-6 py-4">
+ <Header gobackText={postContent.backButton} publishedText={postContent.published} date={postEntry.date} />
  <div>{postEntry.title}</div>
  <div>{postEntry.content}</div>
</section>
```

Looking better… now let’s create the body component.

First, define the component and its server-side code:

_src/pods/post/components/body.astro_

```astro
---
import HeartIcon from '#assets/icons/heart.svg';
import type { Post } from '#pods/post-collection/post-collection.model';

interface Props {
  entry: Post;
  likeCount: number;
  minReadText: string;
}

const { entry, likeCount, minReadText } = Astro.props;
---
```

And the markup:

_src/pods/post/components/body.astro_

```astro
<div class="flex flex-col gap-6">
  <h1 class="text-tbase-500/90 text-5xl leading-[1.1] font-bold" id="article-section-heading">
    {entry.title}
  </h1>

  <div class="border-tbase-500/40 mb-2 flex items-center justify-between gap-4 border-y py-2">
    <p class="text-xs">{entry.readTime} {minReadText}</p>
    <div class="flex items-center">
      <button
        type="button"
        class="group w-fit cursor-pointer rounded-full p-1 transition-colors duration-300"
        aria-label="Like"
      >
        <HeartIcon class="h-5.5 w-5.5 transition-colors duration-300 group-hover:text-red-500" />
      </button>
      <span class="text-xs">{likeCount}</span>
    </div>
  </div>
  <div>{entry.content} </div>
</div>
```

Let’s use it inside the post pod.

_src/pods/post/post.pod.astro_

```diff
---
import type { Post } from '#pods/post-collection/post-collection.model';
import Header from '#pods/post/components/header.astro';
+ import Body from '#pods/post/components/body.astro';

// (...)
---

<section class="flex shrink-1 grow flex-col gap-12 px-6 py-4">
  <Header gobackText={postContent.backButton} publishedText={postContent.published} date={postEntry.date} />
+  <Body entry={postEntry} likeCount={likeCount} minReadText={postContent.minRead} />
-  <div>{postEntry.title}</div>
-  <div>{postEntry.content}</div>
</section>
```

Much better, but the content is not yet being rendered as HTML. To fix this, we can use a wrapper with _Marked_ and _highlight.js_ to render the content properly.

_src/pods/post/components/body.astro_

```diff
---
import HeartIcon from '#assets/icons/heart.svg';
import type { Post } from '#pods/post-collection/post-collection.model';
+ import MarkdownRenderer from '#components/markdown-renderer.astro';
---
```

```diff
      <span class="text-xs">{likeCount}</span>
    </div>
  </div>
-  <div>{entry.content}</div>
+  <MarkdownRenderer content={entry.content} />
</div>
```

And… there we go:

```bash
npm run dev
```
