Setup

Dependencies

Como clonamos el repositorio, obtuvimos todas las dependencias necesarias por ahora. No olvide que, después de la clonación, debe ejecutar npm install o npm i dentro de la carpeta. Para iniciar la aplicación, use el comando npm run dev desde la carpeta del proyecto.

¡No instale paquetes npm relacionados con authentication (user) todavía! La paciencia es una virtud.

Definamos el modelo

Nuestra aplicación ya está conectada a la base de datos y podemos ver cuando ejecutamos la aplicación en nuestra terminal:
Connected to Mongo! Database name: "project-management-server".

¡Así que comencemos! Estamos creando una aplicación de gestión de proyectos, así que definamos un esquema para nuestros proyectos.

Agreguemos un archivo project.js en la carpeta models:

```js
// models/project.js

const mongoose = require("mongoose");
const Schema = mongoose.Schema;

const projectSchema = new Schema({
  title: String,
  description: String,
  tasks: [{ type: Schema.Types.ObjectId, ref: "Task" }]
  // owner will be added later on
});

const Project = mongoose.model("Project", projectSchema);

module.exports = Project;
```

Nuestros proyectos tendrán un título y una descripción tanto de tipo String como tasks de tipo ObjectId que hagan referencia al Task model. Más adelante agregaremos la owner property. Mongo agregará automáticamente un campo de id único generado automáticamente, por lo que no necesitamos especificarlo.

Nuestros proyectos tendrán algunas tareas, así que veamos qué podemos hacer con su estructura de datos. Dentro de la carpeta de models, cree un nuevo archivo task.js. Las tareas tendrán las siguientes propiedades:

```js
// models/task.js

const mongoose = require("mongoose");
const Schema = mongoose.Schema;

const taskSchema = new Schema({
  title: String,
  description: String,
  project: { type: Schema.Types.ObjectId, ref: "Project" }
});

const Task = mongoose.model("Task", taskSchema);

module.exports = Task;
```

¡Excelente! Las estructuras de datos están definidas, así que procedamos a definir las rutas.

Define the routes

Adoptando la arquitectura REST, proporcionaremos las siguientes rutas en nuestra API:

Project routes

<table>
<thead>
<tr>
<th>URL</th>
<th>HTTP verb</th>
<th>Request body</th>
<th>Action</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>/api/projects</code></td>
<td>GET</td>
<td>(empty)</td>
<td>Returns all the projects</td>
</tr>
<tr>
<td><code>/api/projects</code></td>
<td>POST</td>
<td>JSON</td>
<td>Adds a new project</td>
</tr>
<tr>
<td><code>/api/projects/:id</code></td>
<td>GET</td>
<td>(empty)</td>
<td>Returns the specified project</td>
</tr>
<tr>
<td><code>/api/projects/:id</code></td>
<td>PUT</td>
<td>JSON</td>
<td>Edits the specified project</td>
</tr>
<tr>
<td><code>/api/projects/:id</code></td>
<td>DELETE</td>
<td>(empty)</td>
<td>Deletes the specified project</td>
</tr>
</tbody>
</table>

Task routes

<table>
<thead>
<tr>
<th>URL</th>
<th>HTTP verb</th>
<th>Request body</th>
<th>Action</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>/api/tasks</code></td>
<td>POST</td>
<td>JSON</td>
<td>Adds a new task</td>
</tr>
<tr>
<td><code>/api/tasks/:id</code></td>
<td>GET</td>
<td>(empty)</td>
<td>Returns the specified task</td>
</tr>
<tr>
<td><code>/api/tasks/:id</code></td>
<td>PUT</td>
<td>JSON</td>
<td>Edits the specified task</td>
</tr>
<tr>
<td><code>/api/tasks/:id</code></td>
<td>DELETE</td>
<td>(empty)</td>
<td>Deletes the specified task</td>
</tr>
</tbody>
</table>

Avancemos y creemos dos archivos dentro de la carpeta de rutas: project-routes.js y task-routes.js.

