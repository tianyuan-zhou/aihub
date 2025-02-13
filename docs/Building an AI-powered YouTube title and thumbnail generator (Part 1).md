
构建 AI 驱动的 YouTube 标题和缩略图生成器（第 1 部分）

###### April 17, 2024 by stein ove helset  

In this article, I’m going to show you how to get started with GPTScript by installing and setting up everything we need, and then build a script that can generate a title and a thumbnail based on the script for a video.  
在本文中，我将向您展示如何通过安装和设置我们需要的一切来开始使用 GPTScript，然后构建一个可以根据视频脚本生成标题和缩略图的脚本。

For additional reference I’ve created this video guide below:  
为了获得更多参考，我在下面创建了这个视频指南： https://youtu.be/r9nJ_lZyiKk

## Getting started  开始

**First of all, what is GPTScript?  
首先，什么是 GPTScript？**

GPTScript is sort of like a natural programming language where you can make or use tools to do a lot of awesome things, for example using the OpenAI API to understand what a text is all about and then generate an image based on this.  
GPTScript 有点像一种自然编程语言，您可以在其中制作或使用工具来做很多很棒的事情，例如使用 OpenAI API 来理解文本的全部内容，然后基于此生成图像。

Instead of explaining too much here, you can read the blog post from the official launch here: [https://www.acorn.io/resources/blog/introducing-gptscript](https://www.acorn.io/resources/blog/introducing-gptscript)  
无需在此处解释太多，您可以在此处阅读正式发布的博客文章：[https://www.acorn.io/resources/blog/introducing-gptscript](https://www.acorn.io/resources/blog/introducing-gptscript)

**Installing and setting up GPTScript  
安装和设置 GPTScript**

It is very easy to get GPTScript up and running on your own computer.  
在您自己的计算机上启动并运行 GPTScript 非常容易。

**A short guide:  简短指南：**

1. First you install the CLI (Command line interface) by running this command:  
    首先，通过运行以下命令安装 CLI（命令行界面）：

```json
MacOS / Linux:
brew install gptscript-ai/tap/gptscript

Windows:
winget install gptscript-ai.gptscript
```

2. Get your OpenAI API key by going here ([https://platform.openai.com/api-keys](https://platform.openai.com/api-keys))  
    通过访问此处获取您的 OpenAI API 密钥 （[https://platform.openai.com/api-keys](https://platform.openai.com/api-keys)）
    
3. Add the API key to your environment by running this command:  
    通过运行以下命令将 API 密钥添加到您的环境中：
    

```json
MacOS / Linux:
export OPENAI_API_KEY="your-api-key”

Windows:
$env:OPENAI_API_KEY = 'your-api-key'
```

And now you should be ready to jump to the next step. If you need more details for the installation, you can go to the official getting started guide for GPTScript by clicking here ([https://docs.gptscript.ai/getting-started](https://docs.gptscript.ai/getting-started)).  
现在您应该准备好跳到下一步。如果您需要有关安装的更多详细信息，可以单击此处访问 GPTScript 的官方入门指南 （[https://docs.gptscript.ai/getting-started](https://docs.gptscript.ai/getting-started)）。

## Prices  价格

Before you start running the commands for running GPTScript, it might be smart to take a look at the price lists here:  
在开始运行运行 GPTScript 的命令之前，查看此处的价目表可能是明智的：  
[https://openai.com/pricing](https://openai.com/pricing)

Using OpenAI’s API is not free, but you do not pay for more than you actually use.  
使用 OpenAI 的 API 不是免费的，但您无需为超出实际使用量付费。

The script we are building in this tutorial will use a language model called GPT-4 Turbo and an Image model called DALL-E 3. Per thumbnail we are generating, we are spending $0.12 for the image model and a few cents for the rest. This can vary a little bit based on what you actually are sending in to the GPTscript, how long the texts are, etc.  
我们在本教程中构建的脚本将使用名为 GPT-4 Turbo 的语言模型和名为 DALL-E 3 的图像模型。我们生成的每个缩略图，我们为图像模型花费 0.12 美元，其余部分花费几美分。这可能会根据您实际发送到 GPTscript 的内容、文本的长度等而略有不同。

Take a good look at the price lists to familiarize your self with them.  
仔细查看价目表以熟悉它们。

## How to create a script  
如何创建脚本

The first thing I want to to now is to create a script we can run. The project we’re building today will be called “gpt-youtube-generator”. You can create a folder with this name on your computer, or you can call the project a different name if you want to do so.  
我现在想做的第一件事是创建一个我们可以运行的脚本。我们今天正在构建的项目将称为“gpt-youtube-generator”。您可以在计算机上创建一个具有此名称的文件夹，或者如果您愿意，可以为该项目命名不同的名称。

Inside the “gpt-youtube-generator” folder, create a script called “youtube-generator.gpt”.  
在 “gpt-youtube-generator” 文件夹中，创建一个名为 “youtube-generator.gpt” 的脚本。

Open up the file you created in an editor and add the following content:  
打开您在编辑器中创建的文件并添加以下内容：

```json
Say Hello GPTScript!
```

This is almost as basic as you can make a script. Let’s try running it by typing this command in a terminal:  
这几乎与您可以制作脚本一样基本。让我们尝试通过在终端中键入以下命令来运行它：

```json
$ gptscript youtube-generator.gpt
```

What this does is that it just tells the script to “Say” “Hello GPTScript!”. Your terminal should now be printing that on the screen.  
它的作用是，它只是告诉脚本 “Say” “Hello GPTScript！”。您的终端现在应该在屏幕上打印它。

## How to get arguments from the command line  
如何从命令行获取参数

Let’s do some changes to the “youtube-generator.gpt” file:  
让我们对 “youtube-generator.gpt” 文件进行一些更改：

```json
description: Generates a YouTube title and thumbnail based on a script.
args: script: The script to generate title and thumbnail for.

Print the script on the screen.
```

As you can see, here are a few new things.  
如您所见，这里有一些新内容。

1. The “description” is not required at this point, but it’s nice to provide a description of what you want this script to achieve.  
    此时，“description” 不是必需的，但最好提供您希望此脚本实现的目标的描述。
    
2. Next we have a key called “args” where we add one argument called “script”. Here you need to provide a short explanation of what the argument is used for  
    接下来，我们有一个名为 “args” 的键，我们在其中添加了一个名为 “script” 的参数。在这里，您需要简要说明该参数的用途
    

Let’s try running it:  让我们尝试运行它：

```json
$ gptscript gpttest.gpt

Results:
OUTPUT:
Could you please provide the script you want to be printed on the screen?
```

Here we did not provide a “script” or any other arguments for the script. This resulted the script to notify about this exact problem. Plus it also says what it wanted with the argument.  
在这里，我们没有为脚本提供 “script” 或任何其他参数。这导致脚本通知了这个确切的问题。此外，它还说明了它想要的参数。

Let’s try again, now with a “`--script`” argument with the value “This is a test”:  
让我们再试一次，现在使用值为 “This is a test” 的 “`--script`” 参数：

````json
$ gptscript gpttest.gpt --script This is a test

Results:
INPUT:
--script This is a test

OUTPUT:
This is a test
```json
Now you can see that the script understands what the input is. When this is processed, we see an output on the screen.

## Instructing the script to generate a title based on the input

Now that we know how to read the arguments from the terminal, we can go one step further and generate a title for a YouTube video based on this.

Let’s make some more changes to the “youtube-generator.gpt” file:
```json
description: Generates a YouTube title and thumbnail based on a script.
args: script: The script to generate title and thumbnail for.

Do the following steps in acsending order:
1. Come up with an appropriate title for the video based on the script.
2. Print the title on the screen.
````

Here isn’t really any new concepts here. We have just instructed GPTScript in what to do with the script we provide. It can be helpful to be specific like we are here when we say that the script should do the following steps in ascending order.  
这里实际上并没有什么新概念。我们刚刚指示 GPTScript 如何处理我们提供的脚本。像我们在这里说脚本应该按升序执行以下步骤一样具体可能会有所帮助。

We can now try running it and see what we get:  
我们现在可以尝试运行它，看看我们得到了什么：

```json
$ gptscript gpttest.gpt --script This is a test for a video where you will learn about GPTScript

Results:
INPUT:
--script This is a test for a video where you will learn about GPTScript

OUTPUT:
"Mastering GPTScript: A Comprehensive Guide"
```

So there you have a unique title generated by the simple GPTScript we have created.  
因此，您有一个由我们创建的简单 GPTScript 生成的唯一标题。

## Creating a tool for making a folder  
创建用于创建文件夹的工具

I want to make a folder where we can store the thumbnail we are going to generate. And I want the name of the folder to be the title of the video.  
我想创建一个文件夹，我们可以在其中存储我们将要生成的缩略图。我希望文件夹的名称成为视频的标题。

To achieve this, we need to create a tool which our script can use.  
为了实现这一点，我们需要创建一个脚本可以使用的工具。

Let’s make some more changes to the “youtube-generator.gpt” file:  
让我们对 “youtube-generator.gpt” 文件进行更多更改：

```json
tools: mkdir, sys.write # 1. new
description: Generates a YouTube title and thumbnail based on a script.
args: script: The script to generate title and thumbnail for.

Do the following steps in acsending order:
1. Come up with an appropriate title for the video based on the script.
2. Create the `${video-title}` folder if it does not already exist. # 2. new/replaced

--- # 3 new
name: mkdir # 4 new
tools: sys.write # 5 new
description: Creates a folder in your system # 6 new
args: dir: Path to the folder to be created. # 7 new

#!bash # 8 new

mkdir -p "${dir}" # 9 new
```

This time we have done quite a few changes. Let’s go through them one by one.  
这次我们做了不少更改。让我们一一介绍一下。

1. First we define which tools we are going to use in this script. “mkdir” is a tool we are creating in this step. “sys.write” is a tool built into GPTScript for writing files, folders, etc in your file system.  
    首先，我们定义将在此脚本中使用哪些工具。“mkdir” 是我们在此步骤中创建的工具。“sys.write” 是 GPTScript 中内置的工具，用于在文件系统中写入文件、文件夹等。
    
2. Instead of just printing the name of the video on the screen, we now call the tool we are creating and instructing it to create a folder with the name of the video. As you can see, GPTScript just automatically understand that it should be using the tool.  
    我们现在不仅仅是在屏幕上打印视频的名称，而是调用我们正在创建的工具并指示它创建一个带有视频名称的文件夹。如您所见，GPTScript 只是自动理解它应该使用该工具。
    
3. All tools should come after the “main” script we are building. Tools are separated by three dashes “`---`”.  
    所有工具都应该位于我们正在构建的 “main” 脚本之后。工具由三个破折号 “`---`” 分隔。
    
4. Define a name for the tool. In our case it will be “mkdir”.  
    定义工具的名称。在我们的例子中，它将是 “mkdir”。
    
5. Tell the tool which other tools to import and use for this exact tool.  
    告诉该工具要导入哪些其他工具并将其用于此确切工具。
    
6. Write a little description for what this tool does.  
    为此工具的功能写一点描述。
    
7. Just like the main script, we want to receive and use an argument. In this case it’s “dir” (the title of the video).  
    就像主脚本一样，我们想要接收并使用一个参数。在本例中，它是 “dir” （视频的标题）。
    
8. GPTScript can’t directly interact with our file system, so in this case, we need to create a simple bash script for creating folders.  
    GPTScript 不能直接与我们的文件系统交互，因此在这种情况下，我们需要创建一个简单的 bash 脚本来创建文件夹。
    
9. Run a command “mkdir” (this is a OS specific commando, not the tool) to create a folder.  
    运行命令 “mkdir” （这是特定于作系统的 commando，而不是工具） 来创建文件夹。
    

If we now run the same command as in the previous step, you will see something like this:  
如果我们现在运行与上一步相同的命令，您将看到如下内容：

```json
$ gptscript gpttest.gpt --script This is a test for a video where you will learn about GPTScript

Results:
INPUT:
--script This is a test for a video where you will learn about GPTScript

OUTPUT:
The folder "Learn About GPTScript: A Comprehensive Guide" has been created
```

Perfect! You should now see a folder in the projects root folder with the name that our script generated. Not only that, but we see that in the output, it explains what it did. And this explanation is a good confirmation that the script understood what we wanted it to do.  
完善！ 现在，您应该在项目根文件夹中看到一个文件夹，其中包含脚本生成的名称。不仅如此，我们还在输出中看到，它解释了它的作用。这个解释很好地证实了脚本理解了我们希望它做什么。

## Creating the tool for generating thumbs  
创建用于生成 Thumb 的工具

Let’s kick things up a little bit and make a tool for generating images. Or at least build a tool that uses a library with DALL-E in the backend for generating images.  
让我们稍微改进一下，制作一个生成图像的工具。或者至少构建一个工具，使用后端带有 DALL-E 的库来生成图像。

At the bottom of the script file, add the following content:  
在脚本文件底部，添加以下内容：

```json
---
name: thumb-generator
tools: github.com/gptscript-ai/image-generation
description: Generates a YouTube thumbnail.
args: script: The script to generate thumbnail from.

Generate a YouTube thumbnail based on the script. Only return the url of the thumbnail.
```

So here we define a new Tool called “thumb-generator”. We instruct it to use a tool called “[github.com/gptscript-ai/image-generation](http://github.com/gptscript-ai/image-generation)”. This tool can be used for various tasks when it comes to image generations.  
所以在这里我们定义了一个名为 “thumb-generator” 的新工具。我们指示它使用一个名为 “[github.com/gptscript-ai/image-generation](http://github.com/gptscript-ai/image-generation)” 的工具。在图像生成方面，该工具可用于各种任务。

Next we add a short description and set ut the arguments we want. This argument comes from the “main” script.  
接下来，我们添加一个简短的描述，并为 ut 设置我们想要的参数。这个参数来自 “main” 脚本。

Below there, we give the tool instructions on what we do. We want it to generate a YouTube thumbnail based on the script, and then we want it to return the url for the thumbnail for us.  
在下面，我们给出了工具说明我们做什么。我们希望它根据脚本生成一个 YouTube 缩略图，然后我们希望它为我们返回缩略图的 url。

Last step before we can test this, is to make some changes further up in the script:  
在我们测试之前的最后一步是在脚本中进一步进行一些更改：

```json
tools: thumb-generator, mkdir, sys.write, sys.download # 1 - Changed
description: Generates a YouTube title and thumbnail based on a script.
args: script: The script to generate title and thumbnail for.

Do the following steps in acsending order:
1. Come up with an appropriate title for the video based on the script.
2. Create the `${video-title}` folder if it does not already exist. 
3. Call thumb-generator to illustrate it. # 2 - New
4. Download the illustration to a file at `${video-title}/thumb.png`. # 3 - New

---
name: mkdir
tools: sys.write
description: Creates a folder in your system
args: dir: Path to the folder to be created.

#!bash

mkdir -p "${dir}"

---
name: thumb-generator
tools: github.com/gptscript-ai/image-generation
description: Generates a YouTube thumbnail.
args: script: The script to generate thumbnail from.

Generate a YouTube thumbnail based on the script. Only return the url of the thumbnail.
```

1. First, we include the new tool we created “thumb-generator”. Then we include one more tool called “sys.download”. This tool makes it possible to download the url we got in the new tool we created.  
    首先，我们包含我们创建的新工具“thumb-generator”。然后我们再包含一个名为“sys.download”的工具。此工具可以下载我们在创建的新工具中获得的 URL。
    
2. Added a step to the script where we tell it to call the new tool we created.  
    向脚本中添加了一个步骤，我们告诉它调用我们创建的新工具。
    
3. Added a step to the script for downloading the thumbnail and storing it in the folder that was created.  
    向脚本中添加了一个步骤，用于下载缩略图并将其存储在创建的文件夹中。
    

Okay, it’s time to try running this again and see what we get.  
好的，是时候再次尝试运行它，看看我们能得到什么。

```json
$ gptscript gpttest.gpt --script This is a test for a video where you will learn about GPTScript

Results:
INPUT:
--script This is a test for a video where you will learn about GPTScript

OUTPUT:
The illustration has been downloaded to the file at `Learn GPTScript: A Comprehensive Guide/thumb.png`.
```

This is the image I got when I ran this command:  
这是我运行此命令时得到的图像：  
![Image of a book with the text, GPTScript](https://necessary-creativity-bc071c3082.media.strapiapp.com/thumb1_b0cdf58656.jpg)  

It doesn’t really look like a YouTube thumbnail, but it is a good starting point.  
它看起来并不像 YouTube 缩略图，但是一个很好的起点。

## Improve the image generator  
改进图像生成器

The image we generated is squared and I also want to do some other changes. Open up the script file again and do the following changes:  
我们生成的图像是方形的，我还想做一些其他更改。再次打开脚本文件并执行以下作：

```json
tools: thumb-generator, mkdir, sys.write, sys.download
description: Generates a YouTube title and thumbnail based on a script.
args: script: The script to generate title and thumbnail for.

Do the following steps in acsending order:
1. Come up with an appropriate title for the video based on the script.
2. Create the `${video-title}` folder if it does not already exist. 
3. Call thumb-generator to illustrate it.
4. Download the illustration to a file at `${video-title}/thumb.png`.

---
name: mkdir
tools: sys.write
description: Creates a folder in your system
args: dir: Path to the folder to be created.

#!bash

mkdir -p "${dir}"

---
name: thumb-generator
tools: github.com/gptscript-ai/image-generation
description: Generates a YouTube thumbnail.
args: script: The script to generate thumbnail from.

Do the following steps in acsending order: # New

1. Come up with a background color to represent the $script which can be used as the background color for the thumbnail. # New
2. Think of a good prompt to generate an image to represent $script. Make sure to to include the ${video-title} of the video as one sentence. 
inside a colored box somewhere in the thumbnail. Only return the URL of the illustration. The thumbnail should be 1792x1024. # New
3. Use the ${background-color} to make sure the edges of the thumbnails fades out. # New
```

All the changes here was made in the “thumb-generator” tool.  
这里的所有更改都是在 “thumb-generator” 工具中完成的。

To make the image generator create something that looks more like a YouTube thumbnail, we need to instruct to make the image a different size (1792×1024).  
要使图像生成器创建看起来更像 YouTube 缩略图的内容，我们需要指示将图像设置为不同的大小 （1792×1024）。

We also did a few other changes like finding a suitable background color, adding the title of the video as text somewhere on the thumbnail and added a fading background color.  
我们还做了一些其他更改，例如找到合适的背景颜色，在缩略图的某处将视频标题添加为文本，并添加淡化的背景颜色。

When I ran the command one more time, the thumbnail looked better, but it still was shaped as a square. So I changed step 2 in the “thumb-generator” to be like this:  
当我再次运行该命令时，缩略图看起来更好了，但它的形状仍然是一个正方形。因此，我将“thumb-generator”中的步骤 2 更改为如下：

```json
2. Think of a good prompt to generate an image to represent $script. Make sure to to include the ${video-title} of the video as one sentence. 
inside a colored box somewhere in the thumbnail. Only return the URL of the illustration. Make sure the thumbnail is 1792x1024 
```

This time I was a little bit more specific when I told the script the size of the thumbnail, and here is the result:  
这一次，当我告诉脚本缩略图的大小时，我更具体了一点，结果如下：  
![thumb2.jpg](https://necessary-creativity-bc071c3082.media.strapiapp.com/thumb2_4c80e654b3.jpg)

The result looks much better now. A little catch is the missing text, but if you run the script a few times, you will see that sometimes it’s there and sometimes it’s not. You can try changing the script a little bit to me more strict for example. But in general, adding text to images using DALL-E isn’t always the best solution.  
现在结果看起来好多了。一个小问题是缺少文本，但如果您运行脚本几次，您会发现有时它存在，有时不存在。例如，您可以尝试将脚本更改为更严格的格式。但一般来说，使用 DALL-E 向图像添加文本并不总是最佳解决方案。

## Summary  总结

That was it for part one. You should now have a basic understanding of what GPTScript is and what it can do. By learning these basics, you have also now built a simple script that can be used for generating images for a YouTube channel, but it can also be used for other things like blog posts and similar.  
第一部分就是这样。您现在应该对 GPTScript 是什么以及它可以做什么有了基本的了解。通过学习这些基础知识，您现在还构建了一个简单的脚本，该脚本可用于为 YouTube 频道生成图像，但它也可以用于其他内容，例如博客文章等。

In the next part of this tutorial, we will make the script even more advanced and also learn how to implement this with a frontend.  
在本教程的下一部分，我们将使脚本更加高级，并学习如何使用前端实现它。







构建 AI 驱动的 YouTube 标题和缩略图生成器（第 2 部分）

In this part of the tutorial, we are going to build a front end for the generator. We are also going to introduce a little python script to make it possible to add much better text to the thumbnails.  
在本教程的这一部分，我们将为生成器构建一个前端。我们还将介绍一个小 python 脚本，以便为缩略图添加更好的文本。

In the [previous part](https://www.acorn.io/resources/tutorials/building-an-ai-powered-you-tube-title-and-thumbnail-generator-part-1), you learned how to set up everything, define tools, run the script, etc.  
在[上一部分中](https://www.acorn.io/resources/tutorials/building-an-ai-powered-you-tube-title-and-thumbnail-generator-part-1)，您学习了如何设置所有内容、定义工具、运行脚本等。

## Prerequisites  先决条件

-Finished part one of this tutorial.  
- 完成本教程的第一部分。

-Python3.  -Python3 的。

-A “ttf” font. You can download one for free from Google Fonts.  
- 一个“ttf”字体。您可以从 Google Fonts 免费下载一个。

## Modifying the script  修改脚本

First, I want to do some modifications to the script.  
首先，我想对脚本进行一些修改。

```yaml
tools: thumb-generator, sys.write, sys.download # 1 - Changed
description: Generates a YouTube title and thumbnail based on a script.
args: theme: An overall characteristic description of the theme for the thumbnail. # 2 - Added
args: script: The script to generate title and thumbnail for. 

Do the following steps in acsending order:

1. Come up with an appropriate title for the video based on the script. Store the title in a file called `title.txt`. # 3 - Changed
2. Call thumb-generator to illustrate it.
3. Download the illustration to a file at `thumb.png`. # 4 - Changed

---
name: thumb-generator
tools: github.com/gptscript-ai/image-generation
description: Generates a YouTube thumbnail.
args: theme: An overall characteristic description of the theme for the thumbnail. # 5 - Added
args: script: The script to generate thumbnail from. 

Do the following steps in acsending order:

# All of the 4 steps has been changed

1. Based on the theme and the script, come up with a background color to represent the script which can be used as the background color for the thumbnail.
2. Do not include any text in the thumbnail.
3. Think of a good prompt to generate an image to represent script and the theme. Only return the URL of the illustration. The thumbnail should be 1792x1024.
4. Use the background color to make sure the edges of the thumbnails fades out.
```

As, you can see, there has been a few changes since last time. Let’s break it down a little bit:  
正如你所看到的，自上次以来已经发生了一些变化。让我们稍微分解一下：

1. We remove the ‘mkdir’ script. We are not longer creating a folder, so we can just remove it from the tools list (And you can remove the whole tool from the script).  
    我们删除了 'mkdir' 脚本。我们不再创建文件夹，因此我们可以将其从工具列表中删除（您可以从脚本中删除整个工具）。
2. We added a new “theme” argument. This theme is something we can pick in the frontend we are building. This can for example be “happy”, or “dark”.  
    我们添加了一个新的 “theme” 参数。这个主题是我们可以在我们正在构建的前端中选择的东西。例如，它可以是 “happy” 或 “dark”。
3. Instead of creating a folder with the name of the video, we now just store the title in a file called “title.txt”. This will be used later for showing the title in the frontend.  
    我们现在不用创建一个带有视频名称的文件夹，而是将标题存储在一个名为 “title.txt” 的文件中。这将用于稍后在前端显示标题。
4. Instead of storing the thumbnail in a sub folder with the title name, we just store it in a file called ‘thumb.png’. This makes it very easy to access from the frontend.  
    我们不是将缩略图存储在带有标题名称的子文件夹中，而是将其存储在名为“thumb.png”的文件中。这使得从前端访问变得非常容易。
5. Added the “theme” argument here as well.  
    这里也添加了 “theme” 参数。
6. We have changed almost all of the four steps here. Mainly just to tell the script to use both the theme and the script. Plus, we instruct it not to include any text in the image since this is something we want to do later.  
    我们几乎更改了这里的四个步骤。主要只是告诉脚本同时使用主题和脚本。另外，我们指示它不要在图像中包含任何文本，因为这是我们稍后要做的事情。

Feel free to run the script a see check out the result. To run this now with both arguments, the command can look something like this:  
随意运行脚本并查看结果。要现在使用两个参数运行此脚本，命令可以如下所示：

```yaml
gptscript generator.gpt --theme happy --script Hey, learn how to build a website for a restaurant by using only HTML and CSS.
```

## Setting up a frontend  设置前端

The frontend will be a Flask app (Flask is a framework based on Python for building web apps).  
前端将是一个 Flask 应用程序（Flask 是一个基于 Python 的框架，用于构建 Web 应用程序）。

In the same folder as your script file, create a new file called “requirements.txt”. It should look like this:  
在脚本文件所在的文件夹中，创建一个名为 “requirements.txt” 的新文件。它应如下所示：

```yaml
Flask==2.0.1
pillow==10.3.0
Werkzeug==2.2.2
```

1. Flask = The framework for building web apps.  
    Flask = 用于构建 Web 应用程序的框架。
2. pillow = A library for working with images.  
    pillow = 用于处理图像的库。
3. Wekzeug = A library for handling urls and endpoints in Flask.  
    Wekzeug = 用于在 Flask 中处理 url 和端点的库。

This requirments.txt file makes it really easy to install the same version as I used when I created this tutorial. To install these three libraries, you can run this command:  
这个 requirments.txt 文件使安装与我创建本教程时使用的相同版本变得非常容易。要安装这三个库，您可以运行以下命令：

```yaml
pip install -r requirements.txt
```

If you don’t have pip installed or you want to keep all of this inside a virtual environment, you can create a virtual environment first by running these two commands:  
如果您没有安装 pip 或希望将所有这些内容保存在虚拟环境中，您可以先通过运行以下两个命令来创建一个虚拟环境：

```yaml
python3 -m venv env
source evn/bin/activate
```

When this is finished, the frontend is ready to be built.  
完成此作后，即可构建前端。

## Setting up the templates  设置模板

Let’s begin by creating a folder in the root folder called templates. We are going to need two templates. First, let’s create one for the front page of our app. Create a file called “index.html” inside the templates [folder.](http://folder.It) It should look like this:  
让我们首先在根文件夹中创建一个名为 templates 的文件夹。我们将需要两个模板。首先，让我们为应用程序的首页创建一个。在 templates 文件夹中创建一个名为 “index.html” 的文件[。](http://folder.It)它应该看起来像这样：

```html
<!doctype html>
<html>
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <script src="https://cdn.tailwindcss.com"></script>
    </head>
    <body class="bg-slate-600">
        <nav class="py-8 px-12 bg-slate-900">
            <a href="/" class="text-2xl text-white">GPT-Thumb</a>
        </nav>

        <main class="py-8 px-12">
            <form method="post" action="/generate" class="space-y-8">
                <div>
                    <label class="text-lg text-white">Script</label><br>

                    <textarea name="script" id="script" class="w-full h-80 py-4 px-8 bg-slate-400 rounded-xl"></textarea>
                </div>

                <div>
                    <label class="text-lg text-white">Color theme</label><br>

                    <select name="theme" id="theme" class="w-full py-4 px-8 bg-slate-400 rounded-xl">
                        <option value="happy" selected>Happy</option>
                        <option value="dark">Dark</option>
                        <option value="sad">Sad</option>
                        <option value="futuristic">Futuristic</option>
                    </select>
                </div>

                <button id="generate" class="py-4 px-8 bg-sky-600 text-white rounded-xl">Generate</button>
            </form>
        </main>
    </body>
</html>
```

As you can see, it’s not a very complicated frontend, but does the job.  
如您所见，它不是一个非常复杂的前端，但可以完成工作。

To make it possible to build the design as rapidly as possible, I have included Tailwind CSS as a CDN. For this design, we have a simple menu at the top, and then a form where you can fill out the script for the video and select a theme you want the thumbnail to be based on.  
为了能够尽快构建设计，我将 Tailwind CSS 作为 CDN 包含在内。对于此设计，我们在顶部有一个简单的菜单，然后是一个表单，您可以在其中填写视频脚本并选择您希望缩略图基于的主题。

Okay, next, we are going to create one more file called “result.html” which will be used to present the title and thumbnail when it’s ready. It should look something like this:  
好的，接下来，我们将再创建一个名为 “result.html” 的文件，当它准备好时，它将用于显示标题和缩略图。它应该看起来像这样：

```html
<!doctype html>
<html>
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <script src="https://cdn.tailwindcss.com"></script>
    </head>
    <body class="bg-slate-600">
        <nav class="py-8 px-12 bg-slate-900">
            <a href="/" class="text-2xl text-white">GPT-Thumb</a>
        </nav>

        <main class="py-8 px-12">
            <h1 class="mb-12 text-3xl text-white">Result</h1>

            <h2 class="mb-2 text-2xl text-white">YouTube title:</h2>

            <h3 class="mb-6 text-xl text-white">{{ title }}</h3>

            <h2 class="mb-2 text-2xl text-white">YouTube thumbnail:</h2>

            <img src="./thumb.png" />
        </main>
    </body>
</html>
```

Again, this is a very simple template. The big change here is that we use a couple of flask tags to show the value from the “title”, and then we just statically show an image called thumb.png. We know that this will be the same name every time.  
同样，这是一个非常简单的模板。这里最大的变化是，我们使用几个 flask 标签来显示 “title” 中的值，然后我们只是静态显示一个名为 thumb.png 的图像。我们知道每次都会使用相同的名称。

## Showing the front page  显示首页

Let’s create a new file to make Flask present the “index.html” template we just created. In the root folder of our project, create a new file called “app.py”. It should look like this:  
让我们创建一个新文件，让 Flask 呈现我们刚刚创建的 “index.html” 模板。在我们项目的根文件夹中，创建一个名为 “app.py” 的新文件。它应该看起来像这样：

```python
import subprocess
import os

from flask import Flask, request, render_template, send_file, redirect
from PIL import Image, ImageDraw, ImageFont
from textwrap import wrap

app = Flask(__name__)

base_dir = os.path.dirname(os.path.abspath(__file__))

SCRIPT_PATH = os.path.join(base_dir, 'generator.gpt')

@app.route('/', methods=['GET'])
def index():
    return render_template('index.html')

if __name__ == '__main__':
    app.run(debug=False)
```

If you’re new to Flask or Python, you might notice a few new things here. Let’s go through this file:  
如果你是 Flask 或 Python 的新手，你可能会注意到这里的一些新内容。让我们来看看这个文件：

-First, we import os and subprocess. These are two modules that comes built in from Python. We use “os” to read file and similar from the disk. “subprocess” is a module we need to make it possible to run the “generator.gpt” script we have created.  
- 首先，我们导入 os 和 subprocess。这是 Python 内置的两个模块。我们使用“os”从磁盘读取文件等。“subprocess”是我们需要的一个模块，它可以运行我们创建的“generator.gpt”脚本。

-Next, we import Flask, PIL and textwrap.  
- 接下来，我们导入 Flask、PIL 和 textwrap。

-Flask and the other modules on the same line are things we need to run this app. We’ll come back to these later.  
-Flask 和同一行上的其他模块是我们运行此应用程序所需的内容。我们稍后会回到这些。

-PIL is the “pillow” library we installed. This is used to read images, write text and draw.  
-PIL 是我们安装的“枕头”库。它用于读取图像、写入文本和绘图。

-textdraw is used to help us figuring out the size of the text we’re drawing on the thumbnail.  
-textdraw 用于帮助我们确定在缩略图上绘制的文本的大小。

-Then, we set up a Flask app, define the base root of our app and create a variable with the absolute path for the “generator.gpt” script.  
- 然后，我们设置一个 Flask 应用程序，定义我们应用程序的基本根，并使用“generator.gpt”脚本的绝对路径创建一个变量。

-After all the config, we can set up a route. This is used to render the index.html template. Notice that Flask automatically looks for a folder called “templates” and find templates inside there.  
- 完成所有配置后，我们可以设置一个路由。这用于渲染 index.html 模板。请注意，Flask 会自动查找一个名为 “templates” 的文件夹，并在其中找到模板。

-The last two lines is there for making the app run.  
- 最后两行用于使应用程序运行。

If you now go to the terminal, we can start the Flask app by running this command:  
如果您现在转到终端，我们可以通过运行以下命令来启动 Flask 应用程序：

```bash
$ python3 app.py
```

Next, head to the browser, open this url “[http://127.0.0.1:5000](http://127.0.0.1:5000/)” and you should see something like this:  
接下来，前往浏览器，打开这个 url “[http://127.0.0.1:5000](http://127.0.0.1:5000/)”，你应该会看到如下内容：

![[Skjermbilde_2024_04_25_kl_08_50_31_7025ced4e2.png]]

## Creating some helper functions  
创建一些帮助程序函数

To make the code as neat and readable as possible, we need a couple of functions to do some jobs for us. Between the “SCRIPT_PATH” variable the the index-route, add this code:  
为了使代码尽可能整洁和可读，我们需要几个函数来为我们做一些工作。在 “SCRIPT_PATH” 变量和 index-route 之间，添加以下代码：

```python
...
SCRIPT_PATH = os.path.join(base_dir, 'generator.gpt')

def get_y_and_heights(text_wrapped, dimensions, margin, font):
    ascent, descent = font.getmetrics()

    line_heights = [
        font.getmask(text_line).getbbox()[3] + descent + margin
        for text_line in text_wrapped
    ]

    line_heights[-1] -= margin
    height_text = sum(line_heights)
    y = (dimensions[1] - height_text) // 2
    
    return (y, line_heights)

def get_title():
    titlefile = open('./title.txt','r')
    title = titlefile.read()
    titlefile.close()
    return title

@app.route('/thumb.png')
def thumb_png():
    return send_file('./thumb.png', mimetype='image/png')

@app.route('/', methods=['GET'])
def index():
    return render_template('index.html')
...
```

I will go through each of these functions.  
我将介绍这些功能中的每一个。

**get_y_and_heights**

This function is helping us figuring out how many lines the text will be, and how tall it is. The “y” variable contains the point on our thumbnail where the first part of the text will be located, and the “line_heights” contains information about how tall each of the lines are.  
这个函数可以帮助我们弄清楚文本将有多少行，以及它有多高。“y” 变量包含缩略图上文本第一部分所在的点，“line_heights” 包含有关每行高度的信息。

**get_title**

This function is a little bit simple. This is just used to read the file called “title.txt” and return the value so it can be use elsewhere.  
这个函数有点简单。这只是用于读取名为 “title.txt” 的文件并返回值，以便可以在其他地方使用。

**@app.router(’/thumb.png’)**

Flask it not really intended to show static files or other types of media files. So we need to create a separate route for this exact purpose. If you now have an image called “thumb.png” in your roor folder, you should be able to visit “http://127.0.0.1:5000/thumb.png” in your browser. You might need to stop and start the Flask server for the changes to take effect.  
Flask 并不真正打算显示静态文件或其他类型的媒体文件。因此，我们需要为此创建一个单独的路由。如果你现在的 roor 文件夹中有一个名为 “thumb.png” 的图像，你应该能够在浏览器中访问 “http://127.0.0.1:5000/thumb.png”。你可能需要停止和启动 Flask 服务器才能使更改生效。

## The generator route  生成器路由

Okay, now that we have the templates and helper functions, we can create the route that will handle the information from the form and run the actual generator script.  
好了，现在我们已经有了模板和帮助程序函数，我们可以创建将处理表单中信息的路由并运行实际的生成器脚本。

Below the route for the thumb, add this code:  
在 thumb 的路线下方，添加以下代码：

```python
@app.route('/generate', methods=['POST'])
def generate():
    script = request.form.get('script')
    theme = request.form.get('theme')

    subprocess.Popen(f"gptscript {SCRIPT_PATH} --theme {theme} --script {script}", shell=True, stdout=subprocess.PIPE).stdout.read()

    font = ImageFont.truetype("Roboto-Black.ttf", 100) # 100 = font size
    img = Image.open("thumb.png")
    draw_interface = ImageDraw.Draw(img, "RGBA")

    title = get_title()

    text_lines = wrap(title, 25) # Number of chars per line

    y, line_heights = get_y_and_heights(
        text_lines,
        (img.width, img.height),
        30, # 30 = Line height
        font
    )   

    yStart = (img.height / 2) - (y / 2) - 100
    xStart = (img.width / 2) 
    shape = [(xStart - 700, yStart), (xStart + 700, yStart + y + 100)] 

    draw_interface.rectangle((shape), fill=(0, 0, 0, 150)) # Background color for rectangle

    # Draw each line of text
    for i, line in enumerate(text_lines):
        line_width = font.getmask(line).getbbox()[2]
        x = ((img.width - line_width) // 2)
        draw_interface.text((x, y), line, font=font, fill="white") # Fill = Text color
        y += line_heights[i]

    # Save the resulting image
    img.save("thumb.png")

    return redirect('/result')
```

I have added a few comments in this script to help you understand the values I have used.  
我在此脚本中添加了一些注释，以帮助您了解我使用的值。

Let’s also break this script down a little bit:  
我们还将这个脚本稍微分解一下：

-First we get the values from the form.  
- 首先，我们从表单中获取值。

-Next, we use the “subprocess” library we imported to run the generator.gpt script. As you can see, we also pass in the theme and the script. This might take a little while to run.  
- 接下来，我们使用导入的 “subprocess” 库来运行 generator.gpt 脚本。如您所见，我们还传入了主题和脚本。这可能需要一点时间才能运行。

-Next, we set up a font to use (Change to the one you have downloaded).  
- 接下来，我们设置要使用的字体（更改为您下载的字体）。

-We use “Image” from Pillow (PIL) to read the image from the disk and then set up a “draw_interface”. The draw_interface is used to draw texts and rectangles on an existing image.  
- 我们使用 Pillow 的 “Image” （PIL） 从磁盘中读取图像，然后设置一个 “draw_interface”。draw_interface 用于在现有图像上绘制文本和矩形。

-Then we get the title from the text file and use the “wrap” function to calculate how many lines the text will be.  
- 然后我们从文本文件中获取标题，并使用 “wrap” 函数计算文本将有多少行。

-Next, we get the y and line_height values from the function we created.  
- 接下来，我们从创建的函数中获取 y 和 line_height 值。

-The next “yStart” and “xStart” is a different calculation we need in order to figure out how big the “square” behind the text should be, and where to place it.  
- 下一个 “yStart” 和 “xStart” 是我们需要的不同计算，以便计算出文本后面的 “square” 应该有多大，以及将其放置在哪里。

-Next, we use the draw_interface to draw the rectangle on the image.  
- 接下来，我们使用 draw_interface 在图像上绘制矩形。

-Then, we loop though the lines of text and draw it on the thumbnail.  
- 然后，我们遍历文本行并将其绘制在缩略图上。

-Last but not least, we save the thumbnail and redirects us to the result page.  
- 最后但并非最不重要的一点是，我们保存缩略图并将我们重定向到结果页面。

If you test this in the browser now, the thumbnail will be generated. But since we don’t have a result page yet, you will get an error.  
如果您现在在浏览器中测试此内容，将生成缩略图。但由于我们还没有结果页面，因此您将收到错误。

## Creating the result page  创建结果页面

Below the route we just created, add the following code:  
在我们刚刚创建的路由下方，添加以下代码：

```python
@app.route('/result', methods=['GET'])
def result():
    return render_template('result.html', title=get_title())
```

One last, but simple route. This renders the “result.html” template we created earlier, and pass in the title of the video so it can be shown.  
最后一条但简单的路线。这将呈现我们之前创建的 “result.html” 模板，并传入视频的标题以便显示它。

If you run everything now, you should see something like this:  
如果现在运行所有内容，您应该会看到如下内容：  

![[Skjermbilde_2024_04_25_kl_09_08_10_ea5e127ced.png]]

## Summary  总结

And that was it for this tutorial. I hope that you know have a better understanding of how things with GPTScript work. You should now also know a little bit about how to build a frontend for it and similar.  
这就是本教程的内容。我希望您知道对 GPTScript 的工作原理有更好的了解。您现在还应该了解一些关于如何为它构建前端和类似内容的知识。

A few suggestions for further improvements:  
进一步改进的一些建议：  
-When you have the results. Make it possible to “regenerate”. Maybe with comments about what you want to change.  
- 当你有结果时。使“再生”成为可能。也许带有关于您想要更改的内容的评论。  
-Add more themes.  - 添加更多主题。  
-Store the title and images in a database.  
- 将标题和图像存储在数据库中。

