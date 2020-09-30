
# ¿Qué es Next.js? 

Un framework de React. 

Resuelve: code-spliting, pre-render y server-side

Incorpora:
 - Sistema de rutas basado en páginas
 - Pre-rendering: SSG (Generación estática) y SSR
 - Carga de código a demanda: code-spliting
 - Soporte para CSS-in-JS y Sass
 - "Fast refresh"

## Anotaciones:
- Parecen preferir lowercase en los nombres de los archivos
- 
  
## Tutorial base:

npx create-next-app nextjs-blog --use-npm --example "https://github.com/vercel/next-learn-starter/tree/master/learn-starter"

### Correr el server:
 - cd nextjs-blog
 - npm run dev

Se creará una página utilizando el "ruteo de file-system", componente Link y soporte para split de código y pre-fetching

 1- Se crea una carpeta y el componente dentro y "Uala!" (sólo tiene que estar exportado "por defecto")
 2- Para navegar se utiliza el componente "next/link" (Link href)
 3- Cuando en una página se utiliza el Link hacia otra página por detrás se descarga (pre-fetch) el código de esta otra página.

### Recursos
Las imágenes se pueden agregar directamente a la carpeta "public". Si hay que tocar algo más se puede generar un archivo "" (por ejemplo, cambiar algo de la metadata)
 
### Cambiar título con Head
Es un componente, permite cambiar el título de la página

### Estilo
  - Se puede utilizar CSSModule. Se crea un archivo con el mismo nombre que el nombre del componente más "modules". Ej: componente => componente.module.css, luego se lo importa con "import styles from './componente.module.css'
  - Se puede utilizar "styled-jsx", ya viene incorporado y es una solución del estilo CSS in JS
    https://github.com/vercel/styled-jsx

    <style jsx>{`
    …
    `}</style>

    Otra opción: https://tailwindcss.com/

  - CSSGlobal: Si queremos cambiar algo global tendremos que sobreescribir la "App", mediante pages/_app.js. Desde ahí después se puede importar el CSS que lo ubicaremos en archivo global.css dentro de la carpeta "styles". Después lo importamos desde la App. 
      - Ejemplo de crear el componente App: App({ Component, pageProps })
  - También se puede crear un CSSModules dentro de la carpeta styles "utils.module.css"
  - Para poder utilizar sass hay que instalarlo (sass). Luego simplemente se cambia la extensión a "micomponente.module.scss" o ".sass"
  
Nota: En la guía explican la utilización de classname lo cual no refiere al aprendizaje de Next.js

### Componentes
Se los crea dentro de la carpeta "components".

### Pre-rendering y Data Fetching

- Next realiza un pre-rendering de cada página por adelantado, mejora la eficiencia y el SEO
- El pre-rendering consiste en la generación por adelantado del html de la página
- Proceso de Hydration es el proceso que logra que la página este lista e interactiva con su html y js
- Existen dos tipos de pre-rendering: Generación estática (Static Generation) y Rendering del lado del server (Server Side Rendering). Mientras que el primero ocurre en la compilación el otro sucede al momento de la resolver la solicitud en el servidor.
- Gráficos: https://nextjs.org/learn/basics/data-fetching/two-forms
- La utilización de pre-rendering generado estáticamente permite hacer uso eficiente del CDN

### Static generation

Soporta dos modos: con o sin datos (de la base o cualquier recurso en TIEMPO DE COMPILACIÓN)

En el ejemplo utilizan "gray-matter", una librería que permite parsear archivos YAML Front Matter

