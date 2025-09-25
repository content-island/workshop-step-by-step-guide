# Server Action Form with Resend

Alright, now that we’ve already played a bit with **Server Actions**, let’s take it one step further.  
This time, we’re going to connect a form to send emails using [Resend](https://resend.com/).

The cool part? We don’t need to install or configure anything new for Server Actions—we’ve already got that set up.
The only package we’ll add is **Resend**:

```bash
npm install resend
```

Next, we need to set up an account on [Resend](https://resend.com/) to obtain an API key. Don't worry about domain configuration; for this example we use onboarding@resend.dev to send the email. Once you have the key, add it to your environment variables in the `.env` file.

_.env_

```
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
    },
  },
});
```

We need to add a new action in the actions directory.

_./src/actions/index.ts_

```diff
+import { Resend } from 'resend';
+import { RESEND_API_KEY } from 'astro:env/server';
+import { z } from 'astro:schema';

export const server = {
+  sendSubscription: defineAction({
+    accept: 'form',
+    input: z.object({
+      email: z.string().email('Invalid email'),
+    }),
+    handler: async input => {
+      try {
+        const { email } = input;
+        await resend.emails.send({
+          from: 'Acme <onboarding@resend.dev>',
+          to: 'Change to your Resend account email to receive the test email',
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

Let's understand the code above:

- We import the Resend library and initialize it with our API key.
- We define a server action named `sendSubscription` that accepts form data.
- We use Zod to validate the input, ensuring that the email is in a valid format.
- In the handler, we attempt to send an email using the Resend service and return a success or error message based on the outcome.

We can now connect this action to our newsletter form.

_./src/pods/newsletter/components/newsletter-wide.astro_

```diff
<section >
    <form
+      id="newsletter-form-wide"
+      method="POST"
    >
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

Let's understand the code above:

- We import the `actions` object from `astro:actions`, which allows us to call our server actions from the client side.
- We add an event listener to the form's submit event to handle the form submission.
- We prevent the default form submission behavior to handle it via JavaScript.
- We create a `FormData` object from the form and call the `sendSubscription` action with this data.
- We check the result of the action call, and if it was successful, we reset the form.

Astro allows you to use server actions in the HTML form directly, but in this case we used JavaScript to have more control over the submission process and handle the response accordingly.

Now we can use the same handleSubmit function in the other newsletter component.
First, we need to add new file `newsletter.business.ts` to export the handleSubmit function.

_./src/pods/newsletter/components/newsletter.business.ts_

```ts
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
+  const form = document.getElementById('newsletter-form-wide');

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
+    form.addEventListener('submit', e => handleSubmit(e));
  }
</script>
```

And finally, we can also use this function in the other newsletter component.
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
