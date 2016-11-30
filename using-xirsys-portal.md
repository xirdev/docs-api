# ** Introduction **

The Xirsys User Interface is built on top of the Xirsys API and can be used to easily create connection points to our ICE/STUN and TURN services and many other communication services using our or your own applications.

The Get Started section of the UI takes you to a quick tutorial that gets you up and running immediately using just a bit of code in an HTML document. The Getting started is only a part of the library of helpful documents our service comes with. See the "docs" section on our website to see more.

![](/assets/lft_menu_getstarted.png)

The Service section is where you'll find the tools to connect your applications using the Xirsys API or use one of our many supported SDKs including Simple Webrtc and more. Here you can also manage all your connection points and manage your secret tokens as well.

![](/assets/service.png)

The Analytics section provides basic to advanced analytics which can be used to decipher how your customers user your applications, or to simply see your account usage per date.

![](/assets/analytics.png)

In the Settings page you can manage your account membership for updating your information and upgrading your account when you need that extra room to grow. Users migrating to V3 from our V2 platform can use the import section to migrate their old connection paths over.

![](/assets/settings.png)

You can find more about our API and how to use it in **Introduction** to the API.

---

# ** Signup **

Signup can be free for those who are testing or you can choose from any one of our pre-packaged membership plans that come with a Service Level Agreement (SLA) and much more bandwidth limits.

**NOTE:** Beta users are assigned a special limited FREE account.

