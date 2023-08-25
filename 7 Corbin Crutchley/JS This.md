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