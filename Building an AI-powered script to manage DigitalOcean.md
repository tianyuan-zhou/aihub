
构建 AI 驱动的脚本来管理 DigitalOcean

In this post we will illustrate the usage of OpenAI and GPTScript to run a recurring task on Digital Ocean. We will check every minute if a given URL is reachable and send the resulting http status code to a webhook.  
在这篇文章中，我们将说明如何使用 OpenAI 和 GPTScript 在 Digital Ocean 上运行重复任务。我们将每分钟检查一次给定的 URL 是否可访问，并将生成的 http 状态代码发送到 webhook。

## High level Instructions  高级说明

First we create the task.gpt text file containing the instructions we want GPTScript to send to the LLM. These instructions are the following ones:  
首先，我们创建 task.gpt 文本文件，其中包含我们希望 GPTScript 发送到 LLM.这些说明如下：

```json
Create a task which verifies every minute if the website WEBSITE_URL is reachable and sends the status code to the following HTTP POST request:
- URL is https://webhooks.app/data
- Authorization bearer is TOKEN
- A json payload must be returned, containing a "message" property with the value of the WEBSITE_URL and the status code returned
```

There are 2 parts in these instructions:  
这些说明包括 2 个部分：

- first we ask to check if a given url (WEBSITE_URL) is available  
    首先，我们要求检查给定的 URL （WEBSITE_URL） 是否可用
- then we ask to send the status code to a webhook and provide its characteristics  
    然后，我们要求将状态代码发送到 Webhook 并提供其特征

