from newsapi import NewsApiClient
import tkinter as tk
from tkinter import messagebox
from datetime import datetime
import networkx as nx
from collections import defaultdict
import nltk
from nltk.tokenize import word_tokenize
from nltk.corpus import stopwords
import matplotlib.pyplot as plt

# Initialize NewsApiClient with your API key
newsapi = NewsApiClient(api_key='api_key_is_here')

# Download NLTK resources (if not already downloaded)
nltk.download('punkt')
nltk.download('stopwords')

def fetch_articles(keyword, start_date, end_date, num_articles, language):
    """Fetch a custom number of articles related to the keyword within a date range and language."""
    try:
        all_articles = []
        page = 1
        while len(all_articles) < num_articles:
            response = newsapi.get_everything(
                q=keyword,  # Search for articles related to the keyword
                language=language,  # Language for news articles
                from_param=start_date,  # Start date (YYYY-MM-DD)
                to=end_date,  # End date (YYYY-MM-DD)
                sort_by='relevancy',  # Sort by relevancy
                page=page  # Page number
            )
            if response.get('status') == 'ok':
                articles = response.get('articles', [])
                all_articles.extend(articles)
                page += 1
                if len(articles) == 0:
                    break  # No more articles available
            else:
                break
        return all_articles[:num_articles]  # Return only the requested number of articles
    except Exception as e:
        messagebox.showerror("Error", f"Error fetching articles: {e}")
        return None

def display_results(data, title):
    """Display the titles and descriptions of the fetched articles without labels."""
    if data:
        result_text = f"\n{title}:\n\n"
        for article in data:
            result_text += f"{article['title']}\n"  # Display the title
            if article['description']:  # Check if description exists
                result_text += f"{article['description']}\n\n"  # Display the description
            else:
                result_text += "\n"  # Add a newline if no description
        return result_text
    else:
        return f"No results found for {title}."

def analyze_text(text, num_nodes, num_top_words):
    """Analyze the text and visualize the co-occurrence network."""
    try:
        # Preprocessing
        tokens = word_tokenize(text.lower())
        stop_words = set(stopwords.words('english'))
        filtered_tokens = [word for word in tokens if word.isalnum() and word not in stop_words]

        # Build co-occurrence network
        window_size = 2
        co_occurrence = defaultdict(int)

        for i in range(len(filtered_tokens)):
            for j in range(i + 1, min(i + window_size + 1, len(filtered_tokens))):
                word1, word2 = filtered_tokens[i], filtered_tokens[j]
                if word1 != word2:
                    co_occurrence[(word1, word2)] += 1

        # Create network
        G = nx.Graph()
        for (word1, word2), weight in co_occurrence.items():
            G.add_edge(word1, word2, weight=weight)

        # Calculate degree centrality
        degree_centrality = nx.degree_centrality(G)

        # Sort nodes by degree centrality and select the top N nodes
        top_nodes = sorted(degree_centrality, key=degree_centrality.get, reverse=True)[:num_nodes]

        # Create a subgraph with the top N nodes
        subgraph = G.subgraph(top_nodes)

        # Visualize the subgraph
        plt.figure(figsize=(14, 10))

        # Scale node sizes based on degree centrality
        node_sizes = [degree_centrality[node] * 5000 for node in subgraph.nodes()]

        # Scale edge widths based on co-occurrence frequency (edge weights)
        edge_weights = [G[u][v]['weight'] for u, v in subgraph.edges()]
        max_edge_weight = max(edge_weights) if edge_weights else 1  # Avoid division by zero
        edge_widths = [10 * (weight / max_edge_weight) for weight in edge_weights]  # Normalize to max 10 pixels

        # Draw the network
        pos = nx.spring_layout(subgraph, k=0.5)
        nx.draw(
            subgraph,
            pos,
            with_labels=True,
            node_size=node_sizes,
            node_color="skyblue",
            font_size=10,
            edge_color="gray",
            width=edge_widths,
            alpha=0.8,
            linewidths=0.5,
        )

        # Add a title
        plt.title(f"Top {num_nodes} Nodes by Degree Centrality (Node Size ∝ Centrality, Edge Width ∝ Co-occurrence Frequency)")

        # Get the top M words by degree centrality
        top_words = sorted(degree_centrality, key=degree_centrality.get, reverse=True)[:num_top_words]
        top_centrality = [degree_centrality[word] for word in top_words]

        # Create a legend or annotation for the top M words
        legend_text = "\n".join([f"{word}: {centrality:.4f}" for word, centrality in zip(top_words, top_centrality)])
        plt.figtext(
            0.95,  # X position (right side of the figure)
            0.9,   # Y position (upper part of the figure)
            legend_text,
            fontsize=12,
            bbox=dict(facecolor="white", alpha=0.8),
            verticalalignment="top",
            horizontalalignment="right",
        )

        # Add an "Ok" button to the graph window
        def on_ok():
            plt.close()
            reset_main_window()

        # Create a button in the graph window
        ax_button = plt.axes([0.8, 0.05, 0.1, 0.075])  # Button position
        btn_ok = plt.Button(ax_button, 'Ok')
        btn_ok.on_clicked(lambda event: on_ok())

        # Show the plot
        plt.show()

    except Exception as e:
        messagebox.showerror("Error", str(e))

