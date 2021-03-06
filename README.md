#node-ses
==========

A simple and reliable Node.js mail for sending mail through Amazon SES.

## Benefits

 * Does only one thing and does it well. Only the [SendEmail](http://docs.aws.amazon.com/ses/latest/APIReference/API_SendEmail.html) API method is implemented.
 * Good error handling:
   * Only "2xx" and "3xx" resposnes from Amazon are considered successful.
   * Amazon's XML format errors are converted to JavaScript options for easy handling.
   * Support for the `debug` module is included if [debugging](#debugging) is needed.
 * Tested and reliable. Includes test suite. Sending email to SES since 2012.

## Synopsis

_This module implements the SendEmail action only. What more do you need? ;)_

```js
var ses = require('node-ses')
  , client = ses.createClient({ key: 'key', secret: 'secret' });

client.sendEmail({
   to: 'aaron.heckmann+github@gmail.com'
 , from: 'somewhereOverTheR@inbow.com'
 , cc: 'theWickedWitch@nerds.net'
 , bcc: ['canAlsoBe@nArray.com', 'forrealz@.org']
 , subject: 'greetings'
 , message: 'your <b>message</b> goes here'
 , altText: 'plain text'
}, function (err, data, res) {
 // ...
});
```

## Installation

`npm install node-ses`

The module has one primary export:

## createClient()

You'll probably only be using this method. It takes an options object with the following properties:

    `key` - (required) your AWS SES key
    `secret` - (required) your AWS SES secret
    `algorithm` - [optional] the AWS algorithm you are using. defaults to SHA1.
    `amazon` - [optional] the amazon end-point uri. defaults to `https://email.us-west-2.amazonaws.com`

Not all AWS regions support SES. Check [SES region support](http://docs.aws.amazon.com/ses/latest/DeveloperGuide/regions.html) to be sure the region you are in is supported.


```js
var ses = require('node-ses')
  , client = ses.createClient({ key: 'key', secret: 'secret' });
```

## client.sendEmail(options, function (err, data, res))

The client created has the method, `sendEmail`. This method receives an options object with the following properties:

    `from` - email address from which to send (required)
    `subject` - string (required). Must be encoded as UTF-8
    `message` - can be html (required). Must be encoded as UTF-8.
    `altText` - plain text version of message. Must be encoded as UTF-8.
    `to` - email address or array of addresses
    `cc` - email address or array of addresses
    `bcc` - email address or array of addresses
    `replyTo` - email address

At least one of `to`, `cc` or `bcc` is required.

Optional properties (overrides the values set in `createClient`):

    `key` - AWS key
    `secret` - AWS secret
    `algorithm` - AWS algorithm to use
    `amazon` - AWS end point

The `sendEmail` method transports your message to the AWS SES service. If Amazon
returns an HTTP status code that's less than `200` or greater than or equal to
400, we will callback with an `err` object that is a direct translation of the XML error Amazon provides.

Check for errors returned since a 400 status is not uncommon.

The `data` returned in the callback is the HTTP body returned by Amazon as XML.
See the [SES API Response](http://docs.aws.amazon.com/ses/latest/DeveloperGuide/query-interface-responses.html) docs for details.

The `res` returned by the callback represents the HTTP response to calling the SES REST API as the [request](https://www.npmjs.org/package/request) module returns it.

## client.sendRawEmail(options, function (err, data, res))

The client supports the ability to send a raw message via the method, `sendRawEmail`. This method receives an options object with the following properties:

    `from` - email address from which to send (required)
    `rawMessage` - the raw text of the message which includes a header and a body (required)

### Notes
Within the raw text of the message, the following must be observed:

* The `rawMessage` value must contain a header and a body, separated by a blank line.
* All required header fields must be present.
* Each part of a multipart MIME message must be formatted properly.
* MIME content types must be among those supported by Amazon SES. For more information, see the [Amazon SES Developer Guide](http://docs.aws.amazon.com/ses/latest/DeveloperGuide/mime-types.html).
* The `rawMessage` content must be base64-encoded, if MIME requires it.

The `sendRawEmail` method transports your message to the AWS SES service. If Amazon
returns an HTTP status code that's less than `200` or greater than or equal to
400, we will callback with an `err` object that is a direct translation of the XML error Amazon provides.

### Example

```js
var CRLF = '\r\n'
  , ses = require('node-ses')
  , client = ses.createClient({ key: 'key', secret: 'secret' })
  , rawMessage = [
    'From: "Someone" <somewhereOverTheR@inbow.com>',
    'To: "brozeph" <joshua.thomas+github@gmail.com>',
    'Subject: greetings',
    'Content-Type: multipart/mixed;',
    '    boundary="_003_97DCB304C5294779BEBCFC8357FCC4D2"',
    'MIME-Version: 1.0',
    '',
    '--_003_97DCB304C5294779BEBCFC8357FCC4D2',
    'Content-Type: text/plain; charset="us-ascii"',
    'Content-Transfer-Encoding: quoted-printable',
    'Hi brozeph,',
    '',
    'I have attached a code file for you.',
    '',
    'Cheers.',
    '',
    '--_003_97DCB304C5294779BEBCFC8357FCC4D2',
    'Content-Type: text/plain; name="code.txt"',
    'Content-Description: code.txt',
    'Content-Disposition: attachment; filename="code.txt"; size=4;',
    '    creation-date="Mon, 03 Aug 2015 11:39:39 GMT";',
    '    modification-date="Mon, 03 Aug 2015 11:39:39 GMT"',
    'Content-Transfer-Encoding: base64',
    '',
    'ZWNobyBoZWxsbyB3b3JsZAo=',
    '',
    '--_003_97DCB304C5294779BEBCFC8357FCC4D2',
    ''
  ].join(CRLF);

client.sendRawEmail({
 , from: 'somewhereOverTheR@inbow.com'
 , rawMessage: rawMessage
}, function (err, data, res) {
 // ...
});
```

Check for errors returned since a 400 status is not uncommon.

The `data` returned in the callback is the HTTP body returned by Amazon as XML.
See the [SES API Response](http://docs.aws.amazon.com/ses/latest/DeveloperGuide/query-interface-responses.html) docs for details.

The `res` returned by the callback represents the HTTP response to calling the SES REST API as the [request](https://www.npmjs.org/package/request) module returns it.

<a name="debugging"></a>
## Debugging

```bash
# Enable in the shell
DEBUG="node-ses" ./server.js
```

```javascript
// ... or temporarily set in your code before `node-ses` is required.
process.env.DEBUG='node-ses';
```


When debugging, it's useful to inspect the raw HTTP request and response send
to Amazon. These can then checked against Amazon's documentation for the [SendMail](http://docs.aws.amazon.com/ses/latest/APIReference/API_SendEmail.html) API method and the [common errors](http://docs.aws.amazon.com/ses/latest/APIReference/CommonErrors.html) that Amazon might return.

To turn on debugging printed to STDERR, set `DEBUG=node-ses` in the environment before running your script. You can also set `process.env.DEBUG='node-ses';` in your code, before the `node-ses` module is required.

See the [debug module](https://www.npmjs.org/package/debug) docs for more debug output possibilities.

## Running the Tests

`make test`

## See Also

 * [nodemailer](https://www.npmjs.com/package/nodemailer) has more features, including attachment support. There are many "transport" plugins available for it, including one for SES.

## Licence

[MIT](https://github.com/aheckmann/node-ses/blob/master/LICENSE)
