# My stuff that I learn

Commands and information I need to remember

---

# Git

### Revert to an earlier commit 

```sh
git revert <commit sha>
```

Note: Reverts the given commit and you end up at the commit before the specified one.

---

### Remove local changes


#### There could be only three categories of files when we make local changes:


    Type 1. Staged Tracked files

    Type 2. Unstaged Tracked files

    Type 3. Unstaged UnTracked files a.k.a UnTracked files

> * Staged - Those that are moved to staging area/ Added to index
> * Tracked - modified files
> * UnTracked - new files. Always unstaged. If staged, that means they are tracked.

#### How to target each area:

1. ``` git checkout . ``` - Removes Unstaged Tracked files ONLY [Type 2] 

2. ``` git clean -f ``` - Removes Unstaged UnTracked files ONLY [Type 3]

3. ``` git reset --hard ``` - Removes Staged Tracked and UnStaged Tracked files ONLY[Type 1, Type 2]

4. ``` git stash -u ``` - Removes all changes [Type 1, Type 2, Type 3]

---

# Ubuntu BASH shell

-Tips, tricks & hax in the terminal

## MongoDB

### Start up MongoDB

In /lib run:

``` mongod --dbpath=$(echo ~)/data/db &```

### If ERROR "Failed to unlink socket file /tmp/mongodb-27017.sock"

Do this to set permission of the socket file to the current user:

``` sudo chown `whoami` /tmp/mongodb-27017.sock ```


## SSL Certificates

#### For problems with the certifcate chain

Check out this [medium article](https://medium.com/@superseb/get-your-certificate-chain-right-4b117a9c0fce)

#### Local testing with HTTPS protocol in node.js + React 


1. Install [mkcert](https://github.com/FiloSottile/mkcert)
2. Run ``` mkcert localhost 127.0.0.1 ::1 ``` in a new folder in the server project directory
3. Import the newly generated certificates in your app.js like so [HTTPS with node](https://nodejs.org/en/knowledge/HTTP/servers/how-to-create-a-HTTPS-server/)
4. Now go to the root certificate path by typing ``` cd $(mkcert -CAROOT) ```
5. Copy the rootCA.pem file to a directory in the Windows file system, for exmaple:  ``` cp rootCA.pem //mnt/c/users/<user>/desktop ``` 
6. Open Windows management console by running ``` mmc ``` in the start menu
7. Open File > Add Snap-in > double click "Certificates" and choose "for current user" then "Done" and then "OK" so you end up with the list in Console Root
8. Expand "Certificates - Current User" > Trusted Root Certification Authorities > Certificates
9. Select Action > All Tasks > Import and choose rootCA.pem from whatever folder you put it in before
10. If you get a security warning, hit verify
11. Restart computer

Now you backend should be set up for local HTTPS development, in some cases you need to set an environment variable for node to find the rootCA, so run this in the terminal before you start or add it in your code somewhere:  </br> ``` export NODE_EXTRA_CA_CERTS="$(mkcert -CAROOT)/rootCA.pem" ```

#### If you get errors like "could not validate certificate chain" or similar

1. If you generated more than one cert, you must find the other ones and bundle them together like described [here](https://medium.com/@superseb/get-your-certificate-chain-right-4b117a9c0fce)
2. Reinstall ubuntu
3. Reinstall ubuntu with another version

#### HTTPS in the front end (must use same certificates as the server)

``` HTTPS=true SSL_CRT_FILE=cert.crt SSL_KEY_FILE=cert.key npm start ```</br>
or for package.json: </br>
``` HTTPS=true SSL_CRT_FILE=cert.crt SSL_KEY_FILE=cert.key react-scripts start ```</br>

## Running snapd on WSL2 Ubuntu

Assuming snap is installed, execute the following to get it working: </br>
``` sudo apt-get update && sudo apt-get install -yqq daemonize dbus-user-session fontconfig ``` </br>
``` sudo daemonize /usr/bin/unshare --fork --pid --mount-proc /lib/systemd/systemd --system-unit=basic.target ``` </br>
``` exec sudo nsenter -t $(pidof systemd) -a su - $LOGNAME ``` </br>

``` snap version ``` </br>

If terminal crashes on third command: </br>
``` wsl --shutdown ``` in windows cmd/terminal
