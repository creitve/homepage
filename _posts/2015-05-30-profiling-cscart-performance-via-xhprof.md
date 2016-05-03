---
layout:     post
title:      Profiling CS-Cart performance via XHProf
date:       2015-05-30 01:00:00
excerpt_separator: <!--more-->
---

XHProf is a profiler designed to analyze PHP applications like CS-Cart or Multi-Vendor. It may help to find a bottlenecks of your store and measure the latency of curtain functions.

1. Install XHProf extension from PECL.

    ```
    [sudo] pecl install xhprof-0.9.4
    ```

    _Keep in mind, that you need to restart the process to be able to use new extension._

2. Switch into the directory where your store is installed.

    ```
    cd /var/www/html/
    ```

3. Clone the XHProf repository to install the latest version of XHProf GUI.

    ```
    git clone https://github.com/phacility/xhprof.git
    ```

4. Insert the line at the beginning of `index.php`.

    ```php
    xhprof_enable(XHPROF_FLAGS_NO_BUILTINS | XHPROF_FLAGS_CPU | XHPROF_FLAGS_MEMORY);
    ```

5. Open `app/functions/fn.control.php`, find `fn_dispatch` function and insert the following lines after the `complete` hook.

    ```php
    $xhprof_data = xhprof_disable();

    include_once DIR_ROOT . "/xhprof/xhprof_lib/utils/xhprof_lib.php";
    include_once DIR_ROOT . "/xhprof/xhprof_lib/utils/xhprof_runs.php";

    $xhprof_runs = new XHProfRuns_Default();
    $run_id = $xhprof_runs->save_run($xhprof_data, "xhprof_foo");

    echo "<!--\n" . "/xhprof/xhprof_html/index.php?run=$run_id&source=xhprof_foo\n" . "--!>\n";
    ```

    _Note, that you shouldn't put this code after the `exit` call. Even if you do it, it won't work._

Open any page of the website and then visit `/xhprof/xhprot_html` to explore profiling reports. You can sort the resulting table in the reports by "Exclude Wall Time" column to find the most slow functions.
