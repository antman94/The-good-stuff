# My stuff that I learn

Commands and information I need to remember

---

# Git

### Revert to an earlier commit 

```sh
git revert <commit sha>
```

Note: Reverts the given commit and you end up at the commit before the specified one.


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

![image](https://user-images.githubusercontent.com/13006925/115860840-5ff2cd80-a432-11eb-983d-2f41e13b0976.png)

### Remove remote files or folders

1. (FILES) ``` git rm --cached <filename> ``` // (Folders) ``` git rm --cached -r <dir_name> ``` 
2. ``` git commit -m "Removed folder from repository" ```
3. ``` git push origin master ```

---


# Ubuntu BASH shell

-Tips, tricks & hax in the terminal

## MongoDB

### Start up MongoDB

In /lib run:

```bash
mongod --dbpath=$(echo ~)/data/db &
```

### If ERROR "Failed to unlink socket file /tmp/mongodb-27017.sock"

Do this to set permission of the socket file to the current user:

```bash 
sudo chown `whoami` /tmp/mongodb-27017.sock 
```


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
```shell 
sudo apt-get update && sudo apt-get install -yqq daemonize dbus-user-session fontconfig 
```

</br>

```shell 
sudo daemonize /usr/bin/unshare --fork --pid --mount-proc /lib/systemd/systemd --system-unit=basic.target 
``` 

</br>

```shell 
exec sudo nsenter -t $(pidof systemd) -a su - $LOGNAME 
``` 
</br>

```shell 
snap version
``` 

</br>

If terminal crashes on third command: </br>
``` wsl --shutdown ``` in windows cmd/terminal

---

# Windows terminal (powershell)

## CURL

### Parsing a JSON response into more readable format, and selecting which parts to display

```powershell 
$request = 'http://localhost:5000/test' 
``` 

<br/>

```powershell 
Invoke-WebRequest $request -UseBasicParsing | ConvertFrom-Json | Select name country 
``` 

<br/>
Result:

```powershell
name        country
-----       -----
ante        sweden
```

---

# NodeJS

## Firebase

### Clear an entire collection in firestore

```javascript
function clearCollection(path) {
  const ref = db.collection(path)
  ref.onSnapshot((snapshot) => {
    snapshot.docs.forEach((doc) => {
      ref.doc(doc.id).delete()
    })
  })
}
// Use it like this:
clearCollection('meetings') 
```

---

# Docker

## Deploy React app

Set up dockerfile like this

```dockerfile
FROM node:14.1-alpine AS builder

WORKDIR /opt/web
COPY package.json ./
RUN npm install

ENV PATH="./node_modules/.bin:$PATH"

COPY . ./
RUN npm run build

FROM nginx:1.17-alpine
RUN apk --no-cache add curl
RUN curl -L https://github.com/a8m/envsubst/releases/download/v1.1.0/envsubst-`uname -s`-`uname -m` -o envsubst && \
    chmod +x envsubst && \
    mv envsubst /usr/local/bin
COPY --from=builder /opt/web/nginx.config /etc/nginx/nginx.template
CMD ["/bin/sh", "-c", "envsubst < /etc/nginx/nginx.template > /etc/nginx/conf.d/default.conf && nginx -g 'daemon off;'"]
COPY --from=builder /opt/web/build /usr/share/nginx/html
```

Create nginx.config like this

```
server {
    listen       ${PORT:-80};
    server_name  _;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $$uri /index.html;
    }
}
```

And static.json like this

```json 
{
  "headers": {
    "/**": {
      "Content-Security-Policy": "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; font-src 'self' data:; connect-src 'self' https://*.okta.com;",
      "Referrer-Policy": "no-referrer, strict-origin-when-cross-origin",
      "Strict-Transport-Security": "max-age=63072000; includeSubDomains",
      "X-Content-Type-Options": "nosniff",
      "X-Frame-Options": "DENY",
      "X-XSS-Protection": "1; mode=block",
      "Feature-Policy": "accelerometer 'none'; camera 'none'; microphone 'none'"
    }
  },
  "https_only": true,
  "root": "build/",
  "routes": {
    "/**": "index.html"
  }
}
```

1. Check that docker daemon is running `` docker ps ``
2. `` cd `` to project root
3. Run ``` docker build -t react-docker . ```

---

# IntelliJ

## Important notes about Intellij + Java + Maven in WSL2 
* Tested combination of versions that works is Oracle Java 17.0.1 with Maven 3.8.3 
* Java and Maven must be installed on the same platform, i.e in only windows or only WSL2 Ubuntu distro file system.
* .bashrc needs to contain the following:  and ``export JAVA_HOME="/usr/lib/jvm/java-17-oracle"`` depending on your paths. You can check them with ``mvn -v `` and ``java --version``
* When giving paths to files inside Intellij, for example an environmental variable config file, the path must consist of forward slashes '/' and not backward slashes '\' or it won't be found.

## Set up Java + Maven project in WSL
Note: IntelliJ will only run with WSL2

### Checklist
### Install maven
Do NOT install maven with apt manager, this causes some strange bugs sometimes. Better download it with for example wget and unzip yourself. </br>
For version 3.8.3: ``sudo wget https://dlcdn.apache.org/maven/maven-3/3.8.3/binaries/apache-maven-3.8.3-bin.tar.gz -P /opt`` to download maven to /opt folder in root. </br>
Then ``sudo tar xzvf apache-maven-3.8.3-bin.tar.gz`` to unzip it in the same (opt) folder. </br>
Also add maven to your PATH: </br>
In /home ``sudo vim .bashrc`` and add ``export PATH=/opt/apache-maven-3.8.3/bin:$PATH`` somewhere in the bottom </br>
Then inside IntelliJ -  </br>
* Set maven home path with Ctrl+alt+S > Build,Execution,Deployment > Build Tools > Maven - Maven Home Path and set to ``\\wsl$\Ubuntu\opt\apache-maven-3.8.3\`` or whatever your path to maven is.
* If project root is a folder inside git root folder - add the project root as a module

### Firewall rules required for WSL and IntelliJ
If the following message is shown in IntelliJ when attempted build, check the following steps. </br>
```python
Abnormal build process termination: Cannot establish network connection from WSL to Windows host (could be blocked by firewall).
```
In admin Powershell: </br>
1. Allow traffic from WSL to windows: </br>
```powershell 
New-NetFirewallRule -DisplayName "WSL" -Direction Inbound  -InterfaceAlias "vEthernet (WSL)"  -Action Allow
```
2. Remove blocking inbound traffic to IDEA: </br>
```powershell
Get-NetFirewallProfile -Name Public | Get-NetFirewallRule | where DisplayName -ILike "IntelliJ IDEA*" | Remove-NetFirewallRule
```

Other problems related to WSL2 + maven ``https://youtrack.jetbrains.com/issue/IDEA-266670``

---

# Postgres

## Postgres in WSL2

https://harshityadav95.medium.com/postgresql-in-windows-subsystem-for-linux-wsl-6dc751ac1ff3 </br>
https://chloesun.medium.com/set-up-postgresql-on-wsl2-and-connect-to-postgresql-with-pgadmin-on-windows-ca7f0b7f38ab </br>
