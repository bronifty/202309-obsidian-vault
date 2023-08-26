[frontend masters blog article](https://frontendmasters.com/blog/vanilla-javascript-reactivity/)

1 works
```ts
const pubSub = {

events: {

// "update": [(data)=>console.log(data)]

},

subscribe(event, callback) {

if (!this.events[event]) this.events[event] = [];

this.events[event].push(callback);

},

publish(event, data) {

if (this.events[event]) this.events[event].forEach(callback => callback(data));

}

};

  

pubSub.subscribe('update', data => console.log(data));

pubSub.publish('update', 'Some update'); // Some update
```

2 change name to type
```js
<script>

const pizzaEvent = new CustomEvent("pizzaDelivery", {

detail: {

type: "supreme",

},

});

  

window.addEventListener("pizzaDelivery", (e) => console.log(e.detail.type));

window.dispatchEvent(pizzaEvent);

  

</script>
```

3 change name to type
```js
<div id="pizza-store"></div>

  

<script>

const pizzaEvent = new CustomEvent("pizzaDelivery", {

detail: {

type: "hello pizza",

},

});

  

const pizzaStore = document.querySelector('#pizza-store');

pizzaStore.addEventListener("pizzaDelivery", (e) => console.log(e.detail.type));

pizzaStore.dispatchEvent(pizzaEvent);
```

4 EventTarget (might be good to introduce the concept)
EventTarget interface methods
[`EventTarget.addEventListener()`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)
[`EventTarget.removeEventListener()`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/removeEventListener)
[`EventTarget.dispatchEvent()`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/dispatchEvent)
```ts
class PizzaStore extends EventTarget {

constructor() {

super();

}

addPizza(flavor) {

// fire event directly on the class

this.dispatchEvent(new CustomEvent("pizzaAdded", {

detail: {

pizza: flavor,

},

}));

}

}

  

const Pizzas = new PizzaStore();

Pizzas.addEventListener("pizzaAdded", (e) => console.log('Added Pizza', e.detail));

Pizzas.addPizza("supreme");
```

5 observer works
```ts
class Subject {
  constructor() {
    this.observers = [];
  }
  addObserver(observer) {
    this.observers.push(observer);
  }
  removeObserver(observer) {
    const index = this.observers.indexOf(observer);
    if (index > -1) {
      this.observers.splice(index, 1);
    }
  }
  notify(data) {
    this.observers.forEach(observer => observer.update(data));
  }
}

class Observer {
  update(data) {
    console.log(data);
  }
}

const subject = new Subject();
const observer = new Observer();

subject.addObserver(observer);
subject.notify('Everyone gets pizzas!');
```
6 proxy works
```ts
const handler = {
  get: function(target, property) {
    console.log(`Getting property ${property}`);
    return target[property];
  },
  set: function(target, property, value) {
    console.log(`Setting property ${property} to ${value}`);
    target[property] = value;
    return true; // indicates that the setting has been done successfully
  }
};

const pizza = { name: 'Margherita', toppings: ['tomato sauce', 'mozzarella'] };
const proxiedPizza = new Proxy(pizza, handler);

console.log(proxiedPizza.name); // Outputs "Getting property name" and "Margherita"
proxiedPizza.name = 'Pepperoni'; // Outputs "Setting property name to Pepperoni"
```

7 object define property

```ts
const pizza = {

_name: 'Margherita', // Internal property

};

  

Object.defineProperty(pizza, 'name', {

get: function() {

console.log(`Getting property name`);

return this._name;

},

set: function(value) {

console.log(`Setting property name to ${value}`);

this._name = value;

}

});

  

// Example usage:

console.log(pizza.name); // Outputs "Getting property name" and "Margherita"

pizza.name = 'Pepperoni'; // Outputs "Setting property name to Pepperoni"

console.log(pizza.name); // Outputs "Getting property name" and "Margherita"
```


- async observers - add example init and run
```ts
class AsyncData {

constructor(initialData) {

this.data = initialData;

this.subscribers = [];

}

subscribe(callback) {

if (typeof callback !== 'function') {

throw new Error('Callback must be a function');

}

this.subscribers.push(callback);

}

async set(key, value) {

this.data[key] = value;

const updates = this.subscribers.map(async (callback) => {

await callback(key, value);

});

await Promise.allSettled(updates);

}

}

const callback1 = async (key, value) => {

console.log(`Callback 1: Key ${key} updated to ${value}`);

};

const callback2 = async (key, value) => {

console.log(`Callback 2: Key ${key} updated to ${value}`);

};

const asyncData = new AsyncData({ pizza: 'Margherita', drinks: 'Soda' });

asyncData.subscribe(callback1);

asyncData.subscribe(callback2);

console.log('Initial data:', asyncData.data);

asyncData.set('pizza', 'Pepperoni')

asyncData.set('pizza', 'Pineapple Bacon')

console.log(' data:', asyncData.data);

  

// asyncData.set('pizza', 'Pepperoni').then(() => {

// console.log('Updated data:', asyncData.data);

// });

// asyncData.set('drinks', 'Water').then(() => {

// console.log('Updated data:', asyncData.data);

// });
```

- observables works
	- notes: observable is a producer of data that can be observed by an observer, which is the reactor to data inputs or what makes the observable reactive
```ts
interface IObserver<T> {

next(value: T): void;

error(err: any): void;

complete(): void;

}

class Observable<T> {

private producer: (observer: IObserver<T>) => (() => void);

  

constructor(producer: (observer: IObserver<T>) => (() => void)) {

this.producer = producer;

}

subscribe(observer: IObserver<T>) {

if (typeof observer !== 'object' || observer === null) {

throw new Error('Observer must be an object with next, error, and complete methods');

}

if (typeof observer.next !== 'function') {

throw new Error('Observer must have a next method');

}

if (typeof observer.error !== 'function') {

throw new Error('Observer must have an error method');

}

if (typeof observer.complete !== 'function') {

throw new Error('Observer must have a complete method');

}

const unsubscribe = this.producer(observer);

return {

unsubscribe: () => {

if (unsubscribe && typeof unsubscribe === 'function') {

unsubscribe();

}

},

};

}

}

  

// Example usage

const producer = (observer: IObserver<string>) => {

observer.next('Pepperoni Pizza');

observer.next('Margherita Pizza');

observer.next('Hawaiian Pizza');

observer.complete();

return () => console.log('Unsubscribed');

};

const observable = new Observable<string>(producer);

const observer: IObserver<string> = {

next: (value) => console.log('Received:', value),

error: (error) => console.log('Error:', error),

complete: () => console.log('Complete!'),

};

const subscription = observable.subscribe(observer);
```

- clean code version of observable
```ts
interface IObserver {

next(value: string): void;

error(err: any): void;

complete(): void;

}

interface IProducer {

produce(observer: IObserver): () => void;

}

interface IObservable {

subscribe(): { unsubscribe: () => void };

}

class PizzaObserver implements IObserver {

next(value: string) {

console.log("Pizza:", value);

}

error(err: any) {

console.log("Pizza Error:", err);

}

complete() {

console.log("Pizza Complete!");

}

}

class DrinkObserver implements IObserver {

next(value: string) {

console.log("Drink:", value);

}

error(err: any) {

console.log("Drink Error:", err);

}

complete() {

console.log("Drink Complete!");

}

}

class PizzaProducer implements IProducer {

produce(observer: IObserver) {

observer.next("Pepperoni Pizza");

observer.next("Margherita Pizza");

observer.next("Hawaiian Pizza");

observer.complete();

return () => console.log("Pizza Unsubscribed");

}

}

class DrinkProducer implements IProducer {

produce(observer: IObserver) {

observer.next("Soda");

observer.next("Water");

observer.next("Juice");

observer.complete();

return () => console.log("Drink Unsubscribed");

}

}

class Observable implements IObservable {

private observer: IObserver;

private producer: IProducer;

constructor(observer: IObserver, producer: IProducer) {

this.observer = observer;

this.producer = producer;

}

subscribe() {

const unsubscribe = this.producer.produce(this.observer);

return {

unsubscribe: () => {

if (unsubscribe && typeof unsubscribe === "function") {

unsubscribe();

}

},

};

}

}

async function main() {

const pizzaObserver = new PizzaObserver();

const drinkObserver = new DrinkObserver();

const pizzaProducer = new PizzaProducer();

const drinkProducer = new DrinkProducer();

const pizzaObservable = new Observable(pizzaObserver, pizzaProducer);

const drinkObservable = new Observable(drinkObserver, drinkProducer);

pizzaObservable.subscribe();

drinkObservable.subscribe();

}

main();
```


### MutationObserver
- for chrome extensions watching for the page dom elements to update (add, update, remove) and fire event to the app to respond (eg if a phone number comes onto the page, convert it to a calleable phone)
- it's used for when you don't have control over or a 3rd party has control over the dom and can update the dom outside the control of your app and your app needs to respond

### IntersectionObserver
- window listens for in view and fires an event, which you can use to trigger an update
```ts
// when entry intersecting, add a class to the target
  const observer = new IntersectionObserver((entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          entry.target.classList.add("animate-in");
        }
      });
    });

    document.querySelectorAll(".MediaItem").forEach((el) => {
      observer.observe(el);
    });```

### Web Animations API
- animate the element on click with keyfames
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      nav {
        display: grid;
        grid-template-columns: repeat(2, 1fr);
        background-color: #f44336;
        padding: 10px 0;
        text-align: center;
        align-items: center;
      }
      nav a {
        color: white;
        text-decoration: none;
        font-size: 18px;
        font-weight: bold;
      }
      nav a:hover {
        text-decoration: underline;
      }
    </style>
  </head>
  <body>
    <nav></nav>
    <main></main>
    <div id="animatedElement" style="width: 100px; height: 100px"></div>
    <script type="module">
      import { html, render } from "https://esm.sh/lit-html@2.8.0";

      const pizzas = [
        { name: "Margherita", toppings: ["Tomato Sauce", "Mozzarella"] },
        {
          name: "Pepperoni",
          toppings: ["Tomato Sauce", "Mozzarella", "Pepperoni"],
        },
        // Add more pizzas here...
      ];

      const navTemplate = () => html`
        <nav>
          <a href="#" @click=${renderHome}>Home</a>
          <a href="#" @click=${renderPizzas}>Pizzas</a>
        </nav>
      `;

      const homeTemplate = () => html`
        <div>
          <h1>Welcome to Our Pizza Restaurant!</h1>
          <p>Experience the best pizzas in town.</p>
        </div>
      `;

      const pizzasTemplate = () => html`
        <div>
          <h1>Our Pizzas</h1>
          <ul>
            ${pizzas.map(
              (pizza) =>
                html`<li>${pizza.name} - ${pizza.toppings.join(", ")}</li>`
            )}
          </ul>
        </div>
      `;

      function renderHome() {
        render(homeTemplate(), document.querySelector("main"));
      }

      function renderPizzas() {
        render(pizzasTemplate(), document.querySelector("main"));
      }

      render(navTemplate(), document.querySelector("nav"));
      renderHome(); // Render the home page by default
    </script>
    <script>
      const el = document.getElementById("animatedElement");

      // Define the animation properties
      const animation = el.animate(
        [
          // Keyframes
          {
            transform: "scale(1)",
            backgroundColor: "blue",
            left: "50px",
            top: "50px",
          },
          {
            transform: "scale(1.5)",
            backgroundColor: "red",
            left: "200px",
            top: "200px",
          },
        ],
        {
          // Timing options
          duration: 1000,
          fill: "forwards",
        }
      );

      // Set the animation's playback rate to 0 to pause it
      animation.playbackRate = 0;

      // Add a click event listener to the element
      el.addEventListener("click", () => {
        // If the animation is paused, play it
        if (animation.playbackRate === 0) {
          animation.playbackRate = 1;
        } else {
          // If the animation is playing, reverse it
          animation.reverse();
        }
      });
    </script>
  </body>
</html>

<!-- 
fetcher is a component scoped ajax utility which can both pitch and catch requests without page transitions; it functions similar to the component hooks use[Loader/Action]Data for [re-]initialization & optimistic ui -->
```