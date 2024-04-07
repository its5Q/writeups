***
**Author**: [its5Q](https://github.com/its5Q)
***
Many people have been asking me to make a write-up, and I decided it would be great to make one to show people how you could solve those tasks using exclusively free and open-source software. It's very much possible, and very painful (just kidding, it was still fun, but certainly a lot harder).

## Main software used

All the solving was done on a Windows machine with some help of Ubuntu running in WSL2 for some of the CLI tools.

For iOS forensics I used:
- [iLEAPP](https://github.com/abrignoni/iLEAPP)
- [ArtEx](https://doubleblak.com/app.php?id=ArtEx2) (it's not open-source as far as I'm aware, but still completely free nonetheless)

For Windows forensics I used:
- [Autopsy](https://www.autopsy.com/) (also, as someone pointed out to me, it includes iLEAPP as a plugin and can ingest iOS images, so you could just use Autopsy for everything, but I didn't know that at the time and used iLEAPP separately)

There's, of course, much more stuff used that'll be mentioned in the write-up for each individual challenge, but those are the main ones that were very handy.

## Ident

**Difficulty**: Baby  
**Description**: **What is the Apple ID used on the imaged iPhone?**
***
I extracted the iOS disk image and loaded it up in a simple iLEAPP GUI that's included with it, which provided me with an HTML report on the image.

Heading into the *Account Data* tab, we can see the accounts that were used on that phone, that's where we get the Apple ID from.

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408000057.png?raw=true)

The flag: `billthemegakill@icloud.com`


## Namedrop

**Difficulty**: Baby  
**Description**: **What is the iPhone owner's full name?**
***
Scrolling a little bit down in the iLEAPP's report tabs, we can find the *Address book*. Opening that, we see the full name of the iPhone owner.

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408000435.png?raw=true)

The flag: `William Phorger` (I see what you did with the name😉)


## Conspirators

**Difficulty**: Warmup  
**Description**: **Which Telegram accounts did the owner discuss shady stuff with?**
***
Thankfully, iLEAPP also supports reading Telegram messages, so we quickly hop into the *Telegram - Messages* tab, and there we could see all communications with his "business" partners as well as their Telegram usernames. Clicking through the pages and noting all his shady contacts, we get the flag.

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408000925.png?raw=true)

The flag: `@diddyflowers, @Sm00thOperat0r, @JesusStreeton1999, @locknload771`


## Visit

**Difficulty**: Tricky  
**Description**: **Where does William live?**
***
This one is, as the difficulty suggests, a bit trickier, and requires you to do a little bit of manual digging. Looking through the data in iLEAPP, it doesn't seem like there's any geolocation data that could point to the William's home location, so we must dig deeper.

Looking at the list of applications installed on the phone in iLEAPP (*Apps - Itunes metadata* tab), I quickly spotted an Uber app installed, so I decided to see what kind of data it stores. After navigating to the app's container path + /Documents, which I found searching for folders with "com.ubercab.UberClient" in their name, we see two SQLite databases. Opening `database.db` in [DB Browser for SQLite](https://sqlitebrowser.org/) and navigating to the `fts_place_table` or `place` tables, we see the home address.

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408002939.png?raw=true)

The flag: `38.58976434380295, -90.32529780538323`


## Username

**Difficulty**: Baby  
**Description**: **What is the username of the laptop user?**
***
Now we get access to the laptop's Windows disk image. I get the username by loading the image in Autopsy and navigating to the Users folder. Alternatively, you can navigate to the "OS Accounts" tab.

The flag: `phorger`


## April Paycheck

**Difficulty**: Warmup  
**Description**: **What is the amount of William's first take in April?**
***
This one was quite challenging without Belkasoft X. In fact, it was my second-to-last challenge that I've solved. The way it was solved using Belkasoft is by looking at extracted Element messages, where they were talking about the April's paycheck, but I couldn't find any open software that would allow decrypting and reading cached Element messages, so here's my solution.

