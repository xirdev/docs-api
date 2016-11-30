# Getting Started Guide - XirSys v3 Beta

Welcome to the XirSys v3 Beta. This new release brings a whole host of new and exciting features to power your next distributed application, including TURN \/ STUN server support, distributed messaging, real-time flexible data storage and more.

This document has been written to provide a quick "getting started" guide in a few short steps. By following this guide, you will quickly have a working video application running in your browser and connected to your XirSys v3 Beta account.

## First Step

For this guide, we're going to use the XirSys XSDK framework repository. You can view the code for this framework at:

```bash
https://github.com/xirdev/xsdk.js
```

Here, however, we'll be using the compiled version of this framework which can be found at:

```bash
http://cdn.xirsys.com/xsdk
```

We'll start by creating a new HTML file on your machine. You can put this wherever you feel best. For this example, we won't need a development web server running, but you're free to do so if you wish.

In your HTML file, copy the following markup:

```html
<DOCTYPE>
<html>
  <head>
    <title>Xirsys XSDK: WebRTC Example</title>
    <link rel="stylesheet" type="text/css" href="//cdn.xirsys.com/css/1.0.0/xsdk.css"></link>
    <script src="//cdn.xirsys.com/libs/1.0.0/xsdk.js"></script>
    <script>
      <!-- your code will go here //-->
    </script>
  </head>
  <body>
  </body>
</html>
```

This file simply provides the bare bones markup necessary for any HTML page, complete with tags to load in the XSDK CSS and JavaScript code necessary to power our application.

## Second Step

The next step will be to call the XSDK library and pass it your account details. This will fire the necessary code to build the video application and connect it to your XirSys account.

Where is says `<!-- your code will go here -->` in the HTML file you just created, remove that line and enter the following code:

```js
new $xirsys.quickstart({
  ident : '',
  secret : '',
  domain : '',
  secure : 1
});
```

If you now open the HTML file in a browser, you should see that it now displays an application like that displayed below.

![](/assets/webrtc-app.gif)

However, this application won't yet work until we provide some more detail.

## Third Step

To acquire these details, we'll need to login to the XirSys dashboard. Go ahead and do this by navigating to the following URL:

```
https://ws.xirsys.com/dashboard/signin
```

Once logged in you should see the screen below:

![](/assets/dashboard-1.gif)

Two of the values we need are right here. The first is the username at the top of the page, depicted by the word `myuser`. We call this the `ident`. Go ahead and copy that word to the ident variable in the page.

The second value here is the `API Token` value, represented by the long list of characters. You can go ahead and put that in the HTML page, also.

The remaining `domain` value will need a bit more effort, as the XirSys platform will require that you create this value in the dashboard in order to expose it to your application.

In the dashboard, the domain value is represented as `Channels`. It represents a namespace, which is a way to tag a room or group of data within the XirSys platform so that you can reference it later.

We can create a new domain by simply typing it in the field provided where it says `Click to Add Channel` and then clicking the `+` button to the right of the field. The field will only accept a limited number of valid characters, including upper and lower case letters, numbers, hyphons, periods and underscores. Don't worry, though. If you type anything unsupported, it will tell you.

Once you have created your Channel, go ahead and put that same value you just entered into the `domain` variable in the HTML page.

Your HTML page should now look something like this:

```html
<DOCTYPE>
<html>
  <head>
    <title>Xirsys XSDK: WebRTC Example</title>
    <link rel="stylesheet" type="text/css" href="//cdn.xirsys.com/css/1.0.0/xsdk.css"></link>
    <script src="//cdn.xirsys.com/libs/1.0.0/xsdk.js"></script>
    <script>
      new $xirsys.quickstart({
        ident : 'myuser',
        secret : '1d484d60-6af4-1234-5678-abc123456789',
        domain : 'some-domain.com',
        secure : 1
      });
    </script>
  </head>
  <body>
  </body>
</html>
```

If you go ahead and run two copies of the HTML page in your browser, you should now be able to get one copy to connect to the other.

