---
layout: post
title: "Using SparkPost as a Custom E-mail Provider with Auth0"
description: "Learn how to use the new SparkPost integration with Auth0."
longdescription: "We have recently added support for more e-mail providers to the Auth0 platform. In this short tutorial, we will learn how to enable custom e-mail providers in Auth0, and how to setup SparkPost for all Auth0 related notifications."
date: 2018-02-22 12:30
category: Auth0-based Tutorial, Integration, SparkPost
author:
  name: Sebastián Peyrott
  url: https://twitter.com/speyrott?lang=en
  mail: speyrott@auth0.com
  avatar: https://en.gravatar.com/userimage/92476393/001c9ddc5ceb9829b6aaf24f5d28502a.png?size=200
design:
  image: https://cdn.auth0.com/blog/sparkpost/logo.png
  bg_color: "#222222"
  image_size: "70%"
tags:
- email
- e-mail
- provider
- auth0
- sparkpost
related:
- 2017-10-13-how-do-i-modernize-my-legacy-system
- 2018-02-12-auth0-joins-the-google-cloud-partner-platform
- 2017-12-13-our-journey-toward-saas-customization-and-extensibility-at-auth0
---

We have recently added support for one more e-mail provider to the [Auth0](https://auth0.com/) platform: [SparkPost](https://www.sparkpost.com/). In this short tutorial, we will learn how to enable custom e-mail providers in Auth0 and how to setup SparkPost for all Auth0 related notifications. Read on!

{% include tweet_quote.html quote_text="Learn how to use SparkPost to send e-mails for your Auth0 apps!" %}

---

## Introduction
Flexibility is one of the most important parts of the Auth0 platform. In fact, it is so important to us that we even [developed a product](https://auth0.com/extend/) to help others add flexibility to their architectures. One of the many pieces that can be configured to our customers' likings is the e-mail provider. E-mail providers, in the Auth0 platform, are external services that handle the delivery of all e-mails generated by our identity platform. For example, when a new user registers his or her e-mail address during a common user + password signup process, an e-mail is sent to the user's mailbox to validate the address. On [passwordless logins](https://auth0.com/passwordless) with e-mails, an e-mail is sent to the user's registered e-mail address, etc.

These tasks, by default, are handled by our internal e-mail service. However, the built-in service has several limitations:

- Customization of the content of the e-mails is very limited.
- Rate limiting is always in place.
- All e-mails are sent from a single address.
- High bounce rates to specific destination addresses may limit your ability to send e-mails to other addresses.

These restrictions are very limiting for a production environment. For this reason, it is strongly recommended that customers choose one of the integrated e-mail providers. At the moment, there are five alternatives for sending e-mails:

- The limited built-in service
- [Custom SMTP](https://auth0.com/docs/email/providers#configure-a-custom-smtp-server-for-sending-email)
- [Mandrill](https://auth0.com/docs/email/providers#configure-mandrill-for-sending-email)
- [SendGrid](https://auth0.com/docs/email/providers#configure-sendgrid-for-sending-email)
- [SparkPost](https://auth0.com/docs/email/providers#configure-sparkpost-for-sending-email)

[Mandrill](https://www.mandrill.com/), [SendGrid](https://sendgrid.com/) and [SparkPost](https://www.sparkpost.com/) are three e-mail service providers with powerful APIs to interact with their service. Additionally, full flexibility can be achieved through the use of a [custom SMTP server](https://auth0.com/docs/email/providers#configure-a-custom-smtp-server-for-sending-email). This alternative allows you to setup any provider (or your own server) as long as it provides an SMTP interface.

In this post, we will focus specifically on [SparkPost](https://www.sparkpost.com/). Other e-mail providers, except for the generic Custom SMTP option, can be configured in a similar way.

## Step 1: Get an Auth0 Account
If you haven't done so already, register for a <a href="https://auth0.com/signup" data-amp-replace="CLIENT_ID" data-amp-addparams="anonId=CLIENT_ID(cid-scope-cookie-fallback-name)">free Auth0 account</a>. The free tier provides all that's necessary to get a small application up and running with as many as 7000 active users and unlimited logins! Of course, you can also setup your own e-mail provider. Customization options for e-mail content are limited though. But this won't be an issue for our tutorial.

Normally, after signing up, you would create a new [Client](https://auth0.com/docs/clients) using the Auth0 dashboard. Clients identify different applications that interact with the Auth0 server. In other words, if you were to develop a mobile application that authenticates users at some point, you would create a client for that application.

For this tutorial, however, we won't be using any clients, so you can simply sign up and move on to step 2.

## Step 2: Get a SparkPost Account
We will also need a SparkPost account. Go to [www.sparkpost.com](https://www.sparkpost.com/) and click on the `TRY FREE` button on the top right corner of the screen. Complete your user information and follow the steps.

At some point during the sign-up process, you will be presented with an API-key. Take note of this value and store it somewhere safe. You will not be able to get this value again from the SparkPost dashboard. Worry not! If for some reason you forget this value, you will be able to invalidate that API key. You can also create as many API keys as you want from the dashboard.

The API key is used to authorize external users of the API. In other words, this *token* is what allows external services, like Auth0, to interact with the SparkPost API. You will need to inform Auth0 of this value at some point (hint: the next step). Do note that the API key should be considered a **private** value. In other words, do not expose the API key to any services or users unless required. Any holder of the API key can interact with your SparkPost account using the level of access associated with that API key. If you take a look at the SparkPost API key creation screen you will note that there are many checkboxes:

![SparkPost API key creation](https://cdn.auth0.com/blog/sparkpost/api-key.png)

These checkboxes are the permissions associated with the API key about to be created. Any holder of the key has access to those permissions. Auth0 integration only requires a single level of access: `Transmissions: Read/Write`. `Transmissions` are a component of the SparkPost service architecture that allow sending e-mails (single and bulk e-mail operations).

## Step 3: Set API Key in the Auth0 Dashboard
This is the most important step! Now that you have both an Auth0 account and a SparkPost account along with a valid API key, it's time to tell Auth0 about it. Go to the [Auth0 Dashboard](https://manage.auth0.com/) and find the [E-mail Providers section](https://manage.auth0.com/#/emails/provider). Enable custom e-mail providers by toggling the `Use my own Email Provider` button. Now select `SparkPost`. You will be presented with the following screen:

![Auth0 SparkPost API key input](https://cdn.auth0.com/blog/sparkpost/auth0-api-key-input.png)

It really is as simple as that! Put your API key in the input box at the bottom, and put a valid e-mail address at the top. SparkPost places some limitations on the e-mail addresses that are valid for use here. These are all detailed in [SparkPost's documentation](https://developers.sparkpost.com/api/transmissions.html), however, to avoid unpleasant surprises we'll review some of these limitations here:

- To use a custom domain for sending emails, that domain must be validated once using the [SparkPost domain validation process](https://developers.sparkpost.com/api/sending-domains.html). The relevant section of SparkPost's dashboard is [this one](https://app.sparkpost.com/account/sending-domains).

- Custom domains must match **exactly** the validated domain. In other words, if your validated domain is `mail.mysuperdomain.com` you must pick an address that looks like `mymailbox@mail.mysuperdomain.com`. Variants like `mymailbox@mysuperdomain.com` or `mymailbox@mail2.mysuperdomain.com` are not valid. If you don't want the `mail` subdomain to be part of your e-mail, make sure to validate the root domain and not the subdomain using SparkPost's domain validation process. Whatever goes before the `@` is not important, though, you can pick whatever you want.

- There is a domain that can be used for tests in case you don't have a custom domain at the moment: `sparkpostbox.com`. This is known in SparkPost's documentation as the [sandbox](https://developers.sparkpost.com/api/transmissions.html#header-the-sandbox-domain). SparkPost's sandbox domain has strict limitations and should only be used to test that things work once or twice. In fact, there is a hard lifetime limit of five test e-mails from that domain. In other words, after five test e-mails, you won't be able to send any more using that domain. You can use this domain to test the integration with Auth0 but be aware that things will fail after those five e-mails have been sent. Sandbox domains are of the form `mailbox@sparkpostbox.com` where `mailbox` can be anything you want.

With all of this in mind, we might want to test things using the sandbox domain (just one mail this time!). Put a sandbox domain e-mail address in the input box at the top, like `test@sparkpostbox.com` and hit `SEND TEST EMAIL`. In a few seconds, you will receive an e-mail in your main Auth0 e-mail address. This is the e-mail that you used when you signed-up for an Auth0 account. Check it out!

## Step 4: Setup a Custom Domain in SparkPost
SparkPost's sandbox domain is very limited. For this reason, it is very important to setup a custom domain as soon as possible. Fortunately, if you already own a domain this is very simple. If you don't, getting one is out of scope for this tutorial. Fortunately, there are many tutorials on getting a domain on the internet. If you need to buy one, do that and then return to this step. For instance, you can buy one from [Google](https://domains.google/#/) or you can use [Zeit's excellent now.sh service](https://zeit.co/domains) to buy one from the console.

To access the domain verification screen, go to [SparkPost's dashboard](https://app.sparkpost.com/dashboard), then go to `ACCOUNT`, `SENDING DOMAINS` and read the `Set Up For Sending` section.

If you already have a custom domain you will need to decide what process you will use to validate it. SparkPost provides the following validation alternatives:

- Let SparkPost send an e-mail to your custom domain and then follow the link there.

![Sending a verification email](https://cdn.auth0.com/blog/sparkpost/verification-email.png)

- Put a special entry in your DNS server configuration.

![Special DNS entry](https://cdn.auth0.com/blog/sparkpost/dns-domain-verification.png)

Both alternatives are simple enough, however, if you do have the capability to receive e-mails in your domain, this is the simpler, faster option. In contrast, DNS changes may take time to propagate and require especial access credentials to your DNS service or server.

Do note that SparkPost recommends using DNS verification for your domains.

## Step 5: Handling Errors and Failures
Now that you have a fully working integration with SparkPost, it is important to know how to debug things in case anything goes wrong. There are two places to look for important clues: [Auth0 logs](https://manage.auth0.com/#/logs) and the [SparkPost reports](https://app.sparkpost.com/reports/message-events). Normally, Auth0 logs would only show failures in case the API key is invalid or SparkPost is down, for all other issues you should check SparkPost's reports.

To reach [Auth0 logs](https://manage.auth0.com/#/logs) go to the [dashboard](https://manage.auth0.com/) and then click on `Logs` on the sidebar.

![Auth0 logs](https://cdn.auth0.com/blog/sparkpost/auth0-logs.png)

To reach [SparkPost's reports](https://app.sparkpost.com/reports/message-events), go to [SparkPost's dashboard](https://app.sparkpost.com/dashboard) and then click on `REPORTS` on the side panel. Inside the reports section, the `MESSAGE EVENTS` section is the most important. You will find detailed descriptions of what is going on with each message in this section.

![SparkPost message events](https://cdn.auth0.com/blog/sparkpost/message-events.png)

Before getting worked up if things don't work, do remember two important limitations from SparkPost's service: the sandbox domain (sparkpostbox.com) hard limit of five test e-mails, and strict matching of validated custom domains.

## Conclusion
Using custom e-mails providers with Auth0 is very important for a production environment. Although Auth0's built-in service is a great test tool, it cannot match dedicated services like SparkPost. If you want great customization options, rate-limiting that suits your use case, custom send addresses and no limits with regards to bounced e-mails count, you must adopt a dedicated e-mail provider. Fortunately, integration with services like SparkPost, as we have seen in this post, is a breeze. And even then, if you don't find any of the alternatives fitting to your use case, you can still fall back to a custom SMTP solution. Flexibility is part of Auth0's architecture. 

{% include tweet_quote.html quote_text="Auth0 is flexible platform! Setup a custom e-mail provider using SparkPost or other alternatives." %}
