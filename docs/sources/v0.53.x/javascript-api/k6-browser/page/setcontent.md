---
title: 'setContent(html[, options])'
description: 'Browser module: page.setContent(html[, options]) method'
---

# setContent(html[, options])

<!-- TODO: Add link to Events in options.waitUntil description -->

Sets the supplied HTML string to the current page.

<TableWithNestedRows>

| Parameter         | Type   | Default | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ----------------- | ------ | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| html              | string | `''`    | HTML markup to assign to the page.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| options           | object | `null`  |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| options.timeout   | number | `30000` | Maximum operation time in milliseconds. Pass `0` to disable the timeout. The default value can be changed via the [browserContext.setDefaultNavigationTimeout(timeout)](https://grafana.com/docs/k6/<K6_VERSION>/javascript-api/k6-browser/browsercontext/setdefaultnavigationtimeout/), [browserContext.setDefaultTimeout(timeout)](https://grafana.com/docs/k6/<K6_VERSION>/javascript-api/k6-browser/browsercontext/setdefaulttimeout/), [page.setDefaultNavigationTimeout(timeout)](https://grafana.com/docs/k6/<K6_VERSION>/javascript-api/k6-browser/page/setdefaultnavigationtimeout/) or [page.setDefaultTimeout(timeout)](https://grafana.com/docs/k6/<K6_VERSION>/javascript-api/k6-browser/page/setdefaulttimeout/) methods. Setting the value to `0` will disable the timeout. |
| options.waitUntil | string | `load`  | When to consider operation to have succeeded. See Events for more details.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |

</TableWithNestedRows>

### Returns

| Type            | Description                                                                |
| --------------- | -------------------------------------------------------------------------- |
| `Promise<void>` | A Promise that fulfills when the page has been set with the supplied HTML. |

### Example

{{< code >}}

```javascript
import { browser } from 'k6/browser';

export const options = {
  scenarios: {
    browser: {
      executor: 'shared-iterations',
      options: {
        browser: {
          type: 'chromium',
        },
      },
    },
  },
};

export default async function () {
  const page = await browser.newPage();

  const htmlContent = `
    <!doctype html>
    <html>
      <head><meta charset='UTF-8'><title>Test</title></head>
      <body>Test</body>
    </html>
  `;

  await page.setContent(htmlContent);
}
```

{{< /code >}}