First of all, I found a screenshot of some bank transactions in the user's Pictures folder. Then, the intended way was using the [aCropalypse](https://en.wikipedia.org/wiki/ACropalypse) vulnerability in the Windows' screenshot tool to get the full screenshot, but I found a much easier (unintended) way - the full screenshot was cached by the snipping tool, and I was able to find it in the *Images* tab of files sorted by filetype.

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408004404.png?raw=true)

The URL on the screenshot is actually wrong as the challenge authors didn't anticipate this way of solving. The correct URL for the bank is https://crbk.org, which you can find in the *Safari history* of the iPhone image.

After navigating to the bank URL and resetting the password with his username and card number, we see the latest transactions with one incoming from Maximilian Senntens - that's the one we need.

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408004959.png?raw=true)

The flag: `7012.39`


## Party

**Difficulty**: Tricky  
**Description**: **Where did the gang go to celebrate their success together in March?**
***
In this one, we return to the iOS image. Looking at other apps installed, there's an app called *Splitwise*, which is an app that eases the process of splitting the bill between multiple people.

Repeating the same steps like we did with Uber, we go to the app's folder and look for databases that might contain valuable information. We find one at `Library/Application Support`, and looking through it we find the `SWExpense` table which lists all the expenses created in the app, one of which is "Get-together at Cunetto".

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408005600.png?raw=true)

From the Telegram messages we extracted using iLEAPP, we can learn that the gang celebrated their success on the 20th of March, which matches the date in the database. Looking up the place on Google Maps, we get the coordinates.

The flag: `38.61034,-90.28030`


## Crypto

**Difficulty**: Warmup  
**Description**: **Which file does the guy keep his encrypted container in?**
***
This one I solved just by looking at files grouped by size in Autopsy, and there it was, in the `200MB - 1GB` tab.

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408005834.png?raw=true)

The flag: `C:\Users\phorger\Documents\desktop.ini:vault.vhdx`


## Luxury

**Difficulty**: Warmup  
**Description**: **Which luxurious item did Phorger put his laundered money into?**
***
I assumed that this task would require access to the encrypted volume, as it's right after the task where you need to find that volume, so I got to digging for the password or the recovery key. 

Firstly, I saw that there was a text file in the *Recent Documents* tab named `BitLocker Recovery Key 1C3C8323-FED0-45C9-8A56-BFA3D4871811.TXT`. After that, I spent quite some time looking for the recovery key by running the `strings` utility with `grep` over the whole image, thinking it was possibly deleted but not overwritten, but I didn't have much success there.

Then when looking in the iOS image, I remembered about the Notes app. iLEAPP doesn't support note extraction, so I had to use ArtEx for that. Selecting `Notes` in the `TimeLine` tab, I see 6 events, with one being the note with the Bitlocker Recovery key - bingo!

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408010917.png?raw=true)

I mounted the image on my PC with the recovery key by double-clicking the vhdx file, and I found a bunch of PDFs with printer manuals, as well as a `Spending.xlsx` file. In there, we find the most expensive item he bought. 

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408011127.png?raw=true)

The flag: `Rolex Submariner Date ref 126619LB`


## Vacation

**Difficulty**: Tricky  
**Description**: **Which concert were Phorger and his girlfriend planning to attend in May?**
***
This one was, indeed, tricky. It took a while to find, and checking the Firefox cache was kind of a Hail Mary to see if, perhaps, there was anything useful there. And, indeed, there was. 

Autopsy can show the list of cached files, but, as far as I'm aware, it can only extract cache metadata, not the file content. So, with that knowledge, I turned to Google and found an awesome tool [snappy-fox](https://github.com/berdav/snappy-fox) for decompressing Firefox morgue cache files. 

