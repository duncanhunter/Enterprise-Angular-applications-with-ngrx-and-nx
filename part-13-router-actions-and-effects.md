# Part 13 - Router actions and effects

_**libs/admin-portal/users/containers/user-list.component.html**_

```ts
{{users$ | async | json}}
```

#### 1 .Add a new presentational components for users

```
ng g c components/users-table -a=admin-portal/users
```

* Add a toolbar module

```
ng g c components/users-table-toolbar -a=admin-portal/users
```

* Add a toolbar with select field

```html
<mat-toolbar>
  <mat-form-field >
    <mat-select [value]="selectedCountry" (selectionChange)="filter($event.value)" >
      <mat-option>None</mat-option>
      <mat-option value="australia" >Australia</mat-option>
      <mat-option value="england">England</mat-option>
      <mat-option value="usa">USA</mat-option>
    </mat-select>
  </mat-form-field>
</mat-toolbar>
```

#### 2. Add filter actions to update state

```ts
import { Action } from '@ngrx/store';
import { User } from '@demo-app/data-models';

export enum UsersActionTypes {
  LoadUsers = '[Users] Load',
  LoadUsersSuccess = '[Users] Load Sucess',
  LoadUsersFail = '[Users] Load Fail',
  SetUsersFilter = '[Users] Set Filter'
}

export class LoadUsersAction implements Action {
  readonly type = UsersActionTypes.LoadUsers;
}

export class LoadUsersSuccessAction implements Action {
  readonly type = UsersActionTypes.LoadUsersSuccess;
  constructor(public payload: User[]) {}
}

export class LoadUsersFailAction implements Action {
  readonly type = UsersActionTypes.LoadUsersSuccess;
  constructor(public payload: any) {}
}

export class SetUsersFiltersAction {
  readonly type = UsersActionTypes.SetUsersFilter;
  constructor(public payload: string) {}
}

export type UsersActions =
  | LoadUsersAction
  | LoadUsersSuccessAction
  | LoadUsersFailAction
  | SetUsersFiltersAction;
```

* Update the users state to have a selectedCountry 

```ts
import { EntityState } from '@ngrx/entity';
import { User } from '@demo-app/data-models';

export interface Users extends EntityState<User> {
  selectedUserId: number;
  loading: boolean;
  selectedCountry: string
}

export interface UsersState {
  readonly users: Users;
}
```

* Update the initial state

_**libs/admin-portal/users/src/+state/users.interface.ts**_

```ts
import { Users } from './users.interfaces';
import { EntityState, EntityAdapter, createEntityAdapter } from '@ngrx/entity';
import { User } from '@demo-app/data-models';

export const adapter: EntityAdapter<User> = createEntityAdapter<User>({});

export const usersInitialState: Users = adapter.getInitialState({
  selectedUserId: null,
  loading: false,
  selectedCountry: 'none'
});
```

* Remove load dispatch aciton from component

_**libs/admin-portal/users/src/+state/users.init.ts**_

```ts
ngOnInit() {
  // this.store.dispatch(new usersActions.LoadUsersAction());
  this.users$ = this.store.select(selectAllUsers);
}
```

#### 3. Add router method to user-list component

```ts
updateUrlFilters(country: string): void {
  const navigationExtras: NavigationExtras = {
    replaceUrl: true,
    queryParams: {country}
  };
  this.router.navigate([`/users`], navigationExtras);
}
```

* Add an effect to listen for the route change

```ts
  @Effect()
  loadUsersFromRoute = this.dataPersistence.navigation(UserListComponent, {
    run: (a: ActivatedRouteSnapshot, state: UsersState) => {
      return this.usersService
        .getUsers(a.queryParams['country'])
        .pipe(
          map((users: User[]) => new usersActions.LoadUsersSuccessAction(users))
        );
    },
    onError: (a: ActivatedRouteSnapshot, e: any) => {
      return new usersActions.LoadUsersFailAction(e);
    }
  });
```

* Update the user service

```ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable()
export class UsersService {
  constructor(private httpClient: HttpClient) {}

  getUsers(country: string = null) {
    const url = country !== null
      ? `http://localhost:3000/users?country=${country}`
      : `http://localhost:3000/users`;

    return this.httpClient.get(url);
  }
}

```



