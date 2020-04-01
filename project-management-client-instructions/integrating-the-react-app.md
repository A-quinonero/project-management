Comencemos creando la aplicación React usando el comando CLI:

```
$ create-react-app project-management-client
```

Ya hemos actualizado el puerto donde se está ejecutando nuestro servidor; Lo cambiamos de 3000 a 4000, por lo que no necesitamos realizar ningún cambio en nuestro cliente. Resumiendo:

    project-management-server runs on http://localhost:4000
    project-management-client runs on http://localhost:3000

Para ejecutar nuestro servidor usamos npm run dev y para iniciar nuestro cliente, usamos npm start.

Basic Structure

Todavía no profundizaremos en las rutas, pero por ahora crearemos dos componentes, para crear un proyecto y mostrarlo, y vamos desde allí.

Utilizaremos axios para realizar la llamada a nuestro servidor y para ese propósito, necesitamos instalarlo (ejecute la siguiente línea en la terminal del cliente):

```
$ npm install axios
```

Recuerde importar axios en la parte superior de cada componente que se está comunicando con el backend.

```
import axios from 'axios';
```

Como sabemos que tendremos algunas rutas, instalemos react-router-dom aquí también:

```
$ npm install react-router-dom
```

Entonces, comencemos: dentro de la carpeta src crea la carpeta components y en ella crea la carpeta projects. Dentro de la carpeta projects, cree AddProject.js y ProjectList.js. Hasta ahora, tenemos esta estructura:

```
src
├── components
    └── projects
            └── AddProject.js
            └── ProjectList.js
```

Comenzaremos construyendo el componente `<AddProject />`. Este componente mostrará el formulario y se encargará de su envío. Al manejar el envío del formulario, nos referimos al uso de axios para llegar a una ruta back-end y entregar algunos datos enviados desde el frontend (o simplemente podemos decir que los envió el usuario después de completar el formulario y enviarlo). Así que hagamos esto:

```js
// components/projects/AddProject.js

import React, { Component } from "react";
import axios from "axios";

class AddProject extends Component {
  constructor(props) {
    super(props);
    this.state = { title: "", description: "" };
  }

  handleFormSubmit = event => {
    event.preventDefault();
    const title = this.state.title;
    const description = this.state.description;
    axios
      .post("http://localhost:4000/api/projects", { title, description })
      .then(() => {
        // this.props.getData();
        this.setState({ title: "", description: "" });
      })
      .catch(error => console.log(error));
  };

  handleChange = event => {
    const { name, value } = event.target;
    this.setState({ [name]: value });
  };

  render() {
    return (
      <div>
        <form onSubmit={this.handleFormSubmit}>
          <label>Title:</label>
          <input
            type="text"
            name="title"
            value={this.state.title}
            onChange={e => this.handleChange(e)}
          />
          <label>Description:</label>
          <textarea
            name="description"
            value={this.state.description}
            onChange={e => this.handleChange(e)}
          />

          <input type="submit" value="Submit" />
        </form>
      </div>
    );
  }
}

export default AddProject;
```

::: Nota al margen: Deje this.props.getData() comentado por ahora, pronto lo necesitaremos.

La mayor parte del código parece muy familiar, ¿verdad? Porque ya cubrimos los conceptos básicos del uso de formularios en React.
Importemos el componente `<AddProject />` a App.js y lo renderizaremos allí por ahora.

```js
// App.js
import React, { Component } from "react";
import "./App.css";
import AddProject from "./components/projects/AddProject";

class App extends Component {
  render() {
    return (
      <div className="App">
        <AddProject />
      </div>
    );
  }
}

export default App;
```

checkpoint

Ahora puede ir a http://localhost:3000 y verá un formulario allí. Asegúrese de que su servidor se esté ejecutando e intente agregar un nuevo proyecto. Verifique su base de datos y debería ver un nuevo project en la projects collection.
Entonces, cuando tenemos al menos un project en nuestra base de datos, procedamos a crear el componente `<ProjectList />`. Dentro de este componente usaremos axios para obtener todos los projects del backend.

