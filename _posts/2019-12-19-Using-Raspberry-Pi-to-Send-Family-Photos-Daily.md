---
layout: post
title: Using Raspberry Pi to Send Photos Daily
date: 2019-12-19
image: post1_2.jpeg
---

In this post, I'm going to show you how to automatically send an email every day containing a randomly selected photo and inspirational quote(the quote of the day from BrainyQuote). This works really well for family photos or friend groups. I've found this to be one of the best gifts as unlike most gifts it can't be put away in a closet and forgotten about. In addition, it makes good use of those millions of photos and reminds the family or friends of fun times and memories.

We will do this using a Raspberry Pi and Python. We will use Beauitful Soup(bs4) and requests to get the quotes, smtplib and email to send the email, and os and random to get the random images.

Let's begin by setting up our directory on the rpi.

`mkdir dailyPhotos`


First, find a couple hundred photos that you would like to use. You then need to transfer these photo into a directory on your rpi.

I'll show you one method that works if you are using ssh but there are definitely other ways such as using dropbox or a usb stick.

This method will only work if ssh is enabled on your rpi.

Firstly, on your computer, create a directory with your photos. Then create a zip of the directory that contains the photos. You can usually do this by left-clicking on the directory.

Now, we're going to transfer the zip file from your computer to the rpi using scp.

Run this on your computer: 

`scp path_to_pics.zip pi@raspberrypi.local:/home/pi/dailyPhotos`

Now, unzip the zip file using 

`unzip pics.zip`


You should now have your photos on your rpi!

Now, we will setup the code to actually do the emailing.

Open a blank file using `nano script.py`

Now copy in the following script.
Be sure to change the emails and password. 

```python
import os
from bs4 import BeautifulSoup
import requests
import smtplib
import email
from email.mime.image import MIMEImage
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.base import MIMEBase
import random

###EDIT THIS###
recipients = {"sample.email@gmail.com", "sample123@gmail.com"}
sender = "sample1234@gmail.com"
senderPass = "password"
emailHost = "smtp.gmail.com"
##for yahoo change to "smtp.mail.yahoo.com"
###############

#create email
msg = MIMEMultipart()
msg['Subject'] = 'Photo of the Day'
msg['From'] = sender
#seperate emails with commas but make one string
msg['To'] = ", ".join(recipients)

#choose random photo and add to email
path = "/home/pi/dailyPhotos/pics"
files = os.listdir(path)
index = random.randrange(0, len(files))
fp = open(path+"/"+files[index], 'rb')
img = MIMEImage(fp.read())
msg.attach(img)

#find quote
r = requests.Session().get('https://www.brainyquote.com/quote_of_the_day')
soup = BeautifulSoup(r.text, 'html.parser')
l = "\n\n"+soup.find("meta", {"name": "twitter:description"})['content']
t = MIMEText(l)
msg.attach(t)

#send email
s = smtplib.SMTP(host=emailHost, port=587)
s.starttls()
s.login(sender, senderPass)
s.sendmail(sender, recipients, msg.as_string())


```
Now we need to install bs4(Beautiful soup)

`sudo apt-get install python3-bs4`

We can now send emails! You can test the script by running `python3 script.py`

If you get a SMTP authentication error, this means you need to log in to the sender google account and under security in the manage your account settings, turn off two-factor authentication and turn on less secure app access. 
If you want to keep two-factor authentication, you can generate an app password but its probably easier to just create a new google account with the laxer security settings to send emails from.

All that is left to do is to automate it so emails are sent daily. We will use crontab to do this.

Open crontab using `crontab -e`
At the bottom of the file, add 
`12 0 * * * python3 /home/pi/dailyPhotos/script.py`

This will execute the command `python3 /home/pi/dailyPhotos/script.py` every day at 12:00, which means an email will be sent every day at 12!

You may also want to check that your rpi clock is set to the right time zone to ensure you receive the emails at the correct time.
