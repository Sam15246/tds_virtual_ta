# Virtual Teaching Assistant for IIT Madras Tools in Data Science (TDS)

This project creates a Virtual Teaching Assistant for IIT Madras’ Tools in Data Science course. It uses a Retrieval-Augmented Generation (RAG) model to answer student questions on Discourse, based on course content and forum posts. Built with FastAPI and deployed on Vercel, it uses data up to April 15, 2025.

## Features

- Answers student queries using course materials and Discourse forum data.
- Utilizes a RAG pipeline for context-aware responses.
- FastAPI backend for API endpoints.
- Automated data scraping from course and forum sources.
- Data preprocessing to build a knowledge base.
- Deployable on Vercel for serverless operation.

## Technical Details & Methodology

- **Data Scraping**
  - `scrape_course.py`: Uses Playwright to crawl the course website, extract all internal links, and convert HTML content to Markdown using `markdownify`. Each page is saved as a Markdown file with metadata (title, URL, timestamp). All visited links are tracked to avoid duplication.
  - `scrape_discourse.py`: Uses Playwright and BeautifulSoup to authenticate (manual login if needed), then paginates through the Discourse forum’s JSON API to collect all topics in the TDS category within a date range. Each topic and its posts are saved as cleaned JSON files, with HTML content converted to plain text.

- **Data Preprocessing**
  -  `preprocess.py`: After scraping, this script processes the Markdown and Discourse JSON files, splits them into smaller text chunks, generates vector embeddings for each chunk (using an external embedding API), and stores them in the `knowledge_base.db` SQLite database for efficient retrieval.

- **Knowledge Base**
  - `knowledge_base.db`: SQLite database with two main tables:
    - `markdown_chunks`: Stores course content chunks, their metadata, and embeddings.
    - `discourse_chunks`: Stores forum post chunks, their metadata, and embeddings.

- **Backend API**
  - `app.py`: FastAPI app that exposes endpoints for querying the knowledge base.
    - Accepts user questions (optionally with images).
    - Generates embeddings for the query and finds the most similar content chunks using cosine similarity.
    - Aggregates and ranks results, then uses an LLM (via an external API) to generate a final answer, optionally including source links.
    - Supports multimodal queries (text + image) using GPT-4o Vision API.
    - Handles CORS, logging, and error management.

- **Deployment**
  - `vercel.json`: Configures the project for serverless deployment on Vercel.
  - The backend is stateless except for the SQLite database, which must be present and up-to-date.

- **Prompt Testing**
  - `project-tds-virtual-ta-promptfoo.yaml`: Used with Promptfoo to evaluate and refine prompt quality for the LLM.

## Project Structure

```
.
├── app.py                  # Main FastAPI application
├── knowledge_base.db       # SQLite database with course and forum data
├── scrape_course.py        # Script to scrape course content
├── scrape_discourse.py     # Script to scrape Discourse forum posts
├── preprocess.py           # Script to preprocess scraped data and build the knowledge base
├── requirements.txt        # Python dependencies
├── README.md               # Project documentation
├── LICENSE                 # License file
├── vercel.json             # Vercel deployment configuration
└── project-tds-virtual-ta-promptfoo.yaml # Promptfoo config for prompt testing
```

## Setup

1. **Clone the repository:**
   ```powershell
   git clone <repo-url>
   cd tds_virtual_ta
   ```

2. **Install dependencies:**
   ```powershell
   pip install -r requirements.txt
   ```

3. **Scrape data:**
   - To scrape course content:
     ```powershell
     python scrape_course.py
     ```
   - To scrape Discourse forum posts:
     ```powershell
     python scrape_discourse.py
     ```

4. **Preprocess data:**
   - After scraping, run the preprocessing script to create the knowledge base:
     ```powershell
     python preprocess.py
     ```

5. **Run the FastAPI server locally:**
   ```powershell
   uvicorn app:app --reload
   ```

## Deployment

- The project is configured for deployment on Vercel. See `vercel.json` for settings.
- Ensure all environment variables and database files are correctly set up before deploying.

## Usage

- Access the API endpoints as documented in the FastAPI app.
- The assistant will answer questions using the latest available data in `knowledge_base.db`.

## License

See the `LICENSE` file for details.