```js
// components/projects/ProjectList.js

import React, { Component } from "react";
import axios from "axios";
import { Link } from "react-router-dom";

import AddProject from "./AddProject"; // <== !!!

class ProjectList extends Component {
  constructor() {
    super();
    this.state = { listOfProjects: [] };
  }

  getAllProjects = () => {
    axios.get(`http://localhost:4000/api/projects`).then(responseFromApi => {
      this.setState({
        listOfProjects: responseFromApi.data
      });
    });
  };

  componentDidMount() {
    this.getAllProjects();
  }

  render() {
    return (
      <div>
        <div>
          {this.state.listOfProjects.map(project => {
            return (
              <div key={project._id}>
                <Link to={`/projects/${project._id}`}>
                  <h3>{project.title}</h3>
                </Link>
                {/* <p style={{maxWidth: '400px'}} >{project.description} </p> */}
              </div>
            );
          })}
        </div>
        <div>
          <AddProject getData={() => this.getAllProjects()} /> {/* <== !!! */}
        </div>
      </div>
    );
  }
}

export default ProjectList;
```

Veamos qué es interesante aquí:

- utilizamos el método del ciclo de vida de componenteDidMount() para obtener los datos de la API (esta es una práctica muy estándar);
- usamos map() para enumerar los proyectos (no olvide dar a cada elemento el ID de la base de datos como clave con key = {project.\_id});
- usamos el componente `<Link />` de la biblioteca react-router-dom para poder cambiar dinámicamente la URL y, en nuestro caso, para ir a la página de detalles del proyecto (eso es lo siguiente en lo que debemos trabajar);
- vemos que el componente `<AddProject />` estará realmente anidado dentro de `<ProjectList />`, así que vamos y elimínelo de App.js y también dejemos de comentar this.props.getData() dentro de `<AddProject />`;
- comentamos la línea de project description ya que construiremos la página de detalles, la descripción será visible allí.

Antes de ir a http://localhost:3000 y ver la lista de proyectos, primero tenemos que configurar el Router. Si recuerdas, comenzamos el proceso configurando el núcleo de nuestra aplicación, index.js, donde representamos el componente madre <App />.

```js
// index.js

import React from "react";
import ReactDOM from "react-dom";
import "./index.css";
import App from "./App";
import * as serviceWorker from "./serviceWorker";
import { BrowserRouter as Router } from "react-router-dom"; // <== !!!

ReactDOM.render(
  <Router>
    <App />
  </Router>,
  document.getElementById("root")
);
serviceWorker.unregister();
```

Después de importar BrowserRouter, lo usamos envolviendo la etiqueta `<App />`.

En este momento, nuestro App.js se ve así:

```js
// App.js

import React, { Component } from "react";
import "./App.css";
import ProjectList from "./components/projects/ProjectList";

class App extends Component {
  render() {
    return (
      <div className="App">
        <ProjectList />
      </div>
    );
  }
}

export default App;
```

Agreguemos otro archivo en la carpeta de components/projects, un ProjectDetails.js y agreguemos solo estas dos líneas de código por ahora:

```js
// components/projects/ProjectDetails.js

import React, { Component } from 'react';
import axios from 'axios';
import { Link, Redirect } from 'react-router-dom';

class ProjectDetails extends Component {
    constructor(props){
        super(props);
        this.state = {};
    }

  render(){
    return <div>Welcome to project details page!</div>
  }
}

export default ProjectDetails;
```

Avanzando, dentro de la carpeta components creemos otra carpeta navbar y dentro de esta carpeta creemos Navbar.js.
Hasta ahora, tenemos esta estructura:

```
src
├── components
    └── projects
    ├       └── AddProject.js
    ├       └── ProjectList.js
    ├       └── ProjectDetails.js
    ├
    └── navbar
          └── Navbar.js
