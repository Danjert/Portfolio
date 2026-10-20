# TryHackMe Corridor Room

**Tools:** Kali Linux | TryHackMe Lab machine | OpenVPN | John The Ripper | Firefox | Name That Hash

**Introduction:** In this challenge, I will explore potential Insecure Direct Object Reference (IDOR) vulnerabilities . According to OWASP Cheat Sheet Series Insecure Direct Object Reference (IDOR) is a vulnerability that arises when attackers can access or modify objects by manipulating identifiers used in a web application's URLs or parameters. It occurs due to missing access control checks, which fail to verify whether a user should be allowed to access specific data.

## Staging
The **Lab Machine** would act as the vulnerable target of this THM room with which I will interact through a unique IP.

As the **Attacker Machine** I'll use a Kali Linux machine hosted on VirtualBox:

Start the Lab Machine through THM interface and initialize the Kali Machine through VirtualBox.

Establishing the VPN Connection between Lab Machine and Attacker Machine using OpenVPN:
* My first step is to check if OpenVPN is installed by using the terminal :

    ```bash
    openvpn --version
    ```
    ![Corridor 1](./Assets/Corridor1.png)
    **(Was Installed)**

* Download the config file from THM and proceeded to establish the connection by the OpenVPN terminal command and ping Lab Machine IP to verify reach. 
    ```Bash
    #Establishing the connection
    sudo openvpn FileName.ovpn
    ``` 
    ```Bash
    #Verifying the connection
    Ping -c 3 10.67.177.23
    ```
    **(Connection established and Ping was successful)**
<br>

## Process

Now I can access the Lab Machine IP through Firefox and see an image of a corridor with 13 doors:
    ![Corridor 2](./Assets/Corridor2.png)

The only thing that I can do in this web application is to click on the doors which will take me to an empty room where I can just go back to the previous corridor, but I can see that each room have an unique identifier on the URL e.g:
* Corridor: <http://10.67.177.23/>
* First door to the left: <http://10.67.177.23/c4ca4238a0b923820dcc509a6f75849b>
* Door in the middle: <http://10.67.177.23/8f14e45fceea167a5a36dedd4bea2543>

I can see that these identifiers are similar to hashes; I can check them on the terminal by installing **the Name That Hash tool**
```Bash
sudo apt update && sudo apt install name-that-hash -y
```

and using
```Bash
nth -t c4ca4238a0b923820dcc509a6f75849b
```
**which would tell me that most likely this is a MD5 Hash**
    
With this information I can make a text file **hash.txt** and fill it with the hashes for each door for further analysis with the John The Ripper tool
```Bash
#Searching the list of hashes
john --format=raw-md5 hash.txt
```
```Bash
#Showing the results of the search
john --show --format=raw-md5 hash.txt
```
This shows me that each hash corresponds to a number on each door.
    ![Corridor 3](./Assets/Corridor3.png)

Reviewing the explanation of the challenge it says **You have found yourself in a strange corridor. Can you find your way back to where you came?** assuming that I have two options regarding the door where I came from numbers: 14 or 0.

Using the terminal
```Bash
#Generating the hash for 14
"echo -n "14" | md5sum"
```
```Bash
#Generating the hash for 0
"echo -n "0" | md5sum"
```
to generate the hashes **ab3238922bcc25a6f606eb525ffdc56** and **cfcd208495d565ef66e7dff9f98764da** respectively so I can add those to the URL to check if can access something I shouldn't.

The URL for **14** <http://10.67.177.23/ab3238922bcc25a6f606eb525ffdc56> doesn't work
    ![Corridor 4](./Assets/Corridor4.png)

I found the **flag** using the hash for **0** <http://10.67.177.23/cfcd208495d565ef66e7dff9f98764da>
    ![Corridor 5](./Assets/Corridor5.png)
