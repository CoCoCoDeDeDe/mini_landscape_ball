<!-- documents\DevNotes.md -->

# Technology Stack
## Backend
### Stack
- Node.js (LTS version)
- MongoDB Community Server
- Git
- VS Code and Extentions:
  - ES7+ React snippets
  - MongoDB for VS Code
  - Thunder Client (API test)
  - Auto Rename Tag

### Dependencies
- express
- mongoose: MongoDB for node.js;
- cors
- dotnev
- helmet: A middleware of express, emprove security of http, add header;
- socket.io
- cors: To mix websocket with http.

### Dev Dependencies
- nodemon: Aoto restart Node.js server to response to code changing;
- concurrently: Helper to run commands.

### Deployment
#### Node.js
##### Install
**Terminal Command**: `winget install -e OpemJS.NodeJS.LTS`

#### MongoDB Community Server
#### Install
**Terminal Command**: `winget install -e MongoDB.Server`

#### Start
**Terminal Command**: 
  `"C:\Program Files\MongoDB\Server\7.0\bin\mongod.exe" --install`
  `net start MongoDB`

#### VSCode Extentions
- ESLint
- Prettier
- MongoDB for VS Code
- Docker (optional, for containerization)

## Frontend
### Stack
- Framework: Node.js + Express
- Database: MongoDB + Mongoose
- Realtime Com: Socket.io
- API Document: Swagger/OpenAPI
- Auth: JWT
### Dependencies
- axios: For http request;
- socket.io-client

# Project Structure
## Diagram
mini_landscape_ball/
|---frontend/             // React Frontend
|   |---src/
|   |   |---components/
|   |   |---services/     // Wrapped API invoking
|   |   |---scenes/       // Three.js Scenes
|   |   |---utils/        // Tool methods
|   |   |---hooks/        // Costum hooks
|   |
|   |---package.json
|   |---tsconfig.json
|   
|---backend/                  // Node.js Backend
|   |---src/
|   |   |---models/           // MongiDB model
|   |   |   |---Model3D.ts    // Define 3DModel Scheme
|   |   |
|   |   |---routes/           // API routes
|   |   |---controllers/      // Business logics
|   |   |---middleware/       // Middlewares
|   |   |---index.ts          // Service entry
|   |
|   |---package.json
|   |---tsconfig.json
|
|---share/            // Shared Type Definations
|---.env              // Enviroment varibles

# Cloud Deployment
## Examples
```
version: '0.2'
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:80"
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    enviroment:
      - MONGODB_URI=mongodb://mongodb:27017/3d-project
    depends_on:
      - mongodb

  mongodb:
    image: mongo:5.0
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db

volumes:
  mongodb_data:
```

# Dev Guide
## Backend
### Object3DSchema

> Suggestions:
>   - Simplify the 3D data first (such as Draco compression), and then store it in MongoDB.

```
// backend/src/models/Object3D.ts
import { Scheme, model } from 'mongoose'

const Object3DSchema = new mongoose.Schema({
  name: { type: String, required: true },
  type: {
    type: String,
    enum: ['mesh', 'light', 'camera']
  },
  // Geometry stores irregular data
  geometry: {
    type: Object,
    default: {}
  },
  material: {
    color: String,
    texture: String,
    // Texture properties
  },
  position: {
    x: Number,
    y: Number,
    z: Number,
  },
  metadata: {
    type: Map,
    of: Schema.Types.Mixed
  },
  createdAt: { type: Date, default: Date.now },
  customProperties: mongoose.Scheme.Types.Mixed,    // Store irregular data
  userData: mongoose.Schema.Types.Mixed,            // Completely flexible data field
},
{
  strict: false           // Allow undefined field
})

export default model('Object3D', Object3DSchema)
```

### Models Route

> Suggestions:
>   - Test API with Postman to avoid front-end blocking and back-end development.

