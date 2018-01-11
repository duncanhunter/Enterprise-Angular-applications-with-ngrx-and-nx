# Part 3 - Angular Services

```
ng g service auth -a=auth
```

```
import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders } from '@angular/common/http';

@Injectable()
export class AuthService {
  constructor(private httpClient: HttpClient) {}

  login(username: string, password: string) {
    return this.httpClient.post('http://localhost:3000/login', {
      username: username,
      password: password
    });
  }
}
```

```ts
import { NgModule, ModuleWithProviders } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Route } from '@angular/router';
import { LoginComponent } from './container/login/login.component';
import { HttpClientModule } from '@angular/common/http';
import { AuthService } from './services/auth.service';

export const authRoutes: Route[] = [
  { path: 'login', component: LoginComponent }
];
const COMPONENTS = [LoginComponent];

@NgModule({
  imports: [CommonModule, RouterModule, HttpClientModule],
  declarations: [COMPONENTS],
  exports: [COMPONENTS],
  providers: [AuthService]
})
export class AuthModule {
  static forRoot(): ModuleWithProviders {
    return {
      ngModule: AuthModule,
      providers: [AuthService]
    };
  }
}
```

```ts
AuthModule.forRoot()
```

Add a folder called server to the root directory

Add a file called db.json and index.js

add json-server

```
npm i json-server --save-dev
```

_**package.json**_

```
scripts: {
"server" : "json-server --watch ./server/db.json"
}
```

_**server/db.json**_

```
{
    "users": [
      {
        "id": 1,
        "username": "duncan",
        "country": "australia",
        "password": "123"
      },
      {
        "id": 2,
        "username": "sarah",
        "country": "england",
        "password": "123"
      },
      {
        "id": 3,
        "username": "admin",
        "country": "usa",
        "password": "123"
      },
      {
        "username": "test2",
        "password": "123",
        "id": 4
      }
    ],
    "profiles": [
      {
        "id": 1,
        "userId": 1,
        "job": "plumber"
      },
      {
        "id": 2,
        "userId": 2,
        "job": "developer"
      },
      {
        "id": 3,
        "userId": 3,
        "job": "manager"
      }
    ]
  }
```

_**server/index.js**_

```
const jsonServer = require('json-server');
const server = jsonServer.create();
const router = jsonServer.router('db.json');
const middlewares = jsonServer.defaults();
const db = require('./db.json');

server.use(middlewares);
jsonServer.defaults([{ noCors: true }]);
server.use(jsonServer.bodyParser);

server.post('/login', (req, res, next) => {
  const user = db.users.filter(
    user =>
      user.username === req.body.username && user.password === req.body.password
  )[0];

  user
    ? res.send({ ...removerPasswordFromUser(user), token: checkIfAdmin(user) })
    : res.status(500).send('Incorrect username or password')
});

server.post('/register', (req, res) => {
  const user = db.users.filter(user => user.username === req.body.username)[0];

  user === undefined || user === null
    ? res.send({
        ...removerPasswordFromUser(req.body),
        token: checkIfAdmin(req.body)
      }) && db.users.push(req.body)
    : res.status(500).send('User already exists');
});

server.use('/users', (req, res, next) => {
  if (isAuthorized(req) || req.query.bypassAuth === 'true') {
    next();
  } else {
    res.sendStatus(401);
  }
});

server.use(router);
server.listen(3000, () => {
  console.log('JSON Server is running');
});

function removerPasswordFromUser(user) {
  delete user.password;
  return user;
}

function checkIfAdmin(user, bypassToken = false) {
  return user.username === 'admin' || bypassToken === true
    ? 'admin-token'
    : 'user-token';
}

function isAuthorized(req) {
  return req.headers.authorization === 'admin-token' ? true : false;
}
```



