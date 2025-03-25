# LLM-based Web Scraping with ScrapeGraphAI

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/)

This guide explains how to use ScrapeGraphAI and large language models to simplify web scraping and automate data extraction.

- [Why use ScrapeGraphAI?](#why-use-scrapegraphai)
- [Prerequisites](#prerequisites)
- [Setting Up Your Environment](#setting-up-your-environment)
- [Scraping Data with ScrapeGraphAI](#scraping-data-with-scrapegraphai)
  - [Writing the Scraper Code](#writing-the-scraper-code)
  - [Using Proxies with ScrapeGraphAI](#using-proxies-with-scrapegraphai)
- [Cleaning and Preparing Data](#cleaning-and-preparing-data)

## Why use ScrapeGraphAI?

Traditional web scraping requires writing complex, time-consuming code specific to each website layout, which often breaks when sites change.

[ScrapeGraphAI](https://scrapegraphai.com/) leverages large language models (LLMs) to interpret and extract data like a human, allowing you to focus on the data rather than the layout. By integrating LLMs, ScrapeGraphAI improves data extraction, automates content aggregation, and enables real-time analysis.

## Prerequisites

You need the following prerequisites:

* [Python 3.x](https://www.python.org/downloads/).
* [An OpenAI account](https://platform.openai.com/signup) to access [GPT-4](https://openai.com/index/gpt-4/).

## Setting Up Your Environment

Create a virtual environment:

```bash
python -m venv venv
```

Then, activate the virtual environment. On macOS and Linux:

```bash
source venv/bin/activate
```

On Windows, you can use this command:

```powershell
venv\Scripts\activate
```

Install ScrapeGraphAI and its dependencies:

```bash
pip install scrapegraphai
playwright install
```

The `playwright install` command sets up the necessary browsers for Chromium, Firefox, and WebKit.

To manage environment variables securely, install `python-dotenv`:

```bash
pip install python-dotenv
```

It’s important to protect sensitive information, like API keys. To do that, store environment variables in a `.env` file to keep it separate from the code files.

Create a new file named `.env` in the project directory and add the following line specifying your OpenAI key:

```python
OPENAI_API_KEY="your-openai-api-key"
```

This file should not be committed to version control systems like Git. To prevent this, add `.env` to your `.gitignore` file.

## Scraping Data with ScrapeGraphAI

Start by scraping product data from [Books to Scrape](http://books.toscrape.com/), a demo website specifically for practicing web scraping techniques. This website mimics an online bookstore, offering a variety of books across different genres, complete with prices, ratings, and availability status:

![Books to Scrape website](https://brightdata.com/wp-content/uploads/2024/10/Books-to-Scrape-website-1024x772.png)

In traditional HTML scraping, you manually inspect elements to extract data. With ScrapeGraphAI, simply specify your desired data using a prompt, and the LLM extracts it for you.

ScrapeGraphAI offers various graphs for different scraping needs:

* **[SmartScraperGraph](https://scrapegraph-ai.readthedocs.io/en/latest/scrapers/types.html#smartscrapermultigraph)**: Single-page scraper using a prompt and URL or local file.
* **[SearchGraph](https://scrapegraph-ai.readthedocs.io/en/latest/scrapers/types.html#searchgraph)**: Multi-page scraper extracting data from search engine results.
* **[SpeechGraph](https://scrapegraph-ai.readthedocs.io/en/latest/scrapers/types.html#speechgraph)**: Extends SmartScraperGraph with text-to-speech, generating an audio file.
* **[ScriptCreatorGraph](https://scrapegraph-ai.readthedocs.io/en/latest/scrapers/types.html#scriptcreatorgraph-scriptcreatormultigraph)**: Outputs a Python script for scraping the specified URL.

You can also create custom graphs by combining nodes to fit specific needs.

To ensure accurate scraping, configure the scraper properly, including a clear prompt, model selection, proxies for georestricted content, and headless mode for efficiency. Proper setup influences the precision of your extracted data.

### Writing the Scraper Code

Create a new file named `app.py` and insert the following code:

```python
from dotenv import load_dotenv
import os
from scrapegraphai.graphs import SmartScraperGraph

# Load environment variables from .env file
load_dotenv()

# Access the OpenAI API key
OPENAI_API_KEY = os.getenv('OPENAI_API_KEY')

# Configuration for ScrapeGraphAI
graph_config = {
    "llm": {
        "api_key": OPENAI_API_KEY,
        "model": "openai/gpt-4o-mini",
 }
}

# Define the prompt and source
prompt = "Extract the title, price and availability of all books on this page."
source = "http://books.toscrape.com/"

# Create the scraper graph
smart_scraper_graph = SmartScraperGraph(
 prompt=prompt,
 source=source,
 config=graph_config
)

# Run the scraper
result = smart_scraper_graph.run()

# Output the results
print(result)
```

This code imports essential modules like `os` and `dotenv` for managing environment variables, and the `SmartScraperGraph` class from ScrapeGraphAI for scraping. It loads environment variables via `dotenv` to keep sensitive data (e.g., API keys) secure. The code then configures an LLM for scraping, specifying the model and API key. This configuration, along with the site URL and scraping prompt, defines the `SmartScraperGraph`, which is executed using the `run()` method to gather the specified data.

To run the code, use the command `python app.py` in your terminal. The output will look like this:

```
{
    "books": [
        {
            "title": "A Light in the Attic",
            "price": "£51.77",
            "availability": "In stock"
        },
        {
            "title": "Tipping the Velvet",
            "price": "£53.74",
            "availability": "In stock"
        }, ...
       ]
}
```

> **Note**:
> 
> Make sure you have [`grpcio`](https://pypi.org/project/grpcio/) package installed to avoid potential errors.

While ScrapeGraphAI makes the data extraction part of web scraping easy, there are still some common challenges, like CAPTCHAs and IP blocks.

To mimic browsing behavior, you can implement timed delays in your code. You can also utilize rotating proxies to avoid detection. Additionally, CAPTCHA-solving services like Bright Data’s CAPTCHA solver or Anti Captcha can be integrated into your scraper to automatically solve CAPTCHAs for you.

> **Important**:
> 
> Always ensure that you’re compliant with a website’s terms of service. Scraping for personal use is often acceptable, but redistributing data can have legal implications.

## Using Proxies with ScrapeGraphAI

ScrapeGraphAI lets you set up a proxy service to avoid IP blocking and access georestricted pages. To do that, add the following to your `graph_config`:

```python
graph_config = {
    "llm": {
        "api_key": OPENAI_API_KEY,
        "model": "openai/gpt-4o-mini",
 },
    "loader_kwargs": {
        "proxy": {
            "server": "broker",
            "criteria": {
                "anonymous": True,
                "secure": True,
                "countryset": {"US"},
                "timeout": 10.0,
                "max_tries": 3
 },
 },
 }
}
```

This configuration tells ScrapeGraphAI to use a free proxy service that matches your criteria.

To use a custom proxy server from a provider like Bright Data, alter your `graph_config` as follows, inserting your server URL, username, and password:

```python
graph_config = {
    "llm": {
        "api_key": OPENAI_API_KEY,
        "model": "openai/gpt-4o-mini",
 },
    "loader_kwargs": {
        "proxy": {
            "server": "http://your_proxy_server:port",
            "username": "your_username",
            "password": "your_password",
 },
 }
}
```

Using a custom proxy server offers several benefits, particularly for large-scale web scraping. It gives you control over the proxy location, enabling you to access georestricted content. Custom proxies are also more reliable and secure than free proxies, reducing the risk of IP blocks or rate-limiting.

## Cleaning and Preparing Data

After scraping, it's important to clean and preprocess the data, especially if you're using it for AI models. Clean data ensures that your models learn from accurate, consistent information, improving their performance and reliability. Data cleaning typically includes handling missing values, correcting data types, normalizing text, and removing duplicates.

Here’s an example of how to clean your scraped data using [pandas](https://pypi.org/project/pandas/):

```python
import pandas as pd

# Convert the result to a DataFrame
df = pd.DataFrame(result["books"])

# Remove currency symbols and convert prices to float
df['price'] = df['price'].str.replace('£', '').astype(float)

# Standardize availability text
df['availability'] = df['availability'].str.strip().str.lower()

# Handle missing values if any
df.dropna(inplace=True)

# Preview the cleaned data
print(df.head())
```

This code cleans the data by removing the currency symbol from the book prices, standardizes the availability status by converting it to lowercase, and handles any missing values.

Install the `pandas` library for data manipulation before running that code:

```bash
pip install pandas
```

Open your terminal and run `python app.py`. The output should look like this:

```
                                   title  price availability
0                   A Light in the Attic  51.77     in stock
1                     Tipping the Velvet  53.74     in stock
2                             Soumission  50.10     in stock
3                          Sharp Objects  47.82     in stock
4  Sapiens: A Brief History of Humankind  54.23     in stock
```

This is just an example of cleaning scraped data; the process varies based on the data and LLM use case. Cleaning ensures your language models receive structured and meaningful input.

You can find all the code for this tutorial in [this GitHub repo](https://github.com/mikeyny/scrapegraphai-demo).

## Conclusion

ScrapeGraphAI uses LLMs for adaptive web scraping, adjusting to website changes and extracting data intelligently. However, scaling scraping comes with challenges like IP blocks, CAPTCHAs, and legal compliance.

Bright Data offers solutions to address these challenges, including Web Scraper APIs, proxy services, and [Serverless Scraping](https://brightdata.com/products/web-scraper/functions). They also provide ready-to-use datasets from over a hundred popular websites.

Start your free trial today!