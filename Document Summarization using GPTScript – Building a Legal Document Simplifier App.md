使用 GPTScript 进行文档摘要 – 构建法律文档简化应用程序

###### March 15, 2024 by atulpriya sharma  

Data is the new oil—I’m sure you’ve heard this phrase a gazillion times as we witness an unprecedented explosion of data. From emails to social media posts and research papers, the sheer amount of data is a blessing and a curse.  
数据是新的石油 — 我相信您已经多次听到这句话，因为我们目睹了前所未有的数据爆炸。从电子邮件到社交媒体帖子和研究论文，庞大的数据量是福也是祸。

This information overload is difficult for individuals and organizations alike, as digging through it and finding what’s relevant to you is tedious. This is where the advent of Artificial Intelligence and large language models (LLMs) is a blessing. These help us summarize these massive amounts of data into byte-sized information using different document summarization techniques.  
这种信息过载对个人和组织来说都很困难，因为挖掘它并找到与您相关的内容是乏味的。这就是人工智能和大型语言模型 （）LLMs 的出现是一种祝福的地方。这些帮助我们使用不同的文档摘要技术将这些海量数据总结成字节大小的信息。

In this blog post, we’ll dive into document summarization to understand what it is and its challenges. We’ll also see how to use GPTScript for document summarization with a demo.  
在这篇博文中，我们将深入研究文档摘要，以了解它是什么及其挑战。我们还将通过演示了解如何使用 GPTScript 进行文档摘要。

## Document Summarization 101  
文档摘要 101

As the name suggests, document summarization is a technique that extracts the most important information from a large text document and presents a concise summary. This makes it easy for us to comprehend and make sense of large documents, which otherwise would be complex or take a lot of time.  
顾名思义，文档摘要是一种从大型文本文档中提取最重要的信息并呈现简洁摘要的技术。这使我们很容易理解和理解大型文档，否则这些文档会很复杂或需要大量时间。

For example, imagine reading a 200-page research paper. It will take a few days, if not weeks, to fully understand it. Instead, you can understand all the information in minutes by sending it to an LLM using document summarization.  
例如，想象一下阅读一篇 200 页的研究论文。完全理解它需要几天，如果不是几周的话。相反，您可以通过将信息发送到LLM使用文档摘要来在几分钟内理解所有信息。

### How it works  运作方式

There are two ways document summarization works:  
文档摘要有两种工作方式：

1. **Extractive**: In this method, we identify and extract key phrases from a large document. These extracted snippets are then stitched together to form the summary. Since exact sentences are taken from the document, it’s important to ensure that the original context is preserved.  
    **提取**式：在这种方法中，我们从大型文档中识别并提取关键短语。然后将这些提取的片段拼接在一起形成摘要。由于确切的句子取自文档，因此确保保留原始上下文非常重要。
2. **Abstractive**: This method generates new sentences to summarize the document. It involves extracting text, analyzing it using deep learning and natural language algorithms to understand it, and creating a summary from it. This is an advanced technique that often mimics human-like behavior.  
    **抽象**：此方法生成新句子来总结文档。它涉及提取文本，使用深度学习和自然语言算法对其进行分析以理解它，并从中创建摘要。这是一种高级技术，通常模仿类似人类的行为。

Document summarization techniques, as discussed above, have many advantages. They help reduce the time required to analyze documents and enable faster decision-making. These techniques highlight important texts, enabling faster document retrieval. All of this combined helps with better knowledge retention.  
如上所述，文档摘要技术具有许多优点。它们有助于减少分析文档所需的时间并实现更快的决策。这些技术突出显示重要文本，从而更快地检索文档。所有这些结合在一起有助于更好地保留知识。

### Challenges  挑战

While document summarization sounds like an easy thing to do, for computers, it is still a complex task. Some common challenges associated with document summarization are:  
虽然文档摘要听起来很容易，但对于计算机来说，它仍然是一项复杂的任务。与文档摘要相关的一些常见挑战是：

#### Context Limitation  上下文限制

