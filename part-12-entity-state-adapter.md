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
    <button *ngIf="(user$ | async)?.role === 'admin'" [routerLink]="['/users']">Users</button>
  </div>
</mat-toolbar>
<ng-content></ng-content>
```



