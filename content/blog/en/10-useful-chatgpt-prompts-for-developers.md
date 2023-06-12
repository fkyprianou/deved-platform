---
title: 10 Useful ChatGPT Prompts For Developers
description: Explore a curated collection of ChatGPT prompts designed to assist
  software developers in their daily tasks
thumbnail: /content/blog/10-useful-chatgpt-prompts-for-developers/chatgpt-prompts.png
author: michael-crump
published: true
published_at: 2023-06-13T19:04:36.457Z
updated_at: 2023-06-13T09:28:34.684Z
category: inspiration
tags:
  - chatgpt
  - ai
  - developers
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

Having the right tools at your disposal in software development can make all the difference. Enter [ChatGPT](https://openai.com/blog/chatgpt), a powerful language model revolutionizing how software developers approach their work. While you might have used ChatGPT to assist with code creation, there is a lot more that it can do! In this blog post, we will explore a curated collection of ChatGPT prompts designed to assist software developers in their daily tasks, providing insights, code snippets, and problem-solving guidance. Let's get started! 

## Getting Started

**\#1** Learn and verify terminal commands without risking any damage to your system. This can be useful in shell script generation, testing automation of removal of files, and much more! 

**Prompt ([Author Credit](https://github.com/f)):** I want you to act as a linux terminal. I will type commands and you will reply with what the terminal should show. I want you to only reply with the terminal output inside one unique code block, and nothing else. do not write explanations. do not type commands unless I instruct you to do so. When I need to tell you something in English, I will do so by putting text inside curly brackets {like this}. My first command is pwd.

![Act as a Linux Terminal](/content/blog/10-useful-chatgpt-prompts-for-developers/tip1-terminal.png "Act as a Linux Terminal")

**Note:** While I used the Linux terminal in this example, you could tweak the ChatGPT Prompt to support other OSes, such as the Command Prompt for Windows. 

**\#2** Use ChatGPT as a JavaScript console to test in a "clean" environment.

**Prompt ([Author Credit](https://github.com/omerimzali)):** I want you to act as a javascript console. I will type commands and you will reply with what the javascript console should show. I want you to only reply with the terminal output inside one unique code block, and nothing else. do not write explanations. do not type commands unless I instruct you to do so. when I need to tell you something in english, I will do so by putting text inside curly brackets {like this}. My first command is console.log("Hello World");

![Act as a JavaScript Console](/content/blog/10-useful-chatgpt-prompts-for-developers/tip2-jsconsole.png "Act as a JavaScript Console")

**\#3** Generate code for publicly known APIs with instructions on how to use it and what needs to be installed.

\
**Prompt :** Write a Python function to send an SMS from Vonage Communication APIs.

![Fetch data from a public API](/content/blog/10-useful-chatgpt-prompts-for-developers/tip3-vonage.png "Fetch data from a public API")

**\#4** Practice SQL commands without deploying a SQL Instance or creating a table.

**Prompt ([Author Credit](https://github.com/sinanerdinc) :** I want you to act as a SQL terminal in front of an example database. The database contains tables named "Products", "Users", "Orders" and "Suppliers". I will type queries and you will reply with what the terminal would show. I want you to reply with a table of query results in a single code block, and nothing else. Do not write explanations. Do not type commands unless I instruct you to do so. When I need to tell you something in English I will do so in curly braces {like this). My first command is 'SELECT TOP 10 * FROM Products ORDER BY Id DESC'.

![Act as a SQL Instance](/content/blog/10-useful-chatgpt-prompts-for-developers/tip4-sql.png "Act as a SQL Instance")

**\#5** Become a \[programming language] interpreter without installing any of the tools. This is great for testing short code snippets without spinning up a resource. 

**Prompt ([Author Credit](https://github.com/akireee)) :** I want you to act like a Python interpreter. I will give you Python code, and you will execute it. Do not provide any explanations. Do not respond with anything except the output of the code. The first code is: "print('hello world!')"

![Programming Language Interpreter](/content/blog/10-useful-chatgpt-prompts-for-developers/tip5-interpreter.png "Programming Language Interpreter")

**\#6** Create Regular Expressions tailored to your specific requirements.

\
**Prompt ([Author Credit](https://github.com/ersinyilmaz)) :** I want you to act as a regex generator. Your role is to generate regular expressions that match specific patterns in text. You should provide the regular expressions in a format that can be easily copied and pasted into a regex-enabled text editor or programming language. Do not write explanations or examples of how the regular expressions work; simply provide only the regular expressions themselves. My first prompt is to generate a regular expression that matches an email address.

![Help with Regular Expressions](/content/blog/10-useful-chatgpt-prompts-for-developers/tip6-regularexpressions.png "Help with Regular Expressions")

**\#7** Find bugs in short code snippets. 

**Prompt :** Find bugs in the following JavaScript code: 

```javascript
function add(a, b} {
    return a + b;
}
```

![Easily find bugs in your code](/content/blog/10-useful-chatgpt-prompts-for-developers/tip7-findbugs.png "Easily find bugs in your code")

**\#8** Generate architectural diagrams for your project requirements.

\
**Prompt ([Author Credit](https://github.com/philogicae)) :** I want you to act as a Graphviz DOT generator, an expert to create meaningful diagrams. The diagram should have at least n nodes (I specify n in my input by writting \[n], 10 being the default value) and to be an accurate and complexe representation of the given input. Each node is indexed by a number to reduce the size of the output, should not include any styling, and with layout=neato, overlap=false, node \[shape=rectangle] as parameters. The code should be valid, bugless and returned on a single line, without any explanation. Provide a clear and organized diagram, the relationships between the nodes have to make sense for an expert of that input. My first diagram is: "The water cycle \[8]".

![Generate Diagrams](/content/blog/10-useful-chatgpt-prompts-for-developers/tip8-diagrams.png "Generate Diagrams")

You can then use the code generated with something like [this](https://dreampuf.github.io/GraphvizOnline/) to generate the image and export it to your computer. 

![Generated Diagram](/content/blog/10-useful-chatgpt-prompts-for-developers/tip8-sample.png "Generated Diagram")

**\#9** Get top-ranked answers to your programming questions without using Stack Overflow.

**Prompt ([Author Credit](https://github.com/5HT2)) :** I want you to act as a Stack Overflow post. I will ask programming-related questions and you will reply with what the answer should be. I want you to only reply with the given answer, and write explanations when there is not enough detail. do not write explanations. When I need to tell you something in English, I will do so by putting text inside curly brackets {like this}. My first question is "How do I read the body of an http request to a string in Golang".

![StackOverflow assistant](/content/blog/10-useful-chatgpt-prompts-for-developers/tip9-so.png "StackOverflow assistant")

**\#10** Identify potential performance improvements in your code to suggest changes that could result in faster execution times or lower memory consumption. 

**Prompt :** Optimize the following JavaScript code: 

```javascript
codeBlock='''function example() {
  var startTime = new Date().getTime();
  // ... code ...
  var endTime = new Date().getTime();
  return (endTime - startTime) / 1000;
}'''
```

![Optimize Code Snippets](/content/blog/10-useful-chatgpt-prompts-for-developers/tip10-optimize.png "Optimize Code Snippets")

**BONUS!** I'm sure all of us have been stuck with some Git problems before. Use ChatGPT to provide guidance on how to overcome it. 

**Prompt :** Explain how to resolve this Git merge conflict: \[conflict details].”

```text
$ git status
> # On branch branch-b
> # You have unmerged paths.
> #   (fix conflicts and run "git commit")
> #
> # Unmerged paths:
> #   (use "git add ..." to mark resolution)
> #
> # both modified:      styleguide.md
> #
> no changes added to commit (use "git add" and/or "git commit -a")
```

![Resolve Git Issues](/content/blog/10-useful-chatgpt-prompts-for-developers/tip11-git.png "Resolve Git Issues")

## Wrap-up

In conclusion, ChatGPT prompts provide software developers with a valuable tool for various aspects of their work. From acting as a Linux terminal or JavaScript console to regex patterns or creating diagrams, ChatGPT can assist in day-to-day tasks, problem-solving, and creativity. ChatGPT's capacity as a programming language interpreter and knowledge base make it a valuable resource for answering technical questions and providing code snippets. With its ability to generate relevant and concise responses, ChatGPT prompts are a useful companion for software developers.

Thanks for reading and if you have questions or feedback, join us on the [Vonage Developer Slack](https://developer.vonage.com/community/slack) or send me a Tweet on [Twitter](https://twitter.com/mbcrump), and I will get back to you.