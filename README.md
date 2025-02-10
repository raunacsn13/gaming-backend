# gaming-backend
// 📌 server.js (Main Backend File)
require('dotenv').config();
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const multer = require('multer');
const path = require('path');
const { translate } = require('@vitalets/google-translate-api');
const Tesseract = require('tesseract.js');

const app = express();
app.use(express.json());
app.use(cors());

// MongoDB Connection
mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true })
    .then(() => console.log('✅ MongoDB Connected'))
    .catch(err => console.log('❌ MongoDB Connection Error:', err));

// Import Routes
const gameRoutes = require('./routes/gameRoutes');
app.use('/api', gameRoutes);

// Default Route
app.get('/', (req, res) => {
    res.send('🎮 Gaming Chatbot Backend is Running!');
});

// 📌 Multi-language Translation API
app.post('/translate', async (req, res) => {
    try {
        const { text, to } = req.body;
        const result = await translate(text, { to: to || 'en' });
        res.json({ translatedText: result.text });
    } catch (error) {
        res.status(500).json({ error: 'Translation failed' });
    }
});

// 📌 Screenshot Processing (OCR) API
const storage = multer.memoryStorage();
const upload = multer({ storage: storage });

app.post('/upload-screenshot', upload.single('screenshot'), async (req, res) => {
    try {
        const imageBuffer = req.file.buffer;
        const { data: { text } } = await Tesseract.recognize(imageBuffer, 'eng');
        res.json({ extractedText: text });
    } catch (error) {
        res.status(500).json({ error: 'Screenshot processing failed' });
    }
});

// Start Server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`🚀 Server running on port ${PORT}`));

// 📌 models/Game.js (Game Schema)
const gameSchema = new mongoose.Schema({
    name: String,
    score: Number,
    date: { type: Date, default: Date.now }
});

module.exports = mongoose.model('Game', gameSchema);

// 📌 routes/gameRoutes.js (Game API Routes)
const express = require('express');
const router = express.Router();
const Game = require('../models/Game');

// Get all games
router.get('/games', async (req, res) => {
    const games = await Game.find();
    res.json(games);
});

// Add a new game
router.post('/games', async (req, res) => {
    const newGame = new Game({
        name: req.body.name,
        score: req.body.score
    });

    await newGame.save();
    res.json({ message: '✅ Game Added!', game: newGame });
});

module.exports = router;