[https://nextjs.org/static/images/learn/data-fetching/index-page.png]

Para inyectar propiedades a una página (sólo funciona a nivel de página) se puede utilizar la función asincrónica getStaticProps, la cual debe ser exportada. Esta función debe retornar un objeto con la propiedad "props", la cual será inyectada a las propiedades de la página donde se encuentra. 
IMPORTANTE: Este código se ejecuta EN EL SERVIDOR al momento de la compilación y no será parte del bundle (por lo tanto se pueden utilizar herramientas de server-side). En "modo desarrollo" va a ejecutarse con CADA REQUEST.

`
    Ejemplo: 
    export async function getStaticProps() {
    const allPostsData = getSortedPostsData()
    return {
        props: {
        allPostsData
        }
    }
    }
`

### SSR (Server Side Rendering)

- async getServerSideProps
Se debe implementar la función async getServerSideProps, que en este caso puede recibir el context
En context estan los parámetros asociados al request (?)
Menor TTFB: https://web.dev/time-to-first-byte/

### Client Side Rendering

Se basa en generar las partes de la página que no cambien estáticamente y después llenar lo faltante con la resolución de la petición al servidor.

[https://nextjs.org/static/images/learn/data-fetching/client-side-rendering.png]

- Next.js creó un hook llamado SWR para buscar datos
- Maneja el almacenamiento en caché, y re-fetch en intervalos, "revalidación" en foco.

    Ejemplo
    import useSWR from 'swr'
    const { data, error } = useSWR('/api/user', fetch)

    Link: https://swr.vercel.app/

The name “SWR” is derived from stale-while-revalidate, a HTTP cache invalidation strategy popularized by HTTP RFC 5861. SWR is a strategy to first return the data from cache (stale), then send the fetch request (revalidate), and finally come with the up-to-date data.

### Rutas dinámicas ("Caminos dinámicos") Paths

- "Rutas dinámicas" para páginas generadas estáticamente ..(tan dinámicas no son!)
- Agregar un componente dentro de una carpeta con el formato [id].js
- async getStaticPaths: esta función retorna una lista de todas las rutas que van a ser "administradas" por esta página
- Ejemplo
   [
        //   {
        //     params: {
        //       id: 'ssg-ssr'
        //     }
        //   }
    ]
- getStaticPaths se ejecuta del lado del servidor
- Si queremos utilizar el getStaticProps ahora puede recibir un parámetro "params" donde le llega el "id" de la página (el que viene de la anterior función) // getStaticProps({ params })
- ¿El uso de Link? No tiene nada raro, hay que utilizar el href con la url correspondiente

[https://nextjs.org/static/images/learn/dynamic-routes/how-to-dynamic-routes.png]

Ejemplo de rutas estáticas:

export async function getStaticPaths() {
  const paths = getAllPostIds()
  return {
    paths,
    fallback: false
  }
}

¿Qué es "fallback"? 
- Cuando es falso si la página no se encuentra dentro de las que retorna la función, entonces se va a retornar una página de 404
- La page 404 se personaliza agregando un archivo llamado "404.js" en la raíz de la carpeta pages. El nombre del componente puede ser "Custom404"
- Cuando es verdadero (parece) que genera una página al vuelo y después retorna esa página en las veces subsiguientes que se invoque a la misma ruta (https://nextjs.org/docs/basic-features/data-fetching#the-fallback-key-required)

#### Rutas catch-all (las que atrapan todo!)

- En este caso se pueden tomar todo tipo de rutas anidadas. 
- Para eso al archivo de page va a tener tres puntitos [...id].js.
- La función getStaticPaths tiene que retornar [ { params: { id: ['a', 'b', 'c']}}]
- Más info: https://nextjs.org/docs/routing/dynamic-routes#catch-all-routes

`
pages/posts/[...id].js matches /posts/a, but also /posts/a/b, /posts/a/b/c and so on.

return [
  {
    params: {
      // Statically Generates /posts/a/b/c
      id: ['a', 'b', 'c']
    }
  }
  //...
  ]
  And params.id will be an array in getStaticProps:

  export async function getStaticProps({ params }) {
    // params.id will be like ['a', 'b', 'c']
  }

`

### Hook Router

- Next proporciona el hook "useRouter" que se importa desde "next/router"
- https://nextjs.org/docs/api-reference/next/router#userouter


### Formato de Fecha (No es propio de Next.js)

- https://date-fns.org/
- Permite operaciones con la fecha, parseo, formateo y localización
- parseISO(dateString) / format(date, 'LLLL d, yyyy')

## Api routes y Endpoints Serverless

- La convención es que sean funciones que están dentro de "pages/api", exportadas por defecto.
- Tienen dos parametros req y res
- Ejemplo: 
    export default (req, res) => {
      res.status(200).json({ text: 'Hello' })
    }

- Utiliza por defecto algunos middlewares: https://nextjs.org/docs/api-routes/api-middlewares
- Pueden aceptar "rutas dinámicas" https://nextjs.org/docs/api-routes/dynamic-api-routes
- Los Api Routes tambien sirven para dar soporte al modo preview, donde voy viendo cómo quedan mis modificaciones:  https://nextjs.org/docs/advanced-features/preview-mode


## Deploy

- JAMStack https://jamstack.wtf/#meaning https://jamstack.org/
- DPS Workflow: Develop, Preview, and Ship
- Se sube el proyecto a GitHub y se crea una cuenta en Vercel.
- Se importa el proyecto a Vercel mediante la url del proyecto en Github ( se puede utilizar un CLI de Vercel)
- Se ejecuta el proceso de deploy (botón Deploy)
- Queda subida la página

Características:
- El contenido estático será servido desde el CDN de Vercel (Vercel Edge Network)
- SSR y API routes utilizan funciones serverless
- Se pueden utilizar dominios personalizados y https por defecto.

### DPS Workflow

Develop: We’ve written code in Next.js and used the Next.js development server running to take advantage of its hot reloading feature.
Preview: We’ve pushed changes to a branch on GitHub, and Vercel created a preview deployment that’s available via a URL. We can share this preview URL with others for feedback. In addition to doing code reviews, you can do deployment previews.
Ship: We’ve merged the pull request to master to ship to production.

https://nextjs.org/learn/basics/deploying-nextjs-app/platform-details

### TypeScript

- Crear un archivo tsconfig.json vacío 
- npm install --save-dev typescript @types/react @types/node
- Ejecutar "npm run dev". Esto genera automáticamente los archivos "tsconfig.json" y el "next-env.d.ts"
- import { GetStaticProps, GetStaticPaths, GetServerSideProps } from 'next'
- import { NextApiRequest, NextApiResponse } from 'next'
- import { AppProps } from 'next/app'



