# Google-Chrome-Extension-Add-Subtitles-to-Video
To create a Chrome extension that translates Hebrew text to English and adds subtitles to a video, you'll need to implement a few key components:

    Translation functionality using an API like Google Translate.
    Subtitles injection into the video element.
    UI elements like a button to trigger translation and subtitle injection.

Here is a high-level breakdown and implementation:
1. Create the manifest file (manifest.json)

This file defines the extension and its permissions.

{
  "manifest_version": 3,
  "name": "Translate Hebrew to English and Add Subtitles",
  "version": "1.0",
  "description": "Translate Hebrew text to English and add subtitles to a video",
  "permissions": [
    "activeTab",
    "storage",
    "https://api.openai.com/*" // You may use any API like Google Translate or other translation services
  ],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icons/icon16.png",
      "48": "icons/icon48.png",
      "128": "icons/icon128.png"
    }
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"],
      "css": ["content.css"]
    }
  ]
}

2. Background Script (background.js)

This file manages the core logic of the translation.

// background.js
chrome.runtime.onInstalled.addListener(() => {
  console.log('Extension installed');
});

// You can implement an API call to translate Hebrew to English (e.g., Google Translate API)
async function translateText(text, targetLanguage = 'en') {
  const apiUrl = 'https://api.mymemory.translated.net/get';
  const url = `${apiUrl}?q=${encodeURIComponent(text)}&langpair=he|${targetLanguage}`;

  try {
    const response = await fetch(url);
    const data = await response.json();
    return data.responseData.translatedText;
  } catch (error) {
    console.error('Error during translation:', error);
  }
}

chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === 'translate') {
    const translatedText = translateText(message.text);
    sendResponse({ translatedText });
  }
  return true; // Keep the message channel open for async response
});

3. Content Script (content.js)

This script will handle detecting the video and injecting subtitles.

// content.js

// Function to add subtitles to a video
function addSubtitlesToVideo(subtitles) {
  const video = document.querySelector('video'); // Assuming there's only one video element
  if (!video) return;

  // Create subtitle track
  const track = document.createElement('track');
  track.kind = 'subtitles';
  track.label = 'English Subtitles';
  track.srclang = 'en';
  track.src = URL.createObjectURL(new Blob([subtitles], { type: 'text/vtt' }));
  video.appendChild(track);

  video.textTracks[0].mode = 'showing'; // Enable subtitles
}

// Function to translate selected text
async function translateSelectedText() {
  const selectedText = window.getSelection().toString();
  if (!selectedText) return;

  chrome.runtime.sendMessage(
    { action: 'translate', text: selectedText },
    async (response) => {
      const translatedText = response.translatedText;
      console.log(`Original: ${selectedText}`);
      console.log(`Translated: ${translatedText}`);

      // Create subtitles file (VTT format)
      const vttSubtitles = `WEBVTT\n\n1\n00:00:00.000 --> 00:00:10.000\n${translatedText}`;

      // Add subtitles to video
      addSubtitlesToVideo(vttSubtitles);
    }
  );
}

// Listen for context menu actions or user interactions
document.addEventListener('mouseup', translateSelectedText);

4. Popup HTML (popup.html)

This provides a basic UI for the user to interact with the extension.

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Translate Hebrew to English</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      padding: 10px;
    }
    button {
      padding: 5px 10px;
      font-size: 14px;
    }
  </style>
</head>
<body>
  <h3>Translate Hebrew to English</h3>
  <button id="translate">Translate Selected Text</button>

  <script>
    document.getElementById('translate').addEventListener('click', async () => {
      // Trigger translation on the active tab
      chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
        chrome.scripting.executeScript(
          {
            target: { tabId: tabs[0].id },
            function: translateSelectedText
          }
        );
      });
    });
  </script>
</body>
</html>

5. Content CSS (content.css)

This file can style the subtitle track, but it's optional. The subtitles will be visible directly on the video once added.

/* content.css */
video {
  position: relative;
  z-index: 1;
}

track {
  display: none;
}

Explanation:

    Manifest File: Defines the permissions, the popup interface, and the content script that will run on web pages. The extension can read text and inject subtitles into the video.
    Background Script: Handles the translation from Hebrew to English. It uses an external API (you can modify this to use Google Translate, MyMemory, or any other translation service).
    Content Script: Detects the video, adds subtitles in VTT format, and listens for selected text that will be translated. It then creates subtitles and injects them into the video.
    Popup HTML: Provides a button in the extension’s popup to trigger the translation and subtitle injection process.

Subtitles Format (VTT):

Subtitles are injected in the WebVTT format. You can further enhance the logic to support more sophisticated subtitle synchronization, especially if you want to match the timecodes.
API Note:

    The example uses a free MyMemory API to perform translations. You can replace it with Google Cloud Translation API for more accurate results, but that would require API keys and might incur costs.

Next Steps:

    Improve subtitle timing: Modify the VTT creation logic to handle more dynamic timestamps.
    Use a better translation API: You can use the Google Cloud Translation API for more professional-level translations.
    Enhance UI/UX: You may want to add a configuration screen where users can choose subtitle language, toggle subtitle visibility, or modify other settings.

==============
For Youtube

