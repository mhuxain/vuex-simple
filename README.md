# vuex-simple

A simpler way to write your Vuex store in Typescript

## Usage

1. Install module:

`npm install vuex-simple --save`

2. Install reflect-metadata package:

`npm install reflect-metadata --save`

and import it somewhere in the global place of your app before any service declaration or import (for example in app.ts):

`import "reflect-metadata";`

3. Enabled following settings in tsconfig.json:

```json
"emitDecoratorMetadata": true,
"experimentalDecorators": true,
```

**Note:** *vuex-simple* uses the Container from [typedi](http://github.com/pleerock/typedi).


## Example

#### Module

```ts
// store/modules/counter.ts

import { Mutation, Module, Getter, State } from 'vuex-simple';

@Module('counter')
class CounterModule {

  @State()
  public counter: number = 0;

  // A simple getter that is cached and made reactive by vuex
  @Getter()
  public get myGetter() {
    console.log('my getter!');
    return 42;
  }

  // A simple mutation function
  // You can only modify your state in here, or Vuex will give you an error
  @Mutation()
  public increment() {
    this.counter++;
  }

  @Mutation()
  public incrementBy(nb: number) {
    this.counter += nb;
  }

  public async asyncIncrement() {
    await new Promise(r => setTimeout(r, 500));
    // simply call mutation function like you would any other function
    this.increment();
  }
}
```

#### Store

```ts
// store/index.ts

import Vue from 'vue';

import VuexSimple, { getStoreBuilder } from 'vuex-simple';

import CounterModule from './modules/counter';

Vue.use(VuexSimple);

const storeBuilder = getStoreBuilder();
storeBuilder.loadModules([
  CounterModule
]);
const store = storeBuilder.create();

export default store;

```

#### Usage

```ts
// In your vue component

import { Container, Inject } from 'vuex-simple';
import CounterModule from '@/store/modules/counter';

@Component
export default class MyComponent extends Vue {

  @Inject()
  public counterModule!: CounterModule;

  public get readState() {
    // access state like a property
    return this.counterModule.counter;
  }

  public get readGetter() {
    // access getter like a property
    return this.counterModule.myGetter;
  }

  public commitIncrement() {
    // call mutation like a function
    this.counterModule.increment();
  }

  public commitIncrementBy(number: id) {
    // call with parameter / payload
    this.counterModule.incrementBy(10);
  }

  public callAction() {
    counterModule.asyncIncrement();
  }

}

// Outside of a Vue component, you can also use Container to access the module
const counterModule = Container.get(CounterModule);


```

## Features

#### Module

To create a module, we declare a new class and decorate it with `@Module(namespace)`

#### Inject

Inject another module or service in your module with `@Inject()`. This feature will enable you to easily split up your code across multiple files in a logic way.

#### State

To tell the module which properties of the class will compose the state of the vuex module, we need to decorate those properties with `@State()`

#### Getter

To add a getter, we simply write a normal getter and add a `@Getter()` decorator to it.

#### Mutation

To add a mutation, we simply write a normal function and add a `@Mutation()` decorator to it. For now, mutations can only have at most 1 parameter.

#### Action

To add an action, we simply write a normal function and add a `@Action()` decorator to it. As for mutations, actions can, for now, only have at most 1 parameter.

**Note on actions:** vuex use actions to do their async stuff, but we don't really need an action for that, a simple function that can call our mutations is all we need, as shown above.</br>
So for now `@Action` is still included, but it may happen that it is removed in later versions.

#### How to setup your store

1. Use `Vue.use(VuexSimple)`

2. Get a StoreBuilder instance.

```ts
// 1. Use getStoreBuilder()
const storeBuilder = getStoreBuilder();

// 2. Create a new instance yourself
const storeBuilder = new StoreBuilder();

```

**Note:** `getStoreBuilder()` always returns the same instance

3. (optional) Initialize your store options.

```ts

// 1. When using getStoreBuilder()
const storeBuilder = getStoreBuilder();
storeBuilder.initialize({
  modules: {},
  state: {},
  strict: false
})

// 2. Pass directly in the StoreBuilder constructor
const storeBuilder = new StoreBuilder({
  modules: {},
  state: {},
  strict: false
});

```

4. (optional) You can also use a custom container instance.

```ts

const myContainer = Container.of('myContainer');
const storeBuilder = getStoreBuilder();

// 1. Using the initialize function or in the StoreBuilder constructor
storeBuilder.initialize({
  container: myContainer
})

// 2. Using the 'useContainer' function
storeBuilder.useContainer(myContainer);

```

**Note:** Be careful to set the container before you load your modules!

5. Use the StoreBuilder to load your modules

```ts
const storeBuilder = getStoreBuilder();
storeBuilder.loadModules([
  CounterModule
]);
```

6. We finish by creating the store with `storeBuilder.create()`


## FAQ

**How to split up your modules:**</br>
There are different ways to split up your modules:
1. Do all the heavy lifting (like API requests and such) in other files or services and inject them in your module
2. Split up your modules into smaller modules. If necessary, you can also inject other modules into your main module.

In the case the 2 solutions above don't work out for you, you can also use inheritance to split up your files, though it isn't the most recommended way:

```ts
class CounterState {
  @State()
  public counter: number = 10;
}

class CounterGetters extends CounterState {
  @Getter()
  public get counterPlusHundred() {
    return this.counter + 100;
  }
}

class CounterMutations extends CounterGetters {
  @Mutation()
  public increment() {
    this.counter++;
  }
}

class CounterActions extends CounterMutations {
  public async incrementAsync() {
    await new Promise(r => setTimeout(r, 500));
    this.increment();
  }
}

@Module('counter')
class CounterModule extends CounterActions {}
```

## Contributors

If you are interested and want to help out, don't hesitate to contact me or to create a pull request with your fixes / features.

The project now also contains samples that you can use to directly test out your features during development.

1. Clone the repository

2. Install dependencies

`npm install`

3. Launch samples

`npm run serve`

4. Launch unit tests situated in *./tests*. The unit tests are written in Jest.

`npm run test:unit`


## Bugs

The core features presented should be stable.
But as this module is still in development, it will probably still have some bugs here and there.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

- Inspired in some parts by [vuex-type-safety](https://github.com/christopherkiss/vuex-type-safety)
