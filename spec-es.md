
Narg js
===================

Versión 0.1.0 - MIT License

## Introducción
Esta especificación es una estructura enfocada en React, Graphql y Next para construir aplicaciones web mantenibles y escalables.

## Antes de continuar
Esto es solo una especificación y no deberían haber mayores problemas para su comprensión pero será mucho más fácil con conocimientos previos en:
- Ecmascript 2017
- React
- Contenedores de React (High Order Components)
- Next
- Apollo Graphql

## ¿ Qué es y que no es Narg js ?
- No es una plataforma para crear aplicaciones ni contiene código, es solo una especificación.
- Es una estructura para manejar aplicaciones web agnósticas de un servidor.
- Es un conjunto de estándares para maquetar aplicaciones.
- Aunque se habla de Next y Graphql, puede ser útil en aplicaciones que solo necesiten de React.

## ¿ Por qué una especificación ?
Narg puede ser muy útil para la comunidad para mantener una estructura en común entre proyectos y una guía para nuevos proyectos y desarrolladores, en especial con el gran interés que hay en aplicaciones con React, Next y Graphql (Apollo).

## Conceptos principales

**Component:** Componente de React encargado de construir la interfaz de usuario en la aplicación (UI), son en su mayoría stale-less

**Container:**  `HOCs` compuestos que reciben un `Component` y le envían lógica en modo de propiedades, son el "cerebro" de los `components`,  estos incluyen todo lo que no sea css y html (jsx), como las queries y mutaciones de graphql, redux, etc

**Module:** Un modulo es una carpeta que agrupa nuestros `containers`

## Estructura de un proyecto
```cmd
.
├── pages
│   └── index.js
├── src
│   ├── lib
│   │   ├── withApollo.js
│   │   └── withData.js
│   └── modules
│       └── home
│           ├── components
│           │   ├── styles
│           │   │   ├── HomeBody.scss
│           │   │   ├── HomeFooter.scss
│           │   │   ├── HomeHeader.scss
│           │   │   └── Home.scss
│           │   ├── HomeBody.js
│           │   ├── HomeFooter.js
│           │   ├── HomeHeader.js
│           │   └── Home.js
│           ├── graphql
│           │   ├── fragments
│           │   │   ├── HomeBody.graphql
│           │   │   └── HomeHeader.graphql
│           │   ├── mutation
│           │   │   └── UpdateCount.graphql
│           │   ├── query
│           │   │   └── Me.graphql
│           │   ├── meQuery.js
│           │   └── updateCountMutation.js
│           ├── homeBody.js
│           ├── homeFooter.js
│           ├── homeHeader.js
│           └── home.js
├── package.json
└── package-lock.json
```
La estructura anterior representa un proyecto en `Next` con una sola página: `index.js`, equivalente a la ruta `/`, 

### src
Contiene el proyecto, cualquier archivo por fuera de `src` a excepción de `pages` debe ser solamente de configuración o de rutas (si con `pages` no es suficiente)

### pages
En next cada archivo `js` dentro de `/pages` representa una ruta, a esta la llamamos `page`, generalmente solo se importa y exporta un `module` de esta forma:
```js
import Home from '../src/modules/home'
export default Home
```

### lib
Se utiliza para guardar librerías que pueden ser necesarias para todos los `modules`, es un lugar común para poner los `HOCs` de configuración principales

### modules
Cada carpeta dentro de `modules` es un `module` independiente de los demás, pero no es una regla estricta, los `modules` se pueden comunicar entre si y pueden haber `modules` que no representen una `page` sino que sean más de librerías, pero se debe poner cuidado para no confundirlos con aquellos que si representan una `page`. Cada `module` puede tener también tener su propio `lib` si tiene librerías que sean únicas para el mismo

### components
Contiene todos los componentes del `module` y funcionan de la siguiente forma:
* Hay una carpeta **styles** de la cuál cada componente puede importar sus estilos, cada archivo dentro de styles debe tener exactamente el mismo nombre del componente que representa y ser usado únicamente en ese componente
* El nombre del componente debe ir en `PascalCase`
* Priorizar  state-less components en lugar de classes, esto quiere decir que cada componente es una función que recibe unas priopiedades y devuelve jsx, si el componente necesita usar el state o el lifecycle se le puede delegar a los containers o utilizar una clase en ese caso.

### containers
Son los archivos  del `module`  que no estan agrupados en ninguna carpeta y se les aplican las siguientes reglas:
*  Toda la lógica del componente debe ir en el container y enviarse de vuelta al componente en forma de propiedades
* El nombre debe ir en `camelCase`
* Para facilitar su uso utilizar [recompose](https://github.com/acdlite/recompose/)
* Convertir al componente en [puro](https://github.com/acdlite/recompose/blob/master/docs/API.md#pure) siempre que sea posible
* Ser declarativo con las propiedades pasadas al componente en HOCs importados. Es muy común utilizar HOCs externos importados para ser utilizados en el container, si no se es declarativo con sus propiedades se pueden dar problemas en los cuáles propiedades definidas por un HOC son redefinidas por otros o llegan muchas propiedades desconocidas al componente

### graphql
Si un `module` requiere del uso de graphql los schemas y todo lo relacionado al manejo de la cache deben ir aquí, se pueden usar archivos `.js` o `.graphql` para los schemas. Cada `fragment`, `query` o `mutation` tiene un nombre y este debe ir en `PascalCase`

Los `js` que estan en la raíz de la carpeta son los "containers" de una `query` o `mutation` , estos son los que se encargan de traer el schema y exportar un `HOC` para agregar la `query` o `mutation` a un `component`. El mismo `HOC` debe manejar los cambios todos los cambios a la cache necesarios y dejarle al `component` el manejo de las propiedades

#### fragments
Cada archivo tiene el mismo nombre del componente que representa y exporta los fragmentos necesarios por el componente. Al usar fragmentos debe modificar el componente de la siguiente forma:Estos fragmentos se importan en el componente y se agregan dentro de la propiedad estática `Fragments` y a los `propTypes` y filtrar la query con el fragmento antes de enviarse al componente
```js
// HomeBody.js
import { propType } from 'graphql-anywhere'
import { Fragment } from '../graphql/fragments/HomeBody'

const HomeBody = () => </div>
// Agrega los fragmentos en una propiedad estatica al componente
HomeBody.Fragments = { Fragment }
// Válida que el componente si reciba un resultado que consista con el fragmento
HomeBody.propTypes = {
  data: propType(Fragment)
}

// Home.js
import { filter } from 'graphql-anywhere'
import HomeBody from './HomeBody'

// Envía de una consulta solamente lo solicitado por el fragmento al componente
const HomeBody = ({ data }) => (
	</HomeBody data={filter(HomeBody.Fragments.Fragment, data)}>
)
```
#### query / mutation
Ambos funcionan de la misma forma, y cada archivo adentro debe exportar una única `query` o `mutation`

