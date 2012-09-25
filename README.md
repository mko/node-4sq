node-4sq
==================

Fault-tolerant Foursquare API wrapper for Node JS. With slight update.


Install
-------

    npm install node-4sq

Use
---

The Foursquare module takes a configuration parameter containing logging and client keys. It also supports alternate
Foursquare URLs if necessary, (but that is unlikely).

    var config = {
      "secrets" : {
        "clientId" : "CLIENT_ID",
        "clientSecret" : "CLIENT_SECRET",
        "redirectUrl" : "REDIRECT_URL"
      }
    }

    var foursquare = require("node-4sq")(config);

Once instantiated, you just need to set up endpoints on your own server that match your OAuth configuration
in Foursquare.  Using Express, for example:

    var app = express.createServer();

    app.get('/login', function(req, res) {
      res.writeHead(303, { "location": Foursquare.getAuthClientRedirectUrl() });
      res.end();
    });


    app.get('/callback', function (req, res) {
      Foursquare.getAccessToken({
        code: req.query.code
      }, function (error, accessToken) {
        if(error) {
          res.send("An error was thrown: " + error.message);
        }
        else {
          // Save the accessToken and redirect.
        }
      });
    });

Foursquare API Version and Deprecation Warnings
-----------------------------------------------

Foursquare allows consumers to specify a "version" of their API to invoke, based on the date that version became active.
For example, passing a version string of "20110101" uses the API as of Jan 1, 2011.  By default, this library will use
a version of today's date.

To enable a different version of the API, add the following to configuration.

    var config = {
      ...
      "foursquare" : {
        ...
        "version" : "20110101",
        ...
      }
      ...
    }

When using an older API, Foursquare will provide deprecation warnings, (if applicable). By default, this library will
write these warnings to the log, which will only be visible if logging for "node-4sq" is turned on, (see
"Logging", below).

You can configure this library to throw an error instead:

    var config = {
      ...
      "foursquare" : {
        ...
        "warnings" : "ERROR",
        ...
      }
      ...
    }


Logging
-------

This module uses Log4js to log events. By default, everything is set to "OFF" and no appenders are configured. If you
want to output logging messages from the different modules of this library, you can add overrides to your configuration
object.  For example, to log INFO (and higher) messages in Venues to the console:

    var config = {
      "log4js" : {
        "appenders" : [{
          "type" : "console"
        }],
        "levels" : {
          "node-4sq.Venues" : "INFO"
        }
      }
      ...
    }

    var foursquare = require("node-4sq")(config);

For a list of existing logging points, refer to [config-default.js](https://github.com/clintandrewhall/node-4sq/blob/master/lib/config-default.js).

For more information, see: https://github.com/csausdev/log4js-node

Testing
-------

To test, you need to create a config.js file in the /test directory as follows:

    module.exports = {
      "secrets" : {
        "clientId" : "YOUR_CLIENT_ID",
        "clientSecret" : "YOUR_CLIENT_SECRET",
        "redirectUrl" : "http://localhost:3000/callback" // This should also be set in your OAuth profile.
      }
    };

Then, simply invoke the test.js file with Node.JS:

    node test.js

If you hit [http://localhost:3000](http://localhost:3000), you'll be redirected for an authentication token.

If you hit [http://localhost:3000/test](http://localhost:3000/test), you'll test the entire library with no authentication, (and get appropriate
errors for protected endpoints).

If you hit [http://localhost:3000/deprecations](http://localhost:3000/deprecations), you'll test an endpoint with older versions and
errors vs. warnings.

Testing results will be logged to the console.

All tests use examples as suggested by the [Foursquare Endpoint Explorer](https://developer.foursquare.com/docs/explore.html).

Documentation
-------------

Detailed documentation is available in the /docs directory.

(In the latest version of JSDoc 3, the file names are replaced with random identifiers. Not sure why, we'll see if they can get that fixed soon.)

How To Use Write-Enabled
-------------

Essentially: How to Check In.

Unlike its predecessors node-foursquare and node-foursquare-2, node-4sq exposes POST access to the Foursquare API. Honestly, it's not implemented as nicely as it could be, but it works. No convenience methods have been added yet like .CheckIn or anything. Right now, you construct a POST to the API using `Foursquare.Core.postApi` like so:

	Foursquare.Core.postApi('/endpoint', 4SQ_ACCESS_TOKEN, { checkinOptions }, function(error, results) {
		if(error) {
			res.send("An error was thrown: " + error.message);
			return;
		} else {
			res.send(results);
			return;
		}
	});

In this case:

- `/endpoint` is the Foursquare API Endpoint to which you want to POST.
- `4SQ_ACCESS_TOKEN` is your access token retrieved when the user logs into 4sq with your app.
- `checkinOptions` is a JSON object with the configurations for the desired endpoint.

Notes
-----

Needed a write-enabled version for a prototype/demo at an event faster than a pull request could grant, so this version happened. Hopefully, the changes in this will get merged back into node-4sq so as to not continue fragmenting things.

This project is a write-enabled version of: https://github.com/gamebox/node-foursquare
Which is an ever-so-slight enhancement of: https://github.com/clintandrewhall/node-foursquare
Which is a refactoring and enhancement of: https://github.com/yikulju/Foursquare-on-node
