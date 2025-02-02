MTA - Mail Transfer Server
==========================

.. image:: Postfix-logo.*
    :alt: Postfix Logo
    :align: right

`Postfix <http://www.postfix.org/>`_ is a free and open-source mail transfer
agent (MTA) that routes and delivers electronic mail on a Linux system. It is
estimated that around 26% of public mail servers on the internet run Postfix.

.. contents:: \


Prerequisites
-------------

    * :doc:`/server/mariadb/index`
    * :doc:`/server/mail/dovecot`
    * :doc:`/server/clamav`
    * :doc:`/server/mail/rspamd`
    * :doc:`/server/dns/unbound`
    * :doc:`/server/dehydrated`


Install Software
----------------

::

    $ sudo apt-get install postfix postfix-mysql


On Ubuntu server 20.04 (focal) the installed version is Postfix version 3.4.13,
released on June 14, 2020.


Main Configuration
---------------------

Postfix uses a number of different configuration files, with the
:file:`/etc/postfix/main.cf` being the most important.

The documentation website has `a page dedicated to main.cf
<http://www.postfix.org/postconf.5.html>`_ which includes every possible
configuration paramter.

The whole file, as presented below, is also provided for download at
:download:`/server/config-files/etc/postfix/main.cf` here.


General Settings
^^^^^^^^^^^^^^^^

.. literalinclude:: /server/config-files/etc/postfix/main.cf
    :language: ini
    :linenos:
    :lineno-match:
    :start-at: # General Mail Server Setttings
    :end-before: # Local Mailboxes and Aliases Maps


Local Mailboxes and Aliases Maps
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. literalinclude:: /server/config-files/etc/postfix/main.cf
    :language: ini
    :linenos:
    :lineno-match:
    :start-at: # Local Mailboxes and Aliases Maps
    :end-before: # Virtual Domains, Mailboxes and Aliases Maps


Virtual Domains, Mailboxes and Aliases Maps
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. literalinclude:: /server/config-files/etc/postfix/main.cf
    :language: ini
    :linenos:
    :lineno-match:
    :start-at: # Virtual Domains, Mailboxes and Aliases Maps
    :end-before: # TCP/IP Protocols Settings


TCP/IP Protocols Settings
^^^^^^^^^^^^^^^^^^^^^^^^^

.. literalinclude:: /server/config-files/etc/postfix/main.cf
    :language: ini
    :linenos:
    :lineno-match:
    :start-at: # TCP/IP Protocols Settings
    :end-before: # General TLS Settings


.. index:: Cipher Suite; Set in Postfix


General TLS Settings
^^^^^^^^^^^^^^^^^^^^

.. literalinclude:: /server/config-files/etc/postfix/main.cf
    :language: ini
    :linenos:
    :lineno-match:
    :start-at: # General TLS Settings
    :end-before: # SMTP Client Settings


.. index::
    single: Internet Protocols; SMTP

SMTP Client Settings
^^^^^^^^^^^^^^^^^^^^

.. literalinclude:: /server/config-files/etc/postfix/main.cf
    :language: ini
    :linenos:
    :lineno-match:
    :start-at: # SMTP Client Settings
    :end-before: # SMTP Server Settings


SMTP Server Settings
^^^^^^^^^^^^^^^^^^^^^^

.. literalinclude:: /server/config-files/etc/postfix/main.cf
    :language: ini
    :linenos:
    :lineno-match:
    :start-at: # SMTP Client Settings
    :end-before: # SMTP Server SASL Authentication Settings


SMTP Server SASL Authentication
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. literalinclude:: /server/config-files/etc/postfix/main.cf
    :language: ini
    :linenos:
    :lineno-match:
    :start-at: # SMTP Server SASL Authentication Settings
    :end-before: # SMTP Server Restrictions


SMTP Server Restrictions
^^^^^^^^^^^^^^^^^^^^^^^^

.. literalinclude:: /server/config-files/etc/postfix/main.cf
    :language: ini
    :linenos:
    :lineno-match:
    :start-at: # SMTP Server Restrictions
    :end-before: # Mail User Agent (MUA) Restrictions


Mail User Agent (MUA) Restrictions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. literalinclude:: /server/config-files/etc/postfix/main.cf
    :language: ini
    :linenos:
    :lineno-match:
    :start-at: # Mail User Agent (MUA) Restrictions
    :end-before: # Mail Filters (Milters)


