Plugins
=======

Plugins provide much of the functionality of Stethoscope and make it easy to add new functionality
for your environment and to pick-and-choose what data sources, outputs, etc. make sense for your
organization.

Configuration for each plugin is provided in your :file:`instance/config.py` file by defining a
top-level ``PLUGINS`` dictionary. Each key in the dictionary must be the name of a plugin (as given
in :file:`setup.py`, e.g., ``es_notifications``) and the corresponding value must itself be a
dictionary with keys and values as described in the sections below for each individual plugin. For
example, your :file:`config.py` might contain:

.. code:: py

    PLUGINS = {
      'bitfit': {
        'BITFIT_API_TOKEN': '...',
        'BITFIT_BASE_URL': 'https://api.bitfit.com/',
      },
    }

The above example configures only the ``bitfit`` plugin.

.. note:: Many plugins involve communicating with external servers. Each of these respects an
   optional ``TIMEOUT`` configuration variable which controls the timeout for these external
   connections. If not set for a particular plugin, the top-level configuration variable
   ``DEFAULT_TIMEOUT`` is used. If this is not set, no timeout will be enforced.

Data Sources
------------

Google
^^^^^^

Google provides:

-  Detailed device information on ChromeOS, Android and iOS devices via their mobile management
   services.
-  Account information.

