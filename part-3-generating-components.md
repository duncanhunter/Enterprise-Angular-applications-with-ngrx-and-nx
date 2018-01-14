# Part 2 - Generating components and nx libs

#### 1. Generate a lib

* Run the below command to see all the lib options

```
ng g lib --help
```

* Add a new lib called auth. We will not lazy load this lib as auth will always be used by our app

```
ng g lib auth --routing --parent-module=apps/customer-portal/src/app/app.module.ts
```

#### 2. Add container and presentational components

* Add a new container component to the auth lib

```
ng g c containers/login -a=auth
```

* Add a new presentational component to the auth lib

```
ng g c components/login-form -a=auth
```

You may decide not to make this a presentational component but it makes it easier to test and refactor

* Add a route to the auth module and a custom components array to make it easier to re-export components.

_**libs/auth/src/auth.module.ts**_

```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Route } from '@angular/router';
import { LoginComponent } from './containers/login/login.component';
import { LoginFormComponent } from './components/login-form/login-form.component';

export const authRoutes: Route[] = [
  { path: 'login', component: LoginComponent },
];
const COMPONENTS = [LoginComponent, LoginFormComponent];

@NgModule({
  imports: [CommonModule, RouterModule],
  declarations: [COMPONENTS],
  exports: [COMPONENTS]
})
export class AuthModule {}
```

* Inspect the index.ts file which is for exporting public api surface for the lib.

* Inspect .angular-cli.json libs array allow us to use -a with libs to get cli scaffolding and code generation

* Delete everything but the router-outlet on the apps app.component.html file

_**apps/customer-portal/src/app.components.ts**_

```ts
<router-outlet></router-outlet>
```

* Add the presentational component to the container component.

_**libs/auth/src/containers/login/login.component.html**_

```html
<login-form (submit)="login($event)"></login-form>
```

* Add login function to container component class

_**libs/auth/src/containers/login/login.component.ts**_

```ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss']
})
export class LoginComponent implements OnInit {
  constructor() {}

  ngOnInit() {}

  login(username: string, password: string) {
    console.log(username, password);
  }
}
```

* Add basic login form to presentational component

_**libs/auth/src/components/login-form/login-form.component.html**_

```html
<input type="text" #username>
<input type="text" #password>
<button (click)="login(username.value, password.value)">login</button>
```



