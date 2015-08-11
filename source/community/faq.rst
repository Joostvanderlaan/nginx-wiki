Frequently Asked Questions
==========================

Please review this entire FAQ before querying IRC or sending questions to the mailing list.

HI I have a question about the nginx webserver whenever I start nginx by ``sudo ./nginx start`` and then type the ``ps -ef | grep nginx`` command I get the following output. (when I am in my ``/etc/init.d`` directory)

.. code-block:: bash

  1000 10760 10617 0 13:40 pts/0 00:00:00 grep --color=auto nginx



General Questions
-----------------

How do you pronounce "Nginx"?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
There seems to be some confusion surrounding the pronunciation of Nginx.

`Pronunciation Key <http://dictionary.reference.com/help/luna/Spell_pron_key.html>`_

**Correct**

* **en**-juhn-**eks***
* Engine-X

**Incorrect**

* **en**-jingks

.. _faq.is_it_safe:

Is it safe to use the development branch in production?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
In general, all releases (development or otherwise) are quite stable.
This site runs the latest development version at all times.
Many nginx users tend to represent an "early adopter" crowd, so a large segment is using the bleeding edge version at any given point.
The major and most important point about development version is that it can occasionally break APIs for 3rd party modules (APIs are sometimes changed in the development branch).
Other than that, it gets all non-emergency bugfixes first.

That said, if stability is crucial it is best to briefly hold off on deployment after a development release; critical bugs tend to show up within the first couple days (which often results in another release immediately afterwards).
If no new release shows up in two or three days, then it's likely no one has found any critical bugs.
In the event that you discover a bug, capture a debug log and submit a descriptive bug report!


How do I generate an .htpasswd file without having Apache tools installed?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* In Linux (and other Posix):  given users John, Mary, Jane and Jim and passwords V3Ry, SEcRe7, V3RySEcRe7 and SEcRe7PwD, in order to generate a password file named .htpasswd, you would issue:

  .. code-block:: bash

    printf "John:$(openssl passwd -crypt V3Ry)\n" >> .htpasswd # this example uses crypt encryption

  .. code-block:: bash

    printf "Mary:$(openssl passwd -apr1 SEcRe7)\n" >> .htpasswd # this example uses apr1 (Apache MD5) encryption

  .. code-block:: bash

    printf "Jane:$(openssl passwd -1 V3RySEcRe7)\n" >> .htpasswd # this example uses MD5 encryption

  .. code-block:: bash

    (PASSWORD="SEcRe7PwD";SALT="$(openssl rand -base64 3)";SHA1=$(printf "$PASSWORD$SALT" | openssl dgst -binary -sha1 | sed 's#$#'"$SALT"'#' | base64);printf "Jim:{SSHA}$SHA1\n" >> .htpasswd) # this example uses SSHA encryption

* Or, you may use the following `crypt.pl <https://gist.github.com/3349662>`_ Perl script. Simply save it as e.g. ``crypt.pl`` and ``chmod 700 crypt.pl`` in order to be able to execute it.

  .. code-block:: perl

    #!/usr/bin/perl
    use strict;

    chomp(my $filename=$ARGV[0]);
    chomp(my $username=$ARGV[1]);
    chomp(my $password=$ARGV[2]);

    if (!$filename || !$username || !$password) {
      print "USAGE: ./crypt.pl filename username password\n\n";
    } else {
      open my $fh, ">>", $filename or die $!;
      print $fh $username . ":" . crypt($password, $username) . "\n";
      close $fh or die $!;
    }

* Or, you may use the `htpasswd.py <http://trac.edgewall.org/browser/trunk/contrib/htpasswd.py>`_ python script.



Why isn't my $foo (e.g. rewrite, proxy, location, unix:/$PATH, etc.) configuration working for me?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Start by investigating possible problem causes. Review :doc:`../start/topics/tutorials/debugging` and carefully look **LINE BY LINE** through the error log.

If you can't determine the problem cause through testing, experimentation, searches on the 'net, etc., then **gather all relevant details** and clearly explain your problem on IRC or in a note to the mailing list.
(If you are new to interacting with FOSS support communities, please read: `How To Ask Questions The Smart Way <http://catb.org/~esr/faqs/smart-questions.html>`_.)


Are there other, similar web servers?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* `Cherokee <http://www.0x50.org/>`_
* `Lighttpd (Lighty) <http://www.lighttpd.net/>`_
* `thttpd <http://www.acme.com/software/thttpd/>`_

What most people mean by "similar" in this context is: "lightweight" or "not Apache".
You can find many comparisons using Google, but most web servers fall into two categories: process-based (forking or threaded) and asynchronous.
Nginx and Lighttpd are probably the two best-known asynchronous servers and Apache is undoubtedly the best known process-based server.
Cherokee is a lesser-known process-based server (but with very high performance).

The main advantage of the asynchronous approach is *scalability*.
In a process-based server, each simultaneous connection requires a thread which incurs significant overhead.
An asynchronous server, on the other hand, is event-driven and handles requests in a single (or at least, very few) threads.

While a process-based server can often perform on par with an asynchronous server under light loads, under heavier loads they usually consume far too much RAM, which significantly degrades performance.
Also, they degrade much faster on less powerful hardware or in a resource-restricted environment such as a VPS.

