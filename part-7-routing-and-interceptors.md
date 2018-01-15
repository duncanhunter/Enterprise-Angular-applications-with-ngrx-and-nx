# Part 7 - Routing and interceptors

#### 1.Add a lib for a users profile page

* Run the following command to add a new lib

```
ng g lib user-profile
```

* Add a lazy loaded lib with routing. Note this will add linting rules to .angular-cli.json to stop adding this module to other modules.

```
ng g lib user-profile --routing --lazy --parent-module=apps/customer-portal/src/app/app.module.ts
```

* Add a user-profile container component.

```
ng g c containers/user-profile -a=user-profile
```

#### 2. Add a method in the subscription to navigate to the login page on login

* Run the following command to navigate

_**libs/auth/src/containers/login/login.component.ts**_

```ts
import { Component, OnInit } from '@angular/core';
import { AuthService } from './../../services/auth.service';
import { Authenticate, User } from '@demo-app/data-models';
import { Router } from '@angular/router';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss']
})
export class LoginComponent implements OnInit {
  constructor(private router: Router, private authService: AuthService) {}

  ngOnInit() {}

  login(authenticate: Authenticate) {
    this.authService
      .login(authenticate)
      .subscribe((user: User) =>
        this.router.navigate([`/user-profile/${user.id}`])
      );
  }
}
```

* Add default routes to app module

_**apps/customer-portal/src/app/app.module.ts**_

```ts
RouterModule.forRoot(
    [
      { path: '', pathMatch: 'full', redirectTo: 'user-profile' },
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

_**libs/user-profile/src/user-profile.module.ts**_

```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { UserProfileComponent } from './containers/user-profile/user-profile.component';
import { RouterModule } from '@angular/router';

@NgModule({
  imports: [
    CommonModule,
    RouterModule.forChild([{ path: ':id', component: UserProfileComponent }])
  ],
  declarations: [UserProfileComponent]
})
export class UserProfileModule {}
```

#### 3. Add a route guard to protect profile page

* Generate a guard wit the CLI

```
ng g guard guards/auth/auth -a=auth
```

* Add a static forRoot method and register services and the guard

_**libs/auth/src/auth.module.ts**_

```ts
import { NgModule, ModuleWithProviders } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Route } from '@angular/router';
import { LoginComponent } from './containers/login/login.component';
import { HttpClientModule } from '@angular/common/http';
import { AuthService } from './services/auth.service';
import { MaterialModule } from '@demo-app/material';
import { ReactiveFormsModule } from '@angular/forms';
import { AuthGuard } from './guards/auth/auth.guard';
import { LoginFormComponent } from './components/login-form/login-form.component';

export const authRoutes: Route[] = [
  { path: 'login', component: LoginComponent }
];
const COMPONENTS = [LoginComponent, LoginFormComponent];

