# Server Action Form with Resend

Alright, now that we’ve already played a bit with **Server Actions**, let’s take it one step further.  
This time, we’re going to connect a form to send emails using [Resend](https://resend.com/).
The cool part? We don’t need to install or configure anything new for Server Actions—we’ve already got that set up.

The only package we’ll add is **Resend**:

```bash
npm install resend
```

Next, we need to set up an account on [Resend](https://resend.com/) to get an API key. Don't worry about domain configuration; for this example, we'll use onboarding@resend.dev to send emails. Once you have the key, add it to your environment variables in the `.env` file.

_.env_

```
FROM_EMAIL=your_verified_email_here
TO_EMAIL=recipient_email_here
RESEND_API_KEY=your_resend_api_key_here
```

And also add it to your `astro.config.mjs` file:

```diff
export default defineConfig({
  // other configurations...
  env: {
    schema: {
+      RESEND_API_KEY: envField.string({
+        context: 'server',
+        access: 'secret',
+        optional: false,
+        default: 'INFORM_VALID_TOKEN',
+      }),
+    FROM_EMAIL: envField.string({
+        context: 'server',
+        access: 'secret',
+        optional: false,
+        default: 'INFORM_VALID_EMAIL',
+      }),
+    TO_EMAIL: envField.string({
+        context: 'server',
+        access: 'secret',
+        optional: false,
+        default: 'INFORM_VALID_EMAIL',
+      }),
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

## Creating our action

Inside _./src/actions/index.ts_, let’s drop in a new action that will handle the email sending:

```diff
+import { Resend } from 'resend';
+import { RESEND_API_KEY, FROM_EMAIL, TO_EMAIL } from 'astro:env/server';
+import { z } from 'astro:schema';

+const resend = new Resend(RESEND_API_KEY);

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
+  sendSubscription: defineAction({
+    accept: 'form',
+    input: z.object({
+      email: z.string().email('Invalid email'),
+    }),
+    handler: async input => {
+      try {
+        const { email } = input;
+        await resend.emails.send({
+          from: FROM_EMAIL,
+          to: TO_EMAIL,
+          subject: 'Hello World',
+          html: `<p>Congrats on sending your <strong>${email}</strong>!</p>`,
+        });
+        return { success: true, message: 'E-mail sent successfully ✅' };
+      } catch (error) {
+        return { success: false, message: 'There was an error sending the e-mail ❌' };
+      }
+    },
+  }),
};

```

Quick breakdown:

- We validate the form input with Zod (so only valid emails get through)
- We call Resend to send the email
- We return a simple success/fail response for the UI

## Connecting the form

Now we can connect this action to our newsletter forms. First, we'll update the wide newsletter component

_./src/pods/newsletter/components/newsletter-wide.astro_

```diff
<section >
    <form
+      id="newsletter-form-wide"
+      method="POST"
    >
    //(...)
    </form>
</section>


+<script>
+  import { actions } from 'astro:actions';
+  const form = document.getElementById('newsletter-form-wide');

+  const handleSubmit = async (event: Event) => {
+    event.preventDefault();
+    const form = event.target as HTMLFormElement;
+    const sendFormData = new FormData(form);
+    const result = await actions.sendSubscription(sendFormData);
+    if (result.data?.success) {
+      form.reset();
+    }
+  };

+  if (form && form instanceof HTMLFormElement) {
+    form.addEventListener('submit', e => handleSubmit(e));
+  }
+</script>
```

Here's what's happening:

- We import the `actions` object from `astro:actions`, which lets us call our server actions from the client side
- We add an event listener to the form's submit event to handle the form submission
- We prevent the default form submission behavior so we can handle it with JavaScript
- We create a `FormData` object from the form and call the `sendSubscription` action with this data
- We check the result of the action call, and if it was successful, we reset the form

Astro lets you use server actions directly in HTML forms, but here we're using JavaScript to have more control over the submission process and handle the response properly.

## Reusing the submit logic

Now we can reuse the same handleSubmit function in the other newsletter component.
First, we'll create a new file `newsletter.business.ts` to export the handleSubmit function.

_./src/pods/newsletter/newsletter.business.ts_

```ts
import { actions } from 'astro:actions';

export const handleSubmit = async (event: Event) => {
  event.preventDefault();
  const form = event.target as HTMLFormElement;
  const sendFormData = new FormData(form);
  const result = await actions.sendSubscription(sendFormData);
  if (result.data?.success) {
    form.reset();
  }
};
```

Now we can import and use this function in newsletter-wide.astro.

_./src/pods/newsletter/components/newsletter-wide.astro_

```diff
<script>
-  import { actions } from 'astro:actions';
+  import { handleSubmit } from '../newsletter.business';
  const form = document.getElementById('newsletter-form-wide');

-  const handleSubmit = async (event: Event) => {
-    event.preventDefault();
-    const form = event.target as HTMLFormElement;
-    const sendFormData = new FormData(form);
-    const result = await actions.sendSubscription(sendFormData);
-    if (result.data?.success) {
-      form.reset();
-    }
-  };

  if (form && form instanceof HTMLFormElement) {
     form.addEventListener('submit', e => handleSubmit(e));
  }
</script>
```

Finally, we'll use this same function in the other newsletter component.

_./src/pods/newsletter/components/newsletter-mini.astro_

```diff
    <form
      class="border-text relative flex items-center justify-between gap-2 rounded-xl border py-2 pr-2 pl-4"
+      id="newsletter-form-mini"
+      method="POST"
      >
     ...
    </form>
</section>

+<script>
+  import { handleSubmit } from '../newsletter.business';
+  const form = document.getElementById('newsletter-form-mini');

+  if (form && form instanceof HTMLFormElement) {
+    form.addEventListener('submit', e => handleSubmit(e));
+  }
+</script>
```

And that’s it! We now have a working newsletter form that uses Astro Server Actions + Resend to actually send emails. Pretty slick, right?
