# Part 15 Deploying your nx monorepo

The AngularCLI only generates bundles. This means that we cannot build the lib itself. We can only do it by building an app that depends on it.

And then run

```
ng build -a=admin-portal
```