I ran that tool over all of morgue files from Telegram Web. This gave me a bunch of decompressed files with no extensions, and as a first step, I added the `.png` extension to all the files using `PowerRename` from [Microsoft PowerToys](https://github.com/microsoft/PowerToys) so Windows Explorer would load image previews for all the files that were actually images. 

Just like that, I find the cover image for a concert in Paris on the 26th of May, thus solving the challenge (you can see the cached filename on the screenshot).

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408012113.png?raw=true)

The flag: `Eric Clapton, Accor Arena, Paris`


## Illustrator

**Difficulty**: Warmup  
**Description**: **What's the name of the person who designed the print template for the bills?**
***
From the Telegram messages we know that William's girlfriend designed the print template and sent it to him on Telegram. At first, I looked if I could find the file somewhere in the iOS dump, before I even had access to the encrypted container, but to no avail. 

Then, after I got the recovery key for the container, I found the Telegram bot sources that was used to control money printing, and there was that Photoshop template that was sent over Telegram. 

Looking at the metadata of the template using `exiftool`, the `History` field catches my attention, containing the absolute path to the PSD file on Drew's machine, and that's how we get the girl's full name.

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408013105.png?raw=true)

The flag: `Drew Linesworth` *(love the wordplay)*


## Homebrew Lab

**Difficulty**: Tricky  
**Description**: **Where is the makeshift lab where they printed the cash located?**
***
For this task I focused on the iOS image as that's where I assumed I would find that info. Looking through locations parsed by iLEAPP and ArtEx, I couldn't find a matching location, so I continued scrambling through the reports, hoping to find a lead. 

And find the lead I did. After looking through the iLEAPP report multiple times, I found something that I overlooked that turned out crucial for my success - **App Snapshots**. This report contains cached screenshot of previously run applications that are displayed when you scroll through the running apps list on your iPhone. 

Examining these cached screenshots, I find a screenshot of Apple Maps, but it's probably not possible to pinpoint the exact house from that screenshot. Then, after going through some more screenshots, I find one of the Reminders app, which reminds William to "turn off the fume hood", with a Geofence attached that has the address of the lab. Challenge solved.

Also, turns out that Geofences extraction is a feature in ArtEx, but for some reason, it didn't work on this image, so this could've a bit easier if it wasn't for that.

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408015241.png?raw=true)

The flag: `900 N 88th St, East St Louis, IL`


## Largest Batch

**Difficulty**: Warmup  
**Description**: **What is the precise moment their largest printing batch was completed?**
***
That one was straightforward. I looked through the Telegram message logs and saw a "baker" bot that William had setup to control the printing process. With that information, I looked at the largest order he made in that bot, and took the timestamp of the message that the bot sent when it was ready. 

There was one pitfall though. Turns out that, despite iLEAPP saying that all the timestamps were displayed in UTC, timestamps for Telegram messages were displayed in local time, so to submit the flag I had to add my timezone to the end.

The flag: `2024-04-02 22:44:30 UTC`


## Device

**Difficulty**: Tricky  
**Description**: **What's the printer model they used to print money?**
***
That one was easy too, I quickly figured it out by going to the `USB Devices Attached` tab in Autopsy and there it was, in its full glory.

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408020102.png?raw=true)

The flag: `HP LaserJet M1132 MFP`


## Night Shift

**Difficulty**: Tricky  
**Description**: **Which ATM did Phorger test his bills on recently?**
***
On this one, I've also looked at all locations extracted with iLEAPP and ArtEx, but couldn't find anything of value for this task. I also tried to search for the bit.ly URL that was send to William in the Internet Archive in hopes that it was archived, but no luck there.

Skimming through the iLEAPP report, WiFi network data peaked my interest, as I could use WiGLE to find access point locations. There were only 2 known network, the first named TWR9 that was the most used network, presumably at his home, and a second one - UCPLPublicWireless, which is a public network of the Univercity City Public Library. The dates correlated nicely with the panicked Telegram messages, so I looked up the nearest ATM to that library on Google Maps. Another one off the list.

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408020940.png?raw=true)

The flag: `Regions Bank on Delmar Blvd`


## Mole

