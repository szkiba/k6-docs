---
title: 'Migrating browser scripts to k6 v0.52'
description: 'A migration guide to ease the process of transitioning to the new k6 browser module version bundled with k6 v0.52'
weight: 04
---

# Migrating browser scripts to k6 v0.52

This guide outlines the key changes you will need to make when moving your existing k6 browser test scripts to the _upcoming latest_ [k6 browser module](https://grafana.com/docs/k6/<K6_VERSION>/javascript-api/k6-experimental/browser) version bundled with [k6 0.52](https://github.com/grafana/k6/releases/tag/v0.52.0).

## Key changes

In the upcoming release of k6, we will be introducing a breaking change that will affect _all scripts_ that use the k6 browser module. The breaking changes are:

1. Converting most of the k6 browser module APIs to asynchronous (async) APIs. That means they will return a `promise` that will `resolve` to a value when the API call succeeds or `reject` when an error occurs.
2. A side effect of making this async changes is that you will need to use a workaround to work with the k6 [check](http://grafana.com/docs/k6/<K6_VERSION>/javascript-api/k6/check/) API.
3. Finally, it's also worth reiterating that the [group](http://grafana.com/docs/k6/<K6_VERSION>/javascript-api/k6/group/) API still doesn't work with async APIs.

If you are interested in the rationale behind this change, refer to the [v0.51 release notes](https://github.com/grafana/k6/releases/tag/v0.51.0).

## Migrating to async

To ensure your scripts work with the upcoming release of k6, you must ensure that all the synchronous (sync) APIs that will be migrated over to async are handled appropriately. In most cases, you will only need to add `await` in front of the API call.

For example, before:

{{< code >}}

```javascript
import { browser } from 'k6/experimental/browser';

// options block

export default async function () {
  const page = browser.newPage();

  // ...
}
```

{{< /code >}}

And after:

{{< code >}}

```javascript
import { browser } from 'k6/experimental/browser';

// options block

export default async function () {
  const page = await browser.newPage();

  // ...
}
```

{{< /code >}}

You might have already encountered async APIs when working with the browser module, such as [page.click](http://grafana.com/docs/k6/<K6_VERSION>/javascript-api/k6-experimental/browser/page/click/), so the use of `async` and `await` keywords might be familiar to you.

The complete list of APIs that will be migrated over to be async can be found in this [GitHub comment](https://github.com/grafana/xk6-browser/issues/428#issuecomment-1964020837). You will also find a screenshot of a comparison between the [fillform.js example](https://github.com/grafana/xk6-browser/blob/main/examples/fillform.js) in `v0.51` and `v0.52` in another [comment](https://github.com/grafana/xk6-browser/issues/428#issuecomment-2141634960) in the same GitHub issue.

### When to migrate

Awaiting on something that’s not a `thenable` simply returns that value. In other words, _it's safe to add the `await` keyword in front of the browser module API methods in your scripts to be ready for the upcoming changes_.

For example this works in both k6 `v0.51` and `v0.52`:

{{< code >}}

```javascript
import { check } from 'k6';
import { browser } from 'k6/experimental/browser';

export const options = {
  scenarios: {
    ui: {
      executor: 'shared-iterations',
      options: {
        browser: {
          type: 'chromium',
        },
      },
    },
  },
  thresholds: {
    checks: ['rate==1.0'],
  },
};

export default async function () {
  const context = await browser.newContext();
  const page = await context.newPage();

  try {
    // Navigate to the website under test.
    await page.goto('https://test.k6.io/', { waitUntil: 'networkidle' });
    await Promise.all([
      page.waitForNavigation(),
      page.locator('a[href="/my_messages.php"]').click(),
    ]);

    // Check that the header reads "Unauthorized".
    const h2 = page.locator('h2');
    const headerOK = (await h2.textContent()) == 'Unauthorized';
    check(headerOK, { header: headerOK });

    // Take a screenshot of the page.
    await page.screenshot({ path: 'screenshot.png' });
  } finally {
    await page.close();
  }
}
```

{{< /code >}}

### Promise chain in v0.51

If you want to promise chain the newly migrated async APIs, you won't be able to do that until after the migration and release of k6 `v0.52`.

For example, you can't do this yet:

{{< code >}}

```javascript
import { browser } from 'k6/experimental/browser';

// options block

export default async function () {
  const page = browser.newPage();

  // ...

  await page.$('#text1').then((text) => text.type('hello world'));

  // ...
}
```

{{< /code >}}

You could do this instead:

{{< code >}}

```javascript
import { browser } from 'k6/experimental/browser';

// options block

export default async function () {
  const page = browser.newPage();

  // ...

  const text = await page.$('#text1');
  await text.type('hello world');

  // ...
}
```

{{< /code >}}

## Working with k6 check

The k6 `check` API wasn't designed to work with async APIs. However, there is a workaround. In your current scripts, you may have used a `check` like so:

{{< code >}}

```javascript
import { browser } from 'k6/experimental/browser';
import { check } from 'k6';

// options block

export default async function () {
  const page = browser.newPage();

  // ...

  check(page, {
    header: (p) => p.locator('h2').textContent() == 'Welcome, admin!',
  });

  // ...
}
```

{{< /code >}}

Since the `locator.textContent` API is being migrated to async, and `check` doesn't work with async APIs, you will need to do something like this:

{{< code >}}

```javascript
import { browser } from 'k6/experimental/browser';
import { check } from 'k6';

// options block

export default async function () {
  const page = browser.newPage();

  // ...

  const h2 = page.locator('h2');
  const headerOK = (await h2.textContent()) == 'Welcome, admin!';
  check(headerOK, { header: headerOK });

  // ...
}
```

{{< /code >}}

## Groups

A note on [groups](https://grafana.com/docs/k6/<K6_VERSION>/using-k6/tags-and-groups/#groups), they don't work with async APIs either, there is no workaround as of yet. Here's the [GitHub issue](https://github.com/grafana/k6/issues/2728) that you can follow to keep up-to-date with relevant news on a group API that works with async APIs.

# Upcoming major breaking changes

This is likely the last significant breaking change until the browser module is considered stable enough to be a non-experimental module in k6. When the browser module is no longer experimental, this should only be a small change in the import path from `k6/experimental/browser` to `k6/browser`.

For more information on graduating extension out of experimental, refer to the [extension graduation process](http://grafana.com/docs/k6/<K6_VERSION>/extensions/explanations/extension-graduation/).

# Where to get extra help

With the release of k6 `v0.52`, you'll also be able to find the latest type definitions and updates to the k6 browser documentation. Refer to [Configure k6 IntelliSense](http://grafana.com/docs/k6/<K6_VERSION>/set-up/configure-k6-intellisense/) for details on how to work with the type definition file in your project.

For all the details, review the _upcoming_ CHANGELOG for k6 version 0.52. You can also ask questions in [the community forum](https://community.grafana.com/c/grafana-k6/k6-browser/79) if you need help.