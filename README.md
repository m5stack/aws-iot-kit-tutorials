# AWS IoT EduKit Tutorials
This repository contains the tutorial content for the [AWS IoT EduKit](https://aws.amazon.com/iot/edukit) program. To view the AWS IoT EduKit content live, visit [https://edukit.workshop.aws](https://edukit.workshop.aws). The tutorials contained in this repository are MIT licensed and free to be modified as needed. To contribute to the code or community, read our [contribution guidelines](https://github.com/aws-samples/aws-iot-edukit-tutorials/blob/main/CONTRIBUTING.md). For support or to connect and have open discourse with the community, view our [GitHub discussions](https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions).

## Repo structure
```bash
.
├── metadata.yml                      <-- Metadata file with descriptive information about the workshop
├── README.md                         <-- This instructions file
├── deck                              <-- Directory for presentation deck
├── resources                         <-- Directory for workshop resources
│   ├── code                          <-- Directory for workshop modules code
│   ├── policies                      <-- Directory for workshop modules IAM Roles and Policies
│   └── templates                     <-- Directory for workshop modules CloudFormation templates
└── workshop                          
    ├── config.toml                   <-- Hugo configuration file for the workshop website
    └── content                       <-- Markdown files for pages/steps in workshop
    └── static                        <-- Any static assets to be hosted alongside the workshop (ie. images, scripts, documents, etc)
    └── themes                        <-- AWS Style Hugo Theme (Do not edit!)
```

## What's Included
This project the following folders:

* `deck`: Future location to store presentation materials. Currently unused. 
* `resources`:  Future location to store any example code, IAM policies, or Cloudformation templates. Currently unused.
* `workshop`: This is the core workshop folder. This is generated as HTML and hosted for presentation for customers.


## Requirements to web application locally
1. [Clone this repository](https://help.github.com/articles/fork-a-repo/).
2. [Install Hugo locally](https://gohugo.io/overview/quickstart/).


# Running the web application locally
## Navigate to the `workshop` directory
All command line directions in this documentation assume you are in the `workshop` directory. Navigate there now, if you aren't there already.

```bash
cd aws-iot-edukit-tutorials/workshop
```

## Launching the website locally
Launch by using the following command:

```bash
hugo serve
```

Go to `http://localhost:1313` in your browser

You should notice three things:

1. You have a left-side **Intro** menu, containing two submenus with names equal to the `title` properties in the previously created files.
2. The home page explains how to customize it by following the instructions.
3. When you run `hugo server`, when the contents of the files change, the page automatically refreshes with the changes. Neat!

Alternatively, you can run the following command in a terminal window to tell Hugo to automatically rebuild whenever a file is changed. This can be helpful when rapidly iterating over content changes.

```bash
hugo server
```

## Adding language translations:
To add additional languages, update the config.toml with the language you're adding.
The example below demonstrates how to add French language translations. To add individual pages in French, go to the corresponding tutorial and section directory in the **workshop/content** folder and add a new file following the pattern `_index.<<language extention>>.md`. So to match the config.toml example below for french, the page filename would be **_index.fr.md**. 

```
[Languages.fr]
title = "Mon atelier AWS"
weight = 2
languageName = "Français"
```
