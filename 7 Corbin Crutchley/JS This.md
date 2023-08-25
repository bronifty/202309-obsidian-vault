[unicorn utterances js this](https://unicorn-utterances.com/posts/javascript-bind-usage#bind)

Summary:
- only regular functions (not arrow) & classes can have their own 'this' context
- in object.method() syntax 'this' refers to the caller (object)
- an arrow function can refer to 'this' (in strict mode es6) only if it's wrapped in a class / object / regular function closure
- with that in mind, arrow function this refers to how it was defined (its provenance) whereas regular function object and class this refers to who is calling it (where it is now)


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

### Bind Call and Apply
```ts
// bind works by binding the definition at assignment time
cup.consume = bowl.consume.bind(bowl); // binds the bowl's consume method to the bowl's this context
cup.consume(); // You eat the  chili . Spicy!

// call and apply work by binding the definition at call time (apply takes an array after this whereas call takes individual csv)
cup.consume.call(bowl); // You eat the  chili . Spicy!
cup.consume.apply(bowl); // You eat the  chili . Spicy!
```

### Arrow Function This Lexical Scope
```ts
class Cup {
  contents = "water";
  consume = () => {
    console.log("You drink the ", this.contents, ". Hydrating!");
  };
}
class Bowl {
  contents = "chili";
  // arrow function this is lexical scope (where its defined ie context is the Bowl class)
  consume = () => {
    console.log("You eat the ", this.contents, ". Spicy!");
  };
}
const cup = new Cup();
const bowl = new Bowl();
cup.consume = bowl.consume; // cup's consume method is reassigned to bowl's consume method which was defined in lexical scope as an arrow function, meaning its this will always be the context where it was defined (bowl); it will never shift to the caller (cup)
cup.consume(); // You eat the chili. Spicy!
```

- even bind can't change the context of this with an arrow function definition because it's lexically scoped (scoped to where it's defined)
```ts
class Cup {
  contents = "water";
  consume = () => {
    console.log("You drink the ", this.contents, ". Hydrating!");
  };
}
class Bowl {
  contents = "chili";
  // arrow function this is lexical scope (where its defined ie context is the Bowl class)
  consume = () => {
    console.log("You eat the ", this.contents, ". Spicy!");
  };
}
const cup = new Cup();
const bowl = new Bowl();
cup.consume = bowl.consume; // cup's consume method is reassigned to bowl's consume method which was defined in lexical scope as an arrow function, meaning its this will always be the context where it was defined (bowl); it will never shift to the caller (cup)
cup.consume(); // You eat the chili. Spicy!
cup.consume = bowl.consume.bind(cup); // doesn't matter because consume is lexically scoped
cup.consume(); // You eat the chili. Spicy!
// Note: cup.consume = bowl.consume is the same as saying cup.consume = bowl.consume.bind(cup) if it is not an arrow function because the caller (cup) is the context whether you bind it or run it dynamically
```

- change from arrow to regular function to unbind the method from its lexical scope
```ts
class Cup {
  contents = "water";
  // arrow function lexically scoped (Cup's consume method will always refer to Cup's this.contents of 'water')
  consume = () => {
    console.log("You drink the ", this.contents, ". Hydrating!");
  };
}
class Bowl {
  contents = "chili";
  // change from arrow to regular function to unbind the method from lexical scope (Bowl's consume method will refer to the caller's this.contents)
  consume() {
    console.log("You eat the ", this.contents, ". Spicy!");
  }
}
const cup = new Cup();
const bowl = new Bowl();
cup.consume = bowl.consume;
cup.consume(); // You eat the water. Spicy!
// reset with new object
const cup2 = new Cup();
const bowl2 = new Bowl();
bowl2.consume = cup2.consume;
bowl2.consume(); // You drink the water . Hydrating!


```


