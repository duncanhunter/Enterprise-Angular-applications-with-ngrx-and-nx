# Part 3 - Angular Services

#### 1. Generate a new angular service

* Run the following command to make a new service in the auth lib

```
ng g service services/auth -a=auth
```

#### 2. Add login method and http post for the login

_**libs/auth/src/services/auth.service.ts**_

```ts
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

* Export the auth service from the auth lib and add a static for root method.

_**libs/auth/src/auth.module.ts**_

```ts
import { NgModule, ModuleWithProviders } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Route } from '@angular/router';
import { LoginComponent } from './containers/login/login.component';
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

* update the app module to import the auth lib

_**apps/customer-portal/src/app/app.module.ts**_

```ts
AuthModule.forRoot()
```

#### 3. Add a json-server to be able to make http requests and mock a real server

* Add a folder called server to the root directory

* Add a file called db.json and index.ts

* add json-server and ts-node

```
npm i json-server ts-node --save-dev
```

* Add the below script to the scripts in the package.json

_**package.json**_

```json
scripts: {
"server" : "json-server --watch ./server/db.json"
}
```

* Add the default mock data to the db.json file

_**server/db.json**_

```json
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

* Add the mock server code below

_**server/index.ts**_

```ts
const jsonServer = require('json-server');
const server = jsonServer.create();
const router = jsonServer.router('server/db.json');
const middlewares = jsonServer.defaults();
const db = require('./db.json');
const fs = require('fs');

server.use(middlewares);
server.use(jsonServer.bodyParser);

server.post('/login', (req, res, next) => { 
  const users = readUsers();

  const user = users.filter(
    u => u.username === req.body.username && u.password === req.body.password
  )[0];

  if (user) {
    res.send({ ...formatUser(user), token: checkIfAdmin(user) });
  } else {
    res.status(400).send('Incorrect username or password');
  }
});

server.post('/register', (req, res) => {
  const users = readUsers();
  const user = users.filter(u => u.username === req.body.username)[0];

  if (user === undefined || user === null) {
    res.send({
      ...formatUser(req.body),
      token: checkIfAdmin(req.body)
    });
    db.users.push(req.body);
  } else {
    res.status(500).send('User already exists');
  }
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

function formatUser(user) {
  delete user.password;
  user.role = user.username === 'admin'
    ? 'admin'
    : 'user';
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

function readUsers() {
  const dbRaw = fs.readFileSync('./server/db.json');  
  const users = JSON.parse(dbRaw).users
  return users;
}
```

* Start the server and leave it running whenever you use the apps.

```
npm run server
```



