# Agendash_AIST

A Dashboard for [Agenda](https://github.com/agenda/agenda). Note: It is a copy from [Agendash](https://www.npmjs.com/package/agendash).

---

### Amendments

- Fix Modal Not closing after submit
- Minor UI Changes
---

### Screenshots

#### Dashboard

![Auto-refresh list of jobs](dashboard.png)

---

#### Create jobs

![See job details, requeue or delete jobs](create.png)

---

#### Search by name, metadata, job status

![Search for a job by name or metadata ](searching.png)

---

### Install

```
npm install --save agendash_aist
```

_Note_: `Agendash_aist` requires mongodb version >2.6.0 to perform the needed aggregate queries. This is your mongo database version, not your node package version! To check your database version, connect to mongo and run `db.version()`.

### Middleware usage

#### Express

Agendash_aist provides Express middleware you can use at a specified path, for example this will
make Agendash available on your site at the `/dash` path. Note: Do not try to mount Agendash
at the root level like `app.use('/', Agendash(agenda))`.

```js
var express = require("express");
var app = express();

// ... your other express middleware like body-parser

var Agenda = require("agenda");
var Agendash = require("agendash_aist");

var agenda = new Agenda({ db: { address: "mongodb://127.0.0.1/agendaDb" } });
// or provide your own mongo client:
// var agenda = new Agenda({mongo: myMongoClient})
agenda.define('<your-task-name>', async (job) => {
  const metadata = job.attrs.data; // Access the metadata within the job function
  console.log('Job triggered with metadata:', metadata);
  // Add your job logic here
});

agenda.start();

app.use("/dash", Agendash(agenda));

// ... your other routes

// ... start your server
```

By mounting Agendash as middleware on a specific path, you may provide your
own authentication for that path. For example if you have an authenticated
session using passport, you can protect the dashboard path like this:

```js
app.use(
  "/dash",
  function (req, res, next) {
    if (!req.user || !req.user.is_admin) {
      res.send(401);
    } else {
      next();
    }
  },
  Agendash(agenda)
);
```

Other middlewares will come soon in the folder `/lib/middlewares/`.
You'll just have to update the last line to require the middleware you need:

```js
app.use(
  "/agendash",
  Agendash(agenda, {
    middleware: "connect",
  })
);
```

Note that if you use a CSRF protection middleware like [`csurf`](https://www.npmjs.com/package/csurf), you might need to [configure it off](https://github.com/agenda/agendash/issues/23#issuecomment-270917949) for Agendash-routes.

### Set Failed Status

```js
agenda.define('<your-task-name>', async (job) => {
  throw new Error('Some Error Message')
});
```

### Standalone usage

Agendash comes with a standalone Express app which you can use like this:

```bash
./node_modules/.bin/agendash --db=mongodb://localhost/agendaDb --collection=agendaCollection --port=3002
```

or like this, for default collection `agendaJobs` and default port `3000`:

```bash
./node_modules/.bin/agendash --db=mongodb://localhost/agendaDb
```

If you are using npm >= 5.2, then you can use [npx](https://medium.com/@maybekatz/introducing-npx-an-npm-package-runner-55f7d4bd282b):

```bash
npx agendash --db=mongodb://localhost/agendaDb --collection=agendaCollection --port=3002
```

Then browse to `http://localhost:3002/`.

### Docker usage

Agendash can also be run within a Docker container like this:

```bash
docker run -p 3000:3000 \
  --env MONGODB_URI=mongo://myUser:myPass@myHost/myDb \
  --env COLLECTION=myAgendaCollection agenda/agendash
```

Then browse to `http://localhost:3000/`.
