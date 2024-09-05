---
layout: single
title:  "Connecting to openAI with python with the API"
date:   2023-07-16 17:45:00 +1000
tags: python OpenAI 
seo_tags: python OpenAI artificial-intelligence API
seo_title: "Connecting to openAI with python with the API"
seo_description: "How to use the OpenAI Python API to develop AI applications using your OpenAI account and API key"
excerpt: "This post describes how to use the OpenAI Python API to build AI applications. We will cover setting up your OpenAI account and API key"
---


![OpenAI logo]({{ site.url }}{{ site.baseurl }}/assets/images/openAi.png)
 

## What is OpenAI

OpenAI is an artificial intelligence research and development lab based in San Francisco, California that specializes in developing and deploying state-of-the-art natural language processing models. OpenAI provides an API (Application Programming Interface) that allows developers to easily access their powerful AI models and integrate them into their own applications.

This post describes how to use the OpenAI Python API to build AI applications. We will cover setting up your OpenAI account and API key, and then we will walk through three examples of how to use the OpenAI Python API to generate text, answer questions, and more.


## Setting up your OpenAI account and API key

Prior to using the OpenAI API you will need an API key. To do this, follow these steps:

1. Go to [the OpenAI webiste](https://platform.openai.com/account/api-keys.)
2. Create an account.
3. Select API Keys in the left-hand menu and click on Create New Key
4. Click on "Create new secret key" to create a new API key.
5. Set an environment variable called `OPENAI_API_KEY` to your API key. This allows you to use the API key without having to code it each time.


### Connecting to and using OpenAI through the API

First, install OpenAI python library

``` python
pip install openai
```

### Authenticating Your secret API Key

To authenticate your API Key, import the openai module and assign your API key to the `api_key` attribute of the module. I use the `os.getenv()` function to get the value of the `OpenAI_API_Key` environment variable (from step 5 above).

``` python
import os
import openai

api_key = os.getenv('OPENAI_API_KEY')
openai.api_key = api_key
```

Let's assume you are using a notebook. Otherwise, use the `print()` function to output to the console. When it reads the `Model.list` it will be in JSON format. You can read load this directly into a pandas dataframe as follows.

``` python
import pandas as pd

models = openai.Model.list()
models_list = pd.DataFrame(models["data"])["id"]
models_list.head(10)
```