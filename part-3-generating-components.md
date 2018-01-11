# Part 2 - Generating components and nx libs

```
ng g lib --help
```

```
ng g lib auth --routing --parent-module=apps/customer-portal/src/app/app.module.ts
```

```
ng g c container/login -a=auth
```

```
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

index.ts is for the public api for the lib

add to cli.json libs array allow us to use --app with libs to get cli scaffolding and code generation

