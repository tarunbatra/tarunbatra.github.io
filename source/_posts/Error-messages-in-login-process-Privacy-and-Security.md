---
title: 'Error messages in login process: Privacy and Security'
date: 2018-04-29 00:44:03
tags:
  - security
  - architecture
category:
  - security
photos: data/images/Error-messages-in-login-process-Privacy-and-Security/cover.jpeg
---

Most of us, while developing sites swear to the rule of making error messages shown to the end-user as specific as possible, which goes a long way in creating a friendly UX. But the same rule if used in the login flow, can have significant privacy and security implications. On entering wrong login credentials, most sites show variations of `Invalid username/password` error at the login screen.

!['Twitter, Github and Spotify's login error messages'][twitter-github-spotify-image]

It can be safely assumed that no user for such emails exists, yet the error messages shown by these prominent sites don't indicate that. Do you see why?

## Protecting user base information
On feeding random emails, a simple login/reset password may show that the user doesn't exist and therefore no login/reset password is possible. A hostile client may create a list of valid users from this information. Not every site cares about it though. Facebook doesn't.

!['Facebook login error message'][facebook-image]

It doesn't hurt for a site like Facebook to leak the user base because everyone's on it, but when the site is something like Ashley Maddison it can have implications.

### Closing the loop
People often argue that the user base is anyways leaked during the registration process, which can't continue if the user already exists. This leak is avoidable too, by continuing the registration through an email. If the user already exists the email may say so, otherwise it may direct the receiver to continue the registration process.


## Deterrence to brute-force
In a brute-force attack, large number of combinations of usernames and passwords are tried to get into a user's account. A dictionary attack is faster and better as it takes advantage of human tendencies of using common passwords and re-using old passwords. Specific messages like `Invalid username` make these attacks faster by eliminating large number of combinations in a few tries.

### Demo

To quantify the difference detailed error messages can make to brute-force or dictionary attacks, I created a script, [login-attack-demo][script-url] which included:

#### Dictionary
A collection of 1575 most commonly used passwords based on [danielmiessler/SecLists][dictionary-url].

#### Naive server
A server which gives out specific error messages based on the failing stage of the login process. If the username is invalid, it'd say `Invalid username`.

#### Smart server
A server which gives out vague error messages on failure of the login process. If the password is invalid and not the username, it'd still say `Invalid username/password`.

#### Attacker
A hostile client which uses the dictionary to generate a pair of username and password and uses them to break into the system. It also analyzes the error messages on the following rules:

* An error message saying the password was invalid, indicates the username was a valid one.
* An error message saying the username was invalid, indicates no combination of password will result in a successful login.
* An inconclusive error message is ignored and the next combination is tried.

### Results
!['Analysis chart'][chart-image]

The blue points indicate the number of tries it took the attacker to break into the user's account with vague error messages, while the red ones are when the error messages were specific. The yellow line is the max number of combinations to try.

It is evident that in theory vague error messages make it quite tough for the attacker, sometimes as much as 1000 times tougher. However, plugging all the holes may make the UX far from ideal. In the end, it all comes to trade off between UX and security. Depending on the space you're operating, such methods can be used to safeguard the users.

[twitter-github-spotify-image]: /data/images/Error-messages-in-login-process-Privacy-and-Security/twitter-github-spotify.png
[github-ebay-image]: /data/images/Error-messages-in-login-process-Privacy-and-Security/github-ebay.png
[amazon-image]: /data/images/Error-messages-in-login-process-Privacy-and-Security/amazon.png
[facebook-image]: /data/images/Error-messages-in-login-process-Privacy-and-Security/facebook.png
[chart-image]: /data/images/Error-messages-in-login-process-Privacy-and-Security/chart.png
[script-url]: https://github.com/tarunbatra/login-attack-demo
[dictionary-url]: https://github.com/danielmiessler/SecLists/blob/master/Passwords/probable-v2-top1575.txt