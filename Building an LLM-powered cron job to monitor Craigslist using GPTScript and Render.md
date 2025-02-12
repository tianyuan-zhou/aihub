构建一个LLM由 -Cron 驱动的作业，以使用 GPTScript 和 Render 监控 Craigslist

###### March 27, 2024 by grant linville  

If you are not familiar with it already, [GPTScript](https://github.com/gptscript-ai/gptscript) is a new scripting language to automate interactions with large language models, such as OpenAI’s GPT-4. With one simple script, you can give the LLM the ability to perform tasks on your computer by giving it the ability to use tools. GPTScript is also useful on servers and in containers, and can perform advanced tasks without requiring programming. In this tutorial, I will walk you through the process of setting up a containerized GPTScript and running it as a cron job in [Render](https://render.com).  
如果您还不熟悉它，[GPTScript](https://github.com/gptscript-ai/gptscript) 是一种新的脚本语言，可以自动与大型语言模型（例如 OpenAI 的 GPT-4）进行交互。通过一个简单的脚本，您可以通过赋予计算机使用工具的能力来赋予LLM在计算机上执行任务的能力。GPTScript 在服务器和容器中也很有用，并且无需编程即可执行高级任务。在本教程中，我将引导您完成设置容器化 GPTScript 并将其作为 cron 作业在 [Render](https://render.com) 中运行的过程。

The job will check Craigslist for new Toyota 4Runners listed for sale in the Phoenix, Arizona area. It will send an email with links and information about any new vehicles that it has not already sent an email about. The goal is to accomplish this without having to write any code, other than the GPTScript and a Dockerfile.  
该工作将在 Craigslist 上查看在亚利桑那州凤凰城地区出售的新丰田 4Runner。它将发送一封电子邮件，其中包含有关尚未发送电子邮件的任何新车的链接和信息。目标是在无需编写除 GPTScript 和 Dockerfile 之外的任何代码的情况下完成此作。

### Prerequisites  先决条件

There are a few things you will need in order to complete this guide:  
要完成本指南，您需要满足以下条件：

- An OpenAI account, with your API key set to the environment variable OPENAI_API_KEY  
    一个 OpenAI 账户，您的 API 密钥设置为环境变量 OPENAI_API_KEY
- A [Render](https://render.com) account  
    [一个 Render](https://render.com) 帐户
- A [SendGrid](https://sendgrid.com) account  
    [一个 SendGrid](https://sendgrid.com) 帐户
- Docker  码头工人
- The ability to push container images to a container registry, such as Docker Hub  
    将容器镜像推送到容器注册表（如 Docker Hub）的能力

### Creating the script  创建脚本

There are two main tasks that our GPTScript needs to accomplish: getting the vehicle information from Craigslist, and sending the email. Let’s start with the Craigslist portion.  
我们的 GPTScript 需要完成两项主要任务：从 Craigslist 获取车辆信息，并发送电子邮件。让我们从 Craigslist 部分开始。

The built-in tool "sys.http.html2text" is useful in circumstances where we want the LLM to fetch a webpage and process its contents in some way. Let’s start with this:  
内置工具 “sys.http.html2text” 在我们希望 LLM 获取网页并以某种方式处理其内容的情况下很有用。让我们从这个开始：

```json
tools: sys.http.html2text

Visit https://phoenix.craigslist.org/search/cta?postedToday=1&query=4runner&sort=date to get information about the
4Runners listed today.
```

Save that to a file called "4runner.gpt". Now let’s run it and see if it works:  
将其保存到名为 “4runner.gpt” 的文件中。现在让我们运行它，看看它是否有效：

`gptscript 4runner.gpt`

This was my output:  这是我的输出：

```json
There are two 4Runners listed today in Phoenix, AZ on Craigslist:

1. **2017 Toyota 4Runner TRD Off Road 4x4**
   - Price: $39,750
   - Location: I-17 & 101
   - [Link to listing](https://phoenix.craigslist.org/nph/cto/d/phoenix-2017-toyota-4runner-trd-off/7729111421.html)

2. **2011 Toyota 4Runner SR5**
   - Price: $21,500
   - Location: Mesa
   - [Link to listing](https://phoenix.craigslist.org/evl/ctd/d/mesa-2011-toyota-4runner-sr5/7729001129.html)
```

(Your output will look different, depending on the day, and how many 4Runners were listed.)  
（您的输出看起来会有所不同，具体取决于日期以及列出的 4Runner 数量。

This looks good. But we need a way of keeping track of which vehicles we have already sent an email about, so that we do not include them in future emails. We will need to use a database for this. Render offers PostgreSQL databases that are easy to set up, so we will go with that.  
这看起来不错。但是我们需要一种方法来跟踪我们已经发送电子邮件的车辆，这样我们就不会在将来的电子邮件中包含它们。为此，我们需要使用数据库。Render 提供易于设置的 PostgreSQL 数据库，因此我们将使用它。

Let’s add some more instructions to the script:  
让我们向脚本添加更多说明：

```json
tools: sys.http.html2text, sys.exec?

Visit https://phoenix.craigslist.org/search/cta?postedToday=1&query=4runner&sort=date to get information about the
4Runners listed today.

Check the PostgreSQL database and see if any of these vehicles have already been stored. If they have been, ignore them.
The `psql` command is available. The database connection string is in the environment variable PGURL. The table is
called "vehicles". If the table doesn't exist yet, create it. The only thing that needs to be stored in the table
is the URL for each vehicle. Don't add any other fields.

For each vehicle that was not already in the database, store the information about it into the database.
```

Here, we are telling the LLM about our database. It knows that it can find the connection string in the PGURL environment variable, and that it has the psql command available to interact with the database. We have also instructed it to only store the URL for each car, so it will create a very simple schema for the database table.  
在这里，我们介绍LLM我们的数据库。它知道它可以在 PGURL 环境变量中找到连接字符串，并且它有可用于与数据库交互的 psql 命令。我们还指示它只存储每辆车的 URL，因此它将为数据库表创建一个非常简单的架构。

We also gave it access to the sys.exec tool so that it can run commands in the container. The "?" indicates that the script should continue to run even if the sys.exec tool returns an error. This is necessary in case the LLM tries to add an entry to the vehicles table, only to discover that it does not exist and needs to create it. It might get an error back the first time it tries something, but it will be able to recover from that state.  
我们还授予它对 sys.exec 工具的访问权限，以便它可以在容器中运行命令。“？” 表示即使 sys.exec 工具返回错误，脚本也应继续运行。如果 尝试LLM向 vehicles 表添加条目，却发现该条目不存在并且需要创建它，则这是必需的。它可能会在第一次尝试某些作时返回错误，但它将能够从该状态中恢复。

We aren’t done with the script yet, since we have not given it the ability to send emails, but we will handle that part later. Next, we are going to set up the database in Render.  
我们还没有完成脚本，因为我们还没有赋予它发送电子邮件的能力，但我们稍后会处理这部分。接下来，我们将在 Render 中设置数据库。

### Creating the Database  创建数据库

Inside of your Render dashboard, create a new project called "4Runner". Then, select "Create new service" and then "New PostgreSQL". Set the Name to `4runner", the Database to "vehicles", the User to "user", and select whichever region you prefer. Leave PostgreSQL Version set to 16. Your configuration should look similar to this:  
在 Render 仪表板中，创建一个名为“4Runner”的新项目。然后，选择“Create new service”，然后选择“New PostgreSQL”。将 Name（名称）设置为 '4runner），将 Database（数据库）设置为 “vehicles”，将 User （用户） 设置为 “user”，然后选择你喜欢的任何区域。将 PostgreSQL Version （PostgreSQL 版本） 设置为 16。您的配置应类似于：

![Render New PostgreSQL](https://necessary-creativity-bc071c3082.media.strapiapp.com/Render_New_Postgre_SQL_be9f1e4907.png)

Scroll down a bit and select the free tier:  
向下滚动一点并选择免费套餐：

![Render PostgreSQL Free Tier](https://necessary-creativity-bc071c3082.media.strapiapp.com/Render_Postgre_SQL_Free_Tier_84e91e1f58.png)

Then, click the "Create Database" button at the bottom of the page. The database may take a couple minutes to create. Once it is ready, you will see some URLs when you scroll down to the Connections section:  
然后，单击页面底部的“创建数据库”按钮。创建数据库可能需要几分钟时间。准备就绪后，当您向下滚动到“连接”部分时，您将看到一些 URL：

![Render Database Connections](https://necessary-creativity-bc071c3082.media.strapiapp.com/Render_Database_Connections_64a07a9b4e.png)

Click the copy button on the "External Database URL". Save this to the environment variable PGURL in your terminal. Now, we are ready to create the container and run it locally.  
单击 “External Database URL” 上的复制按钮。将其保存到终端中的环境变量 PGURL 中。现在，我们已准备好创建容器并在本地运行它。

### Creating the Dockerfile  创建 Dockerfile

Here is the Dockerfile we are going to use:  
以下是我们将要使用的 Dockerfile：

```dockerfile
FROM alpine:latest

COPY 4runner.gpt ./4runner.gpt

RUN apk update && apk add curl postgresql16-client --no-cache && curl https://get.gptscript.ai/install.sh | sh && mkdir /.cache && chmod 777 /.cache

CMD ["gptscript", "--cache=false", "4runner.gpt"]
```

This image is based on Alpine Linux, which is commonly used in containers. We use Alpine’s package manager, apk, to install curl and the PostgreSQL 16 client (the psql command). We then use curl to install GPTScript via the install script. We also set up the .cache directory and modify its permissions so that GPTScript is able to use it without getting any permissions errors.  
此映像基于容器中常用的 Alpine Linux。我们使用 Alpine 的包管理器 apk 来安装 curl 和 PostgreSQL 16 客户端（psql 命令）。然后，我们使用 curl 通过安装脚本安装 GPTScript。我们还设置了 .cache 目录并修改其权限，以便 GPTScript 能够在不出现任何权限错误的情况下使用它。

Next, let’s build and run the image locally.  
接下来，让我们在本地构建并运行映像。

```bash
docker build -t 4runner .
docker run --rm -e PGURL=$PGURL -e OPENAI_API_KEY=$OPENAI_API_KEY 4runner
```

This was my output:  这是我的输出：

```json
All the 4Runners listed today have been successfully stored in the database.
```

Let’s run the container again, but this time create a shell in it so that we can manually verify that the URLs were added to the database:  
让我们再次运行容器，但这次在其中创建一个 shell，以便我们可以手动验证 URL 是否已添加到数据库中：

`docker run --rm -it --entrypoint sh -e PGURL=$PGURL -e OPENAI_API_KEY=$OPENAI_API_KEY 4runner`

Once you are in the container, run:  
进入容器后，运行：

`psql $PGURL -c "SELECT * FROM vehicles;"`

This was my output:  这是我的输出：

```json
                                             url                                              
----------------------------------------------------------------------------------------------
 https://phoenix.craigslist.org/nph/cto/d/phoenix-2017-toyota-4runner-trd-off/7729111421.html
 https://phoenix.craigslist.org/evl/ctd/d/mesa-2011-toyota-4runner-sr5/7729001129.html
(2 rows)
```

Looks good! Now, let’s delete these so that we can try it out with the email later:  
看起来不错！现在，让我们删除这些，以便我们稍后可以尝试使用电子邮件进行尝试：

`psql $PGURL -c "DROP TABLE vehicles;"`

Now, run "exit" to leave the container. Next, we will set up SendGrid so that we can get an API key and start sending emails.  
现在，运行 “exit” 离开容器。接下来，我们将设置 SendGrid，以便我们可以获取 API 密钥并开始发送电子邮件。

### Setting up SendGrid  设置 SendGrid

Create an account at sendgrid.com if you do not already have one. Once you are logged in, go to Settings -> Tracking in the sidebar.  
如果您还没有帐户，请在 sendgrid.com 创建一个帐户。登录后，转到侧边栏中的设置 -> 跟踪。

![SendGrid Tracking Settings](https://necessary-creativity-bc071c3082.media.strapiapp.com/Send_Grid_Tracking_Settings_1fb84a982b.png)

I disabled all of these settings on my account. Tweak them to your own preferences, but I recommend at least disabling Click Tracking so that the links the emails that get sent will look normal.  
我在我的帐户上禁用了所有这些设置。根据您自己的喜好调整它们，但我建议至少禁用点击跟踪，以便发送的电子邮件中的链接看起来正常。

Once you are done, go to Settings -> API Keys, also in the sidebar. Click "Create API Key" in the top right corner. Name your key "4runner" and give it Restricted Access. Leave each setting at No Access, except for Mail Send, which should get Full Access. It should look like this:  
完成后，转到侧边栏中的设置 -> API 密钥。单击右上角的“创建 API 密钥”。将您的密钥命名为“4runner”，并为其提供受限访问权限。将每个设置保留为“无访问权限”，但邮件发送除外，它应该获得完全访问权限。它应该看起来像这样：

![SendGrid API Key Settings](https://necessary-creativity-bc071c3082.media.strapiapp.com/Send_Grid_API_Key_Settings_c64ffac8c9.png)

Click "Create & View" to get your API key. Save it to the environment variable SENDGRID_API_KEY.  
单击“创建和查看”以获取您的 API 密钥。将其保存到环境变量SENDGRID_API_KEY。

The last thing we need to do in SendGrid is set up sender authentication. Select Settings -> Sender Authentication in the sidebar. You can set up an entire domain if you would like, but that is not necessary for this example, as you can just use your personal email. Click on "Verify Single Sender" and go through all the menus and prompts to get it set up. It will send you an email to verify your request. Once you are done, it is time to finish the GPTScript.  
我们需要在 SendGrid 中做的最后一件事是设置发件人身份验证。在侧边栏中选择设置 -> 发件人身份验证。如果您愿意，您可以设置整个域，但对于此示例来说，这不是必需的，因为您可以只使用您的个人电子邮件。单击“验证单个发件人”并浏览所有菜单和提示以进行设置。它将向您发送一封电子邮件以验证您的请求。完成后，就该完成 GPTScript 了。

### Finishing the script  完成脚本

Next, we will add email capabilities to the script. This is our final version of the script:  
接下来，我们将向脚本添加电子邮件功能。这是我们脚本的最终版本：

```json
tools: send_email, sys.http.html2text, sys.exec?

Visit https://phoenix.craigslist.org/search/cta?postedToday=1&query=4runner&sort=date to get information about the
4Runners listed today.

Check the PostgreSQL database and see if any of these vehicles have already been stored. If they have been, ignore them.
The `psql` command is available. The database connection string is in the environment variable PGURL. The table is
called "vehicles". If the table doesn't exist yet, create it. The only thing that needs to be stored in the table
is the URL for each vehicle. Don't add any other fields.

For each vehicle that was not already in the database, store the information about it into the database, and send
an email with information about those vehicles. If there are no new vehicles, then don't send an email.

---
name: send_email
description: sends an email with the provided contents
args: contents: the contents of the email to send
tools: sys.http.html2text, sys.exec?

IMPORTANT: when setting --header or -H on a cURL command, always use double quotes, never single quotes!
IMPORTANT: when setting --data or -d on a cURL command, always use single quotes, never double quotes! And always escape newlines! (They should look like "\n")

The SendGrid API key is in the environment variable SENDGRID_API_KEY.

Perform the following actions in this order:
1. View the contents of https://docs.sendgrid.com/for-developers/sending-email/api-getting-started to learn about the SendGrid API.
2. Run a cURL command to send an email using the SendGrid API, with the following information:
   to: <email address> (name: <name>)
   from: <email address> (name: <name>)
   reply to: <email address> (name: <name>)
   subject: "4Runner Listings"
   content: $contents
```

> **IMPORTANT**: be sure to fill in all instances of "" and "" with your actual email address and name.  
> **重要提示**：请务必在“”和“”的所有实例中填写您的实际电子邮件地址和姓名。

We added a new tool called send_email that will read the docs for the SendGrid API and then run a cURL command to send an email about the vehicles. (Due to various issues around quotes and environment variables, I had to add those "IMPORTANT" lines to tell the LLM how to not mess up the commands it creates.)  
我们添加了一个名为 send_email 的新工具，它将读取 SendGrid API 的文档，然后运行 cURL 命令来发送有关车辆的电子邮件。（由于引号和环境变量的各种问题，我不得不添加那些“重要”行来告诉如何LLM不弄乱它创建的命令。

We also slightly modified the prompt for the first tool to tell it to send the email, and we gave it access to the new send_email tool.  
我们还略微修改了第一个工具的提示，告诉它发送电子邮件，并授予它访问新 send_email 工具的权限。

With that, the script is finished. Let’s build and push the container image up to a container registry. Make sure it is built for the linux/amd64 platform. I used my personal Docker Hub for this.  
这样，脚本就完成了。让我们构建容器镜像并将其推送到容器注册表。确保它是为 linux/amd64 平台构建的。为此，我使用了我个人的 Docker Hub。

```bash
docker buildx build --push --platform linux/amd64 -t grantlinville/4runner .
```

Now the last thing to do is set up and run the cron job in Render.  
现在要做的最后一件事是在 Render 中设置并运行 cron 作业。

### Setting up the cron job  
设置 cron 作业

Go back to your 4runner project’s dashboard in Render. Click on the + icon, and then "Create New Service".  
在 Render 中返回 4runner 项目的仪表板。单击 + 图标，然后单击“创建新服务”。

![Render Create New Service](https://necessary-creativity-bc071c3082.media.strapiapp.com/Render_Create_New_Service_1378b55ae8.png)

Click "New Cron Job", and then select "Deploy an existing image from a registry" and click "Next".  
单击 “New Cron Job”，然后选择 “Deploy an existing image from a registry” 并单击 “Next”。

![Render New CronJob](https://necessary-creativity-bc071c3082.media.strapiapp.com/Render_New_Cron_Job_745896bc24.png)

In the Image URL field, type in the URL of your container image on whichever registry you uploaded it to. For me, it is "docker.io/grantlinville/4runner". Then click "Next".  
在 Image URL（图像 URL）字段中，键入容器图像的 URL，无论您将其上传到哪个注册表。对我来说，它是“docker.io/grantlinville/4runner”。然后单击“Next”（下一步）。

Name your cron job "4runner" and select whichever region you would like. For the schedule, I recommend setting it to "0 */12 * * *" so that it runs every 12 hours. Set the Instance Type to Starter. Then, add three environment variables. The first is SENDGRID_API_KEY with the value set to the key you got earlier. The second is PGURL, **but this time, set to the Internal Database URL**, instead of the External Database URL from the PostgreSQL database we created earlier. The third is OPENAI_API_KEY, with your key.  
将您的 cron 作业命名为“4runner”并选择您想要的任何区域。对于计划，我建议将其设置为“0 */12 * * *”，以便每 12 小时运行一次。将 Instance Type（实例类型）设置为 Starter（启动器）。然后，添加三个环境变量。第一个是 SENDGRID_API_KEY，其值设置为您之前获得的密钥。第二个是 PGURL，**但这次设置为 Internal Database URL（内部数据库 URL**），而不是我们之前创建的 PostgreSQL 数据库中的 External Database URL（外部数据库 URL）。第三个是 OPENAI_API_KEY，带有您的密钥。

![Render Internal Database URL](https://necessary-creativity-bc071c3082.media.strapiapp.com/Render_Internal_Database_URL_ebc9f5b856.png)

Your cron job settings should look like this:  
您的 cron 作业设置应如下所示：

![Render Cron Job](https://necessary-creativity-bc071c3082.media.strapiapp.com/Render_Cron_Job_508d3e8787.png)

Click "Create Cron Job". The job will not run right away, so in the top right, click the "Trigger Run" button. Once it is done running, you should see that it succeeded:  
点击 “Create Cron Job”。该作业不会立即运行，因此在右上角，单击 “Trigger Run” 按钮。运行完成后，您应该会看到它成功：

![Render Cron Job Succeeded](https://necessary-creativity-bc071c3082.media.strapiapp.com/Render_Cron_Job_Succeeded_002fba71ad.png)

Let’s check the logs:  让我们检查一下日志：

![Render Cron Job Logs](https://necessary-creativity-bc071c3082.media.strapiapp.com/Render_Cron_Job_Logs_29401a283b.png)

It says the email was sent successfully. I found the email in my junk/spam folder. This is probably because it is coming from my own email address, which seems a bit suspicious, but that is how I configured SendGrid since it was the easiest way. As you can see, there are links to three cars in the email:  
它说电子邮件已成功发送。我在我的垃圾邮件文件夹中找到了这封电子邮件。这可能是因为它来自我自己的电子邮件地址，这似乎有点可疑，但这就是我配置 SendGrid 的方式，因为这是最简单的方法。如您所见，电子邮件中有指向三辆汽车的链接：

![4Runner Email](https://necessary-creativity-bc071c3082.media.strapiapp.com/4_Runner_Email_725866da1c.png)

It looks like a new one was posted between when I ran the script myself at the beginning of this tutorial, and now at the end.  
看起来在本教程开始时我自己运行脚本到现在结束时之间发布了一个新的脚本。

To prove that the job will not send an email for cars that it has already notified about, trigger the cron job again and wait for it to finish. The job succeeded again, but looking at the logs, it did not send an email since there were no new listings:  
要证明该作业不会为已通知的汽车发送电子邮件，请再次触发 cron 作业并等待它完成。该作业再次成功，但查看日志，它没有发送电子邮件，因为没有新的列表：

![Render Cron Job Logs 2](https://necessary-creativity-bc071c3082.media.strapiapp.com/Render_Cron_Job_Logs_2_0bf4c37aef.png)

That brings us to the end of this tutorial. This cron job demonstrates the power of GPT-4 with GPTScript to automate advanced tasks without requiring programming.  
这将我们带到本教程的结尾。这个 cron 作业展示了 GPT-4 和 GPTScript 无需编程即可自动执行高级任务的能力。