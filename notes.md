# Epic React

# React Hooks

useState nos permite pasar una función en lugar de un valor, eso hace que esa función se ejecute solo la primera vez que se carga el componente, evitando problemas de performance, ejemplo con localStorage.

```javascript
 const [name, setName] = React.useState(window.localStorage.getItem('name') || initialName)
```
Supongamos que queremos que se cargue como initial state algo que este en el localStorage, de la manera que esta arriba, lo que sucederá es que el localStorage se consultara en cada re renderizado del componente, lo que no tiene sentido.

En cambio, si pasamos una función se ejecutará solo en el primer render.

```javascript
 const [name, setName] = React.useState( () => window.localStorage.getItem('name') || initialName)
```

Esta funcionalidad es llamada lazy initialization.

<br/>

### Dependencias en useEffect

Las dependencias en el useEffect se comparan como un === por lo tanto si pongo objetos, arrays o funciones, para React es como si siempre cambiaran, hay que poner valores que puedan compararse

<br/>

### Hooks flow

![image](./hook-flow.png)

<br>


### Lifting state

Cuando un estado debe ser compartido por dos componentes que son hermanos, es decir que estan al mismo nivel, lo mejor es pasar ese estado al padre, a ese concepto se lo llama lefting state, que significaria llevarlo para arriba.


### Colocate state

Se refiere a lo contrario, dejar el estado lo mas cerca posible de donde sera utilizado, lo mismo con el context, a veces no es necesario que el contxt envuelva toda la app.

### Components composition

Antes de utilizar context, intentar hacer composition de componentes, que es muy eficaz para evitar pasar props innecesariamente, aprovechar que React permite pasar el children como prop para componet, por ejemplo:
 Imaginemos que nuestro componente dashbord tiene un dentro un dashboardContent que dentro tiene un dashboardHeader que usa el name del user para mostrar el menu. En lugar de pasar el user por 3 componentes hasta el lugar donde se usa, la composición ayudaria

```jsx
<Dashboard>
  <DashboardContent>
    <DashboardHeader user={currentUser}>
  </DashboardContent>
<Dashboard>

```

### Derive state

Muchas veces sucede que el primer instinto es tener un state para casa cosa que se nos ocurre que puede cambiar, el problema de eso es que a medida que las cosas crecen y se sigue agregando state es dificil de poder sincronizarlo y mantenerlo, por eso, siempre que se puede es mejor tener state derivado, que significa esto? que nuestro state se forme a partir de otro, por ejemplo:

```javascript
function Board() {
  const [squares, setSquares] = React.useState(Array(9).fill(null))
  const [nextValue, setNextValue] = React.useState(calculateNextValue(squares))
  const [winner, setWinner] = React.useState(calculateWinner(squares))
  const [status, setStatus] = React.useState(calculateStatus(squares))
  function selectSquare(square) {
    if (winner || squares[square]) {
      return
    }
    const squaresCopy = [...squares]
    squaresCopy[square] = nextValue
    const newNextValue = calculateNextValue(squaresCopy)
    const newWinner = calculateWinner(squaresCopy)
    const newStatus = calculateStatus(newWinner, squaresCopy, newNextValue)
    setSquares(squaresCopy)
    setNextValue(newNextValue)
    setWinner(newWinner)
    setStatus(newStatus)
  }
  // return beautiful JSX
}
```

En este caso tenemos bastante state que mantener, y al ejecutar una funcion estamos realizando 4 setStates distintos, una solucionmuy simple a este problema seria:

```javascript
function Board() {
  const [squares, setSquares] = React.useState(Array(9).fill(null))
  const nextValue = calculateNextValue(squares)
  const winner = calculateWinner(squares)
  const status = calculateStatus(winner, squares, nextValue)
  function selectSquare(square) {
    if (winner || squares[square]) {
      return
    }
    const squaresCopy = [...squares]
    squaresCopy[square] = nextValue
    setSquares(squaresCopy)
  }
  // return beautiful JSX
}
```

tener un unico state y los otros valores si vemos se estan calculando a partir de ese state, o sea que cada vez que ese state cambie, el componente se renderizara y los valores se volveran a calcular, sin preocuparnos por setear nosotros un monton de state y quitando complejidad.

### useRef

A veces necesitamos interactuar directamente con el dom, useRef nos da esa posibilidad, tambien useRef nos dara una referencia que a diferencia del state se mantiene siempre igual a traves de los renders, salvo que nosotros cambiemos el valor.


### useEffect

http requests es un side effect comun que tendremos en nuestra aplicación. No hay diferencia con otros side effects, como intereactuar con apis del navegador, por ejemplo localstorage. En todos los casos lo manejamos con el callback del hook `useEffect`. Esto nos asegura que cuando algo cambia, aplicaremos el side effect que queramos basados en esos cambios.
  Algo importante sobre el `useEffect` es que lo unico que podes retornar es una funcion de limpieza, esto tiene implicancias importantes al momento de usar async/await sintaxis.

```javascript
// Esto no funcionará
React.useEffect(async () => {
  const result = await doSomeAsyncThing()
  // hacer algo con el resultado
})
```

Esto no funciona porque cuando se hace una funcion async, automaticamente retorna una promesa, entonces si se quiere usar async/await esta es la forma:


```javascript
React.useEffect(() => {
  async function effect() {
    const result = await doSomeAsyncThing()
    // hacer algo con el resultado
  }
  effect()
})
```

### Manejar errores en request

Hay dos maneras de manejar los erroes en las promesas.

```javascript
// option 1: usando .catch
fetchPokemon(pokemonName)
  .then(pokemon => setPokemon(pokemon))
  .catch(error => setError(error))

// option 2: usando el segundo argunmento del then
fetchPokemon(pokemonName).then(
  pokemon => setPokemon(pokemon),
  error => setError(error),
)
```

Usando la opcion 1 del catch significa que atraparemos el error de la promesa `fetchPokemon` y tambien en caso de que falle el `setPokemon(pokemon)`.

La opcion 2, usando el segundo argumento del then, significa que manejaremos solo si `fetchPokemon` falla. En este caso sabemos que el setPokemon no arrojara un error, React maneja los errores y nos ofrece una manera de manejar esos errores, ver mas adelante `React Error Boundaries`


### Seteo de mas de un estado - inconvenientes

<br>

Vamos a imaginar que hacemos un request para traer info de un pokemon, tambien manejamos un estado de `status` para saber si debemos mostrar un loading, un error, o por ejemplo que indique el nombre que desea buscar, algo asi:


```javascript
  const [pokemon, setPokemon] = React.useState(null)
  const [error, setError] = React.useState(null)
  const [status, setStatus] = React.useState('idle')

  React.useEffect( () => {
    if(!pokemonName) return
    setPokemon(null)
    setStatus('pending')
    fetchPokemon(pokemonName).then(
      pokemonData => {
        setPokemon(pokemonData)
        setStatus('resolved')
      },
      error => {
        setError(error)
        setStatus('rejected')
      }
    )
  },[pokemonName])

```

Ahora bien, que sucede, imaginemos que en el then del fetchPokemon, ponemos primero el setStatus y luego el setPokemon, si bien React intentara hacer todos los cambios de estado de una para tener solo un re render, de hecho es lo que sucede con los eventos y en el useEffect callback (siempre que no sea una async function), si en el handle de un click o dentro de un useEffect haces mas de un seteo de estado, se traducira en un unico re render, pero en este caso, cada seteo de estado causara un re render, por lo tanto si cambiaramos el orden poniendo el setStatus, y en nuestro UI hay algo que depende del estado del pokemon puede romperse la UI, para esos casos lo mejor es manejar el estado como un objeto.

```javascript
setState({status: 'resolved', pokemon})
```

quedaria algo asi:

```javascript
  const [state, setState] = React.useState({ status: 'idle', pokemon: null, error: null})

  React.useEffect( () => {
    if(!pokemonName) return
    setState({status: 'pending', pokemon: null})
    fetchPokemon(pokemonName).then(
      pokemonData => setState({status: 'resolved', pokemon: pokemonData}),
      error => setState({status: 'rejected', pokemon: null, error}),
    )
  },[pokemonName])

```

### ErrorBoundary Component

Un error de JavaScript en una parte de la interfaz no debería romper toda la aplicación. Para resolver este problema, React 16 introduce el nuevo concepto de “Error boundaries”.

Los Error Boundaries funcionan como un bloque catch{} de JavaScript, pero para componentes. Sólo los componentes de clase (class components) pueden ser límites de errores. En la práctica, la mayor parte del tiempo declararás un límite de errores una vez y lo usarás a lo largo de tu aplicación.

