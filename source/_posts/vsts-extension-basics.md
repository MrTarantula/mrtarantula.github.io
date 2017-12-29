title: VSTS Extension Basics
date: 2017-12-28
category: VSTS
tags: ["vsts", "tfs"]
# cover: /images/awesome-bg.jpg  # <= remember to add this line in your post
---

VSTS is a powerful set of tools for managing people, code, deployments, lifecycle, and just about anything you can imagine thanks to its extensibility. Need to replace text in a file at build or release? <!-- more -->Need a new input control type in the work item form? Need to [print some work items](https://marketplace.visualstudio.com/items?itemName=mrtarantula.wiprint)? There's an extension for it.

But what if you need to do something and can't find an extension for it? Just build it! It's easier than you might think. There are many good resources out there, but a lot of them are sometimes lacking, out-of-date, and generally not noob-friendly. I'm hoping I can fix that in this article. 

## Anatomy of an Extension

VSTS extensions are basically just web apps, with some special hooks into VSTS, supplied by `VSS.SDK.js`. Add a reference to it in an HTML page, initialize it, and you're nearly ready. The extension also needs to be packaged by `tfx-cli`, using a manifest file called [`vss-extension.json`](https://docs.microsoft.com/en-us/vsts/extend/develop/manifest).

## `vss-extension.json` - The Good Parts

The `vss-extension.json` file may look scary at first, but there are only a few parts that how your extension works and where it is displayed.

### Scopes

[Scopes](https://docs.microsoft.com/en-us/vsts/extend/develop/manifest#scopes) provide access to specific areas of VSTS for your extension. If you need to modify work items as part of your extension, add the `vso.work_write` scope. If you only need to read the information, only add `vso.work`. You can add as many scopes as you'd like to the array, but of course it is best to limit it to what you actually use.

### Targets

[Targets](https://docs.microsoft.com/en-us/vsts/extend/develop/manifest#installation-targets) define where your extension can be installed. This is useful if you want to limit your extension to being installed only on VSTS, or specific versions of TFS.

### Contributions
[Contributions](https://docs.microsoft.com/en-us/vsts/extend/develop/contributions-overview) describe the actual content of your extension:

```json
{
    ...
    "contributions": [
        {
            "id": "hub-barebones",
            "type": "ms.vss-web.hub",
            "description": "Adds a thing to the Work hub group.",
            "targets": [
                "ms.vss-work-web.work-hub-group"
            ],
            "properties": {
                "name": "A Thing",
                "order": 99,
                "uri": "index.html"
            }
        }
    ]
    ...
}
```

- `type` describes what we are adding: a hub
- `targets[]` describes where we are adding the new hub: the Work hub group
- `properties` contains information used for displaying the hub in VSTS
    - `name` is the title that shows in the top hub menu
    - `order` controls where on the hub menu the name will be displayed, in relation to other hubs
    - `uri` is the html file to be loaded for the extension. `VSS.SDK.js` must be initialized in this file or the extension will fail to load.

## Putting it All Together

Now that we know the basics, let's create an extension.

### Prerequisites

- Install [Node](https://nodejs.org/en/download/)
- Install tfs-cli - `npm install -g tfs-cli`
- Register as a [VSTS Marketplace publisher](http://aka.ms/vsmarketplace-manage)
- Download VSS SDK and copy `VSS.SDK.js` into your extension's folder

>If you are experienced with Node you can also install the VSS SDK with `npm install vss-web-extension-sdk`. This article is meant as an introduction to show how the individual pieces work together.

### index.html

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Barebones Hub</title>
        <!-- Include VSS SDK -->
        <script src="VSS.SDK.js"></script>
    </head>
    <body>
        <h1>I don't do anything!</h1>
        
        <!-- Initialize VSS SDK -->
        <script type="text/javascript">
            VSS.init();
            VSS.notifyLoadSucceeded();
        </script>
    </body>
</html>
```

### vss-extension.json

```json
{
    "manifestVersion": 1,
    "id": "barebones",
    "version": "0.1.0",
    "name": "VSTS Barebones Hub",
    "publisher": "YOURPUBLISHER",
    "public": false,
    "scopes": [],
    "targets": [{
        "id": "Microsoft.VisualStudio.Services"
    }],
    "files": [{
            "path": "index.html",
            "addressable": true
        },
        {
            "path": "VSS.SDK.js",
            "addressable": true
        }
    ],
    "contributions": [{
        "id": "hub-barebones",
        "type": "ms.vss-web.hub",
        "description": "Adds a thing to the Work hub group.",
        "targets": [
            "ms.vss-work-web.work-hub-group"
        ],
        "properties": {
            "name": "A Thing",
            "order": 99,
            "uri": "index.html"
        }
    }]
}
```

### Package Your Extension

From your extension folder, run `tfx extension create`. After a second or two, a `vsix` file will appear in your folder. Upload this file to the [VSTS Marketplace](https://marketplace.visualstudio.com/manage) or TFS to view your new hub.

![A Thing VSTS hub](images/thinghub.png)

Pretty neat! Source code is [here](https://github.com/mrtarantula/vsts-barebones-hub). In a future article I will discuss how to use a [Yeoman](http://yeoman.io) generator to set up an extension with TypeScript, Webpack, and npm libraries.