Mail Filters (Milters)
^^^^^^^^^^^^^^^^^^^^^^

.. literalinclude:: /server/config-files/etc/postfix/main.cf
    :language: ini
    :linenos:
    :lineno-match:
    :start-at: # Mail Filters (Milters)


Map Files and Databases
-----------------------

Postfix uses several maps stored in files to translate things like domains and
mail- addresses at runtime. As these maps can be quite large and change often,
they are not saved in configuration files, but in map-files.

The map files are referenced in the postfix server-configuration. Examples
already used above are the `canonical_maps` or `virtual_mailbox_domains` above.

Such configuration values usually contain a prefix like `hash:` or `mysql:`,
which tell the server in what type of data store the mapping information is
accessible.

If the prefix starts with `hash:` the map is initially is stored as plain text
in the file referenced, but cached later in a hash-database for faster access. A
hashed map or database can change anythme while the server is running, unlike
configuration values, which are only read on server startup, and changes need a
restart of the mail server.

It migth get complicated, the more such files are around as they use different
formats, handling and hashing.

It therefore helps to create a Makefile, so refreshing anything needs just a
single command-line. Create the :download:`/etc/postfix/Makefile
</server/config-files/etc/postfix/Makefile>` as follows:

.. literalinclude:: /server/config-files/etc/postfix/Makefile
    :language: make
    :linenos:
    :tab-width: 4


After every change to any of the map files discussed below, their databases need
to be refreshed as follows::

    $ cd /etc/postfix && sudo make


Alias Map
^^^^^^^^^

The file
:download:`/etc/postfix/aliases.map </server/config-files/etc/postfix/aliases.map>`
contains information on where mails for certain system accounts should be
delivered to.

This is needed for notification and warning mails created by system programs
(like cronjobs) to reach human beings (like the person responsible for system
administration).

.. literalinclude:: /server/config-files/etc/postfix/aliases.map
    :language: bash
    :linenos:


The contents of the file are cached in the database
:file:`/etc/postfix/aliases.db`. Because of that the database must be refreshed
after each and every change made in :file:`/etc/postfix/aliases.map`::

    $ cd /etc/postfix && sudo make


Canonical Sender Map
^^^^^^^^^^^^^^^^^^^^

The file :download:`/etc//postfix/sender_canonical.map
</server/config-files/etc/postfix/sender_canonical.map>` contains information on
which sender-addresses of outgoing mails need to be **rewritten**, before mail
is sent out.

This is needed mostly for system generated mails, as they contain local system
account-names as senders, which will not be accepted as valid internet-mail
addresses. However, since this is a rewrite, after the address is rewritten, you
may not know where the message came from.

.. literalinclude:: /server/config-files/etc/postfix/sender_canonical.map
    :language: bash
    :linenos:


The contents of the file are cached in the database
:file:`/etc/postfix/sender-canonical.db`. Because of that the database must be
refreshed after each and every change made in
:file:`/etc/postfix/sender_canonical.map`::

    $ cd /etc/postfix && sudo make

Canonical Recipient Map
^^^^^^^^^^^^^^^^^^^^^^^

The file :download:`/etc//postfix/recipient_canonical.map
</server/config-files/etc/postfix/recipient_canonical.map>` contains information
on which recipient addresses of outgoing mails need to be **rewritten**, before
mail is sent out.

This is needed mostly for system generated mails, as they contain local system
account-names as senders, which will not be accepted as valid internet-mail
addresses. However, since this is a rewrite, after the address is rewritten, you
may not know where the message originally was intended to. It may be preferrable
to use the *aliases map* decribed above, which forwards the message to a chosen
address, leaving the original recipient headers intact.

.. literalinclude:: /server/config-files/etc/postfix/recipient_canonical.map
    :language: bash
    :linenos:


The contents of the file are cached in the database
:file:`/etc/postfix/recipient-canonical.db`. Because of that the database must
be refreshed after each and every change made in
:file:`/etc/postfix/recipient_canonical.map`::

    $ cd /etc/postfix && sudo make


Canonical Map
^^^^^^^^^^^^^

The file
:download:`/etc/postfix/canonical.map </server/config-files/etc/postfix/canonical.map>`
defines a map on which addresses need always be changed for sender and recipient
in message headers and envelopes:

.. literalinclude:: /server/config-files/etc/postfix/canonical.map
    :language: bash
    :linenos:

