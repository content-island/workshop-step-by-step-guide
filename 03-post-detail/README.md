# 03 Post detail

Let's display the post detail page.

Right now if we try to navigate to a given post detail page, we get a 404 error.

Let' start by create the posts detail page.

We could try to create something like:

_./src/pages/posts/index.astro_

```astro
<h1>Hey I'm the post detail</h1>
```

But this would not work, we have a single route per posts, if we go to Content Island we can see there's a field called `slug` that we can use to create a dynamic route, and on every link we use that post.

What can we do? Astro offers a way to create dynamic routes using square brackets:

- We can add one or more segments in the file name using square brackets to indicate a dynamic segment.
- The name inside the brackets will be the name of the property we can use to access the value of that segment.
- And using _getStaticPaths_ we can tell Astro which pages to generate at build time.

If you are a Marvel fan, remember infinity wars, where Dr. Strange saw millions of futures, and only one where they win? That's what we are going to do here, we are going to tell Astro which pages to generate at build time.

Let's update the name of the from _index.astro_ to

_[slug].astro_

And now let's calculate all the paths we need to generate using _getStaticPaths_.

In order to do that we have to:

- Read the list of posts from the Content Island.
- For each post, return an object with the params property containing the slug of the post.
- Then we can use the props to render the post detail.

We have avialable this api and model in the post-collection pod, here we could decide:

1. Reuse that api and model on the page.
2. Create a copy of that api and model on the page.
3. Promoteo to common code that can be reused.

For the sake of simplicity, let's reuse the api and model on the page.

_./src/pages/posts/[slug].astro_

```astro
---
import { getAllPosts } from '#pods/post-collection.api';
import type { Post } from '#pods/post-collection.model';

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

Let' add a _hero_ component to display the post title.

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

On the aside we have a different one, so let's define it here:

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

Let's give a try

```bash
npm run dev
```

And now let's deep dive into the post content, we will create a separate componente for this.

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

Kind of works but looks like hell, if we take a look to the original post, we need a header and the body, let's create two components.

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

<!-- Article Header -->
<div class="flex items-start justify-between">
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
</div>
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

Looking better... let's create the body component.

Let's start by creating the component and define server side code.

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

And the markup

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
        class="group w-fit cursor-pointer rounded-full bg-white p-1 transition-colors duration-300"
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

Let's use it on the post pod.

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

Getting better, buuuut, the content is not being rendered as HTML, for that we have created a wrapper that uses _Markded_ and _highlight.js_ to render the content properly.

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

Aaand... there we go:

```bash
npm run dev
```