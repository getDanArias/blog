---
layout: post_extend
title: "Top 6 Ways to Troubleshoot Webtasks: Number 5 Will Shock You"
description: ""
longdescription: ""
date: 2018-03-05 08:00
category: Extend, Techincal, Webtasks
press_release: false
is_non-tech: false
author:
  name: "Bobby Johnson"
  url: "https://twitter.com/NotMyself"
  mail: "bobby.johnson@auth0.com"
  avatar: "https://cdn.auth0.com/website/blog/profiles/bobbyjohnson.png"
design:
  bg_color: "#3445DC"
  image: https://cdn.auth0.com/website/blog/extend/why-auth0-chose-serverless-extensibility/logo.png
tags:
- extend
- webtasks
- debugging
related:
  - 2017-08-01-auth0-webtasks-the-quickest-of-all-quick-starts
  - 2017-08-22-for-the-best-security-think-beyond-webhooks
  - 2017-10-04-securing-webtasks-part-1-shared-secret-authorization
---

**TL;DR:** A brief synopsis that includes link to a [github repo](http://www.github.com/).

---


The Webtask platform that powers [Auth0 Extend](https://auth0.com/extend/), [Webtask.io](https://webtask.io/) and [Auth0 Hooks](https://auth0.com/docs/hooks) is a very powerful system. It allows end users to provide the functionality they need by writing simple logic in JavaScript. That JavaScript is then securely sandboxed and executed with extremely low latency while protecting the security and performance of other customers and the platform itself.

Whether you are building a fun weekend project, ensuring your authorized users are members of a specific domain or offering your customers an easy way to customize your SaaS; the Webtask platform allows you to accomplish your goals without placing the burden of hosting, monitoring or scaling on you or your users.

Even with a platform this powerful, you are going to run into issues at times. A service you depend on may go down. Or a bit of logic might not consider all the possible cases that can arise during execution.

When unexpected events happen, how do you go about troubleshooting the issues? In this post, I will show you the most common ways of troubleshooting a webtask. From simple test executions to full local debugging support, we will cover it all.

## The tools built right into the editor

The webtask editor is a full-featured development environment right in your browser. The editor can be embedded directly in your SaaS product and customized to look and feel just like a part of your user interface. 

Auth0 Hooks uses the editor to offer templates tailored to the customization point. During the Client Credentials Exchange event, webtasks give access to the calling client, the requested scope, and audience that allows for customization of the generated access token.

Webtask.io uses all the editor features available to give us as close to a local development experience as possible. It is a snap to add references to our favorite NPM modules, set up a task to run on a schedule or synchronize our code with a GitHub repository.

Auth0 Extend customers can use any of these built-in features or create their own that plug right into the editor.

### Watching execution with the Logs panel

The simplest way to troubleshoot a webtask is to monitor its execution with the Logs panel. The Logs panel is available by default in both Auth0 Hooks and on Webtask.io.

When opened the Logs panel will connect to a live stream of events for our webtask. We can execute the webtask and see the execution happen in the log stream.

Let's give it a shot using Webtask.io. 

- Click on this link to launch the editor at [https://webtask.io/make](https://webtask.io/make)
- Login with the identity provider of your choice
- Click the **Create a new one** link located in the middle of the page
- In the Create New modal select the **Create empty** option
- Name the webtask `log-panel`
- Click the **Save** button

We now have a simple webtask we can use to experiment with the Logs panel. To access it, look for the Logs icon in the upper right-hand corner of the editor. We can also access it using the keyboard shortcut CMD-L.

The Logs panel has a simple interface giving us options to close, search, auto scroll and clear the log. We also see a message letting us know that we have successfully connected to the log stream.

In the footer of the editor, we find the url for our webtask along with a copy icon.

- Copy the url.
- Paste it into a new browser window.

Look at the Logs panel, and you will see information about the execution of the webtask. We can see that a request for webtask execution was received and it was completed successfully returning an HTTP status of 200 in about 100 milliseconds.

- Tick the autoscroll box
- Return to the browser we used to execute the webtask
- Execute the task a few more times by hitting refresh

Look at the Logs panel again and notice that the log is staying in sync with the latest messages. We can search the text for completed webtask executions to ensure successful HTTP 200 responses easily.

- Click the search box in the Logs panel
- Enter the text `finished webtask request`

The log messages are filtered to only the lines containing the search text where we can see that all the executions completed successfully.

We can easily write our messages to the log using the methods we are familiar with on the console object.

- Modify the code using the example below
- Click the **Save** button

```javascript
/**
* @param context {WebtaskContext}
*/
module.exports = function(context, cb) {
  console.log('Query String Values: ', JSON.stringify(context.query));
  cb(null, { hello: context.query.name || 'Anonymous' });
};
```

We have modified the code to log out the stringified version of the query object located on the webtask context.

- Click the **Save** button
- Return to the browser we used to execute the webtask
- Execute the task again
- Modify the url to add the query string `?name=bobby`
- Execute the task again

Return to the editor and look at the Logs panel. We can see both our executions along with our custom log message detailing the contents of the query string values passed during execution.

Using this simple method, we can quickly capture errors and log them into the log stream. 

Note that logs on Webtask.io are not persisted for review later, so you will need to connect via the Logs panel or the CLI and leave them connected while troubleshooting. We will see how to connect via the CLI shortly.

### Testing with sample data using the Runner panel

In the previous section, we used a separate browser window to execute our webtask. This technique was a bit tedious and doesn't work if we need to POST data to our webtask.

Luckily the editor has a built-in way to execute a webtask providing sample data via the Runner panel. It allows us to execute all the HTTP verbs; GET, POST, PUT, PATCH and DELETE. We can set header values, url parameters, and body content.

To access the Runner panel, look for the Runner icon in the upper right-hand corner of the editor. We can also access it using the keyboard shortcut CMD-ALT-R.

Let's run our existing webtask using the Runner panel.

- Click the **Url Params** section of the Runner panel
- Enter `name` for the key
- Enter `bobby` for the value
- Click the **Run** button

The Runner will execute our webtask and display the results including the HTTP status, any headers and the response returned.

If the Logs panel is still open, we can see the execution logged there as well. No need to switch back and forth between multiple windows.

But what about POST data? Let's modify the webtask code so that it pulls information from the body of the request.

- Modify the code using the example below

```javascript
/**
* @param context {WebtaskContext}
*/
module.exports = function(context, cb) {
  console.log('Post Body Values: ', JSON.stringify(context.body));
  cb(null, { hello: context.body.name || 'Anonymous' });
};
```

We have modified the code so that it uses the body object located in the webtask context instead of the query object.

Note that the icon for the Runner in the upper right-hand corner turns into a Run Again icon while the Runner panel is open.  Clicking it will cause the current webtask code to saved and then execute using the last used values in the Runner panel.

Let's give it a try.

- Click the **Run Again** icon

We have now generated a runtime error! Notice that the Runner panel is displaying execution results including an HTTP status of 500. The response also shows details about the error encountered.

In the Logs panel, we also see the same information logged to the logs stream. The error we are receiving is a TypeError with the message "Cannot read property 'name' of undefined."

By scrolling up a bit in the Logs panel, we can see our custom message letting us know that the body object of the webtask context is undefined.

The reason we are receiving this error is that we executed an HTTP GET request against a webtask that is expecting an HTTP POST.

Let's fix our test case with the Runner panel.

- Click the gear icon located near the top right of the Runner panel
- Select `POST` from the url input dropdown menu
- Delete our previous query key/value pair by clicking the Trash icon under URL Params
- Select `JSON` from the Content-Type dropdown menu in the Body editor
- Add the following JSON to the text area
- Click the **Run** button

```javascript
{
  "name": "bobby"
}
```

This execution will succeed, and the Runner panel results will display an HTTP status of 200 as well as our expected response. The custom log message has executed correctly as well.

One final note about the Runner panel functionality; located near the bottom of the panel is a label History. 

- Click the `+` icon located to the right of the History label

The Runner panel has kept a history of executions for us including HTTP verb and HTTP status code. If we click one of the entries, the panel shows details about that execution.

The Runner panel is handy when we have sample data that we know causes a runtime exception or just to use iteratively as a test while developing a webtask.

## Troubleshooting with the CLI

The tools built directly into the editor are great for troubleshooting a single webtask; especially if it is simple. But using console log statements to troubleshoot a complex webtask or a set of collaborative tasks would quickly grow tedious.

The webtask CLI offers several tools out of the box to help troubleshoot issues with webtasks. The CLI can be used in this way for Auth0 Hooks, Webtask.io or Auth0 Extend deployments.

To set up the CLI for use with Webtask.io, visit [this interactive tutorial](https://webtask.io/cli) which will walk you through the process. Auth0 Hooks users can visit [their tenant settings](https://manage.auth0.com/#/tenant/webtasks) in their management dashboard for similar instructions. And finally, Auth0 Extend customers should already have instructions for setting the CLI up to work with your cluster. If you need help, feel free to [join us in our customer Slack](https://auth0-extend.run.webtask.io/slack-signup) for assistance.

### Connecting to the log stream

Using the Logs panel in the editor, we were able to watch live executions of our webtask in real time. With the CLI we can also connect to the log stream, with the added benefit of seeing the logs for all of our webtasks. This is particularly useful when you have scoped your webtasks functions to do a single thing and build functionality up through collaborative tasks.

Connecting to the log stream is a snap with the CLI.

- Execute the command `wt logs`

The webtask CLI will take over your console session and begin streaming log messages. We can see the connection message already.

We can test this functionality out pretty easily using the webtask we created earlier in the editor.

- Click on this link to launch the editor at [https://webtask.io/make](https://webtask.io/make)
- Open the **log-panel** webtask we created earlier
- Click the Runner icon
- Click the **Run** button

We now see the same runtime exception we got earlier streaming to our console session. Let's create a new webtask that uses the requestjs node module to call our existing webtask to see how the streaming logs show output for both webtasks.

- Click the **Create New** icon located in the left-hand navigation of  the editor
- In the Create New modal select the **Create empty** option
- Name the webtask `call-log-panel`
- Click the **Save** button
- Click the **Wrench** icon located in the upper left of the editor
- Select the **NPM Modules** option from the dropdown menu
- Click the **Add Module** button
- Type `request` in the search box
- Select the `request - 2.83.0` option
- Modify the code using the example below
- Click the **Save** button

```javascript
const request = require('request');

/**
* @param context {WebtaskContext}
*/
module.exports = function(context, cb) {
  const opts = {
        method: 'POST',
        url: '<COPY YOUR URL HERE>',
        json: {name: 'bobby'}
    };

  console.log('Posting Values: ', JSON.stringify(opts.json));

  request(opts, (err, res) => {
    if(err){
      return cb(err);
    }
    return cb(null, res.body);
  });
};
```
This code uses the requestjs node module to post a JSON object to our previous webtask and then returns the response body using the callback function.

**Note:** You need to copy the url to our previous webtask into this code before it works.

We can test this using the Runner panel.

- Click the Runner icon
- Click the **Run** button

We can see the custom logs messages from both the `call-log-panel` and the `log-panel` webtasks streamed to the console. Being able to see interleaved logs like this, makes it much easier to track down which webtask is causing a problem in a set of collaborative webtasks.

**Note:** At any time you can hit `ctrl-c` to disconnect from the log stream.

### Run Locally

### Debugging with Devtool

### Debugging with Visual Studio Code

# Outro