Tokens are units of text that an LLM can process. Most LLMs and summarization models have a fixed number of tokens that they can process at once, commonly referred to as the maximum token limit or context window.  
标记是LLM可以处理的文本单元。大多数LLMs摘要模型都有固定数量的标记，它们可以一次处理，通常称为最大标记限制或上下文窗口。

This depends on many things, from the availability of the raw processing power to the algorithm’s efficiency. The size of the context window directly impacts the ability of models to summarize large documents effectively, as crucial information may be spread across multiple parts of the document.  
这取决于许多因素，从原始处理能力的可用性到算法的效率。上下文窗口的大小直接影响模型有效汇总大型文档的能力，因为关键信息可能分布在文档的多个部分。

Due to this limitation, they may be unable to analyze the document effectively, leading to a poor summary.  
由于此限制，他们可能无法有效地分析文档，从而导致摘要不佳。

#### Customization  定制

On the other hand, tailoring the summary to meet specific user needs is still a challenge. Every user may have a different requirement and need different kinds of summary based on their objective or interests.  
另一方面，定制摘要以满足特定的用户需求仍然是一项挑战。每个用户可能有不同的要求，并且需要根据他们的目标或兴趣提供不同类型的摘要。

For example, a CFO might analyze a market research report to understand the trends in numerical data like sales, revenues, EPS, etc. to make informed decisions. For the same report, a product manager would want to know customer feedback and category insights to better understand customer expectations.  
例如，首席财务官可能会分析市场研究报告，以了解销售额、收入、每股收益等数字数据的趋势，从而做出明智的决策。对于同一报告，产品经理希望了解买家反馈和品类洞察，以更好地了解买家的期望。

Thus, providing a one-size-fits-all summary is difficult as diverse users have diverse needs.  
因此，由于不同的用户有不同的需求，因此很难提供一刀切的摘要。

Our focus for this blog post is context limitation. One way to overcome this challenge is to have a dynamic context. You can build models that dynamically alter their context window size to fit a document.  
我们这篇博文的重点是上下文限制。克服这个挑战的一种方法是拥有动态上下文。您可以构建动态更改其上下文窗口大小的模型以适应文档。

Alternatively, you can employ document segmentation or chunking techniques to split the documents into chunks that fit within the context window and treat each chunk as an independent document and part of a whole.  
或者，您可以使用文档分段或分块技术将文档拆分为适合上下文窗口的块，并将每个块视为一个独立的文档和整体的一部分。

In the next section, we will see how we can achieve that using GPTScript.  
在下一节中，我们将了解如何使用 GPTScript 实现这一目标。

## Document Summarization using GPTScript  
使用 GPTScript 进行文档摘要

Having understood document summarization and the challenges pertaining to context windows, we’ll look at document summarization using GPTScript.  
在了解了文档摘要和与上下文窗口相关的挑战之后，我们将研究使用 GPTScript 进行文档摘要。

