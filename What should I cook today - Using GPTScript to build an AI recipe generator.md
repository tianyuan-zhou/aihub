
我今天应该做什么？使用 GPTScript 构建 AI 配方生成器

###### March 15, 2024 by atulpriya sharma  

November 30, 2022, marked the start of the GenAI revolution with the birth of ChatGPT. I remember being in awe of it and bombarding it with queries of all types. I also saw my team utilize ChatGPT for technical tasks, from coding to unit testing.  
2022 年 11 月 30 日，随着 ChatGPT 的诞生，标志着 GenAI 革命的开始。我记得我对它感到敬畏，并用各种类型的查询轰炸它。我还看到我的团队使用 ChatGPT 完成从编码到单元测试的技术任务。

Personally, I felt that ChatGPT was replacing Google for me. If you look at my Google search history, you’ll mostly find "_What do I eat?_" queries. So, I started asking ChatGPT to help me. And I was surprised that some combinations it suggested turned out to be good! But it was a tedious process to type in the ingredients all the time and wait for a response.  
就个人而言，我觉得 ChatGPT 正在取代我 Google。如果你查看我的 Google 搜索历史，你大多会发现 “_What do I eat？_” 的查询。所以，我开始请求 ChatGPT 帮助我。我很惊讶它建议的一些组合竟然很好！但一直输入成分并等待回复是一个乏味的过程。

To make the process more intuitive, I decided to cook up an app that analyzes uploaded photos to suggest recipes based on available ingredients.  
为了让这个过程更直观，我决定制作一个应用程序，它可以分析上传的照片，根据可用的成分推荐食谱。

