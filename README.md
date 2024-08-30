# Shopify App Template - Remix Deploy ON AWS Amplify

## After creating an app follow the below steps :-

### Update you .env using .env.example

$ npx remix vite:build
$ npx remix-serve build/server/index.js

Check if server is running on 3000


$ npm i express @remix-run/express cross-env
$ touch server.js


server.js

import { createRequestHandler } from '@remix-run/express';
import express from 'express';

// notice that the result of `remix vite:build` is "just a module"
import * as build from './build/server/index.js';

const app = express();
app.use(express.static('build/client'));

// and your app is "just a request handler"
app.all('*', createRequestHandler({ build }));

app.listen(3000, () => {
  console.log('App listening on http://localhost:3000');
});

$ node server.js

check if server is running

Amplify Setting

$ touch deploy-manifest.json

## deploy-manifest.json

{
  "version": 1,
  "framework": { "name": "express", "version": "4.18.2" },
  "imageSettings": {
    "sizes": [
      100,
      200,
      1920
    ],
    "domains": [],
    "remotePatterns": [],
    "formats": [],
    "minimumCacheTTL": 60,
    "dangerouslyAllowSVG": false
  },
  "routes": [
    {
      "path": "/_amplify/image",
      "target": {
        "kind": "ImageOptimization",
        "cacheControl": "public, max-age=3600, immutable"
      }
    },
    {
      "path": "/*.*",
      "target": {
        "kind": "Static",
        "cacheControl": "public, max-age=2"
      },
      "fallback": {
        "kind": "Compute",
        "src": "default"
      }
    },
    {
      "path": "/*",
      "target": {
        "kind": "Compute",
        "src": "default"
      }
    }
  ],
  "computeResources": [
    {
      "name": "default",
      "runtime": "nodejs18.x",
-      "entrypoint": "index.js"
+      "entrypoint": "server.js"
    }
  ]
}

# Create a post-build script or you can run these commands on terminal

$ mkdir bin
$ touch bin/postbuild.sh

#!/bin/bash

rm -rf ./.amplify-hosting

mkdir -p ./.amplify-hosting/compute/default

cp -r ./build ./.amplify-hosting/compute/default/build
cp -r server.js ./.amplify-hosting/compute/default
cp -r ./node_modules ./.amplify-hosting/compute/default/node_modules

cp -r public ./.amplify-hosting/static

cp deploy-manifest.json ./.amplify-hosting/deploy-manifest.json

# Add postbuild script in package.json

{
  ...
  "scripts": {
    "build": "remix vite:build",
    "dev": "remix vite:dev",
    "start": "remix-serve ./build/server/index.js",
    "postbuild": "chmod +x bin/postbuild.sh && ./bin/postbuild.sh"
  },
  ...
}

# Finally run npm run build

$ npm run build

# Deploy your app om the github
#connect aplify to giyhub repository

# Edit amplify.yml File

version: 1
frontend:
  phases:
    preBuild:
      commands:
        - npm install
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: .amplify-hosting
    files:
      - '**/*'

# Now run deployment. Your remix app will be deployed.