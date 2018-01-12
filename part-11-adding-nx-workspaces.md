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



