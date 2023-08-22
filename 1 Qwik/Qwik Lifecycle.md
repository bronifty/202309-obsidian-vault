- useTask$ blocking server code (runs during SSR) for synchronizing fw code with userland effects; runs before render (eg update component$ in response to store.count++);  
- useResource$ http streaming for fetch/network request (similar to remix run's loader with defer)
- useVisibleTask$ eagerly render on the client eg lazy load image when visible (eg client:visible attribute:value setting in Astro)


and here is our hydration mystery partially solved with some web remediation around 'mounting' components: TL;DR: mounting components is when they are 1 built 2 inserted & 3 made interactive; Qwik does this via SSR not client ergo no unmount reqd
#### Web Remediation
- Mounting
	- Mounting refers to the process of creating and inserting a component into the DOM. During this phase, the component's initial state is set, and its initial render is performed. In the context of server-side rendering (SSR), mounting means that the component is built and rendered on the server
	- In Qwik's architecture, components are mounted only once, and this happens on the server. The server builds the components, runs any necessary tasks or side effects (such as those defined with `useTask$`), and then sends the fully-rendered HTML to the client. Because Qwik doesn't hydrate components on the client-side, the mounting process doesn't happen again in the browser.
- Unmounting
	- Unmounting refers to the process of removing a component from the DOM, typically when it's no longer needed. Since Qwik's architecture focuses on mounting components on the server and not re-running the same logic on the client-side, there is no traditional unmounting phase in the client-side lifecycle. Components are rendered on the server, and the client simply resumes from the state provided by the server.

One unique aspect of Qwik, is that components are mounted only ONCE across the server and client. This is a property of resumability. What it means, is that if `useTask$` runs during SSR it will not run again in the browser because Qwik does not hydrate.

### [](https://qwik.builder.io/docs/components/tasks/#track)`   `


### Tasks (side effect effect; do work not return value)
- async ops to init component or update state
- similar to useEffect
- async; run on server and browser; run before and can block rendering
- `useTask$()` should be your default go-to API for running asynchronous (or synchronous) work as part of component initialization or state change. It is only when you can't achieve what you need with `useTask$()` that you should consider using `useVisibleTask$()` or `useResource$()`.

#### useTask$ 
- Reruns the `taskFn` when the observed inputs change (similar to useEffect with dep arr).
- Use `useTask` to observe changes on a set of inputs, and then re-execute the `taskFn` when those inputs change.
- The `taskFn` only executes if the observed inputs change. To observe the inputs, use the `obs` function to wrap property reads. This creates subscriptions that will trigger the `taskFn` to rerun.

```ts
useTask$: (first: TaskFn, opts?: UseTaskOptions | undefined) => void
```

```ts
import { component$, useSignal, useTask$ } from "@builder.io/qwik";

export default component$(() => {
	const fibonacci = useSignal<number[]>();
	useTask$(async () => {
		const size = 40;
		const array = [];
		array.push(0, 1);
		for (let i = array.length; i < size; i++) {
			array.push(array[i - 1] + array[i - 2]);
			await delay(1000);
		}
		fibonacci.value = array;
	});
	return <p>{fibonacci.value?.join(", ")}</p>;
});
const delay = (time: number) => new Promise((res) => setTimeout(res, time));
```

In this example

- The `useTask$()` computes the fibonacci number one entry per 100 ms. So 40 entries take 4 seconds to render.
    - The `useTask$()` executes on the server as part of the SSR (the result may be cached in CDN.)
    - Because the `useTask$()` blocks rendering, the rendered HTML page takes 4 seconds to render.
- Because this task has no `track()` it will never rerun, making it effectively an initialization code.
    - Because this component only renders on the server, the `useTask$()` will never run on the browser. (code will never download to browser.)

Notice that `useTask$()` runs **BEFORE** the actual rendering and on the server. Therefore if you need to do DOM manipulation, use [`useVisibleTask$()`](https://qwik.builder.io/docs/components/tasks/#usevisibletask) instead, which runs on the browser after rendering.

Use `useTask$()` when you need to:
- Run async tasks before rendering
- Run code only once before the component is first rendered
- Programmatically run side-effect code when state changes

Note, if you're thinking about loading data (like using `fetch()`) inside of `useTask$`, consider using [`useResource$()`](https://qwik.builder.io/docs/(qwik)/components/state/#useresource) instead. This API is more efficient in terms of leveraging SSR streaming and parallel data fetching.

### On mount

There is not an specific "on-mount" hook in Qwik because `useTask$()` without tracking effectively behaves like a `mount` hook.

This is because `useTask$` runs always at least once when the component is first mounted.

```
import { component$, useTask$ } from '@builder.io/qwik'; export default component$(() => {   useTask$(async () => {    // A task without `track` any state effectively behaves like a `on mount` hook.    console.log('Runs once when the component mounts in the server OR client.');  });   return <div>Hello</div>;});
```

One unique aspect of Qwik, is that components are mounted only ONCE across the server and client. This is a property of resumability. What it means, is that if `useTask$` runs during SSR it will not run again in the browser because Qwik does not hydrate.
### `track()`

There are times when it is desirable to re-run a task when a component state changes. This is done by using the `track()` function. The `track()` function allows you to set up a dependency on a component state on the server (if initially rendered there) and then re-execute the task when the state changes _on the browser_ (the same task will never be executed twice on the server side).

**Note**: If all you want to do is compute a new state from an existing state synchronously, you should use [`useComputed$()`](https://qwik.builder.io/docs/components/state/#usecomputed) instead.

```ts
import { component$, useSignal, useTask$ } from "@builder.io/qwik";
import { isServer } from "@builder.io/qwik/build";
export default component$(() => {
	const text = useSignal("Initial text");
	const delayText = useSignal("");
	useTask$(({ track }) => {
		track(() => text.value);
		const value = text.value;
		const update = () => (delayText.value = value);
		isServer
		? update() // don't delay on server render value as part of SSR
		: delay(500).then(update); // Delay in browser
	});
	return (
		<section>
			<label>
			Enter text: <input bind:value={text} />
			</label>
			<p>Delayed text: {delayText}</p>
		</section>
	);
});
const delay = (time: number) => new Promise((res) => setTimeout(res, time));
```

On the server:
- The `useTask$()` runs on the server and the `track()` function sets up a subscription on `text` signal.
- The page is rendered.

On the browser:
- The `useTask$()` does not have to run (or be downloaded) eagerly because Qwik knows that the task is subscribed to the `text` signal from the server execution.
- When the user types in the input box, the `text` signal changes. Qwik knows that the `useTask$()` is subscribed to the `text` signal and it is at this time that the `useTask$()` closure is brought into the JavaScript VM to be executed.

The `useTask$()`
- The `useTask$()` blocks rendering until it completes. If you don't want to block rendering (as in this case) make sure that the task is resolved, and run the delay work on a separate unconnected promise. (In our case we don't await `delay()`. Doing so would block rendering.)

Sometimes it is required to only run code either in the server or in the client. This can be achieved by using the `isServer` and `isBrowser` booleans exported from `@builder.io/qwik/build` as shown above.

