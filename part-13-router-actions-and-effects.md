# Part 13 - Router actions and effects

#### 1 .Add a new presentational components for users

```
ng g c components/users-table -a=admin-portal/users
```

* Add a toolbar module

```
ng g c components/users-table-toolbar -a=admin-portal/users
```

* Add a toolbar with select field

_**libs/admin-portal/users/src/components/users-table-toolbar/users-table-toolbar.component.html**_

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

_**libs/admin-portal/users/src/components/users-table-toolbar/users-table-toolbar.component.ts**_

```ts
import { Component, OnInit, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'users-table-toolbar',
  templateUrl: './users-table-toolbar.component.html',
  styleUrls: ['./users-table-toolbar.component.scss']
})
export class UsersTableToolbarComponent {
  @Output() onFilter = new EventEmitter<string>();
  selectedCountry = 'none';

  filter(country:string){
    this.onFilter.emit(country);
  }
}
```

* Add a users list table component

_**libs/admin-portal/users/src/components/users-table/users-table.component.ts**_

```ts
import { Component, OnInit, Input, OnChanges, SimpleChanges } from '@angular/core';
import { User } from '@demo-app/data-models';
import { MatTableDataSource } from '@angular/material';

@Component({
  selector: 'app-users-table',
  templateUrl: './users-table.component.html',
  styleUrls: ['./users-table.component.scss']
})
export class UsersTableComponent implements OnChanges {
  @Input() users: User[];

  displayedColumns = ['username', 'country'];
  dataSource: MatTableDataSource<User>;

  ngOnChanges(changes: SimpleChanges) {
    this.dataSource = new MatTableDataSource(this.users);
  }

  setTableDatasource(dataSource): void {
    this.dataSource = new MatTableDataSource(this.users);
  }

}
```

* Add the html

_**libs/admin-portal/users/src/components/users-table/users-table.component.html**_

```html
<div class="mat-elevation-z8">
  <mat-table #table [dataSource]="dataSource">

    <ng-container matColumnDef="username">
      <mat-header-cell *matHeaderCellDef> Username </mat-header-cell>
      <mat-cell *matCellDef="let element"> {{element.username}} </mat-cell>
    </ng-container>

    <ng-container matColumnDef="country">
      <mat-header-cell *matHeaderCellDef> Country </mat-header-cell>
      <mat-cell *matCellDef="let element"> {{element.country}} </mat-cell>
    </ng-container>

    <mat-header-row *matHeaderRowDef="displayedColumns"></mat-header-row>
    <mat-row *matRowDef="let row; columns: displayedColumns;"></mat-row>
  </mat-table>
</div>
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

_**libs/admin-portal/users/src/containers/user-list/user-list.component.html**_

```
<users-table-toolbar (onFilter)="updateUrlFilters($event)"></users-table-toolbar>
<app-users-table [users]="users$ | async"></app-users-table>
```

_**libs/admin-portal/users/src/containers/user-list/user-list.component.ts**_

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



