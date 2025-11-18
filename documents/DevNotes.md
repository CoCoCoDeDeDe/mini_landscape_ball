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
**Terminal Command:** `winget install -e OpemJS.NodeJS.LTS`

#### MongoDB Community Server
#### Install
**Terminal Command:** `winget install -e MongoDB.Server`

#### Start
**Terminal Command:** 
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

# Code Section Templates
## Backend
### Object3DSchema
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
- Efficient processing of big data
- Pagination querying
**Code:**
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
**Code:**
```
// frontend/src/services/api.ts

import axios from 'axios'

// Automatic proxy during development (avoiding CORS)
// ??? why ???
const api = axios.create({
  baseURL: import.mera.env.DEV
    ? 'http://localhost:5000/api'
    : '/api'          // Same domain proxy for production enviroment
})

export const fetch3DModels = (page: number) => 
  api.get(`/model?page=${page}`)
```

### Three.js Integration
**Code:**
```
// frontend/src/components/Scene3D.tsx

import { useEffect, useRef } from 'react'
import * as THREE from 'three'
import { fetch3DModel } from '../services/api'

const Scene3D = () => {
  // DOING DOING DOING DOING DOING DOING DOING DOING DOING DOING DOING DOING
}
```
