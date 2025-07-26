# Polymarket Insights & News Bot (n8n Workflow)

An n8n workflow to get real-time, category-filtered alerts for new markets on Polymarket, delivered instantly to your Telegram channel.

This repository contains a powerful **n8n workflow** designed to monitor the Polymarket prediction market in real-time. It fetches newly created events, intelligently filters them by category, formats the data into clean, readable messages, and delivers them instantly to a Telegram channel.

This bot serves two primary use cases:

1.  **Alpha for Bettors:** Get instant notifications about new markets in your preferred categories (e.g., Crypto, Sports), allowing you to be among the first to analyze and place bets.
2.  **Thematic News Feed:** Follow specific topics (e.g., Geopolitics, Science) to receive breaking news and global events framed as interesting predictive questions.

-----

## ✨ Key Features

  * **Real-Time Monitoring:** The workflow is scheduled to run at frequent intervals to catch new markets as soon as they are listed.
  * **Direct API Integration:** Communicates directly with the Polymarket API for fast, reliable, and structured data retrieval, avoiding fragile web scraping.
  * **Intelligent Category Filtering:** Easily configurable to include or exclude specific market categories like `Crypto`, `Sports`, `Politics`, etc.
  * **Time-Based Filtering:** Fetches only events created within a specific recent timeframe (e.g., the last hour) to ensure relevance.
  * **Robust Data Handling:** Safely handles potential issues like missing tags or malformed data to prevent workflow crashes.
  * **Professional Telegram Formatting:** Delivers alerts using Markdown for clear, well-structured, and easy-to-read messages.

-----

## ⚙️ Workflow Technical Breakdown

The n8n workflow is a sequence of nodes, each performing a specific task to transform raw API data into a useful alert.

1.  **Schedule Trigger:** Automatically starts the workflow at a defined interval (e.g., every hour).
2.  **Create Date Range (`Code` node):**
      * Calculates the ISO timestamp for a specific time in the past (e.g., 1 hour ago).
      * This creates the `start_date_min` parameter needed for the API call, ensuring we only fetch the latest events.
3.  **Fetch PolyMarket Data (`HTTP Request` node):**
      * Makes a `GET` request to the Polymarket `gamma-api/events` endpoint.
      * Uses the `start_date_min` from the previous step to query for new events.
      * The API returns a clean JSON array of event objects.
4.  **Exclude Irrelevant Tags (`Code` node):**
      * This is the core filtering logic.
      * It iterates through each event fetched from the API.
      * It checks the `tags` array of each event against a pre-defined list of categories to **exclude** (e.g., `['Daily', 'Crypto', 'Sports']`).
      * Only events that do **not** contain any of the excluded tags are passed on to the next step.
5.  **Format Telegram Message (`Code` node):**
      * This node transforms the filtered JSON data for each event into a human-readable message formatted with Telegram's Markdown.
      * It constructs the message by extracting the `title`, `slug` (for the URL), `tags`, and details for each market within the event.
      * It includes helper functions to format price changes with emojis (📈/📉) and to present dates in a clean UTC format.
      * **Crucially, it handles cases where `tags` might be missing and truncates long messages to avoid exceeding Telegram's character limit.**
6.  **Send Selected Posts (`Telegram` node):**
      * Sends the final, formatted message to the specified Telegram Chat ID.
      * **Note:** `Parse Mode` should be set to `MarkdownV2` in the node's options to ensure the formatting is rendered correctly.

-----

## 🚀 Setup and Installation Guide

1.  **Import the Workflow:**
      * Download the `PolyMarket_New_Questions.json` file from this repository.
      * In your n8n dashboard, go to `Workflows` and click `Import from File`. Upload the downloaded JSON file.
2.  **Create a Telegram Bot:**
      * In Telegram, message the `BotFather`.
      * Use the `/newbot` command to create a new bot and copy its **API Token**.
      * In n8n, go to `Credentials` -\> `Add credential`, search for "Telegram," and paste your API token.
3.  **Find Your Chat ID:**
      * To send messages to yourself, message the `@userinfobot` on Telegram to get your Chat ID.
      * To send to a channel, add your bot as an administrator to the channel. Then, send a message in the channel and forward it to `@JsonDumpBot` to get the channel's Chat ID.
4.  **Configure the Workflow:**
      * Open the **Send Selected Posts** (Telegram node) in the workflow.
      * Select your newly created Telegram credential.
      * Enter your **Chat ID** in the corresponding field.
      * In the **Additional Fields** section, set the **Parse Mode** to `MarkdownV2`.

-----

## 🔧 Customization

  * **Adjust Schedule:** Click the `Schedule Trigger` node to change how often the workflow runs.
  * **Change Filtering Logic:**
      * To **exclude** categories, add their labels to the `excludedTags` array in the **Exclude Irrelevant Tags** node.
      * To **include only specific** categories, change the code to filter for tags that *are* in your list.
  * **Modify Message Format:** Edit the `Code` node named **Format Telegram Message** to change the text, layout, or emojis in the final alert.
