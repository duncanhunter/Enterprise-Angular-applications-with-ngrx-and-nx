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

```
ng g guard guards/auth  -a=auth
```

```ts
import { NgModule, ModuleWithProviders } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Route } from '@angular/router';
import { LoginComponent } from './container/login/login.component';
import { HttpClientModule } from '@angular/common/http';
import { AuthService } from './services/auth.service';
import { MaterialModule } from '@demo-app/material';
import { ReactiveFormsModule } from '@angular/forms';
import { AuthGuard } from '@demo-app/auth/src/guards/auth.guard';

export const authRoutes: Route[] = [
  { path: 'login', component: LoginComponent }
];
const COMPONENTS = [LoginComponent];

@NgModule({
  imports: [CommonModule, RouterModule, HttpClientModule, MaterialModule, ReactiveFormsModule],
  declarations: [COMPONENTS],
  exports: [COMPONENTS],
  providers: [AuthService, AuthGuard]
})
export class AuthModule {
  static forRoot(): ModuleWithProviders {
    return {
      ngModule: AuthModule,
      providers: [AuthService, AuthGuard]
    };
  }
}
```

```ts
login(username: string, password: string) {
  return this.httpClient.post('http://localhost:3000/login', {
    username: username,
    password: password
  })
  .pipe(tap(user => this.isAuthenticated = true));
}
```

```ts
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
import { Observable } from 'rxjs/Observable';
import { AuthService } from './../services/auth.service';

@Injectable()
export class AuthGuard implements CanActivate {

  constructor(private authService: AuthService) {}

  canActivate(
    next: ActivatedRouteSnapshot,
    state: RouterStateSnapshot): Observable<boolean> | Promise<boolean> | boolean {
    return this.authService.isAuthenticated ? true : false;
  }
}
```

```

export { AuthModule , authRoutes } from './src/auth.module';
export { AuthGuard } from './src/guards/auth.guard';
```

```ts
RouterModule.forRoot(
   [
     { path: 'auth', children: authRoutes },
     {
       path: 'user-profile',
       loadChildren: '@demo-app/user-profile#UserProfileModule',
       canActivate: [AuthGuard]
     }
   ],
   {
     initialNavigation: 'enabled'
   }
),
```