def reset_main_window():
    """Reset the main window to its initial state."""
    entry_keyword.delete(0, tk.END)
    entry_start_date.delete(0, tk.END)
    entry_end_date.delete(0, tk.END)
    entry_num_articles.delete(0, tk.END)
    entry_num_nodes.delete(0, tk.END)
    entry_num_top_words.delete(0, tk.END)
    language_var.set("en")  # Reset language to default
    text_results.delete(1.0, tk.END)  # Clear results text box

def start_search_and_analyze():
    """Start the search based on user input and analyze the text."""
    keyword = entry_keyword.get()
    start_date = entry_start_date.get()
    end_date = entry_end_date.get()
    num_articles = entry_num_articles.get()
    language = language_var.get()
    num_nodes = entry_num_nodes.get()
    num_top_words = entry_num_top_words.get()

    if not keyword or not start_date or not end_date or not num_articles or not num_nodes or not num_top_words:
        messagebox.showerror("Error", "Please fill all fields.")
        return

    # Validate date format
    try:
        datetime.strptime(start_date, '%Y-%m-%d')
        datetime.strptime(end_date, '%Y-%m-%d')
    except ValueError:
        messagebox.showerror("Error", "Invalid date format. Use YYYY-MM-DD.")
        return

    # Validate number of articles
    try:
        num_articles = int(num_articles)
        if num_articles <= 0:
            raise ValueError
    except ValueError:
        messagebox.showerror("Error", "Number of articles must be a positive integer.")
        return

    # Validate number of nodes and top words
    try:
        num_nodes = int(num_nodes)
        num_top_words = int(num_top_words)
        if num_nodes <= 0 or num_top_words <= 0:
            raise ValueError
    except ValueError:
        messagebox.showerror("Error", "Number of nodes and top words must be positive integers.")
        return

    # Fetch articles
    articles = fetch_articles(keyword, start_date, end_date, num_articles, language)
    if articles:
        result = display_results(articles, "Search Results")
        text_results.delete(1.0, tk.END)  # Clear previous results
        text_results.insert(tk.END, result)  # Display new results

        # Combine all article titles and descriptions into a single text for analysis
        combined_text = " ".join([article['title'] + " " + (article['description'] or "") for article in articles])

        # Analyze and visualize the text
        analyze_text(combined_text, num_nodes, num_top_words)
    else:
        text_results.delete(1.0, tk.END)
        text_results.insert(tk.END, "No articles found.")

def exit_app():
    """Close the application."""
    root.destroy()

# GUI Setup
root = tk.Tk()
root.title("News API Search and Text Analysis")

# Labels
tk.Label(root, text="Keyword:").grid(row=0, column=0, padx=10, pady=10)
tk.Label(root, text="Start Date (YYYY-MM-DD):").grid(row=1, column=0, padx=10, pady=10)
tk.Label(root, text="End Date (YYYY-MM-DD):").grid(row=2, column=0, padx=10, pady=10)
tk.Label(root, text="Number of Articles:").grid(row=3, column=0, padx=10, pady=10)
tk.Label(root, text="Language:").grid(row=4, column=0, padx=10, pady=10)
tk.Label(root, text="Number of Nodes:").grid(row=5, column=0, padx=10, pady=10)
tk.Label(root, text="Number of Top Words:").grid(row=6, column=0, padx=10, pady=10)

# Entry Fields
entry_keyword = tk.Entry(root, width=30)
entry_keyword.grid(row=0, column=1, padx=10, pady=10)

entry_start_date = tk.Entry(root, width=30)
entry_start_date.grid(row=1, column=1, padx=10, pady=10)

entry_end_date = tk.Entry(root, width=30)
entry_end_date.grid(row=2, column=1, padx=10, pady=10)

entry_num_articles = tk.Entry(root, width=30)
entry_num_articles.grid(row=3, column=1, padx=10, pady=10)

# Language Dropdown
language_var = tk.StringVar(root)
language_var.set("en")  # Default language is English
language_options = ["en", "ru", "es", "it", "de"]  # Supported languages
language_dropdown = tk.OptionMenu(root, language_var, *language_options)
language_dropdown.grid(row=4, column=1, padx=10, pady=10)

# Number of Nodes and Top Words
entry_num_nodes = tk.Entry(root, width=30)
entry_num_nodes.grid(row=5, column=1, padx=10, pady=10)

entry_num_top_words = tk.Entry(root, width=30)
entry_num_top_words.grid(row=6, column=1, padx=10, pady=10)

# Buttons
tk.Button(root, text="Search and Analyze", command=start_search_and_analyze).grid(row=7, column=0, padx=10, pady=10)
tk.Button(root, text="Exit", command=exit_app).grid(row=7, column=1, padx=10, pady=10)

# Text Box for Results
text_results = tk.Text(root, height=20, width=80)
text_results.grid(row=8, column=0, columnspan=3, padx=10, pady=10)

# Run the application
root.mainloop()
