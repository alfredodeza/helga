.. currentmodule:: helga.settings

.. _config:

Configuring Helga
=================
As mentioned in :ref:`getting_started`, when you install helga, a ``helga`` console script is
created that will run the bot process. This is the simplest way to run helga, however, it will
assume various default settings like assuming that both an IRC and MongoDB server to which you
wish to connect run on your local machine. This may not be ideal for running helga in a production
environment. For this reason you may wish to create your own configuration for helga.


.. _config.custom:

Custom Settings
---------------
Helga settings files are essentially executable python files. If you have ever worked with django
settings files, helga settings will feel very similar. Helga does assume some configuration defaults,
but you can (and should) use a custom settings file. The behavior of any custom settings file you use
is to overwrite any default configuration helga uses. For this reason, you do not need to apply
all of the configuration settings (listed below) known. For example, a simple settings file to connect
to an IRC server at ``example.com`` on port ``6667`` would be::

    SERVER = {
        'HOST': 'example.com',
        'PORT': 6667,
    }

There are two ways in which you can use a custom settings file. First, you could export a ``HELGA_SETTINGS``
environment variable. Alternatively, you can indicate this via a ``--settings`` argument to the ``helga``
console script. For example:

.. code-block:: bash

    $ export HELGA_SETTINGS=foo.settings
    $ helga

Or:

.. code-block:: bash

    $ helga --settings=/etc/helga/settings.py

In either case, this value should be an absolute filesystem path to a python file like ``/path/to/foo.py``,
or a python module string available on ``$PYTHONPATH`` like ``path.to.foo``.



.. _config.default:

Default Configuration
---------------------
Running the ``helga`` console script with no arguments will run helga using a default configuration, which
assumes that you are wishing to connect to an IRC server. For a full list of the included default settings,
see :mod:`helga.settings`.



.. _config.xmpp:

XMPP Configuration
------------------
Helga was originally written as an IRC bot, but now includes XMPP support as well. Since its background
as an IRC bot, much of the language in the documentation and API are geared towards that. For instance,
multi user chat rooms are referred to as 'channels' and users are referred to by a 'nick'. The default
helga configuration will assume that you want to connect to an IRC server. To enable XMPP connections,
you must specify a ``TYPE`` value of ``xmpp`` in your ``SERVER`` settings::

    SERVER = {
        'HOST': 'example.com',
        'PORT': 5222,
        'TYPE': 'xmpp',
        'USERNAME': 'helga',
        'PASSWORD': 'hunter2',
    }

Note above that you also **must** specify a value for ``USERNAME`` and ``PASSWORD``, which will result
in a Jabber ID (JID) of something like ``helga@example.com``. The also assumes that the multi user chat
(MUC) domain for your xmpp server is ``conference.example.com``. This might not always be desirable.
For this reason, you can also specify specific JID and MUC values using the keys ``JID`` and ``MUC_HOST``
respectively. In this instance, the specific JID is used to authenticate and username is not required::

    SERVER = {
        'HOST': 'example.com',
        'PORT': 5222,
        'TYPE': 'xmpp',
        'PASSWORD': 'hunter2',
        'JID': 'someone@example.com',
        'MUC_HOST': 'chat.example.com',
    }

Also, just like IRC support, helga can automatically join chat rooms configured in the setting
:data:`~helga.settings.CHANNELS`. You can configure this a couple of different ways, the easiest
being a shorthand version of the room name, prefixed with a '#'. For example, given a room with
a JID of ``bots@conf.example.com``, the setting might look like::

    CHANNELS = [
        '#bots',
    ]

Alternatively, you *can* specify the full JID::

    CHANNELS = [
        'bots@conf.example.com',
    ]

Just like IRC, you can specify a room password using a two-tuple::

    CHANNELS = [
        ('#bots', 'room_password'),
    ]


.. _config.xmpp.hipchat:

HipChat Support
^^^^^^^^^^^^^^^
`HipChat`_ allows for clients to connect to its service using XMPP. If you are intending to use helga
as a HipChat bot, you will first need to take note of the settings needed to connect (see
`HipChat XMPP Settings`_). This also applies to anyone using the self-hosted HipChat server. A server
configuration for connecting to HipChat might look like::

    SERVER = {
        'HOST': 'chat.hipchat.com',
        'PORT': 5222,
        'JID': '00000_00000@chat.hipchat.com',
        'PASSWORD': 'hunter2',
        'MUC_HOST': 'conf.hipchat.com',
        'TYPE': 'xmpp',
    }

HipChat makes a few assumtions that are different from standard XMPP clients. First, you **must**
specify the :data:`~helga.settings.NICK` setting as the user's first name and last name::

    NICK = 'Helga Bot'

Also, if you want @ mentions to work with command plugins so that this will work::

    @HelgaBot do something

Set :data:`~helga.settings.COMMAND_PREFIX_BOTNICK` as the string '@?' + the @ mention name of the user.
For example, if the @ mention name is 'HelgaBot'::

    COMMAND_PREFIX_BOTNICK = '@?HelgaBot'

Finally, HipChat does not require that room members have unique JID values. Considering a user in a room
might have a JID of ``room@host/user_nick``, the default XMPP backend assumes that ``user_nick`` is unique.
HipChat does something a little different and assumes that the resource portion of the JID is the user's
full name like ``room@host/Jane Smith``, which may not be unique. This means that replies from the bot
that include a nick will say 'Jane Smith' rather than an @ mention like '@JaneSmith'. To enable @ mentions
for bot replies, you should install the `hipchat_nicks`_ plugin and add ``HIPCHAT_API_TOKEN`` to your settings file:

.. code-block:: bash

    $ pip install helga-hipchat-nicks
    $ echo 'HIPCHAT_API_TOKEN = "your_token"' >> path/to/your/settings.py


.. _config.slack:

Slack Support
^^^^^^^^^^^^^
`Slack`_ supports rich formatting for messaging, and Helga includes a connector
for Slack's APIs. A configuration for connecting to Slack might look like::

    SERVER = {
        'TYPE': 'slack',
        'API_KEY': 'xoxb-12345678901-A1b2C3deFgHiJkLmNoPqRsTu',
    }

When you set up a new bot API key, Slack will prompt you to configure the bot's
username. This will be Helga's nickname, and Helga will figure it out
automatically, so do not specify :data:`~helga.settings.NICK` in your
configuration. Similarly, Slack uses the "@ mentions" syntax for addressing
nicks, and the connector has support for this, so you should not set
:data:`~helga.settings.COMMAND_PREFIX_BOTNICK` in your configuration.

.. _`HipChat`: https://www.hipchat.com/
.. _`HipChat XMPP Settings`: https://hipchat.com/account/xmpp
.. _`hipchat_nicks`: https://github.com/shaunduncan/helga-hipchat-nicks
.. _`Slack`: https://www.slack.com/
