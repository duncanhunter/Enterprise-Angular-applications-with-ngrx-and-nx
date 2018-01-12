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