```

Nuestro Navbar.js tendrá algunos enlaces en el futuro, pero en este momento solo uno en realidad:

```js
// components/navbar/Navbar.js

import React from "react";
import { Link } from "react-router-dom";

const navbar = () => {
  return (
    <nav className="nav-style">
      <ul>
        <li>
          <Link to="/projects" style={{ textDecoration: "none" }}>
            Projects
          </Link>
        </li>
      </ul>
    </nav>
  );
};

export default navbar;
```

Ahora hemos configurado todo para poder ir a App.js para configurar algunas rutas allí:

```js
// App.js

import React, { Component } from "react";
import "./App.css";
import { Switch, Route } from "react-router-dom";

import ProjectList from "./components/projects/ProjectList";
import Navbar from "./components/navbar/Navbar";
import ProjectDetails from "./components/projects/ProjectDetails";

class App extends Component {
  render() {
    return (
      <div className="App">
        <Navbar />
        <Switch>
          <Route exact path="/projects" component={ProjectList} />
          <Route exact path="/projects/:id" component={ProjectDetails} />
        </Switch>
      </div>
    );
  }
}

export default App;
```

¡Ok genial! Además, no se preocupe de que nuestra / route todavía no se use, la guardaremos para login/signup, que seguirá en la próxima lección.

Por ahora podemos: crear un nuevo proyecto, mostrar todos los proyectos y al hacer click en el título del proyecto, se nos redirige a la página de detalles del project que por el momento no tiene mucho código. ¡Agreguemos un poco más!

```js
// components/projects/ProjectDetails.js

import React, { Component } from "react";
import axios from "axios";
import { Link } from "react-router-dom";

class ProjectDetails extends Component {
  constructor(props) {
    super(props);
    this.state = {};
  }

  componentDidMount() {
    this.getSingleProject();
  }

  getSingleProject = () => {
    const { params } = this.props.match;
    axios
      .get(`http://localhost:4000/api/projects/${params.id}`)
      .then(responseFromApi => {
        const theProject = responseFromApi.data;
        this.setState(theProject);
      })
      .catch(err => {
        console.log(err);
      });
  };

  render() {
    return (
      <div>
        <h1>{this.state.title}</h1>
        <p>{this.state.description}</p>
        <Link to={"/projects"}>Back to projects</Link>
      </div>
    );
  }
}

export default ProjectDetails;
```

Checkpoint

Más o menos directo: componentDidMount() está ejecutando el método getSingleProject() que inicialmente se comunica con nuestra ruta de back-end a través de la llamada axios. Si todo tiene éxito, estamos actualizando el estado (usando nada más que setState ()) y equiparándolo al objeto del proyecto que obtuvimos de nuestra API. En la parte render(), accedemos a las propiedades del proyecto a través de this.state.title y this.state.description.

El próximo desafío es mostrar el formulario de edición y actualizar el proyecto. En primer lugar, crearemos un nuevo archivo dentro de la carpeta de components/projects: EditProject.js. Este componente tendrá el formulario que hará que onSubmit haga cambios en un project específico.

```js
// components/projects/EditProject.js

import React, { Component } from "react";
import axios from "axios";

class EditProject extends Component {
  constructor(props) {
    super(props);
    this.state = {
      title: this.props.theProject.title,
      description: this.props.theProject.description
    };
  }

  handleFormSubmit = event => {
    const title = this.state.title;
    const description = this.state.description;

    event.preventDefault();

    axios
      .put(`http://localhost:4000/api/projects/${this.props.theProject._id}`, {
        title,
        description
      })
      .then(() => {
        this.props.getTheProject();
        // after submitting the form, redirect to '/projects'
        this.props.history.push("/projects");
      })
      .catch(error => console.log(error));
  };

  handleChangeTitle = event => {
    this.setState({
      title: event.target.value
    });
  };

  handleChangeDesc = event => {
    this.setState({
      description: event.target.value
    });
  };

