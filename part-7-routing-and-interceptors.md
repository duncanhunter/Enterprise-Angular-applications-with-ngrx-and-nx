# Part 7 - Routing and interceptors

```
ng g lib user-profile
```

```
ng g lib user-profile --routing --lazy --parent-module=apps/customer-portal/src/app/app.module.ts
```

```
ng g c containers/user-profile -a=user-profile
```

    login() {
      this.authService
        .login(this.loginForm.value.username, this.loginForm.value.password)
        .subscribe(
          (user: User) => this.router.navigate([`/user-profile/${user.id}`]),
          error => (this.error = error)
      );
    }

```
RouterModule.forRoot(
    [
      { path: 'auth', children: authRoutes },
      {
        path: 'user-profile',
        loadChildren: '@demo-app/user-profile#UserProfileModule'
      }
    ],
    {
       initialNavigation: 'enabled'
    }
),
```

```
@NgModule({
  imports: [
    CommonModule,
    RouterModule.forChild([{ path: ':id', component: UserProfileComponent }])
  ],
  declarations: [UserProfileComponent]
})
export class UserProfileModule {}
```



