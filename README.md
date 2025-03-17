# News API Search and Text Analysis Tool

This Python application allows users to search for news articles using the **NewsAPI** and perform text analysis on the fetched articles. It provides a graphical user interface (GUI) built with **Tkinter** and visualizes the co-occurrence network of words using **NetworkX** and **Matplotlib**.

---

## Features

1. **News Article Search**:
   - Fetch articles based on a keyword, date range, and language.
   - Specify the number of articles to retrieve.
   - Supported languages: English (`en`), Russian (`ru`), Spanish (`es`), Italian (`it`), and German (`de`).

2. **Text Analysis**:
   - Analyze the combined text of article titles and descriptions.
   - Build a co-occurrence network of words.
   - Visualize the network with node sizes proportional to degree centrality and edge widths proportional to co-occurrence frequency.

3. **Interactive GUI**:
   - Input fields for keyword, date range, number of articles, language, number of nodes, and number of top words.
   - Display search results in a text box.
   - Visualize the co-occurrence network in a separate window with an "Ok" button to close the graph and reset the main window.

4. **Error Handling**:
   - Validate user inputs (e.g., date format, positive integers).
   - Display error messages for invalid inputs or API errors.

---

## Requirements

- Python 3.x
- Libraries:
  - `newsapi-python` (for accessing NewsAPI)
  - `tkinter` (for GUI)
  - `networkx` (for graph creation and analysis)
  - `matplotlib` (for graph visualization)
  - `nltk` (for text preprocessing)

Install the required libraries using pip:

```bash
pip install newsapi-python tkinter networkx matplotlib nltk
```

---

## How to Use

1. **Run the Script**:
   - Execute the script in a Python environment.

2. **Input Parameters**:
   - Enter a keyword (e.g., "AI").
   - Specify the start and end dates in `YYYY-MM-DD` format.
   - Choose the number of articles to fetch.
   - Select the language (default is English).
   - Specify the number of nodes and top words for the co-occurrence network.

3. **Search and Analyze**:
   - Click the "Search and Analyze" button to fetch articles and analyze the text.
   - The results will be displayed in the text box, and the co-occurrence network will be visualized in a new window.

4. **Reset or Exit**:
   - After viewing the graph, click "Ok" to close it and reset the main window.
   - Use the "Exit" button to close the application.

---

## Code Structure

- **NewsAPI Integration**:
  - Uses `NewsApiClient` to fetch articles based on user inputs.
  - Handles pagination to retrieve the specified number of articles.

- **Text Analysis**:
  - Tokenizes and preprocesses text using `nltk`.
  - Builds a co-occurrence network using `networkx`.
  - Visualizes the network with `matplotlib`.

- **GUI**:
  - Built with `tkinter` for user interaction.
  - Includes input fields, buttons, and a text box for displaying results.

---

## Example Workflow

1. Enter the keyword "AI".
2. Set the date range to `2023-01-01` to `2023-10-01`.
3. Fetch 10 articles in English.
4. Specify 10 nodes and 5 top words for the co-occurrence network.
5. Click "Search and Analyze" to view the results and visualize the network.

---

## Notes

- Replace the placeholder API key with your own NewsAPI key.
- Ensure `nltk` resources (`punkt` and `stopwords`) are downloaded before running the script.

---

## License

This project is open-source and available under the MIT License.
