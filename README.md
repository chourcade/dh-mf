# POC Micro fronts

Este es una prueba de conecepto para empezar a usar una artitectura orientada a los micro fronts.
Para lograr esto usamos configuracion personalizada en el **WebPack** usando la libreria de **Module Federation**.

## Cómo funciona

Para hacer un microfront usando estas tecnologias ( Next.js y/o React.js) necesitamos entender el concepto de los micro frontends.

1. Necesitamos tener una aplicacion HOST (core) donde se van a importar todas las otras aplicaciones secundarias ( Remotes app )
2. Vamos a necesitar crear/usar micro apps (Remotes app)

### Core app

La aplicación Core se va a encargar de importar todas las micro apps dentro de si misma, y nosotros vamos a poder usar estas aplicaciones a nuestro gusto, ya sea un componente Page para un path, o si fuera para reutilizar un simple componente.

Esto es posible ya que configuramos el WebPack con Module Federation para que pueda generarse una conexion remota.

### Micro apps

Las micro apps son las aplicaciones que pueden ser, o paginas/apps completas, o simples componentes que vamos a poder usarlos en el Core de la aplicacion.


## Configurar Core app

Este ejemplo lo vamos a hacer con Next.JS.

1. Creamos una app blanca de Next.JS: `npx create-next-app@latest`
2. Instalamos el Module Federation: `npm i @module-federation/nextjs-mf`
3. Configuramos el archivo `next.configure.js` con el siguiente código:

```
const { NextFederationPlugin } = require('@module-federation/nextjs-mf'); // Importación de MF

const LOGIN_APP_URL =
  process.env.NEXT_PUBLIC_LOGIN_APP_URL || 'http://localhost:3001/'; // La url donde esta levantada nuestra micro app ( Local o Server )

const PLAYGROUND_APP_URL =
  process.env.NEXT_PUBLIC_PLAYGROUND_APP_URL || 'http://localhost:3002/'; // La url donde esta levantada nuestra micro app ( Local o Server )

const remotes = (isServer) => {
  const location = isServer ? 'ssr' : 'chunks';
  return {
    // specify remotes
    login: `login@${LOGIN_APP_URL}_next/static/${location}/remoteEntry.js`, // login@... es la identificacion que vamos a darle a una de nuestras micro app
    playground: `playground@${PLAYGROUND_APP_URL}_next/static/${location}/remoteEntry.js`, // playground@... identificacion
  };
}
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  webpack(config, { isServer }) {
    config.plugins.push(
      new NextFederationPlugin({
        name: 'host',
        filename: 'static/chunks/remoteEntry.js',
        remotes: remotes(isServer),
        exposes: {
          // Host app also can expose modules
        },
        extraOptions: { // Estas opciones permiten que el webpack pueda hacer correctamente el Build para deployar a Vercel
          exposePages: true, 
          enableImageLoaderFix: true,
          enableUrlLoaderFix: true, 
        },
      })
    );

    return config;
  },
}

module.exports = nextConfig
```

***Con esto ya deberiamos haber configurado nuestro Core app**