Ten en cuenta que los límites de errores sólo capturan errores en los componentes bajo ellos en el árbol. Un límite de errores no puede capturar un error dentro de sí mismo. Si un límite de errores falla tratando de renderizar el mensaje de error, el error se propagará al límite de errores más cercano por encima de él. Esto también es similar al funcionamiento de los bloques catch{} en JavaScript.

Si un error ocurre y no ha sido manejado de alguna manera, React directamente remueve la Aplicacion de la pagina, dejando al usuario con una pantalla blanca.

Los ErrorBoundaries son si o si componentes de clase, ejemplo

```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Actualiza el estado para que el siguiente renderizado muestre la interfaz de repuesto
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // También puedes registrar el error en un servicio de reporte de errores
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // Puedes renderizar cualquier interfaz de repuesto
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}
```

```javascript
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>

```


Esta libreria nos da un ErrorBoundary component sin necesidad de tener que usar clases.

https://github.com/bvaughn/react-error-boundary


<br>
<hr>
<br>

# React Hooks Avanzados
<br>

### useReducer

Este hook es util cuando tenemos que manejar mucho estado y queremos por ejemplo extraerlo fuera del componente.

Ejemplo básico:

```javascript
const countReducer = (state, action) => {

  switch (action.type) {
    case 'INCREMENT':
      return {count: state.count + action.step }

    default:
      throw new Error(`Unsupported action type : ${action.type}`)
  }
}
function Counter({initialCount = 0, step = 2}) {
  const [state, dispatch] = React.useReducer(countReducer, {
    count: initialCount,
  })
  const {count} = state
  const increment = () => dispatch({type: 'INCREMENT', step})
  return <button onClick={increment}>{count}</button>
}
```

<br>

### useCallback

Imaginemos que tenemos el siguiente useEffect

```javascript
React.useEffect(() => {
  window.localStorage.setItem('count', count)
}, [count])
```
Si recordamos, la lista de dependencias en nuestro useEffect manejan de alguna manera cuando ese effect se ejecutara, si no le proveemos las dependencias React ejecutara el effect en cada re render.
 Ahora imaginemos que la funcion que se ejecutara en el useEffect viene como prop o bien como argumento en el caso que sea un custom hook.

  ejemplo:

```javascript
const updateLocalStorage = () => window.localStorage.setItem('count', count)
React.useEffect(() => {
  updateLocalStorage()
}, [updateLocalStorage]) // <-- function as a dependency
```

Aqui lo que sucede es que `updateLocalStorage` esta definida dentro del cuerpo de la funcion, que se re inicializa en cada render, lo que hace que por ser dependencia del useEffect ejecuta un nuevo render y entramos en un loop infinito.
  Aqui es donde el useCallback nos ayuda

  ```javascript
const updateLocalStorage = React.useCallback(
  () => window.localStorage.setItem('count', count),
  [count], // <-- yup! That's a dependency list!
)
React.useEffect(() => {
  updateLocalStorage()
}, [updateLocalStorage])
```

Lo que hara React es que si las dependencias del useCallbak no cambian, nos devolvera siempre la misma funcion en lugar de crear una nueva, en conseguencia nuestro effect no se ejecutara en un loop.


### useContext

Un contexto nos permite compartir ciertas variables de una manera global, es preferible usarlo como ultima opcion y no como primera.
Hay que considerar algo, cada vez que el context se actualice, todos aquellos componentes que esten consumiendo el context de alguna manera se re renderizaran

```javascript
const CountContext = React.createContext()

const CountProvider = (props) => {

  const [count, setCount] = React.useState(0)
  const [other] = React.useState('other')

  return (
    <CountContext.Provider value={{count, setCount, other}} {...props} />
  )

}


function App() {
  return (
    <div>
      <CountProvider>
        <CountDisplay />
        <Counter />
        <Test />
      </CountProvider>
    </div>
  )
}

export default App


```
<br>

### useLayoutEffect
Este hook se usa como useEffect, y probablemente el 99% de las veces se tenga que usar useEffect, pero a veces useLayoutEffect puede ser mejor para ciertas cosas puntuales.

Esto se ejecuta de forma sincrónica inmediatamente después de que React haya realizado todas las mutaciones del DOM. Esto puede ser útil si necesitas hacer mediciones del DOM (como obtener la posición de desplazamiento u otros estilos para un elemento) y luego hacer mutaciones del DOM o desencadenar un re-render sincrónico actualizando el estado.

En cuanto a la programación, esto funciona de la misma manera que componentDidMount y componentDidUpdate. Su código se ejecuta inmediatamente después de que el DOM haya sido actualizado, pero antes de que el navegador haya tenido la oportunidad de "pintar" esos cambios (el usuario no ve realmente las actualizaciones hasta después de que el navegador haya repintado).

Otra situación en la que podrías querer usar UseLayoutEffect en lugar de UseEffect es si estás actualizando un valor (como una ref) y quieres asegurarte de que está actualizado antes de que se ejecute cualquier otro código. Por ejemplo:

```javascript
const ref = React.useRef()
React.useEffect(() => {
  ref.current = 'some value'
})
// then, later in another hook or something
React.useLayoutEffect(() => {
  console.log(ref.current) // <-- this logs an old value because this runs first!
})
```

<br>

### useImperativeHandler
<br>

```javascript
function MyInput() {
  const inputRef = React.useRef()
  const focusInput = () => inputRef.current.focus()
  // where do I put the focusInput method??
  return <input ref={inputRef} />
}
```

En el ejemplo anterior no tendria manera de llamar a focusInput desde fuera del componente.

En realidad, obtendrá un error si intenta pasar una proposición `ref` a un componente de la función
de una función. Entonces, ¿cómo resolvemos esto? Bueno, React ha tenido esta característica llamada
`forwardRef` desde hace tiempo. Así que podemos hacer eso:

```javascript
const MyInput = React.forwardRef(function MyInput(props, ref) {
  const inputRef = React.useRef()
  ref.current = {
    focusInput: () => inputRef.current.focus(),
  }
  return <input ref={inputRef} />
})
```

Esto realmente funciona, sin embargo, hay algunos errores con este enfoque cuando se aplica en el futuro modo concurrente/suspenso de React (además no soporta callback refs). Así que en su lugar, usaremos el hook `useImperativeHandle` para hacer esto:

```javascript
const MyInput = React.forwardRef(function MyInput(props, ref) {
  const inputRef = React.useRef()
  React.useImperativeHandle(ref, () => {
    return {
      focusInput: () => inputRef.current.focus(),
    }
  })
  return <input ref={inputRef} />
})

```

### useDebugValue
<br>
Cuando comenzamos a crear nuestros propios hooks, puede ser util diferenciar su uso en algun componente, aca es donde este hook entra en accion.
 Este hook puede usarse solo en custom hooks, no en componentes.


```javascript
function useCount({initialCount = 0, step = 1} = {}) {
  React.useDebugValue({initialCount, step})
  const [count, setCount] = React.useState(0)
  const increment = () => setCount(c => c + 1)
  return [count, increment]
}
```

Con esto, cuando alguien use nuestro hook, podra ver el initialCount y step desde las devtools.


Tambien acepta una funcion de formateo en caso de que sea necesario.

```javascript
const formatCountDebugValue = ({initialCount, step}) =>
  `init: ${initialCount}; step: ${step}`

function useCount({initialCount = 0, step = 1} = {}) {
  React.useDebugValue({initialCount, step}, formatCountDebugValue)
  const [count, setCount] = React.useState(0)
  const increment = () => setCount(c => c + 1)
  return [count, increment]
}
```


<br>
<br>

# Advanced Patterns

<br>
<br>

### Context Module Functions
<br>

Este patron puede ayudar a evitar duplicaciones y problemas de performance, pero no es valido para ser usado todo el tiempo porque es un overkill en muchas situaciones.

Por ejemplo tenemos este caso donde hay un reducer y un contexto, que nos permite hacer uso de la funcion dispatch de ese reducer a traves de un custom hook


```javascript
// src/context/counter.js
const CounterContext = React.createContext()

function CounterProvider({step = 1, initialCount = 0, ...props}) {
  const [state, dispatch] = React.useReducer(
    (state, action) => {
      const change = action.step ?? step
      switch (action.type) {
        case 'increment': {
          return {...state, count: state.count + change}
        }
        case 'decrement': {
          return {...state, count: state.count - change}
        }
        default: {
          throw new Error(`Unhandled action type: ${action.type}`)
        }
      }
    },
    {count: initialCount},
  )

  const value = [state, dispatch]
  return <CounterContext.Provider value={value} {...props} />
}

function useCounter() {
  const context = React.useContext(CounterContext)
  if (context === undefined) {
    throw new Error(`useCounter must be used within a CounterProvider`)
  }
  return context
}

export {CounterProvider, useCounter}
```

