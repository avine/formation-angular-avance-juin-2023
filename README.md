## Lab 13: NgRx

In this lab you will use NgRx to store the customer details.
When the user proceeds to the checkout, you need to store the customer data.
Then, when they return to the basket page for a new purchase, the customer form should already be pre-filled.

- Add NgRx to your project:

```shell
ng add @ngrx/store
ng add @ngrx/effects
ng add @ngrx/store-devtools
ng add @ngrx/schematics
```

- Install the Chrome Extension: *Redux DevTools*

More infos: https://github.com/reduxjs/redux-devtools

- Create the basket feature:

```shell
ng generate feature shared/store/basket --flat=false --skip-tests
```

- In the `basket.actions.ts` add the `'Fill Customer'` action:

```ts
export const basketActions = createActionGroup({
  source: 'Basket',
  events: {
    'Fill Customer': props<{ customer: Customer }>(),
  }
});
```

- In the `basket.reducer.ts` handle the action you just created:

```ts
export interface State {
  customer: Customer | undefined;
}

export const reducer = createReducer(
  initialState,
  on(basketActions.fillCustomer, (state, { customer }): State => ({ ...state, customer })),
);
```

- In the `basket.selectors.ts` add a selector for the customer

<div class="pb"></div>

- Add the entry point `shared/store/index.ts`:

```ts
import * as fromBasket from './basket/basket.reducer';

export interface AppState {
  [fromBasket.basketFeatureKey]: fromBasket.State,
}

export const appReducers: ActionReducerMap<AppState> = {
  [fromBasket.basketFeatureKey]: fromBasket.reducer,
};
```

- Add the reducers in `app.config.ts`:

```ts
import { appReducers } from './shared/store';

export const appConfig: ApplicationConfig = {
  providers: [
    provideStore(appReducers),
  ],
};
```

- In the `basket-form.component.ts`:
  - In the `checkout()` method, dispatch the action:
    `this.#store.dispatch(basketActions.fillCustomer(...))`
  - In the `constructor()` method, select the filled customer (if defined):
    `this.#store.select(...).subscribe(...)`

Tip: to fill the form with the stored customer, use something like this:

```ts
this.#store
  // Select the customer from the Store
  .select(...)
  .pipe(
    first(),
    // Be sure the customer is not undefined
    filter((customer): customer is Customer => customer !== undefined)
  )
  .subscribe((customer) => {
    // Update the form value
    this.yourFormGroup.setValue(customer);
    this.yourFormGroup.updateValueAndValidity();
  });
```

<div class="pb"></div>

### Bonus

Migrate the basket items to the store.
This time, you will need to use Effets.

- In the `basket.actions.ts` add the `'Fetch'` and `'Fetch Success'` actions:

```ts
export const basketActions = createActionGroup({
  source: 'Basket',
  events: {
    'Fetch': emptyProps(),
    'Fetch Success': props<{ items: BasketItem[] }>(),
  }
});
```

- Handle the `'Fetch'` action in the `basket.effects.ts`
- Handle the `'Fetch Success'` action in the `basket.reducer.ts`

<div class="pb"></div>
