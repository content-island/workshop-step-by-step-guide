# 07 View Transitions

In this step, we will implement view transitions to enhance the user experience when navigating between different views in our application. View transitions provide smooth animations that make the interface feel more dynamic and responsive.

## Step 1: Add View Transition Support

We need to add support for view transitions in our Astro project. This is done by adding the `ClientRouter` component from the `astro:transitions` package to the head section that is shared across all pages in our application. In our case, this is located in our main layout file.

_./src/layouts/layout.astro_

```diff
---
+import { ClientRouter } from "astro:transitions";
---
<head>
... other head elements ...
+<ClientRouter />
</head>
```

With this alone, you can already see that when navigating between pages there is a transition, but we are going to improve it a little.

## Step 2: Implement View Transitions

Now that we have the necessary support for view transitions, we can implement them in our application. We will add transitions to the post detail view.

We use the `transition:name` directive to specify the transition name for elements that should be animated during navigation. In this case, we will use the post title as the transition name.

_./src/pods/post/components/content.astro_

```diff
<h1
class="text-tbase-500/90 text-5xl leading-[1.1] font-bold"
id="article-section-heading"
+transition:name={`${entry.data.title}-title`}
  >
```

_./src/pods/post-collection/components/post-card.astro_

```diff
<h3
class="group-hover:text-primary-700 text-tbase-500/90 text-xl font-bold transition-colors duration-300"
+ transition:name={`${post.data.title}-title`}
>
```

## Step 3: Fix Dark Mode Issue

When using view transitions, we might encounter an issue with dark mode where the transition does not respect the current theme. To fix this, we need to use Astro events.
Remove the previous theme toggle script if you have it, and replace it with the following code in your header component:

_src/components/header.astro_

```html
<script>
  const handleThemeToggle = () => {
    const html = document.documentElement;
    html.classList.toggle('dark');
    localStorage.setItem(
      'theme',
      html.classList.contains('dark') ? 'dark' : 'light'
    );
  };

  const initThemeToggle = () => {
    const button = document.getElementById('theme-toggle');
    const html = document.documentElement;
    const storedTheme = localStorage.getItem('theme');

    if (storedTheme) {
      html.classList.toggle('dark', storedTheme === 'dark');
    }
    if (
      !storedTheme &&
      window.matchMedia('(prefers-color-scheme: dark)').matches
    ) {
      html.classList.add('dark');
    }

    // Clean up previous event listener if it exists
    button?.removeEventListener('click', handleThemeToggle);

    // Add new event listener
    button?.addEventListener('click', handleThemeToggle);
  };

  // Initialize on initial load
  initThemeToggle();

  // Reinitialize after each view transition
  document.addEventListener('astro:after-swap', initThemeToggle);
</script>
```

We can do more things with view transitions, but for now, this is enough to get you started. You can explore more about view transitions in the [Astro documentation](https://docs.astro.build/en/guides/view-transitions/).