```javascript
// src/screens/counter.js
import {useCounter} from 'context/counter'

function Counter() {
  const [state, dispatch] = useCounter()
  const increment = () => dispatch({type: 'increment'})
  const decrement = () => dispatch({type: 'decrement'})
  return (
    <div>
      <div>Current Count: {state.count}</div>
      <button onClick={decrement}>-</button>
      <button onClick={increment}>+</button>
    </div>
  )
}
```

```javascript
// src/index.js
import {CounterProvider} from 'context/counter'

function App() {
  return (
    <CounterProvider>
      <Counter />
    </CounterProvider>
  )
}
```


Si prestamos atencion notaremos que el componente esta creando las funciones increment y decrement que al final de cuentas terminan llamando al dispatch del reducer, no esta muy copado eso, mas por ejemplo si tenemos que empezar a llamar el dispatch un monton de veces.
  La primera inclinación que uno tendria seria decir, ok, paso esas cosas a un helper y listo.


```javascript
const increment = React.useCallback(() => dispatch({type: 'increment'}), [
  dispatch,
])
const decrement = React.useCallback(() => dispatch({type: 'decrement'}), [
  dispatch,
])
const value = {state, increment, decrement}
return <CounterContext.Provider value={value} {...props} />

// now users can consume it like this:

const {state, increment, decrement} = useCounter()
```

Esta no es una mala solución en si misma, pero lo que hace Facebook un monton en su codigo es usar helpers que acepten el dispatch como parametro.


```javascript
// src/context/counter.js
const CounterContext = React.createContext()

function CounterProvider({step = 1, initialCount = 0, ...props}) {
  const [state, dispatch] = React.useReducer(
    (state, action) => {
      const change = action.step ?? step
      switch (action.type) {
        case 'increment': {
          return {...state, count: state.count + change}
        }
        case 'decrement': {
          return {...state, count: state.count - change}
        }
        default: {
          throw new Error(`Unhandled action type: ${action.type}`)
        }
      }
    },
    {count: initialCount},
  )

  const value = [state, dispatch]

  return <CounterContext.Provider value={value} {...props} />
}

function useCounter() {
  const context = React.useContext(CounterContext)
  if (context === undefined) {
    throw new Error(`useCounter must be used within a CounterProvider`)
  }
  return context
}

const increment = dispatch => dispatch({type: 'increment'})
const decrement = dispatch => dispatch({type: 'decrement'})

export {CounterProvider, useCounter, increment, decrement}
```

```javascript
// src/screens/counter.js
import {useCounter, increment, decrement} from 'context/counter'

function Counter() {
  const [state, dispatch] = useCounter()
  return (
    <div>
      <div>Current Count: {state.count}</div>
      <button onClick={() => decrement(dispatch)}>-</button>
      <button onClick={() => increment(dispatch)}>+</button>
    </div>
  )
}
```

<br>
<br>

### Compound Components

Los componentes compuestos son aquellos que funcionan juntos, y que por separado no tienen ninguna funcion útil, es por ejemplo lo que pasa con el select y options.

```html
<select>
  <option value="1">Option 1</option>
  <option value="2">Option 2</option>
</select>
```

El select es el responsable de manejar el estado de la UI, y las options son elemenentos de configuracion de como el select debe operar (especificamente que opciones estan disponibles y cuales son sus valores)

Imaginemos que queremos implementar un select con options, una manera un poco ingenua de hacerlo seria asi:

```jsx
<CustomSelect
  options={[
    {value: '1', display: 'Option 1'},
    {value: '2', display: 'Option 2'},
  ]}
/>
```
Esto si bien funciona, no es muy extendible como lo es un componente compuesto. Por ejemplo, que pasa si queremos agregar mas atributos para el option para renderizar, o que el display cambie en base a lo qeu esta seleccionado?, tendremos que actualizar la api de nuesto componente, lo que es mas trabajo para nosotros y mas trabajo para el usuario que debe aprender a usar el componente y sus posibilidades. Ahi es donde los componentes compuestos son realmente útiles.

Cada componente re utilizable empieza siendo algo simple para un caso de uso especifico. Y hay que intentar no sobre complicarse si no es necesario e intentar de resolver problemas que aun no existen. Pero cuando los cambios empiezan a aparecer vamos a querer que la implementación de nuestros componente sea flexible y cambiable.

React permite clonar los hijos que se recibem y de esa manera agregarles propiedades, por ejemplo.

```javascript
function Foo({children}) {
  return React.Children.map(children, (child, index) => {
    return React.cloneElement(child, {
      id: `i-am-child-${index}`,
    })
  })
}

function Bar() {
  return (
    <Foo>
      <div>I will have id "i-am-child-0"</div>
      <div>I will have id "i-am-child-1"</div>
      <div>I will have id "i-am-child-2"</div>
    </Foo>
  )
}
```


Ejmplo de un toggle con este patron.

Este seria el componente original sin el patron aplicado.

```javascript

function Toggle() {
  const [on, setOn] = React.useState(false)
  const toggle = () => setOn(!on)
  return <Switch on={on} onClick={toggle} />
}

```

Si bien en este caso tal vez aplicar este patron no es necesario, sirve a modo de ejemplo, en el componente original nuestro componente no es factible de customizar demasiado, si quisieramos agregar mas funcionalidad, deberiamos ir y modificar el componente.
 En cambio con la implementación de debajo, aplicando el patron, lo que sucede es que nuestro componente, clona todos los componentes children que recibe y les agregar el estado y la funcionalidad que modifica ese estado, entonces queda en manos del desarrollados que componentes hijos utilizar y como.


```javascript
import * as React from 'react'
import {Switch} from '../switch'

function Toggle(props) {
  const [on, setOn] = React.useState(false)
  const toggle = () => setOn(!on)

  return React.Children.map(props.children, child =>  {
    return React.cloneElement(child, {
      on,
      toggle
    })
  })
}

const ToggleOn = ({on, children}) => {
  return on && <>{children}</>
}

const ToggleOff = ({on, children}) => {
  return !on && <>{children}</>
}

const ToggleButton = ({on, toggle}) => {
  return <Switch on={on} onClick={toggle} />
}

function App() {
  return (
    <div>
      <Toggle>
        <ToggleOn>The button is on</ToggleOn>
        <ToggleOff>The button is off</ToggleOff>
        <ToggleButton />
      </Toggle>
    </div>
  )
}

export default App

```

Una salvedad al momento de clonar los children, si vamos a agrearle alguna prop hay que tener en cuenta que no podemos agregar props a elementos nativos del DOM, ejemplo, un input no puede tener un atributo toggle, o on. La solucion es agregar las props a aquellos componentes de react.

```javascript

function Toggle({children}) {
  const [on, setOn] = React.useState(false)
  const toggle = () => setOn(!on)
  return React.Children.map(children, child => {
    if(typeof child.type === 'function') {
      return React.cloneElement(child, {on, toggle})
    }
    return child
  })
}

```

<br>
<br>

### Flexible Compound Components
<br>
Con el patron anterior, donde clonamos elementos y pasamos props, solo podemos hacerlo sobre los hijos directos, pero dichas props no estaran presentes por ejemplo en el hijo de un hijo. Para que esos elementos tengan acceso podemos user context.


```javascript

import * as React from 'react'
import {Switch} from '../switch'

const ToggleContext = React.createContext()

function Toggle({onToggle, children}) {
  const [on, setOn] = React.useState(false)
  const toggle = () => setOn(!on)

 return <ToggleContext.Provider value={{on, toggle}}>{children}</ToggleContext.Provider>
}

function useToggle(){
  return React.useContext(ToggleContext)
}

function ToggleOn({children}) {
  const {on} = useToggle()

  return on ? children : null
}

function ToggleOff({children}) {
  const {on} = useToggle()
  return on ? null : children
}

function ToggleButton({...props}) {
  const {on, toggle} = useToggle()
  return <Switch on={on} onClick={toggle} {...props} />
}

function App() {
  return (
    <div>
      <Toggle>
        <ToggleOn>The button is on</ToggleOn>
        <ToggleOff>The button is off</ToggleOff>
        <div>
          <ToggleButton />
        </div>
      </Toggle>
    </div>
  )
}

export default App

```