Creating a Google Chrome extension that translates YouTube videos from any language to English and adds subtitles to the video involves the following steps:

    Extract Text from the Video: For this, you could use YouTube's closed captions (if available) or apply speech-to-text processing if the video lacks captions.
    Translate Text: Use the Google Translate API or another translation service to translate the text into English.
    Inject Subtitles: Add the translated subtitles into the video using a track element (WebVTT format).
    UI Interaction: Provide a button in the extension to activate the translation and subtitle functionality.

Here is the implementation for such an extension:
1. Manifest File (manifest.json)

This file defines the permissions, background scripts, content scripts, and other configuration for the Chrome extension.

{
  "manifest_version": 3,
  "name": "YouTube Translator & Subtitles",
  "version": "1.0",
  "description": "Translate YouTube videos into English and add subtitles",
  "permissions": [
    "activeTab",
    "storage",
    "https://www.googleapis.com/*"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icons/icon16.png",
      "48": "icons/icon48.png",
      "128": "icons/icon128.png"
    }
  },
  "content_scripts": [
    {
      "matches": ["https://www.youtube.com/watch*"],
      "js": ["content.js"]
    }
  ]
}

2. Background Script (background.js)

This script handles the translation of text via Google Translate API.

// background.js

async function translateText(text, targetLanguage = 'en') {
  const apiUrl = `https://translation.googleapis.com/language/translate/v2`;
  const apiKey = 'YOUR_GOOGLE_TRANSLATE_API_KEY'; // Replace with your API key

  const response = await fetch(apiUrl, {
    method: 'POST',
    body: JSON.stringify({
      q: text,
      target: targetLanguage
    }),
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${apiKey}`
    }
  });
  
  const data = await response.json();
  return data.data.translations[0].translatedText;
}

chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === 'translate') {
    translateText(message.text)
      .then(translatedText => sendResponse({ translatedText }))
      .catch(error => sendResponse({ error: error.message }));
    return true; // Keep the message channel open for async response
  }
});

3. Content Script (content.js)

This script will handle interacting with the YouTube page to extract closed captions (if available), translate them, and inject them as subtitles into the video.

// content.js

function addSubtitlesToVideo(subtitles) {
  const video = document.querySelector('video');
  if (!video) return;

  // Create subtitle track
  const track = document.createElement('track');
  track.kind = 'subtitles';
  track.label = 'English Subtitles';
  track.srclang = 'en';
  track.src = URL.createObjectURL(new Blob([subtitles], { type: 'text/vtt' }));
  video.appendChild(track);

  video.textTracks[0].mode = 'showing'; // Show subtitles on the video
}

function getYouTubeCaptions() {
  const videoId = window.location.search.split('v=')[1];
  const captionsApiUrl = `https://video.google.com/timedtext?lang=en&v=${videoId}`;

  fetch(captionsApiUrl)
    .then(response => response.text())
    .then(text => {
      // Parse captions and extract the text
      const captions = text.match(/<body>[\s\S]*?<\/body>/g);
      const subtitleText = captions ? captions[0] : '';
      return subtitleText;
    })
    .then(captions => {
      // Translate captions into English
      chrome.runtime.sendMessage({ action: 'translate', text: captions }, (response) => {
        const translatedSubtitles = response.translatedText;
        
        // Add the translated subtitles to the video
        const vttSubtitles = `WEBVTT\n\n1\n00:00:00.000 --> 00:00:10.000\n${translatedSubtitles}`;
        addSubtitlesToVideo(vttSubtitles);
      });
    })
    .catch(error => console.error('Error fetching captions:', error));
}

// Trigger caption extraction and translation when page is loaded
window.addEventListener('load', () => {
  getYouTubeCaptions();
});

4. Popup HTML (popup.html)

This file provides the user interface to interact with the extension. Here, the user can trigger the subtitle translation.

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Translate YouTube Video</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      padding: 10px;
    }
    button {
      padding: 10px;
      font-size: 14px;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <h3>Translate YouTube Video</h3>
  <button id="translate">Translate Video Subtitles</button>

  <script>
    document.getElementById('translate').addEventListener('click', () => {
      // Send a message to the active tab to trigger subtitle translation
      chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
        chrome.scripting.executeScript(
          {
            target: { tabId: tabs[0].id },
            function: getYouTubeCaptions
          }
        );
      });
    });
  </script>
</body>
</html>

5. WebVTT Format for Subtitles (Subtitles Injection)

When injecting subtitles into the video, the subtitles will be formatted in WebVTT format:

WEBVTT

1
00:00:00.000 --> 00:00:10.000
This is the translated subtitle text.

This format includes the time range and the translated text that will be displayed on the video.
6. API Key and Integration

    For Google Translate, you'll need to sign up for Google Cloud Translation API and create an API key.
    Ensure the API key is placed in the background.js file where the translateText function makes the API call to Google Translate.

7. Optional: Handling Speech-to-Text (if no Captions are Available)

In case the video doesn't have captions available, you may want to implement a speech-to-text solution using Google's Cloud Speech API or similar services. This would involve sending the audio stream to the speech-to-text API and then translating the resulting text.
Conclusion:

This Chrome extension will allow users to:

    Translate YouTube video captions (if available) into English.
    Add translated subtitles to the video in real-time.

Next Steps:

    Testing and Debugging: You will need to test this extension thoroughly to ensure captions are fetched and translated correctly.
    Error Handling: Improve error handling to catch cases where captions are unavailable or translation fails.
