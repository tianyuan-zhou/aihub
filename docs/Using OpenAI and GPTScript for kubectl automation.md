
使用 OpenAI 和 GPTScript 实现 kubectl 自动化

###### May 1, 2024 by luc juggery  

In this guide, we’ll show how to use OpenAI via GPTScript to automate a series of commands to deploy and expose an application on a Kubernetes cluster.  
在本指南中，我们将展示如何通过 GPTScript 使用 OpenAI 来自动化一系列命令，以在 Kubernetes 集群上部署和公开应用程序。

## High level overview view  高级概览视图

We’re going to use OpenAI via GPTScript to ask an LLM to perform the following tasks:  
我们将通过 GPTScript 使用 OpenAI 来要求 an LLM 执行以下任务：

1. Deploy the VotingApp into a dedicated namespace  
    将 VotingApp 部署到专用命名空间中
2. Install the Traefik Ingress Controller  
    安装 Traefik Ingress 控制器
3. Create an Ingress resource to expose the VotingApp to the internet  
    创建 Ingress 资源以向 Internet 公开 VotingApp
4. Open a browser to access the application  
    打开浏览器以访问应用程序

We cannot provide these instructions as-is because they are too high level, the LLM will not be able to understand all the hidden details. To accomplish these steps, we’ll create a GPTScript configuration file which provides detailed instructions for the LLM to follow, along with the necessary tools to complete the tasks.  
我们无法按原样提供这些说明，因为它们太高级了，LLM将无法理解所有隐藏的细节。为了完成这些步骤，我们将创建一个 GPTScript 配置文件，该文件提供了详细的说明，LLM以及完成任务所需的工具。

## Creating the GPTScript configuration file  
创建 GPTScript 配置文件

We start by creating a file named _kubectl.gpt_. This file contains the detailed steps that GPTScript will use to run the Kubernetes-related tasks. Because the LLM can’t know all the required details (like the URL for the VotingApp’s YAML specification), we need to provide additional information.  
我们首先创建一个名为 _kubectl.gpt_ 的文件。此文件包含 GPTScript 将用于运行 Kubernetes 相关任务的详细步骤。由于无法LLM知道所有必需的详细信息（例如 VotingApp 的 YAML 规范的 URL），因此我们需要提供其他信息。

Here’s what the enriched instruction set looks like:  
扩充的指令集如下所示：

```json
1. Create a Namespace named vote but do not fail if it already exists
2. Deploy in the vote namespace the application which yaml specification is available at https://luc.run/vote.yaml
3. Use a single command to wait for all the Pods in the vote namespace to be ready
4. Install Traefik ingress controller in kube-system namespace with helm only if it is not already installed in this namespace
5. Make sure the Traefik Pod is in running status
6. Wait for the IP address of the traefik Service to be available and save it in the file ./lb-ip.txt
7. Create the file ./ingress.yaml and make sure it contains the yaml specification of an Ingress resource which exposes the vote-ui Service on vote.LBIP.nip.io and the result-ui Service on result.LBIP.nip.io, first making sure to replace the LBIP placeholders with the content of the file ./lb-ip.txt
8. Create the Ingress resource specified in ./ingress.yaml
9. Open a browser on vote.LBIP.nip.io but make sure to replace the LBIP placeholder with the content of lb-ip.txt in this URL first
```

The steps are sequential, and each task depends on the successful completion of the previous ones. To ensure proper execution, we’ll need to specify that function calls should be made one at a time and in the order provided. We’ll also need to provide the set of tools (definitions of functions) that the model should be aware of.  
这些步骤是连续的，每个任务都取决于前几个任务的成功完成。为了确保正确执行，我们需要指定应按照提供的顺序一次进行一个函数调用。我们还需要提供模型应该注意的工具集（函数定义）。

## Specifying tools and constraints  
指定工具和约束

GPTScript allows use to define tools to perform specific tasks. In our example, we use four tools:  
GPTScript 允许使用定义工具来执行特定任务。在我们的示例中，我们使用四个工具：

- **sys.write**: to write to local files  
    **sys.write**：写入本地文件
- **kubectl**: to run Kubernetes commands  
    **kubectl**：运行 Kubernetes 命令
- **helm**: to manage Helm charts  
    **helm**：管理 Helm 图表
- **browser**: to open a web browser.  
    **浏览器**： 打开 Web 浏览器。

The first section of _kubectl.gpt_ specifies these tools and adds additional constraints:  
_kubectl.gpt_ 的第一部分指定了这些工具并添加了额外的约束：

```json
tools: sys.write, kubectl, helm, browser

Do not make parallel function calls. Only call one function at a time.
Perform the following tasks in order:
```

Next, we define the user-defined tools at the end of the file. Below is the definition of the **kubectl** tool:  
接下来，我们在文件末尾定义用户定义的工具。以下是 **kubectl** 工具的定义：