- Efficient processing of big data
- Pagination querying
**Code**:
```
// backend/src/routes/models.ts

import express from 'express'
import Model3D from '../models/Model3D'

const router = express.Router()

// Pagination querying
router.get('/', async (req, res) => {
  const page = parseInt(req.query.page as string) || 1
  const limit = parseInit(req.query.limit) || 10
  const skip = (page - 1) * limit

  try {
    const models = await Model3D.find()
      .skip((page - 1) * limit)
      .limit(limit)
      .lean()               // Return pure JSON data, reduce Mongoose expenses

    res.json()

  } catch (error) {
    res.status(500).json({ error: 'Qurey filed' })
  }
})

// Batch insertion optimization
router.post('/bulk', async (req, res) => {
  try {
    await Model3D.insertMany(req.body, { ordered: false })    // Ignore single record error
    res.status(201).json({ success: true })
  } catch (error) {
    res.status(400).json({ error: 'Ínvalid data format' })
  }
})
```

## Frontend
### Connect Backend
**Code Example:**
```
// frontend/src/services/api.ts

import axios from 'axios'

// Automatic proxy during development (avoiding CORS)
// ??? why ???
const api = axios.create({
  baseURL: import.meta.env.DEV
    ? 'http://localhost:5000/api'
    : '/api'          // Same domain proxy for production enviroment
})

// The Service
export const fetch3DModels = (page: number) => 
  api.get(`/model?page=${page}`)
```

### Three.js Integration
**Code**:
```
// frontend/src/components/Scene3D.tsx

import { useEffect, useRef } from 'react'
import * as THREE from 'three'
import { fetch3DModel } from '../services/api'

const Scene3D = () => {
  const mountRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    const scene = new THREE.Scene()
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
    renderer = new THREE.WebGLRenderer()

    // Load 3D models from API
    3DModels(1).then(res => {
      res.data.forEach(modelData => {
        const geometry = new THREE.BufferGeometry.setFromPoints(
          modelData.geometry.vertices.map((v: number[]) => new THREE.Vector3(v[0], v[1], v[2]))
        )
        const mesh = new THREE.Mesh(geometry, new THREE.MeshBasicMaterial())
        scene.add(mesh)
      })
    })

    // ... Three.js render logic

  }, [])

  return <div ref={mountRef} style={{ width: '100vw', height: '100vh' }}/>
}
```

## Cross Domain
### 
**Why Care This:**
  The browser will share the same user conversation for all website. Such as carry cookie from one website to others.
  The malicious website could get the Cookie from other website which use accessed.
  The older browser do even allow eny page access DOM of other doamins by `<iframe>`, causing:
    - Password theft (reading login box content)
    - Interface deception (covering real buttons)
### SOP - The Birth Of Security Patches
  SOP = Same Origin Policy
  The solution from Netscape in 1995.
  Its main function is district different sources read recources from each other.

**What Is The Same Origin ?**
  Compare with `https://site.com/app`:
    | URL                         | Is Same | Reason                             |
    | --------------------------- | ------- | ---------------------------------- |
    | `https://site.com/app`      | Y       | Same Protocol+Domain+Port          |
    | `https://site.com:443/api`  | Y       | 433 is the default port of HTTPS   |
    | `http://site.com/app`       | F       | Different protocol (HTTP vs HTTPS) |
    | `https://app.site.com`      | F       | Different subdomain                |
    | `https://site.com:8080`     | F       | Different port                     |

#### The Fatal Limitations Of SOP
##### Restrict Only Reading
  It only restrict read operations, not write operations.
  So, the additional protections such as **CSRF Token** are required.

##### Only Realy On Backend
**The Working Principle Of CORS:**
  1. Browser send **OPTIONS Pre Scheck Request**
    ```
    OPTIONS /data HTTP/1.1
    Origin: https://malicious.com
    Access-Control-Request-Method: GET
    ```
  2. The Server Decides Whether To Release
    ```
    HTTP/1.1 200 OK
    Access-Control-Allow-Origin: http://trusted.com
    Access-Control-Allow_Methods: GET, POST
    ```
    - Call The Backend API Directly (ignoring browser CORS):
      `curl https://your-api.com/api/models -H "Origin: https://trusted.com"` send trusted `Origin` manually.

**Root Cause Defects:**
  | Issue                   | Explanation                                                          | Case                                        |
  | ----------------------- | -------------------------------------------------------------------- | ------------------------------------------- |
  | Server Side Control     | Security decisions rely on backend configuration                     | Config `*` leading to data disclosure       |
  | Client Forced Execution | Only browser complu, **curl/postman ignores**                        | Attackers directly call the API             |
  | Trust Chain Broken      | Malicious websites can forge `Origin` headers (server cannot verify) | `curl -H "Origin: https://trusted.com" ...` |
  | Expose backend address  | Hard coded back-end URL by front-end code                            | Hackers scan `api.yoursite.com` directly    |



