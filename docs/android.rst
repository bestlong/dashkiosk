.. _android:

Android application
===================

This is a simple Android application whose purpose is to display
fullscreen non-interactive dashboards on Android devices. Its main use
is to be run from an Android stick plug on some TV to run the web
application to display dashboards.

.. warning::
   The web engine embedded inside the application is based on Chromium
   but is not up-to-date. Until this is resolved, it is therefore
   safer to only display trusted data. One of the main selling point
   of this application is to have a decent web browser with hardware
   acceleration. An obvious replacement would be to use a regular
   browser.
   

Supported devices
-----------------

Currently, the minimal version of Android is 4.1 (Jelly
Bean). *Dashkiosk* is using the `Crosswalk project`_ to provide an
up-to-date webview with support of recent technologies.

There are a lot of Android devices that you can choose to run
Dashkiosk on. When choosing one, prefer the ones which can be upgraded
to Android 4.2.

Many cheap devices are using a Rockchip SoC limited to a 720p
resolution. This can be circumvented with some non official
experimental firmwares but usually, this is a sufficient resolution to
display dashboards.

The following devices [#devices]_ are known to work reasonably well:

 - `MK809III Mini PC (RK3188 SoC) <https://www.amazon.com/MK809III-Android-Mali-400-OpenGLES2-0-OpenVG1-1/dp/B00CZ7RBIU>`_
 - `T95N-Mini M8SPro (Amlogic S905X) <https://www.fasttech.com/products/1110/10023228/5021800>`_
 - `M96X-II MINI (Amlogic S905X) <https://www.fasttech.com/products/9580600>`_

.. _issue: https://github.com/vincentbernat/dashkiosk/issues/new

Features
--------

 - It registers as a possible home screen. It is therefore to run the
   application on boot.

 - It provides a really fullscreen webview. Absolutely no space lost
   in bars.

 - No possible interactions. If run on a tablet, the user is mostly
   locked out. However, there are still some way to interact with the
   device while the application is running by invoking the settings
   and changing the home application from here.

 - Prevent the device going to sleep.

Compilation
-----------

If you don't want to compile the Android app yourself, you can
download a `pre-compiled version from GitHub`_.

.. _pre-compiled version from GitHub: https://github.com/vincentbernat/dashkiosk/releases/

Building from source is just a matter of following those two simple
steps:

  1. Clone the `git repository`_.

  2. Build the application with the following command::

        ./gradlew assemble

At the end of the compilation, you get
``build/outputs/apk/dashkiosk-android-debug.apk`` that should be
installed on the Android device.

Installation
------------

You need the ``adb`` tool. On Debian and Ubuntu, you can install the
``android-tools-adb`` package to get it. Otherwise, it is available in
the ``platform-tools`` of the Android SDK. If you didn't install the
SDK yourself, it should be in ``~/.android-sdk``. If ``adb`` is not
present in the ``platform-tools`` directory, you can install it with::

    tools/android update sdk --no-ui --all --filter platform-tool

You can then install the APK on a device attached through USB on your
computer with the following command::

    adb install -r build/outputs/apk/dashkiosk-android-debug.apk

Alternatively, you can just point a browser to the APK and you will
get proposed to install it. You need to ensure that you allowed the
installation of APK from unknown sources.

The next step is to run the configuration panel. This panel can be
accessed by using the back button while the loading screen is
running. It can be accessed later by clicking on the pen icon in the
action bar.

Configuration
-------------

The **orientation** is configured to *landscape* by default. You can
choose either *auto* or *portrait*.

If you want to lock a bit the application, you can **lock settings**
to prevent any further modifications. You can still revert the changes
by invoking the preferences activity with ``adb``::

    adb shell am start -n \
       com.deezer.android.dashkiosk/com.deezer.android.dashkiosk.DashboardPreferences

The important part is to input the **receiver URL**. You can check
that this is the correct URL with any browser. You should see a
dashboard with some nice images cycling.

The **timeout** is not really important. Until the application is able
to make contact with the receiver, it will try to reload the receiver
if the timeout is reached.

Alternatively, the configuration can be done at compile-time by
modifying ``res/xml/preferences.xml``.

Certificates
------------

Server certificates
~~~~~~~~~~~~~~~~~~~

Unfortunately, it is currently not possible to trust third-party
certificates. Trusted certificates are built into the app and cannot
be modified.

The only possibility is to accept untrusted certificates in the
preferences. This makes TLS useless and you could just use HTTP,
except if you are interested in client certificates. In this case,
blindly trusting the server certificate doesn't allow an attacker to
use your client certificate for its own requests (client has to
demonstrate its ability to sign a the whole handshake with its
certificate, including the "server certificate" message).

Client certificates
~~~~~~~~~~~~~~~~~~~

It is possible to use client certificates. The support is still quite
new and may be troublesome to implement. Be sure to use ``adb
logcat -s DashKiosk AndroidRuntime`` while running to spot any error.

Creating a keystore
+++++++++++++++++++

Currently, you can only provide one client certificate and it will be
used with any site requesting a client certificate. The certificate
needs to be provided as a BKS (BouncyCastle KeyStore). You can either
use ``keytool`` or `Portecle`_, a graphical tool to manage such a
store. You can find a `cheatsheet`_ to use ``keytool``. If you already
have your client certificate as a PKCS#12 file, you only need to use
``keytool -importkeystore``::

    keytool -importkeystore \
            -destkeystore clientstore.bks \
            -deststoretype BKS \
            -provider org.bouncycastle.jce.provider.BouncyCastleProvider \
            -providerpath /usr/share/java/bcprov.jar \
            -srckeystore client.p12 \
            -srcstoretype PKCS12

You will be prompted the password to protect the newly created
keystore and the password protecting the PKCS#12 file. Ensure you use
the same password for both: ``keytool`` seems to protect the private
key with the password from the PKCS#12 file while *Dashkiosk* will use
the same password for the private key and for the keystore.

On Debian, ``bcprov.jar`` is from the ``libbcprov-java`` package. Be
sure to only put one keypair in the store. *Dashkiosk* wil always use
the first one.

If you have your certificates in PEM format, you can convert them in
PKCS#12 with the following command::

    openssl pkcs12 -export -out client.p12 \
                   -in cert.pem \
                   -inkey key.pem \
                   -certfile ca.pem

You can import several certificates in the keystore.

Providing the keystore to the application
+++++++++++++++++++++++++++++++++++++++++

There are two ways to provide a client certificate to the
application. The first one is to put the certificate on the
filesystem. For example, in ``/sdcard/dashkiosk.bks``. Then, in the
preferences, ensure to untick *Embedded keystore* and tick *External
keystore*, then specify the path to the keystore in *Keystore
path*. The second one is to embed the client certificate directly into
the application. Replace the file ``res/raw/clientstore.bks`` by your
own and recompile the application. In the preferences, ensure you tick
*Embedded keystore*. In both cases, you also need to provide the
password protecting the keystore.

Grant permissions to read the keystore
++++++++++++++++++++++++++++++++++++++

Starting from Android 6, you also have to grant *Dashkiosk* the
permission to access the keystore if you use the external one. This
can be done in *Android Settings*. Go to *Applications*, click on
*Dashkiosk*. You should see a *Permissions* tab. The only item in this
tab should be *Storage*. Enable it.

Usage
-----

Once configured, just run the application as usual. You can also click
on the home button and choose the application from here to make it
starts on boot.

Troubleshooting
---------------

Still with ``adb``, you can see the log generated by the application
with the following command::

    adb logcat -s DashKiosk AndroidRuntime

The log also includes Javascript errors that can be generated by the
dashboards. Javascript errors from the receiver are prefixed with
``[Dashkiosk]``.

.. _Android SDK: https://developer.android.com/studio/index.html#downloads
.. _Gradle: https://gradle.org/
.. _git repository: https://github.com/vincentbernat/dashkiosk-android
.. _Crosswalk project: https://crosswalk-project.org/
.. _Portecle: http://portecle.sourceforge.net/
.. _cheatsheet: https://github.com/vincentbernat/dashkiosk-android/blob/master/certificates/generate

.. rubric:: Footnotes

.. [#devices] Please, open an `issue`_ if you want to contribute to this list.