<br>
<br>

### Props collection and props getters
<br>
Muchos componentes reutilizables y hooks que creamos tienen casos de usos comunes y seria bueno que sean faciles de usar de la manera correcta. Por ejemplo, imaginemos un toggle, que por cuestion de accesibilidad debemos pasarle una prop aria-pressed true o false si esta encendido o no, ademas del onClick. Si ese es un caso comun deberiamos facilitar la manera de usarlo.
Tal vez en este caso no son muchas las cosas que hay que recordar, pero en hooks complejos la lista de props que hay que considerar y recordar puede ser grande, entonces es una buena idea tomar los casos de uso comun y crear un objeto de props que la gente pueda hacerle spread sin preocuparse demasiado.

```javascript

function useToggle() {
  const [on, setOn] = React.useState(false)
  const toggle = () => setOn(!on)
  const togglerProps = {
    'aria-pressed': on,
    onClick: toggle
  }
  return {on, toggle, togglerProps}
}

function App() {
  const {on, togglerProps} = useToggle()
  return (
    <div>
      <Switch on={on} {...togglerProps} />
      <hr />
      <button aria-label="custom-button" {...togglerProps}>
        {on ? 'on' : 'off'}
      </button>
    </div>
  )
}
```
<br>

Toggle props ejemplo:
Partiendo del caso anterior, imaginemos el siguiente escenario

```javascript
function App() {
  const {on, togglerProps} = useToggle()
  return (
    <div>
      <Switch on={on} {...togglerProps} />
      <hr />
      <button aria-label="custom-button"
       {...togglerProps}
       onClick={() => console.log('some action')}
       >
        {on ? 'on' : 'off'}
      </button>
    </div>
  )
}
```

Queremos que en el onClick de nuestro boton suceda algo, por ejemplo llamar a analytics o lo que sea. En este caso estariamos sobre escribiendo el onClick que viene del hook useToogle, por ende, no se ejecutara lo que uno espera por parte del hook. Porsupuesto podemos arreglarlo cambiando el orden de las props, pero en ese caso nuestro onClick dejaria de funcionar.
<br>

Para solucionar el incoveniente mencionado, podemos hacer que el hook se encargue de componer todas las props que le pasemos junto con las que el mismo hook tiene, asegurándonos de que todo funcione como se espera.

```javascript

const callAll = (...fns) => (...args) => fns.forEach(fn => fn?.(...args))

function useToggle() {
  const [on, setOn] = React.useState(false)
  const toggle = () => setOn(!on)

  function getTogglerProps({onClick, ...props} = {}) {
    return {
      'aria-pressed': on,
      onClick: callAll(onClick, toggle),
      ...props,
    }
  }

  return {
    on,
    toggle,
    getTogglerProps,
  }
}

function App() {
  const {on, getTogglerProps} = useToggle()
  return (
    <div>
      <Switch {...getTogglerProps({on})} />
      <hr />
      <button
        {...getTogglerProps({
          'aria-label': 'custom-button',
          onClick: () => console.info('onButtonClick'),
          id: 'custom-button-id',
        })}
      >
        {on ? 'on' : 'off'}
      </button>
    </div>
  )
}

```
Es preferible usar props getters en lugar de props collection porque nos da mas flexibilidad.


<br>
<br>

### State reducer
Durante la vida de nuestros componentes reutilizables, estos son utlizados en muchas maneras y contextos diferentes, a medida que pasa el tiempo nos van solicitando que agreguemos mas features o caracteristicas a nuestros componentes para soportar diferentes escenarios.
  Podriamos agregar props a nuestros componentes y lógica a nuestro reducer para manejar los distintos casos, pero hay una lista interminable de customizaciones lógicas que los usuarios podrian querer, y no queres codear cada una de las posibilidades para los posibles casos.
En este ejemplo podemos ver que estabamos definiendo un reducer, pero a la vez damos la posibilidad al usuario de que pase un reducer al momento de iniciar el hook, dandole mucha flexibilidad, pero si no la necesita, simplemente el default reducer sera utilizado.


  ```javascript

const callAll = (...fns) => (...args) => fns.forEach(fn => fn?.(...args))

function toggleReducer(state, {type, initialState}) {
  switch (type) {
    case 'toggle': {
      return {on: !state.on}
    }
    case 'reset': {
      return initialState
    }
    default: {
      throw new Error(`Unsupported type: ${type}`)
    }
  }
}

function useToggle({initialOn = false, reducer = toggleReducer} = {}) {
  const {current: initialState} = React.useRef({on: initialOn})
  const [state, dispatch] = React.useReducer(reducer, initialState)
  const {on} = state

  const toggle = () => dispatch({type: 'toggle'})
  const reset = () => dispatch({type: 'reset', initialState})

  function getTogglerProps({onClick, ...props} = {}) {
    return {
      'aria-pressed': on,
      onClick: callAll(onClick, toggle),
      ...props,
    }
  }

  function getResetterProps({onClick, ...props} = {}) {
    return {
      onClick: callAll(onClick, reset),
      ...props,
    }
  }

  return {
    on,
    reset,
    toggle,
    getTogglerProps,
    getResetterProps,
  }
}

function App() {
  const [timesClicked, setTimesClicked] = React.useState(0)
  const clickedTooMuch = timesClicked >= 4

  function toggleStateReducer(state, action) {
    switch (action.type) {
      case 'toggle': {
        if (clickedTooMuch) {
          return {on: state.on}
        }
        return {on: !state.on}
      }
      case 'reset': {
        return {on: false}
      }
      default: {
        throw new Error(`Unsupported type: ${action.type}`)
      }
    }
  }

  const {on, getTogglerProps, getResetterProps} = useToggle({
    reducer: toggleStateReducer,
  })

  return (
    <div>
      <Switch
        {...getTogglerProps({
          disabled: clickedTooMuch,
          on: on,
          onClick: () => setTimesClicked(count => count + 1),
        })}
      />
      {clickedTooMuch ? (
        <div data-testid="notice">
          Whoa, you clicked too much!
          <br />
        </div>
      ) : timesClicked > 0 ? (
        <div data-testid="click-count">Click count: {timesClicked}</div>
      ) : null}
      <button {...getResetterProps({onClick: () => setTimesClicked(0)})}>
        Reset
      </button>
    </div>
  )
}
  ```


<br>
<br>

### Control Props
<br>

A veces, las personas quieren poder manejar el estado interno de nuestro componente desde afuera. el State Reducer nos permite manejar que estado cambia cuando un cambio de estado sucede, pero a veces quien usa el componente quieren cambiar el estado ellos mismos. Podemos permitir eso con una feature llamada "Control Props".
 Basicamente es el mismo concepto que manejamos en React para los elemenos del form.


```javascript
function MyCapitalizedInput() {
  const [capitalizedValue, setCapitalizedValue] = React.useState('')

  return (
    <input
      value={capitalizedValue}
      onChange={e => setCapitalizedValue(e.target.value.toUpperCase())}
    />
  )
}
```

En este caso el componente que esta implementando "control props" es el input. Normalmente el input controla el estado por si solo (cuando renderizamos el input sin especificarle un value). Pero una vez que agregamos la prop 'value', seremos los responsables de actualizar ese valor a traves del onChange y el uso de estado.
  Esa flexibilidad nos permite cambiar el estado programaticamente, por ejemplo, para sincronizar valores.


  ```javascript
function MyTwoInputs() {
  const [capitalizedValue, setCapitalizedValue] = React.useState('')
  const [lowerCasedValue, setLowerCasedValue] = React.useState('')

  function handleInputChange(e) {
    setCapitalizedValue(e.target.value.toUpperCase())
    setLowerCasedValue(e.target.value.toLowerCase())
  }

  return (
    <>
      <input value={capitalizedValue} onChange={handleInputChange} />
      <input value={lowerCasedValue} onChange={handleInputChange} />
    </>
  )
}
```


Ejemplo:


