[code raiders qwik full stack saas](https://coderaiders.com/site/view?pkgId=35)
[youtube series](https://www.youtube.com/watch?v=zLHYDY9dAbs&list=PLkswEDcfBXYcl1gW7L5zyCVF9LpGhlOqu)
[code raiders setup reference](https://coderaiders.com/site/view?pkgId=35)
asd

```bash
npm ci
touch .env
```

```bash
SUPABASE_URL=your-supabase-url  
SUPABASE_SECRET_KEY=your-supabase-secret-key
```


### Routing
- named pages do not work. Named component imports are fine, but the pages need to all be index.ts (for endpoints) or tsx for pages
- **`[id]`** is a directory that represents a dynamic route segment, in this example `id` is the string parameter accessible by `useLocation().params.id`.
- **`[...catchall]`** is a directory that represents a dynamic catch-all route, in this example `catchall` is the string parameter accessible by `useLocation().params.catchall`.
- **`index.tsx|mdx` files** are the pages/endpoints/middleware.
- **`layout.tsx` files** are the layouts.

```
src/routes/blog/index.tsx → /blog

src/routes/user/[username]/index.tsx → /user/:username (/user/foo)

src/routes/post/[...all]/index.tsx → /post/* (/post/2020/id/title)
```


- url segment is state 
	- eg src/routes/user/[username]/index.tsx returns 'username' as the result of a call to useLocation.params which is the key and the value is whatever you visit in the url at the location
		- eg localhost/user/foo will return 'foo' as the value mapped to useLocation.params.username 
```ts
// src/routes/user/[username]/index.tsx
import { component$ } from "@builder.io/qwik";
import { useLocation } from "@builder.io/qwik-city";

export default component$(() => {
	const loc = useLocation();
	return <div>Hello {loc.params.username}!</div>;
});
```

Inside the `src/routes` directory, all files named `index` are considered pages/endpoint/middleware, Qwik supports the following extensions: `.ts`, `.tsx`, `.md` and `.mdx`.


### Page Endpoint Middleware
Pages/endpoint/middleware are the leaf nodes of the routing tree, ie, **the modules that will handle the request and return an HTTP response**.

### Page index.tsx
- When `index.tsx` or `index.ts` exports a Qwik component as the default export, Qwik City will render the component and return an HTML response as a webpage.
	- tsx/md[x] are for pages, ts is for endpoints and middleware

```ts
// src/routes/index.tsx

import { component$ } from '@builder.io/qwik'; 

export default component$(() => {  
	return <h1>Hello World</h1>;
});
```

### Endpoint.ts
- tsx/md[x] are for pages, ts is for endpoints and middleware
- A `index.ts` can also access the HTTP request directly and return a raw HTTP response without involving any Qwik Component. This is done by exporting an `onRequest` method or `onGet`, `onPost`, `onPut`, `onDelete` depending on if you only want to handle a specific request given its HTTP method.
```ts
// src/routes/index.ts
import type { RequestHandler } from '@builder.io/qwik-city'; 

export const onGet: RequestHandler = ({ json }) => { 
	json(200, { message: 'Hello World' });
};
```
- Notice that in the last example, there is no default export. This is because we are not rendering a Qwik component, but rather we are handling the request directly and returning a JSON response. This is useful to implement RESTful APIs or any other type of HTTP endpoint.

### Page + Endpoint
- needs to be a tsx file if it's rendering typed jsx but a tsx file can also resolve json as an endpoint
- the following file will return the html or json response, depending on whether the query param format=json is appended to the url
```tsx
// src/routes/endpoints/index.tsx
import { component$ } from "@builder.io/qwik";
import type { RequestHandler } from "@builder.io/qwik-city";

export const onRequest: RequestHandler = ({ headers, method, query, json }) => {
	headers.set("Cache-Control", "private");
		if (query.get("format") === "json" || method === "POST") {
		json(200, { message: "Hello World" });
	}
};

export default component$(() => {
	return <h1>Hello World</h1>;
});
```
- GET http://45.79.208.40:5173/endpoints/?format=json  or POST http://45.79.208.40:5173/endpoints/ returns
```json
{"message": "Hello World"}
```
- http://45.79.208.40:5173/endpoints/ returns
```html
# Hello World

[Made with ♡ by Builder.io2023-08-15T19:04:22.698Z](https://www.builder.io/)
```


### Layout Files
picture a dashboard site where all the pages are under the `/admin/*` directory:

- **Shared request handling:** The request cookies need to be validated before even rendering the page, otherwise, render a blank 401 page.
- **Shared UI:** All pages share a common header showing the user's name and profile picture.

Instead of repeating the same code in each route, we can use layouts to automatically reuse common parts, and also to add middleware to the route.
```ts
src/
	routes/
		admin/
			layout.tsx // layout for admin module
			index.tsx
		layout.tsx // layout for shell
		index.tsx 
```

### Middleware Layouts
Since layouts can implement request handling with `onRequest` or `onGet`, `onPost`, `onPut`, `onDelete`, they can be used to implement middleware, for example, to validate the request cookies before rendering the page.

For the route `https://example.com/admin`, the `onRequest` methods will be executed in the following order:

1. `src/routes/layout.tsx`'s `onRequest`
2. `src/routes/admin/layout.tsx`'s `onRequest`
3. `src/routes/admin/index.tsx`'s component

### Nested layouts

Layouts also **provide a way to add common UI to the rendered page**. For example, if we want to add a common header to all the routes, we can add a `Header` component to the root layout.

For the given example, the Qwik components will be rendered in the following order:

1. `src/routes/layout.tsx`'s component
2. `src/routes/admin/layout.tsx`'s component
3. `src/routes/admin/index.tsx`'s component

```
<RootLayout>  
	<AdminLayout>    
	<AdminPage />  
	</AdminLayout>
</RootLayout>
```

## SPA Navigation

With Qwik, the distinction between MPA and SPA disappears; every app can be both at the same time. The choice is no longer an architectural design determined at the beginning of the project, instead this decision can be made for every link.

Qwik provides a `<Link>` component and `useNavigate()` hook. These can be used to initiate an SPA refresh or navigation between pages.

The `Link` component is the recommend way to navigate as it uses the HTML `<a>` tag, which is the most accessible way to move between pages. However, if you need to navigate programmatically, you can use the `useNavigate()` hook.

```ts
import { component$ } from '@builder.io/qwik';
import { Link, useNavigate } from '@builder.io/qwik-city'; 

export default component$(() => { 
	const nav = useNavigate(); 
	return ( 
		<div> 
			<Link href="/about">About (prefered)</Link> 
			<button onClick$={() => nav('/about')}>About</button> 
		</div> 
	);
});
```

> The `Link` component uses the `useNavigate()` hook internally.

### Refreshing

The `Link` with the `reload` prop can be used to refresh the current page. You can also call the `nav()` function from the `useNavigate()` hook, without arguments.

```ts
import { component$ } from '@builder.io/qwik';
import { Link, routeLoader$, useNavigate } from '@builder.io/qwik-city'; 

export const useServerTime = routeLoader$(() => {  
// This will re-execute in the server when the page refreshes.  
	return Date.now();
}); 

export default component$(() => {  
	const nav = useNavigate();  
	const serverTime = useServerTime();   
	return (    
		<div>      
			<Link reload>Refresh (better accesibility)</Link>      
			<button onClick$={() => nav()}>Refresh</button>      
			<p>Server time: {serverTime.value}</p>    
		</div>  
	);
});
```

> When the page refreshes, all the matching `routeLoader$` and server handlers (`onRequest`) will re-execute in the server and the UI will re-render accordingly.

> While refreshing the page, the `isNavigating` boolean from `useLocation()` will be `true` until the page is fully rendered.


### Scroll Restoration
- scroll restoration (the part of the page you visited before back button) is kept in the history array
	-  similar to deep linking by an id
- this is SPA behavior MPA doesn't support this
- Link and a (anchor) are interchangeable such that Link will give SPA context links (even between pages) and <a /> will force a full page refresh without context

### Request Event
- all the properties and methods of the request event that come into the handler in endpoints
```ts
import type { RequestHandler } from "@builder.io/qwik-city";

export const onRequest: RequestHandler = ({ headers, method, query, json }) => {
	headers.set("Cache-Control", "private");
	if (query.get("format") === "json" || method === "POST") {
		json(200, { message: "Hello World" });
	}
};
```
Each request handler, such as `onRequest`, `onGet`, `onPost`, etc., are passed in a `RequestEvent` object as the first argument to the handler. The `RequestEvent` object contains utility functions and properties to get and set values to the server's request and response. This object contains the following properties:

- `basePathname`: The base pathname of the request, which can be configured at build time. Defaults to `/`.
- `cacheControl`: Convenience function to set the [Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control) response header.
- `cookie`: HTTP request and response [cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies). Use the `get()` method to retrieve a request cookie value. Use the `set()` method to set a response cookie value.
- `env`: Platform provided environment variables.
- `error`: When called, the response will immediately end with the given status code. This could be useful to end a response with `404`, and use the 404 handler in the routes directory. See [Status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) for which status code should be used.
- `getWritableStream`: Low-level access to write to the HTTP response stream. Once `getWritableStream()` is called, the status and headers can no longer be modified and will be sent over the network.
- `headers`: HTTP [response headers](https://developer.mozilla.org/en-US/docs/Glossary/Response_header).
- `html`: Convenience method to send an HTML body response. The response will be automatically set the `Content-Type` header to`text/html; charset=utf-8`. An `html()` response can only be called once.
- `json`: Convenience method to JSON stringify the data and send it in the response. The response will be automatically set the `Content-Type` header to`application/json; charset=utf-8`. A `json()` response can only be called once.
- `locale`: Which locale the content is in. The locale value can be retrieved from selected methods using `getLocale()`.
- `method`: HTTP request [method](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) value.
- `next`: Call the next request handler. This is useful for middleware.
- `params`: URL path params which have been parsed from the current url pathname segments. Use `query` to instead retrieve the query string search params.
- `parseBody`: This method will check the request headers for a `Content-Type` header and parse the body accordingly. It supports `application/json`, `application/x-www-form-urlencoded`, and `multipart/form-data` content types. If the `Content-Type` header is not set, it will return `null`.
- `pathname`: URL pathname value. Does not include the protocol, domain, query string (search params) or hash.
- `platform`: Platform specific data and functions.
- `query`: URL query string [URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) value. Use `params` to instead retrieve the route params found in the url pathname.
- `redirect`: URL to redirect to. When called, the response will immediately end with the correct redirect status and headers. See [Redirects](https://developer.mozilla.org/en-US/docs/Web/HTTP/Redirections) for which status code should be used.
- `request`: HTTP [Request](https://developer.mozilla.org/en-US/docs/Web/API/Request).
- `send`: Send a body response. The `Content-Type` response header is not automatically set when using `send()` and must be set manually. A `send()` response can only be called once.
- `sharedMap`: Shared Map across all the request handlers. Every HTTP request will get a new instance of the shared map. The shared map is useful for sharing data between request handlers.
- `status`: HTTP response [status code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status). Sets the status code when called with an argument. Always returns the status code, so calling `status()` without an argument will can be used to return the current status code.
- `text`: Convenience method to send an text body response. The response will be automatically set the `Content-Type` header to`text/plain; charset=utf-8`. An `text()` response can only be called once.
- `url`: HTTP request [URL](https://developer.mozilla.org/en-US/docs/Web/API/URL).

## Concepts

### Reactivity

- Misko is using Proxy for his in-built reactivity in Qwik & it's very fundamental js; it's like straight out of eloquent javascript book. the Proxy acts like a middleware which takes a request, does processing in relation to that request, then passes it on... so the question of how do we get reactive values (updates based on changes) is a question of how do we construct these Proxies for the state to be updated? useStore wraps your state value (eg count:0) in a proxy. so that when you say count++ (or rather, when you inc the variable wrapping count), only the component containing that state updates (the [en]closure).

- the reactivity graph is serialized to HTML so the browser can use the info sans being forced to do a single pass through all components to rebuild the graph.

- Reactivity can be done in a few ways:

1. Using explicit registration of listeners using `.subscribe()` (for example, RxJS)
2. Using implicit registration using a compiler (for example, Svelte)
3. Using implicit registration using proxies.

Qwik uses proxies for a few reasons:

1. Using explicit registration such as `.subscribe()` would require the system to serialize all of the subscribed listeners to avoid hydration. Serializing subscribed closures would not be possible as all the subscribe functions would have to be lazy-loaded and asynchronous (too expensive).
2. Using the compiler to create graphs implicitly would work, but only for components. Intra component communications would still require `.subscribe()` methods and hence suffer the issues described above.

Because of the above constraints, Qwik uses proxies to keep track of the reactivity graph.

- Use [`useStore()`](https://qwik.builder.io/docs/components/state/#usestore) to create a store proxy.
- The proxy notices the reads and creates subscriptions that are serializable.
- The proxy notices the writes and uses the subscription information to invalidate the relevant components.

```ts
export const Counter = component$(() => { 
	const store = useStore({ count: 0 }); 
	return <button onClick$={() => store.count++}>{store.count}</button>;
});
```

1. proxy created for state by useStore
2. component created with subscription to state via proxy
3. server serializes state and subscription (the subscription relationship between component and state) to html
4. 


## Counter Example

```
export const Counter = component$(() => {  const store = useStore({ count: 0 });   return <button onClick$={() => store.count++}>{store.count}</button>;});
```

1. The server performs an initial render of the component. The server rendering includes creating the proxy represented by `store`.
2. The initial render invokes the OnRender method, which has a reference to the `store` proxy. The rendering puts the proxy to "learn" mode. During the build-up of JSX, the proxy observes a read of the `count` property. Because the proxy is in "learn" mode, it records that the `Counter` has a subscription on the `store.count`.
3. The server serializes the state of the application into HTML. This includes the `store` as well as subscription information which says that `Counter` is subscribed to `store.count`.
4. In the browser, the user clicks the button. Because the click event handler closed over `store`, Qwik restores the store proxy. The proxy contains the application state (the count) and the subscription, which associates the `Counter` with `state.count`.
5. The event handler increments the `store.count`. Because the `store` is a proxy, it notices the write and uses the subscription information to invalidate the `Counter`.
6. After `requestAnimationFrame`, the `Counter` downloads the rendering function and re-runs the OnRender method.
7. During the OnRender, the subscription list is cleared, and a new subscription list is built up by observing what reads the JSX building performs.

## Unsubscribe example

```
export const ComplexCounter = component$(() => {  const store = useStore({ count: 0, visible: true });   return (    <>      <button onClick$={() => (store.visible = !store.visible)}>        {store.visible ? 'hide' : 'show'}      </button>      <button onClick$={() => store.count++}>increment</button>      {store.visible ? <p>{store.count}</p> : null}    </>  );});
```

This example is a more complicated counter.

- It contains the `increment` button, which always increments `store.count`.
- It contains a `show`/`hide` button which determines if the count is shown.

1. On the initial render, the count is visible. Therefore the server creates a subscription that records that `ComplexCounter` needs to get re-rendered if either `store.count` or `store.visible` changes.
2. If the user clicks on `hide`, the `ComplexCounter` rerenders. The re-rendering clears all of the subscriptions and records new ones. This time the JSX does not read `store.count`. Therefore, only `store.visible` gets added to the list of subscriptions.
3. User clicking on `increment` will update `store.count`, but doing so will not cause the component to re-render. This is correct because the counter is not visible, so re-rendering would be a no-op.
4. If the user clicks `show`, the component will re-render and this time the JSX will read both `store.visible` as well as `store.count`. The subscription list is once again updated.
5. Now, clicking on `increment` updates the `store.count`. Because the count is visible, the `ComplexCounter` is subscribed to `store.count`.

Notice how the set of subscriptions automatically updates as the component renders different branches of its JSX. The advantage of the proxy is that the subscriptions update automatically as the applications execute, and the system can always compute the smallest set of invalidated components.

## [](https://qwik.builder.io/docs/concepts/reactivity/#deep-objects)Deep objects

So far, the examples show the store (`useStore()`) was a simple object with primitive values.

```
export const MyComp = component$(() => {  const store = useStore({    person: { first: null, last: null },    location: null  });   store.location = {street: 'main st'};   return (    <section>      <p>{store.person.last}, {store.person.first}</p>      <p>{store.location.street}</p>    </section>  );})
```

In the above examples, Qwik will automatically wrap child objects `person` and `location` into a proxy and correctly create subscriptions on all deep properties.

The wrapping behavior described above has one surprising side-effect. Writing and reading from a proxy auto wraps the object, which means that the identity of the object changes. This should normally not be an issue, but it is something that the developer should keep in mind.

```
export const MyComp = component$(() => {  const store = useStore({ person: null });  const person = { first: 'John', last: 'Smith' };  store.person = person; // store.person auto wraps object into proxy   if (store.person !== person) {    // The consequence of auto wrapping is that the object identity changes.    console.log('store auto-wrapped person into a proxy');  }});
```

## [](https://qwik.builder.io/docs/concepts/reactivity/#out-of-order-rendering)Out-of-order rendering

Qwik components are rendered out of order. A component can be rendered without forcing a parent component to be rendered first or a child components to be rendered as a consequence of the component render. This is an important property of Qwik because it allows Qwik applications to only re-render components which have been invalidated due to a state change rather than re-rendering the whole component tree on a state change.

When components are rendered, they need to have access to their props. Parent components create props. The props must be [serializable](https://qwik.builder.io/docs/concepts/resumable/#serialization) for the component to render independently from the parent.

## [](https://qwik.builder.io/docs/concepts/reactivity/#invalidating-child-components)Invalidating child components

When re-rendering a component, the child component's props either stay the same or are updated. A child component only invalidates if its props change.

```
export const Child = component$((props: { count: number }) => {  return <span>{props.count}</span>;}); export const MyApp = component$(() => {  const store = useStore({ a: 0, b: 0, c: 0 });   return (    <>      <button onClick$={() => store.a++}>a++</button>      <button onClick$={() => store.b++}>b++</button>      <button onClick$={() => store.c++}>c++</button>      {JSON.stringify(store)}       <Child count={store.a} />      <Child count={store.b} />    </>  );});
```

In the above example, there are two `<Child/>` components.

- Every time a button is clicked, one of the three counters is incremented. A change of counter state will cause the `MyApp` component to re-render on each click.
- If `store.c` has been incremented, none of the child components get re-rendered. (And therefore, their code does not get lazy-loaded)
- If `store.a` has been incremented, then only `<Child count={store.a}/>` will re-render.
- If `store.b` has been incremented, then only `<Child count={store.b}/>` will re-render.

Notice that the child components only re-render when their props change. This is an important property of Qwik applications as it significantly limits the amount of re-rendering the application must do on some state change. While less re-rendering has performance benefits, the real benefit is that large portions of the applications do not get downloaded if they don't need to be re-rendered.