[GTPScript](https://github.com/gptscript-ai/gptscript) is a scripting language that allows you to automate your interactions with an LLM using natural language. With syntax in natural language, it is easy to understand and implement. One of the highlights of GPTScript is its ability to mix natural language prompts with traditional scripts, making it extremely flexible and versatile.  
[GTPScript](https://github.com/gptscript-ai/gptscript) 是一种脚本语言，可让您自动与LLM使用自然语言进行交互。使用自然语言的语法，它很容易理解和实现。GPTScript 的亮点之一是它能够将自然语言提示与传统脚本混合，使其非常灵活和通用。

### Legal Simplifier  法律简化器

To understand how document summarization using GTPScript works, we’ll build a web app called Legal Simplifier. As the name suggests, this application will simplify large legal documents that are otherwise difficult to understand.  
要了解使用 GTPScript 进行文档摘要的工作原理，我们将构建一个名为 Legal Simplifier 的 Web 应用程序。顾名思义，此应用程序将简化原本难以理解的大型法律文档。

Here’s a high-level overview of how it works:  
以下是其工作原理的高级概述：

- The app allows users to upload a large legal document in PDF format.  
    该应用程序允许用户上传 PDF 格式的大型法律文件。
- The GPTScript employs a Python script that reads this document and “creates chunks” based on a predefined context size until it reaches the end of the file.  
    GPTScript 使用一个 Python 脚本来读取此文档并根据预定义的上下文大小“创建块”，直到它到达文件末尾。
- Each chunk is sent to the LLM (OpenAI in this case), and a summary is returned and stored in a markdown file `summary.md`.  
    每个块都发送到LLM（在本例中为 OpenAI），并返回摘要并将其存储在 `summary.md` 的 markdown 文件中。
- The summary for each chunk is then analyzed as a whole, and a final summary is generated.  
    然后，将每个块的摘要作为一个整体进行分析，并生成最终摘要。
- This summary is then displayed to the user on the screen.  
    然后，此摘要将在屏幕上显示给用户。

I’ve made the application available in the examples section of the GPTScript examples repo, but I’ll also walk through how I built it so you can build your own document summarizer.  
我已在 GPTScript 示例存储库的示例部分提供了该应用程序，但我还将介绍如何构建它，以便您可以构建自己的文档摘要器。

#### Pre-requisites  先决条件

Before you can start, make sure you have all of the following:  
在开始之前，请确保您具备以下所有条件：

- An OpenAI API Key  OpenAI API 密钥
- Python 3.8 or later  Python 3.8 或更高版本
- Flask  猪肉

#### Writing the GPTScript  编写 GPTScript

The first step to building the Legal Simplifier is to create the GPTScript. Since GPTScript is written primarily in natural language, it’s very easy to write the script for summarizing documents. Below is the `legalsimplifier.gpt` script.  
构建 Legal Simplifier 的第一步是创建 GPTScript。由于 GPTScript 主要用自然语言编写，因此编写用于总结文档的脚本非常容易。下面是 `legalsimplifier.gpt` 脚本。

```json
tools: legal-simplifier, sys.read, sys.write
You are a program that is tasked with analyizing a legal document and creating a summary of it.
Create a new file "summary.md" if it doesn't already exist.
Call the legal-simplifier tool to get each part of the document. Begin with index 0.
Do not proceed until the tool has responded to you.
Once you get "No more content" from the legal-simplifier stop calling it.
Then, print the contents of the summary.md file.

---
name: legal-simplifier
tools: doc-retriever, sys.read, sys.append
description: Summarizes a legal document
As a legal expert and practicing lawyer, you are tasked with analyzing the provided legal document.
Your deep understanding of the law equips you to simplify and summarize the document effectively.
Your goal is to make the document more accessible for non-experts, leveraging your expertise to provide clear and concise insights.
Get the part of legal document at index $index.
Read the existing summary of the legal document up to this point in summary.md.
Do not leave out any important points focusing on key points, implications, and any notable clauses or provisions.
Do not introduce the summary with "In this part of the document", "In this segment", or any similar language.
Give a list of all the terms and explain them in one liner before writing the summary in the document.
For each summary write in smaller chunks or add bullet points if required to make it easy to understand.
Use headings and sections breaks as required.
Use the heading as "Summary" as only once in the entire document.
Explain terms in simple language and avoid legal terminologies until unless absolutely necessary.
Add two newlines to the end of your summary and append it to summary.md.
If you got "No more content" just say "No more content". Otherwise, say "Continue".

---
name: doc-retriever
description: Returns a part of the text of legal document. Returns "No more content" if the index is greater than the number of parts.
args: index: (unsigned int) the index of the part to return, beginning at 0
#!python3 main.py "$index"
```

Let’s understand what we’re doing in this gptscript:  
让我们了解一下我们在这个 gptscript 中做了什么：

1. In the first part, we define what the model is supposed to do. It creates a new `summary.md` file and calls the `legal-simplifier` tool to retrieve and analyze the document at index zero and go until the end of the file when it should print the summary into a `summary.md` file.  
    在第一部分中，我们定义模型应该做什么。它创建一个新的 `summary.md` 文件，并调用 `legal-simplifier` 工具来检索和分析索引 0 处的文档，直到文件末尾，它应该将摘要打印成`一个 summary.md` 文件。
2. In the second part, we define the `legal-simplifier` tool. This tool is tasked to analyze the ‘chunk’ of the document. The prompt is in natural language and is self-explanatory.  
    在第二部分中，我们定义了`法律简化工具`。该工具的任务是分析文档的 “块”。提示是自然语言的，不言自明。
3. In the last part, we define the `doc-retriever` tool that calls a Python script `main.py`, which goes through the document and creates `chunks`, which is returned to the `doc-retriever` for analysis.  
    在最后一部分中，我们定义了 `doc-retriever` 工具，该工具调用 Python 脚本 `main.py`，该工具遍历文档并创建`块`，并将其返回给 `doc-retriever` 进行分析。

#### Document Segmentation  文档分割

The Python script reads the PDF file uploaded by the user and breaks it into chunks. The crux of the script lies in the `TokenTextSplitter` that is responsible for breaking the documents into chunks based on the context size provided for the `gpt-4-turbo-preview` model.  
Python 脚本读取用户上传的 PDF 文件并将其分解为块。该脚本的关键在于 `TokenTextSplitter`，它负责根据为 `gpt-4-turbo-preview` 模型提供的上下文大小将文档分解为块。

Below is a part of the main.py script.  
以下是 main.py 脚本的一部分。

```python

…
# Initializing a TokenTextSplitter object with specified parameters
splitter = TokenTextSplitter(
	chunk_size=10000,
	chunk_overlap=10,
	tokenizer=tiktoken.encoding_for_model("gpt-4-turbo-preview").encode)
…

```

### Executing the GPTScript  执行 GPTScript

To execute this script, you must first configure the `OPENAI_API_KEY` environment variable.  
要执行此脚本，您必须首先配置 `OPENAI_API_KEY` 环境变量。

The script requires an image named `legal.pdf` in the current directory. So you can find a large legal document and save it with the name `legal.pdf` in the directory. We’ll use the [Indian Contact Act document](https://lddashboard.legislative.gov.in/sites/default/files/A1872-09.pdf) for analysis.  
该脚本需要当前目录中名为 `legal.pdf` 的图像。因此，您可以找到一个大型法律文档，并将其与名称 `legal.pdf` 一起保存在目录中。我们将使用 [Indian Contact Act 文档](https://lddashboard.legislative.gov.in/sites/default/files/A1872-09.pdf)进行分析。

Execute the script using the following command:  
使用以下命令执行脚本：

`gptscript legalsimplifier.gpt`

- The script uses `doc-retriever` tool to call the `main.py` script to chunk the document.  
    该脚本使用 `doc-retriever` 工具调用 `main.py` 脚本来对文档进行分块。
- These chunks are analyzed by `legal-simplifier` tool, and the summary is saved in `summary.md`  
    这些块由 `legal-simplifier` 工具进行分析，并将摘要保存在 `summary.md`

The whole process takes a few minutes to complete based on the size of the document, and you can see the output in the terminal or the `summary.md` file.  
根据文档的大小，整个过程需要几分钟才能完成，您可以在终端或 `summary.md` 文件中看到输出。

## Building a UI in Python  
在 Python 中构建 UI

With the scripts running as expected, let us add a UI. We’ll be using Flask to build one, but you can use any other framework.  
脚本按预期运行后，我们添加一个 UI。我们将使用 Flask 来构建一个，但你可以使用任何其他框架。

The web app has a simple UI that allows the user to upload a file to the directory and then execute the `legalsimplifier.gpt` script using [subprocess](https://docs.python.org/3/library/subprocess.html).  
Web 应用程序具有一个简单的 UI，允许用户将文件上传到目录，然后使用 [subprocess](https://docs.python.org/3/library/subprocess.html) 执行 `legalsimplifier.gpt` 脚本。

#### Backend Logic  后端逻辑

Below are the steps that were involved in building this:  
以下是构建此内容所涉及的步骤：

- After the environment setup, the first step was to integrate `legalsimplifier.gpt` in Python. I created an `app.py` file and used Python’s [subprocess](https://docs.python.org/3/library/subprocess.html) capability to invoke the GPTScript. I defined the path to the GPTScript and used `prun` to invoke the script using `gptscript` prompt  
    环境设置完成后，第一步是在 Python 中集成 `legalsimplifier.gpt`。我创建了一个 `app.py` 文件，并使用 Python 的[子进程](https://docs.python.org/3/library/subprocess.html)功能来调用 GPTScript。我定义了 GPTScript 的路径，并使用 `prun` 通过 `gptscript` 提示符调用脚本
    
    ```python
      ```python
        # Execute the script to summarize the legal document
              	subprocess.run(f"gptscript {SCRIPT_PATH}", shell=True, check=True) 
      ```
    ```
    
- Once I got the `legalsimplifier.gpt` to work from my Python app, I added Flask to the project and made necessary changes with respect to routes. Added a `/` route that would execute when the app is launched and another `/upload` route that will execute when the file is uploaded.  
    在我让 `legalsimplifier.gpt` 从我的 Python 应用程序中工作后，我将 Flask 添加到项目中，并对路由进行了必要的更改。添加了一个 `/` 路由，该路由将在应用程序启动时执行，另一个 `/upload` 路由将在上传文件时执行。
    
- Finally, I added the logic to handle file upload. This function takes the file uploaded by the user and saves it as `legal.pdf`, which will be used by `legalsimplifier.gpt`.  
    最后，我添加了处理文件上传的逻辑。此函数获取用户上传的文件并将其保存为 `legal.pdf`，这将由 `legalsimplifier.gpt` 使用。
    

#### Frontend UI  前端 UI

- After the backend was ready, it was time to build the frontend. I used simple HTML, CSS, JS, and Bootstrap to build the front end.  
    后端准备好后，就该构建前端了。我使用简单的 HTML、CSS、JS 和 Bootstrap 来构建前端。
- Since we are using Flask, all the UI-related files go to `templates` directory. So created an index. html file with a few elements.  
    由于我们使用的是 Flask，因此所有与 UI 相关的文件都转到 `templates` 目录。所以创建了一个索引。html 文件，其中包含一些元素。
- The page also has client-side logic written in JS to take the form input, send it to the backend, get the response, and display it on the screen.  
    该页面还具有用 JS 编写的客户端逻辑，用于获取表单输入、将其发送到后端、获取响应并将其显示在屏幕上。

_Pro Tip: You can even use ChatGPT to build the UI for this.  
专业提示：您甚至可以使用 ChatGPT 为此构建 UI。_

Once both these parts are ready, execute the application. If you’re using an IDE like VSCode, simply hit F5, or you can execute `flask run` or `python app.py`.  
这两个部分都准备好后，执行应用程序。如果您使用的是 VSCode 等 IDE，只需按 F5，或者您可以执行 `flask run` 或 `python app.py`。

![[doc-1.png]]
The application will be available at [http://127.0.0.1:5000/](http://127.0.0.1:5000/). Go ahead and upload a PDF file. The GPTScript runs in the background, segments the document, analyzes it, and shows a concise summary.  
该应用程序将于 [http://127.0.0.1:5000/](http://127.0.0.1:5000/) 年提供。继续上传 PDF 文件。GPTScript 在后台运行，对文档进行分段、分析并显示简洁的摘要。

![[doc-2.jpg]]
## Summary  总结

In this blog post we saw how document summarization plays a crucial role in analyzing and summarizing large documents. The technique helps us navigate through the context limitations of LLMs and still helps summarize documents precisely.  
在这篇博文中，我们看到了文档摘要如何在分析和总结大型文档方面发挥关键作用。该技术帮助我们了解上下文限制LLMs，并且仍然有助于精确总结文档。

Not only theory, but we also also how we are able to implement document summarization using GPTScript by using custom Python script along with natural language prompts.  
不仅是理论，我们还知道如何通过使用自定义 Python 脚本和自然语言提示来使用 GPTScript 实现文档摘要。

Give [GPTScript](https://github.com/gptscript-ai/gptscript) a try to see how simple it is to build AI applications. If you’re new here or need guidance, join [GPTScript discord server](https://discord.gg/9sSf4UyAMC) and get all your queries answered.  
试试 [GPTScript](https://github.com/gptscript-ai/gptscript)，看看构建 AI 应用程序有多么简单。如果您是新手或需要指导，请加入 [GPTScript discord 服务器](https://discord.gg/9sSf4UyAMC)并回答您的所有问题。