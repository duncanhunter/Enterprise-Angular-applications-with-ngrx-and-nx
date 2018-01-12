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



