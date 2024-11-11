+++
archetype = "home"
title = ""
+++

# Latest Posts

<img src="/observability/testing-in-production-progressive-delivery-with-canary-deployments-explained/thumbnail-03.jpg" style="width:50%; float:right; padding: 10px">

## [Testing in Production! Progressive Delivery with Canary Deployments Explained!](/observability/testing-in-production-progressive-delivery-with-canary-deployments-explained)

Today I will make an outrageous claim. Ready? Here it goes... The only testing that truly matters is **testing in production**. The only way to truly verify that a release is working as expected is to run it in production with "real" users and "real" workload. Testing a release before it reaches production is helpful and I am certainly not going to tell you to stop writing and running your unit tests, and functional tests, and integration tests, and whichever other type of testing you might normally do. What I am going to tell you is that you have to test your releases in production. Confirmation that "real" **users got what they expected** is the only thing that truly matters.

**[Full article >>](/observability/testing-in-production-progressive-delivery-with-canary-deployments-explained)**

---

<img src="/internal-developer-platforms/from-docker-to-kubernetes-running-backstage-in-production/thumbnail-02.jpg" style="width:50%; float:right; padding: 10px">

## [From Docker to Kubernetes: Running Backstage in Production!](/internal-developer-platforms/from-docker-to-kubernetes-running-backstage-in-production)

Backstage is great, or not, depending how you look at it. In any case, the important thing to note is that the only thing we're getting is source code. Since it's written in TypeScript, we can run it by executing `yarn` this and that or `node` this and something else. While that's probably okay while developing it, it is silly when running it in production or anywhere else other than our laptops. It's **not 1999** any more. Today we package almost everything into container images and, from there on, you might be runnning it in Docker while I scream at you trying to explain why Docker is not a good idea for anything that should run in production. More likely, you are running your applications in Kubernetes, or as Azure Container Apps, or through Google Cloud Run, or anywhere else. What matters is that **OCI images are the standard**, no matter whether we run something as containers or anything else.

**[Full article >>](/internal-developer-platforms/from-docker-to-kubernetes-running-backstage-in-production)**

---

<img src="/ci-cd/ci-vs-cd-vs-gitops-vs-state-management-whats-the-real-difference/thumbnail-01.jpg" style="width:50%; float:right; padding: 10px">

## [CI vs. CD vs. GitOps vs. State Management: What is the Real Difference?](/ci-cd/ci-vs-cd-vs-gitops-vs-state-management-whats-the-real-difference)

Today I want to answer a set of questions I get fairly often. People ask me to compare GitOps with CI/CD or to explain the difference between CI and CD. At other times, I hear people talk about those terms with confidence, yet often with missunderstanding of what those are. To make things more complicated, tools tend to have missguided names that often make the situation even more complicated. So, today's session will explain the differences between Continuous Integration or CI, Continuous Delivery or CD, and GitOps. We'll add state management to the mix and we'll go not only through processes and activities related to those terms, but also try to demistify the tools in those areas. As a bonus, we'll add visualization challenges. Who knows? We might come to the conclusion that we are doing it all wrong or that our expectations are unrealistic.

**[Full article >>](/ci-cd/ci-vs-cd-vs-gitops-vs-state-management-whats-the-real-difference)**

---

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
