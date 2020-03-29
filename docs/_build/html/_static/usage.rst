.. _usage:

=====
Usage
=====

.. _firststart:

First Start
===============

Open your Grocy website. Click on Settings in the top right corner and then click on *Manage API keys*. Click on *add* and copy the long API key to your clipboard. You can now open Barcode Buddy by visiting your webservers URL. The setup will ask you to enter the API details. Make sure to have the trailing "/api/" for your Grocy URL at the end.


Using The Web UI
================

Overview
--------
When you open the web ui, you will see three cards:

* New Barcodes: Barcodes that are unknown to Grocy, but the name could be looked up
* Unknown Barcodes: Barcodes that are unknown to Grocy and could not be looked up
* Processed Barcodes: A history of all barcodes that were processed by BarcodeBuddy

Special Barcodes
----------------
There are seven special barcodes - if you scan the barcode, BarcodeBuddy goes into a different mode: Eg. in Purchase mode all barcodes that are scanned will be added to Grocys inventory.


+---------------------+-----------------+-----------------------------------------------------------------------------------------+
| Mode                | Default Barcode | Explanation                                                                             |
+---------------------+-----------------+-----------------------------------------------------------------------------------------+
| Consume             | BBUDDY-C        | All items scanned in this mode will be removed from the inventory                       |
+---------------------+-----------------+-----------------------------------------------------------------------------------------+
| Consume (spoiled)   | BBUDDY-CS       | All items scanned in this mode will be removed from the inventory and marked as spoiled |
+---------------------+-----------------+-----------------------------------------------------------------------------------------+
| Purchase            | BBUDDY-P        | All items scanned in this mode will be added to the inventory                           |
+---------------------+-----------------+-----------------------------------------------------------------------------------------+
| Open                | BBUDDY-O        | All items scanned in this mode will be marked as opened                                 |
+---------------------+-----------------+-----------------------------------------------------------------------------------------+
| Inventory           | BBUDDY-I        | Displays the current amount of items in stock of that product                           |
+---------------------+-----------------+-----------------------------------------------------------------------------------------+
| Add to shoppinglist | BBUDDY-AS       | All items scanned in this mode will be added to the default shopping list               |
+---------------------+-----------------+-----------------------------------------------------------------------------------------+
| Quantity            | BBUDDY-Q-?      | Sets a quantity for a barcode. Replace ? with number of items. Eg. if you have a carton |
|                     |                 | with 10 eggs, scan the barcode BBUDDY-Q-10 and then the barcode of the carton. The next |
|                     |                 | time you scan the carton barcode, it will register as 10 units.                         |
+---------------------+-----------------+-----------------------------------------------------------------------------------------+

You can find printable copies of the barcodes in the `example folder <https://github.com/Forceu/barcodebuddy/tree/master/example/defaultBarcodes>`_
.



Using the UI
------------

Once a barcode was added and is not recognised by Grocy, it will be added to the list in the web ui. Simply select the Grocy product from the drop-down list. If the name could be looked up, you will also see checkboxes for each word that was used in the name. If you want the selected Grocy product preselected for a different barcode that includes such a word, tick it and then press either "Consume" or "Add". The barcode will then saved to Grocy and the next time you scan it, the product will automatically be processed.

.. image:: https://github.com/Forceu/barcodebuddy/raw/master/example/screenshots/FullSite_small.png


Adding Barcodes
===============

Adding Barcodes Manually
------------------------

The easiest option, ideally for testing out Barcode Buddy: Simply open the web ui and click on "Add barcode". Enter the barcode (use one line per barcode) and click on "Add".

If you are using a barcode scanner, but don't want to attach it to BarcodeBuddy (yet), you can also plug it into the device that runs the webbrowser and use it to enter the barcodes in the textfield. Each line is parsed as a barcode.

Adding Barcodes automatically
-----------------------------

The preferred way. Most barcode scanners register as a USB keyboard. That way, it is possible to grab the input and send it to Barcode Buddy.

.. _attachingscanner:

Using a physical barcode scanner
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Plug in your barcode scanner to the linux computer / server you will be using. Run the command ``evtest`` as root. You will see a list of devices, select the one that is your barcode scanner and remember the number (eg. event6). Scan a barcode. You will now see output in the evtest programm. If not, you have selected the wrong source.

Docker
"""""""""""""""""

Create a docker container with
::

 docker run -v bbconfig:/config -e ATTACH_BARCODESCANNER=true -p 80:80 -p 443:443 --device /dev/input/eventX f0rc3/barcodebuddy-docker:YOURTAG

where X in ``--device /dev/input/eventX`` is the number of your event you selected previously. You might need to change the values for the ports. Scan a barcode - it should be sent directly to BarcodeBuddy.

Bare Metal
"""""""""""""""""

