# 08 Experience Lab

We have learned a lot... but we have not yet applied our knowledge in a real-world scenario.

Let's get our hands dirty in this hands-on lab!

## Lab Overview

If you click on the about page, you will an empty page, we want to reflect the experience list of the user in this page.

We will use the knowledge we have acquired in the previous modules to implement this feature.

We will explain you the issue, then add tips and hints to help you solve it.

## Goal

Let's run the project.

```bash
npm run dev
```

If you click on the about page, you will see an empty page.

![Empty experience placeholder](./content/empty-experience.jpg)

Our goal is to show something like:

![Full list of experiences](./content/experience-completed.jpg)

## Loading data

We have already our content island project connected to a Contentful project.

We can see that in the model we've got a list of experiences and an experiencie model.

![Experience and the experiencie model](./content/model.jpg)

We are go to start by loading the data from Contentful.

We've got an _experience-collection_ pod.

Let's define a model, in Content Island we have got:

- And Experience Section model that will load all the experiences plus the heading to be used for the section.

- An Experience model that will represent a single experience.

If you are in Content Island you can generate a model that will include the list of nested collection, so the model would be something like.

_./src/pods/experience-collection/experience-collection.model.ts_

```ts
export interface Experience {
  id: string;
  language: "en";
  company: string;
  role: string;
  period: string;
  description: string;
}

export interface ExperienceSection {
  id: string;
  language: "en";
  title: string;
  experienceCollection: Experience[];
}
```

Now let's load the data from Contentful, this time we will indicate that we wat to load the nested collection.

_./src/pods/experience-collection/experience-collection.api.ts_

```ts
import client from "#lib/client.ts";
import type { ExperienceSection } from "./experience-collection.model";

export const getExperience = async () =>
  await client.getContent<ExperienceSection>({
    contentType: "ExperienceSection",
    includeRelatedContent: true,
  });
```

And let's use it in our component.

_./src/pods/experience-collection/experience-collection.pod.astro_

```diff
---
- const experienceContent = {
-  title: 'Experience',
- };
+ import { getExperience } from "./experience-collection.api";
+ const experienceContent = await getExperience();
---

<section class="flex flex-1 flex-col gap-10 px-6" aria-labelledby="experience-section-heading">
  <h2 class="text-tbase-500/90 text-3xl font-bold" id="experience-section-heading">{experienceContent.title}</h2>
+  <ul>
+    {experienceContent.experienceCollection.map((experience) => (
+      <li>
+        <p>{experience.role} @ {experience.company}</h3>
+        <p>{experience.period}</p>
+        <p>{experience.description}</p>
+      </li>
+    ))}
+  </ul>
</section>
```

Now if you go to the about page, you should see a list of experiences (With node desing).

## Adding styles

Now let's add some styles to our component.

```diff
<section class="flex flex-1 flex-col gap-10 px-6" aria-labelledby="experience-section-heading">
  <h2 class="text-tbase-500/90 text-3xl font-bold" id="experience-section-heading">{experienceContent.title}</h2>

-  <ul>
-    {experienceContent.experienceCollection.map((experience) => (
-      <li>
-        <p>{experience.role} @ {experience.company}</h3>
-        <p>{experience.period}</p>
-        <p>{experience.description}</p>
-      </li>
-      ))}
-  </ul>
+  <div class="pl-4">
+    {
+      experienceContent.experienceCollection.map(exp => (
+        <div class="relative flex items-baseline gap-4 before:absolute before:top-5 before:left-[5px] before:z-[-1] before:h-full before:w-0.5 before:bg-gray-300 last:before:hidden">
+          <div class="flex flex-col">
+            <div class="bg-primary-700 dark:border-primary-50 dark:shadow-primary-50 h-3 w-3 rounded-full shadow-[0_0_0_4px] shadow-white dark:shadow-[0_0_0_5px]" />
+          </div>
+
+          <article class="pb-10">
+            <h3 class="text-primary-700 text-lg font-bold">
+              {exp.company} – {exp.role}
+            </h3>
+            <span class="text-tbase-500/90 mb-2 block text-sm font-semibold">{exp.period}</span>
+            <p>{exp.description}</p>
+          </article>
+        </div>
+      ))
+    }
+  </div>
</section>
```
