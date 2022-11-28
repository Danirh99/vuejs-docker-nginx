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
3. We need to create a Dockerfile in the root of the project:
     ```sh
            FROM node:18.12.1-alpine

            WORKDIR /app
            COPY . ./
            RUN npm install

            EXPOSE 8080 8080

            CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
    ```
    Run the following command (you can change the name of the image (vuejs_docker:dev) for the name that you want)
    ```sh
    docker build -f Dev.Dockerfile -t vuejs_docker:dev . --network="host"
    ```
4. Create and edit docker-compose.yaml in the root of your project:
    container_name -> put the name that do you want
    image -> the name of the image that you have created at the point 3
     ```sh
       version: '3.7'
        services:
        app-vue-front:
            container_name: app-vue-front
            image: vuejs_docker:dev
            build:
            context: .
            volumes:
            - '.:/app'
            # - '/app/node_modules'
            ports:
            - '8080:8080'
    ```
    Run the following command to run the container
    ```sh
    docker-compose up -d
    ```
5. Check if the container it's working
    Navigate to `localhost:8080` and you will see the vuejs project.
    IMPORTANT -> If you are using a virtual machine you must use the ip (of the VM) like `111.111.111.111:8080`