Navigate to the example folder in the BarcodeBuddy directory. In the file ``grabInput.sh`` edit the following values:

* If your barcode scanner is attached to the same computer / server:
   * ``SCRIPT_LOCATION``: Replace with the location where your index.php file is located
* If the scanner is attached to a different computer / server:
   * ``SERVER_ADDRESS``: Replace with the URL where your index.php file can be accessed from
   * ``USE_CURL``: Set to "true"
* If the webserver does not run as user www-data (uncommon):
   * ``WWW_USER``: Set to the name of the user

Then run as root
::

 bash grabInput.sh /dev/input/eventX

where X is the number of your event you selected previously. Scan a barcode - it should be sent directly to BarcodeBuddy.

To run the script in the background, run
::

 screen -S barcodegrabber -d -m /bin/bash /path/to/the/barcodebuddy/folder/example/grabInput.sh /dev/input/eventX


Using a 3rd party application / script
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you want to write your own script, there are two ways to send the barcodes to Barcode Buddy: either by calling ``php index.php yourBarcode`` or by calling the URL: ``https://url.to.barcode.buddy/index.php?add=123456789``. Only one barcode can be given with each call. You will only receive a return value if there was an error. If you are using the URL method, you can display the UI with the additional parameter ``showui``. For example:
::

 https://your.webhost.com/index.php?showui&add=123456789

You can also specify the mode for each barcode with a GET parameter. Simply add ``&mode=NEWMODE`` and replace ``NEWMODE`` with one of the following modes:

* consume
* consume_s (Consume spoiled)
* purchase
* open
* inventory

Example
"""""""

To show the current stock on the webui of the Product "Pizza" which already has the barcode "123456" assigned:
::

 https://your.webhost.com/index.php?showui&add=123456&mode=inventory


Using a 3rd party mobile app
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Although we have not released an app (yet), you can use the `Android app QR & Barcode Scanner <https://play.google.com/store/apps/details?id=com.scanner.kataykin.icamesscaner.free>`_ and point it to the index.php file. BarcodeBuddy supports the ``text`` GET variable that is used by the app since version 1.4.1.0.


Web UI: Settings menu
=====================

General Settings
----------------

In this tab you can set the barcodes for changing Barcode Buddy modes. For example, if you scan the barcode "BBUDDY-P", Barcode Buddy will change to "Purchase" mode and add all following items to your Grocy inventory. By default it is in "Consume" mode. The edit field below allows you to set the time in minutes, which is required to pass in order to revert back to the default "Consume" mode. E.g. if "Purchase" mode is active and the field is set to 10 minutes, Barcode Buddy will revert back to "Consume" mode 10 minutes later.

If you scan the "Inventory" barcode, Barcode Buddy will simply output the current stock, but not change any values. If an unknown barcode is scanned, it is added to the regular list.

The "Add to shopping list" barcode adds all future barcodes to the default shopping list.

With the "Revert after single item scan in "Open" or "Spoiled" mode" checkbox ticked, Barcode Buddy only stays in this mode for one scan and then reverts back to the default "Consume" mode. It does not affect the "Purchase" mode however!

With "Remove purchased items from shoppinglist" enabled, items that are scanned in purchase mode are removed from all Grocy shopping lists.

When "more verbose logs" is disabled, only barcode scans are logged in the log part of the main page.

Grocy API
---------
Here you can change your Grocy API details. Refer to :ref:`firststart`.

Websocket Status
----------------
This section gives the status of the websocket server and if BarcodeBuddy is able to connect to it


Web UI: Settings Chores
========================

This menu lists all available Grocy chores. Simply enter a barcode for a chore and press "Add". The next time you scan this barcode, the chore will be executed. To change the barcode, simply edit it and press "Edit". To remove, delete the barcode and press "Edit".


Web UI: Tags
========================

All saved tags are listed here

Adding tags
------------

Scan a barcode that was not recognized by Grocy yet, but could be looked up. Before pressing "Add" or "Consume" in the main menu, select a word from the list to the right. The next time a barcode is looked up that contains the word, the product is preselected.

Managing tags
-------------

The list shows an overview of the tags. Click on "Delete" to remove the tag.


Web UI: Quantities
========================

This features is for products that come in packs containing more than one item.

In the settings you see the quantity barcode (default "BBUDDY-Q-"). If you scan a barcode that starts with this text and has a number at the end, Barcode Buddy sets the quantity of the units from the previously scanned barcode to the number. For example: You scan Barcode "123", which is a pack of 6 eggs. Then you scan the barcode "BBUDDY-Q-6". The next time you scan the barcode "123" in purchase mode, Barcode Buddy will automatically add 6 eggs.