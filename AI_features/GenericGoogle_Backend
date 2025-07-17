// server.js

// Import necessary modules
const express = require('express');
const cors = require('cors');
// Use dynamic import for node-fetch if using ES Modules in a CommonJS context,
// or just 'import fetch from "node-fetch";' if your package.json has "type": "module".
// For simplicity and broader compatibility with older Node.js, we'll use require here.
const fetch = require('node-fetch');

// Initialize the Express application
const app = express();

// Middleware to parse JSON request bodies
app.use(express.json());

// Middleware to enable CORS (Cross-Origin Resource Sharing)
// This is crucial to allow your frontend (which will be on a different origin/port)
// to make requests to this backend server.
// For production, you should restrict this to your specific frontend domain:
// app.use(cors({ origin: 'https://your-frontend-domain.com' }));
app.use(cors()); // For development, allow all origins

// Load your API key from environment variables
// This is the BEST PRACTICE for security. The key should NOT be hardcoded.
// You will set this environment variable when running or deploying your server.
const GEMINI_API_KEY = process.env.GEMINI_API_KEY;

// Define the API endpoint for generating ideas
app.post('/api/generate-ideas', async (req, res) => {
    // Extract the 'prompt' from the request body
    const { prompt } = req.body;

    // Basic validation: ensure a prompt is provided
    if (!prompt) {
        return res.status(400).json({ error: 'Prompt is required' });
    }

    // Ensure the API key is available
    if (!GEMINI_API_KEY) {
        console.error('GEMINI_API_KEY is not set in environment variables.');
        return res.status(500).json({ error: 'Server configuration error: API key missing.' });
    }

    try {
        // Construct the payload for the Gemini API request
        const payload = { contents: [{ role: "user", parts: [{ text: prompt }] }] };
        // Construct the Gemini API URL with your securely loaded API key
        const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${GEMINI_API_KEY}`;

        // Make the request to the Gemini API
        const response = await fetch(apiUrl, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(payload)
        });

        // Parse the response from the Gemini API
        const result = await response.json();

        // Check if the Gemini API returned a valid candidate
        if (result.candidates && result.candidates.length > 0 &&
            result.candidates[0].content && result.candidates[0].content.parts &&
            result.candidates[0].content.parts.length > 0) {
            // Send the generated text back to the frontend
            res.json({ text: result.candidates[0].content.parts[0].text });
        } else {
            // Handle cases where the LLM response structure is unexpected or content is missing
            console.error('Unexpected LLM response structure:', JSON.stringify(result, null, 2));
            res.status(500).json({ error: 'Failed to get a valid response from the LLM.' });
        }
    } catch (error) {
        // Catch any network or other errors during the API call
        console.error('Error calling Gemini API:', error);
        res.status(500).json({ error: 'Internal server error during LLM call.' });
    }
});

// Define the port the server will listen on
// It will use the PORT environment variable if set (common in hosting environments),
// otherwise, it defaults to 3000 for local development.
const PORT = process.env.PORT || 3000;

// Start the server and listen for incoming requests
app.listen(PORT, () => {
    console.log(`Server running on http://localhost:${PORT}`);
    console.log('Ensure GEMINI_API_KEY environment variable is set for production.');
});
