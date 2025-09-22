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

```diff
---
 <Layout title={entry.title}>
+  <Hero url={entry.image.url} slot="hero" />
  <h2>{entry.title}</h2>
 </Layout>
```

On the aside we have a different one, so let's define it here:

```diff
---
import Layout from '#layouts/layout.astro';
import Hero from '#components/hero.astro';
+ import MiniBioPod from '#pods/mini-bio/mini-bio.pod.astro';
+ import NewsletterPod from '#pods/newsletter/newsletter.pod.astro';
+ import PopularPostsPod from '#pods/popular-posts/popular-posts.astro';
import { getAllPosts } from '#pods/post-collection/post-collection.api';
```

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
