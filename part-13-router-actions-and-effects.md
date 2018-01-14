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

```ts
@Effect()
 loadUsersFromROute = this.dataPersistence.navigation(UserListComponent, {
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