The contents of the file are cached in the database
:file:`/etc/postfix/canonical.db`.
To update the database after changes run the following::

    $ cd /etc/postfix && sudo make


SMTP Generic Map
^^^^^^^^^^^^^^^^

The file
:download:`/etc/postfix/generic.map </server/config-files/etc/postfix/generic.map>`
contains a lookup table that perform address rewriting in the Postfix |SMTP|
client, typically to transform a locally valid address into a globally valid
address when sending mail across the Internet:

.. literalinclude:: /server/config-files/etc/postfix/generic.map
    :language: bash
    :linenos:

This is needed when the local machine does not have its own Internet domain
name, but uses something like **localdomain.local** instead.

The contents of the file are cached in the database
:file:`/etc/postfix/canonical.db`. To update the database after changes run the
following::

    $ cd /etc/postfix && sudo make


Virtual Domains Map
^^^^^^^^^^^^^^^^^^^
We referenced this the file
:file:`/etc/postfix/sql/virtual-domains.cf` in the main configuration
file :file:`main.cf` above as *virtual_mailbox_domains*. It contains the MariaDB
database server connection information and the SQL Query for Postfix to lookup
the virtual domains hosted here:

.. code-block:: ini
    :linenos:

    #
    # MySQL connection and query to lookup which mail domains are hosted here
    #
    # Note: Postfix can't access UNIX sockets outside of '/var/spool/postfix'
    # so make sure to use a numerical IP addresses only for 'hosts'!
    hosts = 127.0.0.1
    user = postfix
    password = ****************
    dbname = vimbadmin
    query = SELECT 1 FROM domain WHERE domain='%s' AND active = '1'


Use the user, password and database defined in :doc:`virtual`.


Virtual Mailboxes Map
^^^^^^^^^^^^^^^^^^^^^

Same as with the virtual domain map above, the following MySQL connection
defined in :file:`/etc/postfix/sql/virtual-mailboxes.cf` is used to lookup
which mailboxes are hosted here:

.. code-block:: ini
    :linenos:

    #
    # MySQL connection and query to lookup which mailboxes are hosted here
    #
    # Note: Postfix can't access UNIX sockets outside of '/var/spool/postfix'
    # so make sure to use a numerical IP addresses only for 'hosts'!
    hosts = 127.0.0.1
    user = postfix
    password = ****************
    dbname = vimbadmin
    query = SELECT 1 FROM mailbox WHERE username = '%s' AND active = '1'


Note that the only difference to the above one is the SQL query. looking for
mailboxes instead of domains.


Virtual Alias Map
^^^^^^^^^^^^^^^^^

Finally we also create :file:`/etc/postfix/sql/virtual-aliases.cf` to give
Postfix a way to lookup mailbox aliases at the MySQL database server:

.. code-block:: ini
    :linenos:

    #
    # MySQL connection and query to lookup mail-address aliase and their mailbox
    #
    # Note: Postfix can't access UNIX sockets outside of '/var/spool/postfix'
    # so make sure to use a numerical IP addresses only for 'hosts'!
    hosts = 127.0.0.1
    user = postfix
    password = ****************
    dbname = vimbadmin
    query = SELECT goto FROM alias WHERE address = '%s' AND active = '1'


TLS Policy Map
^^^^^^^^^^^^^^

TLS encryption is not mandatory between SMTP servers, since it based on old
standards, which cannot be changed, without mail delivery to fail on a
catastrphic global scale.

Most SMTP servers today support TLS encryption trough the STARTTLS command which
allows to upgrade an unencrypted connection to an encrypted one, even after the
connection is alreeady built.

But the landscape is, many servers use old outdated and weak encryption
stadards, many use self-signed certificates. As mail sevrice provider, you have
no choice, but to suuuport these as well, or else you and your users won't be
able to exchange mail with any of them.

A TLS policy map helps you to achieve the best possible encryption, at least
with servers you know, while still having standards-compliant but bad encryption
with all the other ones.

The :file:`/etc/postfix/sql/tls-policy.cf` file :


.. code-block:: ini
    :linenos:

    hosts = 127.0.0.1
    user = postfix
    password = ****************
    dbname = vimbadmin
    query = SELECT policy, params FROM tlspolicies WHERE domain = '%s'


Master Process Configuration
----------------------------

Postfix consists of many daemons and utilities, some running all the time and
others started when needed. The :file:`/etc/postfix/master.cf` confiugration
file is the centralized control center from where the Postfix master process
gets his guidelines on what, when and how all these programs have to be started.

