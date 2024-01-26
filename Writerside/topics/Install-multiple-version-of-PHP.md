# Install multiple version of PHP

In this guide we are going to see how to install multiple version of PHP different from the  one shipped with Debian. 
At the moment of writing current Debian version (12) is shipped with PHP 8.

## Adding repository
Because on the official Debian repository only one version is available we need to add an alternative repository that
contains the missing once. In this guide we are going to use _ondrej/php_ ppa (via sury packages).

First things we need to install requirements so we are going to use:
```shell
apt-get update && apt-get -y install lsb-release ca-certificates curl
```

Now we need to retrieve repository signing key. We can do this with the following command:
```shell
curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/php/apt.gpg
```

Then as last step we can add the repository as follows:
```shell
sh -c 'echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list'
```

## Installing packages
Now that we have added _ondrej/php_ repos we can finally refresh packages list with:
```shell
apt-get update 
```

As last step we can install the desired php version. Is **important** to indicate specific version of PHP desired. We
can install following this packages name schema:
```shell
phpX.Y-<package>
```
Where _X_ is the main version of php (e.g. 5,7,8) and _Y_ the subversion (for example for php 7.3 will be 3). In the end
we need to specify with packages we want.

Warning!
: If are going to install a packages that **require** _php_ as dependencies is possible that APT wil try to install
the latest version available beside the one already installed. To mitigate this issue you can switch to the desired
_fpm engine_ on your webserver.