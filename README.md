# nodejs-express-letsencrypt

## Tutorial:  Set up Node.js and Express with free Let's Encrypt SSL cert

* Test example is running a React website on a Node.js Digital Ocean droplet: Node.js Express server on Ubuntu 16.04 + React static build.

* Deploy the Node.js droplet
* `mkdir [your domain path]`  this could be something like `/var/www/mywebsite.com/` or `/var/www/demo.mywebsite.com/`
* You could put your visible pages in `/var/www/mywebsite.com/public/` or something similar -- we'll use that for this tutorial.
* The static React site, for this tutorial, would be `/var/www/mywebsite.com/public/build/`
* Note: It is possible to change these paths depending on your needs. We'll keep this convention for the tutorial.
* Check to make sure your Node.js and NPM versions are ok (if oncfigured incorrectly could be lower versions-- you should get recent ones with the Digital Ocean droplet, but others may not give it to you automatically)
* `node -v`
* `npm -v`
* We should install the following Node tools to help us run the server. I am mostly just using forever, but nodemon can also be helpful when developing.
* `npm install forever nodemon -g`
* `npm init public`
* `npm install --save express compression morgan`
* The module compression is for performance (gzip web pages), and morgan may be used for logging.

* You can create a server with one entry file With Node.js and Express - we'll be using server.js

* Top of the server.js file

```javascript
const express = require('express');
const path = require('path');
const app = express();
const http = require('http');
const https = require('https');
const fs = require('fs');
const querystring = require('querystring');
const compression = require('compression');
const bodyParser = require('body-parser');

```

* Next part inside server.js. 

```javascript
app.use(compression()); // GZIP http sent files for performance
app.use(express.static(path.join(__dirname, 'build'))); // using build as root for create-react app
app.use( bodyParser.json() );       // to support JSON-encoded bodies

```

* Next inside server.js. For connecting to static React build and React Router 4, we'll make routing go to public/build/index.html - since we are in public we do __dirname and then build and then index.html


```javascript
// Set route for React build
app.get('/*', function (req, res) {
  res.sendFile(path.join(__dirname, 'build', 'index.html'));
});

```

* Finally the server is on and listening on port 80

```javascript
app.listen(80);
```

* Upload your static create-react-app build directory to `/var/www/mywebsite.com/public/`
* So you should have a build directory now under public, for exmaple: `/var/www/mywebsite.com/public/build/index.html` and `/var/www/mywebsite.com/public/build/manifest.json` etc. 
* At this point we have an http working web server (but not https yet).
* Check it back in `/var/www/mywebsite.com/public/` with `node server.js`. Check your website. http://[your DOMAIN ie. www.mysite.com]
* While `node server.js` is still running, CTRL-C to stop it. Now try `nohup nice sudo forever start server.js &`
* This should keep your server up. To test, logout of your console and check to make sure the site is still up.

* Now to install Let's Encrypt

```
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install certbot 

```
* **Finalize install:**

```
sudo certbot certonly
```

* Please Note the certificate files, which will be listed when your certs are ready. You will need that for putting into the server.js file (as stated below)

**To renew** (it should automatically be set to auto-renew, but you can check with this):

```
sudo certbot renew --dry-run
```

* **LASTLY** In `server.js`:
```
https.createServer({
        key: fs.readFileSync("/etc/letsencrypt/live/[yourdomain]/privkey.pem"),
        cert: fs.readFileSync("/etc/letsencrypt/live/[yourdomain]/fullchain.pem")
}, app).listen(443);

```







* Check the status of you https SSL cert.
* https://www.ssllabs.com/ssltest/analyze.html?d=[put your domain here]
https://www.sslshopper.com/ssl-checker.html#hostname=[put your domain here]
...