# Part 9 - nx state libs

```
npm i github:ngrx/schematics-builds --save-dev
```

```
https://github.com/ngrx/platform/issues/674
```

```
ng generate action Auth --collection @ngrx/schematics
```

```
ng g lib auth-state
```

```
ng g ngrx auth-state --module=libs/auth-state/src/auth-state.module.ts
```

```ts
import { Action } from '@ngrx/store';
import { User } from '@demo-app/data-models';

export enum AuthStateActionTypes {
  Login = '[AuthState] Login',
  LoginSuccess = '[AuthState] Login Success',
  LoginFail = '[AuthState] Login Fail'
}

export class LoginAction implements Action {
  readonly type = AuthStateActionTypes.Login;
  constructor(public payload: User) {}
}

export class LoginSuccessAction implements Action {
  readonly type = AuthStateActionTypes.LoginSuccess;
  constructor(public payload) {}
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
    .ofType(authActions.AuthActionTypes.Login)
    .pipe(
      mergeMap((action: authActions.LoginAction) =>
        this.authService
          .login(action.payload.username, action.payload.password)
          .pipe(
            tap(console.log),
            map(user => new authActions.LoginSuccessAction(user)),
            catchError(error => of(new authActions.LoginFailAction(error)))
          )
      )
    );

  constructor(private actions$: Actions, private authService: AuthService) {}
}
```

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
  login$ = this.dataPersistence.fetch(authActions.AuthActionTypes.Login, {
    run: (action: authActions.LoginAction, state: AuthState) => {
      return this.authService
        .login(action.payload.username, action.payload.password)
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

```ts
import { Component, OnInit } from '@angular/core';
import { FormGroup, FormControl, Validators } from '@angular/forms';
import { User } from '@demo-app/data-models';
import { Store } from '@ngrx/store';
import { AuthState } from './../../+state/auth.interfaces';
import * as authActions from './../../+state/auth.actions';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss']
})
export class LoginComponent implements OnInit {
  error: string;

  constructor(private store: Store<AuthState>) {}

  loginForm = new FormGroup({
    username: new FormControl('', [Validators.required]),
    password: new FormControl('', [Validators.required])
  });

  ngOnInit() {}

  login() {
    this.store.dispatch(
      new authActions.LoginAction({
        username: this.loginForm.value.username,
        password: this.loginForm.value.password
      })
    );
  }
}
```

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
  login$ = this.dataPersistence.fetch(authActions.AuthActionTypes.Login, {
    run: (action: authActions.LoginAction, state: AuthState) => {
      return this.authService
        .login(action.payload.username, action.payload.password)
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
    .ofType(authActions.AuthActionTypes.NavigateToProfile)
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

```ts
import { Action } from '@ngrx/store';
import { User } from '@demo-app/data-models';

export enum AuthActionTypes {
  Login = '[Auth] Login',
  LoginSuccess = '[Auth] Login Success',
  LoginFail = '[Auth] Login Fail',
  NavigateToProfile = '[Auth] Navigate To Profile'
}

export class LoginAction implements Action {
  readonly type = AuthActionTypes.Login;
  constructor(public payload) {}
}

export class LoginSuccessAction implements Action {
  readonly type = AuthActionTypes.LoginSuccess;
  constructor(public payload: User) {}
}

export class LoginFailAction implements Action {
  readonly type = AuthActionTypes.LoginFail;
  constructor(public payload) {}
}

export class NavigateToProfileAction implements Action {
  readonly type = AuthActionTypes.NavigateToProfile;
  constructor(public payload:number) {}
}

export type AuthActions = 
  LoginAction
  | LoginFailAction
  | LoginSuccessAction
  | NavigateToProfileAction;
```



