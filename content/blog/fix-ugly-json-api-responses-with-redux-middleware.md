+++
contenttype = "article"
date = "2016-05-22T17:18:09-04:00"
draft = false
image = ""
slug = "fix-ugly-json-api-responses-redux-middleware"
title = "Fix Ugly JSON API Responses with Redux Middleware"

+++

***NOTE:*** *This post uses an example Redux application, but won't go deep into explaining how Redux actually works. Check out the (top-notch) [Redux documentation](http://redux.js.org/) to get started. However you won't be expected to know [React](https://facebook.github.io/react/), which is often used with Redux applications. For rendering in this example, we just use plain JavaScript.*

### The problem of ugly JSON responses

As a JavaScript app developer, you'll inevitably be handling JSON object responses from servers --- either your own, or someone else's. When you get lucky, the keys on those responses are already be `camelCased`. Other times, those keys may be formatted according to another convention, like `snake_case` or `PascalCase`.

To a layperson this may seem like a small inconvenience. But it can become confusing to juggle variables names with the wrong casing style, especially if you've already ascribed a different meaning to variables with that style (for example, `PascalCase` variables normally refer exclusively to class names in a JavaScript app).

### A tentative solution

And no one wants to fill their code with statements like `var textContent = response.text_content`. Instead, there are existing modules out there that you can install with [`npm`](npmjs.com), that will transform your whole JSON response for you. A popular, and dead-simple one is [`camelize`](https://www.npmjs.com/package/camelize). Watch:

{{<highlight jsx>}}
import camelize from 'camelize';

const responseData = {
  data_root: {
    some_prop: {
      prop_a: 1,
      prop_b: 2
    },
    some_other_prop: [
      {
        item_a: 'x'
      },
      {
        item_b: 'y'
      }
    ]
  }
};

console.log(camelize(responseData));
/* prints:
 * { dataRoot: 
 *    { someProp: { propA: 1, propB: 2 },
 *      someOtherProp: [ [Object], [Object] ] } }
 */
{{</highlight>}}

If the keys in your original response are `PascalCase`, `camelize` won't be much help, but you can use a similar module called [`penrillian-camelize`](https://www.npmjs.com/package/penrillian-camelize) that adds support for converting from `PascalCase` to `camelCase`. If there's some other transform you need to apply to your keys, you can use [`recursive-json-key-transform`](https://www.npmjs.com/package/recursive-json-key-transform) (disclaimer: I published this module), which lets you specify your own function to apply to all the keys in your JSON response. You might want to check out the module [`i`](https://www.npmjs.com/package/i), which contains a collection of common string transforms that can be used in conjunction with `recursive-json-key-transform`.

### Integrating this in your app's data flow
