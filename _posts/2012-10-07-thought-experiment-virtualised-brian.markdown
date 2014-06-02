---
layout: post
title: "Virtualised Brian - A thought experiment"
date: 2012-07-10
description: What would it mean if we could combine the concepts of Virtualisation and the Brain? What would be the possible? A couple of days ago at lunch with Paul, I thought about this and lets see where we ended up.
---

What would it mean if we could combine the concepts of [Virtualisation](http://en.wikipedia.org/wiki/Virtualization) and the [Brain](http://en.wikipedia.org/wiki/Human_brain)? What would be the possible? A couple of days ago at lunch with **[Paul](http://www.linkedin.com/in/philion)**, I thought about this and lets see where we ended up.

## Assumptions

1. The scene is set in a distant future.
2. Evolution has not imparted increased brain utilization.
3. During sleep or people who are not using their brains or people with death sentence or life imprisonment or other long term imprisonment (Ethical issues aside) may be put into a "base functions only" state.
4. The "suspended" brain only needs a fraction of its total capacity (in addition to the brain stem and the spine) to manage life supporting body functions.
5. **Important**: Humans have found a way to "install" a Xen/KVM like virtualization layer into brains.

## Basics
**Brain**, is the key organ of every human being. Although the human brain represents only 2% of the body weight, it receives 15% of the cardiac output, 20% of total body oxygen consumption, and 25% of total body glucose utilization. This complex organ is responsible for all the advancements we see today.

**Virtualisation** technoloies such as KVM and Xen have revolutionized computing and are a cornerstone to cloud computing. There are different forms of virtualization(complete, partial or para virtualization) but in any form it is a big step forward.

Given the current virtualisation architectures (such as the ones used by Amazon, Google and other opensource project such as KVM and XEN), there is a host systems which is Dom0 and is the one that provides the guest consciousness(maybe multiple) with access to the underlying wetware. This is an important bit, we want the host to have some elevated access level in terms of being able to manage the other brain instances (referred to as "instances" from now on). This way in case we have a requirement of having to shutdown/revoke access/bill brain power the Dom0 instance is the one which does this.

## Details
The Dom0 here is pretty thing and is the instance which is closest to the the wetware. It provides the other instances with networking (ability to connect to the wet-ware internet), on demand computing resource (how much of the brain capacity are we allocating to this instance), billing subsystem (if we are doing this I suppose we want to do some form of accounting on the usage).

## Networking
I propose that the host body is connected to a sort of an internet via some non-surgically attached means. Something like some sort of headgear which allows for high bandwidth connection to the outside world. From what I understand (and I am not a neurosurgeon), this may require a one time interface implant. Which can be controlled via thought and a physical hardware switch - You always want a big red button on the back of your head which shuts down the virtualized brains.

The different instances within the brain would communicate over the different neurological pathways already existing inside the brain. The only thing which I can see as the bottle neck is the [corpus callosum](http://en.wikipedia.org/wiki/Corpus_callosum). This is a part which is like a narrow bridge inside each of our brains. My point is, the inter-hemisphere communication would be slower than the intra-hemisphere communication and the slowest one would be inter-brain communication.

## Compute resources
This is the real thing that we are supporting, having virtually unlimited amount of brain power for on-demand use is the biggest thing. It would let the human race jump atleast 2-3 orders of magnitude in terms of innovation ability. My assumption is that, the current rate of scientific or progress in other fields limited by the amount of "thinking" which people in these fields can do at a time. If a brain virtualization was available and if we tought ourselves to package and construct thoughts in a way such that they can be run a compute jobs or more like programs, scientists could potentially think continuously and work out hundreds or millions of thoughts in parallel, eliminate the bad ones in a single day which would otherwise take years.

### Types of compute resources
As far as I know/understand the human brain's parts are fundamentally different in terms of the kind of work/computation that they can perform. Hence, we could serve two kinds of resource types.

1. **Left brain instances**: numerical computation (exact calculation, numerical comparison, estimation), direct fact retrieval, grammar/vocabulary and literal functions.
2. **Right brian instances**: numerical computation (approximate calculation, numerical comparison, estimation), intonation/accentuation, prosody, pragmatic and contextual functions.

In adddition to the basic brain functions, there may also be somehigher level specialized brian "abilities". Certain brains are proficient in certain abilities. Let's say a brain  which provides "Calculus abilities" so that calculus is a basic and high performant capability so that if a scientist needs to perform some complex math, she/he could get a specialized "calculus brain" which would be able to perform integral/differential calculus with much higher efficiency when compared to a "basic brain". This does not mean that a the "basic brain" is any less capable of performing calculation but just that the neural pathways for doing these functions would be slower and less performant by an order of magnitude.

This means, there is an absolute incentive to learning more and diverse things, at the same time, specializing in certain fields would make certain attributes a basic capability of your brain. I draw these assumption from personal experience, ater years of math and computer science, I can solve certain thoughts almost natively i.e. I don't have to think much when I look at an algorithm and try to figure out the algorithmic complexity or maybe when I need to calculate powers of 2 or a simple/medium level calculus problem.


### Thinking workflow reimagined
The new thought process would probably be something like this:

1. Think of a big new idea.
2. Figure out different parts of the idea, the different thought components involved.
3. Reserve instances of the instances that you may need to reason each of these thoughts.
4. Send thought "jobs" to these remote instances over some wetware communcation media/network for processing and contemplation.
5. Watch the brain cluster for failures and re-initiate thoughts for which the brain instances got terminated.
6. Get processed results back from instances as they complete.
7. Repeat till desired result is obtained or you give up.

Now where does this take us, we have at our disposal, unlimited brain power. Now given that you can yourself virtualize your own brain in case your original brain gets tired, you could background your thoughts, send them to another instance, sleep or get some down time and later retrieve your calculated/processed thoughts.

With such technology humans would stop being isolated beings and would become more like hive consciousness beings.

What does this mean to people who give out resources? When you sleep or are just not using your brain you can "rent out" your brain, wake up and continue where you left off. In case you need more computation resources, terminate the guest instances and reclaim some more local resources.

##Brain Resource management and billing
The natural question here is what about the usual management of your own brain resources. Maybe you just feel tired or want to go on vacation and not have anything running in your brain except you. Being Dom0 or the host consciousness you have additional control over the capabilities of your resources. This allows you to suspend or maybe completely disconnect the brain from external entities and hence run in isolation.

The host may find that some instance has gone astray and misbehaving. This is something that should be caught by either the virtualization layer or something like a monitor process available to the host. This way it can be terminated or in extreme cases, banned completely - I am thinking of API token based access control if OAuth is too heavy-weight. :) 

The other aspect of doing all this may be purely financial. We just want to "sell" brain compute power. Billed by the second or minute or hour, whatever is appropriate. This might be the economy of the real "*information age*".

## Brain Storage
Now that we have got around to performing computation, we need some way storing the exabytes of thoughts and memories that we would generate, even if they are transient thoughts.

The virtualization layer may provide access to "brain storage", which may let a user store and process thoughts which are not constrained by the brian of the person initiating the thought. This is really important if we want the compute resources to be of any use, my hypothesis is that the original thought is probably really small thought so the network utilization for transferring thought is probably very small (A picture or phrase or some other brain wave chatter) but the scratch space requirement to build the thought tree and work through the all different things involved may be really large. Maybe even span beyond the capabilities of a single brain, needing *sharding of the thought* across different brians.

The basic requirement of this storage platform is complete isolation of thoughts. It has to be impossible for any other brain instance or even the host to look at the information/ideas stored by a different user. Something like encrypting the thoughts with some for of encryption and signing it for integrity.

## Conclusion
Where are we at the end of this? I believe we are making huge amount of progress in the fields of science and technology and I think this might be just a pipe dream but then so was flying and handheld computers. Having unlimited brain power would change the world in a fundamental way. It would not only push the human race years forward but also make us much closer to each other. The concept of political boundaries would essentially cease to exist. Internet would be a thing of the past (or just a minor subset of the **Cognitive-Net or Cognet**). Peace would be just a thought that would become natural (I hope) to every human and essential to the continued survival.