1. Go to the Signup page from the Xirsys UI Dashboard: [https://portal.xirsys.com/dashboard/signup](https://portal.xirsys.com/dashboard/signup)

2. Enter a unique username which will be used for authentication to the API. Your username must begin with a letter, be longer than 3 characters and must not contain any special symbols other than an underscore or dash ("_", "-").

3. Enter a secure password, something you can remember is best.

4. Enter your email and company name. Please make sure your email is valid, we will use this for verification and to contact you regarding your account only should we need to.

5. Finally please read our Terms of Service and Privacy Policy and select the checkbox letting us know that you understand them.

6. Now click on the "Create Account" button and you should be let into your new account instantly.

    ![](/assets/signup.png)

We will send a helpful confirmation letter to the email you provided above. Please check your junk folder on your email or contact us if you do not receive this letter, or if you have any issues signing up (experts@xirsys.com).

Once you've created your account you can simply go to [https://ws.xirsys.com/dashboard/signin](https://portal.xirsys.com/dashboard/signin) to sign into it anytime.

   ![](/assets/signin.png)

---

# ** Migrating to Xirsys V3 from V2 **

Once you've created a new Xirsys account on V3 you can import all your old connection channels from version 2 into your new account on version 3.

**NOTE:** Importing your v2 channels, will completely override any V3 channels you may have currently including your secret token. 

1. Login to your V3 account and click on Settings in the left main menu.

    ![](/assets/lft_menu_settings.png)
    
2. Copy the secret token from your Old v2 account. At the bottom of the settings panel in the Xirsys V3 Interface paste your old V2 secret token in the "Import Past Account" text field entry.

    ![](/assets/import_v2.png)
    
3. Click on "Import My V2 Account" button and wait for a successful response from the UI.

Upon success, your V2 connection channels as well as your V2 secret token will be imported into your V3 account. Click on Services to see your channels and token.

---

# **Creating a Connection Channel**

To connect to your applications to the Xirsys STUN/TURN service you need a point of reference, which we call your channel. These channels help as a connection points to your account and they help segregate your applications from each other. Channels can have multiple sub channels within them, similar to having a folder with subfolders in on your computer.

Your First Channel:

1. Select Services from the left main menu.

    ![](/assets/lft_menu_services.png)

2. If this is your first channel you will be prompted to create one first.

    ![](/assets/first_channel.png)

3. Give your channel a unique name. Channel names must begin with a letter and can contain numbers but cannot have any spaces or special characters other than underscores, dashes and periods.

4. Once you've created the channel the UI will change to the management screen where you can manage your secret token, add more channels, see the connection details to your channel via the Xirsys API as well as quick start samples you can use so connect right away to your channel.

Now that you have created a channel try connecting an application to it. Follow the Quick Start ([https://portal.xirsys.com/dashboard/docs?p=quick-start](https://portal.xirsys.com/dashboard/docs?p=quick-start)) guide to create an instant application you can connect to with friends.

---

# **Connecting Using JS**

If you are building your own custom WebRTC application, then you might be looking to simply use our STUN/TURN service to test and build without too much complication of security.

In this scenario were assuming you are building in javascript and you need to connect to the Xirsys Channel you created to test things out.

1. Login to your xirsys account, and click on "Services" from the left main menu.

    ![](/assets/lft_menu_services.png)

2. Select the channel you created, or go ahead and add a new channel and select it from the channels menu.

    ![](/assets/manage_ch.png)

3. Scroll down to the "Quick Connect Example" pane and click "Javascript" from the tabs. Your channel and details should already be filled into the required paths. Select and copy the code sample and paste it into the <body> tag of an HTML page.

    ![](/assets/js_ex_tab.png)

4. Upload and run the HTML file. Upon success you should see a return JSON object with a list of UDP and TCP STUN and TURN server paths. These paths will be your connection points for your application to begin negotiating a connection between peers over WebRTC.

Keep in mind this scenario exposes your secret token and user ident which someone that has access to the html file can take. We recommend you use this for testing internally or with trusted people only. If you feel that someone has taken your information all you have to do is renew your secret token from within Services of the UI. Immediately secure your credentials using a server, and let technologies like Node.js, .NET or PHP secure your calls to your application.

For security see connecting with Node.js example.

---

# **Connecting Using Node.js**

If you have your own WebRTC application, or you are using an SDK and you want to secure your account credentials, node.js is a great way to do this. It will keep your credentials hidden from anyone looking at your source code. You can use other programming languages as well, like php, or .NET, it really depends on your needs.

1. Setup your Node.js server locally or on a machine or service.

2. Create an empty server.js file to run with your Node.js setup.

3. Login to your xirsys account, and click on "Services" from the left main menu.

    ![](/assets/lft_menu_services.png)

4. Select the channel you created, or go ahead and add a new channel and select it from the channels menu.

    ![](/assets/manage_ch.png)

5. Scroll down to the "Quick Connect Example" pane and click "Node.js" from the tabs. Your channel and details should already be filled into the required paths. Select and copy the code sample and paste it into your empty "server.js" file using a simple text editor like Sublime.

    ![](/assets/nj_ex_tab.png)

6. Run the Node.js server.

The server will immediately query the Xirsys API for an available list of STUN and TURN server paths that you can safely send down to your client applications without exposing your account credentials. You can build out or client applications with your node server and server your client files directly OR use your node server as an API of sort to just receive the Xirsys details securely.

---

# **Using Analytics**

Analytics page helps you see how much bandwidth your applications are using per date, based on the account you have. You can see within the months if you might have to update to more bandwidth or leave as is. Using our advanced analytics (coming soon) you can even understand how and when your clients are using the applications.

To use your analytics data were assuming you've created a few channels and connected to them, creating some type of usage.

1. Login to your xirsys account, and click on "Analytics" from the left main menu.

    ![](/assets/lft_menu_usage.png)

2. At the top of the Analytics section, you will see your current bandwidth usage. This is updated monthly at the moment, and shows you how close your usage is to your account limit.

    ![](/assets/usage_meter.png)

3. In the Channels section, you can choose the default "All Channels" to see the usage for the entire account. Or you can select a specific channel or subchannel to see the usage for the channel specifically.

    ![](/assets/usage_ch.png)

4. To the Right of your channels is a simple usage breakdown based off STUN and TURN connections and the usage they have. You can update the date of the report to see how your applications did in the past months. Or click Refresh if you're on the current date to see how the apps are doing as time goes by.

    ![](/assets/usage_graph.png)

5. At the bottom click on SHOW ANALYTICS which will bring in a more detailed analytic view of your usage (Alpha).

    ![](/assets/usage_adv_bt.png)

We will be updating our analytics efforts as our services grow, this usage data can be used to recommend action or to guide decision making for your applications. If you have any suggestion in regards of better information please feel free to voice your comments to experts@xirsys.com.
