# Part 13 - Router actions and effects

_**libs/admin-portal/users/containers/user-list.component.html**_

```ts
{{users$ | async | json}}
```

```
ng g c components/users-table -a=admin-portal/users
```

```
ng g c components/users-table-toolbar -a=admin-portal/users
```



