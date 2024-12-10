[![](https://img.shields.io/endpoint?style=plastic&url=https%3A%2F%2Fosstrack.io%2Fjson%2Ffreshness%2Fgithub%2Fflepied%2Fsecond-brain-agent%2Fmain)](https://osstrack.io/freshness/github/flepied/second-brain-agent/main)<br/>[![](https://pbs.twimg.com/profile_images/1862771229712949249/B7NC8ARY_400x400.jpg)](https://x.com/2ndBrainAIagent)

# 🧠 Second Brain AI agent

## Introducing the Second Brain AI Agent Project: Empowering Your Personal Knowledge Management

Are you overwhelmed with the information you collect daily? Do you often find yourself lost in a sea of markdown files, videos, web pages, and PDFs? What if there's a way to seamlessly index, search, and even interact with all this content like never before? Welcome to the future of Personal Knowledge Management: The Second Brain AI Agent Project.

### 📝 Inspired by Tiago Forte's Second Brain Concept

Tiago Forte's groundbreaking idea of the Second Brain has revolutionized the way we think about note-taking. It’s not just about jotting down ideas; it's about creating a powerful tool that enhances learning and creativity. Learn more about Building a Second Brain by Tiago Forte [here](https://fortelabs.com/blog/basboverview/).

### 💼 What Can the Second Brain AI Agent Project Do for You?

1. Automated Indexing: No more manually sorting through files! Automatically index the content of your markdown files along with contained links, such as PDF documents, YouTube videos, and web pages.

2. Smart Search Engine: Ask questions about your content, and our AI will provide precise answers, using the robust OpenAI Large Language Model. It’s like having a personal assistant that knows your content inside out!

3. Effortless Integration: Whether you follow the Second Brain method or have your own unique way of note-taking, our system seamlessly integrates with your style, helping you harness the true power of your information.

4. Enhanced Productivity: Spend less time organizing and more time innovating. By accessing your information faster and more efficiently, you can focus on what truly matters.

### ✅ Who Can Benefit?

* Professionals: Streamline your workflow and find exactly what you need in seconds.
* Students: Make study sessions more productive by quickly accessing and understanding your notes.
* Researchers: Dive deep into your research without getting lost in information overload.
* Creatives: Free your creativity by organizing your thoughts and ideas effortlessly.

### 🚀 Get Started Today

Don't let your notes and content overwhelm you. Make them your allies in growth, innovation, and productivity. Join us in transforming the way you manage your personal knowledge and take the leap into the future.

## Details

If you take notes using markdown files like in the Second Brain method or using your own way, this project automatically indexes the content of the markdown files and the contained links (pdf documents, youtube video, web pages) and allows you to ask question about your content using the OpenAI Large Language Model.

The system is built on top of the [LangChain](https://python.langchain.com/) framework and the [ChromaDB](https://www.trychroma.com/) vector store.

The system takes as input a directory where you store your markdown notes. For example, I take my notes with [Obsidian](https://obsidian.md/). The system then processes any change in these files automatically with the following pipeline:

```mermaid
graph TD
A[Markdown files from your editor]-->B[Text files from markdown and pointers]-->C[Text Chunks]-->D[Vector Database]-->E[Second Brain AI Agent]
```

From a markdown file, [transform_md.py](transform_md.py) extracts the text from the markdown file, then from the links inside the markdown file, it extracts pdf, url, youtube video and transforms them into text. There is some support to extract history data from the markdown files: if there is an `## History` section or the file name contains `History`, the file is split in multiple parts according to `<day> <month> <year>` sections like `### 10 Sep 2023`.

From these text files, [transform_txt.py](transform_txt.py) breaks these text files into chunks, create a vector embeddings and then stores these vector embeddings into a vector database.

The second brain agent uses the vector database to get context for asking the question to the large language model. This process is called [Retrieval-augmented generation (RAG)](https://python.langchain.com/docs/use_cases/question_answering/).

In reality, the process is more complex than a standard RAG. It is analyzing the question and then using a different chain according to the intent:

```mermaid
flowchart TD
    A[Question] --> C[/Get Intent/]
    C --> E[Summary Request] --> EA[/Extract all the chunks/] --> EB[/Summarize chunks/]
    C --> F[pdf or URL Lookup] --> FA[/Extract URL/]
    C --> D[Activity report]
    C --> G[Regular Question]
    D --> DA[/Get Period metadata/] --> DB[/Get Subject metadata/] --> DC[/Extract Question without time/] --> H[/Extract nearest documents\nfrom the vector database\nfiltered by the metadata/]
    G --> GA[/Step back question/] --> GB[/Extract nearest documents\nfrom the vector database/]
    H --> I[/Use the documents as context\nto ask the question to the LLM/]
    GB --> I
```

## Installation

You need a Python 3 interpreter, [`poetry`](https://github.com/python-poetry/install.python-poetry.org) and the `inotify-tools` installed. All this has been tested under Fedora Linux 38 on my laptop and Ubuntu latest in the CI workflows. Let me know if it works on your system.

Get the source code:

```ShellSession
$ git clone https://github.com/flepied/second-brain-agent.git
```

Copy the example .env file and edit it to suit your settings:

```ShellSession
$ cp example.env .env
```

Install the dependencies using [poetry](https://python-poetry.org/):

```ShellSession
$ poetry install
```

There is a bug between poetry, torch and pypi, to workaround just do:

```ShellSession
$ poetry run pip install torch
```

Then to use the created virtualenv, do:

```ShellSession
$ poetry shell
```

### systemd services

To install systemd services to manage automatically the different scripts when the operating system starts, use the following command (need sudo access):

```ShellSession
$ ./install-systemd-services.sh
```

To see the output of the md and txt services:

```ShellSession
$ journalctl --unit=sba-md.service --user
$ journalctl --unit=sba-txt.service --user
```

### Doing a similarity search with the vector database

```ShellSession
$ ./similarity.py "What is LangChain?" type=notes
```

### Searching for new connections between notes

Use the vector store to find new conncetions between notes:

```ShellSession
$ ./smart_connections.py
```

### Launching the web UI

Launch this command to access the web UI:

```ShellSession
$ streamlit run second_brain_agent.py
  You can now view your Streamlit app in your browser.

  Local URL: http://localhost:8502
  Network URL: http://192.168.121.112:8502
```

Here is an example:

![Screenshot](screenshot.png "Screenshot")

## Development

Install the extra dependencies using [poetry](https://python-poetry.org/):

```ShellSession
$ poetry install --with test
```

And then run the tests, like this:

```ShellSession
$ poetry run pytest
```

### pre-commit

Before submitting a PR, make sure to activate [pre-commit](https://pre-commit.com/):

```ShellSession
poetry run pre-commit install
```
