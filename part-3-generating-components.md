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

* Add a new container component to the auth lib

```
ng g c container/login -a=auth
```

* Add a new presentational component to the auth lib

```
ng g c components/login-form -a=auth
```

You may decide not to make this a presentational component but it makes it easier to test and refactor

_**libs/auth/src/auth.module.ts**_

```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Route } from '@angular/router';
import { LoginComponent } from './container/login/login.component';

export const authRoutes: Route[] = [
  { path: 'login', component: LoginComponent },
];
const COMPONENTS = [LoginComponent];

@NgModule({
  imports: [CommonModule, RouterModule],
  declarations: [COMPONENTS],
  exports: [COMPONENTS]
})
export class AuthModule {}
```

* Inspect the index.ts file which is for exporting public api surface for the lib.

* Inspect .angular-cli.json libs array allow us to use -a with libs to get cli scaffolding and code generation

2. Delete everything but the router-outlet on the apps app.component.html file

_**apps/customer-portal/src/app.components.ts**_

```ts
<router-outlet></router-outlet>
```

```html
<input type="text" #username>
<input type="text" #password>
<button (click)="login(username.value, password.value)">login</button>
```

```ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss']
})
export class LoginComponent implements OnInit {

  constructor() { }

  ngOnInit() {
  }

  login(username: string, password: string) {
    console.log(username, password);
  }
}
```