Note: the webhook URL [https://webhooks.app](https://webhooks.app) is a simple and free endpoint used for demo purposes. It allows to inspect the content of the payloads sent from various systems  
请注意： webhook URL [https://webhooks.app](https://webhooks.app) 是一个简单且免费的端点，用于演示目的。它允许检查从各种系统发送的有效负载的内容

Both WEBSITE_URL and TOKEN are placeholders that we want to set dynamically. In order to do so we change task.gpt so it:  
WEBSITE_URL 和 TOKEN 都是我们想要动态设置的占位符。为此，我们更改 task.gpt，使其：

- defines and url and a token as string arguments  
    定义 AND URL 和令牌作为字符串参数
- uses the ${} format to reference these values dynamically  
    使用 ${} 格式动态引用这些值

```json
args: url: URL of the website to check
args: token: webhook token

Create a task which verifies every minute if the website ${url} is reachable and sends the status code to the following HTTP POST request:
- URL is https://webhooks.app/data
- Authorization bearer is ${token}
- A json payload must be returned, containing a "message" property with the value of the ${url} and the status code returned
```

Let’s try to run this first version of the script as-is.  
让我们尝试按原样运行脚本的第一个版本。

But first we get a personal token from [https://webhooks.app](https://webhooks.app) (9326bdf257548c4277e8e395a2a772 in this example)  
但首先我们从 [https://webhooks.app](https://webhooks.app) 获取个人令牌（本例中为 9326bdf257548c4277e8e395a2a772）

![webhook-token.png](https://necessary-creativity-bc071c3082.media.strapiapp.com/webhook_token_fba68284a1.png)

Next we run the script using the following command which provides:  
接下来，我们使用以下命令运行脚本，该命令提供：

- the url to check: vote.votingapp.xyz which is a demo application  
    要检查的 URL：vote.votingapp.xyz 这是一个演示应用程序
- the token to send the status code to  
    要将状态代码发送到的令牌

```scss
$ gptscript task.gpt --token 9326bdf257548c4277e8e395a2a772 --url https://vote.votingapp.xyz
```

We get an output similar to:  
我们得到类似于以下内容的输出：

````json
OUTPUT:

```python
import requests
import time

def check_website_status(url, token):
    while True:
        try:
            response = requests.get(url)
            status_code = response.status_code
        except requests.exceptions.RequestException:
            status_code = "Error"

        payload = {
            "message": f"{url} {status_code}"
        }
        headers = {
            "Authorization": f"Bearer {token}",
            "Content-Type": "application/json"
        }

        requests.post("https://webhooks.app/data", json=payload, headers=headers)
        time.sleep(60)

check_website_status("https://vote.votingapp.xyz", "9326bdf257548c4277e8e395a2a772")
````

As we can see, from our instructions the model understands it needs to generate some code. It returned a Python script which checks the status of our website every 60 seconds and send the status code  
正如我们所看到的，从我们的指令中，模型明白它需要生成一些代码。它返回了一个 Python 脚本，该脚本每 60 秒检查一次我们网站的状态并发送状态代码

This python script would probably work fine but the main goal we want to achieve is a better automation of the whole thing. In order to do so we will define additional tools that the LLM can use in its replies.  
这个 python 脚本可能会正常工作，但我们想要实现的主要目标是更好地实现整个过程的自动化。为此，我们将定义LLM可以在其回复中使用的其他工具。

## Defining tools  定义工具

we will consider the following hierarchy of tools in this example:  
在此示例中，我们将考虑以下工具层次结构：

![tools-hierarchy.png](https://necessary-creativity-bc071c3082.media.strapiapp.com/tools_hierarchy_cb19343866.png)

First, we define create-regular-task which is a high-level tool. From 2 arguments, the command to run and its associated schedule, this tool relies on 2 other tools to create a virtual machine and to create a crontab entry into that one.  
首先，我们定义 create-regular-task，这是一个高级工具。从 2 个参数、要运行的命令及其关联的计划来看，该工具依赖于其他 2 个工具来创建虚拟机并创建该虚拟机的 crontab 条目。

```json
tools: create-vm, create-crontab-entry
name: create-regular-task
description: Manage the creation of a crontab on a remove VM
args: command: command to be run in a crontab without the schedule part
args: schedule: schedule to be used in a crontab

Perform the actions in the following order:

1. Create a virtual machine on DigitalOcean, wait for its IP to be returned and save it in ./vm.ip
2. Create a crontab entry for command ${command} and schedule ${schedule} in the VM which IP address is in file ./vm.ip
```

The tools referenced in create-regular-task are create-vm and create-crontab-entry:  
create-regular-task 中引用的工具是 create-vm 和 create-crontab-entry：

Below is the definition of create-vm which is in charge of creating a VM on DigitalOcean and saving its IP address to a file:  
以下是 create-vm 的定义，它负责在 DigitalOcean 上创建 VM 并将其 IP 地址保存到文件中：

```json
tools: sys.exec
name: create-vm
description: create a virtual machine on DigitalOcean

You are an operator which can use the doctl command line tool to interact with DigitalOcean infrastructure

Perform the actions in this exact order:

1. Get the ID of the ssh-key named gptscript and save it in ./key.id
2. Create a Virtual Machine in the new-york datacenter named cron making sure to provide the id from ./key.id as the ssh-key of the new droplet
3. Wait for the VM to be up and running and save its IP address in ./vm.ip
```

Below is the definition of create-crontab-entry with is in charge of sshing into the VM created by create-vm, creating a shell script with the command to run, and making sure this script is running every minute via the VM’s crontab:  
以下是 create-crontab-entry 的定义，它负责 ssh 到 create-vm 创建的 VM 中，使用要运行的命令创建一个 shell 脚本，并确保此脚本通过 VM 的 crontab 每分钟运行一次：

```json
tools: sys.exec, sys.write
name: create-crontab-entry
description: Create a crontab entry in a remote VM
args: command: command to run in a crontab on the remote VM
args: schedule: schedule for the crontab

Perform the step in this exact order taking into account that if you need to call a ssh command you must use user root and the IP address which value is in ./vm.ip

1. Create a bash file containing the ${command} to run without the schedule part, make it executable, and make sure the components are correctly escaped first.
2. Send this file to the remote VM via ssh saving it to /tmp/cron.sh on the remove VM
3. Create a crontab entry calling /tmp/cron.sh file for the schedule specified in the ${command}
```

Both create-vm and create-crontab-entry relies on base tools (sys.exec and sys.exec + sys.write respectively) making it possible to call local functions and to create files.  
create-vm 和 create-crontab-entry 都依赖于基本工具（分别为 sys.exec 和 sys.exec + sys.write），从而可以调用本地函数和创建文件。

Now that everything is in place we can test our set of actions.  
现在一切都已准备就绪，我们可以测试我们的作集。

## The whole thing in action  
整个事情都在行动

The final content of task.gpt is the following one:  
task.gpt 的最终内容如下：

```json
tools: create-regular-task
args: url: URL of the website to check
args: token: webhook token

Create a task which verifies every minute if the website ${url} is reachable and sends the status code to the following HTTP POST request:
- URL is https://webhooks.app/data
- Authorization bearer is ${token}
- A json payload must be returned, containing a "message" property with the value of the ${url} and the status code returned

---
tools: create-vm, create-crontab-entry
name: create-regular-task
description: Manage the creation of a crontab on a remote VM
args: command: command to be run in a crontab without the schedule part
args: schedule: schedule to be used in a crontab

Perform the actions in the following order:

1. Create a virtual machine on DigitalOcean, wait for its IP to be returned and save it in ./vm.ip
2. Create a crontab entry for command ${command} and schedule ${schedule} in the VM which IP address is in file ./vm.ip

---
tools: sys.exec, sys.write
name: create-crontab-entry
description: Create a crontab entry in a remote VM
args: command: command to run in a crontab on the remote VM
args: schedule: schedule for the crontab

Perform the step in this exact order taking into account that if you need to call a ssh command you must use user root and the IP address which value is in ./vm.ip

1. Create a bash file containing the ${command} to run without the schedule part, make it executable, and make sure the components are correctly escaped first.
2. Send this file to the remote VM via ssh saving it to /tmp/cron.sh on the remove VM
3. Create a crontab entry calling /tmp/cron.sh file for the schedule specified in the ${command}

---
tools: sys.exec
name: create-vm
description: create a virtual machine on DigitalOcean

You are an operator which can use the doctl command line tool to interact with DigitalOcean infrastructure

Perform the actions in this exact order:

1. Get the ID of the ssh-key named gptscript and save it in ./key.id
2. Create a Virtual Machine in the new-york datacenter named cron making sure to provide the id from ./key.id as the ssh-key of the new droplet
3. Wait for the VM to be up and running and save its IP address in ./vm.ip
```

Before running the test, we need:  
在运行测试之前，我们需要：

- a token to send the HTTP status code to the webhook  
    用于将 HTTP 状态代码发送到 Webhook 的令牌
- a DigitalOcean personal access token  
    DigitalOcean 个人访问令牌
- a ssh key  一个 SSH 密钥

- Webhook token: we will use the same token as in the first part  
    Webhook 令牌：我们将使用与第一部分相同的令牌
    
- Creation of a DO personal access token:  
    创建 DO 个人访问令牌：
    

First we log into our DigitalOcean account, then use the API menu at the bottom left and click “Generate New Token”. From the modal we give the token a name (gptscript in this example) and select both Read and Write scopes:  
首先，我们登录我们的 DigitalOcean 帐户，然后使用左下角的 API 菜单并单击“生成新令牌”。从模态中，我们为令牌命名（在本例中为 gptscript）并选择读取和写入范围：

![do-pat.png](https://necessary-creativity-bc071c3082.media.strapiapp.com/do_pat_1e2c1fb99d.png)

Once created we export the value of that token in the DIGITALOCEAN_ACCESS_TOKEN environment variable  
创建后，我们将该令牌的值导出到 DIGITALOCEAN_ACCESS_TOKEN 环境变量中

- Creation of a SSH key:  
    创建 SSH 密钥：

First we create a ssh key on your local machine using the following command:  
首先，我们使用以下命令在本地计算机上创建一个 ssh 密钥：

```bash
ssh-keygen -f /tmp/do_gptscript
```

This creates both do_gptscript (private key) and do_gpscript.pub (public key) in the /tmp directory  
这将在 /tmp 目录中创建 do_gptscript（私钥）和 do_gpscript.pub（公钥）

Next, from the DigitalOcean control panel, we navigate to the Settings menu at the bottom left and then go to the Security tab. From there we copy the public key (/tmp/do_gptscript.pub) and name it gptscript  
接下来，从 DigitalOcean 控制面板中，我们导航到左下角的“设置”菜单，然后转到“安全”选项卡。从那里我们复制公钥 （/tmp/do_gptscript.pub） 并将其命名为 gptscript

![do-ssh-key.png](https://necessary-creativity-bc071c3082.media.strapiapp.com/do_ssh_key_76cb452989.png)

We’re all set and can now run the whole example:  
我们已准备就绪，现在可以运行整个示例：

```json
gptscript --cache=false ./task.gpt --url https://vote.votingapp.xyz --token 9326bdf257548c4277e8e395a2a772
```

The output is similar to the following one (full output is provided for reference):  
输出类似于以下（提供完整的输出以供参考）：

```json
10:25:13 started  [main] [input=--url https://vote.votingapp.xyz --token 9326bdf257548c4277e8e395a2a772]
10:25:13 sent     [main]
         content  [1] content | Waiting for model response...
         content  [1] content | tool call create-regular-task -> {"defaultPromptParameter":"Create a task which verifies every minute if the website https://vote.votingapp.xyz is reachable and sends the status code to the following HTTP POST request:n- URL is https://webhooks.app/datan- Authorization bearer is 9326bdf257548c4277e8e395a2a772n- A json payload must be returned, containing a "message" property with the value of the https://vote.votingapp.xyz and the status code returned"}
10:25:19 started  [create-regular-task(2)] [input={"defaultPromptParameter":"Create a task which verifies every minute if the website https://vote.votingapp.xyz is reachable and sends the status code to the following HTTP POST request:n- URL is https://webhooks.app/datan- Authorization bearer is 9326bdf257548c4277e8e395a2a772n- A json payload must be returned, containing a "message" property with the value of the https://vote.votingapp.xyz and the status code returned"}]
10:25:19 sent     [create-regular-task(2)]
         content  [2] content | Waiting for model response...
         content  [2] content | tool call create-vm -> {"defaultPromptParameter":"Create a virtual machine on DigitalOcean. ...
10:25:21 started  [create-regular-task(3)->create-vm(3)] [input={"defaultPromptParameter":"Create a virtual machine on DigitalOcean."}]
10:25:21 sent     [create-regular-task(3)->create-vm(3)]
         content  [3] content | Waiting for model response...
         content  [3] content | tool call exec -> {"command":"doctl compute ssh-key list --format ID,Name --no-header | gre ...
10:25:23 started  [create-regular-task(4)->create-vm(4)->sys.exec(4)] [input={"command":"doctl compute ssh-key list --format ID,Name --no-header | grep gptscript | awk '{print $1}' > ./key.id"}]
10:25:23 sent     [create-regular-task(4)->create-vm(4)->sys.exec(4)]
10:25:24 ended    [create-regular-task(4)->create-vm(4)->sys.exec(4)]
10:25:24 continue [create-regular-task(3)->create-vm(3)]
10:25:24 sent     [create-regular-task(3)->create-vm(3)]
         content  [3] content | Waiting for model response...
         content  [3] content | tool call exec -> {"command":"doctl compute droplet create cron --region nyc1 --image ubunt ...
10:25:27 started  [create-regular-task(5)->create-vm(5)->sys.exec(5)] [input={"command":"doctl compute droplet create cron --region nyc1 --image ubuntu-20-04-x64 --size s-1vcpu-1gb --ssh-keys $(cat ./key.id) --wait --format PublicIPv4 --no-header > ./vm.ip"}]
10:25:27 sent     [create-regular-task(5)->create-vm(5)->sys.exec(5)]
10:26:01 ended    [create-regular-task(5)->create-vm(5)->sys.exec(5)]
10:26:01 continue [create-regular-task(3)->create-vm(3)]
10:26:02 sent     [create-regular-task(3)->create-vm(3)]
         content  [3] content | Waiting for model response...         content  [3] content | The virtual machine has been created on DigitalOcean, and its IP address has been saved in `./vm.ip` ...
10:26:03 ended    [create-regular-task(3)->create-vm(3)]
10:26:03 continue [create-regular-task(2)]
10:26:03 sent     [create-regular-task(2)]
         content  [2] content | Waiting for model response...
         content  [2] content | tool call create-crontab-entry -> {"command":"* * * * * curl -o /dev/null -s -w "%{http_co ...
10:26:10 started  [create-regular-task(6)->create-crontab-entry(6)] [input={"command":"* * * * * curl -o /dev/null -s -w "%{http_code}" https://vote.votingapp.xyz | xargs -I {} curl -X POST -H "Content-Type: application/json" -H "Authorization: Bearer 9326bdf257548c4277e8e395a2a772" -d '{"message": "https://vote.votingapp.xyz status code: {}"}' https://webhooks.app/data","schedule":"* * * * *"}]
10:26:10 sent     [create-regular-task(6)->create-crontab-entry(6)]
         content  [6] content | Waiting for model response...
         content  [6] content | tool call write -> {"content":"curl -o /dev/null -s -w "%{http_code}" https://vote.voting ...
10:26:14 started  [create-regular-task(7)->create-crontab-entry(7)->sys.write(7)] [input={"content":"curl -o /dev/null -s -w "%{http_code}" https://vote.votingapp.xyz | xargs -I {} curl -X POST -H "Content-Type: application/json" -H "Authorization: Bearer 9326bdf257548c4277e8e395a2a772" -d '{"message": "https://vote.votingapp.xyz status code: {}"}' https://webhooks.app/data","filename":"cron.sh"}]
10:26:14 sent     [create-regular-task(7)->create-crontab-entry(7)->sys.write(7)]
10:26:14 ended    [create-regular-task(7)->create-crontab-entry(7)->sys.write(7)]
10:26:14 continue [create-regular-task(6)->create-crontab-entry(6)]
10:26:14 sent     [create-regular-task(6)->create-crontab-entry(6)]
         content  [6] content | Waiting for model response...
         content  [6] content | tool call exec -> {"command":"chmod +x cron.sh"}
10:26:15 started  [create-regular-task(8)->create-crontab-entry(8)->sys.exec(8)] [input={"command":"chmod +x cron.sh"}]
10:26:15 sent     [create-regular-task(8)->create-crontab-entry(8)->sys.exec(8)]
10:26:15 ended    [create-regular-task(8)->create-crontab-entry(8)->sys.exec(8)]
10:26:15 continue [create-regular-task(6)->create-crontab-entry(6)]
10:26:16 sent     [create-regular-task(6)->create-crontab-entry(6)]
         content  [6] content | Waiting for model response...
         content  [6] content | tool call exec -> {"command":"cat ./vm.ip"}
10:26:17 started  [create-regular-task(9)->create-crontab-entry(9)->sys.exec(9)] [input={"command":"cat ./vm.ip"}]
10:26:17 sent     [create-regular-task(9)->create-crontab-entry(9)->sys.exec(9)]
10:26:17 ended    [create-regular-task(9)->create-crontab-entry(9)->sys.exec(9)]
10:26:17 continue [create-regular-task(6)->create-crontab-entry(6)]
10:26:18 sent     [create-regular-task(6)->create-crontab-entry(6)]
         content  [6] content | Waiting for model response...         content  [6] content | tool call exec -> {"command": "scp cron.sh root@159.65.218.18:/tmp/cron.sh"}
         content  [6] content | tool call exec -> {"command": "ssh root@159.65.218.18 'echo "* * * * * /tmp/cron.sh" | cr ...
10:26:21 started  [create-regular-task(10)->create-crontab-entry(10)->sys.exec(10)] [input={"command": "scp cron.sh root@159.65.218.18:/tmp/cron.sh"}]
10:26:21 started  [create-regular-task(11)->create-crontab-entry(11)->sys.exec(11)] [input={"command": "ssh root@159.65.218.18 'echo "* * * * * /tmp/cron.sh" | crontab -'"}]
10:26:21 sent     [create-regular-task(10)->create-crontab-entry(10)->sys.exec(10)]
10:26:21 sent     [create-regular-task(11)->create-crontab-entry(11)->sys.exec(11)]
10:26:27 ended    [create-regular-task(10)->create-crontab-entry(10)->sys.exec(10)]
10:26:27 ended    [create-regular-task(11)->create-crontab-entry(11)->sys.exec(11)]
10:26:27 continue [create-regular-task(6)->create-crontab-entry(6)]
10:26:28 sent     [create-regular-task(6)->create-crontab-entry(6)]
         content  [6] content | Waiting for model response...         content  [6] content | The bash file has been created, made executable, and sent to the remote VM at /tmp/cron.sh. Addition ...
10:26:30 ended    [create-regular-task(6)->create-crontab-entry(6)]
10:26:30 continue [create-regular-task(2)]
10:26:30 sent     [create-regular-task(2)]
         content  [2] content | Waiting for model response...         content  [2] content | The task has been successfully completed. A crontab entry was created on the remote VM to verify eve ...
10:26:33 ended    [create-regular-task(2)]
10:26:33 continue [main]
10:26:33 sent     [main]
         content  [1] content | Waiting for model response...         content  [1] content | The task has been successfully set up to verify every minute if the website https://vote.votingapp.xyz is reachable and to send the status code to the specified HTTP POST request.
10:26:37 ended    [main]

INPUT:

--url https://vote.votingapp.xyz --token 9326bdf257548c4277e8e395a2a772

OUTPUT:

The task has been successfully set up to verify every minute if the website https://vote.votingapp.xyz is reachable and to send the status code to the specified HTTP POST request.


```

We can then verify that a new VM has been created in our DigitalOcean control panel:  
然后，我们可以验证是否已在 DigitalOcean 控制面板中创建新的 VM：

![do-cron-vm.png](https://necessary-creativity-bc071c3082.media.strapiapp.com/do_cron_vm_320542b9f2.png)

We can ssh into that VM and verify that a crontab entry has been created:  
我们可以通过 ssh 连接到该 VM 并验证是否已创建 crontab 条目：

```json
root@cron:~# crontab -l
* * * * * /tmp/cron.sh
```

We can see _/tmp/cron.sh_ is executed every minute as requested.  
我们可以看到 _/tmp/cron.sh_ 会按照请求每分钟执行一次。

If we have a closer look at its content, we can see a one liner has been generated to perform the actions :  
如果我们仔细查看其内容，我们可以看到已经生成了一行代码来执行作：

- a first cURL command is used to get the status code returned when trying to access the URL [https://vote.votingapp.xyz](https://vote.votingapp.xyz)  
    第一个 cURL 命令用于获取尝试访问 URL 时返回的状态代码 [https://vote.votingapp.xyz](https://vote.votingapp.xyz)
- the result of the command is piped into another cURL in charge of sending the status code to a webhook  
    命令的结果通过管道传输到另一个 cURL，该 cURL 负责将状态代码发送到 webhook

```json
root@cron:~# cat /tmp/cron.sh
curl -o /dev/null -s -w "%{http_code}" https://vote.votingapp.xyz | xargs -I {} curl -X POST -H "Content-Type: application/json" -H "Authorization: Bearer 9326bdf257548c4277e8e395a2a772" -d '{"message": "https://vote.votingapp.xyz status code: {}"}' https://webhooks.app/data
```

We can see the result from [https://webhooks.add/dashboard](https://webhooks.add/dashboard) where a new message is received every minute  
我们可以看到每分钟收到一条新消息的 [https://webhooks.add/dashboard](https://webhooks.add/dashboard) 的结果

![result.png](https://necessary-creativity-bc071c3082.media.strapiapp.com/result_b4642c49c0.png)

Once we are done, we need to delete the DigitalOcean VM from the control panel (note: we could also define another tool to make sure the cleanup is done)  
完成后，我们需要从控制面板中删除 DigitalOcean VM（注意：我们还可以定义另一个工具来确保清理完成）

## Key takeaways  关键要点

This blog post illustrates a use case for GPTScript in defining a regular task. Through the definition of specific tools which GPTScript provides to the LLM we made possible the interactions with the API of a cloud provider and the configuration of a virtual machine. If you haven’t already, visit [GPTScript.ai](http://GPTScript.ai) to download and install GPTScript.  
这篇博文说明了 GPTScript 在定义常规任务中的一个用例。通过定义 GPTScript 提供的特定工具，LLM我们实现了与云提供商的 API 的交互和虚拟机的配置。如果您还没有，请访问 [GPTScript.ai](http://GPTScript.ai) 下载并安装 GPTScript。


Ref:
https://youtu.be/fqzqcZDBm0E
luc juggery  April 16, 2024

