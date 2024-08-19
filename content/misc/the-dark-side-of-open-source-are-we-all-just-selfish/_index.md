
+++
title = 'The Dark Side of Open Source: Are We All Just Selfish?'
date = 2024-08-18T16:00:00+00:00
draft = false
+++

There's a lot of talk about open source and about some companies, big and small, taking advantage of it, and about some other companies changing licenses of their projects to be less permissive. Those conversations often end up with people yelling at each other. "**Damn MongoDB for changing their license!**" "**AWS is evil for taking advantage of Elastic!**" "**Env0 is taking advantage of HashiCorp!**" "**F\*\*K HashiCorp!**" "**Long live Elastic!**"

<!--more-->

{{< youtube 4l_kK90khNA >}}

Just as "normal" people are often split between their aligance to a political party and will be behind it no matter what that party does, software engineers tend to be split between **open source purists** and whatever the other side is called.

The truth is actually fairly simple. Most software engineers **take advantage of open source one way or another** and they do that for very selfish reasons. You, yes you are probably taking advantage of open source and maintainers of open source projects. It's not that only companies are "selfish", most of us, individuals are selfish as well.

The reality is that most of us are using or working on open source projects for very egoistic reasons. We are all in it for **money**, and that's okay, most of the time.

Let me explain.

Many think that open source is all about anyone being able to use projects to their benefit **for free** and about people freely contributing to projects in their **free time**. While that might be the case for some projects, more often than not, especially when bigger projects are concerned, that is not the case.

Open source is, first and foremost, a **go-to market strategy**. A company invests cash, often from investors, to start an open source project. That investment is spent on paying full-time maintainers to work on the project because, surprise surprise, people cannot or do not want to work on a project full time without being paid. Full-time dedication requires influx of money and that influx is an investment. Someone invests in a project and that someone wants something in return.

By now, you are probably screaming at your monitor thinking that you are screaming at me. You are probably saying that there is an open source project that is not motivated by money, that there is a project where everyone working on it is doing it in their free time, and where there are no expectations other than "We are nice people, we are doing this out of goodness of our hearts, and **we do not expect anything in return**."

There are such projects, but they are often small and they can continue thriving without dedicated maintainers, without investment, and without asking for anything in return. Those are exceptions. Most open source projects are very expensive to maintain and require dedicated engineers. You know... Those that need to earn their living as well.

Take Kubernetes for example. On the first look, it might seem like everyone working on it is a volunteer, and that is indeed the case. As far as I know, no one is forced to contribute to Kubernetes. What matters, however, is that most of the people working on it are being **paid to work on it**. Some are literally being paid to work on Kubernetes, while others are having a permission from their company to work on Kubernetes during their office hours. That company is investing money in open source by paying people to work on something other than whatever that company needs them to work at.

Now, I am not saying that no one contributes to Kubernetes without any compensation but, rather, that many do and that without those people the project would not exist in the form it exists today. It would not have the dedication needed to be what it is today.

In any case... What matters is that Google, Microsoft, AWS and a bunch of other companies, big and small, are paying people to work on Kubernetes. They have an interest to do that, and that's okay. There is nothing wrong having an interest to invest time and money into something. As long that that interest is mutual, as is, most of the time, the case with Kubernetes, we have people with similar interests willing to work together to accomplish something that is **benefitial to everyone**. Everyone in need of Kubernetes can use Kubernetes for free. The same goes for Linux and endless other projects.

Kubernetes is, however, an exception, just as Linux is an exception. Both are so big that **many companies have common interest** to contribute to it. The important note here is that I said many, NOT one company.

Those interests can have many different forms. One company's interest could be sale of additional features on top of a project. That would be, in case of Linux, RedHat, SUSE, Canonical, and many others. In some other cases, that could be consulting or support. There is no one better to help you with implementation of a project but people who work on that project every single day. It could also be marketing, employee retention, or many other reasons. The important thing to note is that most of the people and companies that contribute to open source projects have interests to do so, and that by itself is not necessarily bad.

However, everything I explained so far is related to what's happening with projects once they are already out there and for this story to make sense, we need to go back and talk about the beginnings.

**Why start an open source project?**

Now, to be clear, from the start, I'm excluding from this story small projects like, for example, NodeJS libraries. I'm talking about mid to large size projects that require investments to thrive.

Why do companies start such projects in the first place? Why do they invest into something only to give it away for free?

