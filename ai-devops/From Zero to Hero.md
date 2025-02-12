# Harnessing AI to Become a DevOps Pro: From Zero to Hero  
利用 AI 成为 DevOps 专家：从零到英雄

![[./pics/Harnessing_AI_to_Become_a_Dev_Ops_Pro_Intro_From_Zero_to_Hero_0b496846db.png]]

###### October 11, 2024 by stein ove helset  

Follow me on this journey as I let AI teach me how to be come a DevOps pro, from zero to hero. At least, from zero to “I understand a little bit”.  
跟随我踏上这段旅程，让 AI 教我如何成为一名 DevOps 专业人士，从零到英雄。至少，从零到“我明白一点”。

I know that AI can help me a lot with DevOps. I have already tested out that before when I let [Clio](https://github.com/gptscript-ai/clio) build me a static blog using Hugo, and deploy it to a live server on Amazon Web Service. We even sat up automated deployment together. I learned a little bit, but I would still be lost if I tried doing it without the AI guidence.  
我知道 AI 可以在 DevOps 方面对我有很大帮助。我之前已经测试过了，当时我让 [Clio](https://github.com/gptscript-ai/clio) 使用 Hugo 为我构建了一个静态博客，并将其部署到 Amazon Web Service 上的实时服务器。我们甚至一起坐下来进行自动化部署。我学到了一些东西，但如果我在没有 AI 指导的情况下尝试这样做，我仍然会迷失方向。

> What is Clio? Learn more about Clio, our AI copilot that runs in your CLI, [here](https://blog.cl.io/).  
> 什么是 Clio？[在此处](https://blog.cl.io/)了解有关 Clio 的更多信息，Clio 是我们在 CLI 中运行的 AI 副驾驶。

In this series, I’m going to let AI teach me how I can do this on my own. I know vaguely what a DevOps do of course, but I don’t know much about CI/CD, infrastructure, containerization, etc. So I’m even going to need help from AI to know what I need to learn.  
在本系列中，我将让 AI 教我如何自己完成这项工作。当然，我大致知道 DevOps 的作用，但我对 CI/CD、基础设施、容器化等了解不多。因此，我什至需要 AI 的帮助才能知道我需要学习什么。

I’m going to use Clio, my AI assistant, as my teacher. The cool thing about using AI for things like this is that I can ask questions, make her recommend things for me, asking for assignments or videos to watch to dive deeper into things, etc. And I don’t have to thing about being that annoying student that always raises his hand, haha 🙂  
我将使用我的 AI 助手 Clio 作为我的老师。使用 AI 做这样的事情很酷的一点是，我可以提出问题，让她为我推荐东西，要求观看作业或视频以更深入地了解事物等。而且我不必做那个总是举手的烦人学生，哈哈 🙂

The first thing I’m going to do is to ask what I need to learn in order to become a DevOps engineer.  
我要做的第一件事是询问我需要学习什么才能成为一名 DevOps 工程师。

## Asking Clio  问 Clio

If you want to try this your self, you can go to [https://github.com/gptscript-ai/clio](https://github.com/gptscript-ai/clio). Here, you will find installation guides and similar.  
如果你想自己尝试一下，你可以去 [https://github.com/gptscript-ai/clio](https://github.com/gptscript-ai/clio)。在这里，您会找到安装指南和类似指南。

When you have Clio running on your computer, you can just start talking to her like you would have done with a teacher in real life.  
当你的电脑运行 Clio 时，你可以开始与她交谈，就像你在现实生活中与老师交谈一样。

```bash
>>> I want to become a DevOps. Maybe not a pro, but enough to understand how things work and even build and deploy projects to the cloud. Can you briefly describe 8 important topics I need to learn? Start with the basics please.

Absolutely! Here are 8 important topics to get you started on your journey to becoming proficient in DevOps:

1. Version Control Systems (VCS)
• Description: Learn how to use version control systems like Git. This is fundamental for tracking changes in
your code and collaborating with others.
• Key Tools: Git, GitHub, GitLab, Bitbucket.
2. Continuous Integration/Continuous Deployment (CI/CD)
• Description: Understand the principles of CI/CD to automate the process of integrating code changes, testing,
and deploying applications.
• Key Tools: Jenkins, GitHub Actions, GitLab CI, CircleCI, Travis CI.
3. Containerization
• Description: Learn how to package applications and their dependencies into containers to ensure consistency
across different environments.
• Key Tools: Docker, Podman.
4. Orchestration
• Description: Manage and automate the deployment, scaling, and operation of containers.
• Key Tools: Kubernetes, Docker Swarm, OpenShift.
5. Infrastructure as Code (IaC)
• Description: Use code to manage and provision infrastructure, making it easier to automate and scale.
• Key Tools: Terraform, Ansible, CloudFormation, Pulumi.
6. Cloud Providers
• Description: Get familiar with major cloud service providers and their services for computing, storage,
networking, and more.
• Key Providers: AWS, Azure, Google Cloud Platform (GCP), DigitalOcean.
7. Monitoring and Logging
• Description: Implement monitoring and logging to keep track of application performance and troubleshoot
issues.
• Key Tools: Prometheus, Grafana, ELK Stack (Elasticsearch, Logstash, Kibana), Splunk.
8. Security
• Description: Learn the best practices for securing your applications and infrastructure.
• Key Concepts: Secure coding practices, secrets management, network security, compliance.
```

As you can see, I got a list of 8 different topics. Based on what I read here, I think this could be a good list of topics. These 8 topics will be an article in this series.  
如你所见，我得到了 8 个不同主题的列表。根据我在这里读到的内容，我认为这可能是一个不错的主题列表。这 8 个主题将成为本系列的一篇文章。

## Before I get started  开始之前

Before I start diving directly in to part 1, let me just ask if there is anything I should do before I start.  
在我开始直接进入第 1 部分之前，让我先问一下在开始之前我应该做些什么。

```>>>

Yes, there are a few prerequisites and additional considerations that can help you on your DevOps journey:

### Prerequisites:

1. Basic Programming Knowledge:
• Description: Understanding basic programming concepts is essential. You don't need to be an expert, but
familiarity with at least one programming language (e.g., Python, JavaScript, Java) will be very helpful.
• Why: Many DevOps tools and scripts require some level of programming.
2. Basic Linux/Unix Command Line Skills:
• Description: Familiarity with the command line interface (CLI) in Linux/Unix is crucial.
• Why: Most DevOps tools and environments are based on Linux/Unix systems.
3. Networking Fundamentals:
• Description: Understanding basic networking concepts such as IP addresses, DNS, firewalls, and load balancers.
• Why: Networking is a core component of deploying and managing applications.
4. Understanding of Software Development Lifecycle (SDLC):
• Description: Familiarity with the stages of software development from planning to deployment and maintenance.
• Why: DevOps practices are integrated into the SDLC to improve efficiency and collaboration.


### Additional Considerations:

1. Mindset and Culture:
• Description: DevOps is as much about culture as it is about tools and practices. It emphasizes collaboration,
communication, and continuous improvement.
• Why: A DevOps mindset helps in breaking down silos between development and operations teams.
2. Soft Skills:
• Description: Skills such as problem-solving, communication, and teamwork are crucial.
• Why: DevOps involves working closely with different teams and stakeholders.
3. Community and Resources:
• Description: Engage with the DevOps community through forums, meetups, and conferences.
• Why: Learning from others' experiences and staying updated with the latest trends and tools is beneficial.
4. Hands-On Practice:
• Description: Practical experience is invaluable. Set up your own projects, contribute to open-source projects,
or use sandbox environments provided by cloud providers.
• Why: Hands-on practice helps in reinforcing theoretical knowledge and gaining real-world experience.
5. Documentation and Learning Resources:
• Description: Make use of official documentation, online courses, tutorials, and books.
• Why: There is a wealth of information available that can guide you through each topic.


By ensuring you have these prerequisites and considerations in mind, you'll be better prepared to dive into the
world of DevOps. If you need recommendations for specific resources or have any other questions, feel free to ask!
```

I think this was a good answer, and it makes a lot of sense to know most of what she’s talking about here. Luckily for me, I’ve been a programmer for over 15 years, so this is things I already know. I guess that when I start with the 8 parts, some of these topics like command line, networking and similar will be brought into light. If I’m stuck at a point, I can just ask my teacher.  
我认为这是一个很好的回答，了解她在这里谈论的大部分内容非常有意义。幸运的是，我已经做了超过 15 年的程序员，所以这是我已经知道的事情。我想，当我从 8 个部分开始时，其中一些主题（如命令行、网络等）将被揭示出来。如果我在某个点上遇到困难，我可以问我的老师。

## Summary  总结

Okay, so far I have learned about what I need to learn in order to become a DevOps engineer. I know a little bit about what we’re going to go through over the next weeks and similar.  
好的，到目前为止，我已经了解了成为一名 DevOps 工程师需要学习什么。我对接下来几周我们将要经历的事情以及类似的事情有所了解。

I have not tried learning a new skill this way before, so it will be interesting to see how it goes. I hope the series sounds interesting and that you want to follow me and Clio on this journey.  
我以前没有尝试过以这种方式学习新技能，所以看看它如何发展会很有趣。我希望这个系列听起来很有趣，并且您想跟随我和 Clio 踏上这段旅程。

---

To jump to a specific part of this series **Harnessing AI to Become a DevOps Pro** click one the link to one of the 8 key areas of DevOps that we will be exploring:  
要跳转到本系列的特定部分 **利用 AI 成为 DevOps 专业人士**，请单击我们将探索的 DevOps 的 8 个关键领域之一的链接之一：

1. [Version control Systems (VCS)  
    版本控制系统 （VCS）](https://acorn-preview.vercel.app/resources/tutorials/harnessing-ai-to-become-a-devops-pro-part-1-version-control-systems)
2. [Continuous Integration/Continuous Deployment (CI/CD)  
    持续集成/持续部署 （CI/CD）](https://www.acorn.io/resources/tutorials/harnessing-ai-to-become-a-devops-pro-part-2-ci-cd)
3. [Containerization  集装箱](https://www.acorn.io/resources/tutorials/harnessing-ai-to-become-a-devops-pro-part-3-containerization)
4. [Orchestration  配器](https://www.acorn.io/resources/tutorials/harnessing-ai-to-become-a-devops-pro-part-4-orchestration)
5. [Infrastructure as Cloud (IaC)  
    基础设施即云 （IaC）](https://www.acorn.io/resources/tutorials/harnessing-ai-to-become-a-devops-pro-part-5-iac)
6. [Cloud Providers  云提供商](https://www.acorn.io/resources/tutorials/harnessing-ai-to-become-a-devops-pro-part-6-cloud-providers)
7. [Monitoring and Logging  监控和日志记录](https://www.acorn.io/resources/tutorials/harnessing-ai-to-become-a-devops-pro-part-7-monitoring-and-logging)
8. [Security  安全](https://www.acorn.io/resources/tutorials/harnessing-ai-to-become-a-devops-pro-intro-from-zero-to-hero/_wp_link_placeholder)

All you need to get started is some basic programming knowledge!  
您只需具备一些基本的编程知识即可开始！

Follow along each week as we release detailed guides to each of the topics above.  
请关注我们每周发布上述每个主题的详细指南。