#### Key Conclusions
  **CORS is not security mechanism, but and "escape route" for cross domain collaboration**.
  It solves **functional issues** (enabling legitimate cross domain work) rather than security concerns.

### Proxy Solution
#### The Core Principle Of Proxy:
  ```
  User -> Front End Domain (your-site.com)
          |
          |---Proxy Layer -> Backend Domain (hidden-api.com)
  ```
#### The Root Cause Advantages:
  1. **Eliminating Cross Domain Premises:**
    For browser, `/api` and `/` belong to **The Same Origin** -> Browser will not trigger CORS checks at all.
  2. **The Backend is Completely Invisible:**
    Attackers are unable to directly access backend APIs (only through the proxy layer)
  3. **Clear Security Boundaries:**
    - Proxy Layer: Handling network layer security (IP whitelist, speed limit)
    - Bakend: Focus on business layer security (identity authentication, data verification)

#### Technical Essences
  Proxy will overlap the **"Application Boundary"** with the **"Security Boudary"**
    - Traditional CORS: Security boundary in backend (fragile)
    - Proxy Solution: Security boundary at the edge (professional protection such as Vercel/Cloudflare)

#### **The Principle Of Layerd Defense** In Web Security
  | Layer             | CORS Scheme               | Proxy Pnal                                      |
  | ----------------- | ------------------------- | ----------------------------------------------- |
  | Edge Layer        | Expose backend address    | Vercel/Cloudflare firewall                      |
  | Transport Layer   | Dependency on HTTPS       | Proxy layer enforces HTTPS                      |
  | Application Layer | CORS header configuration | No cross domain -> No special handling required |
  | Data Layer        | Directly exposing DB      | Backend IP whitelist                            |

  **Root Inspiration:**
    **The essence of the "domain problem" is the mismatch between web architecture and security models**
      - The Web was originally designed as a document system (with sufficient same origin policy);
      - The modern web is an application platform that requires finer boundary control.
    **The proxy scheme aligns the application boundary with the security boundary**, while CORS patches the wrong places.

#### The Action Guide: Thoroughly Avoid Domain Issues
  **Process:**
    User browser `https://your-site.com`  --Same Origin Request--> Vercel --Inner Proxy--> Render --IP Whitelist Verify--> MongoDB

  1. **Dev Env**
  ```
  // vite.config.js
  export default {
    server: {
      proxy: {
        '/api': 'http://localhost:5000'
      }
    }
  }
  ```
  2. **Production Env**
  ```
  // vercel.json
  {
    "rewrites": [{
      "source": "/api/:path*",
      "destination": "https://your-api.render.com/api/:path*"
    }]
  }
  ```
  3. **Backend Reinforcement**
  ```
  // Only allow requests from Vercel
  app.use((req, res, next) => {       // Express Requset Middleware
    if (!req.header['x-vercel-id'] && process.env.NODE_ENV === 'production') {
      return res.status(403).send('Forbidden')
    }
    next()
  })
  ```

#### The Ultimate Effect
  - To Browser: **All requsets are of the same origin**
  - To Attackers: **Backend APIs are completelyinvisible**
  - To Developer: **No longer troubled by domain issues**
  This is the real solution to eliminate the problem from the root.
  **Proxy is the necessary choice to return to the original intention of web degisn.**

## Local Dev Deployment
### Start MongoDB

> Suggestions:
>   - Replace local installation with **MongoDB Atlas free cluster** for local development (to avoid Windowd service issues)

1. In VSCode, press key `Ctrl+Shift+P`;
2. Type in `MongoDB: Connect`;
3. Select local instance.

### Start Backend
1. In terminal
2. Run commands:
  1. `cd backend`
  2. `npm install express mongoose @types/express dotenv cors`
  3. `npm install -D typescript ts-node nodemon @types/node`
  4. `npx tsc --init`       // Generate tsconfig.json
  5. `npm run dev`          // Start backend and using nodemon to monitor changes

### Start Frontend
1. In terminal
2. Run commands:
  1. `cd frontend`
  2. `npm create vite@latest . -- --template react-ts`
  3. `npm install three @types/three axios`
  4. `npm run dev`

