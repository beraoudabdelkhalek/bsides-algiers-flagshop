# bsides-algiers-2023-flagshop
writeup for the flag shop challenge in the web category from bsides algiers ctf
# info about the challenge
- description: They got some fancy flags out there, but that one really takes time to get.
- solves: 7
- author: ouxs
- the code provides is in app.py and utils.py
# solution
[home page](./writeup1.png?raw=true "home")
  when we first register a user to the website ,we see in the home page 3 flags and to solve the challenge we need to by the flag that has a price of 40.99.\
  we can see that we have an initial balance of 10 and we have an option to redeem a ticket, in order to see the format of the tickets\
we must see the code ,in app.py after checking the /redeem route