**Difficulty**: Hard  
**Description**: **Who leaked the technical data on the bill validator to the gang?**
***
This task required some creative thinking with the data you had. I found the first clue to solving the task by finding an entry in the `Recent Documents` in Autopsy, which pointed to the Y: drive, that is the encrypted BitLocker container we found earlier.

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408021407.png?raw=true)

However, if we navigate to that vault, we don't see that PDF. Another little clue that I had found is in the browser history:

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408021642.png?raw=true)

This got me thinking that I possibly need to recover deleted files from the vault, and I did so using [Recuva](https://www.ccleaner.com/recuva) because that's what I already had installed previously. This yielded me several PDF files, one of which is the one we were looking for. I know that PDFs often have digital signatures, so at that point I was pretty certain that I needed to look at the digital signature to figure out the leaker.

I opened the PDF in Adobe Acrobat, and it immediately complained to me about the invalid digital signature, as well as generously displaying the signer's name. 17:0 for the detective :)

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408022241.png?raw=true)

The flag: `Kenneth Leek` *(the jokes never stop)*


## Financial Institution

**Difficulty**: Hard  
**Description**: **Which offshore financial institution did the gang bank with?**
***
I was surprised by such low amount of solves on this challenge, as it turned out not as hard as I initially thought. The way I solved this challenge is by decrypting the messages that were exchanged between Mr. C and William. I did so by looking at their previous messages and seeing them talk about Shortcuts for encryption. Googling "Apple Shortcut encryption" I find out that there's an app called Shortcuts that allows to make custom scenarios to automate stuff.

Armed with that knowledge, I find the Shortcuts SQLite database at `/var/mobile/Library/Shortcuts/Shortcuts.sqlite` and open it up. I find the SecuEncrypt and SecuDecrypt shortcuts in the `ZSHORTCUT` table, and in the `ZSHORTCUTACTIONS` I find the code that powers those shortcuts. It's some JavaScript embedded in a plist format encoded in UTF-16, but I didn't bother to use tools to parse the plist and just opened that SQLite database in [EmEditor](https://www.emeditor.com/) (fancy Notepad) with UTF-16LE encoding and ripped out the code from there.

![img](https://github.com/its5Q/writeups/blob/master/BelkaCTF%206/img/Pasted%20image%2020240408024057.png?raw=true)

Analyzing the code, I saw that there are integer variables `a` and `b` derived from the key string, and their values are bound to range `[0;255]`, so it's very easy to bruteforce them as it's just ~65535  combinations. I modified the decryption code to cycle through all possible values of those numbers and try to decrypt the message, and if the message contains strictly printable ASCII, it outputs it to the console.

Going one by one through the encrypted messages and decrypting them, I strike gold and find the following message: `This one is for you personally. Bank Name: Crooked River Bank. Account Name: Consulting Services Inc. Account Number: 9876543210. SWIFT Code: CRVBPA2P. Bank Address: 50 Offshore Blvd, Panama City, Panama`

The flag: `CRVBPA2P`



## Statement

**Difficulty**: Hard  
**Description**: **Paste Phorger's entire bank statement here, containing all his offshore transactions**
***
We have access to the bank account from past challenges, so let's press the download button and download all the transactions.

Uh-oh, what's that? I need 2FA? Well, that's a bummer. How am I going to get the code?

Turns out - it's not that hard :) Long before I unlocked this challenge, I spotted an iTunes backup on the laptop's image, so I kept that in the back of my mind. While looking for the info on other challenges, I decided to run my toolkits on that backup aswell, hoping to find a little more info than from the image provided by Belkasoft. 

In the iLEAPP report, I notice a new photo, that conveniently turns out to be a screenshot of the Google Authenticator's migration QR-code. I opened Google Authenticator on my phone, scanned the QR code, and now I could generate my own 2FA codes, and that allowed me to download the full transaction data.

The flag: a bit too big to paste here, I will leave that as an exercise to the reader :)