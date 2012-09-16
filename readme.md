#Commands and Utilities for OS X Development

A set of commands to set up a VirtualHost workflow on OS X's default Apache setup. This is my personal approach to a set-up, and requires a little adjustment of the Apache defaults to work.

One thing about the Apache install on OS X (Lion and Mountain Lion) compared to Ubuntu for instance is the difficulty in setting up VirtualHosts. I like the way there is a separate folder for `/sites-available` and `/sites-enabled` on Ubuntu and always wondered why this wasn't the same on OS X.

For my setup, I have mimiced the Ubuntu approach only slightly, as I see no real need for both folders on a local development environment. Sure, if the Mac was a server for instance, I would probably do it. Well, I would actually use Mac OS X Server but that's a whole different story ;)

##Disclaimer and License

I hold absolutely no responsibility if this fucks your setup. It's your setup, and your responsibility, so I advise you to read and understand what the two commands do to your setup. I use this every day and it works fine for me, but I may have missed some points so if you think I have, let me know and I'll add them. If you have suggestions, then please, let me know.

This is licensed under a [Creative Commons CC0](http://creativecommons.org/publicdomain/zero/1.0/) License.

##Set up Folders for development

To begin, we need to add some folders to our setup.

1.	Create a `Sites` folder for every active user on the system, including the Shared user.
2.	Create a `__logs/vhosts` folder within each `Sites` folder. This will store a log for your specific VirtualHosts.
2.	Create a `vhosts` folder under `/etc/apache2`.
3.	Copy the `template.http` file into `/etc/apache2/vhosts`.

**Personal Tip:** I always set up another folder within my Sites folder called `__setup` in which I symlink to all the setup files for my Apache (`/etc/apache2/httpd.conf`, `/etc/apache2/extra/httpd-vhosts.conf`, `/etc/apache2/vhosts`), my hosts (`/etc/hosts`) and my PHP configuration (`/etc/php.ini`). It just makes it quick and easy to get to them when I need to.

Once we have those folders set up, we can move on to configuring our Apache environment, and also configuring PHP.

##Set up Apache defaults

1.	Open `/etc/apache2/httpd.conf` for editing.
2.	Once open, do each of the following:
	-	Uncomment `LoadModule php5_module libexec/apache2/libphp5.so`.
	-	Set the `ServerAdmin` to a useful value.
	-	Set the `DocumentRoot` to "/Users/Shared/Sites" (This is a more easily accessible location than the default).
	-	Set the `<Directory>` location from the default `DocumentRoot` to the value we just set. The result should be `<Directory "/Users/Shared/Sites">`. **Don't touch the settings for `<Directory />`**.
	-	Within the block we changed, set `Options Indexes FollowSymLinks MultiViews`.
	-	Uncomment `Include /private/etc/apache2/extra/httpd-vhosts.conf`.

You've probably noticed, we've left the User and Group settings as default (`_www` and `_www`). This is because we are setting up a multi user environment here. I do this as I have a different logon to separate my work work from personal work. Also, it isn't as safe to use another user or group. It will mean permissions issues later down the line, but we can resolve them easily. If you want to change this, then you must remember to freflect the changes in the `fixperms` script.

##Configure Apache for custom Vhosts

Here's the part where we configure Apache to use the `vhosts` folder we set up earlier.

1.	Open `/etc/apache2/extra/http-vhosts.conf` for editing.
2.	Once open, do each of the following:
	-	Comment out the default vhost configurations starting `<VirtualHost *:80>`.
	-	Add `Include /private/etc/apache2/vhosts/*.conf` to the bottom of the file.

Very simple.

##Add commands for vhost configuration

These commands are the secret to this quick and easy setup of VirtualHosts.

1.	Add `mkvhost` and `fixperms` to `/usr/local/bin`.
2.	Run `sudo chmod +x /usr/local/bin/mkvhost /usr/local/bin/fixperms` to make them executable.

##Configure PHP

1.	Run `sudo cp /etc/php.ini-default php.ini`.
2.	Open `/etc/php.ini` for editing.
3.	Once open, do each of the following:
	-	Set `max_execution_time = 60` I always do this in development.
	-	If you want a higher memory limit, `memory_limit = 256M`.
	-	Set `display_errors = On` in development.
	-	Set `html_errors = On` for development, as it's useful.
	-	Set `post_max_size` to a higher value. I have been known to use `1026M` here before, dependant on what the project was. This also has a knock on effect…
	-	Set `upload_max_filesize` to just less than the `post_max_size`. I have been known to use `1024M` here. I shall explain a little below.
	-	Rename all of the Dynamic Extenions to use`.so` instead of `.dll`. *(Why this is default to Windows `.dll` in PHP as a Unix developed language I will never know.)*
	-	Uncomment any extension you know you will use.
	-	Configure any defaults you need for these extensions.

**Personal Tip:** Setting the `upload_max_filesize` to just less than the `post_max_size` will help when your website users upload files that are larger than `post_max_size`, [as described here](http://stackoverflow.com/questions/2133652/how-to-gracefully-handle-files-that-exceed-phps-post-max-size/2133726#2133726). A very annoying thing happens as PHP will just dump the entire `$_POST` array with no warning. Doing it this way means that you will at least get a PHP error when `upload_max_filesize` is exceeded.

##Creating VirtualHosts

So now you have everything set up, we're ready to create VirtualHosts. Firstly, lets try starting Apache to ensure we've set it up properly.

    sudo apachectl -k start

If any errors pop up, deal with them accordingly.

Once you're ready to start, then run the following command

    sudo mkvhost username www.sitename.com

This will take you through a few options, tell you if anything is wrong, and set up your VirtualHost. It will also add an entry in your `/etc/hosts` file.
