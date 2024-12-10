🧠 Second Brain AI agent
Introducing the Second Brain AI Agent Project: Empowering Your Personal Knowledge Management
Are you overwhelmed with the information you collect daily? Do you often find yourself lost in a sea of markdown files, videos, web pages, and PDFs? What if there's a way to seamlessly index, search, and even interact with all this content like never before? Welcome to the future of Personal Knowledge Management: The Second Brain AI Agent Project.

📝 Inspired by Tiago Forte's Second Brain Concept
Tiago Forte's groundbreaking idea of the Second Brain has revolutionized the way we think about note-taking. It’s not just about jotting down ideas; it's about creating a powerful tool that enhances learning and creativity. Learn more about Building a Second Brain by Tiago Forte here.

💼 What Can the Second Brain AI Agent Project Do for You?
Automated Indexing: No more manually sorting through files! Automatically index the content of your markdown files along with contained links, such as PDF documents, YouTube videos, and web pages.

Smart Search Engine: Ask questions about your content, and our AI will provide precise answers, using the robust OpenAI Large Language Model. It’s like having a personal assistant that knows your content inside out!

Effortless Integration: Whether you follow the Second Brain method or have your own unique way of note-taking, our system seamlessly integrates with your style, helping you harness the true power of your information.

Enhanced Productivity: Spend less time organizing and more time innovating. By accessing your information faster and more efficiently, you can focus on what truly matters.

✅ Who Can Benefit?
Professionals: Streamline your workflow and find exactly what you need in seconds.
Students: Make study sessions more productive by quickly accessing and understanding your notes.
Researchers: Dive deep into your research without getting lost in information overload.
Creatives: Free your creativity by organizing your thoughts and ideas effortlessly.
🚀 Get Started Today
Don't let your notes and content overwhelm you. Make them your allies in growth, innovation, and productivity. Join us in transforming the way you manage your personal knowledge and take the leap into the future.

Details
If you take notes using markdown files like in the Second Brain method or using your own way, this project automatically indexes the content of the markdown files and the contained links (pdf documents, youtube video, web pages) and allows you to ask question about your content using the OpenAI Large Language Model.

The system is built on top of the LangChain framework and the ChromaDB vector store.

The system takes as input a directory where you store your markdown notes. For example, I take my notes with Obsidian. The system then processes any change in these files automatically with the following pipeline:


From a markdown file, transform_md.py extracts the text from the markdown file, then from the links inside the markdown file, it extracts pdf, url, youtube video and transforms them into text. There is some support to extract history data from the markdown files: if there is an ## History section or the file name contains History, the file is split in multiple parts according to <day> <month> <year> sections like ### 10 Sep 2023.

From these text files, transform_txt.py breaks these text files into chunks, create a vector embeddings and then stores these vector embeddings into a vector database.

The second brain agent uses the vector database to get context for asking the question to the large language model. This process is called Retrieval-augmented generation (RAG).

In reality, the process is more complex than a standard RAG. It is analyzing the question and then using a different chain according to the intent:


Installation
You need a Python 3 interpreter, poetry and the inotify-tools installed. All this has been tested under Fedora Linux 38 on my laptop and Ubuntu latest in the CI workflows. Let me know if it works on your system.

Get the source code:

$ git clone https://github.com/flepied/second-brain-agent.git
Copy the example .env file and edit it to suit your settings:

$ cp example.env .env
Install the dependencies using poetry:

$ poetry install
There is a bug between poetry, torch and pypi, to workaround just do:

$ poetry run pip install torch
Then to use the created virtualenv, do:

$ poetry shell
systemd services
To install systemd services to manage automatically the different scripts when the operating system starts, use the following command (need sudo access):

$ ./install-systemd-services.sh
To see the output of the md and txt services:

$ journalctl --unit=sba-md.service --user
$ journalctl --unit=sba-txt.service --user
Doing a similarity search with the vector database
$ ./similarity.py "What is LangChain?" type=notes
Searching for new connections between notes
Use the vector store to find new conncetions between notes:

$ ./smart_connections.py
Launching the web UI
Launch this command to access the web UI:

$ streamlit run second_brain_agent.py
  You can now view your Streamlit app in your browser.

  Local URL: http://localhost:8502
  Network URL: http://192.168.121.112:8502
Here is an example:

Screenshot

Development
Install the extra dependencies using poetry:

$ poetry install --with test
And then run the tests, like this:

$ poetry run pytest
pre-commit
Before submitting a PR, make sure to activate pre-commit:

poetry run pre-commit install
