# Part 12 - Entity state adapter

```
ng g guard guards/auth-admin/auth-admin -a=auth
```

```ts
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot, Router } from '@angular/router';
import { Observable } from 'rxjs/Observable';
import { AuthService } from './../../services/auth.service';
import { AuthState, getUser } from '@demo-app/auth';
import { Store } from '@ngrx/store';
import { map } from 'rxjs/operators';
import { User } from '@demo-app/data-models';

@Injectable()
export class AuthAdminGuard implements CanActivate {

  constructor(
    private router: Router,
    private store: Store<AuthState>) {}

  canActivate(
    next: ActivatedRouteSnapshot,
    state: RouterStateSnapshot): Observable<boolean> {
      return this.store.select(getUser).pipe(
        map((user: User) => {          
          if(user && user.role === 'admin') {
            return true;
          } else {
            this.router.navigate([`/auth/login`]);
            return false;
          }
        })
      )
  }
}
```

Make a users lib in an admin directory

```
ng g lib users --directory=admin-portal --routing --lazy --parent-module=apps/admin-portal/src/app/app.module.ts
```

```
ng g c containers/user-list -a=admin-portal/users
```

```
ng g ngrx users --module=libs/admin-portal/users/src/users.module.ts
```

```ts
@NgModule({
  imports: [
    CommonModule,
    RouterModule.forChild([
      { path: '', pathMatch: 'full', component: UserListComponent }
    ]),
    StoreModule.forFeature('users', usersReducer, {
      initialState: usersInitialState
    }),
    EffectsModule.forFeature([UsersEffects])
  ],
  declarations: [UserListComponent],
  providers: [UsersEffects]
})
export class UsersModule {}
```

```ts
 RouterModule.forRoot([
  { path: '', pathMatch: 'full', redirectTo: 'user-profile' },
  { path: 'auth', children: authRoutes },
  {
    path: 'user-profile',
    loadChildren: '@demo-app/user-profile#UserProfileModule',
    canActivate: [AuthGuard]
  },
  {
    path: 'users',
    loadChildren: '@demo-app/admin-portal/users#UsersModule',
    canActivate: [AuthAdminGuard]
  }
]),
```

```ts
import { NgModule, ModuleWithProviders } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Route } from '@angular/router';
import { LoginComponent } from './container/login/login.component';
import { HttpClientModule, HTTP_INTERCEPTORS } from '@angular/common/http';
import { AuthService } from './services/auth.service';
import { MaterialModule } from '@demo-app/material';
import { ReactiveFormsModule } from '@angular/forms';
import { AuthGuard } from './guards/auth.guard';
import { AuthInterceptor } from './interceptors/auth.interceptor';
import { StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';
import { authReducer } from './+state/auth.reducer';
import { authInitialState } from './+state/auth.init';
import { AuthEffects } from './+state/auth.effects';
import { AuthAdminGuard } from './guards/auth-admin/auth-admin.guard';

export const authRoutes: Route[] = [
  { path: 'login', component: LoginComponent }
];
const COMPONENTS = [LoginComponent];

@NgModule({
  imports: [
    CommonModule,
    RouterModule,
    HttpClientModule,
    MaterialModule,
    ReactiveFormsModule,
    StoreModule.forFeature('auth', authReducer, {initialState: authInitialState}),
    EffectsModule.forFeature([AuthEffects])
  ],
  declarations: [COMPONENTS],
  exports: [COMPONENTS],
  providers: [
    AuthService,
    AuthGuard,
    AuthAdminGuard,
    AuthEffects
  ]
})
export class AuthModule {
  static forRoot(): ModuleWithProviders {
    return {
      ngModule: AuthModule,
      providers: [
        AuthService,
        AuthGuard,
        AuthAdminGuard,
        {
          provide: HTTP_INTERCEPTORS,
          useClass: AuthInterceptor,
          multi: true
        }
      ]
    };
  }
}
```

```html
<mat-toolbar color="primary" fxLayout="row">
  <span>Admin Portal</span>
  <div class="right-nav">
    <span>{{(user$ | async)?.username}}</span>
    <button mat-button *ngIf="(user$ | async)?.role === 'admin'" [routerLink]="['/users']">Users</button>
  </div>
</mat-toolbar>
<ng-content></ng-content>
```

