---
layout: post
title: MS Teams - Easy access to Channels
subtitle: Usability nightmare that is Teams Channels
author: jjk_charles
categories: tips
tags: msteams ms-teams-channels
---

Majority of our Working hours are spent staring at communication software like [Teams](https://www.microsoft.com/en-us/microsoft-teams), Slack, etc. (this is true at least for us IT Folks since the [pandemic](https://en.wikipedia.org/wiki/COVID-19)) When they have become an integral part of our daily working life, it is important that they provide the best possible experience to not help improve collaboration rather than hinder it.

Below is a (actually, one of the) gripe I have about MS Teams and how I ended up mitigating it.

### The Problem 
Teams has the concept of Chats and Channel Conversations. Chats being the 1-on-1 interactions as well as Group Chats, whereas [Channel Conversation](https://support.microsoft.com/en-us/office/first-things-to-know-about-channels-in-microsoft-teams-8e7b8f6f-0f0d-41c2-9883-3dc0bd5d4cda) refers to the Team discussions. 

The primary difference between the two boils down to,
1. How you access them?
    
    Chats can be access from "Chat" icon, and Channels through the "Teams" icon and navigating to the Channel you are interested in.

2. How the discussions are conducted?

    Chats are basically organized in a linear timeline with ability to "Reply" to older messages.

    Channels are organized as [threads](https://en.wikipedia.org/wiki/Conversation_threading).

In lots of scenarios, threaded conversation makes sense as it provides context on the discussion that is going on.

> So, if it is great, where lies the problem?!

This is where those two distinctions between Chat and Channel Conversations come into play. For one, Chat live in its own screen and Channels live in their own screen. There is no ability to access one from the other. Since Channels get a second-class citizen treatment within Teams, it causes a huge adoption issue for Channels.

While I am not the first one to express this*, Microsoft doesn't seem to be making any attempts to alleviate it. 


#### How did I solve for this?
Well, I don't know yet if I can confidently solved this or not, as I have just shared this with my team, and only time will tell how well received it is.

> So, to put some disclaimer, this isn't a clean solution by any means, as it is still a major inconvenience but this at least tries to keep things little bit manageable.

##### Approach 1
This approach uses the Browser's ability to install a site as an App (aka PWA), where I suggested the team to install MS Teams as a PWA and leave it on the "Team" screen for quick access to Channels.

##### Approach 2
This approach uses Teams' feature to pop out a Chat into its own window. If you have this capability turned ON, anytime you click on message notifications, the corresponding Chat/Channels will open in a new window.

But, unfortunately, this feature can be triggered manually from the UI only for Chats, and not for Channels. You can pop-out a specific conversation from the Channel but not the Channel itself.

Below are the steps I followed to pop-out a Channel Conversation any time I wanted to,

1. Open the Channel you are interested in, and go to the `Files` tab
2. Click on the ellipsis and choose `Open in SharePoint`
3. In the web page that opens, click on `Go to Channel`
4. Now, when prompted open the link using MS Teams (and optionally set is as the default behavior)
5. Create a shortcut for this URL (or) bookmark it in browser for easier access to it later on

![Open in SharePoint](https://i.postimg.cc/65M4xBNM/image.png 'Open in SharePoint' )

![Go to Channel](https://i.postimg.cc/mrzmFdwG/image.png 'Go to Channel' )

While this is not the cleanest solution, it does seem to help a little bit. Again, this mainly works because I collaborate mainly on 2-3 Channels on a regular basis. If you do collaborate on multiple Channels, this could become cumbersome (but still better than switching back and forth between Chat and Teams screens!).

Do you face similar problems with Teams? How are you tackling it?