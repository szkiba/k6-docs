---
title: 'isDisabled([options])'
description: 'Browser module: locator.isDisabled method'
---

# isDisabled([options])

Checks if the element is disabled.

<TableWithNestedRows>

| Parameter       | Type   | Default | Description                                                                                                                                                                                                                                                                                                         |
| --------------- | ------ | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| options         | object | `null`  |                                                                                                                                                                                                                                                                                                                     |
| options.timeout | number | `30000` | Maximum time in milliseconds. Pass `0` to disable the timeout. Default is overridden by the `setDefaultTimeout` option on [BrowserContext](https://grafana.com/docs/k6/<K6_VERSION>/javascript-api/k6-browser/browsercontext/) or [Page](https://grafana.com/docs/k6/<K6_VERSION>/javascript-api/k6-browser/page/). |

</TableWithNestedRows>

### Returns

| Type            | Description                                                                   |
| --------------- | ----------------------------------------------------------------------------- |
| `Promise<bool>` | A Promise that fulfills with `true` if the element is disabled, else `false`. |

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
  await page.goto('https://test.k6.io/browser.php');
  const text = page.locator('#input-text-disabled');
  const isDisabled = await text.isDisabled();
  if (isDisabled) {
    console.log('element is disabled');
  }
}
```

{{< /code >}}
