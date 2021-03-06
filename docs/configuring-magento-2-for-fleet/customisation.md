You can customise the deployment of your Magento site if needed.

Using a custom document root
----

By default your store will be served from the top level directory of your
repository. This means that a request to your-website.com/index.php will
be routed to the index.php at the root of your repository.

In some cases you want to store your application under a subdirectory and
have that directory be used as the 'document root' of your site. For example,
the top level of your repository might include a 'mystore' directory containing
your your Magento site, as well as other files and directories that shouldn't
be publically accessible. In such a case, your-website.com/index.php should be
routed to mystore/index.php.

```
my_repo/
├── mystore/
│   ├── index.php
│   └── [...]
├── super_secret_stuff/
│   └── [...]
└──  README.txt
```

This can be accomplished by creating a `.fleet/config` file in the root of your
repository and specifying the desired root directory via the the `webroot`
option.

```INI
webroot = mystore/
```

The directory should be relative to the root of your repository. Multiple levels
of subdirectory are fine, as are leading and trailing slashes.

The structure of your repository would then be as below.

```
my_repo/
├── .fleet/
│   └── config
├── mystore/
│   ├── index.php
│   └── [...]
├── super_secret_stuff/
│   └── [...]
└──  README.txt
```

The custom root setting will be applied when you create a release. If you
provide an incorrectly formatted config file, choose a web root that doesn't
exist or isn't a directory then the release creation will fail.

Your site's web root will always be accessible via the /home/deploy/.fleet-webroot
symbolic link, even if you haven't specified a custom root directory. The
structure of your repository won't be changed, and all your other files will
still be accessible at /home/deploy/app/public.

Using a custom VCL
----

In the event you want to provide your own VCL for varnish, rather than allowing
your Magento plugin to generate and apply the VCL, you can supply the file as
part of your codebase.

If the file '.fleet/varnish.vcl' exists in your codebase, it will be automatically
loaded into Varnish and made the active VCL.

Providing a crontab
----

You can supply a crontab file as a part of your codebase. The crontab should be placed in the root directory of your codebase.

If named *.crontab-admin*, the crontab will be installed on your admin node.

If named *.crontab-fe*, the crontab will be installed on all frontend nodes.

If you have tasks which should only be executed if the release is active, you can invoke  `/usr/local/bin/is-fleet-node-active`
to test that the node is active.

Your codebase will be available at /home/deploy/app/public/

For example:
```
*/5 * * * * /usr/local/bin/is-fleet-node-active && /home/deploy/app/public/do-something
```

All Fleet nodes use UTC time.

**Warning:** Do not use the `@reboot` cron time specification.  The instance
your job executes on may not have been fully initialised when this schedule
fires; this may result in unreliable behaviour.  Instead, use the Fleet feature
to [run scripts on instance
boot](customisation#running-scripts-on-instance-boot).

Note that the standard Magento cron is invoked independently of any provided crontab, you do not need to invoke it yourself. By default, Magento cron runs on the admin node and reads as follows:

```
*/5 * * * *    deploy    /usr/local/bin/is-fleet-node-active && /bin/sh /home/deploy/.fleet-webroot/cron.sh
```

Running scripts on instance boot
----

You can supply a script to be run at instance startup.

If you include an executable `.fleet/onboot` this will be executed when each instance is booted.

You have access to the following environment variables:

 * `ROLE` (eg. fe, admin, varnish)
 * `ENVIRONMENT` (eg. prod)
 * `RELEASE` (the Release ID)
 * `DOMAIN` (myfleet.f.nchr.io)


Using a Fleet-specific env.php
----

You can supply an env.php which will only be used by Fleet if you prefer to keep different settings for different environments.

If under app/etc/ you name a file *env.php-fleet* it will be renamed to *env.php* on deployment.

If you need more customisation of which *env.php* is used, you can use the onboot script to drop in a new *env.php* file.
This can be used to use a different *env.php* for each environment, or to modify the *env.php* based on the domain being served.

For example:
```
cat .fleet/onboot
#!/bin/bash
if [[ -e "/home/deploy/app/public/app/etc/env.php.${ENVIRONMENT}" ]]; then
  mv "/home/deploy/app/public/app/etc/env.php.${ENVIRONMENT}" /home/deploy/app/public/app/etc/env.php
fi
```

In general this is discouraged, as it breaks the guarantee that code deployed in one environment will
behave the same when deployed in another environment.

Platform customisations
----

Fleet is a platform, as such customisations of certain configuration cannot be done on a per-Fleet basis.

In general, custom server configuration cannot be supported, however if there is some configuration change which would improve the platform for all users we are open to suggestions.