```javascript

const callAll = (...fns) => (...args) => fns.forEach(fn => fn?.(...args))

const actionTypes = {
  toggle: 'toggle',
  reset: 'reset',
}

function toggleReducer(state, {type, initialState}) {
  switch (type) {
    case actionTypes.toggle: {
      return {on: !state.on}
    }
    case actionTypes.reset: {
      return initialState
    }
    default: {
      throw new Error(`Unsupported type: ${type}`)
    }
  }
}

function useToggle({
  initialOn = false,
  reducer = toggleReducer,
  onChange,
  on: controlledOn,
} = {}) {
  const {current: initialState} = React.useRef({on: initialOn})
  const [state, dispatch] = React.useReducer(reducer, initialState)
  const onIsControlled = controlledOn != null
  const on = onIsControlled ? controlledOn : state.on

  function dispatchWithOnChange(action) {
    if (!onIsControlled) {
      dispatch(action)
    }
    onChange?.(reducer({...state, on}, action), action)
  }

  const toggle = () => dispatchWithOnChange({type: actionTypes.toggle})
  const reset = () =>
    dispatchWithOnChange({type: actionTypes.reset, initialState})

  function getTogglerProps({onClick, ...props} = {}) {
    return {
      'aria-pressed': on,
      onClick: callAll(onClick, toggle),
      ...props,
    }
  }

  function getResetterProps({onClick, ...props} = {}) {
    return {
      onClick: callAll(onClick, reset),
      ...props,
    }
  }

  return {
    on,
    reset,
    toggle,
    getTogglerProps,
    getResetterProps,
  }
}

function Toggle({on: controlledOn, onChange}) {
  const {on, getTogglerProps} = useToggle({on: controlledOn, onChange})
  const props = getTogglerProps({on})
  return <Switch {...props} />
}

function App() {
  const [bothOn, setBothOn] = React.useState(false)
  const [timesClicked, setTimesClicked] = React.useState(0)

  function handleToggleChange(state, action) {
    if (action.type === actionTypes.toggle && timesClicked > 4) {
      return
    }
    setBothOn(state.on)
    setTimesClicked(c => c + 1)
  }

  function handleResetClick() {
    setBothOn(false)
    setTimesClicked(0)
  }

  return (
    <div>
      <div>
        <Toggle on={bothOn} onChange={handleToggleChange} />
        <Toggle on={bothOn} onChange={handleToggleChange} />
      </div>
      {timesClicked > 4 ? (
        <div data-testid="notice">
          Whoa, you clicked too much!
          <br />
        </div>
      ) : (
        <div data-testid="click-count">Click count: {timesClicked}</div>
      )}
      <button onClick={handleResetClick}>Reset</button>
      <hr />
      <div>
        <div>Uncontrolled Toggle:</div>
        <Toggle
          onChange={(...args) =>
            console.info('Uncontrolled Toggle onChange', ...args)
          }
        />
      </div>
    </div>
  )
}
```

<br>
<br>
<br>

# Performance
<br>
<br>

## Code Splitting

Te permite cargar el codigo solo cuando se necesita. Imaginemos que mostraremos algo pesado de cargar al momento de hacer un click en un boton, no es necesario cargarlo antes del click, por lo que podriamos hacer un dynamic import para cargar el codigo solo cuando lo necesitemos.

```javascript
import('/some-module.js').then(
  module => {
    // do stuff with the module's exports
  },
  error => {
    // there was some error loading the module...
  },
)
```

React tiene soporte para cargar modulos como si fueran React components. El modulo debe tener un React component como default export y es necesario usaer React Suspense para renderizar un fallback mientras el componente se carga.

```javascript
// smiley-face.js
import * as React from 'react'

function SmileyFace() {
  return <div>😃</div>
}

export default SmileyFace

// app.js
import * as React from 'react'

const SmileyFace = React.lazy(() => import('./smiley-face'))

function App() {
  return (
    <div>
      <React.Suspense fallback={<div>loading...</div>}>
        <SmileyFace />
      </React.Suspense>
    </div>
  )
}
```

otro ejemplo:

```javascript
import * as React from 'react'

const Globe = React.lazy(() => import('../globe'))

function App() {
  const [showGlobe, setShowGlobe] = React.useState(false)

  return (
    <div
      style={{
        display: 'flex',
        alignItems: 'center',
        flexDirection: 'column',
        justifyContent: 'center',
        height: '100%',
        padding: '2rem',
      }}
    >
      <label style={{marginBottom: '1rem'}}>
        <input
          type="checkbox"
          checked={showGlobe}
          onChange={e => setShowGlobe(e.target.checked)}
        />
        {' show globe'}
      </label>
      <div style={{width: 400, height: 400}}>
        <React.Suspense fallback={<div>loading globe...</div>}>
          {showGlobe ? <Globe /> : null}
        </React.Suspense>
      </div>
    </div>
  )
}

export default App
```


En el caso anterior se esta cargando un globo terraque cuando se hace click en un checkbox, ahora bien, imaginemos que el 90% del tiempo la gente ingresa a la pagina para ver ese globo, no queremos que primero espere para cargar la pagina y luego para cargar el globo. Seria bueno empezar a cargar el globo tan pronto el usuario hace un hover.


```javascript

import * as React from 'react'

const loadGlobe = () => import('../globe')
const Globe = React.lazy(loadGlobe)

function App() {
  const [showGlobe, setShowGlobe] = React.useState(false)

  return (
    <div
      style={{
        display: 'flex',
        alignItems: 'center',
        flexDirection: 'column',
        justifyContent: 'center',
        height: '100%',
        padding: '2rem',
      }}
    >
      <label
        style={{marginBottom: '1rem'}}
        onMouseEnter={loadGlobe}
        onFocus={loadGlobe}
      >
        <input
          type="checkbox"
          checked={showGlobe}
          onChange={e => setShowGlobe(e.target.checked)}
        />
        {' show globe'}
      </label>
      <div style={{width: 400, height: 400}}>
        <React.Suspense fallback={<div>loading globe...</div>}>
          {showGlobe ? <Globe /> : null}
        </React.Suspense>
      </div>
    </div>
  )
}

export default App

```


Si estas utilizando webpack para el bundle, podes usar webpack magic component para hacer prefetch de dynamics imports (https://webpack.js.org/api/module-methods/#magic-comments)


```javascript
import(/* webpackPrefetch: true */ './some-module.js')
```

Webpack agregara al head

```javascript
<link rel="prefetch" as="script" href="/static/js/1.chunk.js">
```
De esta manera el browser el browser hara el load del js dentro del browser cache, estando disponible por adelantado.

Otra cosa que se puede chequear es 'webpackChunkName', usarlo, permitirá a webpack poner módulos comunes en el mismo chunk. Esto es bueno para componentes que queremos cargar juntos en el mismo chunk (para reducir múltiples requests para múltiples módulos que necesitaremos que estén juntos.)



<br>
<br>

## useMemo

Este hook nos permite memorizar los resultados de cálculos que pueden ser pesados de calcular. Pensemos que si es una funcion que esta en nuestro componente se ejecutara en cada re render.

```jsx
function Distance({x, y}) {
  const distance = calculateDistance(x, y)
  return (
    <div>
      The distance between {x} and {y} is {distance}.
    </div>
  )
}
```

Si el componente padre se re renderiza, o si agregamos algo de estado y este cambia, el componente se estaria re renderizando, realizando el cálculo de la distancia en cada re render.

```jsx
function Distance({x, y}) {
  const distance = React.useMemo(() => calculateDistance(x, y), [x, y])
  return (
    <div>
      The distance between {x} and {y} is {distance}.
    </div>
  )
}
```

El uso del hook nos permite re calcular la distancia solo cuando el valor x o y cambia, y no en cada re render.

<br>
<br>

## React Memo

Un componente se re renderiza por los siguientes motivos:
- Alguna de sus props cambia
- Cambia un estado interno del componente
- Esta consumiendo velues de un contexto que cambiaron
- Su padre se re renderizo

Si bien React es muy rápido, a veces puede ser util indicarle a React que cierta parte del arbol se re renderice solo si el estado cambio.
 Para lograr eso react nos ofrece 'React.PureComponent', 'shouldComponentUpdate', estos dos para class components y 'React.memo' para functional components.
 Debemos usar estas herramientas solo cuando sean necesarias, y no andar colocando React.memo en cada componente.
 Aqui un ejemplo, aplicando React.memo en este caso, evitamos que el Input se re renderice cuando el count cambia.

```jsx
function CountButton({count, onClick}) {
  return <button onClick={onClick}>{count}</button>
}
CountButton = React.memo(CountButton)

function NameInput({name, onNameChange}) {
  return (
    <label>
      Name: <input value={name} onChange={e => onNameChange(e.target.value)} />
    </label>
  )
}
NameInput = React.memo(NameInput)


function Example() {
  const [name, setName] = React.useState('')
  const [count, setCount] = React.useState(0)
  const increment = () => setCount(c => c + 1)
  return (
    <div>
      <div>
        <CountButton count={count} onClick={increment} />
      </div>
      <div>
        <NameInput name={name} onNameChange={setName} />
      </div>
      {name ? <div>{`${name}'s favorite number is ${count}`}</div> : null}
    </div>
  )
}

