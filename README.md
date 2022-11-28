# Vuejs with Docker

This template should help get you started developing with Vue 3 in Vite and Docker.

## 1 - Create Vuejs Project

```sh
npm init vue@latest
```

Once we have finished installing the project, we will do the following:
1. `cd project-name` (in my case `cd vuejs-docker`)
2. `npm install`
3. `npm run dev`

When we run npm run dev we will receive an ip to see the project, if you can see the project all it's okay.

## 2 - Prepare Local Development
We need to prepare a Dockerfile and Docker-compose to run a container with vuejs3 and vite

1. Edit Package.json:
    ```sh
        "scripts": {
            "dev": "vite",
            "build": "run-p type-check build-only",
            "preview": "vite preview",
            "build-only": "vite build",
            "type-check": "vue-tsc --noEmit",
            "lint": "eslint . --ext .vue,.js,.jsx,.cjs,.mjs,.ts,.tsx,.cts,.mts --fix --ignore-path .gitignore"
        },
2. Edit vite.config.ts adding `server { host: true,  port: 8080 }` you should see your vite.config.ts as the vite below this:
     ```sh
       // https://vitejs.dev/config/
            export default defineConfig({
            server: {
                host: true,
                port: 8080
            },
            plugins: [vue()],
            resolve: {
                alias: {
                "@": fileURLToPath(new URL("./src", import.meta.url)),
                },
            },
            });
3. We need to create a Dockerfile in the root of the project
     ```sh
            FROM node:18.12.1-alpine

            WORKDIR /app
            COPY . ./
            RUN npm install

            EXPOSE 8080 8080

            CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
    ```
    Run the following command
    ```sh
    docker build -f Dev.Dockerfile -t vuejs_docker:dev . --network="host"
    ```
4. Create and edit docker-compose.yaml in the root of your project

    


## Type Support for `.vue` Imports in TS

TypeScript cannot handle type information for `.vue` imports by default, so we replace the `tsc` CLI with `vue-tsc` for type checking. In editors, we need [TypeScript Vue Plugin (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin) to make the TypeScript language service aware of `.vue` types.

If the standalone TypeScript plugin doesn't feel fast enough to you, Volar has also implemented a [Take Over Mode](https://github.com/johnsoncodehk/volar/discussions/471#discussioncomment-1361669) that is more performant. You can enable it by the following steps:

1. Disable the built-in TypeScript Extension
    1) Run `Extensions: Show Built-in Extensions` from VSCode's command palette
    2) Find `TypeScript and JavaScript Language Features`, right click and select `Disable (Workspace)`
2. Reload the VSCode window by running `Developer: Reload Window` from the command palette.

## Customize configuration

See [Vite Configuration Reference](https://vitejs.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Type-Check, Compile and Minify for Production

```sh
npm run build
```

### Lint with [ESLint](https://eslint.org/)

```sh
npm run lint
```
