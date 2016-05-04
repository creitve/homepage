---
layout:     post
title:      Compiling ngx_pagespeed into Nginx in Ubuntu
date:       2016-05-02 01:00:00
excerpt_separator: <!--more-->
---

Usually in Ubuntu we might install the default Nginx very easy. But if we want some extra modules that can't be compiled as dynamic modules (such as Google's Pagespeed), we still need to recompile Nginx with those modules statically.

1. Install the PPA repository `ppa:nginx/stable`.

    ```
    sudo add-apt-repository -y ppa:nginx/stable
    ```

2. Create folder and download the latest version of [ngx_pagespeed](https://developers.google.com/speed/pagespeed/module/) and [psol](https://developers.google.com/speed/pagespeed/psol) modules.

    ```
    sudo mkdir -p /usr/src/pagespeed
    cd /usr/src/pagespeed
    NPS_VERSION=1.9.32.4
    sudo wget https://github.com/pagespeed/ngx_pagespeed/archive/release-${NPS_VERSION}-beta.zip
    sudo unzip release-${NPS_VERSION}-beta.zip
    cd ngx_pagespeed-release-${NPS_VERSION}-beta/
    sudo wget https://dl.google.com/dl/page-speed/psol/${NPS_VERSION}.tar.gz
    sudo tar -xzvf ${NPS_VERSION}.tar.gz
    ```

3. Find and uncomment the line that starts with `deb-src`. Please make sure that your source list is up-to-date.

    ```
    sudo cat<<EOF>/etc/apt/sources.list.d/nginx-stable-trusty.list
    deb http://ppa.launchpad.net/nginx/stable/ubuntu trusty main
    deb-src http://ppa.launchpad.net/nginx/stable/ubuntu trusty main
    EOF
    sudo apt-get update
    ```

4. Install utilities for building packages, source codes and also dependencies for compilation.

    ```
    sudo apt-get install -y dpkg-dev
    sudo mkdir /usr/src/nginx
    cd /usr/src/nginx
    sudo apt-get source nginx
    sudo apt-get build-dep nginx
    ```

5. Add full path to the module with `--add-module` option into the `debian/rules` file.

    ```
    full_configure_flags := \
                            $(common_configure_flags) \
                            ...
                            --add-module=/usr/src/pagespeed/ngx_pagespeed-release-1.9.32.4-beta
    ```

6. Build the package.

    ```
    cd /usr/src/nginx/nginx-1.10.0
    sudo dpkg-buildpackage -b
    ```

Be patient, it takes from 5 to 15 minutes. If you do everything correctly, you will see a bunch of `.deb` packages.
