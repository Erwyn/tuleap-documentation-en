.. _dev-javascript:

Javascript
==========

Javascript code should always go in "scripts" folders.

* For the core, it should go in "src/www/scripts/"
* For plugins, it should go in "plugins/<my-plugin>/scripts/"

Build the javascript files
--------------------------

From the root directory of the Tuleap sources (you must have npm installed):

    .. code-block:: bash

        $ npm install
        $ npm run build

This command will install the tools needed, transpile the javascript to make it compatible with Internet Explorer 11 and minify (compress) it.

    .. important::

        * you have to run ``npm run build`` each time you edit a Javascript file.
        * Built javascript files go to the "assets" folder. You should never modify files in "assets" folders as they are removed when you rebuild.

While you are working, in the appropriate "scripts" folder, run the following command:

    .. code-block:: bash

        $ npm run watch

    .. important::

        * ``npm run watch`` will automatically rebuild Javascript after changes. It will also run the unit tests.
        * ``npm run test`` will run the unit tests once. It is used by the Continuous integration to validate changes.
        * ``npm run coverage`` will run the unit tests, generate HTML reports and open your default browser with those reports. You can use it to navigate in the code and check out how well unit tests cover the production code.

Where can I run npm commands ?
------------------------------

Tuleap has a long history and as more functionalities are added, more javascript is added too. Nowadays, it can get hard to know where to run ``npm run watch`` in all those plugins.
The safest way to know is to find ``package.json`` files. Here's a bash command to let you do that:

    .. code-block:: bash

        $ find . -type f -name 'package.json' -print -o -path '*/node_modules' -prune

This command returns all the folders containing a ``package.json`` file. You can run npm commands in them. Remember that not all npm scripts will be implemented, some of these folder may only have a "build" script.

.. code-block:: text

    ./ # Tuleap root folder
    ./plugins/admindelegation/www/scripts/
    ./plugins/agiledashboard/www/js/
    ./plugins/agiledashboard/www/js/kanban/
    ./plugins/agiledashboard/www/js/planning-v2/
    ./plugins/botmattermost/
    ./plugins/botmattermost_agiledashboard/
    ./plugins/botmattermost_git/
    ./plugins/create_test_env/scripts/
    ./plugins/crosstracker/scripts/
    ./plugins/document/scripts/
    ./plugins/frs/www/js/angular/
    ./plugins/git/www/scripts/
    ./plugins/graphontrackersv5/www/scripts/
    ./plugins/hudson/www/js/
    ./plugins/label/www/scripts/
    ./plugins/ldap/www/scripts/
    ./plugins/pullrequest/www/scripts/
    ./plugins/statistics/www/js/
    ./plugins/svn/www/scripts/
    ./plugins/testmanagement/www/scripts/
    ./plugins/timetracking/www/scripts/
    ./plugins/tracker/grammar/
    ./plugins/tracker/www/scripts/
    ./plugins/tuleap_synchro/scripts/
    ./plugins/velocity/www/scripts/
    ./src/www/scripts/
    ./src/www/themes/common/tlp/

Best-practices for Tuleap
-------------------------

When you submit a patch for review, we may request changes to better match the following best practices. Please try to follow them.

* Always use a Javascript file. No manual <script> tags.
* Whenever you need to run code when the page is loaded, do it like this:

    .. code-block:: javascript

        /** index.js */
        document.addEventListener("DOMContentLoaded", () => {
            // Your initialization code
        });

* Always manipulate the DOM in only one place. For example when using :ref:`Vue <vue-js>`, do not change datasets or class names in .vue files. Do it in "index.js"
* Leverage ES6 and later versions! We have setup Babel_ to let you use the full power of ES6 and beyond AND still have code compatible with the older browsers.
* In Burning Parrot pages, you can rely on the DOM4_ APIs as we already include a polyfill.
* Always make sure the Browser APIs you are using (for example DOM, Location, CustomEvent, etc.) work on our list of :ref:`supported browsers <user_supported_browsers>`. To do that you can check with the `Can I use`_ website.

Dates and time:
---------------

When you want to display a Date with time, for example the last update date of a document, you should take timezone issues into consideration.
Tuleap Users can choose their preferred timezone when registering or jon their user profiles.
This user-defined timezone can be different from both:

* their *browser*'s timezone (which is actually the timezone chosen in their Operating System).
* the Tuleap server's timezone.

Therefore, the only way to correctly display and format dates is to compute the date into the **user-defined** timezone which is the only one that will be consistent for both PHP code and Javascript code in the browser.

Unfortunately, `Native browser APIs <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DateTimeFormat>`_ always function in the *browser* timezone so we *must* use moment_ and moment-timezone_ to correctly display timestamps.

.. code-block:: JavaScript

    /** package.json */
    /**
     * Include "moment", "moment-timezone" and "phptomoment" packages
     */

.. code-block:: JavaScript

    /** webpack.config.js */
    /**
     * Add the moment locale plugin to stop moment from importing files for every locale ever
     * created. We only want the files for locales we support.
     */
    {
        //...
        plugins: [
            webpack_configurator.getMomentLocalePlugin()
        ]
        //...
    }

.. code-block:: JavaScript

    /** index.js */
    import moment from "moment";
    import "moment-timezone";

    //...

    document.addEventListener("DOMContentLoaded", () => {
        let user_locale = document.body.dataset.userLocale;
        user_locale = user_locale.replace(/_/g, "-");

        const user_timezone = document.body.dataset.userTimezone;
        const datetime_format = document.body.dataset.dateTimeFormat;

        // Set the default locale and timezone for moment
        moment.tz.setDefault(user_timezone);
        moment.locale(user_locale);
    });

.. code-block:: JavaScript

    /** a javascript or Vue file where you need to display a date */
    import moment from "moment";
    import phptomoment from "phptomoment";

    // The date received from Tuleap's REST API, formatted as an ISO 8601 string
    const date = "2018-11-23T11:24:17+01:00";

    // The date formatted as a "time ago"
    const formatted_date = moment(date).fromNow();
    // "11 days ago"

    // The date formatted according to the user-defined format
    const formatted_full_date = moment(date).format(phptomoment(datetime_format));
    // "23/11/2018 11:24" with French format "d/m/Y H:i"

    // datetime_format is the same variable as in index.js above. Pass it along using Vuex store.

Resources
~~~~~~~~~

- Exploring ES6: http://exploringjs.com/es6/index.html
- Can I use, to check what is available for major browsers: https://caniuse.com/

.. _Babel: https://babeljs.io/
.. _Can I use: https://caniuse.com/
.. _DOM4: https://github.com/WebReflection/dom4
.. _exploring ES6: http://exploringjs.com/es6/index.html
.. _moment: https://momentjs.com/
.. _moment-timezone: https://momentjs.com/timezone/