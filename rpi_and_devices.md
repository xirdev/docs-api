# Devices and Neurons

Some ideas on branding.

I think this is an important topic not as an pending task, but for investor information and branding of the platform over Xirsys the WebRTC implementation.

IMO it's crucial investors realise that the platform is not tied to WebRTC and online services. So what they are investing in is a platform that can go in many potential directions. The fact that there is a concrete implmentation of the platform (Xirsys) is excellent, but that the platform can be used in many contexts is even more exciting.

In particular i think that showing an experimental example of the platform in a device context would be  killer. Same tech used in completely different scenario to Xirsys.

But we need a means of thinking about the platform as opposed to the implementation.

## Analogy

The core of Xirsys, the VSL, is maybe better described as a "neuron". It may respond to various stimuli; http, websockets and TCP/IP signals. Crucially, it can record it's stimuli, knows about and can communicate with other neurons.

> Neurons must continuously gather information about the internal state of the organism and its external environment, evaluate this information, and coordinate activities appropriate to the situation and to the person's current needs.
> http://www.cerebromente.org.br/n12/fundamentos/neurotransmissores/neurotransmitters2.html

![](/neuron.gif)

Indeed, I think VSL should be renamed "neuron". The term provides the required granularity and function of the existing VSL - it's a coordination primitive and sensor.

A cluster of neurons in neuroanatomy is called a "nucleus".

> https://www.boundless.com/physiology/textbooks/boundless-anatomy-and-physiology-textbook/overview-of-the-nervous-system-11/collections-of-nervous-tissue-112/clusters-of-neuronal-cell-bodies-608-5976/

Since Xirsys is the WebRTC brand **maybe a better name for 'the platform' is 'Nucleus'?**

We now have a nice seque into the nerves-project.org for dropping neurons onto RPis via Nerves :)

## Branding

Thus, imo, when talking of the platform, and establishing the correct conceptual framework for investors , talking of neurons deployed to devices forming a nucleus of functionality is the way to go.

This notion creates a clear distinction conceptually and if we have the core deployed, even just experimentally to devices, is the concrete manifestation.

## Summary

So that we're clear, the current VSL, sans Dashboard, with 99% probability (haven't tried it yet) can be dropped onto an RPi or other devices using the nerves-project.org.

The VSL is a generic endpoint and coordination cell/neuron. As such each neuron knows about all other neurons, and their state (kv4).

So the function of a nucleus of neurons is the concrete implementation, e.g. Xirsys and Turn.

Good intro to nerves-project.org

https://youtu.be/poWoCWDLxRU

## Fitness

When we get to deploying nucleus and neurons automatically, this whole concept plays well into biological fitness; which nueron's are performing well or not, should they be taken down automatically - netflix style, or be better deployed elsewhere for the benefit of the organism.

So on and so forth.






