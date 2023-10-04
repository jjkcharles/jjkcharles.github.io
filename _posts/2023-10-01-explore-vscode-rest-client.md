---
layout: post
title: VSCode REST Client
subtitle: Exploring REST Client - Finding a Breath of Fresh Air in the World of API Tools
author: jjk_charles
categories: explorations
tags: api vscode-plugins REST
---

Due to the popularity of [Postman](https://www.postman.com/) and it having been very accessible through browser extenstion ([in the past](https://blog.postman.com/goodbye-postman-chrome-app/)), it has become the defacto standard API client within my organization. While I can't change the choice of API tools used within my teams, as they largely rely on workflows relating to passing around collections etc. I have been for a long time wanting to try out other tools to see if there are better alternatives out there.

I have in the past tried multiple tools some which are below,
* [Insomnia](https://insomnia.rest/)
* [Thunder Client](https://www.thunderclient.com/)
* [Nightingale](https://nightingale.rest/)

One common thing about it all is the user interface remains somewhat similar to one another - you have collections, requests, tabs to shuffle across APIs, variables for customizing dataflow. After trying them out, none of them gave me a feeling of wanting to switch over to them.

And, to be clear I am purely interested in capabilities that allow accessing APIs alone and I don't heavily rely on Postman for API Testing, API Documentation (for this, I use [SwaggerHub](https://swagger.io/tools/swaggerhub/)), Mocking etc.


## The Problem
One of the gripe I have over Postman and others I have tried out over the years, is the need to constantly switch tabs and enter data across multiple places when I want to trigger requests to multiple endpoint under the same/similar context.

Imagine you have multiple endpoints that share different level of details about your online order, and all of them take order number as the input, with some of them taking additional input parameters on top of order number. The way most tools handle inputs is through request-specific variables (or) through collection/environment variables. While setting a collection/environment variable does achieve setting the variable once and having it cascaded across different requests it is still an hassle to switch back and forth between tabs anytime data needs to be modified.


## REST Client

Recently I came across [REST Client](https://github.com/Huachao/vscode-restclient) (which is actually an [extension for VSCode](https://marketplace.visualstudio.com/items?itemName=humao.rest-client))

### Installation
Assuming you already have VSCode, to install the extension search for `humao.rest-client` in VSCode.

### Features
REST Client offers ability to access REST APIs, gRPC and GraphQL based APIs. Below are few of the core capabilities,

* Run Http requests and visualize results in a different pane
* Save response to disk
* Access history for API Requests made & use them to initiate a Request
* Fold/Unfold API Response that is in JSON or XML format
* Environments & Variables support

### Standout Capabilities
Below are few capabilities, that stood out to me,
#### 1. Single text file that can contain all your API definition
The text file where you put in all your Requests can follow [HTTP standards](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html), with each individual Requests separated our by `###` in their own line. This is a great alternative to having N number of tabs depending on the number of endpoints you are dealing with. 
#### 2. Support for file-scoped variable
This is another benfit to having all the Requests defined in a single file. Where any [variables defined with a File-scope](https://github.com/Huachao/vscode-restclient#file-variables) will be available for use across all the Requests. For instance, going by the example mentioned above - if you have 5 endpoints all requiring order number as input, you could define the order number variable once and refer it in all the requests; invoking the endpoints for a different order number is as simple as updating one single variable which propagates it across all the endpoints. 
#### 3. Split-pane view for api-definitions and viewing results. This allows you to work on multiple API endpoints without switching tabs/screens
Results of the current Request show in a split-editor view, so there is no toggling between windows/tabs even for accessing the response headers. Also, the status bar shows useful information like breakdown of time taken & breakdown of response size.
#### 4. Support for accessing JSON/XML response/headers for other API requests that you have already made
This is one of the great things where, lets say you have two interlinked API endpoints, where you need to pull data points from one's Response and feed it into the Request of another. There is no need to copy/paste those across Reqests OR setup "test scripts" to copy response values into a Environment/Collection variable for accessing it outside the Request context.

Take below example, if one of the endpoints returns all objects and in a different endpoint you want to pass in the "id" of first item returned to get more details, you can have it setup as below,

{% raw %}
```http
# File Variables #
@baseUrl = api.restful-api.dev

###

# List of objects by ids
// @name allObjects
GET https://{{baseUrl}}/objects
    ?id=ff8081818ad150c5018ade970fab1017
    &id=13

# Single object
GET https://{{baseUrl}}/objects
    ?id={{allObjects.response.body.$[0].id}}
```
{% endraw %}

{% raw %}
>Notice the usage of `// @name allObjects` which assigns a name to that Request, and *once it is run*, the response (including headers) can be accessed in a different Request within the same file. It is referenced in another Request with the syntax - `{{allObjects.response.body.$[0].id}}`
{% endraw %}

#### 5. Support for prompting user input when running a Request
Rather than provide all the inputs upfront, if you want to prompt user to key-in data at the time of triggering the request, that is possible too. You do it as below,

{% raw %}
```http
# File Variables #
@baseUrl = api.restful-api.dev

###

# Single object
// @prompt Id Enter the ID value you want to retrieve details for:
GET https://{{baseUrl}}/objects
    ?id={{Id}}

###
```
{% endraw %}

## What next?
With a new kind of user experience, this extension definitely does feel like a breath a fresh air, in a space crowded with multiple tools offering kind of very similar experience. Overall, this is a well rounded offering and covers majority of features anyone would expect to get out of an API client. Like I mentioned earlier, this isn't going to become a replacement for Postman especially at Work, as it would then entail changes to  workflows my teams are already adopting to. 

But, I am going to start using it for non-trivial API specs I have been maintaining for my personal hobby projects.

After getting a feel of what this extension offers, I was looking to see if there are similar kind of approaches used by other such tools. And, I indeed stumbled upon one - which is probably little newer that REST Client, and thus may be not as popular as this one.

[httpYac](https://httpyac.github.io/) is the one I found. This though is very similar to REST Client in lots of ways, from the looks of it, it does seem way more feature-rich than REST Client. With pretty good additions like plugin system, Scripting support (Javascript) and the ability for it to be plugged into CI/CD pipelines. 

Pretty eager to give this a try sometime soon!