  render() {
    return (
      <div>
        <hr />
        <h3>Edit form</h3>
        <form onSubmit={this.handleFormSubmit}>
          <label>Title:</label>
          <input
            type="text"
            name="title"
            value={this.state.title}
            onChange={e => this.handleChangeTitle(e)}
          />
          <label>Description:</label>
          <textarea
            name="description"
            value={this.state.description}
            onChange={e => this.handleChangeDesc(e)}
          />

          <input type="submit" value="Submit" />
        </form>
      </div>
    );
  }
}

export default EditProject;
```

En este punto, estamos muy familiarizados con los formularios: ya que construimos con éxito el componente `<AddProject />`. Como puede ver, este componente depende en gran medida de las props que se pasan a este componente y veremos justo ahora dónde y por qué.
Entonces, veamos ahora, dónde debemos renderizar este componente. Podemos hacer eso aquí dentro del componente `<ProjectDetails />`. Realmente es cuestión de su preferencia: si lo desea, también puede hacer que el formulario se muestre en una página separada. Procederemos a configurar la funcionalidad de edit/update dentro del details component.

```js
// components/projects/ProjectDetails.js
...

import EditProject from './EditProject';

class ProjectDetails extends Component {

...

    renderEditForm = () => {
          if(!this.state.title){
            this.getSingleProject();
          } else {
          //{...props} => so we can have 'this.props.history' in Edit.js
            return <EditProject theProject={this.state} getTheProject={this.getSingleProject} {...this.props} />
          }
    }

   render(){
    return(
      <div>
        <h1>{this.state.title}</h1>
        <p>{this.state.description}</p>
        <div>{this.renderEditForm()} </div> {/* !!! */}
        <Link to={'/projects'}>Back to projects</Link>
      </div>
    )
    }
}
export default ProjectDetails;
```

Se llama al método renderEditForm() dentro del método render() y lo que hace es básicamente esto: comprueba si this.state tiene alguna propiedad (elegimos el título), y si eso es cierto, invoca el método getSingleProject() que obtiene el objeto project de nuestra API y lo establece en el estado del componente. En la próxima ejecución de renderEditForm(), está renderizando el componente `<EditProject />` con props pasados ​​a sí mismo. Aquí podemos ver que lo que se está pasando es theProject, que es realmente EL project que estamos viendo en esta página de detalles y estamos pasando el método getSingleProject() como props getTheProject. Ahora, si miras hacia atrás a nuestro componente `<EditProject />`, todo tiene más sentido, ¿verdad?

Estamos casi al final de la primera parte de nuestra aplicación. Solo agreguemos la funcionalidad de eliminación para que podamos tener CRUD completo.

Dentro del componente `<ProjectDetails />`, agregaremos el método deleteProject() que hará llamadas axios al backend y eliminará la ruta, y tendremos un botón que activará la ejecución de este método.
Nuestro componente `<ProjectDetails />` completo al final se ve así:

```js
// components/projects/ProjectDetails.js

import React, { Component } from 'react';
import axios from 'axios';
import { Link } from 'react-router-dom';
import EditProject from './EditProject';

class ProjectDetails extends Component {
  constructor(props){
    super(props);
    this.state = {};
  }

  componentDidMount(){
    this.getSingleProject();
  }

  getSingleProject = () => {
    const { params } = this.props.match;
    axios.get(`http://localhost:4000/api/projects/${params.id}`)
    .then( responseFromApi =>{
      const theProject = responseFromApi.data;
      this.setState(theProject);
    })
    .catch((err)=>{
        console.log(err)
    })
  }

  renderEditForm = () => {
    if (!this.state.title) {
      this.getSingleProject();
    } else {
      //{...props} => so we can have 'this.props.history' in Edit.js
      return (
        <EditProject
          theProject={this.state}
          getTheProject={this.getSingleProject}
          {...this.props}
        />
      );
    }
  };

// DELETE PROJECT:
  deleteProject = () => {
    const { params } = this.props.match;
    axios.delete(`http://localhost:4000/api/projects/${params.id}`)
    .then( () =>{
        this.props.history.push('/projects'); {/* !!! */}

    })
    .catch((err)=>{
        console.log(err)
    })
  }