@NgModule({
  imports: [
    CommonModule,
    RouterModule,
    HttpClientModule,
    MaterialModule,
    ReactiveFormsModule
  ],
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

* update the authService to set a local flag before we add ngrx later

_**libs/auth/src/services/auth.service.ts**_

```ts
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Authenticate } from '@demo-app/data-models';
import { tap } from 'rxjs/operators';

@Injectable()
export class AuthService {
  isAuthenticated: boolean;
  constructor(private httpClient: HttpClient) {}

  login(authenticate: Authenticate) {
    return this.httpClient
      .post('http://localhost:3000/login', authenticate)
      .pipe(tap(user => (this.isAuthenticated = true)));
  }
}
```

* Add auth guard logic

_**libs/auth/src/guards/auth/auth.guard.ts**_

```ts
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot, Router } from '@angular/router';
import { Observable } from 'rxjs/Observable';
import { AuthService } from './../services/auth.service';

@Injectable()
export class AuthGuard implements CanActivate {

  constructor(
    private router: Router,
    private authService: AuthService) {}

  canActivate(
    next: ActivatedRouteSnapshot,
    state: RouterStateSnapshot): Observable<boolean> | Promise<boolean> | boolean {
    if(this.authService.isAuthenticated) {
      return true;
    } else {
      this.router.navigate([`/auth/login`]);
      return false;
    }
  }
}
```

* add auth guard to main routes

_**apps/customer-portal/src/app/app.module.ts**_

```ts
import { NgModule } from '@angular/core';
import { AppComponent } from './app.component';
import { BrowserModule } from '@angular/platform-browser';
import { NxModule } from '@nrwl/nx';
import { RouterModule } from '@angular/router';
import { StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';
import { StoreDevtoolsModule } from '@ngrx/store-devtools';
import { environment } from '../environments/environment';
import { StoreRouterConnectingModule } from '@ngrx/router-store';
import { authRoutes, AuthModule } from '@demo-app/auth';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { AuthGuard } from '@demo-app/auth';

@NgModule({
  imports: [
    BrowserModule,
    NxModule.forRoot(),
    RouterModule.forRoot(
      [
        { path: '', pathMatch: 'full', redirectTo: 'user-profile' },
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
    StoreModule.forRoot({}),
    EffectsModule.forRoot([]),
    !environment.production ? StoreDevtoolsModule.instrument() : [],
    StoreRouterConnectingModule,
    AuthModule.forRoot(),
    BrowserAnimationsModule
  ],
  declarations: [AppComponent],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

#### 4. Add angular interceptor

* Update auth service to set a token in local storage

_**libs/auth/src/services/auth.service.ts**_

```ts
import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { tap } from 'rxjs/operators';
import { User, Authenticate } from '@demo-app/data-models';

@Injectable()
export class AuthService {
  user: User;
  isAuthenticated: boolean;

  constructor(private httpClient: HttpClient) {}

  login(authenticate: Authenticate) {
    return this.httpClient
      .post('http://localhost:3000/login', authenticate)
      .pipe(
        tap((user: User) => {
          this.isAuthenticated = true;
          this.user = user;
          this.setAuthToken(user.token);
        })
      );
  }

  setAuthToken(token: string) {
    localStorage.setItem('token', token);
  }

  getAuthToken(): string {
    return localStorage.getItem('token');
  }

  clearAuthToken() {
    localStorage.removeItem('token');
  }
}
```

* Add an angular interceptor

```ts
import { Injectable, Injector } from '@angular/core';
import {
  HttpEvent,
  HttpInterceptor,
  HttpHandler,
  HttpRequest
} from '@angular/common/http';
import { Observable } from 'rxjs/Observable';
import { AuthService } from '../services/auth.service';
import { switchMap, tap } from 'rxjs/operators';
import { of } from 'rxjs/Observable/of';

@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  authService: AuthService;
  constructor(private injector: Injector) {}

  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    this.authService = this.injector.get(AuthService);
    const token = this.authService.getAuthToken();

    if (token) {
      const authReq = req.clone({
        headers: req.headers.set('Authorization', token)
      });
      return next.handle(authReq);
    } else {
      return next.handle(req);
    }
  }
}
```

```
import { NgModule, ModuleWithProviders } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Route } from '@angular/router';
import { LoginComponent } from './container/login/login.component';
import { HttpClientModule, HTTP_INTERCEPTORS } from '@angular/common/http';
import { AuthService } from './services/auth.service';
import { MaterialModule } from '@demo-app/material';
import { ReactiveFormsModule } from '@angular/forms';
import { AuthGuard } from './/guards/auth.guard';
import { AuthInterceptor } from './interceptors/auth.interceptor';

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
    ReactiveFormsModule
  ],
  declarations: [COMPONENTS],
  exports: [COMPONENTS],
  providers: [
    AuthService,
    AuthGuard
  ]
})
export class AuthModule {
  static forRoot(): ModuleWithProviders {
    return {
      ngModule: AuthModule,
      providers: [
        AuthService,
        AuthGuard,
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



