
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

