python-ddns
===========

A simple dynamic DNS howto using BIND9's DNSSEC update system and python.
=========================================================================


##1.

Needed: linux machine with bind9, python and a web server with a cgi-bin folder setup. This how-to assumes you are using a Debian-based distro such as Debian or Ubuntu.

##2.

Generate keys needed for nsupdate. Change dynamic.mydomain.com to whatever you want. Making them be the same is easiest.

    $ dnssec-keygen -a HMAC-MD5 -b 512 -n USER dynamic.mydomain.com.

This will create two files something like: Kdynamic.mydomain.com.+157+12505.[key|private]

##3.

Move the files to /etc/dyndns, or anywhere. Set the permissions or ownership so that the web server process can read them.

##4.

In your bind config file (/etc/bind/named.conf.local or wherever you define your mydomain.com zone) add this into the zone definition:

    allow-update {
        key dynamic.mydomain.com.;
    };

##5.

Create a file in /etc/bind/ called mydomain.com.keys.conf. The secret is from the "key" file created in step 2.

    key dynamic.mydomain.com. {
        algorithm HMAC-MD5;
        secret "4bz[...]57zy5qg==";
    };

##6.

At the end of named.conf add:

    include "/etc/bind/mydomain.com.keys.conf";

##7.

Create the cgi script in /usr/lib/cgi-bin/dyndns, or wherever your apache configuration puts the cgi-bin folder at. The package "sh" might not be installed. Go to http://www.pip-installer.org to install pip, and install sh with "sudo pip install sh". You can rewrite it to use os.system if you prefer.

Change the 1234 in the "adler32" function to some random number that only you know. It needs to match the number in the next step. This is just a secret used to verify the hash key and allow updates.


##8.

You'll need python, and you may need to install some modules with pip

    sudo apt-get install python python-pip

You can try:

    sudo apt-get install python-sh

If that doesn't work you'll have to try:

    sudo pip install sh


##9.

Create getddkey in /usr/local/bin. Change the 1234 to some random number that only you know. This needs to match the number you changed in the previous step. Set this executable "chmod +x getddkey".


##10.

You can enable updates for any subdomain, such as x.mydomain.com or home.mydomain.com, by getting the hash key for it. To get the key, run getddkey and pass the subdomain name.

    $ getddkey x
    86639914
    $ getddkey home
    372508155

(Note that your keys should be different if you changed the 1234 to some other number.)

##11.

Now to do updates use URLs like this:

    http://mydomain.com/cgi-bin/dyndns?name=x&key=86639914
    http://mydomain.com/cgi-bin/dyndns?name=x&key=86639914&ip=10.0.0.1
    http://mydomain.com/cgi-bin/dyndns?name=home&key=372508155

Leave the ip parameter off to use the IP you are calling from. If you have https setup pointing to the same cgi-bin folder you can use that to hide the key.
