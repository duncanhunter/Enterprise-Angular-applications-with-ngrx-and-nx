# Part 11 - Adding nx workspaces

```
ng g app admin-portal --style scss --routing
```

```
ng g ngrx app --module=apps/admin-portal/src/app/app.module.ts  --root
```

```json
"scripts": {
   ...
   "customer-portal": "ng serve -a=customer-portal -p=4200",
   "admin-portal": "ng serve -a=admin-portal -p=4201",
   ...
}
```

add auth module

```ts
@NgModule({
  imports: [
  ...
  AuthModule.forRoot()
  ...
  ],
  declarations: [AppComponent],
  bootstrap: [AppComponent],
  providers: [AppEffects]
})
export class AppModule { }
```

```
ng g lib layout --directory=admin-portal
```

```
<mat-toolbar color="primary" fxLayout="row">
```

```html
<span>Admin Portal</span>
  <div class="right-nav">
    <span>{{(user$ | async)?.username}}</span>
  </div>
</mat-toolbar>
<ng-content></ng-content>
```

```html
<app-layout>
    <router-outlet></router-outlet>
</app-layout>
```

```css
@import '~@angular/material/prebuilt-themes/deeppurple-amber.css';

body {
    margin: 0;
}
```

```ts
@NgModule({
  imports: [
  BrowserModule,
  NxModule.forRoot(),
  RouterModule.forRoot(
    [
      { path: 'auth', children: authRoutes }
    ],
    {
      initialNavigation: 'enabled'
    }
  ),  
  ...
```



