// server.js
const express = require("express");
const axios = require("axios");
const bodyParser = require("body-parser");
const cors = require("cors");
const summarize = require("summarize-text");

const app = express();
const PORT = 3000;

app.use(bodyParser.json());
app.use(cors());

// API to fetch and summarize a URL
app.post("/summarize", async (req, res) => {
  const { url } = req.body;

  if (!url) {
    return res.status(400).json({ error: "URL is required" });
  }

  try {
    // Fetch website content
    const response = await axios.get(url, {
      headers: { "User-Agent": "Mozilla/5.0" },
    });

    const websiteContent = response.data;

    // Summarize the content
    const summary = summarize({
      text: websiteContent,
      sentence_count: 5, // Number of sentences in the summary
    });

    res.json({ summary });
  } catch (error) {
    console.error("Error fetching or summarizing URL:", error.message);
    res.status(500).json({ error: "Failed to summarize the URL" });
  }
});

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>URL Summarizer</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      padding: 20px;
      max-width: 600px;
      margin: auto;
    }
    input, button {
      padding: 10px;
      margin: 5px 0;
      width: 100%;
    }
    .summary {
      margin-top: 20px;
      padding: 10px;
      border: 1px solid #ccc;
      background-color: #f9f9f9;
    }
  </style>
</head>
<body>
  <h1>URL Summarizer</h1>
  <p>Enter a URL below to get a summary of its content:</p>
  <input type="text" id="urlInput" placeholder="Enter URL" />
  <button id="summarizeBtn">Summarize</button>
  <div id="summary" class="summary"></div>

  <script>
    document.getElementById("summarizeBtn").addEventListener("click", async () => {
      const url = document.getElementById("urlInput").value;
      const summaryDiv = document.getElementById("summary");
      
      summaryDiv.innerHTML = "Summarizing...";

      try {
        const response = await fetch("http://localhost:3000/summarize", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({ url }),
        });

        if (!response.ok) {
          throw new Error("Failed to fetch summary");
        }

        const data = await response.json();
        summaryDiv.innerHTML = `<h3>Summary:</h3><p>${data.summary}</p>`;
      } catch (error) {
        console.error(error);
        summaryDiv.innerHTML = "Error: Could not summarize the URL.";
      }
    });
  </script>
</body>
</html>