```

React.memo nos permite usar un comparador custom para que nosotros decidamos cuando el componente se deberia re renderizar.



```javascript

function ListItem({
  getItemProps,
  item,
  index,
  selectedItem,
  highlightedIndex,
  ...props
}) {
  const isSelected = selectedItem?.id === item.id
  const isHighlighted = highlightedIndex === index
  return (
    <li
      {...getItemProps({
        index,
        item,
        style: {
          fontWeight: isSelected ? 'bold' : 'normal',
          backgroundColor: isHighlighted ? 'lightgray' : 'inherit',
        },
        ...props,
      })}
    />
  )
}
ListItem = React.memo(ListItem, (prevProps, nextProps) => {
  // true means do NOT rerender
  // false means DO rerender

  // these ones are easy if any of these changed, we should re-render
  if (prevProps.getItemProps !== nextProps.getItemProps) return false
  if (prevProps.item !== nextProps.item) return false
  if (prevProps.index !== nextProps.index) return false
  if (prevProps.selectedItem !== nextProps.selectedItem) return false

  // this is trickier. We should only re-render if this list item:
  // 1. was highlighted before and now it's not
  // 2. was not highlighted before and now it is
  if (prevProps.highlightedIndex !== nextProps.highlightedIndex) {
    const wasPrevHighlighted = prevProps.highlightedIndex === prevProps.index
    const isNowHighlighted = nextProps.highlightedIndex === nextProps.index
    return wasPrevHighlighted === isNowHighlighted
  }
  return true
})

```

Algo importante, si a nuestro componente que wrappeamos con React.memo le pasamos valores primitivos, React se encargara del cálculo y no hace falta que hagamos un comparador custom.
 En el caso anterior seria, pasar 'isSelected' y 'isHighlighted' como prop en lugar de calcularlo.

<br>
<br>


## Optimizar values del contexto.


```jsx
const CountContext = React.createContext()

function CountProvider(props) {
  const [count, setCount] = React.useState(0)
  const value = [count, setCount]
  return <CountContext.Provider value={value} {...props} />
}
```
Cada vez que el CountProvider es re renderizado the value object es uno nuevo, por ende, aunque el count no haya cambiado, todos los componentes que lo consumen seran re renderizados.
 Una manera facil de solucionar este problema es usando useMemo para el value del contexto.

 ```jsx
const CountContext = React.createContext()

function CountProvider(props) {
  const [count, setCount] = React.useState(0)
  const value = React.useMemo(() => [count, setCount], [count])
  return <CountContext.Provider value={value} {...props} />
}
```


Otra opcion para mejorar la performance con los contexts, es separándolos, tal vez un componente haga uso del dispatch del context anterior, pero no del state o viceverza.


```javascript

function AppProvider({children}) {
  const [state, dispatch] = React.useReducer(appReducer, {
    dogName: '',
    grid: initialGrid,
  })
  return (
    <AppStateContext.Provider value={state}>
      <AppDispatchContext.Provider value={dispatch}>
        {children}
      </AppDispatchContext.Provider>
    </AppStateContext.Provider>
  )
}

function useAppState() {
  const context = React.useContext(AppStateContext)
  if (!context) {
    throw new Error('useAppState must be used within the AppProvider')
  }
  return context
}

function useAppDispatch() {
  const context = React.useContext(AppDispatchContext)
  if (!context) {
    throw new Error('useAppDispatch must be used within the AppProvider')
  }
  return context
}

```



<br>
<br>


## Slice app state

Algo que podemos hacer para mejorar la performance, por ejemplo en aquellos casos que se consume state de un context y tal vez eso hace que un monton de componentes se re rendericen sin necesidad, es hacer un slice del state que necesitamos con un HOC (high order componet).

![image](./grid.png)

Esta grilla por ejemplo, cada una de las celdas tiene un evento onClick, que despacha un action en nuestro reducer de un context, por lo que hace que todas las celdas se re rendericen porque cada una de ellas estan haciendo uso del state del context.
  En esta situacion se puede hacer un HOC que haga de wrap del componente, donde se accede al state deseado y se pasa como prop al componente wrappeado.
Este patron es el que usa react-redux con su connect para acceder a las partes del store desesadas.

```javascript

function withStateSlice(Comp, slice) {
  const MemoComp = React.memo(Comp)
  function Wrapper(props, ref) {
    const state = useAppState()
    const cell = slice(state, props)
    return <MemoComp ref={ref} state={cell} {...props} />
  }
  Wrapper.displayName = `withStateSlice(${Comp.displayName || Comp.name})`
  return React.memo(React.forwardRef(Wrapper))
}

function Cell({state: cell, row, column}) {
  const dispatch = useAppDispatch()
  const handleClick = () => dispatch({type: 'UPDATE_GRID_CELL', row, column})
  return (
    <button
      className="cell"
      onClick={handleClick}
      style={{
        color: cell > 50 ? 'white' : 'black',
        backgroundColor: `rgba(0, 0, 0, ${cell / 100})`,
      }}
    >
      {Math.floor(cell)}
    </button>
  )
}
Cell = withStateSlice(Cell, (state, {row, column}) => state.grid[row][column])


```

<br>
<br>
<br>

# Testing
<br>
<br>

### React testing library

Ejemplo básico

```javascript
import * as React from 'react'
import {render, screen} from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import Counter from '../../components/counter'

test('counter increments and decrements when the buttons are clicked', () => {
  render(<Counter />)
  const increment = screen.getByRole('button', {name: /increment/i})
  const decrement = screen.getByRole('button', {name: /decrement/i})
  const message = screen.getByText(/current count/i)

  expect(message).toHaveTextContent('Current count: 0')
  userEvent.click(increment)
  expect(message).toHaveTextContent('Current count: 1')
  userEvent.click(decrement)
  expect(message).toHaveTextContent('Current count: 0')
})

```

### Detalles de implementación

Una de las cosas más importantes que hay que recordar a la  hora de probar nuestro software de la manera que se utiliza es evitar probar los detalles de implementación. "Detalles de implementación" es un término que se refiere a cómo una abstracción logra un determinado resultado. Gracias a la expresividad del código, normalmente se puede conseguir el mismo resultado utilizando detalles de implementación completamente diferentes. Por ejemplo:


```javascript
const multiply = (a, b) => a * b
```

vs

```javascript
function multiply(a, b) {
  let total = 0
  for (let i = 0; i < b; i++) {
    total += a
  }
  return total
}
```

Uno de ellos es más claro que el otro, pero eso es irrelevante para el punto:
La implementación de tus abstracciones no importa a los usuarios de tu abstracción y si quieres tener confianza en que sigue funcionando a través de a través de  efactorizaciones, entonces **tampoco deberían hacerlo tus pruebas.**

Ejemplo de React:

```javascript
function Counter() {
  const [count, setCount] = React.useState(0)
  const increment = () => setCount(c => c + 1)
  return <button onClick={increment}>{count}</button>
}
```

Esta sería una manera de acceder al button, para clickearlo y realizar el test

```javascript
const {container} = render(<Counter />)
container.firstChild // <-- that's the button
```

However, what if we changed it a bit:

```javascript
function Counter() {
  const [count, setCount] = React.useState(0)
  const increment = () => setCount(c => c + 1)
  return (
    <span>
      <button onClick={increment}>{count}</button>
    </span>
  )
}
```

Sin embargo, si lo cambiamos un poco el etiquetado:

```javascript
function Counter() {
  const [count, setCount] = React.useState(0)
  const increment = () => setCount(c => c + 1)
  return (
    <span>
      <button onClick={increment}>{count}</button>
    </span>
  )
}
```

Nuestro test se rompería, y la única diferencia existente entre los dos casos es lo que wrappea al button. Pero no hay diferencia en lo que el usuario ve o en como funciona el componente.
  Entonces, hay una mejor manera de acceder al button para hacer el test, libre de detalles de implementación, y si hiceramos un refactor, seguiría funcionando.


```javascript
render(<Counter />)
screen.getByText('0') // <-- that's the button
// or (even better) you can do this:
screen.getByRole('button', {name: '0'}) // <-- that's the button
```

Respecto a como hacer query de los elementos del dom(como tomarlos para el testing), hay un orden de prioridades recomendado que puede encontrarse aqui
 https://testing-library.com/docs/queries/about

Ademas esta página es buena referencia, donde pasamos html y nos dirá cual es el query mas recomendado.
https://testing-playground.com/


### Helpers

screen.debug() -> podes ver en consola que se renderiza.


### Test de un form, ejemplo

```javascript
import * as React from 'react'
import {render, screen} from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import Login from '../../components/login'

