+++
archetype = "home"
title = ""
+++

# Latest Posts

<img src="/internal-developer-platforms/internal-developer-platform-day-2-operations-solved-with-kubernetes-and-crossplane/thumbnail-03.jpg" style="width:50%; float:right; padding: 10px">

## [Internal Developer Platform Day 2 Operations Solved with Kubernetes and Crossplane](/internal-developer-platforms/internal-developer-platform-day-2-operations-solved-with-kubernetes-and-crossplane)

I did it. No! We did it. Actually, that's not correct either. Someone else did it. Doesn't matter who did it. What matters is that one of, in my opinion, big problems has been solved. Developers can now not only create, update, and remove their applications and infrastructure, but they can also get to know **what's happening with the resources they are managing**.

**[Full article >>](/internal-developer-platforms/internal-developer-platform-day-2-operations-solved-with-kubernetes-and-crossplane)**

---

<img src="/internal-developer-platforms/getting-started-with-backstage-from-zero-to-operational-dev-portal/thumbnail-02.jpg" style="width:50%; float:right; padding: 10px">

## [Getting Started with Backstage: From Zero to Operational Dev Portal](/internal-developer-platforms/getting-started-with-backstage-from-zero-to-operational-dev-portal)

Today we're going to explore Backstage from newbee perspective. We'll see what it is, what it's main components are, and how to set it up. We'll go from nothing to an operational portal in development mode. Unlike other similar tools that often require us to simply configure and run them, with Backstage we have to get the source code, develop or configure what we need, and run it on our laptop to see the result. Later on, once we're happy with what we did we might explore how to package it all up, run it in production, and do whatever else needs to be done.

Let's start from the beginning.

**[Full article >>](/internal-developer-platforms/getting-started-with-backstage-from-zero-to-operational-dev-portal)**

---

<img src="/ai/unlock-the-power-of-gpus-in-kubernetes-for-ai-workloads/thumbnail-01.jpg" style="width:50%; float:right; padding: 10px">

## [Unlock the Power of GPUs in Kubernetes for AI Workloads](/ai/unlock-the-power-of-gpus-in-kubernetes-for-ai-workloads)

Here's a question. Where do we run AI models? Everyone knows the answer to that one. We run them in **servers with GPUs**. GPUs are much more efficient at processing AI models or, to be more precise, at inference.

Here's another question. How do we manage models across those servers? The answer to that question is... Kubernetes.

**[Full article >>](/ai/unlock-the-power-of-gpus-in-kubernetes-for-ai-workloads)**

---

<img src="/terminal/discover-the-future-of-shells-with-nushell/thumbnail-03.jpg" style="width:50%; float:right; padding: 10px">

## [The Future of Shells with Nushell! Shell + Data + Programming Language](/terminal/discover-the-future-of-shells-with-nushell)

Implementation and maintenance of terminal is **tedious**, especially when runtime portion of it is concerned. For example, if we'd like to be notified when a potential breach is happening, we'll likely use a tool like [Falco](https://www.google.com/search?q=falco+terminal&sourceid=chrome&ie=UTF-8). It's a great tool. It's potentially one of the best if not the best tool of it's kind. It allows us to define an infinite number of rules that, when one of them is met, will fire notifications. That's the problem though. We have to define all those rules or, at least, accept a significant number of rules that are available out of the box. Essentially, we need to predict everything that should not be allowed to happen or, if we prefer the other way around, everything that is allowed. That is tedious and you are likely going to end up frustrated at best, in an asylum at worst. After all, **who can predict all the bad things that might happen?** and who is fully aware of all high and low level calls that applications are making? I certainly can't.

**[Full article >>](/terminal/discover-the-future-of-shells-with-nushell)**

---

<img src="/security/stop-writing-tedious-security-rules-let-kubescape-do-the-work/thumbnail-02.jpg" style="width:50%; float:right; padding: 10px">

## [Stop Writing Tedious Security Rules! Let Kubescape Do the Work](/security/stop-writing-tedious-security-rules-let-kubescape-do-the-work)

Implementation and maintenance of security is **tedious**, especially when runtime portion of it is concerned. For example, if we'd like to be notified when a potential breach is happening, we'll likely use a tool like [Falco](https://www.google.com/search?q=falco+security&sourceid=chrome&ie=UTF-8). It's a great tool. It's potentially one of the best if not the best tool of it's kind. It allows us to define an infinite number of rules that, when one of them is met, will fire notifications. That's the problem though. We have to define all those rules or, at least, accept a significant number of rules that are available out of the box. Essentially, we need to predict everything that should not be allowed to happen or, if we prefer the other way around, everything that is allowed. That is tedious and you are likely going to end up frustrated at best, in an asylum at worst. After all, **who can predict all the bad things that might happen?** and who is fully aware of all high and low level calls that applications are making? I certainly can't.

**[Full article >>](/security/stop-writing-tedious-security-rules-let-kubescape-do-the-work)**

---

<img src="/containers/stop-losing-requests-learn-graceful-shutdown-techniques/thumbnail-01.jpg" style="width:50%; float:right; padding: 10px">

## [Stop Losing Requests! Learn Graceful Shutdown Techniques](/containers/stop-losing-requests-learn-graceful-shutdown-techniques)

Look at this.

I will send a request to the application,...

```sh
curl "http://silly-demo.127.0.0.1.nip.io/fibonacci?number=50"
```

...and simulate failure or upgrade or any similar action by deleting the Pod where the application is running.

```sh
kubectl --namespace a-team delete pod \
    --selector app.kubernetes.io/name=silly-demo
```

The output of the `curl` command is as follows.

```
<html>
<head><title>502 Bad Gateway</title></head>
<body>
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

Since we initiated the delete process before the server returned a response we got `502 Bad Gateway` message. The application was deleted before it could respond and I, the user of that application, failed to get what I was looking for. That's horrible experience that could have been improved by enabling the application to shut down gracefully.

**[Full article >>](/containers/stop-losing-requests-learn-graceful-shutdown-techniques)**