```
ng g service services/users -a=admin-portal/users
```

```ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable()
export class UsersService {

  constructor(private httpClient: HttpClient) { }

  getUsers() {
    return this.httpClient.get(`http://localhost:3000/users`);
  }

}
```

change directory to +state file

```ts
ng g action users --collection @ngrx/schematics
```

```ts
import { Action } from '@ngrx/store';
import { User } from '@demo-app/data-models';

export enum UsersActionTypes {
  LoadUsers = '[Users] Load',
  LoadUsersSuccess = '[Users] Load Sucess',
  LoadUsersFail = '[Users] Load Fail'
}

export class LoadUsersAction implements Action {
  readonly type = UsersActionTypes.LoadUsers;
}

export class LoadUsersSuccessAction implements Action {
  readonly type = UsersActionTypes.LoadUsersSuccess;
  constructor(private payload: User[]) {}
}

export class LoadUsersFailAction implements Action {
  readonly type = UsersActionTypes.LoadUsersSuccess;
  constructor(private payload: any) {}
}

export type UsersActions =
  | LoadUsersAction
  | LoadUsersSuccessAction
  | LoadUsersFailAction;
```

```ts
import { Injectable } from '@angular/core';
import { Effect, Actions } from '@ngrx/effects';
import { DataPersistence } from '@nrwl/nx';
import { of } from 'rxjs/observable/of';
import 'rxjs/add/operator/switchMap';
import { UsersState } from './users.interfaces';
import * as usersActions from './users.actions';
import { UsersService } from './../services/users.service';
import { map } from 'rxjs/operators';
import { User } from '@demo-app/data-models';

@Injectable()
export class UsersEffects {
  @Effect()
  loadData = this.dataPersistence.fetch(
    usersActions.UsersActionTypes.LoadUsers,
    {
      run: (action: usersActions.LoadUsersAction, state: UsersState) => {
        return this.usersService
          .getUsers()
          .pipe(
            map(
              (users: User[]) => new usersActions.LoadUsersSuccessAction(users)
            )
          );
      },

      onError: (action: usersActions.LoadUsersAction, error) =>
        new usersActions.LoadUsersFailAction(error)
    }
  );

  constructor(
    private actions: Actions,
    private dataPersistence: DataPersistence<UsersState>,
    private usersService: UsersService
  ) {}
}
```

```
npm install @ngrx/entity
```

_**libs/admin-portal/users/+state/interfaces.init.ts**_

```ts
import { EntityState } from '@ngrx/entity';
import { User } from '@demo-app/data-models';

export interface Users extends EntityState<User> {
  selectedUserId: number;
  loading: boolean;
}

export interface UsersState {
  readonly users: Users;
}
```

_**libs/admin-portal/users/+state/users.init.ts**_

```ts
import { Users } from './users.interfaces';
import { EntityState, EntityAdapter, createEntityAdapter } from '@ngrx/entity';
import { User } from '@demo-app/data-models';

export const adapter: EntityAdapter<User> = createEntityAdapter<User>({});

export const usersInitialState: Users = adapter.getInitialState({
  selectedUserId: null,
  loading: false
});
```

_**libs/admin-portal/users/+state/users.reducer.ts**_

```ts
import { Users } from './users.interfaces';
import * as usersActions from './users.actions';
import { adapter } from './users.init';

export function usersReducer(
  state: Users,
  action: usersActions.UsersActions
): Users {
  switch (action.type) {
    case usersActions.UsersActionTypes.LoadUsers: {
      return { ...state, loading: true };
    }

    case usersActions.UsersActionTypes.LoadUsersSuccess: {
      return adapter.addMany(action.payload, { ...state, loading: false });
    }

    default: {
      return state;
    }
  }
}

export const getSelectedUserId = (state: Users) => state.selectedUserId;

export const {
  // select the array of user ids
  selectIds: selectUserIds,

  // select the dictionary of user entities
  selectEntities: selectUserEntities,

  // select the array of users
  selectAll: selectAllUsers,

  // select the total user count
  selectTotal: selectUserTotal
} = adapter.getSelectors();
```

_**libs/admin-portal/users/containers/user-list.component.html**_

```
{{users$ | async | json}}
```



