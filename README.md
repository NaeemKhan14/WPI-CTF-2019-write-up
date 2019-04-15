This is a write up from my participation in WPI's CTF competition. I got placed at 42nd place out of 586 teams with a score of 981. The event had a number of challenges, and I will be explaining the ones that I managed to clear in this document.

# Cryptography
**1. zoomercrypt**

###### Descrption
> My daughter is using a coded language to hide her activities from us!!!! Please, help us find out what she is hiding!

![Zoomer Crypt](https://github.com/NaeemKhan14/WPI-CTF-2019-write-up/blob/master/zoomercrypt.jpg)

We were given the above picture to decode the message from. I converted all emojis into unicode first, which resulted in a string like this:

    U+1F603 U+1F601 U+1F615
    
I noticed that all of them have one thing in common: **U+1F6**, which leads to the conclusion we only need the last two bits of the code, which resulted into the following series of numbers:

    03 01 15 17 08 17 07 0B 04 17 { 06 13 04 13 02 08_ 0E 03 03 01 13 06 07}
    
Then tracing each number to their alphabetical position, we get:

    DBP RI RH LER{GNENCI_ODDBNGH}
    
Now in the picture message, it mentions that the message using ROT15 substitution cipher, so we shift each letter 15 positions backwards to get:

    OMA CT CS WPC{RYPYNT_ZOOMYRS}
    
Now I was stuck at this point for a long time. I do not know if I made a mistake somehow in the process of decrypting, or it was intentional, but the resulting string had 2 letters wrong. After thinking about it for a while, I realized that the name of the challenge is "**zoomer**crypt", so our decrypted flag should be OMA CT CS WPC{RYPYNT_ZOOM**E**RS}, which made sense. Hence after replacing every instance of letter Y, we get:

    OMA CT CS WPC{REPENT_ZOOMERS}
    
Which now made a lot of sense. So after changing the WPC to WPI (which is the flag format), I successfully managed to decrypt the message!

    Flag: WPI{REPENT_ZOOMERS}
    
**2. jocipher**

###### Descrption
> Decrypt PIY{zsxh-sqrvufwh-nfgl} to get the flag!

We were given a compiled python file which the text was encrpted with. I used **uncompyle6** python decrypter to decompile the file to get access to its source code. That resulted in the following code:

> https://github.com/NaeemKhan14/WPI-CTF-2019-write-up/blob/master/jocipher_Decompiled
    
I modified the code slightly by adding the following code in the condition at line #133, to test a number of range for the correct seed to decrypt with:

    if args.decode:
      for i in range(50):
      print(i)
      ret = decode(args.string, i)

With that, the flag was found at seed #48.

    Flag: WPI{xkcd-keyboard-mash}
    
# Reversing
**1. strings**

For this challenge, we were given an execuable file to run. I opened the file in a text editor and search for the key word "WPI" and found the flag in it.

    Flag: WPI{What_do_you_mean_I_SEE_AHH_SKI}
    
# Recon
**1. Chrip**

Perhaps the easiest challenge of them all. We were given a picture of a blue bird, with a watermark "Seige" written on it. That hinted that the answer is somewhere on twitter. Upon searching for Seige Technologies's (which were the sponsor of this event) twitter page, I found the key in their tweet.

    Flag: WPI{sp0nsored_by_si3ge}
    
# Web
**1. getaflag**

###### Descrption

    Come on down and get your flag, all you have to do is enter the correct password ...

    http://getaflag.wpictf.xyz:31337/ (or 31338 or 31339)
    
The challenge required us to guess the correct password. Upon inspecting the source of the page, we found an encrpted key commented out:

> \<!-- SGV5IEdvdXRoYW0sIGRvbid0IGZvcmdldCB0byBibG9jayAvYXV0aC5waHAgYWZ0ZXIgeW91IHVwbG9hZCB0aGlzIGNoYWxsZW5nZSA7KQ== -->

The trailing == suggest that it is a base64 encoding, so after decrypting it, I found the message:

> Hey Goutham, don't forget to block /auth.php after you upload this challenge ;)

That lead us to /auth.php page which contained the hints to solve this challenge. The page contained the following pseudo code:
    
      // Pseudocode
      $passcode = '???';
      $flag = '????'

      extract($_GET);
      if (($input is detected)) {
        if ($input === get_contents($passcode)) {
          return $flag
        } else {
          echo "Invalid ... Please try again!"
        }
      }
      
This hint shows us how to authentication works. So by manipulating the data sent to the website, I passed empty value for **input** and **passcode** which resulted in the comparison to be successful, resulting it in showing us the flag. The URL was passed onto auth.php as follows:

    http://getaflag.wpictf.xyz:31337/?input=&passcode=
    
The leads to the success page, but the flag itself is outputted as a console log using JavaScript. So I had to open the browser console to get the flag.

    Flag: WPI{1_l0v3_PHP}
    
# Linux
**1. suckmore-shell**

###### Descrption
    Here at Suckmore Software we are committed to delivering a truly unparalleled user experience. Help us out by testing our latest project.

    ssh ctf@107.21.60.114
    pass: i'm a real hacker now

    Brought to you by acurless and SuckMore Software, a division of WPI Digital Holdings Ltd.

After logging in through ssh, I searched for every file in every directory of the path which contained the string "WPI" using the **grep -r WPI /PATH**. After a number of attempts, I found the flag to be hidden in /home/ctf/flag file. 

    Flag: WPI{bash_sucks0194342}

**2. pseudo-random**

###### Descrption
    
    ssh ctf@prand.wpictf.xyz
    pass: random doesn't always mean random

    made by acurless

Probably the hardest challenge for me. The name of the challenge suggested that the task has something to do with linux's pseudo-random generator /dev/random and /dev/urandom. After trying to use urandom a couple of times, I realized it is giving me the same result instead of being random. When I ran the command **file** on urandom, it suggested it is an ascii text file, which lead to the conclusion that this file has been modified. When I inspected /dev/random using the **file** command, it described it as **openssl enc'd data with salted password**. That lead me to the conclusion that this file has been encrpted using openSSL and /dev/urandom is the key (password) file for it. So I used the command **openssl enc -aes-256-cbc -d -salt -in random -kfile urandom** which showed the following decrypted message with the flag:


    Being holy in our church means installing a wholly free operating system--GNU/Linux is a good choice--and not putting any non-free software on your computer. Join the Church of Emacs, and you too can be a saint!
    And lo, it came to pass, that the neophyte encountered the Beplattered One and humbly posed the question "Oh great master, is it a sin to use vi?" And St. IGNUcuis dist thus reply unto him, "No, my young hacker friend, it is not a sin. It is a penance."
    WPI{@11_Ur_d3v1c3s_r_b3l0ng_2_us}
    
# Miscellaneous
**1. Remy's Epic Adventure**

###### Descrption
    
    A tribute to the former God Emperor of CSC and Mankind, Ultimate Protector of the CS Department, and Executor of Lord Craig Shue's Divine will.

    https://drive.google.com/drive/folders/1RmSgqBhd-_8CFYvvdJXqGDbvzjE3bcwG?usp=sharing

    Author: Justo

This was one of the most fun and most frustrating challenge of them all. We were given a game to beat. In the final level of the game, there is a boss which we have to beat to get the flag. After trying to kill it for an hour, I used a small python script using Pynput which left clicks every 0.001 ms. Even with that, after 20 minutes of trying I could not kill the boss. Which made me realise that challenge requires us to kill it by other means. I used a tool called **Cheat Engine**. This tool lets you collect information about running processes, and lets you manipulate values stored in the registers to modify the game behavior. After loading the game in cheat engine, I found that the boss has 2147483646 HP (no wonder I couldn't kill it!). So I changed the value of its HP to 0 and that instantly killed it. After getting out of the stage, the flag is displayed on the screen to be written down.

    Flag: WPI{j0in_th3_illumin@ti_w1th_m3}
    
# PWN
**1. Source pt1**

###### Descrption
    ssh source@source.wpictf.xyz -p 31337 (or 31338 or 31339).
    Password is sourcelocker

    Here is your babybuff.

    made by awg

This task required us to log into their server using the provided details. Upon login, you are promoted to write a password to gain access to the source code:

> Enter the password to get access to https://www.imdb.com/title/tt0945513/

Upon entering a wrong password, the connection closes with the following error:

    Pasword auth failed
    exiting
    Connection to source.wpictf.xyz closed.
    
So I tried to find the length of the password. After several tries, I noticed that the connection simply closes if you enter a password above a certain length; without any error. That made me narrow the length of the password down to around 110 characters. Upon entering a random string of length 109 characters as password, the source of the program is displayed where the flag is hidden in a comment. Upon inspecting the source code, I noticed that the length of the password is 100 characters, and by entering a certain amount of characters, we overflowed the buffer; giving us access to the source code.

    Flag: WPI{Typos_are_GrEaT!}
    
