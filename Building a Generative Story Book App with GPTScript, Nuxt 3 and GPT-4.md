
使用 GPTScript、Nuxt 3 和 GPT-4 构建生成式故事书应用程序 – 第 1 部分

###### April 3, 2024 by tyler slaton  

In this post, we are going to explore the process of creating an interactive storybook that can take a prompt or a full story and transform it into an illustrated children’s book. We are going to achieve this using Nuxt 3 and GPTScript. The journey will be divided into two main parts:  
在这篇文章中，我们将探讨创建交互式故事书的过程，该故事书可以采用提示或完整故事并将其转化为插图儿童读物。我们将使用 Nuxt 3 和 GPTScript 来实现这一目标。旅程将分为两个主要部分：

1. Crafting a GPTScript  制作 GPTScript
2. Integrating GPTScript into a web application  
    将 GPTScript 集成到 Web 应用程序中

In part 2 of this post, we’ll be turning this code into a complete web application that can be hosted and used to view stories. The complete product will look like [https://story-book-main.onrender.com/](https://story-book-main.onrender.com/) and you can see the final source code [here](https://github.com/gptscript-ai/story-book).  
在本文的第 2 部分中，我们将把这些代码转换为一个完整的 Web 应用程序，可以托管并用于查看故事。完整的产品将类似于 [https://story-book-main.onrender.com/](https://story-book-main.onrender.com/)，您可以[在此处](https://github.com/gptscript-ai/story-book)查看最终源代码。

We have to start somewhere, so let’s get the basics done!  
我们必须从某个地方开始，所以让我们完成基础工作吧！

# Getting Started  开始

## GPTScript

GPTScript is an innovative natural language scripting language that empowers you to prompt Large Language Models (LLMs) using tools that you have defined or imported.  
GPTScript 是一种创新的自然语言脚本语言，使您能够使用已定义或导入的工具提示大型语言模型 （LLMs）。

### Setup  设置

Setting up GPTScript is quite simple. You can find a detailed guide [here](https://docs.gptscript.ai/getting-started) that will walk you through the steps. To give you a brief overview, all you need to do is install the CLI, set up any necessary API keys, and you’re good to go.  
设置 GPTScript 非常简单。您可以[在此处](https://docs.gptscript.ai/getting-started)找到详细指南，它将引导您完成这些步骤。为了给您一个简短的概述，您需要做的就是安装 CLI，设置任何必要的 API 密钥，然后就可以开始了。

## Introduction to Nuxt 3  Nuxt 3 简介

Nuxt 3 is a comprehensive application framework that leverages the power of Vue to construct applications.  
Nuxt 3 是一个全面的应用程序框架，它利用 Vue 的强大功能来构建应用程序。

### Setup  设置

You can learn more about Nuxt 3 [here](https://nuxt.com/docs/getting-started/introduction). However, for the purpose of this tutorial, all you need to ensure is that you have Node installed, with a minimum version of v20.11.1.  
您可以[在此处](https://nuxt.com/docs/getting-started/introduction)了解有关 Nuxt 3 的更多信息。但是，对于本教程，您只需确保已安装 Node，最低版本为 v20.11.1。

# Building the Story Book  构建故事书

## Setting up the repo  设置存储库

Let’s start by setting up a Nuxt application. There are many options available during this process, and for this tutorial, most of them will work. Choose what fits you best, but we’ll be using **npm** for packaging. If you prefer to follow these commands closely, select **npm**.  
让我们从设置 Nuxt 应用程序开始。在此过程中有许多选项可用，对于本教程，其中大多数都可以使用。选择最适合您的选项，但我们将使用 **npm** 进行打包。如果您希望严格遵循这些命令，请选择 **npm**。

```bash
npx nuxi@latest init
```

After generating our new repository, the next step is to install two crucial modules: **gptscript** and **@nuxt/ui**  
生成新的存储库后，下一步是安装两个关键模块：**gptscript** 和 **@nuxt/ui**  
The **gptscript** module allows us to easily execute GPTScripts, while the **@nuxt/ui** module provides user-friendly front-end components.  
**gptscript** 模块让我们可以轻松执行 GPTScripts，而 **@nuxt/ui** 模块则提供了用户友好的前端组件。

```bash
 npm install @gptscript-ai/gptscript @nuxt/ui
```

Now, we need to include our Nuxt modules into **nuxt.config.ts**  
现在，我们需要将 Nuxt 模块包含在 **nuxt.config.ts**

Your file should look like the one below:  
您的文件应如下所示：

```tsx
// <https://nuxt.com/docs/api/configuration/nuxt-config>
export default defineNuxtConfig({
  devtools: { enabled: true },
  modules: [
    '@nuxt/ui',
    '@nuxtjs/tailwindcss',
  ]
})
```

With our essential dependencies installed, we can now create some important files. It’s worth noting that larger LLM workloads, including GPTScript, can take some time to complete. Therefore, as we design our API, it’s essential to provide the user with as much feedback as possible about the process status. We’ll achieve this using SSE ([server-side events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events)) and the **gptscript** node module. With this in mind, let’s create our API routes:  
安装基本依赖项后，我们现在可以创建一些重要文件。值得注意的是，较大的LLM工作负载（包括 GPTScript）可能需要一些时间才能完成。因此，在我们设计 API 时，必须为用户提供尽可能多的有关进程状态的反馈。我们将使用 SSE（[服务器端事件](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events)）和 **gptscript** 节点模块来实现这一点。考虑到这一点，让我们创建我们的 API 路由：

```bash
mkdir -p server/api/story
mkdir pages
mkdir components
touch server/api/story/index.post.ts
touch server/api/story/sse.get.ts
touch pages/index.vue
```

In Nuxt, routing is file-based. In this case, we’ve created two API routes and a page to be served:  
在 Nuxt 中，路由是基于文件的。在本例中，我们创建了两个 API 路由和一个要服务的页面：

- **server/api/story/index.post.ts**: POST – [http://localhost.com:3000/api/story](http://localhost.com:3000/api/story)  
    **server/api/story/index.post.ts**：POST – [http://localhost.com:3000/api/story](http://localhost.com:3000/api/story)
- **server/api/story/sse.get.ts**: GET – [http://localhost:3000/api/story/sse](http://localhost:3000/api/story/sse)  
    **server/api/story/sse.get.ts**：GET – [http://localhost:3000/api/story/sse](http://localhost:3000/api/story/sse)
- **pages/index.vue**: GET – [http://localhost:3000](http://localhost:3000/)  
    **pages/index.vue**： GET – [http://localhost:3000](http://localhost:3000/)

The POST request allows us to create a story, while the GET request enables us to receive status updates. Essentially, we’ll make one request to execute a GPTScript and another to monitor it. Finally, our page will allow us to write some Vue to display the execution state.  
POST 请求允许我们创建一个故事，而 GET 请求使我们能够接收状态更新。本质上，我们将发出一个请求来执行 GPTScript，另一个请求来监控它。最后，我们的页面将允许我们编写一些 Vue 来显示执行状态。

## Implementing the POST route  
实现 POST 路由

Acorn Labs has provided a handy Node module that handles the majority of the work in running our script. Although we’ll write our GPTScript in a different section, let’s assume for now that the file will be named **story-book.gpt**  
Acorn Labs 提供了一个方便的 Node 模块，可以处理运行脚本的大部分工作。虽然我们将在不同的部分编写 GPTScript，但现在让我们假设该文件将被命名为 story-book.gpt  
Now, let’s get ready to delve into the code!  
现在，让我们准备好深入研究代码吧！

```tsx
import gptscript from '@gptscript-ai/gptscript'
import { Readable } from 'stream'

// We need to define the request body that we'll receive from the user.
// In this case, we want a prompt and the number of pages.
type Request = {
    prompt: string;
    pages: number;
}

// Here's a type to store executions of running GPTScripts.
export type RunningScript = {
    stdout: Readable;
    stderr: Readable;
    promise: Promise<void>;
}

// We're not using a database in this tutorial, so we'll simply store them in memory.
// It's all for the sake of simplicity and demonstration.
export const runningScripts: Record<string, RunningScript>= {}

// Now, we define the Nuxt handler. readBody and setResponseStatus are imported implicitly by Nuxt.
export default defineEventHandler(async (event) => {
    const request = await readBody(event) as Request

		// We need both prompt and pages to be defined in the body
    if (!request.prompt) {
        throw createError({
            statusCode: 400,
            statusMessage: 'prompt is required'
        });
    }
    if (!request.pages) {
        throw createError({
            statusCode: 400,
            statusMessage: 'pages are required'
        });
    }

		// We call the GPTScript node module to run our (yet to be written) story-book.gpt file.
    const {stdout, stderr, promise} = await gptscript.streamExecFile(
	    'story-book.gpt',
	    `--story ${request.prompt} --pages ${request.pages}`,
	    {})

		// Since we're not done processing, we respond with a 202 to inform
		// the client that we've received the request and are processing it.
    setResponseStatus(event, 202)

		// We add the running script to our in-memory store.
    runningScripts[request.prompt] = {
        stdout: stdout,
        stderr: stderr,
        promise: promise
    }
})

```

Here’s a breakdown of what’s happening in the code:  
以下是代码中发生的情况的细分：

1. We define certain types to hold the request’s body and the response from the GPTScript module.  
    我们定义了某些类型来保存请求正文和来自 GPTScript 模块的响应。
2. A Nuxt handler for this route is established.  
    为此路由建立了 Nuxt 处理程序。
3. The body is read as a Request type.  
    正文将读取为 Request 类型。
4. We ensure the user has provided a prompt and has specified the number of pages for the story.  
    我们确保用户已提供提示并指定了故事的页数。
5. The GPTScript’s node module is utilized to execute the script.  
    GPTScript 的 node 模块用于执行脚本。
6. Finally, we keep the running script in our in-memory store.  
    最后，我们将正在运行的脚本保存在内存中的存储中。

## Implementing the SSE route  
实施 SSE 路由

After successfully creating a route to initiate a GPTScript, the next step is to develop a pathway for tracking its progress. This is where Server Side Events (SSE) become useful. SSE allows us to stream updates from the GPTScript to the front-end in real-time. To ensure this works properly, we have to set specific headers and write updates to the newly opened stream as they arrive from the GPTScript. Now, let’s explore the code to see how we can accomplish this.  
成功创建启动 GPTScript 的路由后，下一步是开发跟踪其进度的路径。这就是服务器端事件 （SSE） 派上用场的地方。SSE 允许我们将更新从 GPTScript 实时流式传输到前端。为了确保它正常工作，我们必须设置特定的标头，并在新打开的流从 GPTScript 到达时将更新写入它们。现在，让我们探索代码，看看我们如何做到这一点。

```tsx
// Import the in-memory store for running scripts
import { runningScripts } from '@/server/api/story/index.post';

// Define the Nuxt handler. getQuery, setResponseStatus, and setHeaders are 
// imported implicitly by Nuxt.
export default defineEventHandler(async (event) => {
		// The prompt is the key to retrieve the value from the runningScripts
		// store.
    const { prompt } = getQuery(event);
    if (!prompt) {
        throw createError({
            statusCode: 400,
            statusMessage: 'prompt is required'
        });
    }
		
		// Retrieve the running script from the store
    const runningScript = runningScripts[prompt as string];
    if (!runningScript) {
        throw createError({
            statusCode: 404,
            statusMessage: 'running script not found'
        });
    }

		// Since we found the script, we can go ahead and set the status to
		// 200. Along with this we need to set specific headers that indicate
		// that the connection should be kept alive and is an event stream.
    setResponseStatus(event, 200);
    setHeaders(event, {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Credentials': 'true',
        'Connection': 'keep-alive',
        'Content-Type': 'text/event-stream',
    });
    
    // Setup an event handler for new data being submitted to the stdoutBuffer.
    let stdoutBuffer = '';
    runningScript.stdout.on('data', (data) => {
        stdoutBuffer += data;
        // This ensures that we only write responses when they're terminated
        // with a newline.
        if (data.includes('n')) {
        		// Write the complete line to the event stream.
            event.node.res.write(`data: ${stdoutBuffer}nn`);
            stdoutBuffer = '';
        }
    });

		// Setup an event handler for new data being submitted to the stderrBuffer.
    let stderrBuffer = '';
    runningScript.stderr.on('data', (data) => {
        stderrBuffer += data;
        // This ensures that we only write responses when they're terminated
        // with a newline.
        if (data.includes('n')) {
		        // Write the complete line to the event stream.
            event.node.res.write(`data: ${stderrBuffer}nn`);
            stderrBuffer = '';
        }
    });

		// This ensures that Nuxt won't automatically handle the request.
    event._handled = true;
    
    // Wait for the promise to complete and whether its a success or 
    // error terminate the stream with a final message.
    await runningScript.promise.then(() => {
        event.node.res.write('data: donenn');
        event.node.res.end();
    }).catch((error) => {
        setResponseStatus(event, 500);
        event.node.res.write(`data: error: ${error}nn`);
        event.node.res.end();
    });
});
```

Alright, let’s unpack this a bit more!  
好了，让我们再解释一下！

1. First, we import the in-memory store for running scripts. This is crucial as it helps us retrieve updates for any running script.  
    首先，我们导入用于运行脚本的内存存储。这很关键，因为它可以帮助我们检索任何正在运行的脚本的更新。
2. Next, we define the Nuxt handler. This is where the magic happens!  
    接下来，我们定义 Nuxt 处理程序。这就是魔法发生的地方！
3. We then extract the **prompt** from the query parameters. Of course, if you’re dealing with larger values, you might want to consider replacing this with a body. However, for the purpose of this demonstration, we’ve chosen to use a query parameter.  
    然后，我们从查询参数中提取**提示**。当然，如果您正在处理更大的值，则可能需要考虑将其替换为正文。但是，出于本演示的目的，我们选择使用查询参数。
4. Afterward, we verify that the **prompt** has a corresponding key in the runningScripts store. If it doesn’t, we need to return an error to inform the user.  
    之后，我们验证 **prompt** 在 runningScripts 存储中是否有相应的 key。如果没有，我们需要返回一个错误来通知用户。
5. Once we’ve confirmed the existence of the script, we set the response code and the necessary headers to implement Server-Sent Events (SSE). This is the part that allows us to stream updates from the GPTScript right to the front-end in real-time!  
    确认脚本存在后，我们设置响应代码和必要的标头以实现服务器发送事件 （SSE）。这是允许我们将更新从 GPTScript 实时流式传输到前端的部分！
6. Now comes the interesting part. We write out event handlers for both **stdout** and **stderr**. The responses are buffered, ensuring that only complete lines are sent to the front-end. These events are then written to the event stream.  
    现在是有趣的部分。我们为 **stdout** 和 **stderr** 写出事件处理程序。响应被缓冲，确保只有完整的行被发送到前端。然后，这些事件被写入事件流。
7. We then instruct Nuxt not to automatically resolve the request. This gives us more control over the process and allows us to handle any exceptions or errors that might occur.  
    然后，我们指示 Nuxt 不要自动解决请求。这让我们对流程有更多的控制权，并允许我们处理可能发生的任何异常或错误。
8. Lastly, once the promise resolves, we determine how to terminate and respond based on whether the operation was successful or not.  
    最后，一旦 Promise 解析，我们就会根据作是否成功来确定如何终止和响应。

That’s it! With these steps, you’ve successfully set up the API and can begin work on writing the GPTScript to perform the generation.  
就是这样！ 通过这些步骤，您已成功设置 API，可以开始编写 GPTScript 来执行生成。

## Writing the GPTScript  编写 GPTScript

Next, we will write a GPTScript for our API to execute.  
接下来，我们将编写一个 GPTScript 供我们的 API 执行。

```yaml
tools: story-writer, story-illustrator, mkdir, sys.write, sys.read, sys.download
description: Writes a children's book and generates illustrations for it.
args: story: The story to write and illustrate. Can be a prompt or a complete story.
args: pages: The number of pages to generate

You are a story-writer. Do the following steps sequentially.

1. Come up with an appropriate title for the story based on the ${prompt}
2. Create the `public/stories/${story-title}` directory if it does not already exist.
3. If ${story} is a prompt and not a complete children's story, call story-writer
   to write a story based on the prompt.
4. Take ${story} and break it up into ${pages} logical "pages" of text.
5. For each page:
   - For the content of the page, write it to `public/stories/${story-title}/page<page-number>.txt and add appropriate newline
     characters.
   - Call story-illustrator to illustrate it. Be sure to include any relevant characters to include when
     asking it to illustrate the page.
   - Download the illustration to a file at `public/stories/${story-title}/page<page_number>.png`.

---
name: story-writer
description: Writes a story for children
args: prompt: The prompt to use for the story
temperature: 1

Write a story with a tone for children based on ${prompt}.

---
name: story-illustrator
tools: github.com/gptscript-ai/image-generation
description: Generates a illustration for a children's story
args: text: The text of the page to illustrate

Think of a good prompt to generate an image to represent $text. Make sure to
include the name of any relevant characters in your prompt. Then use that prompt to
generate an illustration. Append any prompt that you have with ". In an pointilism cartoon
children's book style with no text in it". Only return the URL of the illustration.

---
name: mkdir
tools: sys.write
description: Creates a specified directory
args: dir: Path to the directory to be created. Will create parent directories.

#!bash

mkdir -p "${dir}"

```

This document contains a lot of prompt engineering and GPTScript specific terminology. Let’s clarify each term:  
本文档包含许多提示工程和 GPTScript 特定术语。让我们澄清每个术语：

1. **tools** – This tells GPTScript what tools are available when executing the script. In this case, we’re providing access to several tools we’ve created: **story-writer**, **story-illustrator**, and **mkdir**, as well as some built-in system tools: **sys.write**, **sys.read**, **sys.download**.  
    **tools** – 这告诉 GPTScript 在执行脚本时可以使用哪些工具。在这种情况下，我们提供对我们创建的几个工具的访问：**story-writer**、**story-illustrator** 和 **mkdir**，以及一些内置的系统工具：**sys.write**、**sys.read**、**sys.download**。
2. **description** – This provides the Language Model (LLM) with information about the tool’s function and usage.  
    **description** – 这为语言模型 （）LLM 提供有关工具功能和用法的信息。
3. **args** – These are the values that the user supplies when running the script. Recall from our API that we pass **pages** and **prompt** here.  
    **args** – 这些是用户在运行脚本时提供的值。回想一下，在我们的 API 中，我们在此处传递**页面**并**提示**。
4. Everything else until the next **– – –** (which marks the start of a new tool) comprises the script’s prompt. In this case, we’re giving the LLM details about its role and the process of generating a story. When run, the LLM will choose what tools to employ and how to use them.  
    直到下一个 **– – – （**标志着新工具的开始）之前的所有内容都包含脚本的提示符。在这种情况下，我们将提供有关其角色和生成故事的过程LLM的详细信息。运行时，LLM将选择要使用的工具以及如何使用它们。
5. **– – –** – This marks the beginning of a new in-file tool. In this context, we’ve defined three tools (**story-writer**, **story-illustrator**, and **mkdir**).  
    **– – – –** 这标志着新的文件内工具的开始。在此上下文中，我们定义了三个工具（**story-writer**、**story-illustrator** 和 **mkdir**）。
6. **#!bash** – This instructs the LLM to execute the following logic specifically using bash.  
    **#！bash** – 这指示 专门LLM使用 bash 执行以下逻辑。

With our GPTScript finished, we’re reading to start building out the front-end!  
完成 GPTScript 后，我们正在阅读以开始构建前端！

## Streaming Events to the Front-end  
将事件流式传输到前端

With our API ready, we can start building the front-end.  
准备好 API 后，我们可以开始构建前端。

### Understanding our UI stack (optional)  
了解我们的 UI 堆栈（可选）

In the following sections, we will cover the user interface (UI) aspect of this web application. For further understanding, you can explore the three main technologies involved through the links below. However, we will only provide a brief overview of each for demonstration purposes.  
在以下部分中，我们将介绍此 Web 应用程序的用户界面 （UI） 方面。为了进一步了解，您可以通过下面的链接探索所涉及的三种主要技术。但是，我们仅出于演示目的提供每种技术的简要概述。

- **Vue** – Nuxt uses this framework for component-based rendering. Read more about it here if you’re interested.  
    **Vue** – Nuxt 使用此框架进行基于组件的渲染。如果您有兴趣，请在此处阅读更多相关信息。
- **TailwindCSS** – This CSS library allows for styling through predefined classes it provides. The majority of the styling in this tutorial is done through TailwindCSS.  
    **TailwindCSS** – 此 CSS 库允许通过它提供的预定义类进行样式设置。本教程中的大部分样式都是通过 TailwindCSS 完成的。
- **@nuxt/ui** – This plug-and-play component library for Nuxt provides many of the core components used in this tutorial. It’s a great tool for quickly designing impressive UI’s. Any component in this tutorial that starts with a **U** (like **UCard** or **UButton**) is from this library.  
    **@nuxt/ui** – 这个适用于 Nuxt 的即插即用组件库提供了本教程中使用的许多核心组件。它是快速设计令人印象深刻的 UI 的绝佳工具。本教程中以 **U** 开头的任何组件（如 **UCard** 或 **UButton**）都来自此库。

Please note that we will not be writing the code to display your rendered stories on the front-end. The implementation of this is relatively straightforward. Since the purpose of this tutorial is to learn how to interact with GPTScript at an API level, you can learn more about displaying rendered stories by visiting the [story-book example in the GPTScript repo](https://github.com/gptscript-ai/gptscript/tree/main/examples/story-book).  
请注意，我们不会编写代码来在前端显示您渲染的故事。实现起来相对简单。由于本教程的目的是学习如何在 API 级别与 GPTScript 交互，因此您可以通过访问 [GPTScript 存储库中的 story-book 示例](https://github.com/gptscript-ai/gptscript/tree/main/examples/story-book)来了解有关显示渲染故事的更多信息。

### Getting our app.vue in order  
整理我们的 app.vue

First, let’s start by setting up our **app.vue** to render pages and be styled appropriately.  
首先，让我们从设置 **app.vue** 开始渲染页面并设置适当的样式。

```html
<!-- app.vue -->
<script lang="ts" setup>
// Sets up site header data.
useHead({
  title: 'Story Book',
  meta:  [
    { charset: 'utf-8' },
    { name: 'apple-mobile-web-app-capable', content: 'yes' },
    { name: 'format-detection', content: 'telephone=no' },
    { name: 'viewport', content: `width=device-width` },
  ],
})
</script>

<template>
  <div>
    <div class="grid grid-rows-[1fc,1fr] fixed top-0 right-0 bottom-0 left-0">
      <div class="overflow-auto border-t-2 border-transparent">
        <div class="p-5 lg:p-10 max-w-full w-full h-full mx-auto">
	        <!-- Tells Nuxt to render the pages directory -->
          <NuxtPage class="pb-10" />
        </div>
      </div>
    </div>
  </div>
</template>
```

The crucial part of this code is **<NuxtPage/>**. Adding this component tells Nuxt to render **.vue** files in the **pages** directory, using the same file-based routing format as the **server** directory.  
这段代码的关键部分是 **<NuxtPage/>**。添加这个组件会告诉 Nuxt 在 **pages** 目录中渲染 **.vue** 文件，使用与**服务器**目录相同的基于文件的路由格式。

### Writing the index.vue  编写 index.vue

Next, let’s update our **index.vue** to include the necessary code for creating a story and rendering updates from GPTScript on the front-end.  
接下来，让我们更新 **index.vue** 以包含创建故事和在前端渲染来自 GPTScript 的更新所需的代码。

```html
<script setup lang="ts">
// Define the variables we will need for the script
const MAX_MESSAGES = 20;
const pendingStory = ref<EventSource>();
const pendingStoryMessages = ref<Array<string>>([]);
const pageOptions = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
const state = reactive({
    prompt: '',
    pages: 0,
})

// addMessage ensures that the number of messages in the pendingStoryMessages
// never exceeds the MAX_MESSAGSE
const addMessage = (message: string) => {
    pendingStoryMessages.value.push(message)
    if (pendingStoryMessages.value.length >= MAX_MESSAGES) {
        pendingStoryMessages.value.shift();
    }
}

// Create a function for the form's submit button to trigger
async function onSubmit () {
		// Using Nuxt's fetch, submit a request to generate a new story
		// to the backend by calling JSON.stringify on the form's state.
    const response = await fetch('/api/story', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(state)
    })

		// When a new request is sent, clear the buffer.
    pendingStoryMessages.value = []
    
    if (response.ok) {
        // On success, create a new EventSource for the specified prompt.
        // When a message is recieved, push it into the stream. Look for the
        // termination character we specified earlier with "done" or "error".
        const es = new EventSource(`/api/story/sse?prompt=${state.prompt}`)
        es.onmessage = async (event) => {
            addMessage(event.data)
            if ((event.data as string) === 'done') {
                es.close()
                pendingStory.value = undefined
            } else if ((event.data as string).includes('error')) {
                es.close()
                pendingStory.value = undefined
                addMessage(`An error occurred while generating the story: ${event.data}`)
            }
        }
        pendingStory.value = es
    } else {
		    // If an error occured, add it to the message buffer.
        addMessage(`An error occurred while generating the story: ${response.statusText}`)
    }
}
</script>

<template>
    <div class="flex flex-col lg:flex-row space-y-10 lg:space-x-10 lg:space-y-0  h-full">
        <UCard class="h-full w-full lg:w-1/2">
            <template #header>
                <h1 class="text-2xl">Create a new story</h1>
            </template>
            <UForm class="h-full" :state=state @submit="onSubmit">
                <UFormGroup size="lg" class="mb-6" label="Pages" name="pages">
                    <USelectMenu class="w-1/4 md:w-1/6"v-model="state.pages" :options="pageOptions"/>
                </UFormGroup>
                <UFormGroup class="my-6" label="Story" name="prompt">
                    <UTextarea :ui="{base: 'h-[50vh]'}" size="xl" class="" v-model="state.prompt" label="Prompt" placeholder="Put your full story here or prompt for a new one"/>
                </UFormGroup>
                <UButton :disable="!pendingStory" size="xl" icon="i-heroicons-book-open" type="submit">Create Story</UButton>
            </UForm>
        </UCard>
        <UCard class="h-full w-full lg:w-1/2">
            <template #header>
                <h1 class="text-2xl">GPTScript Story Generation Progress</h1>
            </template>
            <div class="h-full">
                <h2 v-if="pendingStory" class="text-zinc-400 mb-4">GPTScript is currently building the story you requested. You can see its progress below.</h2>
                <h2 v-else-if="!pendingStory && pendingStoryMessages" class="text-zinc-400 mb-4">Your story finished! Submit a new prompt to generate another.</h2>
                <h2 v-else class="text-zinc-400 mb-4">You currently have no stories processing. Submit a prompt for a new story to get started!</h2>
                <div class="h-full">
                    <pre class="h-full bg-zinc-950 px-6 text-white overflow-x-scroll rounded shadow">
                        <p v-for="message in pendingStoryMessages">> {{ message }}</p>
                    </pre>
                </div>
            </div>
        </UCard>
    </div>
</template>

```

1. First, we initialize some necessary variables.  
    首先，我们初始化一些必要的变量。
    1. **MAX_MESSAGES** – This variable sets a limit for the maximum number of messages that can be displayed on the front-end.  
        **MAX_MESSAGES** – 此变量设置可在前端显示的最大消息数的限制。
    2. **pendingStory** – This is a reactive reference to an EventSource that represents the current story being generated.  
        **pendingStory** – 这是对 EventSource 的反应式引用，表示正在生成的当前故事。
    3. **pendingStoryMessages** – This is a reactive reference to an array that will hold the messages streamed from the server.  
        **pendingStoryMessages** – 这是对数组的反应式引用，该数组将保存从服务器流式传输的消息。
    4. **state** – This is a reactive state object that holds the user’s input for the story prompt and the number of pages.  
        **state** – 这是一个反应式状态对象，用于保存用户对故事提示的输入和页数。
2. Next, we create a function called **addMessages** that adds a new message to the **pendingStoryMessages** array and ensures that the array’s length never exceeds **MAX_MESSAGES**.  
    接下来，我们创建一个名为 **addMessages** 的函数，该函数将新消息添加到 **pendingStoryMessages** 数组中，并确保数组的长度永远不会超过 **MAX_MESSAGES**。
3. Then we create **onSubmit** which is triggered when the form is submitted. This function sends a POST request to the server to create a new story. It also handles the server’s response, initiates the EventSource for tracking the story’s progress, and handles any errors that might occur during the process.  
    然后，我们创建 **onSubmit**，它在提交表单时触发。此函数向服务器发送 POST 请求以创建新故事。它还处理服务器的响应，启动 EventSource 以跟踪故事的进度，并处理在此过程中可能发生的任何错误。
4. Finally, we write out the **Vue** template that will actually render all of the processed data from the previous steps. This includes a form for user input and a display area for the messages streamed from the server.  
    最后，我们写出 **Vue** 模板，它实际上将呈现前面步骤中所有已处理的数据。这包括一个用于用户输入的表单和一个用于从服务器流式传输的消息的显示区域。

## Testing it out  测试

After preparing everything, we can begin testing. First, set the **OPENAI_API_KEY** and then start the local Nuxt development server.  
准备好一切后，我们就可以开始测试了。首先，设置 **OPENAI_API_KEY**，然后启动本地 Nuxt 开发服务器。

```bash
export OPENAI_API_KEY=<your key here>
npm run dev

```

Pro tip: If you want to set your **OPENAI_API_KEY** securely, you can use a **.env** file and source.  
专业提示：如果要安全地设置**OPENAI_API_KEY**，可以使用 **.env** 文件和源。

```bash
echo "OPENAI_API_KEY=" > .env
open .env # add your key in this file
source .env

```

The application will start at **localhost:3000**. Opening this address in your browser will display the following interface.  
应用程序将从 **localhost：3000** 启动。在浏览器中打开此地址将显示以下界面。

![blank-app](https://www.acorn.io/wp-content/uploads/2024/11/1_0ec86096e5.png)

When we enter a story and submit, we can see the updates stream on the right-hand side.  
当我们输入故事并提交时，我们可以在右侧看到更新流。

![filled app](https://www.acorn.io/wp-content/uploads/2024/11/2_388f2c0e25.png)

Once completed, the UI will indicate that the process is finished. You can then view the generated files in the **public/stories** directory where the LLM has written everything.  
完成后，UI 将指示该过程已完成。然后，您可以在 **public/stories** 目录中查看生成的文件，其中LLM已写入所有内容。

With that, we’re done with this part.  
至此，我们完成了这部分。

# Wrapping up  结束语

Congratulations! If you’ve made it this far, you now have an understanding of how to integrate and use GPTScript effectively within your own set of APIs. This guide has delved into the use of the GPTScript node module, a powerful tool for programmatically streaming updates to a front-end. Next time we’ll be diving into tidying up our code, rendering the story, and making this a fully deployable web application.  
祝贺！ 如果您已经走到了这一步，那么您现在已经了解了如何在您自己的 API 集中有效地集成和使用 GPTScript。本指南深入探讨了 GPTScript 节点模块的使用，这是一个强大的工具，用于以编程方式将更新流式传输到前端。下次，我们将深入整理代码、渲染故事，并使其成为完全可部署的 Web 应用程序。

If you’re interested in further expanding your knowledge, connecting with others using GPTScript, or learning about the development of GPTScript, we strongly recommend visiting our [GitHub repository](https://github.com/gptscript-ai/gptscript).  
如果您有兴趣进一步扩展您的知识、使用 GPTScript 与他人联系或了解 GPTScript 的开发，我们强烈建议您访问我们的 [GitHub 存储库](https://github.com/gptscript-ai/gptscript)。  
If you’d like to chat about building your own projects using GPTScript, [schedule a conversation.](https://www.acorn.io/contact)  
如果您想聊聊如何使用 GPTScript 构建自己的项目，[请安排对话。](https://www.acorn.io/contact)






使用 GPTScript、Nuxt 3 和 GPT-4 构建生成式故事书应用程序 – 第 2 部分
###### May 2, 2024 by tyler slaton  
五月 2， 2024by Tyler Slaton

## Introduction  介绍

Welcome back to the second part of our Story Book tutorial! In this part, we will dive deeper into how to expand the features of our story book generator and discuss a bit more of what you can do with [GPTScript](https://www.acorn.io/resources/tutorials/building-a-generative-story-book-app-using-gpt-script-nuxt-3-and-gpt-4-part-2/gptscript.ai). We’ll also make it a fully deployable application. You can read the first part [here](http://acorn.io/resources/tutorials/building-a-generative-story-book-app-with-gpt-script-nuxt-3-and-gpt-4-part-1).  
欢迎回到我们故事书教程的第二部分！在这一部分中，我们将深入探讨如何扩展故事书生成器的功能，并讨论您可以使用 [GPTScript](https://www.acorn.io/resources/tutorials/building-a-generative-story-book-app-using-gpt-script-nuxt-3-and-gpt-4-part-2/gptscript.ai) 做更多的事情。我们还将使其成为一个完全可部署的应用程序。你可以[在这里](http://acorn.io/resources/tutorials/building-a-generative-story-book-app-with-gpt-script-nuxt-3-and-gpt-4-part-1)阅读第一部分。

You can find the open-source repo for the code at [https://github.com/gptscript-ai/story-book](https://github.com/gptscript-ai/story-book) and the hosted version of it at [https://story-book.gptscript-demos.ai/](https://story-book.gptscript-demos.ai/).  
您可以在 [https://github.com/gptscript-ai/story-book](https://github.com/gptscript-ai/story-book) 中找到代码的开源存储库，在 [https://story-book.gptscript-demos.ai/](https://story-book.gptscript-demos.ai/) 中找到代码的托管版本。

## Prerequisites  先决条件

The same prerequisites from last time apply here, but I also highly recommend reading through the first part in this series before proceeding. We’ll be building on that application directly so that context will be very important as we move forward.  
上次的先决条件同样适用于此处，但我也强烈建议在继续之前通读本系列的第一部分。我们将直接基于该应用程序进行构建，因此在我们前进的过程中，上下文将非常重要。

## Quick recap  快速回顾

Last time we got an application that:  
上次我们收到的应用程序是：

- Use GPTScript to generate a story.  
    使用 GPTScript 生成故事。
- Streams updates from GPTScript from the backend to the front-end using SSE.  
    使用 SSE 将 GPTScript 的更新从后端流式传输到前端。
- Learned some basics about how to use Nuxt as a full stack framework.  
    学习了一些有关如何将 Nuxt 用作全栈框架的基础知识。

We’re missing a few things from this and have various improvements to make. So, to enumerate those, by the end of this blog post we’ll:  
我们从中遗漏了一些东西，并且需要进行各种改进。因此，为了列举这些内容，在这篇博文的结尾，我们将：

- Improve the SSE implementation to stream updates in a more reliable and scalable way.  
    改进 SSE 实施，以更可靠和可扩展的方式流式传输更新。
- Go through the motions of some prompt engineering to better prompt the AI for stories.  
    通过一些提示工程的动作来更好地提示 AI 的故事。
- Fully package the application as a container image that can be deployed anywhere that prefer using Github Actions.  
    将应用程序完全打包为容器映像，该映像可以部署在喜欢使用 Github Actions 的任何位置。
- Some UI improvements to pad out the user experience for our application.  
    一些 UI 改进，以填充应用程序的用户体验。

We’ve got a lot to cover so let’s jump right into it!  
我们有很多内容要介绍，所以让我们直接开始吧！

## Overhauling our approach to storing stories  
彻底改革我们存储故事的方法

Last time, we stored our stories in the **public** directory. This was nice as it, when run locally, we were able to serve those files using Nuxt. However, when we take this application into an actual environment the **public** directory does not dynamically update. With the current state of our application, this means that the stories that the AI generates will not be served properly. Let’s get to fixing this!  
上次，我们将故事存储在 **public** 目录中。这很好，当在本地运行时，我们能够使用 Nuxt 提供这些文件。但是，当我们将此应用程序带入实际环境时，**public** 目录不会动态更新。根据我们应用程序的当前状态，这意味着 AI 生成的故事将无法得到正确提供。让我们来解决这个问题！

First, let’s create an environment variable that we can set that will change the path that stories are stored. In the **nuxt.config.ts** file, let’s update it to look like the following:  
首先，让我们创建一个环境变量，我们可以设置该变量来更改故事的存储路径。在 **nuxt.config.ts** 文件中，让我们将其更新为如下所示：

```tsx
// https://nuxt.com/docs/api/configuration/nuxt-config
export default defineNuxtConfig({
  devtools: { enabled: true },
  modules: [
    '@nuxt/ui',
    '@nuxtjs/tailwindcss',
  ],
  runtimeConfig: {
    storiesVolumePath: 'stories', // NUXT_STORIES_VOLUME_PATH
  }
})
```

Now this environment variable will be loaded into the application wherever the application is deployed. We need to tell the AI that it should store the files here so we’ll need to do two things:  
现在，这个环境变量将被加载到应用程序中，无论应用程序部署在何处。我们需要告诉 AI 它应该将文件存储在这里，因此我们需要做两件事：

1. Update the GPTScript to accept and act on this **NUXT_STORIES_VOLUME_PATH** as an input  
    更新 GPTScript 以接受此输入并处理此**NUXT_STORIES_VOLUME_PATH**
2. Update the call execution to pass along **NUXT_STORIES_VOLUME_PATH**  
    更新调用执行以传递 **NUXT_STORIES_VOLUME_PATH**

Let’s start by updating our GPTScript’s base tool:  
让我们从更新 GPTScript 的基本工具开始：

```yaml
tools: story-writer, story-illustrator, mkdir, sys.write, sys.read, sys.download
description: Writes a children's book and generates illustrations for it.
args: story: The story to write and illustrate. Can be a prompt or a complete story.
args: pages: The number of pages to generate
args: path: The path that the story should be written to

You are a story-writer. Do the following steps sequentially:

1. Come up with an appropriate title for the story based on the ${prompt}
2. Create the `${path}/${story-title}` directory if it does not already exist.
3. If ${story} is a prompt and not a complete children's story, call story-writer
   to write a story based on the prompt.
4. Take ${story} and break it up into ${pages} logical "pages" of text.
5. For each page:
   - For the content of the page, write it to `${path}/${story-title}/page<page-number>.txt and add appropriate newline
     characters.
   - Call story-illustrator to illustrate it. Be sure to include any relevant characters to include when
     asking it to illustrate the page.
   - Download the illustration to a file at `${path}/${story-title}/page<page_number>.png`.
```

With this change, you’ll see we added the **path** arg and utilize it in steps 2 and 5 appropriately. Finally let’s update the GPTScript execution to pass this. In **server/api/story/index.post.ts**, let’s update the call to GPTScript to pass the environment variable.  
通过此更改，您将看到我们添加了**路径** arg 并在步骤 2 和 5 中适当地使用它。最后，让我们更新 GPTScript 执行以传递此命令。在 **server/api/story/index.post.ts** 中，让我们更新对 GPTScript 的调用以传递环境变量。

```tsx
import gptscript from '@gptscript-ai/gptscript'
import { Readable } from 'stream'

type Request = {
    prompt: string;
    pages: number;
}

export type RunningScript = {
    stdout: Readable;
    stderr: Readable;
    promise: Promise<void>;
}

export const runningScripts: Record<string, RunningScript>= {}

export default defineEventHandler(async (event) => {
    const request = await readBody(event) as Request

    if (!request.prompt) {
        throw createError({
            statusCode: 400,
            statusMessage: 'prompt is required'
        });
    }

    if (!request.pages) {
        throw createError({
            statusCode: 400,
            statusMessage: 'pages are required'
        });
    }

		// Grab the value of the environment variable and pass it to the GPTScript.
    const {storiesVolumePath } = useRuntimeConfig()
    const {stdout, stderr, promise} = await gptscript.streamExecFile(
        'story-book.gpt', `--story ${request.prompt} --pages ${request.pages} --path ${storiesVolumePath}`, {})

    setResponseStatus(event, 202)

    runningScripts[request.prompt] = {
        stdout: stdout,
        stderr: stderr,
        promise: promise
    }
})
```

Now we can set where we want stories to be written by setting **NUXT_STORIES_VOLUME_PATH**. When we build our image in a later step this will be extremely useful as we will want to store these stories in some sort of persistent disk (hence the **VOLUME_PATH**).  
现在我们可以通过设置 **NUXT_STORIES_VOLUME_PATH** 来设置我们想要写故事的位置。当我们在后续步骤中构建图像时，这将非常有用，因为我们希望将这些故事存储在某种持久磁盘中（因此**VOLUME_PATH**）。

## Stamping out the remaining routes we need  
淘汰我们需要的剩余路线

In our last tutorial we cut out a couple of routes in the actual [https://story-book.gptscript-demos.ai/](https://story-book.gptscript-demos.ai/) application in order to be brief with the implementation. Since we’re coming back to this to get everything fully ready, let’s go ahead and stamp out the remaining basic routes used in the story book today.  
在上一个教程中，我们在实际的 [https://story-book.gptscript-demos.ai/](https://story-book.gptscript-demos.ai/) 应用程序中剪掉了几个路由，以便简要地实现。既然我们要回到这里来让一切都做好充分的准备，那么让我们继续消除今天故事书中使用的其余基本路由。

```bash
touch "server/api/story/index.get.ts" "server/api/story/[title].get.ts"
```

This creates two files for us that we’ll want to fill out so let’s tackle them one at a time.  
这将为我们创建两个文件，我们将需要填写这些文件，因此让我们一次处理一个文件。

### server/api/story/index.get.ts  
服务器/api/story/index.get.ts

In the **index.get.ts** file go ahead and add the following:  
在 **index.get.ts** 文件中，继续并添加以下内容：

```tsx
import fs from 'fs'

export default defineEventHandler(async (event) => {
    try {
		    // Read all of the files in the storiesVolumePath.
        return await fs.promises.readdir(useRuntimeConfig().storiesVolumePath)
    } catch (error) {
        // If the error is a 404 error, we can throw it directly.
        if ((error as any).code === 'ENOENT') {
            throw createError({
                statusCode:    404,
                statusMessage: 'no stories found',
            })
        }
        // If any other error occurs, throw it.
        throw createError({
            statusCode:    500,
            statusMessage: `error fetching stories: ${error}`,
        })
    }
})
```

This code dynamically lists all generated stories. It checks the storage path for story files and displays them. If an error, such as a non-existent directory, occurs, it returns a 404 error, ensuring a robust user experience.  
此代码动态列出所有生成的故事。它检查故事文件的存储路径并显示它们。如果发生错误（例如不存在的目录），它将返回 404 错误，从而确保强大的用户体验。

### server/api/story/[title].get.ts  
服务器/api/story/[title].get.ts

```tsx
import fs from 'fs'
import path from 'path';
import type { Pages } from '@/lib/types'

export default defineEventHandler(async (event) => {
    try {
		    // The [title] in the name of this file is a router param. This
		    // is how we grab that value.
        let title = getRouterParam(event, 'title');
        if (!title) {
            throw createError({
                statusCode: 400,
                statusMessage: 'title is required'
            });
        }

				// The name of the story could have values that need to be decoded
        title = decodeURIComponent(title);

				// Grab the files in the storiesVolumePath
        const storiesVolumePath = useRuntimeConfig().storiesVolumePath
        const files = await fs.promises.readdir(path.join(storiesVolumePath, title));
        
        // For each file in the title's directory, parse the txt files and image files.
        // Combine them all together into a singular Pages const.
        const pages: Pages = {};
        for (const file of files) {
            if (!file.endsWith('.txt')) continue
            const page = await fs.promises.readFile(path.join(storiesVolumePath, title, file), 'utf-8');
            pages[file.replace('.txt', '').replace('page', '')] = {
                image_path: `/api/image/${path.join(name, file.replace('.txt', ''))}`,
                content: page
            }
        }

        return pages
    } catch (error) {
        // if the error is a 404 error, we can throw it directly
        if ((error as any).code === 'ENOENT') {
            throw createError({
                statusCode:    404,
                statusMessage: 'story found',
            })
        }
        throw createError({
            statusCode:    500,
            statusMessage: `error fetching story: ${error}`,
        })
    }
})
```

This API fetches specific story content by using the title from the URL. It decodes the title, reads the story’s content, and combines text and images into a structured format. This ensures a smooth user experience, even when errors like a missing story occur.  
此 API 使用 URL 中的标题获取特定故事内容。它对标题进行解码，读取故事的内容，并将文本和图像组合成结构化格式。这确保了流畅的用户体验，即使发生缺失故事等错误时也是如此。

You’ll notice that we are importing a **Pages** type that we have yet to create, so let’s make that now.  
你会注意到我们正在导入一个尚未创建的 **Pages** 类型，所以现在让我们开始导入它。

```bash
mkdir lib
touch lib/types.ts
```

And then we can add the type in there.  
然后我们可以在那里添加类型。

```tsx
export type Pages = Record<string, Page>;
export type Page = {
    image_path: string;
    content: string;
}
```

With that the basic functionality for the API is complete. However, you may have also noticed in the parsing of files that we’re referencing a route we have yet to implement – **/api/image**.  
这样，API 的基本功能就完成了。但是，您可能还注意到，在解析文件时，我们引用了一个尚未实现的路由 – **/api/image**。

## server/api/image/[…path].get.ts  
服务器/api/image/[...路径].get.ts

Since we moved our storage of stories out of the **public** directory, we no longer have anything serving images. To fix this, we can implement a very simple image serving route that will grab it from the **NUXT_STORIES_VOLUME_PATH** and return them over HTTP.  
由于我们将故事存储移出了 **public** 目录，因此我们不再有任何提供图像的内容。为了解决这个问题，我们可以实现一个非常简单的图像服务路由，该路由将从 **NUXT_STORIES_VOLUME_PATH** 中获取它并通过 HTTP 返回它们。

First create the file for Nuxt. Let’s also install **sharp** which is an image conversion and compression library for JavaScript. We’ll be using it to optimize our image serving.  
首先为 Nuxt 创建文件。我们还安装 **sharp**，这是一个用于 JavaScript 的图像转换和压缩库。我们将使用它来优化我们的图像服务。

```tsx
touch "server/api/image/[...path].get.ts"
```

Then we can go ahead and implement image serving logic.  
然后，我们可以继续实现图像服务逻辑。

```tsx
import { existsSync } from 'fs';
import sharp from 'sharp';
import path from 'path';

export default defineEventHandler(async (event) => {
    // Build the image path out of the [...path] router param. The ... in the file
    // name means that any subroutes under image fall under this route as well.
    const imagePath = path.join(useRuntimeConfig().storiesVolumePath, `${getRouterParam(event, 'path')}.png`);    
    if (!existsSync(imagePath)) {
        throw createError({
            statusCode: 404,
            statusMessage: 'file not found'
        });
    }    

		// We'll be returning a JPEG, so set the header.
    event.node.res.setHeader('Content-Type', 'image/jpeg'); 

    // Compress the image using sharp.
    const compressedImage = await sharp(imagePath)
        .jpeg()
        .toBuffer();

		// Tell Nuxt that we'll be manually handling the response and then we
		// use the underlying node.res.end call to terminate with the image.
		event._handled = true;
    event.node.res.end(compressedImage); 
    return event;
});

```

The code serves story illustrations effectively by storing images outside the **public** directory and using the **sharp** library to optimize images before delivery. It maintains a persistent server connection, ensuring real-time updates and a smooth user experience.  
该代码通过将图像存储在**公共**目录之外并在交付前使用 **sharp** 库优化图像来有效地提供故事插图。它维护持久的服务器连接，确保实时更新和流畅的用户体验。

Now we can request and serve any images in the **NUXT_STORIES_VOLUME_PATH.**  
现在我们可以请求和提供 NUXT_STORIES_VOLUME_PATH 中的任何图像**。**

## Adding the ability to render Stories in-app  
添加在应用程序内呈现 Stories 的功能

Now that we have an API that can serve stories from the API, let’s render this on the front-end so users can actually view the stories that they’ve created. First, let’s create the page that we’ll need.  
现在我们有一个可以从 API 提供故事的 API，让我们在前端呈现它，以便用户可以实际查看他们创建的故事。首先，让我们创建我们需要的页面。

```bash
mkdir pages/stories
touch "pages/stories/[name].vue"
```

Then let’s go ahead and implement the story rendering logic.  
然后，我们继续实现故事渲染逻辑。

```html
<script setup lang="ts">
    import type { Pages } from '@/lib/types'
    import unmangleStoryName from '@/lib/unmangle';

    const route = useRoute()

    const name = route.params.name as string
    const pages = ref<Pages>({})
    const currentPage = ref(1)
    const pdfCreating = ref(false)

    onMounted(async () => pages.value = await $fetch(`/api/story/${name}`) as Pages )
    const nextPage = () => currentPage.value++
    const prevPage = () => currentPage.value--
</script>

<template>
    <div class="page text-2xl w-full">
		    <!-- Header w/ title and page number -->
        <div class="flex justify-center">
            <div class="w-full pt-20 md:pt-10 p-2">
                <div class="text-center">
                    <h1 class="text-4xl font-semibold mb-6">
                        {{ unmangleStoryName(name) }}
                    </h1>
                </div>
                <UDivider class="mt-10 text-xl">Page {{ currentPage }}</UDivider>
            </div>
        </div>
        
	      <!-- Buttons for changing the page -->
        <div class="h-full flex flex-col items-center justify-center">
            <UButton v-if="currentPage > 1" @click="prevPage"
                class="absolute left-[2vw] top-1/2"
                :ui="{ rounded: 'rounded-full' }" 
                color="gray"
                icon="i-heroicons-arrow-left" 
            />
            <UButton v-if="currentPage < Object.keys(pages).length" @click="nextPage"
                class="absolute right-[2vw] top-1/2"
                :ui="{ rounded: 'rounded-full' }"
                color="gray"
                icon="i-heroicons-arrow-right"
            />
        </div>
        
        <!-- Page text and image rendering -->
        <div v-if="Object.keys(pages).length > 0" class="flex-col xl:flex-row flex content-center justify-center space-y-20 xl:space-x-20 xl:space-y-0 items-center m-6 lg:m-16">
            <div class="xl:w-1/2 max-w-1/2 text-2xl">
                <p v-html="pages[currentPage].content" style="white-space: pre-line;"></p>
            </div>
            <img class="xl:w-2/5" :src="pages[currentPage].image_path" alt="page image" />
        </div>
    </div>
</template>
```

This code allows for the display of individual stories on a webpage. It fetches the story details from an API, allows users to navigate through story pages, and renders the story text and accompanying images. It uses Vue’s **useRoute()** function to extract the story name from the URL and displays the title and page number. It also handles navigation with **nextPage** and **prevPage** functions, and uses the **v-html** directive to render page content as HTML.  
此代码允许在网页上显示单个故事。它从 API 获取故事详细信息，允许用户浏览故事页面，并呈现故事文本和随附的图像。它使用 Vue 的 **useRoute（）** 函数从 URL 中提取故事名称并显示标题和页码。它还使用 **nextPage** 和 **prevPage** 函数处理导航，并使用 **v-html** 指令将页面内容呈现为 HTML。

In that we reference an **unmangle** import we have yet to implement. This is a function that takes the folder name of stories and turns them into the title format a user would expect. First make the file.  
因为，我们引用了一个尚未实现的 **unmangle** 导入。这是一个函数，它获取故事的文件夹名称，并将它们转换为用户期望的标题格式。首先制作文件。

```bash
touch lib/unmangle.ts
```

Then we can implement the **unmangle** function.  
然后我们就可以实现 **unmangle** 函数了。

```json
const unmangleStoryName = (storyName: string): string => {
		// Replace the -'s with spaces and capitalize the first letter of each word
    return storyName.replaceAll('-', ' ').replace(/(?<=s)bw/g, c => c.toUpperCase()).replace(/^w/, c => c.toUpperCase());
}
export default unmangleStoryName;
```

After that, rendering the story should look something like this.  
之后，渲染故事应如下所示。

![[story_6e453a73c0.png]]

With that complete, we’re able to go to **localhost:3000/story/[title]** to render stories in **NUXT_STORIES_VOLUME_PATH.** Next up we’ll start improving how we stream updates to the UI through SSE.  
完成后，我们能够转到 **localhost：3000/story/[title]** 以 NUXT_STORIES_VOLUME_PATH 呈现故事**。**接下来，我们将开始改进通过 SSE 将更新流式传输到 UI 的方式。

## Improving our use of SSE  
改进我们对 SSE 的使用

When we last touched the SSE implementation, we had a post route that we would submit a prompt and page count and a separate get route to see updates based on the prompt. This is fine at a smaller scale but once we start to take things into a system where more than one user can create stories at a time scaling starts to fall apart. For example, there are a limited number of streams that we can connect to on the front-end.  
当我们上次接触 SSE 实现时，我们有一个 post 路由，我们将提交一个提示和页面计数，以及一个单独的 get 路由来查看基于提示的更新。这在较小的规模下很好，但是一旦我们开始将多个用户可以同时创建故事的系统引入，扩展就开始分崩离析。例如，我们可以在前端连接的流数量有限。

This is where we’re at now.  
这就是我们现在所处的位置。

![[Story-2.webp]]

A more scalable approach to this is to instead have a single get route where all updates for stories get pushed to. Each message that gets pushed through the stream will have an ID that we generate for that story so we know which message is for which story. This means that we only ever have one stream open on the front-end instead of one per generating story. That will look something like this.  
一种更具可扩展性的方法是使用单个 get 路由，故事的所有更新都推送到该路由。通过流推送的每条消息都将具有我们为该故事生成的 ID，因此我们知道哪条消息适用于哪个故事。这意味着我们在前端只有一个流，而不是每个生成故事一个流。这将看起来像这样。

![[Story-1.webp]]

Let’s get to implementing it!  
让我们开始实施它吧！

### Updating the API  更新 API

The complexity of this section relative to the previous ones will increase here. Let’s take it one step at a time so that we understand what we’re doing and why.  
与前面的部分相比，本节的复杂性在这里会增加。让我们一步一步来，以便我们了解我们在做什么以及为什么。

First, let’s start by updating the POST route to:  
首先，让我们首先将 POST 路由更新为：

1. Have a single in-memory event stream  
    具有单个内存中事件流
2. Simplify how we associate what scripts are running  
    简化关联正在运行的脚本的方式
3. Create an ID for each running script instead of using their prompt  
    为每个正在运行的脚本创建一个 ID，而不是使用其提示符
4. Add a **.on()** function for the **stdout** and **stderr** that the node module returns.  
    为 node 模块返回的 **stdout** 和 **stderr** 添加 **.on（）** 函数。
5. Add handling directly to the promises instead of writing them to a store and watching.  
    将 handling 直接添加到 promise 中，而不是将它们写入 store 并进行监视。

```tsx
import gptscript from '@gptscript-ai/gptscript'
import { Readable } from 'stream'
import type { StreamEvent } from '@/lib/types'

type Request = {
    prompt: string;
    pages: number;
}

// This is more simple in-memory store that will tell us which prompts are pending
// based on their ID.
export const runningScripts: Record<string, string>= {}

// This is a single in-memory stream that the front-end will connect to. It is 
// in memory for the app right now but you could make this instead stored in a
// Reddis cache of some sort to scale even better.
export const eventStream = new Readable({read(){}});

export default defineEventHandler(async (event) => {
    const request = await readBody(event) as Request

    // Ensure that the request has the required fields
    if (!request.prompt) { throw createError({
        statusCode: 400,
        statusMessage: 'prompt is required'
    })}
    if (!request.pages) { throw createError({
        statusCode: 400,
        statusMessage: 'pages are required'
    })}

    // Run the script with the given prompt and number of pages
    const {storiesVolumePath } = useRuntimeConfig()
    const opts: { cacheDir?: string } = gptscriptCachePath ? { cacheDir: gptscriptCachePath } : {}
    const {stdout, stderr, promise} = await gptscript.streamExecFile(
        `story-book.gpt`, `--story ${request.prompt} --pages ${request.pages}`, opts)

    // Generate an ID and add it to the runningScripts object
    const id = Math.random().toString(36).substring(2, 14);
    runningScripts[id] = request.prompt

    // Setup listening for data that is received from the script. stdout is always the final message.
    stdout.on('data', (data) => {
        const outboundEvent: StreamEvent = {id, message: data.toString(), final: true}
        eventStream.push(`data: ${JSON.stringify(outboundEvent)}nn`)
    })
    stderr.on('data', (data) => { 
        const outboundEvent: StreamEvent = {id, message: data.toString()}
        eventStream.push(`data: ${JSON.stringify(outboundEvent)}nn`)
    })

    // Handle the promise by deleting the running script from the runningScripts object
    promise
        .catch((err) => { 
            const outboundEvent: StreamEvent = {id, message: err.message, final: true, error: true}
            eventStream.push(`data: ${JSON.stringify(outboundEvent)}nn`)
        })
        .finally(() => { delete runningScripts[id] })
    setResponseStatus(event, 202)
})
```

This new implementation frees us from having to watch the **promise**, **stdout**, and **stderr** through an in-memory struct. Instead, their updates will be automatically written to the singular in-memory event stream we created. You’ll remember that we want to associate each message with an ID so we know what message is meant for which prompt. To do this, this code uses a **StreamEvent** type that we have yet to write so let’s implement that now.  
这个新的实现使我们不必通过内存中的结构体来监视 **promise**、**stdout** 和 **stderr**。相反，它们的更新将自动写入我们创建的单个内存中事件流。你会记得，我们希望将每条消息与一个 ID 相关联，以便我们知道什么消息对应哪个提示。为此，此代码使用了我们尚未编写的 **StreamEvent** 类型，因此我们现在实现它。

We can simply add its type to **lib/types.ts** to accomplish this.  
我们可以简单地将其类型添加到 **lib/types.ts** 来完成此作。

```tsx
export type Pages = Record<string, Page>;

export type Page = {
    image_path: string;
    content: string;
}

export type StreamEvent = {
    id: string; // This will be how we associate a message with its prompt
    message: string; // The message from GPTScript
    final?: boolean; // Indicates if we have any more messages
    error?: boolean; // Indicates if this message is an error
}
```

With all of this implemented, our SSE route can get much simpler.  
实施所有这些后，我们的 SSE 路线可以变得更加简单。

```tsx
import { eventStream } from '@/server/api/story/index.post';

export default defineEventHandler(async (event) => {
    setResponseStatus(event, 200);
    setHeaders(event, {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Credentials': 'true',
        'Connection': 'keep-alive',
        'Content-Type': 'text/event-stream',
    });
    
    event._handled = true;
    eventStream.on('data', (data) => event.node.res.write(data));
});

```

Finally, let’s add a route that allows us to get all of the pending jobs out of the **runningScripts** store.  
最后，让我们添加一个路由，允许我们从 **runningScripts** 存储中获取所有待处理作业。

```bash
touch server/api/story/pending.get.ts
```

```tsx
import { runningScripts } from '@/server/api/story/index.post';
export default defineEventHandler(async () => { return runningScripts });
```

With this implement the new intended flow of receiving updates is for the front-end to:  
通过此实现，接收更新的新预期流程是让前端：

1. On load, connect to the SSE route to start listening for events  
    在加载时，连接到 SSE 路由以开始侦听事件
2. On post, the backend will read the stream updates and place them in the appropriate locations  
    在发布时，后端将读取流更新并将其放置在适当的位置

Let’s wire up our front-end to do just that now.  
现在让我们连接我们的前端来做到这一点。

## Updating the UI  更新 UI

The actual story-book application has a lot of UI features that are relatively trivial to add. Things like the navigation bar, theme toggle, etc. If you’d like to see the code for these things you can see them in the source code of the story-book. However, for the sake of brevity, we’re going to be only updating the essential pieces for getting our application functional. As such, we’ll be adding a section for:  
实际的 story-book 应用程序有很多 UI 功能，添加起来相对来说很容易。比如导航栏、主题切换等。如果你想看到这些东西的代码，你可以在 story-book 的源代码中看到它们。但是，为了简洁起见，我们将只更新使应用程序正常运行的基本部分。因此，我们将添加以下部分：

1. Pending stories  待处理的故事
2. Stories  故事

We’ll tackle each of these one at a time, starting with…  
我们将逐一解决这些问题，从......

### Pending stories  待处理的故事

Last time we had a very simple interface where the form to create a story was on the left of the page and its progress was on the right. We’re going to be overhauling that approach to instead have a few sections. The first of these sections will be for “pending stories” where we’ll have a button to create new stories and hover-able pending stories with their progress streamed from the backend.  
上次我们有一个非常简单的界面，其中创建故事的表单位于页面左侧，其进度位于右侧。我们将彻底修改这种方法，改为包含几个部分。这些部分中的第一个部分将用于“待处理故事”，其中我们将有一个按钮来创建新故事和可悬停的待处理故事，其进度从后端流式传输。

We’re going to be creating components for this. Components are reusable chunks of the UI that contain both mark-up and the required logic for their rendering and we’ll be making components for all of the new UI updates. In Nuxt components are automatically imported into every file on our behalf so long as they’re in the **components** directory.  
我们将为此创建组件。组件是 UI 的可重用块，其中包含标记和渲染所需的逻辑，我们将为所有新的 UI 更新制作组件。在 Nuxt 中，只要组件位于 **components** 目录中，它们就会代表我们自动导入到每个文件中。

Let’s create it along with the file for the **PendingStories** component.  
让我们将它与 **PendingStories** 组件的文件一起创建。

```bash
mkdir components
touch components/PendingStories.vue
```

In this component we’re going to be rendering the pending stories that are stored in the backend along with a hover-able section for each of them that displays progress for that particular story. This will be done by two get calls:  
在这个组件中，我们将渲染存储在后端的待处理故事，以及每个故事的可悬停部分，用于显示该特定故事的进度。这将通过两个 get 调用来完成：

1. Getting the pending stories  
    获取待处理的故事
2. Opening an EventSource to our **/api/story/sse** endpoint  
    打开一个 EventSource 到我们的 **/api/story/sse** 端点

With that understood, let’s write the code for our **PendingStories** component!  
理解了这一点后，让我们为 **PendingStories** 组件编写代码！

```html
<script setup lang="ts">
import { useMainStore } from '@/store'
import type { StreamEvent } from '@/lib/types'

// We'll be talking about what a store is along with how to use it next
const store = useMainStore()

// Toast's are @nuxt/ui's notification framework. Since its a composable,
// we call useToast() here. 
const toast = useToast()

// Store the pending stories
const pendingStories = computed(() => store.pendingStories)

const previousMessages = ref<Record<string, string>>({})
const messages = ref<Record<string, string>>({})

// An event source is a JS DOM object for connecting to compatible SSE
// endpoints.
const es = ref<EventSource>()

// addMessage will take an id and message and add it to the appropriate
// key in messages.
const addMessage = (id:string, message: string) => {
    if (!messages.value[id]) messages.value[id] = ''
    if (!message) return
    messages.value[id] += message
}

// When we mount the component, fetch any pending stories
onMounted(async () => store.fetchPendingStories() )

// Close the stream when the component is unmounted
onBeforeUnmount(() => { es.value?.close() })

// Before we mount the component, open an event stream and setup the 
// event handlers for it.
onBeforeMount(() => { 
    es.value = new EventSource('/api/story/sse')
    
    // Setup the EventSource event handler
    es.value.onmessage = (event) => {
		    // Parse the event as StreamEvent
        const e = JSON.parse(event.data) as StreamEvent
        addMessage(e.id, e.message)

				// If this is the final message, determine if it is a 
				// success or failure
        if (e.final) {
		        // In either case, fetch the stories
            store.fetchStories()
            
            
            if (e.error) {
	            // If this was an error message, then truncate the prompt and notify
	            // the user that its creation failed. The previous message from our
	            // error message will be the failure reason.
                const truncatedPrompt = pendingStories.value[e.id].length > 50 ? pendingStories.value[e.id].substring(0, 50) + '...' : pendingStories.value[e.id]
                toast.add({
                    id: 'story-generating-failed',
                    title: `"${truncatedPrompt}" Generation Failed`,
                    description: `${previousMessages.value[e.id]}.`,
                    icon: 'i-heroicons-x-mark',
                    timeout: 30000,
                })
            } else {
								// Successful story creation
                toast.add({
                    id: 'story-created',
                    title: 'Story Created',
                    description: `A story you requested has been created.`,
                    icon: 'i-heroicons-check-circle-solid',
                })
            }
            
            // Delete all of the messages from the variable for this pending story
            delete messages.value[e.id]
            
            // Update the pending stories
            store.fetchPendingStories()
        }

        // Previous message is stored for error handling. When an error occurs, the previous message is the error message.
        if (messages.value[e.id]) {
            previousMessages.value[e.id] = e.message
        }
    }}
)
</script>

<template>
    <div class="flex flex-col space-y-2">
			  <!-- We'll create the component soon-->
        <New class="w-full"/>
        
        <!--
	        UPopover is an element that appears when, in this case, you hover
	        an element. We create one with a loading button for every pending
	        story that will recieve the appropriate messages.
	      -->
        <UPopover mode="hover" v-for="(prompt, id) in pendingStories" >
            <UButton truncate loading :key="id" size="lg" class="w-full text-xl" color="white" :label="prompt" icon="i-heroicons-book-open"/>

            <template #panel>
                <UCard class="p-4 w-[80vw] xl:w-[40vw]">
                    <h1 class="text-xl">Writing the perfect story...</h1>
                    <h2 class="text-zinc-400 mb-4">GPTScript is currently writing a story. Curious? You can see what it is thinking below!</h2>
                    <pre class="h-[26vh] bg-zinc-950 px-6 text-white overflow-scroll rounded shadow flex flex-col-reverse" style="white-space: pre-line;">
                        {{ messages[id] ? messages[id] : 'Waiting for response...'}}
                    </pre>
                </UCard>
            </template>
        </UPopover>
    </div>
</template>
```

This Vue component, created with the Composition API, is designed to render a list of pending stories. Let’s break down its key features:  
这个 Vue 组件是使用组合式 API 创建的，旨在呈现待处理的故事列表。让我们分解一下它的主要功能：

- **pendingStories** is a computed property that fetches the list of pending stories from the store.  
    **pendingStories** 是一个计算属性，用于从 store 中获取待处理故事的列表。
- **previousMessages** and **messages** are used to store the most recent message and the entire message history for each story ID, respectively.  
    **previousMessages** 和 **messages** 分别用于存储每个故事 ID 的最新消息和整个消息历史记录。
- **es** is a reference to an EventSource object, utilized for Server-Sent Events (SSE).  
    **es** 是对 EventSource 对象的引用，用于服务器发送事件 （SSE）。

Three lifecycle hooks are employed:  
使用了三个生命周期钩子：

- **onMounted** is activated when the component first loads, triggering a fetch of the pending stories.  
    **onMounted** 在组件首次加载时激活，从而触发对待处理 stories 的获取。
- **onBeforeUnmount** ensures the EventSource is closed when the component is unmounted, preventing potential memory leaks.  
    **onBeforeUnmount** 确保在卸载组件时关闭 EventSource，从而防止潜在的内存泄漏。
- **onBeforeMount** sets up the EventSource and its event handlers prior to the component rendering.  
    **onBeforeMount** 在组件渲染之前设置 EventSource 及其事件处理程序。

In the **onBeforeMount** hook, a new EventSource is created to listen to the **/api/story/sse** endpoint. An **onmessage** event handler is set up to manage incoming events from the server. Each event is parsed as a **StreamEvent**, the message is added to the appropriate message history, and a check is performed to determine if the message is the final one for a given story. If it is, a fetch of the stories from the server is triggered, the success or failure of the story generation is determined, and a notification is sent to the user accordingly. The message history for that story ID is then cleaned up and the pending stories are fetched again.  
在 **onBeforeMount** 钩子中，创建一个新的 EventSource 来侦听 **/api/story/sse** 端点。设置 **onmessage** 事件处理程序来管理来自服务器的传入事件。每个事件都被解析为 **StreamEvent**，消息被添加到相应的消息历史记录中，并执行检查以确定该消息是否是给定故事的最终消息。如果是，则触发从服务器获取故事，确定故事生成的成功或失败，并相应地向用户发送通知。然后清理该故事 ID 的消息历史记录，并再次获取待处理的故事。

In the template section, a **New** component (to be created) is rendered, followed by a list of **UPopover** components for each pending story. Each **UPopover** contains a **UButton** that displays a loading animation and the story prompt, and a **UCard** that shows the current progress of the story generation. The **UCard** includes a **pre** element that renders the message history for the given story ID, enabling the user to receive real-time updates on the story generation process.  
在模板部分中，将呈现一个 **New** 组件（待创建），后跟每个待处理故事的 **UPopover** 组件列表。每个 **UPopover** 都包含一个 **UButton**，用于显示加载动画和故事提示，以及一个 **UCard**，用于显示故事生成的当前进度。**UCard** 包括一个 **pre** 元素，该元素呈现给定故事 ID 的消息历史记录，使用户能够接收有关故事生成过程的实时更新。

There are a few things here that we have yet to talk about or implement.  
这里有一些事情我们尚未讨论或实施。

1. Our store  本店
2. The **New** component  
    **New** 组件

Let’s start with our store.  
让我们从我们的商店开始。

### Creating our store  创建我们的商店

Stores in component-based frameworks like Vue and React allow you to have state that persists across components and pages. In our case, it allows for our components to more easily react to new stories and pending stories. For this tutorial, we’ll use [pinia](https://pinia.vuejs.org/).  
在基于组件的框架（如 Vue 和 React）中存储允许你拥有跨组件和页面持续存在的状态。在我们的例子中，它允许我们的组件更容易对新 story 和待处理 story 做出反应。在本教程中，我们将使用 [pinia](https://pinia.vuejs.org/)。

```bash
npm i pinia @pinia/nuxt
```

Pinia comes with a Nuxt module as well so we need to add it to our config.  
Pinia 也带有一个 Nuxt 模块，因此我们需要将其添加到我们的配置中。

```tsx
// https://nuxt.com/docs/api/configuration/nuxt-config
export default defineNuxtConfig({
  devtools: { enabled: true },
  modules: [
    '@nuxt/ui',
    '@nuxtjs/tailwindcss',
    '@pinia/nuxt',
  ],
  runtimeConfig: {
    storiesVolumePath: 'stories', // NUXT_STORIES_VOLUME_PATH
  },
})

```

With that done, we need to create our store files.  
完成后，我们需要创建我们的 store 文件。

```bash
mkdir store
touch store/index.ts
```

And finally, we can implement the store.  
最后，我们可以实现 store。

```tsx
// store/index.ts
import { defineStore } from 'pinia'

export const useMainStore = defineStore({
    id: 'main',
    // This is the state that we're keeping track of
    state: () => ({
        pendingStories: {} as Record<string, string>,
        stories: [] as string[],
    }),
    // These are the actions that we can perform on that state. They're 
    // all relatively simple data retrieval actions.
    actions: {
        addStory(name: string) {
            this.stories.push(name)
        },
        addStories(names: string[]) {
            names.forEach(name => {
                if (!this.stories.includes(name)) {
                    this.stories.push(name)
                }
            })
        },
        removeStory(name: string) {
            this.stories = this.stories.filter(s => s !== name)
        },
        async fetchStories() {
            this.addStories(await $fetch('/api/story') as string[])
        },
        async fetchPendingStories() {
            this.pendingStories = await $fetch('/api/story/pending') as Record<string, string>
        },
    }
})
```

Here, we’re defining a store using **pinia’s** **defineStore()** function. This store will be used to manage the state of our application. The state we’re tracking includes **pendingStories**, which is an object with keys as story IDs and values as prompts, and **stories**, which is an array of story names. The **actions** section specifies the different operations we can perform on our state. We have addStory() to add a single story, **addStories()** for adding multiple stories, and **removeStory()** to delete a story from our state. **fetchStories()** and **fetchPendingStories()** are asynchronous actions that fetch the current stories and pending stories respectively from the backend using the **$fetch()** function. The fetched data is then added to our state.  
在这里，我们使用 **pinia 的****defineStore（）** 函数定义一个 store。这个 store 将用于管理我们应用程序的状态。我们跟踪的状态包括 **pendingStories**，它是一个以键作为故事 ID，以值作为提示的对象，以及 **stories**，它是一个故事名称数组。**actions** 部分指定了我们可以对 state 执行的不同作。我们有 addStory（） 来添加单个故事，**addStories（）** 用于添加多个故事，**removeStory（）** 用于从我们的 state 中删除故事。**fetchStories（）** 和 **fetchPendingStories（）** 是异步作，它们使用 **$fetch（）** 函数分别从后端获取当前故事和待处理的故事。然后，获取的数据被添加到我们的状态中。

Let’s move on to update how we create stories now.  
让我们继续更新我们现在创建故事的方式。

### New form  新建表单

Last time, we had our form appear on the screen at all times. This is fine but since we’re moving toward a full application we want to instead have a button that opens a modal to fill out the form.  
上次，我们的表单一直显示在屏幕上。这很好，但由于我们正在转向完整的应用程序，因此我们希望有一个按钮来打开一个模态框来填写表单。

```html
<script setup lang="ts">
import { useMainStore } from '@/store'

// Declare necessary values and utilize the store
const store = useMainStore()
const open = ref(false)
const pageOptions = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
const toast = useToast()

// Variable to contain the values of the form
const formValues = reactive({
    prompt: '',
    pages: 0,
})

// onSubmit will submit the values of the form to our API
async function onSubmit () {
		// Create the story using fetch with our formValues
    const response = await fetch('/api/story', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formValues)
    })

    if (response.ok) {
		    // If the response succeeded, notify the user and fetch
		    // the pending stories
        toast.add({
            id: 'story-generating',
            title: 'Story Generating',
            description: 'Your story is being generated. Depending on the length, this may take a few minutes.',
            icon: 'i-heroicons-pencil-square-solid',
        })
        store.fetchPendingStories()
    } else {
			  // If the response failed, notify the user of the failure reason
        toast.add({
            id: 'story-generating-failed',
            title: 'Story Generation Failed',
            description: `Your story could not be generated due to an error: ${response.statusText}.`,
            icon: 'i-heroicons-x-mark',
        })
    }
    // Finally, close the modal 
    open.value = false
}
</script>

<template>
    <UButton size="lg" color="gray" class="w-full text-xl" icon="i-heroicons-plus" @click="open = true">New Story</UButton>
    <!-- When the above UButton is clicked, UModal will open -->
    <UModal fullscreen v-model="open" :ui="{width: 'sm:max-w-3/4 w-4/5 md:w-3/4 lg:w-1/2', }">
        <UCard class="h-full">
            <template #header>
                <div class="flex items-center justify-between">
                    <h1 class="text-2xl">Create a new story</h1>
                    <UButton color="white" icon="i-heroicons-x-mark-20-solid" class="-my-1" @click="open = false" />
                </div>
            </template>
            <div >
                <UForm class="h-full" :state=state @submit="onSubmit">
                    <UFormGroup size="lg" class="mb-6" label="Pages" name="pages">
                        <USelectMenu class="w-1/4 md:w-1/6"v-model="formValues.pages" :options="pageOptions"/>
                    </UFormGroup>
                    <UFormGroup class="my-6" label="Story" name="prompt">
                            <UTextarea :ui="{base: 'h-[50vh]'}" size="xl" class="" v-model="formValues.prompt" label="Prompt" placeholder="Put your full story here or prompt for a new one"/>
                    </UFormGroup>
                    <UButton size="xl" color="white" icon="i-heroicons-book-open" type="submit">Create Story</UButton>
                </UForm>
            </div>
        </UCard>    
    </UModal>
</template>
```

This component allows users to create a new story through a form in a modal. It enhances user experience by keeping the main page uncluttered. Upon form submission, an API call initiates the story generation on the server, and users receive notifications about the process. It’ll look like this:  
此组件允许用户通过模态中的表单创建新故事。它通过保持主页整洁来增强用户体验。提交表单后，API 调用将在服务器上启动故事生成，并且用户会收到有关该流程的通知。它看起来像这样：

![[create_13bfbca455.png]]

### Updating the homepage  更新主页

Finally, let’s update our homepage to use all of these new components.  
最后，让我们更新我们的主页以使用所有这些新组件。

```html
<script setup lang="ts">
    import { useMainStore } from '@/store'
    const store = useMainStore()
    onMounted(async () => store.fetchStories() )
</script>

<template>
    <UContainer class="w-full h-full flex items-center justify-center">
        <div class="w-full h-full md:h-1/2 p-2 mt-32">
            <h1 class="text-4xl text-center">Welcome to the Story Book!</h1>
            <h2 class="mt-6 text-xl text-center text-gray-500">
                Select one of today's story below or generate a new one. 
            </h2>
            <div class="flex mt-10 flex-col space-y-10 lg:flex-row lg:space-x-10 lg:space-y-0">
                <UCard class="w-full lg:w-1/2">
                    <template #header>
                        <h1>Generate a new story</h1>
                    </template>
                    <PendingStories />
                </UCard>
                <UCard class="w-full lg:w-1/2">
                    <template #header>
                        <h1>Completed Stories</h1>
                    </template>
                    <Stories />
                </UCard>
            </div>
        </div>
    </UContainer>
</template>
```

Our application should now look a little something like this now:  
我们的应用程序现在应该看起来有点像这样：

![[home_04af8b512e.png]]

We’re in the home stretch! Let’s get this application packed and wrap up.  
我们正处于最后冲刺阶段！让我们打包并总结此应用程序。

## Bundling this into a container  
将此 bbundle 到容器中

If we want to actually deploy this thing, we’ll likely want it to be bundled into a container. To do this, we’ll need a Dockerfile.  
如果我们想要实际部署这个东西，我们可能希望将其捆绑到一个容器中。为此，我们需要一个 Dockerfile。

```bash
touch Dockerfile
```

And then we can fill it out.  
然后我们可以填写它。

```docker
FROM node:20 as dev
WORKDIR /app
COPY . .
RUN npm install
RUN ./node_modules/.bin/gptscript --assemble story-book.gpt > story-book-assembled.gpt
EXPOSE 3000
CMD [ "npm", "run", "dev" ]

FROM node:20 as prod
WORKDIR /app
COPY --from=dev /app .
RUN npm run build
CMD [ "npm", "run", "start" ]

```

When you go to deploy this, you’ll need to make sure you do so we your **OPENAI_API_KEY** variable set. Most if not all deployment services available today allow this.  
当您要部署它时，您需要确保这样做，以便我们设置了 **OPENAI_API_KEY** 变量。目前可用的大多数（如果不是全部）部署服务都允许这样做。

Finally, we’ll want to setup our CI/CD to build and deploy this image in Github. I’ll be using Github actions since its easy to integrate with any repo but feel free to choose your CI/CD platform of choice.  
最后，我们需要设置 CI/CD 以在 Github 中构建和部署此映像。我将使用 Github作，因为它很容易与任何存储库集成，但请随意选择您选择的 CI/CD 平台。

First, create the relevant files.  
首先，创建相关文件。

```bash
mkdir -p .github/workflows
touch release.yaml
```

Then write the workflow.  然后编写工作流程。

```yaml
name: release
on:
  push:
    branches:
    - main
    tags:
    - v*

jobs:
  build_push:
    runs-on: ubuntu-latest
    steps:
    - name: Check out the repo
      uses: actions/checkout@v3

    - name: Login to ghcr.io
      uses: docker/login-action@v1
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Build and push the image
      uses: docker/build-push-action@v2
      with:
        context: .
        push: true
        tags: ghcr.io/${{ github.repository }}:${{ github.ref_name }}
```

This is a drop-in action and will work in any repo it is put into. It uses Github’s authentication key to push the image to GHCR.  
这是一个 drop-in作，适用于它放入的任何存储库。它使用 Github 的身份验证密钥将映像推送到 GHCR。

With that, we’re done! You can take this image and deploy it anywhere you please.  
这样，我们就完成了！您可以获取此映像并将其部署到您喜欢的任何位置。