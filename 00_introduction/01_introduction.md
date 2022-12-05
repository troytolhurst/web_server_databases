# Introduction to the Module

Most of the time, software doesn't just sit on your computer, it has to be
packaged up and made available to users somehow. In the current era, this
typically involves either deploying it to the web, or in some other way
integrating it with the Internet. Usually this involves something called 'The
Cloud'.

This module is designed to help you learn about cloud computing, what it is, how
to navigate cloud systems, and how to work with a simple CI-CD deployment
system. You will also learn a lot about the Internet in a wider sense.

## Key Terms

Here are some key terms for this module.

* **Deployment**  
  The process of taking some software that is running on your computer and
  making it run on the Internet where it is accessible to other people. For
  this, you will typically also try to make it a bit more robust.

* **Abstraction**  
  The process of hiding or ignoring the details of something in order to focus
  on the important parts. For example, when you drive a car, you don't need to
  know precisely how the engine works. Those details are _abstracted_. This is
  achieved through an _abstraction layer_ — the controls for the car that allow
  you to ignore the details of the engine.

* **Cloud Computing**  
  To understand this term it is best to compare it to the alternative. In the
  past, if you wanted to run a program you would have to buy all of the physical
  computers that you would need in order to run it, set them up, probably
  connect them to the Internet, and maintain them yourself. If you needed the
  servers for only a few weeks, you still had to buy them outright, and then
  hopefully sell them to someone else. This is typically called _on-premise_
  computing.

  In Cloud Computing, you go to a cloud provider company like Amazon Web
  Services (AWS) and say "I've packaged up my web application into this file —
  can you run between 2 and 12 copies of it depending on how many are needed to
  serve the traffic it is getting." And AWS will go and do that. It's a bit more
  complex than that in practice, but that is the general idea.

  Cloud computing as a way of _abstracting_ away the details of the physical
  machinery that you need to run your program. The _abstraction layer_ can vary
  according to the problem being solved, but often involves creating virtual
  computers and connecting them with virtual connections. The cloud provider
  then handles the details of how to implement that.

* **CI-CD**  
  This stands for Continuous Integration (CI) and Continuous Deployment (CD).
  
  Continuous Integration is a process where an engineer pushes their code to a
  repository, a cloud system then downloads the code and runs some tests to
  check whether it is suitable for going into the main codebase. These tests
  typically include running the automated tests written by the developer, but
  may also include other tests for code style, test coverage, or security
  testing.

  Continuous Deployment is a process where after these tests are run the
  software is automatically deployed — typically to a cloud system.

## How to think about the learning of this module

The previous modules were very practical. You read some material, got an
exercise, and likely spent a fair chunk of time coding.

This module is going to feel different. We do want you to go away with some
practical skills — setting up CI-CD to deploy a web application — but you could
probably learn the practical tasks of this module in a single day. There's more
or less one right way to achieve it.

What we really want you to get here is a deeper understanding of cloud systems.
For this you will need to understand the fundamentals of networking and a bit
about how the Internet works.

Since the cloud is so good at _abstracting_ away details, we're going to talk
quite a bit about the technological details that underpin the cloud — what it
abstracts away. And to achieve this, we're going to talk about the history of
networking.

This is so that when you see a control panel like this:

![A screenshot from AWS featuring a lot of detailed
information](../resources/aws-screenshot.png)

You're going to have the foundational understanding necessary to learn how to
use it.

## How to use this module

Work through the material in the main contents from top to bottom.

For each page you can pick whether to engage with the text content or the video
alternative. Consider following the examples and ensuring you understand what is
going on.

As you work through you will see expanding boxes like this:

<details>
  <summary>:speech_balloon: Try clicking me to see what happens.</summary>

  ---

  Surprise!

  Sometimes these will provide a bit more context, extra challenge, answer
  questions you might have, or help you fix errors we suspect might come up for
  some learners.
  
  They are always optional. Open them if you want to or skip them if you don't.

  ---

</details>

If the page isn't directly practical, you will see at the bottom a list of
Investigations. These are research exercises for you to undertake. They have
ratings from :hot_pepper: to
:hot_pepper::hot_pepper::hot_pepper::hot_pepper::hot_pepper: to indicate how
challenging we think they are.

To help you deepen your learning, we'd recommend you pick a few of these
investigations according to what you find interesting and challenging. Most
people probably shouldn't do every single one of them just due to time, but
you're welcome to try it.

Aim to get through to the end of phase three (setting up CI-CD) by the end of
the module.

<details>
  <summary>:speech_balloon: These :hot_pepper:s are a cool idea — who came up with them.</summary>

  ---

  A Makers alumni called Rita. Here was her explanation, slightly amended:

  > We were talking about a more complex workshop you ran in week three. I
  > thought it was great you decided to expose students to something more
  > advanced, and brought up my main criticism of Makers — sometimes there is a
  > lot of hesitation when it comes to students' ability to manage complexity
  > for themselves. John confirmed that there is fear of overwhelming students.
  > 
  > So my solution, have chilli ratings :hot_pepper::hot_pepper::hot_pepper:
  > like on a menu. Then, people can calibrate how stressed they should feel
  > about not understanding something — and it can be their responsibility to
  > manage it, not the coaches. It would allow people who enjoy spicy to have
  > access to that, and those who don't to feel less overwhelmed. (You did that
  > with the workshops too, with a verbal how much should I worry if I don't get
  > all of this? rating)

</details>

That's it. To get started, head back to the [main page](../README.md).


<!-- BEGIN GENERATED SECTION DO NOT EDIT -->

---

**How was this resource?**  
[😫](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=00_introduction%2F01_introduction.md&prefill_Sentiment=😫) [😕](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=00_introduction%2F01_introduction.md&prefill_Sentiment=😕) [😐](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=00_introduction%2F01_introduction.md&prefill_Sentiment=😐) [🙂](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=00_introduction%2F01_introduction.md&prefill_Sentiment=🙂) [😀](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=00_introduction%2F01_introduction.md&prefill_Sentiment=😀)  
Click an emoji to tell us.

<!-- END GENERATED SECTION DO NOT EDIT -->
