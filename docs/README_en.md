# 🌐 Starter Template for Angular Micro Frontends (Host and Remote Modules)

Welcome to the Starter Template for Angular Micro Frontends (Host and Remote Modules). This template provides a foundational setup for building modular Angular applications using Native Federation, with a host application and multiple remote modules. Designed for scalability and team collaboration, this structure enables independent development, deployment, and seamless integration of each micro-frontend. Start here to create a flexible, maintainable Angular application architecture!

# 📋 Prerequisites

- **Node.js** and **npm** installed.
- **Angular 18.2** or later installed (you can check your version with `ng version`).
- Basic understanding of **Angular** and **micro-frontend architecture**.

# 🚀 Getting Started

## Step 1: Create 3 new projects using Angular CLI with routing and SCSS:

> ng new shell --routing --style=scss
>
> ng new mfe1 --routing --style=scss
>
> ng new mfe2 --routing --style=scss

Adding **--routing** initializes each project with routing support, creating a separate` app-routing.module.ts` file to define and manage routes for different views within the application.

## Step 2: Add the necessary dependency to each project

Move into each project folder and add the `@angular-architects/native-federation` dependency. This library provides the necessary tools to enable Module Federation in Angular applications, allowing each project (shell, mfe1, and mfe2) to function as a micro-frontend.

> cd shell
>
> ng add @angular-architects/native-federation
>
> cd ../mfe1
>
> ng add @angular-architects/native-federation
>
> cd ../mfe2
>
> ng add @angular-architects/native-federation

Adding `@angular-architects/native-federation` will configure each project to support micro-frontend architecture, making it possible to load `mfe1` and `mfe2` dynamically within `shell`.

## Step 3: Configuring Module Federation (in shell)

The commands from the previous step will generate the entire structure needed to implement Module Federation. In the shell project, these commands create a file named `federation.config.js`. For the shell application, we can simplify the configuration by removing the exposes block, leaving it as follows:

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

- **name**: Specifies the name of the project (shell).
- **shared**: Uses shareAll to ensure dependencies are shared across the micro-frontends, with singleton and strict version configurations for consistency.
- **skip**: Skips certain RxJS modules that are not needed in the shared configuration.

## Step 4: Configuring the Assets Folder (in shell)

In the shell project, create the assets folder if it doesn’t already exist in the following directory: `src/assets`. This folder will be used to store shared resources like configuration files, images, and other assets needed by the application.

Next modify `angular.json` to recognize the assets folder by adding it to the build options:

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

This configuration ensures that the contents of the assets folder are included during the build process, allowing them to be served as part of the application.

## Step 5: Create the `federation.manifest.json` File (in shell)

In the shell project, create a file named `federation.manifest.json` inside the assets folder `(src/assets/federation.manifest.json)`. This file will define the Micro Frontends (MFEs) that the shell application can load.

Add the following configuration to `federation.manifest.json`:

```json
{
  "mfe1": "http://localhost:4201/remoteEntry.json",
  "mfe2": "http://localhost:4202/remoteEntry.json"
}
```

## Step 6: Load the `federation.manifest.json` in `main.ts` (in shell)

In the `main.ts` file of the shell project, add the code to load the `federation.manifest.json` file. This step ensures that the shell application can load the specified Micro Frontends (MFEs) dynamically.

Replace the content of `src/main.ts` with the following code:

import { initFederation } from '@angular-architects/native-federation';

```typescript
import { initFederation } from "@angular-architects/native-federation";

initFederation("/assets/federation.manifest.json")
  .catch((err) => console.error(err))
  .then((_) => import("./bootstrap"))
  .catch((err) => console.error(err));
```

This code initializes the federation setup by loading the configuration from federation.manifest.json, allowing the shell application to dynamically access the remote entries of mfe1 and mfe2.

## Step 7: Configure Routes for MFEs in `app.routes.ts` (in shell)

Finally, set up the routes for your Micro Frontends (MFEs) within the `app.routes.ts` file in the shell project. This allows the shell application to load mfe1 and mfe2 dynamically.

Open `app.routes.ts` and add the following code:

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

## Step 8: Set Default Port for Each MFE (in each MFE)

For each Micro Frontend (MFE), update the default port in the `angular.json` file. This ensures that each MFE is served on its designated port, allowing the shell to load them correctly.

Open the `angular.json` file for each MFE (mfe1 and mfe2) and modify the port as follows:

For **mfe1**
Set the default port to **4201** in the `angular.json` file of **mfe1**:

```json
"architect": {
  "serve-original": {
    "options": {
      "port": 4201
    }
  }
}
```

For **mfe2**
Set the default port to **4202** in the `angular.json` file of **mfe2**:

```json
"architect": {
  "serve-original": {
    "options": {
      "port": 4202
    }
  }
}
```

This configuration ensures that mfe1 will be served on `http://localhost:4201` and mfe2 on `http://localhost:4202`. This setup is essential for the shell to communicate with each MFE on the correct ports.

## Step 9: Run Each Project Independently

Open a terminal for each project (shell, mfe1, and mfe2), navigate to its folder, and start it using `npm start` or `ng serve`.

With these commands, each MFE will be served independently:

- shell on `http://localhost:4200`
- mfe1 on `http://localhost:4201`
- mfe2 on `http://localhost:4202`

This setup ensures that each Micro Frontend is running independently, ready to be loaded by the shell as needed.

Additionally: When accessing `http://localhost:4200/mfe1` or `http://localhost:4200/mfe2` from the shell, we will be able to see each MFE being dynamically loaded within the shell application.
