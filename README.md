# Ved-Purana-BOT
To create a bot that answers queries about the Ramayana, Mahabharata, Vedas, and Puranas based on the PDFs linked on the provided page, we need to break the task into a few key steps:

    Download the PDFs: The first task is to programmatically download all the PDFs available on the page. You can use libraries like requests or beautifulsoup to scrape the page and extract the links to the PDF files.

    Extract Text from PDFs: After downloading the PDFs, we need to extract the text from them. We can use libraries like PyMuPDF (fitz), PyPDF2, or pdfplumber to extract text from the PDFs.

    Preprocess the Extracted Text: Once text extraction is done, we need to clean and preprocess the data (e.g., split it into sections like chapters, verses, etc.).

    Create a Query Interface: Use a simple chatbot interface (for example, using Flask or FastAPI) to accept queries, search through the extracted text, and respond with the relevant text.

    Search Mechanism: Implement a search mechanism to search for specific verses or topics in the extracted text. We can use keyword matching or advanced natural language processing (e.g., using OpenAI's GPT model for more natural conversations).

Detailed Implementation

Here is the Python code for each step:
1. Scraping PDF Links

pip install beautifulsoup4 requests

import requests
from bs4 import BeautifulSoup

def get_pdf_links(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')

    # Find all links to PDF files
    pdf_links = []
    for link in soup.find_all('a', href=True):
        href = link['href']
        if href.endswith('.pdf'):
            pdf_links.append(href)

    return pdf_links

# Example usage
url = "https://vedpuran.net/download-all-ved-and-puran-pdf-hindi-free/"
pdf_links = get_pdf_links(url)
print(pdf_links)

This script will extract all links to the PDF files available on the webpage.
2. Downloading the PDFs

You can use the requests library to download the PDFs from the links.

import os

def download_pdfs(pdf_links, download_folder):
    if not os.path.exists(download_folder):
        os.makedirs(download_folder)

    for link in pdf_links:
        filename = link.split("/")[-1]
        filepath = os.path.join(download_folder, filename)
        response = requests.get(link)

        with open(filepath, 'wb') as file:
            file.write(response.content)
        print(f"Downloaded: {filename}")

# Example usage
download_folder = "pdfs"
download_pdfs(pdf_links, download_folder)

This function will download the PDFs to the pdfs folder.
3. Extracting Text from PDFs

Now that the PDFs are downloaded, we need to extract the text. We'll use the PyMuPDF (fitz) library for text extraction.

pip install pymupdf

import fitz  # PyMuPDF

def extract_text_from_pdfs(pdf_folder):
    pdf_texts = {}
    
    for filename in os.listdir(pdf_folder):
        if filename.endswith(".pdf"):
            pdf_path = os.path.join(pdf_folder, filename)
            doc = fitz.open(pdf_path)
            text = ""
            for page_num in range(len(doc)):
                page = doc.load_page(page_num)
                text += page.get_text("text")
            pdf_texts[filename] = text
    
    return pdf_texts

# Example usage
pdf_texts = extract_text_from_pdfs(download_folder)
print(pdf_texts.keys())  # Print names of PDFs

This function will extract text from all the PDFs and store them in a dictionary where the keys are the filenames.
4. Preprocessing Text

After extracting the text, we should preprocess it (e.g., splitting it into chapters or verses, removing unwanted characters, etc.).

import re

def preprocess_text(text):
    # For example, split text into paragraphs, or look for patterns for verses
    verses = re.split(r"\d+\.", text)  # Assuming verses are numbered like 1., 2., etc.
    return [verse.strip() for verse in verses if len(verse.strip()) > 0]

# Example usage
preprocessed_texts = {filename: preprocess_text(text) for filename, text in pdf_texts.items()}
print(preprocessed_texts['ramayana.pdf'][:3])  # Print first 3 verses of Ramayana

This function preprocesses the extracted text into verses.
5. Querying Mechanism

For a basic keyword-based querying mechanism, we can simply search for keywords in the preprocessed text. A more advanced solution would use a language model like GPT for more advanced query-answering.

def search_text(query, preprocessed_texts):
    query = query.lower()
    results = []

    for filename, verses in preprocessed_texts.items():
        for index, verse in enumerate(verses):
            if query in verse.lower():
                results.append({
                    'source': filename,
                    'verse_index': index + 1,
                    'verse': verse
                })

    return results

# Example usage
query = "dharma"
results = search_text(query, preprocessed_texts)

for result in results:
    print(f"Source: {result['source']}, Verse {result['verse_index']}: {result['verse']}")

This function will search the preprocessed texts for the keyword and return relevant verses.
6. Flask Web Application for Chatbot Interface

Finally, we can wrap this all up into a simple Flask web application to create a chatbot interface.

pip install Flask

from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/ask", methods=["POST"])
def ask():
    data = request.json
    query = data.get('query')

    results = search_text(query, preprocessed_texts)

    if results:
        response = [{"source": result['source'], "verse": result['verse']} for result in results]
        return jsonify({"answer": response})
    else:
        return jsonify({"answer": "Sorry, no relevant information found."})

if __name__ == "__main__":
    app.run(debug=True)

Run the app:

python app.py

You can then query the chatbot using a POST request to /ask with a JSON payload like:

{
    "query": "dharma"
}

7. Conclusion

This code helps you:

    Scrape PDF links from the provided website.
    Download the PDFs to your local machine.
    Extract text from those PDFs.
    Preprocess the extracted text into manageable pieces like verses.
    Implement a basic querying system to search for specific terms or concepts within the texts.
    Create a Flask-based API to interact with the system via queries.

You can further enhance this system by adding more advanced NLP models, integrating an advanced search engine, or using OpenAI GPT to generate more conversational responses based on the context of the queries.
