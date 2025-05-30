# json-server
## Orginal Repo link: [here](https://github.com/typicode/json-server)

## Install

```shell
npm install json-server
```

## Usage

Create a `db.json` or `db.json5` file

```json
{
  "posts": [
    { "id": "1", "title": "a title", "views": 100 },
    { "id": "2", "title": "another title", "views": 200 }
  ],
  "comments": [
    { "id": "1", "text": "a comment about post 1", "postId": "1" },
    { "id": "2", "text": "another comment about post 1", "postId": "1" }
  ],
  "profile": {
    "name": "typicode"
  }
}
```

<details>

<summary>View db.json5 example</summary>

```json5
{
  posts: [
    { id: '1', title: 'a title', views: 100 },
    { id: '2', title: 'another title', views: 200 },
  ],
  comments: [
    { id: '1', text: 'a comment about post 1', postId: '1' },
    { id: '2', text: 'another comment about post 1', postId: '1' },
  ],
  profile: {
    name: 'typicode',
  },
}
```

</details>

Pass it to JSON Server CLI

```shell
$ npx json-server db.json
```

Or,
```shell
json-server --watch ./database/db.json
```

Get a REST API

```shell
$ curl http://localhost:3000/posts/1
{
  "id": "1",
  "title": "a title",
  "views": 100
}
```

Run `json-server --help` for a list of options

## Routes

Based on the example `db.json`, you'll get the following routes:

```
GET    /posts
GET    /posts/:id
POST   /posts
PUT    /posts/:id
PATCH  /posts/:id
DELETE /posts/:id

# Same for comments
```

```
GET   /profile
PUT   /profile
PATCH /profile
```

## Params

### Conditions

- ` ` → `==`
- `lt` → `<`
- `lte` → `<=`
- `gt` → `>`
- `gte` → `>=`
- `ne` → `!=`

```
GET /posts?views_gt=9000
```

### Range

- `start`
- `end`
- `limit`

```
GET /posts?_start=10&_end=20
GET /posts?_start=10&_limit=10
```

### Paginate

- `page`
- `per_page` (default = 10)

```
GET /posts?_page=1&_per_page=25
```

### Sort

- `_sort=f1,f2`

```
GET /posts?_sort=id,-views
```

### Nested and array fields

- `x.y.z...`
- `x.y.z[i]...`

```
GET /foo?a.b=bar
GET /foo?x.y_lt=100
GET /foo?arr[0]=bar
```

### Embed

```
GET /posts?_embed=comments
GET /comments?_embed=post
```

## Delete

```
DELETE /posts/1
DELETE /posts/1?_dependent=comments
```

## Serving static files

If you create a `./public` directory, JSON Server will serve its content in addition to the REST API.

You can also add custom directories using `-s/--static` option.

```sh
json-server -s ./static
json-server -s ./static -s ./node_modules
```

## Notable differences with v0.17

- `id` is always a string and will be generated for you if missing
- use `_per_page` with `_page` instead of `_limit`for pagination
- use Chrome's `Network tab > throtling` to delay requests instead of `--delay` CLI option


## Setup Custom Routes & Middlewares

### Add Custom Routes

Create a `routes.json` file to **remap routes**:

```json
// routes.json
{
  "/api/posts": "/posts",
  "/api/comments": "/comments"
}
```

Then run:

```bash
json-server --watch db.json --routes routes.json
```

Now, `/api/posts` maps to `/posts`.


### Add Custom Middleware

Create a `server.js` file:

```js
// server.js
const jsonServer = require('json-server');
const server = jsonServer.create();
const router = jsonServer.router('db.json');
const middlewares = jsonServer.defaults();

// Custom middleware
server.use((req, res, next) => {
  console.log(`[${req.method}] ${req.url}`);
  // Example: Add a timestamp
  req.body.createdAt = Date.now();
  next();
});

// Use default middlewares (logger, static, cors, etc)
server.use(middlewares);

// Custom routes
server.use(jsonServer.rewriter({
  '/api/*': '/$1'
}));

// Use the default router
server.use(router);

// Start the server
server.listen(3000, () => {
  console.log('JSON Server is running on http://localhost:3000');
});
```

---

### Run the Custom Server

```bash
node server.js
```

---

### Use with `npm` Script

In your `package.json`:

```json
"scripts": {
  "start-server": "node server.js"
}
```

Run it using:

```bash
npm run start-server
```

---

## ✅ Output Example

With the above setup, hitting:

```http
GET http://localhost:3000/api/posts
```

will return the `posts` array, and your custom middleware will log requests and modify `req.body`.

---

More advanced middlewares like **auth**, **rate limiting**, or **JWT**, let me know!
