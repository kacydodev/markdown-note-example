## Backend Node.js Express.js

```bash
code dashboard-nodejs-express
```

See also: [The Network Tool and Related Comcepts](https://scrimba.com/s05e8338en)

## REQ/RES cycle

![image-20250910124337909](/assets/image-20250910124337909.png)

![image-20250910124451796](/assets/image-20250910124451796.png)

## Setup Node.js

[Node.js 2025 Guide: How to Set Up Express.js with TypeScript, ESLint, and Prettier](https://medium.com/@gabrieldrouin/node-js-2025-guide-how-to-setup-express-js-with-typescript-eslint-and-prettier-b342cd21c30d)

### Practices

Filter queries

```
Challenge:
1. When a user hits the /api endpoint with query params, filter the data so
we only serve objects that meet their requirements.

The user can filter by the following properties:
  industry, country, continent, is_seeking_funding, has_mvp

Test Cases

/api?industry=renewable%20energy&country=germany&has_mvp=true
  Should get the "GreenGrid Energy" object.

/api?industry=renewable%20energy&country=germany&has_mvp=false
  Should not get any object

/api?continent=asia&is_seeking_funding=true&has_mvp=true
  should get for objects with IDs 3, 22, 26, 29
```

## Express Router/Routes Pattern

```
.
└── src/
    ├── controllers/
    │   ├── getAllData.js
    │   └── getDataByPathParams.js
    ├── router/
    │   └── apiRoutes.js
    └── index.js
```

```js
// index.js

...
import { apiRouter } from './routes/apiRouter.js';

const app = express();
app.use('/api', apiRouter);

...
```

```js
// apiRoute.js
// import controller functions

export const apiRouter = express.Router();

// Call apiRouter.get() to pass in route string
// and controllder functions
```

```js
// getAllData.js

export const getAllData = (req, res) => {
  // handle request and response
  ...
}
```

### Not Found Route

```javascript
// index.js

...

// make sure to place this at the end of route handling block
app.use((req, res) => {
  res.status(404);
  res.json({ message: 'Endpoint not found. Please check with API Document.' });
});

...
```

### CORS

```javascript
// CORS expained:
// https://scrimba.com/learn-expressjs-c062las154/~0xpz
const app = express();
app.use(cors());
```

## SQLite

[SQLite Client for Node.js Apps](https://www.npmjs.com/package/sqlite#sqlite-client-for-nodejs-apps)

[SQLite3 in Node with better-sqlite3](https://www.youtube.com/watch?v=IooIXYf0PIo)

See also: [Node.js Path Module](https://www.w3schools.com/nodejs/nodejs_path.asp)

See also: [SQLite Tutorial](https://www.sqlitetutorial.net/)

See also: [SQLite basic SELECT statement - Exercises, Practice, Solution](https://www.w3resource.com/sqlite-exercises/sqlite-basic-simple-exercises.php)

Example of inserting columns into db - Remember to add `db.close()`

![image-20251023232024027](/assets/image-20251023232024027.png)

Example of how to serve database with Express:

![image-20251023232619855](/assets/image-20251023232619855.png)

See more: [Up and Running with SQLite3 in a NodeJS API](https://www.youtube.com/watch?v=_RtpUaBSie0) | [SQLite Tutorial](https://www.sqlitetutorial.net/sqlite-nodejs/connect/) | [source code](https://github.com/prof3ssorSt3v3/node-sqlite3)

### How to create table

> _Use the exec() method to write SQL to create a table called 'products'. It should have the following columns:_
>
> _id (unique key)_
>
> _title (required, text)_
>
> _artist (required, text)_
>
> _price (required, floating-point number)_
>
> _image (required, text) (this is "text" because it will hold an image url)_
>
> _year (integer)_
>
> _genre (text)_
>
> stock (integer)

```javascript
// createTable.js

...

await db.exec(`
    CREATE TABLE IF NOT EXISTS products (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      title TEXT NOT NULL,
      artist TEXT NOT NULL,
      price REAL NOT NULL,
      image TEXT NOT NULL,
      year INTEGER,
      genre TEXT,
      stock INTEGER
    )
  `);

await db.close();
```

See also: [splite3 Methods Overview](https://scrimba.com/learn-expressjs-c062las154/~0xku)

### How to seed table

```javascript
// seedTable.js
...

const db = await open({
  filename: path.join('__dirname', '..', 'src', 'data', 'database.db'),
  driver: sqlite3.Database,
});

try {
  await db.exec('BEGIN TRANSACTION');

  for (const { id, title } of todoObject) {
    await db.run(
      `
        INSERT INTO todolist (id, title)
        VALUES (?, ?)
      `,
      [id, title],
    );
  }
  await db.exec('COMMIT');
  console.log(`'todolist' Table seeded ✅`);
} catch (err) {
  await db.exec('ROLLBACK');
  console.log('Error inserting data ❌', err.message);
} finally {
  await db.close();
  console.log('Database connection closed 🔐');
}

...

```

### How to get data

```javascript
// getData.js

...

try {
  const query = 'SELECT * FROM todolist WHERE tag = ?';
  const params = ['urgent'];

  const todolist = await db.all(query, params);
  console.log(todolist)
} catch(err) {
  ...
} finally {
  ...
}

...
```

Get list of genres to pass onto frontend:

```javascript
try {
	...

  const genreRows = await db.all('SELECT DISTINCT genre from products');
  const genres = genreRows.map(row => row.genre);
  res.json(genres);
	// outputs [ 'rock', 'indie', 'ambient', 'folk', 'punk' ]
} catch { ... }
```

Filter by query if query exists:

```javascript
try {
  ...

  let query = ' SELECT * FROM todolist';
  let params = [];

  const { tag } = req.query;

  if(tag) {
    query += ' WHERE tag = ?';
    param.push(tag);
  }

  const todolist = await db.all(query, params)
  res.json(todolist)
} catch { ... }
```

### Partial query seach

[Scrimba - Add Search Functionality](https://scrimba.com/learn-expressjs-c062las154/~04tf)

[SQL Where Contains String – Substring Query Example](https://www.freecodecamp.org/news/sql-where-contains-string-substring-query-example/)

```javascript
query =
	'SELECT * FROM products WHERE title LIKE ? OR artist LIKE ? OR genre LIKE ?';
// % indicates partial match
const searchPattern = `%${search}%`;
// 3 searchPattern to match 3 "?" from query
params.push(searchPattern, searchPattern, searchPattern);

const products = await db.all(query, params);
```

#### Frequently Used Queries

- Get all genre without repeat:
  ```javascript
  'SELECT DISTINCT genre FROM products';
  ```

## Authentication

See also: [Securing Express APIs using OAuth2 and JSON Web Tokens](https://dev.to/rowjay007/securing-express-apis-using-oauth2-and-json-web-tokens-47o8)

The sign up steps

![image-20251028094818376](/assets/image-20251028094818376.png)

#### JWT Explained

- [JWT Authentication | Node JS and Express tutorials for Beginners](https://www.youtube.com/watch?v=favjC6EKFgw)

- [gitdagray / express_jwt](https://github.com/gitdagray/express_jwt)

- [Introduction to JSON Web Tokens](https://www.jwt.io/introduction#why-use-json-web-tokens)

![JWT.io Debugger](https://www.jwt.io/_next/image?url=https%3A%2F%2Fcdn.auth0.com%2Fwebsite%2Fjwt%2Fintroduction%2Fdebugger.png&w=3840&q=75)

## Express.js Questions

- `tsc` vs. `ts-node`: [What's the difference between tsc (TypeScript compiler) and ts-node?](https://stackoverflow.com/questions/51448376/whats-the-difference-between-tsc-typescript-compiler-and-ts-node)

  > Most common practice is that tsc is used for production build and ts-node for development purposes running in --watch mode along with nodemon. This is a command i often use for development mode for my node/typescript projects:
  >
  > ```json
  > "dev": "nodemon -w *.ts -e ts -x ts-node --files -H -T ./src/index.ts"
  > ```

- `Error: process.env.PORT not working: Make sure that .env is in root folder`

  **Solution:** run `npm install dotenv`

  ```javascript
  import dotenv from 'dotenv';
  dotenv.config({ debug: true });
  ```

- [How to Set Up ESLint and Prettier in a TypeScript Project](https://dev.to/forhad96/-how-to-set-up-eslint-and-prettier-in-a-typescript-project-3pi2)

- `req.query` type issue

  ```typescript
  Argument of type 'string | ParsedQs | string[] | ParsedQs[] | undefined' is not assignable to parameter of type 'string'.
  Type 'undefined' is not assignable to type 'string'
  ```

  **Solution:** [How to type `request.query` in express using TypeScript?](https://stackoverflow.com/questions/63538665/how-to-type-request-query-in-express-using-typescript)

- [Error: Can't set headers after they are sent to the client](https://stackoverflow.com/questions/7042340/error-cant-set-headers-after-they-are-sent-to-the-client)
  **Solution:** `return res.send('something')`

- [What are express.json() and express.urlencoded()?](https://stackoverflow.com/questions/23259168/what-are-express-json-and-express-urlencoded)
