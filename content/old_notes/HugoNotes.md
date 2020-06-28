---
author: author
date: 2017-03-14T14:42:39-04:00
description: description
keywords:
- key
- words
tags:
title: Hugo Notes
topics:
- topic 1
type: note
---
---
### Hugo Reference

[Hugo](https://gohugo.io/overview/introduction/) is a static HTML Site generator. It takes directories of markdown and compiles them into linked HTML websites. 
#### Setup
1. Install Hugo 
  - **a)** Install the binary from apt

        ``` sh
        $ sudo apt install hugo 
        ```
  - **b)** Install through [go](../GoNotes)

        ``` sh
        $ go get -u -v github.com/spf13/hugo
        ```


#### Commands

1. Create new content

     ``` sh 
     $ hugo new <path>/<thing>.md
     ```


2. Serve Site Locally

    ``` sh 
    $ hugo server
    ```

#### Deploying to Github

Using this link: http://gohugo.io/tutorials/github-pages-blog/

``` sh
$ ./deploy.sh
```

##### Adding Theme
To get a theme, do git submodule add <theme.git> 

``` sh 
$ cd themes
$ git submodule add <Theme-URL>
```

##### Cloning 
To clone, submodules must also be cloned. To do this, use the following command

``` sh 
$ git clone <url>  --recursive
```
*Or* after cloning normally

``` sh 
$ git submodule update themes (or other submodule directory)
```

Must checkout the master branch for the ./public branch as well

**NOTE** Must make a ssh key for the computer that is pushing to github. 

https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/

