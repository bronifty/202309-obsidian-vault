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

4 
```ts

```