I knew I had to leverage OpenAI’s vision API, but how could I do that without going through the learning curve? That’s when I came across [this tweet](https://x.com/ibuildthecloud/status/1757789265264796003?s=20) about GPTScript that simplifies interaction with OpenAI APIs.  
我知道我必须利用 OpenAI 的视觉 API，但是如果不经过学习曲线，我怎么能做到这一点呢？就在那时，我看到了这条关于 GPTScript 的[推文](https://x.com/ibuildthecloud/status/1757789265264796003?s=20)，它简化了与 OpenAI API 的交互。

In this blog post, I’ll tell you how I built RecipAI using GPTScript to analyze a photograph for ingredients and, based on the ingredients, suggest a recipe.  
在这篇博文中，我将告诉您如何使用 GPTScript 构建 RecipAI 来分析成分照片，并根据成分建议食谱。

## But First – What Is GPTScript?  
但首先 - 什么是 GPTScript？

Most of us have interacted with ChatGPT at least once and found it fascinating. While the interactive UI is good for generating text, getting coding assistance, and performing similar tasks, leveraging the full potential effectively requires understanding the nuances of crafting prompts and interacting with APIs, which means that it’s a steep learning curve.  
我们大多数人都至少与 ChatGPT 互动过一次，并发现它很有趣。虽然交互式 UI 有利于生成文本、获得编码帮助和执行类似任务，但要有效地利用全部潜力，需要了解制作提示和与 API 交互的细微差别，这意味着这是一个陡峭的学习曲线。

Enter [GPTScript](https://gptscript.ai/), a scripting language that is designed to bridge this gap. It is tailored to automate your interactions with an LLM like OpenAI. What makes GPTScript so interesting is that it uses natural language as the programming language for building apps powered by LLMs.  
进入 [GPTScript](https://gptscript.ai/)，这是一种旨在弥合这一差距的脚本语言。它专为自动化您与 OpenAI 等 LLM。GPTScript 之所以如此有趣，是因为它使用自然语言作为编程语言来构建由 LLMs。

### How does GPTScript work?  GPTScript 是如何工作的？

At the core of GPTScript lies tools. Tools are like functions that perform a particular set of actions. And just like function calls, tools can call other tools as well. These tools are also written using natural language prompts. The best part is using custom commands to invoke a program instead of natural language.  
GPTScript 的核心在于工具。工具类似于执行一组特定操作的函数。就像函数调用一样，工具也可以调用其他工具。这些工具也是使用自然语言提示编写的。最好的部分是使用自定义命令而不是自然语言来调用程序。

All these tools are written in a ".gpt" file, and each file can have multiple tools. The decision of whether a tool will be executed or not is determined by the AI model. Read more about [how GPTScript works](https://github.com/gptscript-ai/gptscript?tab=readme-ov-file#example).  
所有这些工具都写在一个 “.gpt” 文件中，每个文件可以有多个工具。是否执行工具的决定由 AI 模型决定。详细了解 [GPTScript 的工作原理](https://github.com/gptscript-ai/gptscript?tab=readme-ov-file#example)。

With GPTScript, users can build powerful tools using natural language or seamlessly mix natural language prompts with conventional scripting elements to interact with LLMs. This makes it a good option for anyone who wants to harness the power of generative AI without the traditional barriers. You can read more on the project’s GitHub page about [how GPTScript works](https://gptscript.ai/).  
借助 GPTScript，用户可以使用自然语言构建强大的工具，或者将自然语言提示与传统脚本元素无缝混合，以与 LLMs。对于任何想要在没有传统障碍的情况下利用生成式 AI 的力量的人来说，这都是一个不错的选择。您可以在该项目的 GitHub 页面上阅读有关 [GPTScript 工作原理](https://gptscript.ai/)的更多信息。

## Building RecipAI – a smart recipe generator  
构建 RecipAI – 智能配方生成器

The idea of building the recipe generator stemmed from my constant battle with the question, “What to cook today?”. While I had used ChatGPT in the past to get some ideas, the biggest pain was telling it all the ingredients I had in my kitchen or had bought at the store. When I first saw GPTScript, I thought I might be able to whip up an app that would make the process much easier by uploading a photo of the ingredients and letting AI tell me what to do; I had to short-circuit the process.  
构建食谱生成器的想法源于我不断与这个问题的斗争，“今天要做什么？虽然我过去曾使用 ChatGPT 来获取一些想法，但最大的痛苦是告诉它我厨房里的所有食材或在商店买的所有食材。当我第一次看到 GPTScript 时，我想我也许能够制作一个应用程序，通过上传成分的照片并让 AI 告诉我该怎么做，让这个过程变得更加容易;我不得不缩短这个过程。

## How RecipAI works  RecipAI 的工作原理

I built the application in python, using [GPTScript](https://github.com/gptscript-ai/gptscript) under the hood, and powered by OpenAI APIs. It uses [GPTScript vision](https://github.com/gptscript-ai/vision) for analyzing an image and GPTScript to generate a recipe based on the ingredients.  
我在 python 中构建了应用程序，在后台使用 [GPTScript](https://github.com/gptscript-ai/gptscript)，并由 OpenAI API 提供支持。它使用 [GPTScript 视觉](https://github.com/gptscript-ai/vision)来分析图像，并使用 GPTScript 根据成分生成食谱。

Below is a high-level overview of how it works:  
以下是其工作原理的高级概述：

1. A user takes/uploads a photo of groceries and uploads it to the web app.  
    用户拍摄/上传杂货照片并将其上传到 Web 应用程序。
2. Using GPTScript Vision, it analyzes the photo and identifies the ingredients.  
    它使用 GPTScript Vision 分析照片并识别成分。
3. These ingredients are taken as inputs by GPTScript to generate a recipe.  
    GPTScript 将这些成分作为输入来生成配方。

I’ve made the application available in the examples section of the [GPTScript examples repo](https://github.com/gptscript-ai/gptscript/tree/main/examples/recipegenerator), but I’ll also walk through the process of how I built it so you can build your own or something similar.  
我已经在 [GPTScript 示例存储库](https://github.com/gptscript-ai/gptscript/tree/main/examples/recipegenerator)的示例部分提供了该应用程序，但我还将介绍我如何构建它的过程，以便您可以构建自己的应用程序或类似应用程序。

## Prerequisites  先决条件

Before you can start, make sure you have all of the following:  
在开始之前，请确保您具备以下所有条件：

- An OpenAI API Key  OpenAI API 密钥
- Python 3.8 or later  Python 3.8 或更高版本
- Node.js and npm  Node.js 和 npm
- Flask  猪肉

## Writing the GPTScript  编写 GPTScript

Creating the GPTScript is the first step in building the Recipe Generator. Since GPTScript is written primarily in natural language, it’s very easy to write the script for generating recipes. Below is the `recipegenerator.gpt` script  
创建 GPTScript 是构建配方生成器的第一步。由于 GPTScript 主要用自然语言编写，因此编写生成配方的脚本非常容易。下面是 `recipegenerator.gpt` 脚本

```json

tools: sys.find, sys.read, sys.write, recipegenerator, tool.gpt

Perform the following steps in order:
1. Give me a list of 5 to 10 ingredients from the picture grocery.png located in the current directory and put those extracted ingredients into a list.
2. Based on these ingredients list, suggest one recipe that is quick to cook and create a new recipe.md file with the recipe

---

name: recipegenerator
description: Generate a recipe from the list of ingredients
args: ingredients: a list of available ingredients.
tools: sys.read
Generate 1 new recipe based on the ingredients list

```

The script calls `tool.gpt`, part of GPTScript vision, the default tool for Vision CLI. So we are essentially using two tools here, the default `tool.gpt` and our custom tool named `recipegenerator` which calls tool.gpt to analyze the image. Apart from that, the script is self-explanatory.  
该脚本调用 `tool.gpt`，它是 GPTScript vision 的一部分，是 Vision CLI 的默认工具。所以我们在这里基本上使用了两个工具，默认的 `tool.gpt` 和名为 `recipegenerator` 的自定义工具，它调用 tool.gpt 来分析图像。除此之外，该脚本是不言自明的。

We’re asking the model to:  
我们要求模型：

1. Analyze the image and identify 5-10 ingredients from it.  
    分析图像并从中识别 5-10 种成分。
2. Analyze these ingredients and generate a recipe.  
    分析这些成分并生成食谱。

## Executing the GPTScript  执行 GPTScript

To execute this script, you must first configure the `OPENAI_API_KEY` environment variable. Further, you need to install the npm modules required by GPTScript vision CLI. You can simply run `npm install` in the directory, and it will install all the dependencies.  
要执行此脚本，您必须首先配置 `OPENAI_API_KEY` 环境变量。此外，您需要安装 GPTScript vision CLI 所需的 npm 模块。您只需在目录中运行 `npm install`，它就会安装所有依赖项。

With this, our recipegenerator.gpt is ready to be executed.  
这样，我们的 recipegenerator.gpt 就可以执行了。

The script requires an image named `grocery.png` in the current directory. So you can click a picture or use any image and save it with the name `grocery.png` in the directory. We’ll use the following image as an example:  
该脚本需要当前目录中名为 `grocery.png` 的映像。因此，您可以单击图片或使用任何图像并将其命名为 `grocery.png` 保存在目录中。我们将使用下图作为示例：

![[cook.jpg]]
Execute the script using the following command:  
使用以下命令执行脚本：

`gptscript recipegenerator.gpt`

- The script uses `tool.gpt` to interact withl OpenAI LLM and starts by analyzing the image.  
    该脚本使用 `tool.gpt` 与 OpenAI LLM，并从分析图像开始。
- Once the ingredients are identified, the recipe generator takes those ingredients, generates a recipe, and saves it into a `recipe.md` file.  
    确定成分后，配方生成器会获取这些成分，生成配方，并将其保存到 `recipe.md` 文件中。

The whole process takes a few seconds to complete, and you can see the output in the terminal or the generated files.  
整个过程需要几秒钟才能完成，您可以在终端中看到输出或生成的文件。

## Python App  Python 应用程序

With the scripts running as expected, let us add a UI. We’ll be using Flask to build one, but you can use any other framework.  
脚本按预期运行后，我们添加一个 UI。我们将使用 Flask 来构建一个，但你可以使用任何其他框架。

The web app has a simple UI that allows the user to upload a file to the directory and then execute the `recipegenerator.gpt` script using [subprocess](https://docs.python.org/3/library/subprocess.html).  
Web 应用程序具有一个简单的 UI，允许用户将文件上传到目录，然后使用 [subprocess](https://docs.python.org/3/library/subprocess.html) 执行 `recipegenerator.gpt` 脚本。

## Backend Logic  后端逻辑

Below are the steps that were involved in building this:  
以下是构建此内容所涉及的步骤：

- After the environment setup, the first step was to integrate `recipegenerator.gpt` in Python. I created an `app.py` file and used Python’s [subprocess](https://docs.python.org/3/library/subprocess.html) capability to invoke the GPTScript. I defined the path to the GPTScript and used popen to invoke the script using `gptscript` prompt  
    环境设置完成后，第一步是在 Python 中集成 `recipegenerator.gpt`。我创建了一个 `app.py` 文件，并使用 Python 的[子进程](https://docs.python.org/3/library/subprocess.html)功能来调用 GPTScript。我定义了 GPTScript 的路径，并使用 popen 通过 `gptscript` 提示符调用脚本

```python
# Execute the script to generate the recipe
   subprocess.Popen(f"gptscript {SCRIPT_PATH}", shell=True, stdout=subprocess.PIPE, cwd=base_dir).stdout.read()
```

- Once I got the `recipegenerator.gpt` to work from my Python app, I added Flask to the project and made necessary changes with respect to routes. Added a `/` route that would execute when the app is launched, and another `/upload` route that will execute when the file is uploaded.  
    在我让 `recipegenerator.gpt` 从我的 Python 应用程序中工作后，我将 Flask 添加到项目中，并对路由进行了必要的更改。添加了一个 `/` 启动应用程序时执行的路由，以及另一个 `/upload` 上传文件时执行的路由。
- Finally, I added the logic to handle file uploads. This function takes the file uploaded by the user and saves it as `grocery.png,` which will be used by the recipegenerator.  
    最后，我添加了处理文件上传的逻辑。此函数获取用户上传的文件并将其保存为 `grocery.png，`供 recipegenerator 使用。

## Frontend UI  前端 UI

- After the backend was ready, it was time to build the front end. I used simple HTML, CSS, JS, and Bootstrap to build the front end.  
    后端准备就绪后，就可以构建前端了。我使用简单的 HTML、CSS、JS 和 Bootstrap 来构建前端。
- Since we are using Flask, all the UI-related files go to `templates` directory. So created an index. html file with a few elements, including a loader that shows up when the script is still executing, form elements to allow the user to upload a file  
    由于我们使用的是 Flask，因此所有与 UI 相关的文件都转到 `templates` 目录。所以创建了一个索引。HTML 文件，其中包含一些元素，包括在脚本仍在执行时显示的加载程序，以及用于允许用户上传文件的表单元素
- The page also has client-side logic written in JS to take the form input, send it to the backend, get the response, and display it on the screen.  
    该页面还具有用 JS 编写的客户端逻辑，用于获取表单输入、将其发送到后端、获取响应并将其显示在屏幕上。

_Pro Tip: You can even use ChatGPT to build the UI for this.  
专业提示：您甚至可以使用 ChatGPT 为此构建 UI。_

Once both these parts are ready, execute the application. If you’re using an IDE like VSCode, simply hit F5, or you can execute `flask run` or `python app.py`  
准备好这两个部分后，执行应用程序。如果您使用的是 VSCode 之类的 IDE，只需按 F5，或者您可以执行 `flask run` 或 `python app.py`

The application will be available at [http://127.0.0.1:5000/](http://127.0.0.1:5000/). Go ahead and upload a photo with your grocery/ingredients. The GPTScript runs in the background, analyzes the photo for ingredients, and shows the recipe.  
该应用程序将于 [http://127.0.0.1:5000/](http://127.0.0.1:5000/) 上提供。继续上传一张带有您的杂货/食材的照片。GPTScript 在后台运行，分析照片中的成分，并显示食谱。

## Closing Thoughts  结束语

GPTScript makes it super easy to use natural language to interact with OpenAI. This recipe generator is just one of the many things you can do with AI and GPTScript. There are many more examples that you can try or even contribute your existing use cases and show the world what you’re up to.  
GPTScript 使使用自然语言与 OpenAI 交互变得非常容易。这个食谱生成器只是您可以使用 AI 和 GPTScript 完成的众多事情之一。您可以尝试更多示例，甚至可以贡献您现有的用例，并向世界展示您的研究成果。

If you’re struggling to interact with LLMs, give [GPTScript](https://github.com/gptscript-ai/gptscript) a try. If you’re new here or need guidance, join [GPTScript discord server](https://discord.gg/9sSf4UyAMC) and get all your queries answered.  
如果您正在努力与 LLMs，请尝试使用 [GPTScript](https://github.com/gptscript-ai/gptscript)。如果您是新来的或需要指导，请加入 [GPTScript discord 服务器](https://discord.gg/9sSf4UyAMC)并回答您的所有疑问。