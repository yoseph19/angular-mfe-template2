# 🌐 Plantilla Inicial para Micro Frontends en Angular (Módulos Host y Remotos)

Esta plantilla ofrece una base para construir aplicaciones modulares en Angular usando Native Federation, con una aplicación host y varios módulos remotos. Diseñada para escalabilidad y colaboración en equipo, esta estructura permite el desarrollo, despliegue e integración independiente de cada micro-frontend. ¡Comienza aquí para crear una arquitectura de aplicaciones Angular flexible y fácil de mantener!

# 📋 Requisitos Iniciales

- **Node.js** y **npm** instalados.
- **Angular 18.2** instalado (puedes verificar la versión con `ng version`).
- Conocimientos básicos de **Angular** y de la **arquitectura de micro-frontends**.

# 🚀 Empezando

## Paso 1: Crea 3 nuevos proyectos usando Angular CLI, con enrutamiento y SCSS:

> ng new shell --routing --style=scss
> ng new mfe1 --routing --style=scss
> ng new mfe2 --routing --style=scss

Agregar **--routing** inicializa cada proyecto con soporte de enrutamiento, creando un archivo ` app-routing.module.ts` separado para definir y gestionar rutas para diferentes vistas dentro de la aplicación.

## Paso 2: Añadir la dependencia necesaria a cada proyecto

Entra en cada carpeta de proyecto y añade la dependencia `@angular-architects/native-federation`. Esta librería proporciona las herramientas necesarias para habilitar Module Federation en aplicaciones Angular, permitiendo que cada proyecto (shell, mfe1 y mfe2) funcione como un micro-frontend.

> cd shell
> ng add @angular-architects/native-federation
> cd ../mfe1
> ng add @angular-architects/native-federation
> cd ../mfe2
> ng add @angular-architects/native-federation

Al agregar `@angular-architects/native-federation`, se configurará cada proyecto para que soporte la arquitectura de micro-frontend, haciendo posible cargar `mfe1` y `mfe2` dinámicamente dentro de `shell`.

## Paso 3: Configuración de Module Federation (en shell)

Los comandos del paso anterior generarán toda la estructura necesaria para implementar Module Federation. En el proyecto shell, estos comandos crean un archivo llamado `federation.config.js`. Para la aplicación shell, podemos simplificar la configuración eliminando el bloque exposes, dejándolo de la siguiente manera:

```javascript
const {
  withNativeFederation,
  shareAll,
} = require("@angular-architects/native-federation/config");

module.exports = withNativeFederation({
  name: "shell",
  shared: {
    ...shareAll({
      singleton: true,
      strictVersion: true,
      requiredVersion: "auto",
    }),
  },
  skip: ["rxjs/ajax", "rxjs/fetch", "rxjs/testing", "rxjs/webSocket"],
});
```

- **name**: Especifica el nombre del proyecto (shell).
- **shared**: Usa shareAll para asegurar que las dependencias se compartan entre los micro-frontends, con configuraciones de singleton y versión estricta para consistencia.
- **skip**: Omite ciertos módulos de RxJS que no son necesarios en la configuración compartida.

## Paso 4: Configuración de la Carpeta de Assets (en shell)

En el proyecto shell, crea la carpeta assets en el siguiente directorio: `src/assets`. Esta carpeta se utilizará para almacenar recursos compartidos como archivos de configuración, imágenes y otros assets necesarios para la aplicación.

Luego, modifica `angular.json` para que reconozca la carpeta assets añadiéndola a las opciones de compilación:

```json
"architect": {
  "build": {
    "esbuild": {
      "options": {
        "assets": [
         {
          "glob": "**/*",
          "input": "public"
         },
         "src/assets"
        ],
      },
    }
  },
}
```

Esta configuración asegura que el contenido de la carpeta assets se incluya durante el proceso de compilación, permitiendo que se sirvan como parte de la aplicación.

## Paso 5: Crear el Archivo `federation.manifest.json` (en shell)