```json
---
name: kubectl
tools: sys.exec
description: use kubectl command to manage k8s resources
args: command: the command kubectl needs to run

You are a kubernetes operator which can run kubectl commands to manage clusters and applications
The only reason you use sys.exec tool must be to use kubectl to run the command provided, this command must start with kubectl
```

Similarly, the definition of the **helm** tool looks like this:  
同样，**helm** 工具的定义如下所示：

```json
---
name: helm
tools: sys.exec
description: use helm command to manage k8s charts
args: command: the command helm needs to run

You are a kubernetes operator which can run helm commands to manage charts
The only reason you use sys.exec tool must be to use helm to run the command provided, this command must start with helm
```

Both the kubectl and helm tools await for a command argument containing the list of parameters to be provided to kubectl and helm binaries respectively.  
kubectl 和 helm 工具都等待一个命令参数，该参数包含要分别提供给 kubectl 和 helm 二进制文件的参数列表。

The browser tool definition is the following one:  
浏览器工具定义如下：

```json
---
name: browser
tools: sys.exec
args: url: the url to open
description: open a browser window

You are only in charge of opening a browser window on the requested url
You can only use the sys.exec tool to open a browser window
```

## Running GPTScript configuration file  
运行 GPTScript 配置文件

First we need to ensure we have the kubectl and helm binaries installed and properly configured to communicate with our Kubernetes cluster. Also, we set the OPENAI_API_KEY environment variable with our OpenAI API key (when no model is defined in a tool, the **gpt-4-turbo-preview** model is used by default).  
首先，我们需要确保已安装并正确配置 kubectl 和 helm 二进制文件，以便与我们的 Kubernetes 集群通信。此外，我们使用 OpenAI API 密钥设置 OPENAI_API_KEY 环境变量（当工具中未定义模型时，默认使用 **gpt-4-turbo-preview** 模型）。

```bash
gptscript ./kubectl.gpt
```

After a few tens of seconds GPTScript should open a browser on the VotingApp  
几十秒后，GPTScript 应该会在 VotingApp 上打开一个浏览器

![votingapp.png](https://necessary-creativity-bc071c3082.media.strapiapp.com/votingapp_c8c848d4ab.png)

## Verifying the underlying cluster  
验证底层集群

To confirm everything is running fine, we can check the status of the application’s Pods:  
为了确认一切运行正常，我们可以检查应用程序的 Pod 的状态：

```json
kubectl get po -n vote
NAME                         READY   STATUS    RESTARTS   AGE
db-647c8f548b-jwbl5          1/1     Running   0          3m52s
redis-6f95f75d56-pk8cp       1/1     Running   0          3m53s
result-658b9b48cd-v44tc      1/1     Running   0          3m52s
result-ui-6965585c9f-5kc2x   1/1     Running   0          3m52s
vote-8588dfc9dc-xvrch        1/1     Running   0          3m53s
vote-ui-6bfbd99b58-mpwfx     1/1     Running   0          3m53s
worker-7744685d89-mwxs5      1/1     Running   0          3m52s
```

We can ensure the Traefik ingress controller is running:  
我们可以确保 Traefik 入口控制器正在运行：

```json
helm list -n kube-system
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART            APP VERSION
traefik kube-system     1               2024-04-21 18:13:39.536214 +0200 CEST   deployed        traefik-27.0.2   v2.11.2   
```

We can also verify the 2 local files created in the process:  
我们还可以验证在此过程中创建的 2 个本地文件：

- _lb-ip.txt_ contains the IP Address of the LoadBalancer exposing the Traefik Ingress Controller:  
    _lb-ip.txt_ 包含公开 Traefik Ingress Controller 的 LoadBalancer 的 IP 地址：

```json
% cat lb-ip.txt 
194.182.168.164
```

- ingress.yaml contain the yaml specification of the Ingress ressource exposing the VotingApp web ui:  
    ingress.yaml 包含暴露 VotingApp Web UI 的 Ingress 资源之 yaml 规范：

```json
% cat ingress.yaml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vote-result-ingress
  namespace: vote
spec:
  rules:
  - host: vote.194.182.168.164.nip.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: vote-ui
            port:
              number: 80
  - host: result.194.182.168.164.nip.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: result-ui
            port:
              number: 80  
```

## Key takeaways  关键要点

Leveraging GPTScript and OpenAI can simplify Kubernetes deployments, making it easier to perform complex tasks with a few simple instructions. By defining custom tools and following a clear sequence of tasks, we can automate various deployment processes. Also, the tools defined and used in this example are generic ones, they could be shared and used in other GPTScript applications.  
利用 GPTScript 和 OpenAI 可以简化 Kubernetes 部署，只需几条简单的说明即可更轻松地执行复杂的任务。通过定义自定义工具并遵循明确的任务顺序，我们可以自动化各种部署过程。此外，此示例中定义和使用的工具是通用工具，它们可以在其他 GPTScript 应用程序中共享和使用。