Where no command-line options are specified, the program uses relevant
configuration values from the :file:`main.cf` configuration file. Alternatively
options can be overridden by command-line parameters like `-o`.

The official documentation website `provides a manual page
<http://www.postfix.org/master.5.html>`_ online.

The whole file, as presented below, is also provided for download at
:download:`/server/config-files/etc/postfix/master.cf` here.

.. index::
    single: Internet Protocols; SMTP


Standard SMTP Dameon
^^^^^^^^^^^^^^^^^^^^

The first daemon specified here is the |SMTP| daemon **smtpd** which runs on the
dedicated |SMTP| TCP port 25, it is enabled by default:

.. literalinclude:: /server/config-files/etc/postfix/master.cf
    :language: bash
    :lines: 8-12


Postscreen SMTP Firewall
^^^^^^^^^^^^^^^^^^^^^^^^

The **postscreen** damon on the second line is a daemon which runs a mininmal
|SMTP| subset to perform some compliance tests on incoming |SMTP| client
connections. If the client passes all tests he gets whitelistet and the
connection is passed along to the real |SMTP| server running behind it.
That is what the second line above is for. An **smtpd** daemon that
takes over connections from **postscreen** only. The **tlsproxy** handles TLS
encryption for postscreen and **dnsblog** does the DNS blacklist lookups.

Since we use the Rspamd milter to separate legitimate from undesired SMTP
connections, the postscreen line can be commented out.

.. literalinclude:: /server/config-files/etc/postfix/master.cf
    :language: bash
    :lines: 8-11,13-14

Internal Postfix SMTP Daemon
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If the conneting SMTP client has passed all the checks performed by
**postscreen**, the client IP address is temporarely whitelisted.

If possible the TCP connection is then handed over to the SMTPD daemon
internally (that is what the "**pass**" keyword means).

Sometimes this is not possible, depeneding on the checks. If the client had to
start already some message delivery in order to complete a check, the connection
can not be handed over to a new SMTP server, as this would confuse the SMTP
client who is already in the middle of a message delivery. In this case,
**postscreen** will tell the connecting SMTP client to abort message delivery
for now and try again later. If the same client connects again later, it will be
already whitelisted and then the connection is "passed" directly to the postfix
SMTP daemon.

Since we don't use postscreen, this daemon can be disabled by commenting out the
line.

.. literalinclude:: /server/config-files/etc/postfix/master.cf
    :language: bash
    :lines: 8-11,15


DNS Blacklist & Whitelist Lookup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The **dnsblog** is a helper service for the postscreen SMTP firewall to lookup,
if connecting hosts are found in any of configured the DNS blacklists or
whitelists.

Since we use the Rspamd milter to separate legitimate from undesired SMTP
connections, the dnsblog line can be commented out.

.. literalinclude:: /server/config-files/etc/postfix/master.cf
    :language: bash
    :lines: 8-11,16


TLS Proxy
^^^^^^^^^

It is used by **postscreen** server to handle TLS connections with inbound SMTP
client connections and by the local **smtp** client to support TLS connection
reuse.

.. literalinclude:: /server/config-files/etc/postfix/master.cf
    :language: bash
    :lines: 8-11,17


Submission Daemon
^^^^^^^^^^^^^^^^^

The submission daemon is a SMTP daemon listening on TCP port 587 for mail user
agents (MUAs aka mail clients like Thunderbird or Outlook) of authorized users
to connecting and submit their outgoing mail.

.. literalinclude:: /server/config-files/etc/postfix/master.cf
    :language: bash
    :lines: 8-11,18-29

The connecting MUA is required to initialize TLS encryption of the connection
with the STARTTLS command before anything else. This controlled by the
"smtpd_tls_security_level=encrypt" option.

The following options enable user authentication over encrypted connections and
various settings to allow mail from authenticated and authorized users to be
sent anywhere.

The last options, marks the submitted mail messages as our own, to be handles
accordingly by our spam filters.


SMTPS Daemon
^^^^^^^^^^^^

The SMTPS daemon is very simliar to the submission daemon. The difference being,
it runs on TCP port 465 and connections are already encrypted, therefore no
STARTTLS is required.

.. literalinclude:: /server/config-files/etc/postfix/master.cf
    :language: bash
    :lines: 8-11,30-40


