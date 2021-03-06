Mission
=======

Provide a generic SMTP server and client framework that can be extended via
callback modules in the OTP style. The goal is to make it easy to send and
receive email in Erlang without the hassle of POP/IMAP. This is *not* a true
mailserver - although you could build one with it.

The SMTP server/client supports PLAIN, LOGIN, CRAM-MD5 authentication as well
as STARTTLS and SSL (port 465).

Also included is a MIME encoder/decoder, sorta according to RFC204{5,6,7}.

I (Vagabond) have had a simple gen_smtp based SMTP server receiving and parsing
copies of all my email for several months and its been able to handle over 100
thousand emails without leaking any RAM or crashing the erlang virtual machine.

Current Participants
====================

+ Andrew Thompson (andrew@hijacked.us)
+ Jack Danger Canty (code@jackcanty.com)a

Client Example
==============

Here's an example usage of the client:

<pre>
gen_smtp_client:send({"whatever@test.com", ["andrew@hijacked.us"],
 "Subject: testing\r\nFrom: Andrew Thompson <andrew@hijacked.us>\r\nTo: Some Dude <foo@bar.com>\r\n\r\nThis is the email body"},
  [{relay, "smtp.gmail.com"}, {username, "me@gmail.com"}, {password, "mypassword"}]).
</pre>

The From and To addresses will be wrapped in &lt;&gt;s if they aren't already,
TLS will be auto-negotiated if available (unless you pass `{tls, never}`) and
authentication will by attempted by default since a username/password were
specified (`{auth, never}` overrides this).

If you want to mandate tls or auth, you can pass `{tls, always}` or `{auth,
always}` as one of the options. You can specify an alternate port with `{port,
2525}` (default is 25) or you can indicate that the server is listening for SSL
connections using `{ssl, true}` (port defaults to 465 with this option).

Server Example
==============

gen_smtp ships with a simple callback server example, smtp_server_example. To start the SMTP server with this as the callback module, issue the following command:

<pre>
gen_smtp_server:start(smtp_server_example).
gen_smtp_server starting at nonode@nohost
listening on {0,0,0,0}:2525 via tcp
{ok,<0.33.0>}
</pre>

By default it listens on 0.0.0.0 port 2525. You can telnet to it and test it:

<pre>
^andrew@orz-dashes:: telnet localhost 2525                                                      [~]
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
220 localhost ESMTP smtp_server_example
EHLO example.com
250-orz-dashes
250-SIZE 10485670
250-8BITMIME
250-PIPELINING
250 WTF
MAIL FROM: andrew@hijacked.us
250 sender Ok
RCPT TO: andrew@hijacked.us
250 recipient Ok
DATA
354 enter mail, end with line containing only '.'
Good evening gentlemen, all your base are belong to us.
.
250 queued as #Ref<0.0.0.47>
QUIT
221 Bye
Connection closed by foreign host.
</pre>

You can configure the server in general, each SMTP session, and the callback module, for example:

<pre>
gen_smtp_server:start(smtp_server_example, [[{sessionoptions, [{allow_bare_newlines, fix}, {callbackoptions, [{parse, true}]}]}]]).
</pre>

This configures the session to fix bare newlines (other options are 'strip', 'true' and 'false', false rejects emails with bare newlines, true passes them through unmodified and strip removes them) and tells the callback module to run the MIME decoder on the email once its been received. The example callback module also supports the following options; relay - whether to relay email on, auth - whether to do SMTP authentication. These are included mainly as an example and are not intended for serious usage.

You can also start multiple SMTP listeners at once by passing more than 1 configuration:

<pre>
gen_smtp_server:start(smtp_server_example, [[], [{protocol, ssl}, {port, 1465}]]).
</pre>

This starts 2 listeners, one with the default config, and one with the default config except that its running in SSL mode on port 1465.

You can connect and test this using the gen_smtp_client via something like:

<pre>
gen_smtp_client:send({"whatever@test.com", ["andrew@hijacked.us"], "Subject: testing\r\nFrom: Andrew Thompson \r\nTo: Some Dude \r\n\r\nThis is the email body"}, [{relay, "localhost"}, {port, 1465}, {ssl, true}]).
</pre>
