[unicorn utterances js this](https://unicorn-utterances.com/posts/javascript-bind-usage#bind)

```ts
class Car {
    wheels = 4;
    
    honk() {
        console.log("Beep beep!");
    }
}
// `fordCar` is an "instance" of Car
const fordCar = new Car();
console.log(fordCar.wheels); // 4
fordCar.honk();
```

```ts
class Car {
  gasTank = 12;
  constructor(mpg = 30) {
    this.mpg = mpg;
  }
  drive(miles = 1) {
    this.gasTank -= miles / this.mpg;
  }
}
const fordCar = new Car(20);
console.log(fordCar.gasTank); // 12
fordCar.drive(30);
console.log(fordCar.gasTank); // 10.5
```

****

```ts
class Cup {
  contents = "water";
  consume() {
    console.log("You drink the ", this.contents, ". Hydrating!");
  }
}
class Bowl {
  contents = "chili";
  consume() {
    console.log("You eat the ", this.contents, ". Spicy!");
  }
}
const cup = new Cup();
const bowl = new Bowl();
// cup.consume();
// bowl.consume();
cup.consume = bowl.consume; // copies the bowl's static method, but not its this context, so bowl consume called on cup's this context 
cup.consume(); // You eat the water Spicy

```

```ts
class Cup {
  contents = "water";
  consume() {
    console.log("You drink the ", this.contents, ". Hydrating!");
  }
}
class Bowl {
  contents = "chili";
  consume() {
    console.log("You eat the ", this.contents, ". Spicy!");
  }
}
const cup = new Cup();
const bowl = new Bowl();
// cup.consume();
// bowl.consume();
cup.consume = bowl.consume;
cup.consume();
cup.consume = bowl.consume.bind(bowl); // binds the bowl's consume method to the bowl's this context
cup.consume(); // You eat the  chili . Spicy!
```

