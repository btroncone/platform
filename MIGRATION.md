# Migration Guide

This guide covers the changes between the ngrx projects migrating from V1.x/2.x to V4.

[@ngrx/core](#ngrxcore)  
[@ngrx/store](#ngrxstore)  
[@ngrx/effects](#ngrxeffects)  
[@ngrx/router-store](#ngrxrouter-store)  
[@ngrx/store-devtools](#ngrxstore-devtools)

## @ngrx/core
@ngrx/core is no longer needed, and can conflict with @ngrx/store. You should remove it from your project.

BEFORE:

```ts
import { compose } from '@ngrx/core/compose';
```

AFTER:

```ts
import { compose } from '@ngrx/store';
```

## @ngrx/store

Previously to be AOT compatible, it was required to pass a function to the `provideStore` method to compose the reducers into one root reducer. The `initialState`
was also provided to the method as an object in the second argument.

BEFORE:


`reducers/index.ts`
```ts
const reducers = {
  auth: fromAuth.reducer,
  layout: fromLayout.reducer
};

const rootReducer = combineReducers(reducers);

export function reducer(state: any, action: any) {
  return rootReducer(state, action);
}
```

`app.module.ts`
```ts
import { StoreModule } from '@ngrx/store';
import { reducer } from './reducers';

@NgModule({
  imports: [
    StoreModule.provideStore(reducer, {
      auth: {
        loggedIn: true
      }
    })
  ]
})
export class AppModule {}
```

This has been simplified to only require a map of reducers that
will be composed together by the library. The second argument is a configuration
object where you can provide the `initialState`.

AFTER:

`reducers/index.ts`

```ts
export const reducers = {
  auth: fromAuth.reducer,
  layout: fromLayout.reducer
};
```

`app.module.ts`
```ts
import { StoreModule } from '@ngrx/store';
import { reducers } from './reducers';

@NgModule({
  imports: [
    StoreModule.forRoot(reducers, {
      initialState: {
        auth: {
          loggedIn: true
        }
      }
    })
  ]
})
export class AppModule {}
```

## @ngrx/effects

### Registering Effects

BEFORE:

`app.module.ts`
```ts
@NgModule({
  imports: [
    EffectsModule.run(SourceA),
    EffectsModule.run(SourceB),
  ]
})
export class AppModule { }
```

AFTER:

`app.module.ts`
```ts
@NgModule({
  imports: [
    EffectsModule.forRoot([
      SourceA,
      SourceB,
      SourceC,
    ])
  ]
})
export class AppModule { }
```

`feature.module.ts`
```ts
@NgModule({
  imports: [
    EffectsModule.forFeature([
      FeatureSourceA,
      FeatureSourceB,
      FeatureSourceC,
    ])
  ]
})
export class FeatureModule { }
```

### Testing Effects

BEFORE:
```ts
import { EffectsTestingModule, EffectsRunner } from '@ngrx/effects/testing';
import { MyEffects } from './my-effects';

describe('My Effects', () => {
  let effects: MyEffects;
  let runner: EffectsRunner;
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [
        EffectsTestingModule
      ],
      providers: [
        MyEffects,
        // other providers
      ],
    });

    effects = TestBed.get(MyEffects);
    runner = TestBed.get(EffectsRunner);
  });

  it('should work', () => {
    runner.queue(SomeAction);

    effects.someSource$.subscribe(result => {
      expect(result).toBe(AnotherAction);
    });
  });
});
```

AFTER:
```ts
import { TestBed } from '@angular/core/testing';
import { provideMockActions } from '@ngrx/effects/testing';
import { hot, cold } from 'jasmine-marbles';
import { MyEffects } from './my-effects';
import { ReplaySubject } from 'rxjs/ReplaySubject';

describe('My Effects', () => {
  let effects: MyEffects;
  let actions: Observable<any>;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [
        MyEffects,
        provideMockActions(() => actions),
        // other providers
      ],
    });

    effects = TestBed.get(MyEffects);
  });

  it('should work', () => {
    actions = hot('--a-', { a: SomeAction, ... });

    const expected = cold('--b', { b: AnotherAction });

    expect(effects.someSource$).toBeObservable(expected);
  });

  it('should work also', () => {
    actions = new ReplaySubject(1);

    actions.next(SomeAction);

    effects.someSource$.subscribe(result => {
      expect(result).toBe(AnotherAction);
    });
  });
});
```

## @ngrx/router-store

BEFORE:

`reducers/index.ts`
```ts
import * as fromRouter from '@ngrx/router-store';

export interface State {
  router: fromRouter.RouterState;
}

const reducers = {
  router: fromRouter.routerReducer
};

const rootReducer = combineReducers(reducers);

export function reducer(state: any, action: any) {
  return rootReducer(state, action);
}
```

`app.module.ts`
```ts
import { RouterModule } from '@angular/router';
import { RouterStoreModule } from '@ngrx/router-store';
import { reducer } from './reducers';

@NgModule({
  imports: [
    StoreModule.provideStore(reducer),
    RouterModule.forRoot([
      // some routes
    ])
    RouterStoreModule.connectRouter()
  ]
})
export class AppModule {}
```

AFTER:

`reducers/index.ts`
```ts
import * as fromRouter from '@ngrx/router-store';

export interface State {
  routerReducer: fromRouter.RouterReducerState;
}

export const reducers = {
  routerReducer: fromRouter.routerReducer
};
```

`app.module.ts`
```ts
import { StoreRouterConnectingModule } from '@ngrx/router-store';
import { reducers } from './reducers';

@NgModule({
  imports: [
    StoreModule.forRoot(reducers),
    RouterModule.forRoot([
      // some routes
    ]),
    StoreRouterConnectingModule
  ]
})
export class AppModule {}
```

## @ngrx/store-devtools

BEFORE:

`app.module.ts`
```ts
import { StoreDevtoolsModule } from '@ngrx/store-devtools';

@NgModule({
  imports: [
    StoreDevtoolsModule.instrumentStore({ maxAge: 50 }),
    // OR
    StoreDevtoolsModule.instrumentOnlyWithExtension({
      maxAge: 50
    })
  ]
})
export class AppModule {}
```

AFTER:

`app.module.ts`
```ts
import { StoreDevtoolsModule } from '@ngrx/store-devtools';
import { environment } from '../environments/environment'; // Angular CLI environment

@NgModule({
  imports: [
    !environment.prod ? StoreDevtoolsModule.instrument({ maxAge: 50 }) : []
  ]
})
export class AppModule {}
```
