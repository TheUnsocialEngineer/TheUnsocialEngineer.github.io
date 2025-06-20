---
layout: post
author: Eoka
---


Antiland Is an "Anonymous Chat Rooms, Dating roleplay game" Chat app with both web ansd Android applications (Apple removed from appstore) with an estimated 40 million users.They claim that despite the app being anonymous and having many filters to enforce that"The point of this app is to have fun, meet new people, and make friends. You can also use this app for dating purposes and find a spouse as well!". I had actually used the app for about a year before i started to uncover the darker side of the app and the issues that it would cause.


It all started when I was bored one day so decided to start browsing the network traffic of the apps web version when I noticed some calls to heartbeat URLs like   
[https://ps.pndsn.com/v2/subscribe/sub-c-24884386-3cf2-11e5-8d55-0619f8945a4f/nbaUk2wskP/0?heartbeat=300&tt=17433233931111506&tr=42&uuid=mV1UqOtkyL&pnsdk=PubNub-JS-Web%2F7.4.5&l_pres=335](https://ps.pndsn.com/v2/subscribe/sub-c-24884386-3cf2-11e5-8d55-0619f8945a4f/nbaUk2wskP/0?heartbeat=300&tt=17433233931111506&tr=42&uuid=mV1UqOtkyL&pnsdk=PubNub-JS-Web%2F7.4.5&l_pres=335). Wanting to see what they were I looked at the response tab in the network traffic and to my surprise it contained a feed of all the chat messages in JSON as they were sent by users.

Curious to see how it worked I opened the URL in a new tab expecting an error however instead the feed of messages appeared, did the "worlds largest anonymous chat app" really leave their direct chat feed public and unauthenticated? To test further I opened the same URL in an incognito window and once again the messages appeared, it seems like it really was just open to the internet. Accessing the URL once only provided new messages in the current chat pool however refreshing would update to show all new messages in real time, accessing the URL later would have the full list of current messages., I had just discovered Antiland's backend chat provider.

![[Pasted image 20250330095215.png]]
<link rel="icon" type="image/png"  href="/assets/img/antiland/Pubnubfeed.png">
PubNub chat feed

Curious about the potential of this I started developing a basic chat logger that would take a chats ID and refresh the PubNub URL every second and log every message that came through. During the development of this POC I realised I could use this same system to create bots in a similar way to Discord and so the Antiland Bot project was born

I Created a python library called [Antiland](https://github.com/TheUnsocialEngineer/Antiland) (based on discord.py) which used this chat message logging system to identify the commands a user would register and a [proof of concept bot](https://github.com/TheUnsocialEngineer/antiland-userbot) to go with it, these bots would use a users token to log in and perform messages such as send messages, access information about chats and users, send images and even have AI respond for fully autonomous chats. However it was while researching the API I stumbled onto even more issues.


API

The web version of Antiland uses a Main.js file located [here](https://www.antiland.com/chat/main.66a3d4583495b4ed.js) which contains all the API routes for things like sending messages, creating groups and other app functions, This was perfect for documenting and developing the API for my bots however as I scrolled through the list of functions I started to notice some that could be abused for malicious intent. each function in the JS file would contain the route of the API path and any parameters that route needed to function correctly which made it extremely simple to identify any URLs with misuse potential

![[Pasted image 20250330095556.png]]

example of functions in the Main.js file

I was able to identify 5 API routes that had the potential to be misused and leak user data

Firstly there is **/functions/v2:profile.confirmEmail** , **/functions/v2:profile.resendEmailVerification** and **/functions/v2:profile.resetPassword** which take either an email address and phone number as parameters.

![[Pasted image 20250330095920.png]]
![[Pasted image 20250330095940.png]]
![[Pasted image 20250330102802.png]]

At a first glance these don't seem that important however the reset password endpoint is unauthenticated and both endpoints have no rate limits. Since both the endpoints send a message to either an email or phone number this means that a bad actor could spam the endpoints with requests leading to these getting flooded with messages  (the requests do require specific app headers to work). It is worth noting that the confirm email does require a user token however this is easy to obtain however spamming the endpoint would likely result in the account being terminated. These endpoints are low severity by themselves as you need to know an email or phone number registered on Antiland however this is also way easier to pull off than it should be

Exposing user details and identifying accounts

There are multiple ways to identify if an email or phone number exist on the Antiland platform, the easiest way would be just spam the route mentioned above with lists and look at the responses however Antiland kindly provides us with some endpoints that Are designed to just do this. 

Firstly there is the aptly named **/functions/v2:profile.validateEmail** which as its name suggests takes and email and will return True if the email is registered on the platform. Once again this is unauthenticated as like with before the request does require specific app header
**x-parse-application-id:fUEmHsDqbr9v73s4JBx0CwANjDJjoMcDFlrGqgY5** and version string URL parameter **version=web/chat/2.0** to work and has 0 rate limits meaning any bad actor with a list of emails can check it against the platform.... what kind of crack did these developers  smoke and where can I get some.

![[Pasted image 20250330100932.png]]

![[Pasted image 20250330101533.png]]

As if this wasn't bad enough there is also the **/functions/v2:profile.getUsernameByPhone** which is similar to the route above however  **IT RETURNS THE USERNAME ASSOCIATED WITH THE PHONE, WHY IS THIS A THING, JUST WHY**

![[Pasted image 20250330102038.png]]

Again due to lack of proper authentication and rate limits this can be abused by spamming the endpoint with a list of phone numbers and not only will it validate if a phone number exists on the platform it will return the username of the account the phone number is linked to, how secure.

![[Pasted image 20250330102008.png]]


![[Pasted image 20250402120642.png]]
Using a python script I am able to run phone numbers through the API and retrieve the usernames of any accounts registered, this by itself is already quite concerning however when using phone numbers gathered from various data breaches a bad actor is then able to unmask the real life identities of the accounts on the platform leading to further exploitation, extortion and blackmail schemes if they wished. (Note: this returns the username not the display name which does add another layer)
this issue is the same that arises when mass looking up emails using the email validation APIs, while they do not expose the username like when using phone numbers it is still allowing bad actors to identify people who are registered on the Antiland platform and their real life identities.


# Security Via obscurity

Another issue I uncovered during all of this was the developers lack of ability to remove old flawed code from the backend after updates. the original bot project worked on the V1 version of the API however the developers updated to a more feature and slightly more secure V2 API, these changes included things such as the get friends endpoints no longer returning the creation dates of accounts in the friends list which could easily be automated by calling the add friend API then a request to get the list of friends, this V1 api also returns the list of all UUIDs that have the user blocked which is again something that can be automated as most the endpoints only need the UUID to lookup an accounts username.


This data  was clearly removed as creation dates and blocked by data are not supposed to be public on the platform to help with the anonymity (I'm pretty sure the v2 API was developed to directly counter features of the bot library) however what the developers failed to do was disable the old API backend meaning even though the routes are no longer in the main.js they are still available through GitHub, the WayBack machine and any other archives of the site. while this is not a glaring security issue it does raise the question of what other deprecated, outdated and potentially unsecure features still exist on the app but are just being hidden from the user.

![[Pasted image 20250512123810.png]]
example of the V1 API route for getting contacts

![[Pasted image 20250512124157.png]]
the new V2 api in the main.js

![[Pasted image 20250512124355.png]]
v2 api response for getting friends.

As you can see from the below screenshot I am still able to use send requests to this legacy API and retrieve the data that the V2 would not give to me which includes the Users creation and last profile update dates and times and the full list of UUIDs of users that have them blocked.

![[Pasted image 20250512124853.png]]
V1 api request and response via REQBIN

At the time of writing this (12/05/ 25) In true Antiland fashion by researching these issues I have discovered yet another flaw in the V1 contacts API, while looking through the reqbin response I noticed a response returned some usernames of the blocked users and not the UUID which it normally would, this is again a flaw that can be used to unmask user identities as the usernames of accounts (which is what the Phone verifier returns) are not public on the site and are normally not the same as a users display name, while this only seems to happen in small numbers across a large sample this would likely be higher.