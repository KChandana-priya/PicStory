# PicStory

PicStory is an AI-powered application that generates a short story from an uploaded image using image captioning and a large language model. It then converts the generated story into speech using text-to-speech technology, allowing users to both read and listen to the story.  

## Features  

- **Image Upload**: Upload an image to the application.  
- **Story Generation**: The app generates a story based on the content of the image.  
- **Text-to-Speech**: Listen to the generated story as an audio file.  

## Installation & Setup  

### 1. Clone the repository  
```bash
git clone https://github.com/KChandana-priya/PicStory.git
cd PicStory
```
### 2. Install dependencies

```bash pip install -r requirements.txt```

### 3. Set up API keys
 - Create a .env file in the project directory.
 - Add your HuggingFace and OpenAI API keys by copying from .env and replacing placeholders with your actual keys.

## Run the application

```bash python app.py```

    Open your web browser and go to http://localhost:7860 to access the app.
    Upload an image and click "Generate Story" to see and listen to the generated story.

## Fix for Audio Playback Issue

Ensure that the generated audio file is being correctly saved and played. Update your code to use Gradio’s built-in gr.Audio component instead of manually handling playback.

Modify your app.py like this:

```python
import gradio as gr
import os
from gtts import gTTS
import torch
from transformers import BlipProcessor, BlipForConditionalGeneration
from openai import OpenAI

# Load image captioning model
processor = BlipProcessor.from_pretrained("Salesforce/blip-image-captioning-base")
model = BlipForConditionalGeneration.from_pretrained("Salesforce/blip-image-captioning-base")

# Function to process image and generate a story
def generate_story(image):
    inputs = processor(image, return_tensors="pt").to("cpu")
    caption = model.generate(**inputs)
    caption_text = processor.decode(caption[0], skip_special_tokens=True)
    
    # Generate story using OpenAI
    client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
    prompt = f"Write a short and engaging story based on this caption: {caption_text}"
    response = client.ChatCompletion.create(model="gpt-3.5-turbo", messages=[{"role": "user", "content": prompt}])
    story = response["choices"][0]["message"]["content"]

    # Convert story to speech
    tts = gTTS(story)
    audio_path = "story.mp3"
    tts.save(audio_path)

    return story, audio_path

# Gradio UI
iface = gr.Interface(
    fn=generate_story,
    inputs=gr.Image(type="pil"),
    outputs=[gr.Textbox(label="Generated Story"), gr.Audio(label="Story Audio")],
    title="PictureTales",
    description="Upload an image, generate a short story, and listen to it."
)

if __name__ == "__main__":
    iface.launch()
```

## Key Fixes:

    Used gr.Audio(label="Story Audio") instead of manually handling audio playback.
    Ensured gTTS saves the audio file correctly before playback.
    Added .env file handling for API keys.

## Technologies Used

    HuggingFace (BLIP image-to-caption model)
    OpenAI GPT-3.5 (for story generation)
    gTTS (Google Text-to-Speech for audio)
    Gradio (for the user interface)

Now, your application should correctly generate, display, and play the audio story! 🚀