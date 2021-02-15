---
title: Creating your online resume for free with JSON resume
category: open-source
tags:
- Github actions
- netlify
- resume
excerpt: A resume is a document that showcases your career achievements and skills.
  It is an entry ticket to get you a job opportunity. Find out how you can create
  your online resume and that for free using JSON resume in this blog post.
---

A resume is a document that showcases your career achievements and skills. Until recently(2 years ago), I used to create a word document and type in my resume and then convert it to pdf. In the last 2 years, I changed my computer, os and I am not able to edit the Microsoft word document in the Linux/Mac computer anymore. All that I wanted is to have my resume available in a website online and be able to update it without worrying about OS and softwares. But creating an online resume needs hosting and a bit of knowledge of HTML. Since I don't have too much exposure on HTML and CSS, I wanted to find an out of the box solution

## Finding tools to build resume
### Resume builder
[JSON resume](https://jsonresume.org/) is an open-source tool that will let you maintain resume in a json format. There are many amazing free [themes](https://jsonresume.org/themes/) that you can use to change the look and feel of your resume too. Json resume also lets you import your profile from LinkedIn and creates your resume json using a [chrome plugin](https://jsonresume.org/getting-started/).

### Source control & Continuous deployment
Then I came up with the idea to maintain my resume in github. It lets me maintain my resume json in Git and all I need is a web browser to edit my resume. Github provides many things free of cost. Github Actions is one among them. Github actions specify a workflow with numerous actions to build your project. Learn more about Github actions in my blog post [here]({% post_url 2020-11-28-github-actions %}). I decided to use Github actions to build my resume.html using node.js.

### Hosting
I learnt about [Netlify](https://www.netlify.com/), when a collegaue showcased his side project with a netlify url. Being free and provide many web solutions out of the box, I wanted my resume to be hosted on Netlify. Netlify also lets you provide [your own domain](https://docs.netlify.com/domains-https/custom-domains/) name instead of the \*.netlify.app url.

## Building the resume
### Creating content and build resume.json
Spend some time in creating your resume, that explains the work you have done and skills you have acquired for each role you hold. I maintained this info in my LinkedIn profile and installed the Google chrome plugin '[Json Resume Exporter](https://chrome.google.com/webstore/detail/json-resume-exporter/caobgmmcpklomkcckaenhjlokpmfbdec)' to create the json file needed by jsonresume. You are also free to create the resume json using the [sample](https://gist.github.com/thomasdavis/c9dcfa1b37dec07fb2ee7f36d7278105) provided by the website.
### Setup the Github repository
Create a Github account and add the resume.json created in the above step.
### Setup Netlify account
Create a new Netlify account if you don't have one already. Usually Netlify needs a Github/Gitlab/Bitbucket account to create an account. Once you have created the account, you need to get an API token as mentioned with this [link](https://app.netlify.com/user/applications#personal-access-tokens). Copy the token and add it as a [secret](https://docs.github.com/en/actions/reference/encrypted-secrets#creating-encrypted-secrets-for-a-repository) to the Github repository created in the previous step.  Now create a [new site](https://www.netlify.com/blog/2016/10/27/a-step-by-step-guide-deploying-a-static-site-or-single-page-app/) in Netlify. Disable the default builds via Netlify as we will be using Github actions to deploy our resume. The zip file method as detailed [here](https://docs.netlify.com/api/get-started/#zip-file-method) will be used in this post for deploying from Github repo.
### Setup Github action to build resume on git push
Now add a workflow YAML file in the repository under the directory `.github/workflows` and add the following:

```yaml
# This workflow will do a clean install of node dependencies, build the resume and deploy to netlify

name: Build & Deploy resume to Netlify

on:
  push:
    branches: [ main ]
    
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Install Node.js
      uses: actions/setup-node@v1
      with:
        node-version: 14.x
    - name: Install resume-cli
      run: |
        npm install -g resume-cli
        npm install jsonresume-theme-elegant 
    - name: Build resume
      run: | 
        mkdir dist
        resume export dist/index.html --theme elegant
    - name: Deploy resume to netlify
      run: |
        zip -r resume.zip dist
        curl -H "Content-Type: application/zip" -H "Authorization: Bearer ${{ secrets.NETLIFY_TOKEN }}" --data-binary "@resume.zip" https://api.netlify.com/api/v1/sites/<your-netlify-site-name>.netlify.app/deploys
```

In the first step, our repository is checked into the ubuntu machine running the Github action. The second action installs Node 14. The third step installs resume cli and the theme `elegant` I am using for my resume website. The fourth step creates a new directory called `dist` and use resume cli to create html files from the resume json file. In the fifth step, we are creating the zip file using the `dist`  directory and uploads the zip to netlify. Note that `NETLIFY_TOKEN` is the api token generated in netlify and added as a secret in the Github repo. In the above yaml replace `<your-netlify-site-name>` with the name of your site in Netlify. From now on, whenever you modify your resume json, the website gets updated automatically. 

### Using custom domain
If you already own or purchase a new domain name, you can use the domain name instead of the netlify url to access your website. The process of adding a custom domain is detailed [here](https://docs.netlify.com/domains-https/custom-domains/). 

I will update the post if I find any better options to build the resume. Feel free to post a comment if you find anything not working or express your thoughts!! Thanks for reading.
