# Part 9 - nx state libs

#### 1. Add a beta ngrx schematics

```
npm i github:ngrx/schematics-builds --save-dev
```

Note: The new schematics from the offical ngrx team are in testing but can be good for now while we find out if the nx team swap to action creators [https://github.com/ngrx/platform/issues/674](https://github.com/ngrx/platform/issues/674)

#### 2. Add auth ngrx state

```
ng generate action auth -a=auth --collection @ngrx/schematics
```

* Delete the actions file from the nx code generated files at the path _**libs/auth/src/+state/auth.actions.ts**_

```
ng generate action Auth --collection @ngrx/schematics
```

* Swap out original for new nx style effect

_**libs/auth/src/+state/auth.actions.ts**_

```ts
import { Action } from '@ngrx/store';
import { User, Authenticate } from '@demo-app/data-models';

export enum AuthStateActionTypes {
Login = '[AuthState] Login',
LoginSuccess = '[AuthState] Login Success',
LoginFail = '[AuthState] Login Fail'
}

export class LoginAction implements Action {
readonly type = AuthStateActionTypes.Login;
constructor(public payload: Authenticate) {}
}

export class LoginSuccessAction implements Action {
readonly type = AuthStateActionTypes.LoginSuccess;
constructor(public payload: User) {}
}

export class LoginFailAction implements Action {
readonly type = AuthStateActionTypes.LoginFail;
constructor(public payload) {}
}

export type AuthStateActions =
LoginAction
| LoginFailAction
| LoginSuccessAction;
```

### 3. Add an effect

_**libs/auth/src/+state/auth.effects.ts**_

```ts
import { Injectable } from '@angular/core';
import { Effect, Actions } from '@ngrx/effects';
import { of } from 'rxjs/observable/of';
import 'rxjs/add/operator/switchMap';
import { AuthState } from './auth.interfaces';
import * as authActions from './auth.actions';
import { map, catchError, tap, mergeMap } from 'rxjs/operators';
import { AuthService } from './../services/auth.service';

@Injectable()
export class AuthEffects {
  @Effect()
  login = this.actions$
    .ofType(authActions.AuthStateActionTypes.Login)
    .pipe(
      mergeMap((action: authActions.LoginAction) =>
        this.authService
          .login(action.payload)
          .pipe(
            map(user => new authActions.LoginSuccessAction(user)),
            catchError(error => of(new authActions.LoginFailAction(error)))
          )
      )
    );

  constructor(private actions$: Actions, private authService: AuthService) {}
}
```

* Swap out original for new nx style effect

_**libs/auth/src/+state/auth.effects.ts**_

```ts
import { Injectable } from '@angular/core';
import { Effect, Actions } from '@ngrx/effects';
import { of } from 'rxjs/observable/of';
import 'rxjs/add/operator/switchMap';
import { AuthState } from './auth.interfaces';
import * as authActions from './auth.actions';
import { map, catchError, tap, mergeMap } from 'rxjs/operators';
import { AuthService } from './../services/auth.service';
import { DataPersistence } from '@nrwl/nx';

@Injectable()
export class AuthEffects {
  @Effect()
  login$ = this.dataPersistence.fetch(authActions.AuthStateActionTypes.Login, {
    run: (action: authActions.LoginAction, state: AuthState) => {
      return this.authService
        .login(action.payload)
        .pipe(map(user => new authActions.LoginSuccessAction(user)));
    },

    onError: (action: authActions.LoginAction, error) => {
      return of(new authActions.LoginFailAction(error));
    }
  });

  constructor(
    private actions: Actions,
    private dataPersistence: DataPersistence<AuthState>,
    private authService: AuthService
  ) {}
}
```

#### 3. Add default state and interface

* Update state interface

_**libs/auth/src/+state/auth.interfaces.ts**_

```ts
import { User } from "@demo-app/data-models";

export interface Auth {
  user: User,
  loading: boolean
}

export interface AuthState {
  readonly auth: Auth;
}
```

* Update state init

_**libs/auth/src/+state/auth.init.ts**_

```ts
import { Auth } from './auth.interfaces';

export const authInitialState: Auth = {
  user: null,
  loading: false
};
```

#### 4. Add reducer code

_**libs/auth/src/+state/auth.reducer.ts**_

```ts
import { Auth } from './auth.interfaces';
import * as authActions from './auth.actions';

export function authReducer(
  state: Auth,
  action: authActions.AuthStateActions
): Auth {
  switch (action.type) {
    case authActions.AuthStateActionTypes.Login: {
      return {
        ...state,
        loading: true
      };
    }
    case authActions.AuthStateActionTypes.LoginSuccess: {
      return {
        ...state,
        loading: false,
        user: action.payload
      };
    }
    default: {
      return state;
    }
  }
}
```

#### 5. Update LoginComponent to dispatch action

_**libs/auth/src/containers/login/login.component.ts**_

```ts
import { Component, OnInit } from '@angular/core';
import { FormGroup, FormControl, Validators } from '@angular/forms';import { User, Authenticate } from '@demo-app/data-models';
import { Store } from '@ngrx/store';
import { AuthState } from './../../+state/auth.interfaces';
import * as authActions from './../../+state/auth.actions';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss']
})
export class LoginComponent implements OnInit {
  constructor(private store: Store<AuthState>) {}

  ngOnInit() {}

  login(authenticate: Authenticate): void {
    this.store.dispatch(new authActions.LoginAction(authenticate));
  }
 }
}
```

#### 6. Add route change action on success

* Add a new action to navigate

_**libs/auth/src/+state/auth.actions.ts**_

```ts
import { Action } from '@ngrx/store';
import { User, Authenticate } from '@demo-app/data-models';

export enum AuthStateActionTypes {
Login = '[AuthState] Login',
LoginSuccess = '[AuthState] Login Success',
LoginFail = '[AuthState] Login Fail',
NavigateToProfile = '[AuthState] Navigate To Profile'
}

export class LoginAction implements Action {
readonly type = AuthStateActionTypes.Login;
constructor(public payload: Authenticate) {}
}

export class LoginSuccessAction implements Action {
readonly type = AuthStateActionTypes.LoginSuccess;
constructor(public payload: User) {}
}

export class LoginFailAction implements Action {
readonly type = AuthStateActionTypes.LoginFail;
constructor(public payload) {}
}

export class NavigateToProfileAction implements Action {
readonly type = AuthStateActionTypes.NavigateToProfile;
constructor(public payload:number) {}
}

export type AuthStateActions =
LoginAction
| LoginFailAction
| LoginSuccessAction
| NavigateToProfileAction;
```

* Add new Effect action to navigate

```ts
import { Injectable } from '@angular/core';
import { Effect, Actions } from '@ngrx/effects';
import { of } from 'rxjs/observable/of';
import 'rxjs/add/operator/switchMap';
import { AuthState } from './auth.interfaces';
import * as authActions from './auth.actions';
import { map, catchError, tap, mergeMap } from 'rxjs/operators';
import { AuthService } from './../services/auth.service';
import { DataPersistence } from '@nrwl/nx';
import { Router } from '@angular/router';
import { User } from '@demo-app/data-models';

@Injectable()
export class AuthEffects {
  @Effect()
  login$ = this.dataPersistence.fetch(authActions.AuthStateActionTypes.Login, {
    run: (action: authActions.LoginAction, state: AuthState) => {
      return this.authService
        .login(action.payload)
        .pipe(
          mergeMap((user: User) => [
            new authActions.LoginSuccessAction(user),
            new authActions.NavigateToProfileAction(user.id)
          ])
        );
    },

    onError: (action: authActions.LoginAction, error) => {
      return of(new authActions.LoginFailAction(error));
    }
  });

  @Effect({ dispatch: false })
  navigateToProfile = this.actions
    .ofType(authActions.AuthStateActionTypes.NavigateToProfile)
    .pipe(
      map((action: authActions.NavigateToProfileAction) =>
        this.router.navigate([`/user-profile/${action.payload}`])
      )
    );

  constructor(
    private actions: Actions,
    private dataPersistence: DataPersistence<AuthState>,
    private authService: AuthService,
    private router: Router
  ) {}
}
```

#### 