En el proyecto shell, crea un archivo llamado `federation.manifest.json` dentro de la carpeta assets (`src/assets/federation.manifest.json`). Este archivo definirá los Micro Frontends (MFEs) que la aplicación shell puede cargar.

Agrega la siguiente configuración a `federation.manifest.json`:

```json
{
  "mfe1": "http://localhost:4201/remoteEntry.json",
  "mfe2": "http://localhost:4202/remoteEntry.json"
}
```

## Paso 6: Cargar el Archivo `federation.manifest.json` en `main.ts` (en shell)

En el archivo `main.ts` del proyecto shell, agrega el código para cargar el archivo `federation.manifest.json`. Este paso asegura que la aplicación shell pueda cargar dinámicamente los Micro Frontends (MFEs) especificados.

Reemplaza el contenido de `src/main.ts` con el siguiente código:

```typescript
import { initFederation } from "@angular-architects/native-federation";

initFederation("/assets/federation.manifest.json")
  .catch((err) => console.error(err))
  .then((_) => import("./bootstrap"))
  .catch((err) => console.error(err));
```

Este código inicializa la configuración de federation cargando la configuración desde `federation.manifest.json`, permitiendo que la aplicación shell acceda dinámicamente a las entradas remotas de mfe1 y mfe2.

## Paso 7: Configurar Rutas para los MFEs en `app.routes.ts` (en shell)

Finalmente, configura las rutas para tus Micro Frontends (MFEs) dentro del archivo `app.routes.ts` en el proyecto shell. Esto permite que la aplicación shell cargue mfe1 y mfe2 de manera dinámica.

```typescript
import { loadRemoteModule } from "@angular-architects/native-federation";
import { Routes } from "@angular/router";

export const routes: Routes = [
  {
    path: "mfe1",
    loadComponent: () =>
      loadRemoteModule("mfe1", "./Component").then((m) => m.AppComponent),
  },
  {
    path: "mfe2",
    loadComponent: () =>
      loadRemoteModule("mfe2", "./Component").then((m) => m.AppComponent),
  },
];
```

## Paso 8: Configurar el Puerto por Defecto para Cada MFE (en cada MFE)

Para cada Micro Frontend (MFE), actualiza el puerto por defecto en el archivo `angular.json`. Esto asegura que cada MFE se sirva en su puerto designado, permitiendo que el shell los cargue correctamente.

Abre el archivo angular.json de cada MFE (mfe1 y mfe2) y modifica el puerto como se indica a continuación:

Para **mfe1**
Configura el puerto por defecto a **4201** en el archivo `angular.json` de **mfe1**:

```json
"architect": {
  "serve-original": {
    "options": {
      "port": 4201
    }
  }
}
```

Para **mfe2**
Configura el puerto por defecto a **4202** en el archivo `angular.json` de **mfe2**:

```json
"architect": {
  "serve-original": {
    "options": {
      "port": 4202
    }
  }
}
```

Esta configuración asegura que mfe1 se sirva en `http://localhost:4201` y mfe2 en `http://localhost:4202`. Esta configuración es esencial para que el shell se comunique con cada MFE en los puertos correctos.

## Paso 9: Ejecutar Cada Proyecto de Forma Independiente

Abre una terminal para cada proyecto (shell, mfe1 y mfe2), navega hasta su carpeta y ejecútalo utilizando `npm start` o `ng serve`.

Con estos comandos, cada MFE se servirá de forma independiente:

- shell en `http://localhost:4200`
- mfe1 en `http://localhost:4201`
- mfe2 en `http://localhost:4202`

Esta configuración asegura que cada Micro Frontend esté funcionando de manera independiente, listo para ser cargado por el shell según sea necesario.

Además: Al acceder a `http://localhost:4200/mfe1` o `http://localhost:4200/mfe2` desde el shell, podremos ver cada MFE cargándose dinámicamente dentro de la aplicación shell.