test('submitting the form calls onSubmit with username and password', () => {
  const handleSubmit = jest.fn()
  render(<Login onSubmit={handleSubmit} />)
  const username = 'chucknorris'
  const password = 'i need no password'

  userEvent.type(screen.getByLabelText(/username/i), username)
  userEvent.type(screen.getByLabelText(/password/i), password)
  userEvent.click(screen.getByRole('button', {name: /submit/i}))

  expect(handleSubmit).toHaveBeenCalledWith({
    username,
    password,
  })
  expect(handleSubmit).toHaveBeenCalledTimes(1)
})

```


### Mock requests

Probar que nuestro código del frontend interactúa con el backend es importante. Así es como el usuario utiliza nuestras aplicaciones, así que es lo que nuestras pruebas deben hacer también si queremos la máxima confianza. Sin embargo, hay varios desafíos que vienen con hacer eso. La configuración necesaria para que esto funcione no es trivial. Es importante, sin duda, que probemos esa integración, pero podemos hacerlo con un conjunto de pruebas E2E utilizando una herramienta como [Cypress](https://cypress.io).

Para nuestras pruebas de componentes de integración y de unidad usareos Jest, pondremos en marcha un servidor simulado para manejar todos los las peticiones de `window.fetch` que hagamos durante nuestras pruebas.

Para manejar estas peticiones fetch, vamos a iniciar un "servidor" que no es servidor, sino simplemente un interceptor de peticiones. Esto hace que sea muy fácil
de configurar las cosas (porque no tenemos que preocuparnos de encontrar un puerto disponible para que el servidor escuche y asegurarse de que estamos haciendo peticiones al puerto correcto) y también nos permite imitar las peticiones hechas a otros dominios.

Podemos usar la libreria [MSW](https://mswjs.io/)  para interceptar los requests e indicar que data devolver.

Ejemplo simple:

```javascript

import * as React from 'react'
import {render, screen, waitForElementToBeRemoved} from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import {build, fake} from '@jackfranklin/test-data-bot'
import {rest} from 'msw'
import {setupServer} from 'msw/node'
import Login from '../../components/login-submission'

const buildLoginForm = build({
  fields: {
    username: fake(f => f.internet.userName()),
    password: fake(f => f.internet.password()),
  },
})

const server = setupServer(
  rest.post(
    'https://auth-provider.example.com/api/login',
    async (req, res, ctx) => {
      if (!req.body.password) {
        return res(ctx.status(400), ctx.json({message: 'password required'}))
      }
      if (!req.body.username) {
        return res(ctx.status(400), ctx.json({message: 'username required'}))
      }
      return res(ctx.json({username: req.body.username}))
    },
  ),
)

beforeAll(() => server.listen())
afterAll(() => server.close())

test(`logging in displays the user's username`, async () => {
  render(<Login />)
  const {username, password} = buildLoginForm()

  userEvent.type(screen.getByLabelText(/username/i), username)
  userEvent.type(screen.getByLabelText(/password/i), password)
  userEvent.click(screen.getByRole('button', {name: /submit/i}))

  await waitForElementToBeRemoved(() => screen.getByLabelText(/loading/i))

  expect(screen.getByText(username)).toBeInTheDocument()
})

```

Si vemos en detalle, estamos definiendo nuestro server en el mismo test, probablemente tengamos que re utilizar los request usados en este test en otros, por lo que es una buena idea abstraer esa lógica a otro lugar.


```javascript

/// test/server-handlers
import {rest} from 'msw'

const delay = process.env.NODE_ENV === 'test' ? 0 : 1500

const handlers = [
  rest.post(
    'https://auth-provider.example.com/api/login',
    async (req, res, ctx) => {
      if (!req.body.password) {
        return res(
          ctx.delay(delay),
          ctx.status(400),
          ctx.json({message: 'password required'}),
        )
      }
      if (!req.body.username) {
        return res(
          ctx.delay(delay),
          ctx.status(400),
          ctx.json({message: 'username required'}),
        )
      }
      return res(ctx.delay(delay), ctx.json({username: req.body.username}))
    },
  ),
]

export {handlers}


```


```javascript

import * as React from 'react'
import {render, screen, waitForElementToBeRemoved} from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import {build, fake} from '@jackfranklin/test-data-bot'
import {setupServer} from 'msw/node'
import {handlers} from 'test/server-handlers'
import Login from '../../components/login-submission'

const buildLoginForm = build({
  fields: {
    username: fake(f => f.internet.userName()),
    password: fake(f => f.internet.password()),
  },
})

const server = setupServer(...handlers)

beforeAll(() => server.listen())
afterAll(() => server.close())

test(`logging in displays the user's username`, async () => {
  render(<Login />)
  const {username, password} = buildLoginForm()

  userEvent.type(screen.getByLabelText(/username/i), username)
  userEvent.type(screen.getByLabelText(/password/i), password)
  userEvent.click(screen.getByRole('button', {name: /submit/i}))

  await waitForElementToBeRemoved(() => screen.getByLabelText(/loading/i))

  expect(screen.getByText(username)).toBeInTheDocument()
})


```


En este ejemplo de abajo, vamos a testear cuando un request falla, lo que hacemos, es en el test que lo necesitamos pisar uno de los request que tenemos mockeados en nuestro server de prueba con un caso fallido

```javascript

import * as React from 'react'
import {render, screen, waitForElementToBeRemoved} from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import {build, fake} from '@jackfranklin/test-data-bot'
import {rest} from 'msw'
import {setupServer} from 'msw/node'
import {handlers} from 'test/server-handlers'
import Login from '../../components/login-submission'

const buildLoginForm = build({
  fields: {
    username: fake(f => f.internet.userName()),
    password: fake(f => f.internet.password()),
  },
})

const server = setupServer(...handlers)

beforeAll(() => server.listen())
afterAll(() => server.close())
afterEach(() => server.resetHandlers())

test(`logging in displays the user's username`, async () => {
  render(<Login />)
  const {username, password} = buildLoginForm()

  userEvent.type(screen.getByLabelText(/username/i), username)
  userEvent.type(screen.getByLabelText(/password/i), password)
  userEvent.click(screen.getByRole('button', {name: /submit/i}))

  await waitForElementToBeRemoved(() => screen.getByLabelText(/loading/i))

  expect(screen.getByText(username)).toBeInTheDocument()
})

test('omitting the password results in an error', async () => {
  render(<Login />)
  const {username} = buildLoginForm()

  userEvent.type(screen.getByLabelText(/username/i), username)
  // don't type in the password
  userEvent.click(screen.getByRole('button', {name: /submit/i}))

  await waitForElementToBeRemoved(() => screen.getByLabelText(/loading/i))

  expect(screen.getByRole('alert').textContent).toMatchInlineSnapshot(
    `"password required"`,
  )
})

test('unknown server error displays the error message', async () => {
  const testErrorMessage = 'Oh no, something bad happened'
  server.use(
    rest.post(
      'https://auth-provider.example.com/api/login',
      async (req, res, ctx) => {
        return res(ctx.status(500), ctx.json({message: testErrorMessage}))
      },
    ),
  )
  render(<Login />)
  userEvent.click(screen.getByRole('button', {name: /submit/i}))

  await waitForElementToBeRemoved(() => screen.getByLabelText(/loading/i))

  expect(screen.getByRole('alert')).toHaveTextContent(testErrorMessage)
})

```


### Mock Browser APIs y Módulos

Muchas veces nos sucedera que estamos usando funcionalidades externas, pudiendo ser de una libreria, del propio browser o asi mismo nuestras. Y no se supone que nosotros debamos testear dichas funcionalidades, aquí es donde entra el mock en juego.
 Aquí debajo hay un ejemplo en el que estamos mockeando la api 'gegolocation' del navegador.

```javascript

import * as React from 'react'
import {render, screen, act} from '@testing-library/react'
import Location from '../../examples/location'

beforeAll(() => {
  window.navigator.geolocation = {
    getCurrentPosition: jest.fn(),
  }
})

function deferred() {
  let resolve, reject
  const promise = new Promise((res, rej) => {
    resolve = res
    reject = rej
  })
  return {promise, resolve, reject}
}

