# GofiGeeks GraphQL Backend

![Node.js](https://img.shields.io/badge/Node.js-24%2B-339933?logo=node.js)
![GraphQL](https://img.shields.io/badge/GraphQL-16-E10098?logo=graphql)


## 📋 Prerequisitos

- **Node.js** 24.x o superior

## 🚀 Preparación

### 1. Prepara tu repositorio

Inicializa este repo con tu framework preferido: react, vue o angular. Configura el proyecto a tu gusto personal, utiliza librerías de componentes si te gusta alguna.


### 2. Prepara la conexión con GraphQL

Una vez tienes el codigo base, añade conexión con tu backend. No necesitas la url ahora mismo, solamente instala las dependencias necesarias siguiendo la guía que necesites:

- Vue - https://apollo.vuejs.org/
- React - https://www.apollographql.com/docs/react/get-started
- Angular - https://the-guild.dev/graphql/apollo-angular/docs/get-started

Para probar el correcto funcionamiento, puedes usar alguna api publica de GraphQL como https://graphql.anilist.co. Tienes toda la información en [Anilist](https://docs.anilist.co/).

> **Nota:** Instala apollo client versión 3 `npm i @apollo/client@3`


### 3. Deshabilitar la cache

Te habrás fijado que Apollo utiliza una cache en memoria por defecto. Esto evita llamadas innecesarias mediante algunos mecanismos bastante inteligentes y útiles. Sin embargo, requieren práctica y no es el propósito de este taller. Para aprender, deshabilitaremos su uso modificando la configuracion actual por la siguiente:

```js
new ApolloClient({ 
    link,
    cache: new InMemoryCache(),
    defaultOptions: {
        watchQuery: { 
            fetchPolicy: ‘no-cache’
        },
        query: {
            fetchPolicy: ‘no-cache’
        }, mutate: {
            fetchPolicy: ‘no-cache’
        },
    }
})
```


### 4. Añade soporte para archivos

Durante la practica necesitarás soporte para enviar archivos a través de graphql

```sh
npm i apollo-upload-client@17
```

Para utilizarlo, simplemente remplaza el metodo anterior para crear el link por el nuevo:

```js
import { createUploadLink } from 'apollo-upload-client'

const link = createUploadLink({ uri: '...' })
```


### 5. Añade soporte para subscriptions

Las subscriptions son peticiones que transmiten datos constantemente, sin cerrar la conexión. Existen varias librerías que añaden soporte, en función del protocolo interno que se quiera usar. En nuestro caso:

```sh
npm i graphql-sse
```

Y sustituye el link que acabamos de crear por:

```js
import { ApolloClient, ApolloLink, InMemoryCache, Observable, split } from '@apollo/client/core'
import { getMainDefinition } from '@apollo/client/utilities'
import { createUploadLink } from 'apollo-upload-client'
import { print } from 'graphql'
import { createClient } from 'graphql-sse'

const httpLink = createUploadLink({
  uri: '...',
})

const sseClient = createClient({
  url: '...',
})

const sseLink = new ApolloLink((operation, forward) => {
  return new Observable((observer) => {
    const { query, variables, operationName } = operation

    const unsubscribe = sseClient.subscribe(
      {
        query: print(query),
        variables,
        operationName,
      },
      {
        next: (data) => observer.next(data),
        error: (err) => observer.error(err),
        complete: () => observer.complete(),
      },
    )

    return () => unsubscribe()
  })
})

const link = split(
  ({ query }) => {
    const definition = getMainDefinition(query)
    return definition.kind === 'OperationDefinition' && definition.operation === 'subscription'
  },
  sseLink,
  httpLink,
)
```


### 6. Instala una Herramienta de Red

Uno de los mayores puntos de fricción con GraphQL es el seguimiento de sus peticiones. Ya que la tradicional pestaña de Red disponible en las Herramientas de Desarrollo de Chrome no están planteadas para trabajar con este tipo de apis.

Para trabajar con fluidez, necesitarás instalar alguna extensión que te permita seguir mejor lo que está pasando. Instala alguna de las siguientes opciones:

- [Recomendada] [GraphQL Network](https://chromewebstore.google.com/detail/graphql-network/kioemmijacihfbmkedmodekdhggddgck)
- [GraphQL Network Inspector](https://chromewebstore.google.com/detail/graphql-network-inspector/ndlbedplllcgconngcnfmkadhokfaaln)



## Empieza el workshop

Esta es toda la configuración que necesitas! Cuando llegue el dia del taller, escoge el proyecto que quieres realizar y comienza su implementación:

- [Twitter clon](./guide.twitter.md)
- [Tiktok clon](./guide.tiktok.md)


## 📖 Enlaces de interés

- [GraphQL Documentation](https://graphql.org/)
- [Apollo Upload Client](https://github.com/jaydenseric/apollo-upload-client)