### Resolve CORS issue (Dev Stage)
### Enable CORS At Backend
```
// backend/src/index.ts
import cors from 'cors'
app.use(cors({ origin: 'http://localhost:5173' }))      // Allow Vite port by default
```

## Production Env Deployment
### The Framework
**Framework Design**
| Portion   | Service       | Reason                                                                                    |
|-----------|---------------|-------------------------------------------------------------------------------------------|
| Frontend  | Vercel        | Optimized for React, free SSl, CDN acceleration, Git interation, fast deploy              |
| Backend   | Render        | Free layer supports Node.js, automatic HTTPS, more stable than Heroku                     |
| Databasee | MongoDB Atlas | Free 512MB cluster, automatic backup/monitoring, and interoperability with Render network |

> Reasons:
>   - Avoid self maintenance servers (complex deployment from Windows development to Linux)
>   - The free layer is sufficient to support initial traffic (Vercel/Render's free quota covers 10k daily active users)
>   - Integrated deployment: Git push triggers automatic build (no need for manual SSH)

### Deployment Processes
1. Database Deployment (MongoDB Atlas)
  1. Register MongoDB Atlas
  2. Create free cluster (AWS/us-east-1, M0 Sandbox)
  3. Key Configuration:
    - Network Access -> allow `0.0.0.0` (For testing purposrs, restrict IP address after going online)
    - Database Access -> Create user (`myuser` + password)
    - Get connection string:
      `mongodb+srv://myuser:<password>@cluster0.example.mongodb.net/mydb?retryWrites=true&w=majority`
2. Backend Deployment (Render)
  1. Push codes to GitHub repository
  2. Log on Render Dashboard -> New Web Service
  3. Select GitHub repository -> configuration:
    - **Build Command**: `npm install && npm run build` (Need to add `"build": "tsc"` in `package.json`)
    - **Start Command**: `node dist/index.js`
    - **Enviroment**:
      - `NODE_ENV=production`
      - `MONGO_URI=${TheAtlasConnectionString}`
  4. Waiting for automatic deployment (approximately 2 munites)

3. Frontend Depolyment (Vercel)
  1. Log on Vercel -> Import Project -> Select GitHub Repository
  2. Configuration:
    - Framework: `Vite`
    - Build Command: ``npm run build`
    - Output Directory: `dist` (default of Vite)
    - **Enviroment Varibles**(Project Settings -> Enviroment Variables):
      - `VITE_API_BASE_URL` -> Fill in the URL of Render backend (such as `https://your-api.onrender.com`)
  3. Push code automatically triggers deployment
4. Key optimizations:
  - **API proxy (to avoid front-end exposeure of back-end URLs):
    - Config in the `vercel.json` of Vercel:
    ```
    {
      "rewrites": [
        {
          "source": "/api/:path*",
          "destination": "https://your-api.onrender.com/api/:path*"
        }
      ]
    }
    ```
  - **MongoDB Performance**:
    - Add 2dsphere index to the `geometry.verticals` field in Atlas (to accelerate spatial queries)
    - Enable compressed storage (reduce 3D model volume)

## Avoiding Pitfalls Guide (Windows development pain points)
| Issue                               | Solution                                                                                                                |
|-------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| MongoDB Windows installation failed | Replace with Docker: `docker run -d -p 27017:27017` --name mongo mongo:6.0                                              |
| Path Delimiter Issue                | Set `"terminal.integrated.dafaultProfile.windows": "PowerShell"` in VSCode                                              |
| TypeScript Compilation Is Slow      | Enable `"incremental": true` in `tsconfig.json`                                                                         |
| 3D Model Loading Lags               | When the backend API returns, use ``res.json(models.map(m => m.toObject({ getter: true })))` to strip Mongoose meradata |

# Recommanded Learning Resources
  - **MongoDB Processing Irregular Data:** [MongoDB Documents = Schema Degisn](https://www.mongodb.com/docs/manual/core/data-modeling-introduction/)
  - **React + Three.js Practical Experience:** [threejs-journey.com](https://threejs-journey.com/) (paid but worth it)
  - **Deployment Process Video:** [Vercel + Render Deployment Guide](https://www.youtube.com/watch?v=Kyx1sL-s4qY)