Pulling numbers from thin air for illustrative purposes: serving 10,000 simultaneous connections would probably only cause Nginx to use a few megabytes of RAM, while Apache would likely consume hundreds of megabytes (if it could do it at all).


Is support for chroot planned?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Unknown at this time.  Unless/until that changes, you can achieve a similar - or better - effect by using OS-level features (e.g. ``BSD Jails``, ``OpenVZ`` with ``proxyarp`` on Linux, etc.).


What about support for something like mod_suexec?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
mod_suexec is a solution to a problem that Nginx does not have.
When running servers such as Apache, each instance consumes a significant amount of RAM, so it becomes important to only have a monolithic instance that handles all one's needs.
With Nginx, the memory and CPU utilization is so low that running dozens of instances of it is not an issue.

A comparable Nginx setup to Apache + ``mod_suexec`` is to run a separate instance of Nginx as the CGI script user (i.e. *the user that would have been specified as suexec user under Apache*), and then proxy to that from the main Nginx instance.

Alternatively, PHP could simply be executed through FastCGI, which itself would be running under a CGI script user account.

.. note::  ``mod_php`` (the module suexec is normally utilized to defend against) does not exist with Nginx.



HTTP
----

What's this @ thing mean?
^^^^^^^^^^^^^^^^^^^^^^^^^
@location is a named location.  Named locations preserve $uri as it was before entering such location.
They were introduced in 0.6.6 and can be reached only via `error_page <|HttpCoreModule|#error_page>`_, `post_action <|HttpCoreModule|#post_action>`_ (since 0.6.26) and `try_files <|HttpCoreModule|#try_files>`_ (since 0.7.27, backported to 0.6.36).



HTTP Proxying
-------------

For which general use cases is Nginx more appropriate than Squid? (And vice versa...)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Nginx is generally deployed as a reverse proxy, not as a caching proxy (like Squid).
The key advantage with Nginx is its nominal RAM and CPU usage under heavy load.
Squid is best applied to cache dynamic content for applications that cannot do it themselves.

The |HttpProxyModule| offers configurations for caching upstream servers.


Can I disable the buffering for upload progress? / How can I display upload progress on the client side?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
These are both very frequently asked questions.

Currently the only solution is the third-party module :doc:`../modules/upload_progress`.

(This functionality is planned for a future release of Nginx.)



Mail Proxying
-------------

Could someone explain how to configure and test the IMAP module (with a complete .conf example)?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Start with the configuration from the :doc:`../start/topics/examples/imapproxyexample`.

For detailed information about different configuration parameters, see the |MailCoreModule| page.

Related resources:

* :doc:`../start/topics/examples/imapauthenticatewithapachephpscript`

.. todo:
   ..
      :doc:`imapauthenticatewithembeddedperlscript`


How can Nginx be deployed as an SMTP proxy, with a Postfix backend?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
First, declare the mail part as follows:

.. code-block:: nginx

  mail {
      server_name mail.somedomain.com;

      auth_http localhost:8008/auth-smtppass.php;

      server {
          listen 212.104.99.24:25;
          protocol smtp;
          timeout 5s;
          proxy on;
          xclient off;
          smtp_auth none;
      }
  }


The example is for unauthenticated e-mail as you can see, but if you need authentication just check out the |MailCoreModule| information on how to achieve it.
Postfix by default doesn't support XCLIENT, so it got turned off in the example as well.

Next, you need to configure the authentication backend. If you just need to have some sort of pass-through mode towards a single address, you can do so with the following code:

.. code-block:: nginx

  http {
      log_format main
          '$remote_addr - $remote_user [$time_local] '
          '"$request" $status $bytes_sent '
          '"$http_referer" "$http_user_agent" '
          '"$gzip_ratio"';

      server {
          listen 127.0.0.1:8008;
          server_name localhost;
          access_log /var/log/nginx/localhost.access_log main;
          error_log /var/log/nginx/localhost.error_log info;

          root /var/www/localhost/htdocs;

          location ~ .php$ {
              add_header Auth-Server 127.0.0.1;
              add_header Auth-Port 25;
              return 200;
          }
      }
  }


Basically, it accepts connections and for a request towards a .php file, it will return with code 200 and with the address of the (in this case) Postfix backend (on 127.0.0.1:25 here).



Load Balancing
--------------

What algorithm does Nginx use to load balance?  Can it balance based on connection load?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Currently, Nginx has round-robin, least connections and ip-hash algorithms (all come with weights).

There are also a number of third-party modules for load balancing.

.. note:: Many users have requested that Nginx implement a feature in the load balancer to limit the number of requests per backend (usually to one).  While support for this is planned, it's worth mentioning that demand for this feature is rooted in misbehaviour on the part of the application being proxied ''to'' (Ruby on Rails seems to be one example). This is not an Nginx issue. In an ideal world, this particular problem fix request would be directed toward the backend application and its ability handle simultaneous requests.



Miscellaneous
-------------

Why do I see "Welcome to nginx!" when I'm trying to access Facebook, Yahoo!, Google or some other well-known website? Is nginx a virus?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Nginx is not a virus. Please check a detailed description here::

  `Welcome to nginx! <http://nginx.org/en/docs/welcome_nginx_facebook.html>`_