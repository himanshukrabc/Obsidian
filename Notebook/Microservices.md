### Web Application

- Portablility ⇒ hosted on central server, can be accessed from every user without installation.

- Ease of Updates ⇒ Need to download and install(For Normal Apps)

- Deployment ⇒ Need to deal with version control and different sys architecture.(For Normal Apps)

- Maintainance ⇒ Changes require redeployment on every system(for normal apps).for web apps redeploy on the server side only.

- Cost ⇒ Multiple apps for multiple systems.

  

### SPA(Single Page Applications)

A Single Page Application is a web application in which the majority of interactions are handled on the client without the need to reach a server with a goal of providing a more fluid user experience

AJAX ⇒ Asynchronous Javascript and XML ⇒ Why SPA?  
It is a pattern which states that on reload most of the content remains same. Thus we need to just update the view instead of reloading the entire content.

To make it fast ⇒

- minimize CSS and scripts

- Delay JS execution

- Handle static files as a part of CDN

- Resources can be cached.

It represents an approach to build web applications.

- Navigation is resolved on the client side.

- Source code is either loaded initially or dynamically without reloading the page.

- Server calls are asynchronous.

- UI is predominantly built on the client.

  

Load distribution ⇒ leverages client compute capability.

SEO ⇒ meta tags are used by search engines to recommend websites. However, for SPAs, there is a single webpage which caters multiple needs which makes it difficult for Web Crawlers to get this data.

  

Predominantly HTML Output ⇒ What if we just want data?

### SOAP

Simple Object Access Protocol

Lightweight XML Protocol for exchanging typed information.

RPC ⇒ Remote Procedure Call.

### Web Server

When a server understands http, it is known as a web server.

### XML

it is markup language used to store and transport data.,

XML Schema ⇒ how the document needs to be structured.

WSDL ⇒ **an XML notation for describing a web service**. A WSDL definition tells a client how to compose a web service request and describes the interface that is provided by the web service provider.

  

### AJAX

1. HTML, CSS

1. DOM ⇒ dynamically display content

1. XMLHttpRequest Object ⇒ provided by the browser to recieve data asynchronously.

Requests are made via this object. Requires a JS call and returns XML data.  
This also facilitates asychronous requests.

  

### REST

REpresentational State Transfer

**ATOM ⇒ is used for RSS feeds.**