There are quite a few reasons why someone starts an open source project. The most common one is **go-to market** strategy. Today, it is very difficult for a company to get the adoption and market penetration with closed-source software. It's not impossible though. It's just hard, very hard, and it depends on the type of the problem a project tries to solve. Still, in order for a company to sell software, it needs adoption of that software and to get adoption, it often needs to be open source.

Here's a question. **Do you think that Terraform would be as popular as it is without it being open source?** **Would Jenkins, Elastic Search, MongoDB, or PostgreSQL be as widely used if they did not start as open source?** **Would most of the servers run Linux if Linus Torwalds did not choose to make it open source from the very beginning?** **Would you use Kubernetes if it was yet another service limited only to Google Cloud?**

The answer to all those questions is "**most likely not**". Without giving software for free as open source, most companies would not stand a chance and almost everything would be in the hands of massively big companies like Google, Amazon, and Microsoft. With a small number of exceptions, companies are **forced** to start as open source. Today, those exceptions are mostly limited to SaaS-only projects.

All in all, the most common reason for the inception of an open source project is the go-to market strategy. If a company is successful, people will adopt the project and, at some later stage, that company will have the opportunity to sell services for that project, or to sell additional, often enterprise, features, or to offer it as a managed service, or something else. Today, adoption is often a prerequisite for sales. Adoption does not guarantee sales, but without the adoption it is close to impossible to sell anything. Again, that's not always the case. There are always exceptions, but I'm ignoring them, for now.

We have plenty of examples of companies that became relatively successful with that strategy. HashiCorp grew to a decent size company thanks to open source, thanks to very wide adoption of their projects like Terraform, Vault, and a few others. The same can be said for Elastic. It is a mid-size company thanks to open source ElasticSearch. Then there is MongoDB, Redis, and many others. The number of those who failed is infinitely bigger, yet many of those who succeeded can thank open source for that.

Hence, we have open source as a go-to market strategy where companies invest heavily in something that can be used for free in hope of gaining return on that investment later through enterprise sales, consulting, support, or whichever other method they can think of.

There's nothing inherently wrong with that. We all benefit. Companies ultimately get a return of their investment while everyone else can choose whether to use open source for free or pay for additional features or services. That's the cost that is often paid by big enterprises rather than individual users or smaller companies. **Everyone uses open source but bigger companies pay for it**, one way or another.

That's, at least, the idea. However, what happens if a company behind an open source project starts being successful and does not need open source any more? What happens when other companies try to monetize on that same project? Well... That's when we see changes to licenses like those we saw with projects created by Elastic, MongoDB, HashiCorp, and others.

The reasons to change the license vary from one project to another but, more often than not, they can be described as either "**we got the adoption, we don't need open source any more**" or "**a much bigger company is threatening us**".

Now, I cannot say for certain which project that changed the license falls into which category and I will not try to speculate by naming names. What I can say is that AWS and other massive companies tend to threaten those who based their business on open source. If AWS, Google, or Azure offer that project as a service, that will inevitably reduce the income or the potential growth of a company behind that project. So, the company that started an open source project is under threat and it is only natural that such a company will change the license so that AWS or any other bigger company cannot monetize on that project and, by doing that, affect the growth of the company behind the project.

However, that is problematic on quite a few levels. To begin with, AWS or whomever starts to offer that project as a service likely did not contribute as much to the project as the company behind it. Still, that is the nature of open source. I might not contribute to Linux but some other project yet I expect to be able to use Linux as the operating system where my project is running and, by doing that, gain some benefits. No business can contribute to all the projects they use.

A more important problem is that changing the license so that "bigger guys" cannot use it also affects smaller companies that might have contributed to it or not, but want to offer some services that use that project directly or indirectly. Depending on the new license, that might affect contributors from other companies or private individuals that invested in that project with their own contributions.

What I'm trying to say is that it is very hard, if not impossible to change the license so that only companies bigger than yours cannot use it any more, so both big and small players are affected.

Still, you can say "**AWS is evil, that project should change the license**", but there is still the question of forks. Anyone can fork any open source project so AWS or anyone else is free to do that as well, and that's exactly what happened with, let's say ElasticSearch, Terraform, and quite a few others. As a result, AWS or whomever was supposed to be prevented from using the project continues using it and now we have the effort split into two or more forks. The only outcome is the waste of time that results in slower pace of the advancement of the project. That is, as it often happens, until the original or the fork or both eventually die.

Hence, one might come to a logical conclusion that starting an open source project was a bad idea in the first place, but that would also be wrong. As I already mentioned, without open source there would be no adoption and without adoption there would be no business that keeps the project going.

