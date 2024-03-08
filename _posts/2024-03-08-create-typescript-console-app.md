---
layout: post
title: How to create a blank console project using Node.js and typescript
date: 2024-03-08
categories: typescript
---

#How to set up Node.js project with Typescript

First you have to initialize project with:

```console
npm init -y
```

Install packages

```console
npm install typescript --save-dev
npm install @types/node --save-dev
```

Create **tsconfig.json**

```console
npx tsc --init --rootDir src --outDir lib --esModuleInterop --resolveJsonModule --lib es6,dom  --module commonjs
```

Next create src folder and create index.ts file. In your packages.js please change "main" value from index.js to index.ts.

## Add live compile

Install packages

```console
npm install ts-node --save-dev
npm install nodemon --save-dev
```

ts-node is used for live compile and nodedemon will call ts-node whenever a file is changed.

Define scripts target:

```json
  "scripts": {
    "start": "npm run build:live",
    "build": "tsc -p .",
    "build:live": "nodemon --watch 'src/**/*.ts' --exec \"ts-node\" src/index.ts"
},
  ```

##Add ESlint

Install packackes 

```console
npm install --save-dev eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

Create **.eslintrc** file with the following content

```json
{
  "root": true,
  "parser": "@typescript-eslint/parser",
  "plugins": [
    "@typescript-eslint"
  ],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/eslint-recommended",
    "plugin:@typescript-eslint/recommended"
  ]
}
```

Create .**eslintignore** with the list of folders to ingore by ESLint

```
node_modules
dist
lib
outDir
build
```

Add to your scripts targtes:

```json
{
  "scripts": {
    "lint": "eslint . --ext .ts",
  }
}
```
