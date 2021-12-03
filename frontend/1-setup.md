# Setting up the frontend

{% hint style="danger" %}
This section is very work-in-progress. Please don't use it yet!
{% endhint %}

TODO:

* Bootstrap the single page app
* Add the wallet context
* Actually do the transactions

We've spent a ton of time building up our counter program.

One little problem: how would users actually use this thing?

Sure, users could build up their own transactions manually and run them after we deploy. But what we truly need is a frontend that supports our users' wallets and allows them easy access to use our counter.

We'll be using React for our frontend code here. The [official docs](https://reactjs.org/docs/getting-started.html#learn-react) are a great place to start if you're not familiar with this framework.

## Setting up the frontend

We'll use `create-react-app` to bootstrap a new single-page application.