So, without open source there would be no Elastic, no HashiCorp, and no MongoDB, yet now that those companies grew to a certain size due the success of those projects, they are threatened by others who also want to monetize on it. We want all the good things that come with opensource to help us grow our business but we do not want anyone else making business with it. That's a bit hypocritical, isn't it?

That leads me to the second big reason companies change their license. Sometimes, they are not threatened by bigger companies but simply they reached the needed adoption and now they do not need it to be open source any more. It helped them create the business and now that everyone needs it, that business can grow faster if the license is changed so that competition is squashed, so that companies need to pay more for it, so that... You get the point. It is **we reached the adoption, we don't need it to be open source any more** type of the conclusion. At least one company admitted that in public and I'll leave it up to you to guess which one it is.

In reality, the problem is very different than what is presented. The real problem is that **no open source project should be owned by a single company**. If ElasticSearch, MongoDB, or Terraform were in a foundation, the situation would be different.

To begin with, projects in foundations tend to have more than one company contributing to them, especially if those projects get wider adoption. The reason is relatively simple. It is very risky for someone to contribute to an open source project owned by a single company since it would be at mercy of that company. If, for example, I make a pull request with a feature I need but that pull request goes against the readmap of the company behind that project, that PR is likely to be rejected. Even if it gets accepted, I am investing time and money into a project owned by someone else. The fact that a project is open source does not mean that no one owns it. Hence, it is risky to invest in projects owned by some other company so such projects often continue being maintained only by companies that own them. That's one of the reasons why such companies feel that others are taking advantage of them. Others do not contribute yet there is a possibility that they will create a business based on it. It's a difficult situation with companies or even individuals being discouraged to contribute yet, due to the nature of open source, they can become competition.

Projects in foundations operate in a very different way. They are owned by a foundation so no single company can unilaterally decide what to do with it. Now, to be clear, a company might convince the foundation to do something that is not in the interest of the project or its users or the competitors. Still, it cannot be a unilateral decisions. You might have started a project but the moment it is moved to a foundation you do NOT own it. As a result, others are more incentivized to find interest in such projects and invest in them so we get **more diverse contributions**, especially from dedicated maintainers. That's, for example, what we can observe with Linux or Kubernetes. Those are massive projects with a wide range of companies that have vested interests in them and those interests are, in big part, created by the relative neutrality of the project. Some of those companies might have bigger influence than others, mostly due to the amount of investments they make, yet not a single one owns it.

![](/logo/cncf.png?height=15vw&classes=inline)
![](/logo/apache-software-foundation.png?height=15vw&classes=inline)

Due to recent changes to some projects, I hope that people will start taking a closer look at the ownership. I hope that people will understand that projects that belong to foundations like CNCF or Apache are more reliable and less prone to upheavals than those owned by a single company.

Now, let's get back to you. I'm not letting you off the hook either.

You might be benefiting from open source without giving anything back, without contributing, and without paying for it. You might be earning money while using open source for free.

While that behavior might raise some criticism, I'll say "**that's okay**". That's what open source is for. Anyone can benefit from it, no matter whether we give something back or not. However, if we are not actively involved in a project, we don't have a moral right to demand anything from it. If a serious vulnerability is discovered in a project you use for free, you do not have a moral right to demand it to be fixed. Bear in mind that I said "moral". You can demand a fix, but no one is forced to do it for you. You can always create a pull request yourself and convert yourself from a user or a company that only benefits from it to someone who also gives back. You can **fix your problems yourself** and you can add features that are missing yourself. Alternatively, you can pay someone to do it for you. That's how many projects are financed. Or... you can just wait and pray to your deity of choice that someone else will do it.

The problem is that companies use open source to power solutions that they sell, yet they expect certain guarantees from those projects. That's greedy. You are free to use Linux on your servers and you are free to setup Kubernetes on those servers and you can use free libraries in your code. That's fine. Just don't complain when your needs are not fully met with those projects yet you do not want to contribute back. Open source means that you are **free to use anything**, yet you are **not entitled to demand anything**.

If you want to do the right thing both for yourself and for your business, **choose projects in foundations** rather than those owned by a single company. Get involved in projects or try to finance them by purchasing enterprise versions and support, or by giving donations. Open source is not like big bang. Open source is not created out of nothing. Open source projects are investments that need to see returns in some way or another. If, for no other reason, then because dedicated maintainers need to eat as well. If you earn from using something that is free, it is only natural that you give something back. That's symbiosis that allows both you and those projects to thrive.
