# Part 13 - Router actions and effects

_**libs/admin-portal/users/containers/user-list.component.html**_

```ts
{{users$ | async | json}}
```

```
ng g c components/users-table -a=admin-portal/users
```

```
ng g c components/users-table-toolbar -a=admin-portal/users
```

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

remove load dispactch aciton from component

```ts
ngOnInit() {
  // this.store.dispatch(new usersActions.LoadUsersAction());
  this.users$ = this.store.select(selectAllUsers);
}
```

```ts
updateUrlFilters(country: string): void {
  const navigationExtras: NavigationExtras = {
    replaceUrl: true,
    queryParams: {country}
  };
  this.router.navigate([`/users`], navigationExtras);
}
```



