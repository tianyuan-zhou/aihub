使用 AI 和 GPTScript 加速 Kubernetes 集群

###### March 28, 2024 by luc juggery  

GPTScript is a new scripting language designed to facilitate interactions with Large Language Models (LLMs). Initially compatible with OpenAI models, plans are underway to extend its utility to other platforms, including local models and solutions from other vendors.  
GPTScript 是一种新的脚本语言，旨在促进与大型语言模型的交互 （LLMs）。最初与 OpenAI 模型兼容，目前正在计划将其实用程序扩展到其他平台，包括本地模型和来自其他供应商的解决方案。

## Using LLM  用 LLM

The primary method for most users to interact with LLMs today involves querying through web interfaces of various solutions like:  
LLMs今天大多数用户与之交互的主要方法涉及通过各种解决方案的 Web 界面进行查询，例如：

- ChatGPT: [https://chat.openai.com](https://chat.openai.com)  
    ChatGPT：[https://chat.openai.com](https://chat.openai.com)
- Le Chat: [https://chat.mistral.ai/chat](https://chat.mistral.ai/chat)  
    Le Chat：[https://chat.mistral.ai/chat](https://chat.mistral.ai/chat)
- Gemini: [https://gemini.google.com/app](https://gemini.google.com/app)  
    双子座：[https://gemini.google.com/app](https://gemini.google.com/app)
- Claude: [https://claude.ai/](https://claude.ai/) (availability may vary by country)  
    Claude：[https://claude.ai/](https://claude.ai/)（可用性可能因国家/地区而异）

For advanced AI application development, APIs provided by these platforms enable programmatic interactions. Developers can leverage these APIs in various programming languages, including Python and Node.js, to integrate LLM functionalities into their applications.  
对于高级 AI 应用程序开发，这些平台提供的 API 支持编程交互。开发人员可以通过各种编程语言（包括 Python 和 Node.js）利用这些 API，将功能集成到LLM他们的应用程序中。

## About Function calling  关于函数调用

OpenAI models support function calling, enabling them to recognize when a function call is necessary and provide the client application with the required function name and parameters. This capability allows client applications to execute commands or functions as determined by the LLM, enriching the interactive experience. This feature is not universal across all models; some may require additional training (fine tuning) to acquire function-calling abilities.  
OpenAI 模型支持函数调用，使它们能够识别何时需要函数调用，并为客户端应用程序提供所需的函数名称和参数。此功能允许客户端应用程序执行由 LLM确定的命令或函数，从而丰富交互式体验。此功能并非在所有模型中都是通用的;有些模型可能需要额外的训练（微调）才能获得函数调用能力。

## GPTScript

GPTScript enhances LLM interactions by incorporating predefined tools that leverage OpenAI models’ function-calling capabilities. These tools include:  
GPTScript 通过整合利用 OpenAI 模型的函数调用功能的预定义工具来增强LLM交互。这些工具包括：

- sys.exec: Executes a command and returns its output  
    sys.exec：执行命令并返回其输出
- sys.download: Downloads content from a URL to a specified location  
    sys.download：将内容从 URL 下载到指定位置
- sys.getenv: Retrieves the value of an environment variable  
    sys.getenv：检索环境变量的值
- sys.http.get: Downloads content from a specified HTTP or HTTPS URL  
    sys.http.get：从指定的 HTTP 或 HTTPS URL 下载内容
- sys.http.post: Sends data to a specified HTTP or HTTPS URL using the POST method  
    sys.http.post：使用 POST 方法将数据发送到指定的 HTTP 或 HTTPS URL
- And more, which can be discovered using gptscript –list-tools  
    以及更多功能，可以使用 gptscript –list-tools 发现

On top of the native GPTScript’s tools, users can create additional tools such as Node.js, bash, or Python script.  
除了原生 GPTScript 的工具之外，用户还可以创建其他工具，例如 Node.js、bash 或 Python 脚本。

GPTScript essentially serves as an intermediary, interpreting function call needs from LLM responses and executing the corresponding actions.  
GPTScript 本质上充当中介，从响应中LLM解释函数调用需求并执行相应的作。

## Bootstrapping a Kubernetes cluster  
引导 Kubernetes 集群

Using GPTScript, we will demonstrate the creation and bootstrapping of a Kubernetes cluster on the Civo cloud platform, employing a GitOps approach for application deployment.  
使用 GPTScript，我们将演示在 Civo 云平台上创建和引导 Kubernetes 集群，采用 GitOps 方法进行应用程序部署。

The process involves:  该过程包括：

- Creating a Civo cluster and waiting for activation  
    创建 Civo 集群并等待激活
- Configuring kubectl to interact with the new cluster  
    配置 kubectl 以与新集群交互
- Using Flux to initialize the cluster and deploy applications  
    使用 Flux 初始化集群和部署应用程序
- Monitoring the deployment process through a web interface  
    通过 Web 界面监控部署过程

This approach illustrates the potential of LLMs to simplify and automate cloud operations.  
这种方法说明了简化和自动化云作的潜力LLMs。

Notes:  笔记：

- While it’s possible to execute these actions using scripting, exploring how LLMs can assist us presents an interesting insight  
    虽然可以使用脚本来执行这些作，但探索如何LLMs帮助我们提供了一个有趣的见解
- The cluster bootstrap step will utilize content from the GitLab repository [https://gitlab.com/techwhale/config-fluxcd](https://gitlab.com/techwhale/config-fluxcd) and deploy Helm applications and infrastructure components defined in this one. For details on managing Helm charts through Flux, visit [https://fluxcd.io/flux/use-cases/helm/](https://fluxcd.io/flux/use-cases/helm/).  
    集群引导步骤将利用 GitLab 存储库中的内容 [https://gitlab.com/techwhale/config-fluxcd](https://gitlab.com/techwhale/config-fluxcd) 并部署此处定义的 Helm 应用程序和基础设施组件。有关通过 Flux 管理 Helm 图表的详细信息，请访问 [https://fluxcd.io/flux/use-cases/helm/](https://fluxcd.io/flux/use-cases/helm/)。

Directly sending these instructions to an LLM without using GPTScript might not be relevant due to the specificity of the commands required. GPTScript excellently demonstrates its value by utilizing the LLM’s function calling capability to execute the necessary commands.  
由于所需命令的特殊性，在不使用 GPTScript 的情况下直接将这些指令发送到 可能LLM无关紧要。GPTScript 通过利用 LLM的函数调用功能来执行必要的命令，出色地展示了它的价值。

To ensure the LLM correctly understand the required actions (and which commands to execute), additional details are added to our instructions:  
为了确保LLM正确理解所需的作（以及要执行的命令），我们的说明中添加了其他详细信息：

```json
Perform the following actions in the following order:

1. Create a Civo cluster named demo version v1.28.2-k3s1, with 2 g4s.kube.medium nodes, disabling traefik2-nodeport application but does not wait for it to be ready
2. Wait for the Civo demo cluster to be active
3. Set the kubectl context to the demo cluster, make sure to use the --save and --switch flags
4. Create a flux-system namespace
5. Create a generic secret named sops-age in the flux-system namespace, this secret has the key age.agekey which value is the content of the file which path is specified in the SOPS_AGE_KEY_FILE environment variable
6. Use flux to bootstrap the dev environment using the repository named config-fluxcd and wait for the command to be finished
7. Wait for 60 seconds
8. Run a process that uses kubectl port-forward to expose the port 9001 of the service weave-gitops in namespace flux-system making sure it runs in the background and use >/dev/null 2>&1
9. Open a browser on the http://localhost:9001
```

Notes:  笔记：

- These instructions are the result of iterative testing  
    这些说明是迭代测试的结果
- Some instructions may appear verbose; however, as LLMs evolve, their ability to understand and execute commands could reduce the need for such verbosity  
    有些指令可能看起来很冗长;然而，随着 LLMs Evolve 的发展，他们理解和执行命令的能力可以减少对这种冗长的需求

We provide the LLM with four tools to facilitate these actions:  
我们提供了LLM四种工具来促进这些作：

- sys.exec: execute a command  
    sys.exec：执行命令
- wait-cluster: ensure the cluster is ready  
    wait-cluster：确保集群已准备就绪
- flux: execute the bootstrap command with the correct parameters  
    Flux：使用正确的参数执行 bootstrap 命令
- browser: launch a web browser for real time monitoring  
    浏览器：启动 Web 浏览器以进行实时监控

Note: wait-cluster, flux and browser are 3 user-defined tools  
注意：wait-cluster、flux 和 browser 是 3 个用户定义的工具

The bootstrap.gpt file outlines the entire process:  
bootstrap.gpt 文件概述了整个过程：

```json
Tools: sys.exec, wait-cluster, flux, browser

Perform the following actions in the following order:

1. Create a Civo cluster named demo version v1.28.2-k3s1, with 2 g4s.kube.medium nodes, disabling traefik2-nodeport application but does not wait for it to be ready
2. Wait for the Civo demo cluster to be active
3. Set the kubectl context to the demo cluster, make sure to use the --save and --switch flags
4. Create a flux-system namespace
5. Create a generic secret named sops-age in the flux-system namespace, this secret has the key age.agekey which value is the content of the file which path is specified in the SOPS_AGE_KEY_FILE environment variable
6. Use flux to bootstrap the dev environment using the repository named config-fluxcd and wait for the command to be finished
7. Wait for 60 seconds
8. Run a process that uses kubectl port-forward to expose the port 9001 of the service weave-gitops in namespace flux-system making sure it runs in the background and use >/dev/null 2>&1
9. Open a browser on the http://localhost:9001

---
name: wait-cluster
description: wait for a civo cluster to be ready
args: cluster: name of the cluster to wait for

#!/bin/bash

while true; do
    status=$(civo k3s show ${cluster} -o json | jq -r '.status')
    if [[ $status == "ACTIVE" ]]; then
        echo "cluster is ACTIVE"
        exit 0
    fi
    sleep 30
done

---
name: flux
description: bootstrap a cluster using flux
args: environment: environment to bootstrap
args: repository: name of the git repository to use

#!/bin/bash

flux bootstrap gitlab 
  --deploy-token-auth 
  --owner=techwhale 
  --repository ${repository} 
  --branch=main 
  --path clusters/${environment}

---
name: browser
description: browser stuff
args: url: domain to open

#!python

import sys
import time
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

# Chrome options to bypass the SSL error for self-signed certificate
chrome_options = Options()
chrome_options.add_argument("--ignore-ssl-errors=yes")
chrome_options.add_argument("--ignore-certificate-errors")

# Initialize the WebDriver
chromedriver_path = '/opt/homebrew/bin/chromedriver'
service = Service(executable_path=chromedriver_path)
driver = webdriver.Chrome(service=service, options=chrome_options)

try:
    # Open the URL
    driver.get(url)
    
    # Wait for login form to be there
    WebDriverWait(driver, 120).until(
      EC.presence_of_element_located((By.ID, "email"))
    )

    # Fill credentials
    email_input = driver.find_element(By.ID, "email")
    email_input.send_keys("admin")
    password_input = driver.find_element(By.ID, "password")
    password_input.send_keys("flux")

    # Validate form
    button = driver.find_element(By.CLASS_NAME, "MuiButton-label")
    button.click()

except Exception as e:
    print("An unexpected error occurred. ", e)
finally:
    time.sleep(180)
    driver.quit()
```

The content can be split into:  
内容可以拆分为：

- Instructions for the LLM and the available tools.  
    的说明LLM和可用工具。
- Definitions for each tool (wait-cluster, flux, browser)  
    每个工具的定义（wait-cluster、flux、browser）

GPTScript interacts with the LLM by sending instructions and acting on the LLM’s responses to call the appropriate tool with the necessary parameters.  
GPTScript 通过发送指令并根据 LLM的响应进行作来调用具有必要参数的适当工具，从而与 进行LLM交互。

Below is the output obtained when running the bootstrap.gpt script:  
以下是运行 bootstrap.gpt 脚本时获得的输出：

```json
$ gptscript ./bootstrap.gpt
18:52:44 started  [main]
18:52:44 sent     [main]
18:52:44 started  [sys.exec(2)] [input={"command":"civo kubernetes create demo --version v1.28.2-k3s1 --nodes 2 --size g4s.kube.medium --remove-applications traefik2-nodeport"}]
18:52:44 sent     [sys.exec(2)]
18:53:28 ended    [sys.exec(2)]
18:53:28 continue [main]
18:53:28 sent     [main]
         content  [1] content | Waiting for model response...
         content  [1] content | tool call wait-cluster -> {"cluster":"demo"}
18:53:30 started  [wait-cluster(3)] [input={"cluster":"demo"}]
18:53:30 sent     [wait-cluster(3)]
18:55:00 ended    [wait-cluster(3)]
18:55:00 continue [main]
18:55:00 sent     [main]
         content  [1] content | Waiting for model response...
         content  [1] content | tool call sys -> {"command":"civo kubernetes config demo --save --switch"}
18:55:01 started  [sys.exec(4)] [input={"command":"civo kubernetes config demo --save --switch"}]
18:55:01 sent     [sys.exec(4)]
18:55:18 ended    [sys.exec(4)]
18:55:18 continue [main]
18:55:18 sent     [main]
         content  [1] content | Waiting for model response...
         content  [1] content | tool call sys -> {"command":"kubectl create namespace flux-system"}
18:55:19 started  [sys.exec(5)] [input={"command":"kubectl create namespace flux-system"}]
18:55:19 sent     [sys.exec(5)]
18:55:19 ended    [sys.exec(5)]
18:55:19 continue [main]
18:55:19 sent     [main]
         content  [1] content | Waiting for model response...
         content  [1] content | tool call sys -> {"command":"kubectl create secret generic sops-age --namespace=flux-system --from-file=age.agekey=${SOPS_AGE_KEY_FILE}"}
18:55:21 started  [sys.exec(6)] [input={"command":"kubectl create secret generic sops-age --namespace=flux-system --from-file=age.agekey=${SOPS_AGE_KEY_FILE}"}]
18:55:21 sent     [sys.exec(6)]
18:55:22 ended    [sys.exec(6)]
18:55:22 continue [main]
18:55:22 sent     [main]
         content  [1] content | Waiting for model response...
         content  [1] content | tool call flux -> {"environment":"dev","repository":"config-fluxcd"}
18:55:23 started  [flux(7)] [input={"environment":"dev","repository":"config-fluxcd"}]
18:55:23 sent     [flux(7)]
► connecting to https://gitlab.com
► cloning branch "main" from Git repository "https://gitlab.com/techwhale/config-fluxcd.git"
✔ cloned repository
► generating component manifests
✔ generated component manifests
✔ component manifests are up to date
► installing components in "flux-system" namespace
✔ installed components
✔ reconciled components
► checking to reconcile deploy token for source secret
✔ configured deploy token "flux-system-main-flux-system-./clusters/dev" for "https://gitlab.com/techwhale/config-fluxcd"
► determining if source secret "flux-system/flux-system" exists
► generating source secret
► applying source secret "flux-system/flux-system"
✔ reconciled source secret
► generating sync manifests
✔ generated sync manifests
✔ sync manifests are up to date
► applying sync manifests
✔ reconciled sync configuration
◎ waiting for GitRepository "flux-system/flux-system" to be reconciled
✔ GitRepository reconciled successfully
◎ waiting for Kustomization "flux-system/flux-system" to be reconciled
✔ Kustomization reconciled successfully
► confirming components are healthy
✔ helm-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ notification-controller: deployment ready
✔ source-controller: deployment ready
✔ all components are healthy
18:55:50 ended    [flux(7)]
18:55:50 continue [main]
18:55:50 sent     [main]
         content  [1] content | Waiting for model response...
         content  [1] content | tool call sys -> {"command":"sleep 60"}
18:55:52 started  [sys.exec(8)] [input={"command":"sleep 60"}]
18:55:52 sent     [sys.exec(8)]
18:56:52 ended    [sys.exec(8)]
18:56:52 continue [main]
18:56:52 sent     [main]
         content  [1] content | Waiting for model response...
         content  [1] content | tool call sys -> {"command":"kubectl port-forward svc/weave-gitops -n flux-system 9001:9001 >/dev/null 2>&1 &"}
18:56:54 started  [sys.exec(9)] [input={"command":"kubectl port-forward svc/weave-gitops -n flux-system 9001:9001 >/dev/null 2>&1 &"}]
18:56:54 sent     [sys.exec(9)]
18:56:54 ended    [sys.exec(9)]
18:56:54 continue [main]
18:56:54 sent     [main]
         content  [1] content | Waiting for model response...
         content  [1] content | tool call browser -> {"url":"http://localhost:9001"}
18:56:55 started  [browser(10)] [input={"url":"http://localhost:9001"}]
18:56:55 sent     [browser(10)]
18:58:57 ended    [browser(10)]
18:58:57 continue [main]
18:58:57 sent     [main]
         content  [1] content | Waiting for model response...         content  [1] content | The process has been completed
18:58:59 ended    [main]
```

For each step, GPTScript sends instructions to the LLM and receives a response. Based on this response, GPTScript then executes a corresponding tool. Let’s examine two steps in detail:  
对于每个步骤，GPTScript 都会向 发送指令LLM并接收响应。根据此响应，GPTScript 然后执行相应的工具。让我们详细研究两个步骤：

- Creation of the cluster  创建集群

```json
18:52:44 started  [main]
18:52:44 sent     [main]
18:52:44 started  [sys.exec(2)] [input={"command":"civo kubernetes create demo --version v1.28.2-k3s1 --nodes 2 --size g4s.kube.medium --remove-applications traefik2-nodeport"}]
18:52:44 sent     [sys.exec(2)]
18:53:28 ended    [sys.exec(2)]
```

This step demonstrates that, based on our instructions, the LLM identified the need to execute the command: `civo kubernetes create demo --version v1.28.2-k3s1 --nodes 2 --size g4s.kube.medium --remove-applications traefik2-nodeport` It then sent this command back, enabling GPTScript to utilize the native sys.exec function for execution.  
此步骤表明，根据我们的指示，确定了LLM执行命令的需求： `civo kubernetes create demo --version v1.28.2-k3s1 --nodes 2 --size g4s.kube.medium --remove-applications traefik2-nodeport` 然后，它将此命令发回，使 GPTScript 能够利用本机 sys.exec 函数执行。

- Bootstrapping the cluster  
    引导集群

```json
18:55:22 sent     [main]
         content  [1] content | Waiting for model response...
         content  [1] content | tool call flux -> {"environment":"dev","repository":"config-fluxcd"}
18:55:23 started  [flux(7)] [input={"environment":"dev","repository":"config-fluxcd"}]
18:55:23 sent     [flux(7)]
► connecting to https://gitlab.com
…
✔ all components are healthy
18:55:50 ended    [flux(7)]
```

This illustrates that, upon receiving our instructions, the LLM understood the flux tool needed to be invoked with the parameters for environment and repository. It then provided this information back so GPTScript can deploy the flux tool accordingly.  
这表明，在收到我们的指示后，LLM理解了需要使用 environment 和 repository 参数调用的 flux 工具。然后，它提供了这些信息，以便 GPTScript 可以相应地部署 flux 工具。

The last intruction of the process utilizes the browser tool to automatically launch a Chrome browser and navigate to [http://localhost:9001](http://localhost:9001), the Weave-GitOps UI. Shortly after, it confirms that all applications are operational.  
该过程的最后一个说明利用浏览器工具自动启动 Chrome 浏览器并导航到 [http://localhost:9001](http://localhost:9001)，即 Weave-GitOps UI。不久之后，它确认所有应用程序都可以运行。

![weave-gitops.png](https://necessary-creativity-bc071c3082.media.strapiapp.com/weave_gitops_4f211fe8c5.png)

Upon inspecting the Civo dashboard, we observe the creation of a cluster named "demo" featuring 2 Medium – Standard nodes, configured to Kubernetes version 1.28.2 as specified.  
在检查 Civo 仪表板时，我们观察到一个名为“demo”的集群的创建，该集群具有 2 个中型 – 标准节点，配置为指定的 Kubernetes 版本 1.28.2。

![civo.png](https://necessary-creativity-bc071c3082.media.strapiapp.com/civo_3cd8b5cad5.png)

The two applications defined in the GitLab repository have been successfully deployed and are now fully operational.  
GitLab 存储库中定义的两个应用程序已成功部署，现在可以全面运行。

- Webhooks  Webhook

![webhooks.png](https://necessary-creativity-bc071c3082.media.strapiapp.com/webhooks_89d0743223.png)

- Voting Application  投票申请

![vote.png](https://necessary-creativity-bc071c3082.media.strapiapp.com/vote_bdd926dec6.png)

![result.png](https://necessary-creativity-bc071c3082.media.strapiapp.com/result_3e6162c8a4.png)

# Key takeaways  关键要点

This blog post illustrates a use case for GPTScript in automating Kubernetes cluster management on the Civo platform. Through detailed instructions and a step-by-step approach it outlined the interactions between GPTScript and the LLM focusing on the tools GPTScript offers to leverage the LLM’s function calling capability.  
这篇博文说明了 GPTScript 在 Civo 平台上自动化 Kubernetes 集群管理的用例。通过详细的说明和分步方法，它概述了 GPTScript 之间的交互，并LLM重点介绍了 GPTScript 提供的用于利用 LLM函数调用功能的工具。