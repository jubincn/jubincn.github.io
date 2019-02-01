## Read note of [TypeScript Basics](https://angular-presentation.firebaseapp.com/typescript/)

### Use ```:``` to specify type information
function add(a: number, b: number) {
  return a + b;
}

### const
```const``` is introducted in ES6. Similar to var, but is block scoped(only exist inside of curly braced block) and can't be reassigned.

### ```interface```
TypeScript allow using interface to define more complex type.
```
interface Puppy {
  name: string;
  age: number;
};

const realPuppy: Puppy = {
  name: 'Pikachu',
  age: 1
};

const notRealPuppy: Puppy = {
  says: 'meow' // Error: this is clearly not a puppy
}
```

### Arrays
Arrays are defined as Array<Type> or Array[]
```
// define array as Array<Type>
const cats: Array<string> = ['Simba', 'Aslan'];
// Type[] does the same thing.
const cats2: string[] = ['Simba', 'Aslan'];

interface Cat {
  name: string,
  age: number
}

const betterCats: Array<Cat> = [
  {name: 'Simba', age: 22},
  {name: 'Aslan', age: 9999}
];
```

### ```class```
```class``` is used to group method and properties together.
```
export class Puppy {
  // This is a method.
  bark(){
    // That's how russian dogs talk.
    return 'Gav gav!!';
  }
}

// Now we can instantiate (create) it
var hotdog = new Puppy();
// And use its methods
console.log(hotdog.bark());
```

### constructor
There is a constructor method in class. It's run when class is instantiated and can take a parameter.
```
export class Puppy {
  constructor(public name: string){
    // Later we'll have code here
  }
  bark(){
    return 'Gav! my name is ' + this.name;
  }
}
```

### constructor parameter
Constructor parameter marked as ```public``` (or ```private```, ```protected```) become class properties.

### ```export```
```export``` is used to share information between files.

### ```filter```
```filter``` is an Array method and can be used to filter array items.
```
const numbers = [12,23,62,34,19,40,4,9];

console.log(numbers.filter(function(n: number){
  return n > 30;
}));

// Or use shorthand function notation.
// (Also called arrow function)
console.log(
  numbers.filter(n => n > 30)
);
```
