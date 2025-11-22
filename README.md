# 🎬 AI UGC Video Ad Generator

This **n8n workflow** automates the production of **User Generated Content (UGC)** style video ads. By inputting product details and photos into a Google Sheet, the workflow orchestrates multiple AI models to generate realistic, viral-style video creative automatically.

It leverages **Kie.AI** for video generation (Veo 3.1, Sora 2) and image editing (NanoBanana), alongside **OpenRouter** and **OpenAI** for scriptwriting and visual analysis.

## 🚀 Features

* **Google Sheets as CMS:** Manage your production queue, prompts, and statuses directly from a spreadsheet.
* **Multi-Model Support:** Selectable generation paths based on the "Model" column:
    * **Veo 3.1:** Direct text-to-video generation for quick results.
    * **NanoBanana + Veo 3.1:** A hybrid pipeline that first places the product in a human hand using AI image editing, analyzes the scene using GPT-4o Vision, and then animates it into a video.
    * **Sora 2:** High-fidelity video generation from product reference images.
* **Intelligent Scripting:** Uses AI Agents to write natural, "influencer-style" scripts and visual prompts based on your Ideal Customer Profile (ICP).
* **Asynchronous Processing:** Handles long-running generation tasks with smart wait-and-check loops.
* **Vision Analysis:** Automatically analyzes intermediate images to ensure the video prompt matches the generated scene perfectly.

## 🛠️ Prerequisites

To use this workflow, you need:

1.  **n8n**: Self-hosted or Cloud (Version 1.0+ recommended).
2.  **Kie.AI API Key**: For accessing Veo, Sora, and NanoBanana models.
3.  **OpenAI API Key**: For GPT-4o (Vision capabilities).
4.  **OpenRouter API Key**: For the LLM used in prompt generation (default is GPT-5-mini).
5.  **Google Cloud Credentials**: OAuth2 credentials to read/write to Google Sheets.

## ⚙️ Setup Guide

### 1. Google Sheet Setup
You must use the specific template structure for the workflow to function correctly.
👉 **[Copy the Google Sheet Template Here](https://docs.google.com/spreadsheets/d/15WIJkzbXbs-bj8BLaUBDNE8HwUCF1-Z_0MZmKE18byA/copy)**

**Columns required:**
* `No`: Unique ID.
* `Product`: Name of the product.
* `Product Photo`: Public URL of the product image.
* `ICP`: Description of the Ideal Customer Profile.
* `Product Features`: Key features to highlight.
* `Video Setting`: Where the video takes place (e.g., "Kitchen", "Gym").
* `Model`: Must be one of: `Veo 3.1`, `Nano + Veo 3.1`, or `Sora 2`.
* `Status`: Set to `Ready` to trigger the workflow.
* `Finished Video`: The workflow will paste the final video URL here.

### 2. Import Workflow
1.  Download the `workflow.json` from this repository.
2.  Open your n8n dashboard.
3.  Click **Add Workflow** -> **Import from File**.

### 3. Configure Credentials
You will need to update the credentials in the following nodes:
* **Google Sheets Nodes:** Connect your Google Drive account.
* **HTTP Request Nodes (Kie.AI):** Add your Kie.AI API Key to the Header Authentication (usually `Authorization: Bearer <YOUR_KEY>`).
* **Chat Model (OpenRouter):** Add your OpenRouter API key.
* **Analyze Image (OpenAI):** Add your OpenAI API key.

## 🔄 Workflow Logic

The workflow operates in three distinct paths based on the **Model** selected in the Google Sheet:

### Path A: Veo 3.1
1.  **Prompt Agent:** Generates a video prompt based on the product info.
2.  **Generation:** Sends request to Veo 3.1.
3.  **Loop:** Checks status until success.
4.  **Update:** Writes the video URL back to the sheet.

### Path B: Nano + Veo (Advanced)
1.  **Image Agent:** Creates a prompt to "hold" the product.
2.  **NanoBanana:** Generates a static image of a human holding the product.
3.  **Vision Analysis:** GPT-4o analyzes the generated image to understand the context/gripping.
4.  **Video Agent:** Writes a video prompt based on the *actual* generated image.
5.  **Veo 3.1:** Animates the static image into a video.

### Path C: Sora 2
1.  **Sora Agent:** Optimizes the prompt specifically for Sora's physics engine.
2.  **Generation:** Sends request to Sora 2 (Image-to-Video).
3.  **Loop:** Checks status until success.