  render(){
    return(
      <div>
        <h1>{this.state.title}</h1>
        <p>{this.state.description}</p>
        <div>{this.renderEditForm()} </div>
        <button onClick={() => this.deleteProject()}>Delete project</button> {/* <== !!! */}
        <br/>
        <Link to={'/projects'}>Back to projects</Link>
      </div>
    )
  }
}

export default ProjectDetails;
```

Projects done!!! CRUD completo funciona, así que pasemos a la parte de tasks. En la mayoría de los casos, tendremos la situación de tener que anidar componentes uno dentro del otro. Así que hagámoslo: nuestras tasks pertenecen a projects, por lo que tendremos que anidarlas dentro del componente project.
Para comenzar a trabajar en esta parte de la aplicación, creemos primero la carpeta de components/tasks y nuestro primer componente relacionado con la tarea: AddTask.js y agreguemos el siguiente código:

```js
// components/tasks/AddTask.js

import React, { Component } from 'react';
import axios from 'axios';

class AddTask extends Component {
  constructor(props){
      super(props);          
      this.state = { title: "", description: "", isShowing: false }; // will help us to toggle addtask form
  }

  handleFormSubmit = (event) => {
    event.preventDefault();
    const title = this.state.title;
    const description = this.state.description;
    const projectID = this.props.theProject._id; // <== we need to know to which project the created task belong, so we need to get its 'id' // it has to be the'id' because we are referencing project by its id in the task  model on the server side ( project: {type: Schema.Types.ObjectId, ref: 'Project'})

    // { title, description, projectID } => this is'req.body' that will be received on the server side in this route, so the names have to match
    axios.post("http://localhost:4000/api/tasks", {title, description, projectID })
    .then( () => {
          // after submitting the form, retrieve project one more time so the new task is displayed as well
        this.props.getTheProject();
        this.setState({title: "", description: ""});
    })
    .catch( error => console.log(error) )
  }

  handleChange = (event) => {
      const {name, value} = event.target;
      this.setState({[name]: value});
  }

  toggleForm = () => {
      if(!this.state.isShowing){
          this.setState({isShowing: true});
      } else {
        this.setState({isShowing: false});
      }
  }

  showAddTaskForm = () => {
    if(this.state.isShowing){
        return(
            <div>
                  <h3>Add Task</h3>
                  <form onSubmit={this.handleFormSubmit}>
                  <label>Title:</label>
                  <input type="text" name="title" value={this.state.title} onChange={ e => this.handleChange(e)}/>

<label>Description:</label>
                  <textarea name="description" value={this.state.description} onChange={ e => this.handleChange(e)} />

                  <input type="submit" value="Submit" />
                  </form>
            </div>
          )
    }
  }

  render(){
    return(
      <div>
            <hr />
            <button onClick={() => this.toggleForm()}> Add task </button>
            { this.showAddTaskForm() }
      </div>
    )
  }
}

export default AddTask;
```

Ahora, cuando creamos el componente AddTask, tenemos que ponerlo en algún lugar de nuestra aplicación para que el usuario pueda verlo. Hagámoslo en la página de detalles del project.

```js
// components/projects/ProjectDetails.js

...

// import the component:
import AddTask from '../tasks/AddTask';

class ProjectDetails extends Component {

  // after the delete project part add the following code:


  renderAddTaskForm = () => {
    if(!this.state.title){
        this.getSingleProject();
      } else {
                // pass the project and method getSingleProject() as a props down to AddTask component
        return <AddTask theProject={this.state} getTheProject={this.getSingleProject} />
      }
  }


