---
title: How Open-Source Licenses Affect Your Projects and Contributions?
description: How open-source licenses affect your projects and contributions?
thumbnail: /content/blog/how-open-source-licenses-affect-your-projects-and-contributions/open-source-licenses.png
author: oleksii-borysenko
published: true
published_at: 2023-03-29T14:49:26.076Z
updated_at: 2023-03-29T14:49:26.087Z
category: inspiration
tags:
  - open-source
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

Developers often do not pay attention to licenses when using various open-source projects. As a result, we often use open-source projects, parts of a project, or functions for our applications and programs without thinking about how they might affect the derivative work. For example, do you need to retain the copyright notice from the original author? What are the requirements and obligations of different licenses? In this article, I also highlight information about popular licenses and their main features.

**Disclaimer:** Information in this blog post should not be considered legal advice. If you are seeking legal advice, then please contact a lawyer.

## What constitutes an open-source license?

First, you need to know that License terms are enforceable copyright conditions under federal copyright law and enforceable under state contract law.

A license for open-source projects is a legal contract that regulates the relationship between one or more authors and the user. It includes the following information:

* Regulation of the responsibility of the authors and contributors to the project.
* Description of the terms of use of the project or code, including use in commercial programs. 
* Definition of what can and cannot be done with the software components, the obligations, and the features of use.

## Open-source license compliance

How do we perceive information that a project is open-source? 
First, it's a project with one or more open-source licenses. Open-source license list you can find by [this link](https://opensource.org/licenses/).
So, if you start from the end, we have ready-for-use projects or proprietary software that uses open-source licenses. But you check the license and find license issues, license conflicts, or use projects without a license. 

The following problems can be affected by license conflict and issues:

* You need to replace and redevelop a part of the source code
* Negative press coverage for non-compliance
* Loss of reputation with the open-source community and customers
* Change the license of your derived work 
* Be able to disclosure corresponding source codes by request

So better to understand the open source license domain and create a related open source license policy. Or integrate license compatibility tools in your development pipeline.

# About a Few Licenses Available to Use

## Apache 2.0

[Apache 2.0](https://opensource.org/license/apache-2-0/) permissive license, last year, there was a tendency to choose Apache 2.0 for the open-source projects that are supported and developed by commercial companies or organizations.
The popularity of this license is constantly growing, not least because this type of license has been chosen as mandatory for projects by the Cloud Native Computing Foundation.
The main reason why this license is popular is unlike other permissive licenses. Apache 2.0 has clause 3 (**[3. Grant of Patent License.](https://github.com/Vonage/vonage-node-sdk/blob/3.x/LICENSE.txt#L73)**), which refers to patents. The Clause governs the disposal of patents: participants grant permission to use any patents related to their contribution. This means open-source project owners, maintainers, and users are protected from potential patent infringement lawsuits.

## Automatic license application

Some projects and resources automatically apply the specific license to code/content you created using the related project.

For example: 

The [ISC License](https://opensource.org/license/isc-license-txt/) is the default license used when setting up a new NPM package with the npm init command. 
The ISC License (ISC) is functionally identical to the MIT License, but with some wording, it seemed unnecessarily removed. MIT - is a simple permissive license; it is short, straightforward, and does not require additional NOTICE files.
[CodePens](https://codepen.io/) are automatically [MIT-licensed](https://opensource.org/license/mit/).

All content created on StackOverflow (including questions and answers) is licensed under Attribution-ShareAlike 4.0 International (CC BY-SA 4.0) and its copyleft. Using StackOverflow snippets can be a problem for your company's legal department.

Most open-source licenses contain specific obligations concerning information and documentation. For example, many open-source licenses require that the respective license text be delivered with the software when distributed.
The following examples show how companies retain the copyright notice from the original author.
The Mobile app has a separate menu item, "License", that contains license text with related copyrights notice.

Interesting facts about open-source licenses that not all know:

* You can apply multiple licenses to one project.
* "[The Unlicense](https://opensource.org/license/unlicense/)" is also a license.
* Participants who contribute to projects with an Apache license are granted permission to use any patents related to their contribution.

# Conclusion

Open-source code and projects contain all solutions you can use daily on your smartphone applications, buy tickets in city terminals, using household appliances. As developers, we use open-source projects and libraries in each sprint. A lot of us contribute to and maintain an open-source project. That is why we need to check the license and read the license text in the project you are using or interacting with. You can discover Vonage's open-source project and SDKs [here](https://github.com/Vonage)