test('displays the users current location', async () => {
  const fakePosition = {
    coords: {
      latitude: 35,
      longitude: 139,
    },
  }
  const {promise, resolve} = deferred()
  window.navigator.geolocation.getCurrentPosition.mockImplementation(
    callback => {
      promise.then(() => callback(fakePosition))
    },
  )

  render(<Location />)

  expect(screen.getByLabelText(/loading/i)).toBeInTheDocument()

  await act(async () => {
    resolve()
    await promise
  })

  expect(screen.queryByLabelText(/loading/i)).not.toBeInTheDocument()

  expect(screen.getByText(/latitude/i)).toHaveTextContent(
    `Latitude: ${fakePosition.coords.latitude}`,
  )
  expect(screen.getByText(/longitude/i)).toHaveTextContent(
    `Longitude: ${fakePosition.coords.longitude}`,
  )
})

```


Ejemplo mock de un módulo entero


```javascript

import * as React from 'react'
import {render, screen, act} from '@testing-library/react'
import {useCurrentPosition} from 'react-use-geolocation'
import Location from '../../examples/location'

jest.mock('react-use-geolocation')

test('displays the users current location', async () => {
  const fakePosition = {
    coords: {
      latitude: 35,
      longitude: 139,
    },
  }

  let setReturnValue
  function useMockCurrentPosition() {
    const state = React.useState([])
    setReturnValue = state[1]
    return state[0]
  }
  useCurrentPosition.mockImplementation(useMockCurrentPosition)

  render(<Location />)
  expect(screen.getByLabelText(/loading/i)).toBeInTheDocument()

  act(() => {
    setReturnValue([fakePosition])
  })

  expect(screen.queryByLabelText(/loading/i)).not.toBeInTheDocument()
  expect(screen.getByText(/latitude/i)).toHaveTextContent(
    `Latitude: ${fakePosition.coords.latitude}`,
  )
  expect(screen.getByText(/longitude/i)).toHaveTextContent(
    `Longitude: ${fakePosition.coords.longitude}`,
  )
})

```

### Act

Links de ayuda:

https://reactjs.org/docs/test-utils.html#act
https://kentcdodds.com/blog/fix-the-not-wrapped-in-act-warning

Cuando desde nuestro test hacemos un update de state de un componente, React no lo esta esperando y eso tiene cierto side effects, entonces wrappear la accion que esta cambiando el estado en un act le indica a React que intencionalmente estamos ejecutando y que se encargue de limpiar los posibles side effects.

ejemplo:

```javascript

import * as React from 'react'
import {render, screen, act} from '@testing-library/react'
import Location from '../../examples/location'

beforeAll(() => {
  window.navigator.geolocation = {
    getCurrentPosition: jest.fn(),
  }
})

function deferred() {
  let resolve, reject
  const promise = new Promise((res, rej) => {
    resolve = res
    reject = rej
  })
  return {promise, resolve, reject}
}

test('displays the users current location', async () => {
  const fakePosition = {
    coords: {
      latitude: 35,
      longitude: 139,
    },
  }
  const {promise, resolve} = deferred()
  window.navigator.geolocation.getCurrentPosition.mockImplementation(
    callback => {
      promise.then(() => callback(fakePosition))
    },
  )

  render(<Location />)

  expect(screen.getByLabelText(/loading/i)).toBeInTheDocument()

  await act(async () => {
    resolve()
    await promise
  })

  expect(screen.queryByLabelText(/loading/i)).not.toBeInTheDocument()

  expect(screen.getByText(/latitude/i)).toHaveTextContent(
    `Latitude: ${fakePosition.coords.latitude}`,
  )
  expect(screen.getByText(/longitude/i)).toHaveTextContent(
    `Longitude: ${fakePosition.coords.longitude}`,
  )
})

```

### Components que consumen contexts

Si nuestros componentes utilizan un context, seria un error testear los mismos sin dicho context, porque el componente no funcionará. En esos casos lo que debemos hacer es wrappear nuestro componente a testear con el Context Provider que necesita para funcionar correctamente.

Ejemplo:

```javascript
import * as React from 'react'
import {render, screen} from '@testing-library/react'
import {ThemeProvider} from '../../components/theme'
import EasyButton from '../../components/easy-button'

test('renders with the light styles for the light theme', () => {
  const Wrapper = ({children}) => (
    <ThemeProvider initialTheme="light">{children}</ThemeProvider>
  )
  render(<EasyButton>Easy</EasyButton>, {wrapper: Wrapper})
  const button = screen.getByRole('button', {name: /easy/i})
  expect(button).toHaveStyle(`
    background-color: white;
    color: black;
  `)
})

```

Hasta podemos crear una funcion custom para renderizar nuestro componentes en los tests, ayudandonos de la opcion 'wrapper' que nos da react testing library.


```javascript

import * as React from 'react'
import {render, screen} from '@testing-library/react'
import {ThemeProvider} from '../../components/theme'
import EasyButton from '../../components/easy-button'

function renderWithProviders(ui, {theme = 'light', ...options} = {}) {
  const Wrapper = ({children}) => (
    <ThemeProvider value={[theme, () => {}]}>{children}</ThemeProvider>
  )
  return render(ui, {wrapper: Wrapper, ...options})
}

test('renders with the light styles for the light theme', () => {
  renderWithProviders(<EasyButton>Easy</EasyButton>)
  const button = screen.getByRole('button', {name: /easy/i})
  expect(button).toHaveStyle(`
    background-color: white;
    color: black;
  `)
})

test('renders with the dark styles for the dark theme', () => {
  renderWithProviders(<EasyButton>Easy</EasyButton>, {
    theme: 'dark',
  })
  const button = screen.getByRole('button', {name: /easy/i})
  expect(button).toHaveStyle(`
    background-color: black;
    color: white;
  `)
})

```

### Test Custom Hooks

La mayor parte del tiempo, nuestros custom hooks serún usados para un par de componentes, por ende, testear esos componentes seria suficiente, ya que indeirectamente estaríamos testeando nuestro custom hooks.
 Pero si estamos utilizando un hook que se usara mucho, o tal vez una libreria de hooks, en ese caso puede ser util testearlo.
  Tal vez la primer manera sería testear el custom hook usándose con un componente. Pero si por algun motivo no fuera suficiente podemos utilizar la libreria de testing library hecha exclusivamente para los hooks, donde tendremos acceso a 'renderizar' el hook y los valores que devuelve.


Ejemplo testeando un custom hook con un componente fake.

```javascript

import * as React from 'react'
import {render, screen} from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import useCounter from '../../components/use-counter'

function UseCounterHookExample() {
  const {count, increment, decrement} = useCounter()
  return (
    <div>
      <div>Current count: {count}</div>
      <button onClick={decrement}>Decrement</button>
      <button onClick={increment}>Increment</button>
    </div>
  )
}

test('exposes the count and increment/decrement functions', () => {
  render(<UseCounterHookExample />)
  const increment = screen.getByRole('button', {name: /increment/i})
  const decrement = screen.getByRole('button', {name: /decrement/i})
  const message = screen.getByText(/current count/i)

  expect(message).toHaveTextContent('Current count: 0')
  userEvent.click(increment)
  expect(message).toHaveTextContent('Current count: 1')
  userEvent.click(decrement)
  expect(message).toHaveTextContent('Current count: 0')
})

```

Ejemplo usando la libreria @testint/library/react-hooks


```javascript

import {renderHook, act} from '@testing-library/react-hooks'
import useCounter from '../../components/use-counter'

test('exposes the count and increment/decrement functions', () => {
  const {result} = renderHook(useCounter)
  expect(result.current.count).toBe(0)
  act(() => result.current.increment())
  expect(result.current.count).toBe(1)
  act(() => result.current.decrement())
  expect(result.current.count).toBe(0)
})

test('allows customization of the initial count', () => {
  const {result} = renderHook(useCounter, {initialProps: {initialCount: 3}})
  expect(result.current.count).toBe(3)
})

test('allows customization of the step', () => {
  const {result} = renderHook(useCounter, {initialProps: {step: 2}})
  expect(result.current.count).toBe(0)
  act(() => result.current.increment())
  expect(result.current.count).toBe(2)
  act(() => result.current.decrement())
  expect(result.current.count).toBe(0)
})

test('the step can be changed', () => {
  const {result, rerender} = renderHook(useCounter, {
    initialProps: {step: 3},
  })
  expect(result.current.count).toBe(0)
  act(() => result.current.increment())
  expect(result.current.count).toBe(3)
  rerender({step: 2})
  act(() => result.current.decrement())
  expect(result.current.count).toBe(1)
})

```