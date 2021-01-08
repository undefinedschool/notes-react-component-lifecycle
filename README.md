# React: Component Lifecycle

### 👉 Ver [todas las notas](https://github.com/undefinedschool/notes)


## Contenido

- [Intro](https://github.com/undefinedschool/notes-react-component-lifecycle/#intro)
- [Mounting](https://github.com/undefinedschool/notes-react-component-lifecycle/#mounting)
  - [`constructor()`](https://github.com/undefinedschool/notes-react-component-lifecycle/#constructor)
  - [`render()`](https://github.com/undefinedschool/notes-react-component-lifecycle/#render)
  - [`componentDidMount()`](https://github.com/undefinedschool/notes-react-component-lifecycle/#componentdidmount)
- [Updating](https://github.com/undefinedschool/notes-react-component-lifecycle/#updating)
  - [`componentDidUpdate()`](https://github.com/undefinedschool/notes-react-component-lifecycle/#componentdidupdate)
- [Unmounting](https://github.com/undefinedschool/notes-react-component-lifecycle/#unmounting)
  - [`componentWillUnmount()`](https://github.com/undefinedschool/notes-react-component-lifecycle/#componentwillunmount)

---

## Intro

Cada vez que nuestra app se ejecuta, los componentes pasan a través de un ciclo específico, que consiste en

1. el componente se agrega al DOM (_Mounting_)
2. el componente actualiza su state o recibe nuevas props (_Updating_)
3. el componente es removido del DOM (_Unmounting_)

En cada una de estas etapas del ciclo (que conocemos como _Component Lifecycle_) contamos con diferentes métodos para utilizar, que React va a ejecutar [siguiendo un orden específico](https://reactjs.org/docs/react-component.html#the-component-lifecycle).

## Mounting

### `constructor()`

Es el método que utilizamos para setear el _estado inicial_ del componente, algo que debe realizarle previo al renderizado del mismo.

```js
constructor(props) {
  super(props);

  this.state = {
    // ...
  }
}
```

### `render()`

Luego de setear el estado inicial con el constructor, el siguiente método en ejecutarse es `render()`. Este método va a retornar un _React Element_ (una representación liviana de un elemento del DOM), que luego React va a utilizar para renderizar el nodo correspondiente en el DOM.

> **Nota:** es importante tener en cuenta que `render()` tiene que ser una función _pura_, es decir, sólo va a retornar un elemento utilizando el state o las props del componente, pero no podemos, por ejemplo, hace un AJAX request dentro (sería un _side effect_).

```js
render() {
  return <h1>Hello, World!</h1>;
}
```

### `componentDidMount()`

Cómo hacemos entonces, si necesitamos ejecutar algún tipo de _side effect_ en nuestro componente para obtener ciertos datos? (AJAX request, agregar event listeners, hacer una query a la db, etc).

Para estos casos existe el método `componentDidMount()`. Este método se ejecuta la 1ra vez que el componente es montado en el DOM, por lo cual es un buen lugar para hacer este tipo de operaciones.

#### Ejemplo: AJAX Request (fetch)

```js
constructor(props) {
  super(props);

  this.state = {
    user: null;
  }
}

componentDidMount() {
  const { username } = this.props;
  
  fetch(username)
    .then(user => this.setState({
      user
    }))
}

render() {
  const { user } = this.state;
  
  return !user ? <p>Loading...</p> : <Dashboard data={user} />;
}
```

## Updating

En esta etapa del ciclo, las operaciones comunes son

- recibir nuevas props o actualizar el state, causando que el componente se vuelva a renderizar
- _re-fetch_ de los datos (AJAX), ver ejemplo debajo
- volver a setear algún listener

### `componentDidUpdate()`

Este método es invocado si el componente recibe algún update (nuevas props o cambio en el state), luego del 1er render. Podemos utilizarlo por ejemplo, cuando necesitamos volver a hacer un fetching de ciertos datos, sin tener que volver a montar el componente.

`componentDidUpdate()` recibe como parámetros las props y state previos, para comparar y decidir si es necesario hacer algo.

```js
componentDidUpdate(prevProps, prevState) {
  const { language } = this.props;
  
  if (language !== prevProps.language) {
    this.setState({repos: null});
  
    fetchRepos(language)
      .then(repos => {
        this.setState({ repos })
      });
  }
}
```

## Unmounting

En esta etapa, solemos hacer _limpieza_ (cleanup), por ejemplo, remover event listeners de componentes que ya no se encuentran montados en el DOM, para evitar memory leaks.

### `componentWillUnmount()`

Este método es invocado cuando el componente está por desmontarse del DOM. En el siguiente ejemplo, agregamos un event listener luego de montar el componente (`componentDidMount()`) y luego lo removemos.

```js
componentDidMount() {
  this.removeListener = listenToRepos(this.props.language, (repos) => {
    this.setState({ repos })
  });
}

componentWillUnmount() {
  this.removeListener();
}
```
