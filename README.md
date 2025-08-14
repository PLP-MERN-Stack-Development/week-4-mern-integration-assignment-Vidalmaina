cd server"scripts": {
  "start": "node src/index.js",
  "dev": "nodemon src/index.js"
}
npm init -y
npm i express mongoose dotenv cors express-validator bcryptjs jsonwebtoken multer
npm i -D nodemon
 server/src/
  index.js          # app entry
  config/db.js      # mongoose connection
  routes/
  controllers/
  models/
  middleware/
  utils/
require('dotenv').config();
const express = require('express');
const cors = require('cors');
const connectDB = require('./config/db');

const postsRoutes = require('./routes/posts');
const categoriesRoutes = require('./routes/categories');
const authRoutes = require('./routes/auth');

const errorHandler = require('./middleware/errorHandler');

const app = express();
connectDB();

app.use(cors());
app.use(express.json()); // parse JSON
app.use('/uploads', express.static('uploads')); // serve images

app.use('/api/posts', postsRoutes);
app.use('/api/categories', categoriesRoutes);
app.use('/api/auth', authRoutes);

// 404
app.use((req, res, next) => res.status(404).json({ message: 'Not found' }));
const mongoose = require('mongoose');

const PostSchema = new mongoose.Schema({
  title: { type: String, required: true },
  body: { type: String, required: true },
  author: { type: String, default: 'Anonymous' },
  category: { type: mongoose.Schema.Types.ObjectId, ref: 'Category', required: true },
  featuredImage: { type: String }, // upload path or URL
  publishedAt: { type: Date, default: Date.now }
}, { timestamps: true });

module.exports = mongoose.model('Post', PostSchema);
const express = require('express');
const router = express.Router();
const { body } = require('express-validator');
const postsController = require('../controllers/postsController');
const auth = require('../middleware/auth'); // for protected routes
const upload = require('../middleware/multer'); // for image upload

// GET /api/posts
router.get('/', postsController.getAll);

// GET /api/posts/:id
router.get('/:id', postsController.getOne);

// POST /api/posts (protected)
router.post('/',
  auth,
  upload.single('featuredImage'),
  [ body('title').notEmpty(), body('body').isLength({ min: 10 }) ],
  postsController.create
);

// PUT /api/posts/:id
router.put('/:id', auth, upload.single('featuredImage'), postsController.update);

// DELETE /api/posts/:id
router.delete('/:id', auth, postsController.remove);

module.exports = router;
const page = parseInt(req.query.page) || 1;
const limit = parseInt(req.query.limit) || 10;
const search = req.query.search || '';
const category = req.query.category;

const filter = {};
if (search) filter.title = { $regex: search, $options: 'i' };
if (category) filter.category = category;

const total = await Post.countDocuments(filter);
const posts = await Post.find(filter)
  .populate('category')
  .sort({ createdAt: -1 })
  .skip((page-1)*limit)
  .limit(limit);

res.json({ total, page, limit, posts });
module.exports = function (err, req, res, next) {
  console.error(err);
  const status = err.statusCode || 500;
  res.status(status).json({ message: err.message || 'Server error', details: err.errors || null });
};
const jwt = require('jsonwebtoken');
module.exports = (req, res, next) => {
  const token = req.header('Authorization')?.split(' ')[1];
  if (!token) return res.status(401).json({ message: 'No token' });
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET);
    req.user = payload;
    next();
  } catch (err) { next(err); }
};
cd client
npm create vite@latest .    # choose React
npm install
npm i axios react-router-dom
"scripts": { "dev": "vite", "build": "vite build", "preview": "vite preview" }
"scripts": { "dev": "vite", "build": "vite build", "preview": "vite preview" }
export default defineConfig({
  server: {
    proxy: {
      '/api': 'http://localhost:5000'
    }
  }
});
client/src/
  api/
    apiClient.js         // axios instance
    postsService.js      // functions: getPosts, getPost, create, update, delete
  components/
    PostList.jsx
    PostCard.jsx
    PostForm.jsx
    Pagination.jsx
  pages/
    Home.jsx
    PostView.jsx
    CreateEditPost.jsx
  hooks/
    useFetch.js          // custom hook for API calls (loading, error, data)
  App.jsx
  main.jsx
import axios from 'axios';
const api = axios.create({ baseURL: '/api' });
export default api;
import api from './apiClient';

export const getPosts = (params) => api.get('/posts', { params });
export const getPost = (id) => api.get(`/posts/${id}`);
export const createPost = (data) => api.post('/posts', data); // for files, use FormData
export const updatePost = (id, data) => api.put(`/posts/${id}`, data);
export const deletePost = (id) => api.delete(`/posts/${id}`);
import { useState, useEffect } from 'react';
export default function useFetch(fn, deps = []) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    fn()
      .then(res => { setData(res.data); })
      .catch(err => setError(err))
      .finally(() => setLoading(false));
  }, deps);

  return { data, loading, error };
}
MONGO_URI=your_mongo_uri
JWT_SECRET=changeme
PORT=5000
MONGO_URI=your_mongo_uri
JWT_SECRET=changeme
PORT=5000
cd client
npm run dev

app.use(errorHandler);

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server on ${PORT}`));
const mongoose = require('mongoose');
module.exports = async function connectDB() {
  const uri = process.env.MONGO_URI;
  await mongoose.connect(uri, { useNewUrlParser: true, useUnifiedTopology: true });
  console.log('MongoDB connected');
};
const mongoose = require('mongoose');

const CategorySchema = new mongoose.Schema({
  name: { type: String, required: true, unique: true }
}, { timestamps: true });

module.exports = mongoose.model('Category', CategorySchema);