  render(){
    return(
      <div>
        <h1>{this.state.title}</h1>
        <p>{this.state.description}</p>
        {/* show the task heading only if there are tasks */}
        { this.state.tasks && this.state.tasks.length > 0 && <h3>Tasks</h3> }
        {/* map through the array of tasks and... */}
        { this.state.tasks && this.state.tasks.map((task, index) => {
            return(
                <div key={ index }>
                {/* ... make each task's title a link that goes to the task details page */}
                    <Link to={`/projects/${this.state._id}/tasks/${task._id}`}>
                        { task.title }
                    </Link>
                </div>
            )

        }) }
        <div>{this.renderEditForm()} </div>
        <button onClick={() => this.deleteProject()}>Delete project</button> {/*<== !!! */}
        <br/>
        <div>{this.renderAddTaskForm()}
</div>

<br/><br/><br/><br/><br/>

        <Link to={'/projects'}>Back to projects</Link>
      </div>
    )
  }
}

export default ProjectDetails;
```

Además, enumeremos las tasks debajo del título de cada project en la página principal /projects también. El usuario no puede hacer click en ellos, esa funcionalidad ya se encuentra en la página de detalles del proyecto.

```js
// components/projects/ProjectList.js
...

class ProjectList extends Component {
  ...

   render(){
    return(
      <div>
        <div>
          { this.state.listOfProjects.map( project => {
            return (
              <div key={project._id}>
                <Link to={`/projects/${project._id}`}>
                  <h3>{project.title}</h3>
                </Link>
                {/*  added so the tasks can be displayed:   */}
                <ul>
                  { project.tasks.map((task, index) => {
                    return <li key={index}>{task.title}</li>
                  }) }
                </ul>
                {/* <p style={{maxWidth: '400px'}} >{project.description} </p> */}
              </div>
            )})
          }
        </div>
        <div>
            <AddProject getData={() => this.getAllProjects()}/> {/* <== !!! */}
        </div>
      </div>
    )
  }
}

export default ProjectList;
```

Terminaremos la lección mostrando la página de task details, así que creemos el archivo components/tasks/TaskDetails.js y agreguemos el siguiente fragmento de código:

```js
// components/tasks/TaskDetails.js

import React, { Component } from 'react';
import axios from 'axios';


class TaskDetails extends Component {
  constructor(props){
    super(props);
    this.state = {};
  }

  componentDidMount(){
    this.getTheTask();
  }

  getTheTask = () => {
    const { params } = this.props.match;

axios.get(`http://localhost:4000/api/projects/${params.id}/tasks/${params.taskId}`)

    .then( responseFromApi =>{
      const theTask = responseFromApi.data;
      this.setState(theTask);
    })
    .catch((err)=>{
        console.log(err)
    })
  }

  render(){
    return(
      <div>
        <h3>TASK DETAILS</h3>
        <h2>{this.state.title}</h2>
        <p>{this.state.description}</p>

        {/* To go back we can use react-router-dom method `history.goBack()` available on `props` object */}
        <button onClick={this.props.history.goBack}>Go Back</button>
      </div>
    )
  }
}

export default TaskDetails;
```

Y ahora tenemos que "decirle" a nuestra app cuándo representar este componente, así que abramos App.js, importemos el componente TaskDetails y agreguemos la siguiente ruta entre la etiqueta <Switch> como la tercera ruta:

```js
// App.js

...

import TaskDetails from './components/tasks/TaskDetails'; // <== import the TaskDetails component


class App extends Component {
  render() {
    return (
      <div className="App">
       <Navbar />
        <Switch>
          <Route exact path="/projects" component={ProjectList}/>
          <Route exact path="/projects/:id" component={ProjectDetails} />
          {/* added to display task details page: */}
          <Route exact path="/projects/:id/tasks/:taskId" component={TaskDetails} /> {/* <== !!! */}
        </Switch>
      </div>
    );
  }
}

export default App;
```

Le dejamos ahora a usted agregar funciones de edición y eliminación de las tasks. Para los aventureros, intente agregar la propiedad “done” para las tareas completadas.