Credentials
'''''''''''

There are a few steps required to set up API access to Google for your domain.

#. Use the `setup tool
   <https://console.developers.google.com/start/api?id=admin&credential=client_key>`_ to create a
   Google API Console project and enable `Admin SDK`_ access for that project. At the "Add
   credentials to your project" stage, click the link for "service account" then "create service
   account". Check the boxes for "Furnish a new private key" and "Enable G Suite Domain-wide
   Delegation". Download the service account's credentials as a JSON file; this is what will be used
   as the content for ``GOOGLE_API_SECRETS`` below.

   .. To enable domain-wide delegation after creating the service account credentials, then from the
      "Credentials" screen, click "Manage service accounts" on the right-hand side. Find the service
      account you created and click the three-dots icon on the far right of the row, then "Edit".

#. You should now see row in the table under "Service accounts" for the newly-created service
   account. Click the corresponding "View Client ID" link and record the numeric client ID from the
   subsequent dialog.
#. Follow the instructions `here
   <https://developers.google.com/admin-sdk/directory/v1/guides/delegation#delegate_domain-wide_authority_to_your_service_account>`_
   to authorize the service account for the specific APIs needed. Stethoscope's defaults are in
   ``GOOGLE_API_SCOPES`` below.
#. You will also need a user account (see ``GOOGLE_API_USERNAME``, below) in your G Suite domain
   which has API access to the `Admin SDK`_. This can be granted via the normal G Suite Admin
   Console.

Configuration
'''''''''''''

-  ``GOOGLE_API_SECRETS``: Service account credentials for Google.
-  ``GOOGLE_API_USERNAME``: Name of the account on whose behalf the service account in
   ``GOOGLE_API_SECRETS`` will act. This account must have permissions to access the APIs from which
   you're gathering data; currently, this is just the `Admin SDK`_.
-  ``GOOGLE_API_SCOPES``: List of scopes required (depends on what information you're using from
   Google). The list in the example below covers the scopes we use.

Example
'''''''

.. code:: py

   'google' : {
     'GOOGLE_API_SECRETS': {
       "client_id": "<redacted>",
       "private_key": "-----BEGIN PRIVATE KEY-----<redacted>-----END PRIVATE KEY-----\n",
       "token_uri": "https://accounts.google.com/o/oauth2/token",
       "client_email": "<redacted>",
       "client_x509_cert_url": "<redacted>",
       "private_key_id": "<redacted>",
       "type": "service_account",
       "auth_uri": "https://accounts.google.com/o/oauth2/auth",
       "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
       "project_id": "<redacted>"
     },
     'GOOGLE_API_USERNAME': "my-service-account@example.com",
     'GOOGLE_API_SCOPES': [
       "https://www.googleapis.com/auth/admin.directory.device.chromeos.readonly",
       "https://www.googleapis.com/auth/admin.directory.device.mobile.readonly",
       "https://www.googleapis.com/auth/admin.directory.user.readonly",
       "https://www.googleapis.com/auth/admin.reports.audit.readonly",
       "https://www.googleapis.com/auth/admin.reports.usage.readonly",
     ],
   }

.. _Admin SDK: https://developers.google.com/admin-sdk/

JAMF
^^^^

JAMF provides detailed device information on OS X systems.

Configuration
'''''''''''''

The JAMF plugin requires the following configuration variables:

-  ``JAMF_API_USERNAME``: Username for interacting with JAMF's API.
-  ``JAMF_API_PASSWORD``: Password for interacting with JAMF's API.
-  ``JAMF_API_HOSTADDR``: JAMF API URL (probably ends with ``JSSResource``).

Example
'''''''

.. code:: py

   'jamf': {
     'JAMF_API_USERNAME': "...",
     'JAMF_API_PASSWORD': "...",
     'JAMF_API_HOSTADDR': "https://example.jamfcloud.com/JSSResource",
   }

Extension Attributes
''''''''''''''''''''

JAMF does not provide enough information out-of-the-box for us to determine the status of all the
default security practices. However, we can define "Extension Attributes" in JAMF to gather the
needed information.

These scripts are available in :file:`docs/jamf_extension_attributes/`.

Auto-Update
+++++++++++

We use six extension attributes to gather information about the auto-update settings on OSX.


- ``1 Auto Check For Updates Enabled``: This attribute covers the "Automatically check for updates"
  setting.

  .. literalinclude:: jamf_extension_attributes/1-auto_check_for_updates.sh
    :language: bash
    :caption:

  .. only:: builder_html

    Download :download:`here <./jamf_extension_attributes/1-auto_check_for_updates.sh>`.


- ``2 Get New Updates in Background Enabled``: Reflects the "Download newly available updates in
  background" setting.

  .. literalinclude:: jamf_extension_attributes/2-get_new_updates_in_background.sh
    :language: bash
    :caption:

  .. only:: builder_html

    Download :download:`here <./jamf_extension_attributes/2-get_new_updates_in_background.sh>`.


- ``3 Install App Updates Enabled``: Covers the "Install app updates" setting.

  .. literalinclude:: jamf_extension_attributes/3-install_app_updates.sh
    :language: bash
    :caption:

  .. only:: builder_html

    Download :download:`here <./jamf_extension_attributes/3-install_app_updates.sh>`.


- ``4 Install OS X Updates Enabled``: Reflects the "Install OS X updates" setting.

  .. literalinclude:: jamf_extension_attributes/4-install_os_x_updates.sh
    :language: bash
    :caption:

  .. only:: builder_html

    Download :download:`here <./jamf_extension_attributes/4-install_os_x_updates.sh>`.


- ``5 Install Security Updates Enabled``: Reflects the "Install security updates" setting.

  .. literalinclude:: jamf_extension_attributes/5-install_security_updates.sh
    :language: bash
    :caption:

  .. only:: builder_html

    Download :download:`here <./jamf_extension_attributes/5-install_security_updates.sh>`.


- ``6 Install System Data Files Enabled``: Reflects the "Install system data files" setting.

  .. literalinclude:: jamf_extension_attributes/6-install_system_data_files.sh
    :language: bash
    :caption:

  .. only:: builder_html

    Download :download:`here <./jamf_extension_attributes/6-install_system_data_files.sh>`.


Firewall
++++++++

The ``Firewall Status`` extension attribute can be gathered using the following script:

.. literalinclude:: jamf_extension_attributes/firewall_status.sh
  :language: bash
  :caption:

.. only:: builder_html

  Download :download:`here <./jamf_extension_attributes/firewall_status.sh>`.

Remote Login
++++++++++++

The ``Remote Login`` extension attribute can be gathered using the following script:

.. literalinclude:: jamf_extension_attributes/remote_login.sh
  :language: bash
  :caption:

.. only:: builder_html

  Download :download:`here <./jamf_extension_attributes/remote_login.sh>`.


Screenlock
++++++++++

The ``Screen Saver Lock Enabled`` extension attribute can be gathered using the following script:

.. literalinclude:: jamf_extension_attributes/screen_saver_lock.sh
  :language: bash
  :caption:

.. only:: builder_html

  Download :download:`here <./jamf_extension_attributes/screen_saver_lock.sh>`.


Wireless MAC Address
++++++++++++++++++++

By default, JAMF only stores two MAC addresses for each device. However, some systems (e.g., Mac
Pros) can have additional MAC addresses. Since we use the wireless MAC address to tie users to
devices, we collect it with an additional extension attribute (``Wireless Mac Address``):

.. literalinclude:: jamf_extension_attributes/wireless_mac_address.sh
  :language: bash
  :caption:

.. only:: builder_html

  Download :download:`here <./jamf_extension_attributes/wireless_mac_address.sh>`.


LANDESK
^^^^^^^

LANDESK provides detailed device information for Windows systems.

Configuration
'''''''''''''

Our LANDESK plugin communicates directly with the LANDESK MSSQL server. It requires the following
configuration variables:

-  ``LANDESK_SQL_HOSTNAME``
-  ``LANDESK_SQL_HOSTPORT``
-  ``LANDESK_SQL_USERNAME``
-  ``LANDESK_SQL_PASSWORD``
-  ``LANDESK_SQL_DATABASE``

Example
'''''''

.. code:: py

  'landesk': {
    'LANDESK_SQL_HOSTNAME': '...',
    'LANDESK_SQL_HOSTPORT': 1433,
    'LANDESK_SQL_USERNAME': '...',
    'LANDESK_SQL_PASSWORD': '...',
    'LANDESK_SQL_DATABASE': '...',
  },


bitFit
^^^^^^

bitFit provides ownership attribution for devices.

Configuration
'''''''''''''

-  ``BITFIT_API_TOKEN``: API token from bitFit.
-  ``BITFIT_BASE_URL``: URL for bitFit's API (e.g., ``https://api.bitfit.com/``).

Example
'''''''

.. code:: py

   'bitfit': {
     'BITFIT_API_TOKEN': '...',
     'BITFIT_BASE_URL': 'https://api.bitfit.com/',
   },

Duo
^^^

Duo provides authentication logs.

.. warning:: Work in Progress

  The ``duo`` plugin currently suffers from a major issue which makes it unsuitable for production use
  at this time. In particular, Duo's API does not provide a method for retrieving only a single user's
  authentication logs *and* the frequency of API requests allowed by Duo's API is severely limited.
  Therefore, some method of caching authentication logs or storing them externally is required.
  However, this has not yet been implemented in Stethoscope.

Configuration
'''''''''''''

The ``duo`` plugin requires the following:

-  ``DUO_INTEGRATION_KEY``: The integration key from Duo.
-  ``DUO_SECRET_KEY``: The secret key for the integration from Duo.
-  ``DUO_API_HOSTNAME``: The hostname for your Duo API server (e.g.,
   ``api-xxxxxx.duosecurity.com``).

Values for the above can be found using `these instructions
<https://duo.com/docs/adminapi#first-steps>`__.

Example
'''''''

.. code:: py

   'duo': {
     'DUO_INTEGRATION_KEY': '...',
     'DUO_SECRET_KEY': '...',
     'DUO_API_HOSTNAME': 'api-xxxxxxx.duosecurity.com',
   },


Notifications and Feedback
--------------------------

Stethoscope's UI provides a place for users to view and respond to alerts or notifications. Plugins
provide the mechanisms to both retrieve notifications from and write feedback to external data
sources.

Notifications from Elasticsearch
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``es_notifications`` plugin reads notifications (or alerts) for the user from an Elasticsearch
cluster so they can be formatted and displayed in the Stethoscope UI.

Configuration
'''''''''''''

As with our other Elasticsearch-based plugins, the ``es_notifications`` plugin requires the
following configuration variables:

-  ``ELASTICSEARCH_HOSTS``: List of host specifiers for the Elasticsearch cluster (e.g.,
   ``["http://es.example.com:7104"]``)
-  ``ELASTICSEARCH_INDEX``: Name of the index to query.
-  ``ELASTICSEARCH_DOCTYPE``: Name of the document type to query.

Example
'''''''

.. code:: py

  'es_notifications': {
    'ELASTICSEARCH_HOSTS': ['http://es.example.com:7104'],
    'ELASTICSEARCH_INDEX': 'stethoscope_notifications',
    'ELASTICSEARCH_DOCTYPE': 'default',
  }

Feedback via REST API
^^^^^^^^^^^^^^^^^^^^^

Stethoscope allows users to respond to the displayed alerts in the UI. The ``restful_feedback``
plugin tells the Stethoscope API to send this feedback on to another REST API.

Configuration
'''''''''''''

The only configuration required for the ``restful_feedback`` plugin is:

-  ``URL``: The URL to which to POST the feedback JSON.

Example
'''''''

.. code:: py

  'restful_feedback': {
    'URL': 'https://feedback.example.com/path/to/feedback/endpoint',
  }

Logging and Metrics
-------------------

Logging Accesses to Elasticsearch
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``es_logger`` plugin tracks each access of Stethoscope's API and logs the access along with the
exact data returned to an Elasticsearch cluster.

Configuration
'''''''''''''

-  ``ELASTICSEARCH_HOSTS``: List of host specifiers for the Elasticsearch cluster (e.g.,
   ``["http://es.example.com:7104"]``)
-  ``ELASTICSEARCH_INDEX``: Name of the index to which to write.
-  ``ELASTICSEARCH_DOCTYPE``: Type of document to write.

Example
'''''''

.. code:: py

  'es_logger': {
    'ELASTICSEARCH_HOSTS': ['http://es.example.com:7104'],
    'ELASTICSEARCH_INDEX': 'stethoscope_accesses',
    'ELASTICSEARCH_DOCTYPE': 'default',
  }

Logging Metrics to Atlas
^^^^^^^^^^^^^^^^^^^^^^^^

The ``atlas`` plugin demonstrates how one might track errors which arise in the API server and post
metrics around those events to an external service. Unfortunately, there is no standard way to
ingest data from Python into `Atlas <https://github.com/Netflix/atlas>`__, so so this plugin is
provided primarily as an example to build upon.

Configuration
'''''''''''''

The ``atlas`` plugin requires:

-  ``URL``: The URL to which to POST metrics.

Example
'''''''

.. code:: py

  'atlas': {
    'URL': 'https://logging.example.com/path/to/logging/endpoint',
  }

Event Transforms
----------------

One type of plugins takes as input the merged stream of events from the event-providing plugins and
applies a transformation to each event if desired. For example, an event-transform plugin might
inject geo-data into each event after looking up the IP for the event with a geo-data service.

VPN Labeler
^^^^^^^^^^^

We provide an example event-transform plugin which tags an event as coming from an IP associated
with a given IP range, e.g., that of a corporate VPN. The ``vpn_labeler`` plugin requires the
following configuration variable:

-  ``VPN_CIDRS``: An iterable of CIDRs, e.g., ``["192.0.2.0/24"]`` (The value of this variable is
   passed directly to ``netaddr.IPSet``, so any value accepted by `that method
   <https://netaddr.readthedocs.io/en/latest/tutorial_03.html>`__ will work.)

Example
'''''''

.. code:: py

  'vpn_labeler': {
    'VPN_CIDRS': [
      '192.0.2.0/24',
    ],
  }


Device Transforms
-----------------

Similarly to event transforms, device transforms take as input a list of devices and modify that
list and/or its elements in some way. For instance, one might want to filter out virtual machines
from one's devices (as below).

VM Filter
^^^^^^^^^

This transform filters out virtual machines (VMs) by searching certain fields in each device's
data for strings matching common patterns for VMs. The `vm_filter` plugin has no configuration
variables.

Manufacturer from MAC Address
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This transform attempts to determine a device's manufacturer from its MAC address(es) and injects
the manufacturer's name into the device data. The `mac_manufacturer` plugin has no configuration
variables.


Batch Plugins
-------------

Incremental Writes to Elasticsearch
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``batch_es`` plugin writes each user's device records to Elasticsearch incrementally (i.e., as
they are retrieved by the batch process).

Configuration
'''''''''''''

-  ``ELASTICSEARCH_HOSTS``: List of host specifiers for the Elasticsearch cluster (e.g.,
   ``["http://es.example.com:7104"]``)
-  ``ELASTICSEARCH_INDEX``: Name of the index to which to write.
-  ``ELASTICSEARCH_DOCTYPE``: Type of document to write.

Example
'''''''

.. code:: py

  'batch_es': {
    'ELASTICSEARCH_HOSTS': ['http://es.example.com:7104'],
    'ELASTICSEARCH_INDEX': 'stethoscope_devices',
    'ELASTICSEARCH_DOCTYPE': 'default',
  }

POSTing a Summary via REST Endpoint
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``batch_restful_summary`` plugin collects all of the data from a run of the batch process and
POSTs that data to an external server via HTTP(S).

-  ``URL``: The URL to which to POST summary data.

Example
'''''''

.. code:: py

  'batch_restful_summary': {
    'URL': 'https://batch.example.com/path/to/endpoint',
  }


Troubleshooting
---------------

Stethoscope includes a script to check connectivity between itself and any configured plugins which
support connectivity tests. Running :program:`stethoscope-connectivity` will attempt to verify
network connectivity and, in many cases, successful authentication with all configured plugins.
Any errors will be printed on the command-line and debug logs written to :file:`connectivity.log`.

.. program:: stethoscope-connectivity

.. option:: --plugins [<plugin name>]+

   Restrict connectivity tests to plugins specified by name (the key used in your ``config.py``).

.. option:: --namespaces [<plugin namespace>]+

   Restrict connectivity tests to plugins in one of the given namespaces (as defined in
   ``setup.py``).