Ya le mostramos todas las rutas para projects y tasks, así que comencemos a ingresarlas en estos archivos.

GET and POST

Lo primero que haremos es permitir que nuestros usuarios agreguen algunos proyectos en la base de datos, entonces comenzamos con la ruta POST para crear un proyecto. En el archivo project-routes.js, agregue el siguiente código para la ruta mencionada:

```js
// routes/project-routes.js
const express = require("express");
const mongoose = require("mongoose");
const router = express.Router();

const Project = require("../models/project");
const Task = require("../models/task");

// POST route => to create a new project
router.post("/projects", (req, res, next) => {
  Project.create({
    title: req.body.title,
    description: req.body.description,
    tasks: []
  })
    .then(response => {
      res.json(response);
    })
    .catch(err => {
      res.json(err);
    });
});

module.exports = router;
```

Aquí utilizamos el método create() y pasamos los parámetros del cuerpo en la solicitud para crear un nuevo proyecto y guardarlo en la base de datos.
Ahora, vamos a app.js y, hacia el final del archivo, solicitemos el archivo project-routes.js recién creado. No olvides agregarles el prefijo api (este paso no es obligatorio pero nos ayudará a largo plazo).

```js
// app.js
...

// ROUTES MIDDLEWARE STARTS HERE:
app.use('/api', require('./routes/project-routes'));
// app.use('/api', require('./routes/task-routes'));
```

Puedes seguir adelante y usar Postman para probar esta ruta.

Ahora pasaremos a definir la primera ruta GET para la collection de proyectos:

```js
// routes/project-routes.js
...

// GET route => to get all the projects
router.get('/projects', (req, res, next) => {
  Project.find().populate('tasks')
    .then(allTheProjects => {
      res.json(allTheProjects);
    })
    .catch(err => {
      res.json(err);
    })
});
```

Vamos a desglosarlo:

1. Obtenemos una referencia de Project mongoose para operar en la colección projects
2. En GET usamos el método find() sin parámetros para recuperar todos los projects
3. Utilizamos una promesa de JavaScript para obtener la respuesta de nuestra base de datos, y la recuperamos como un objeto JSON
4. catch() trata con errores

También debe probar la funcionalidad de esta ruta utilizando Postman.

Complete the API

Ahora que validamos nuestras dos primeras rutas, completemos la API REST:

```js
// routes/project-routes.js

...

// GET route => to get a specific project/detailed view
router.get('/projects/:id', (req, res, next)=>{

  if(!mongoose.Types.ObjectId.isValid(req.params.id)) {
    res.status(400).json({ message: 'Specified id is not valid' });
    return;
  }

  // our projects have array of tasks' ids and
  // we can use .populate() method to get the whole task objects

  Project.findById(req.params.id).populate('tasks')
    .then(response => {
      res.status(200).json(response);
    })
    .catch(err => {
      res.json(err);
    })
})

// PUT route => to update a specific project
router.put('/projects/:id', (req, res, next)=>{

  if(!mongoose.Types.ObjectId.isValid(req.params.id)) {
    res.status(400).json({ message: 'Specified id is not valid' });
    return;
  }

  Project.findByIdAndUpdate(req.params.id, req.body)
    .then(() => {
      res.json({ message: `Project with ${req.params.id} is updated successfully.` });
    })
    .catch(err => {
      res.json(err);
    })
})

// DELETE route => to delete a specific project
router.delete('/projects/:id', (req, res, next)=>{

  if(!mongoose.Types.ObjectId.isValid(req.params.id)) {
    res.status(400).json({ message: 'Specified id is not valid' });
    return;
  }

  Project.findByIdAndRemove(req.params.id)
    .then(() => {
      res.json({ message: `Project with ${req.params.id} is removed successfully.` });
    })
    .catch( err => {
      res.json(err);
    })
})

module.exports = router;
```

Acabamos de utilizar 3 métodos Mongoose integrados para lograr lo que necesitábamos:

     findById() para obtener el project especificado,
     findByIdAndUpdate() para actualizar el project especificado y
     findByIdAndRemove() para eliminar el project especificado.