Other Postfix Daemons and Programs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. literalinclude:: /server/config-files/etc/postfix/master.cf
    :language: bash
    :lines: 8-11,41-70

**qmqpd** - QMQP (Quick Mail Queueing Protocol) daemon. A mail protocol
orginally devloped by Daniel J. Bernstein as part of QMAIL. Disabled.

**pickup** - Postfix local mail pickup. The pickup(8) daemon waits for hints that
new mail has been dropped into the maildrop directory, and feeds it into the
cleanup(8) daemon.

**cleanup** - canonicalize and enqueue Postfix message. The cleanup(8) daemon
processes inbound mail, inserts it into the incoming mail queue, and informs the
queue manager of its arrival.

**qmgr** - Postfix queue manager. The qmgr(8) daemon awaits the arrival of
incoming mail and arranges for its delivery via Postfix delivery processes. The
actual mail routing strategy is delegated to the trivial-rewrite(8) daemon.

**oqmgr** - old Postfix queue manager. Disabled.

**tlsmgr** - Postfix TLS session cache and PRNG manager. The tlsmgr(8) manages
the Postfix TLS session caches. It stores and retrieves cache entries on request
by smtpd(8) and smtp(8) processes, and periodically removes entries that have
expired.

**trivial-rewrite** - Postfix address rewriting and resolving daemon. The
trivial-rewrite(8) daemon processes three types of client service requests:

    #. **rewrite** *context address* - Rewrite an address to standard form,
       according to the address rewriting context (local or remote).
    #. **resolve** *sender address* - Resolve the address to a (transport,
       nexthop, recipient, flags) quadruple.
    #. **verify** *sender address* - Resolve the address for address
       verification purposes.

**bounce** - Postfix delivery status reports. Maintains per-message log files
with delivery status information.

**verify** - Postfix address verification server. Maintains a record of what
recipient addresses are known to be deliverable or undeliverable.

**flush** - Postfix fast flush server. Maintains a record of deferred mail by
destination. This information is used to improve the performance of the SMTP
ETRN request, and of its command-line equivalent, "sendmail -qR" or "postqueue
-f".

**proxymap** - Postfix lookup table proxy server. Provides read-only or
read-write table lookup service to Postfix processes. These services are
implemented with dis- tinct service names: proxymap and proxywrite,
respectively. The purpose of these services is:

    * Overcome chroot restrictions.
    * Consolidate the number of open lookup tables, by open connections and
      files.
    * Provide write access to tables, who don't support more than one
      simultanious write access.

**showq** - list the Postfix mail queue. Reports the Postfix mail queue status.
The output is meant to be formatted by the postqueue(1) command, as it emulates
the Sendmail *mailq* command.

**error** - Postfix error/retry mail delivery agent. Processes delivery requests
from the queue manager. Each request specifies a queue file, a sender address,
the reason for non-delivery (specified as the next-hop destination), and
recipient information.

**discard** - Postfix discard mail delivery agent. Processes delivery requests
from the queue manager. Each request specifies a queue file, a sender address, a
next-hop destination that is treated as the reason for dis- carding the mail,
and recipient information.

**local** - Postfix local mail delivery. Processes delivery requests from the
Postfix queue manager to deliver mail to local recipients. Each delivery request
specifies a queue file, a sender address, a domain or host to deliver to, and
one or more recipients.

**virtual** - Postfix virtual domain mail delivery agent. Designed for virtual
mail hosting services. Originally based on the Postfix local(8) delivery agent,
this agent looks up recipients with map lookups of their full recipient address,
instead of using hard-coded unix password file lookups of the address local part
only.

**anvil** - Postfix session count and request rate control. Maintains statistics
about client connection counts or client request rates.

**scache** - Postfix shared connection cache server. Maintains a shared
multi-connection cache. This information can be used by, for example, Postfix
SMTP clients or other Postfix delivery agents.

**submission-header-cleanup**

**postlog** - Postfix-compatible logging utility. Implements a
Postfix-compatible logging interface for use in, for example, shell scripts.

By default, postlog(1) logs the text given on the command line as one record. If
no text is specified on the command line, postlog(1) reads from standard input
and logs each input line as one record.

By default, logging is sent to syslogd(8) or postlogd(8); when the standard
error stream is connected to a terminal, logging is sent there as well.


References
----------

 * `Postfix Documentation <http://www.postfix.org/documentation.html>`_