Probemos project-routes en Postman.

Al principio, creamos dos archivos dentro de la carpeta de rutas. Uno fue project-routes.js, que llenamos con rutas, y ahora haremos lo mismo para task-routes.js:

```js
// routes/task-routes.js

const express = require("express");
const mongoose = require("mongoose");
const Task = require("../models/task");
const Project = require("../models/project");

const router = express.Router();

// GET route => to retrieve a specific task
router.get("/projects/:projectId/tasks/:taskId", (req, res, next) => {
  Task.findById(req.params.taskId)
    .then(theTask => {
      res.json(theTask);
    })
    .catch(err => {
      res.json(err);
    });
});

// POST route => to create a new task
router.post("/tasks", (req, res, next) => {
  Task.create({
    title: req.body.title,
    description: req.body.description,
    project: req.body.projectID
  })
    .then(response => {
      Project.findByIdAndUpdate(req.body.projectID, {
        $push: { tasks: response._id }
      })
        .then(theResponse => {
          res.json(theResponse);
        })
        .catch(err => {
          res.json(err);
        });
    })
    .catch(err => {
      res.json(err);
    });
});

// PUT route => to update a specific task
router.put("/tasks/:id", (req, res, next) => {
  if (!mongoose.Types.ObjectId.isValid(req.params.id)) {
    res.status(400).json({ message: "Specified id is not valid" });
    return;
  }

  Task.findByIdAndUpdate(req.params.id, req.body)
    .then(() => {
      res.json({
        message: `Task with ${req.params.id} is updated successfully.`
      });
    })
    .catch(err => {
      res.json(err);
    });
});

// DELETE route => to delete a specific task
router.delete("/tasks/:id", (req, res, next) => {
  if (!mongoose.Types.ObjectId.isValid(req.params.id)) {
    res.status(400).json({ message: "Specified id is not valid" });
    return;
  }

  Task.findByIdAndRemove(req.params.id)
    .then(() => {
      res.json({
        message: `Task with ${req.params.id} is removed successfully.`
      });
    })
    .catch(err => {
      res.json(err);
    });
});

module.exports = router;
```

También asegúrese de requerir estas rutas en app.js:

```js
// app.js
...

// Here are routes:
...
app.use('/api', require('./routes/task-routes'));
```

Prueba todas las rutas a través de Postman. Cuando se asegure de que todo funciona correctamente, continúe con el siguiente paso para que podamos finalizar nuestro backend (al menos por ahora).

Enable CORS requests

Sabemos que usaremos esta API para solicitudes provenientes de una aplicación diferente. Para el desarrollo, utilizaremos el servidor de react, que se ejecuta en el mismo puerto que nuestra API, y eso es 3000, así que ahora cambiemos el puerto de nuestra API a 4000 en lugar de 3000. Vaya al archivo .env y cambie el puerto a 4000.

Por defecto, los navegadores bloquearán la comunicación entre las aplicaciones por razones de seguridad, por lo que debemos configurar nuestro servidor para permitirlas.

Afortunadamente, hay un node module que puede ayudarnos: CORS

Instalarlo con:

```
$ npm install cors
```

Y en app.js, impórtelo al principio con todas los demás imports y úselo hacia el final del archivo, justo antes de las rutas, donde verá: // ADD CORS SETTINGS HERE TO ALLOW CROSS-ORIGIN INTERACTION:

```js
// app.js

...

const cors = require('cors');

...

app.use(cors({
  credentials: true,
  origin: ['http://localhost:3000'] // <== this will be the URL of our React app (it will be running on port 3000)
}));

// ROUTES MIDDLEWARE STARTS HERE:
...
```

credentials entrarán en juego cuando presentemos a los usuarios y los origin points a las direcciones URL permitidas, en nuestro caso, es el servidor de React que se ejecuta en el puerto 3000. Como puede ver, el origen es el array, de modo que nos da la oportunidad de agregar tantas URL como las necesitemos.
