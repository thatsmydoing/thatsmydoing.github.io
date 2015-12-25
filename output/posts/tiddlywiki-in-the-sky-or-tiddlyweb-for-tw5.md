<!--
.. title: TiddlyWiki in the Sky (or TiddlyWeb for TW5)
.. slug: tiddlywiki-in-the-sky-or-tiddlyweb-for-tw5
.. date: 2015-12-24 14:48:20 UTC+08:00
.. tags: sysadmin, tiddlywiki
.. category:
.. link:
.. description:
.. type: text
-->

I've always liked [TiddlyWiki](http://tiddlywiki.com). Back when it first came out, it was really amazing. A wiki all in one file, that worked in the browser. It didn't need a backend, it would just save itself as an all new HTML file with all your posts inside. I've used it a lot over the years, as a personal wiki/journal and a class notebook. I even had a blog with it at one point using one of the server-side forks.

Now, there's TiddlyWiki5 which is a rewrite of the original TiddlyWiki that looks a whole lot snazzier, and I assume has better architecture overall. It also has experimental support for all the server-side platforms (particularly TiddlyWeb) that have cropped up.

If you're just looking for a simple server setup for TiddlyWiki5, it has native support for that on its own. There's plenty of documentation on the site. But if you're looking for more advanced features (like storing your posts in git or a database), then you'll need to use it with TiddlyWeb. The problem is that most of the documentation for TiddlyWeb still refers to the old TiddlyWiki.

To support TiddlyWiki5, we'll need a version of the wiki which has the TiddlyWeb plugin already installed and configured. After that, some tweaking is necessary to get TiddlyWeb to provide what the wiki requires.

## Setting Up TiddlyWiki

TiddlyWiki5 provides a command line tool via `npm` that allows building custom versions of the wiki. In fact, it comes with templates, called "editions", that we can use for our setup. Assuming you already have it installed, create the wiki using

    tiddlywiki mywiki --init tw5tank          # create wiki from template

This creates a wiki intended for use with [Tank](https://tank.peermore.com/), which is built on top of TiddlyWeb. From here, you should look in `mywiki/tiddlers/system` which contain the entries for `SiteTitle`, `SiteSubtitle`, `DefaultTiddlers`, and `tiddlyweb-host`. The first 3 should be configured however you want. These are necessary because they're needed before the wiki can load them from the server. `tiddlyweb-host` contains the location of the TiddlyWeb server, this should be `http://localhost:8080/` if you're just testing locally. With everything configured, you can build the new wiki by running

    tiddlywiki mywiki --build

This will output the wiki to `mywiki/output/tw5tank.html`. You can now serve it using your favorite local webserver, like `python -m http.server`.

## Setting Up TiddlyWeb

The TiddlyWeb tutorial recommends using `tiddlywebwiki` which has all the plugins setup for a nice wiki instance for the old TiddlyWiki. It has a lot of features that aren't really needed, so we won't go with that. So first, we'll need to install TiddlyWeb and any plugins we might want to use.

    pip install tiddlyweb tiddlywebplugins.status tiddlywebplugins.cherrypy tiddlywebplugins.cors

Next, we'll need the tiddlyweb configuration in `tiddlywebconfig.py`

    # A basic configuration.
    # `pydoc tiddlyweb.config` for details on configuration items.

    import tiddlywebplugins.status

    config = {
        'system_plugins': ['tiddlywebplugins.status', 'tiddlywebplugins.cors'],
        'secret': '36c98d6d14618c79f0ed2d49cd1b9e272d8d4bd0',
        'wsgi_server': 'tiddlywebplugins.cherrypy',
        'cors.enable_non_simple': True
    }

    original_gather_data = tiddlywebplugins.status._gather_data

    def _status_gather_data(environ):
        data = original_gather_data(environ)
        data['space'] = {'recipe': 'default'}
        return data

    tiddlywebplugins.status._gather_data = _status_gather_data

The tweaks involved are:

 * using the status plugin which the wiki requires
 * monkeypatching the status plugin for the wiki to use the correct "recipe"
 * using cherrypy server instead of the buggy default one
 * using cors since we're not hosting the wiki itself on the same server

With that, we just need to create the store that will hold our data

    twanager recipe default <<EOF
    desc: standard TiddlyWebWiki environment
    policy: {"read": [], "create": [], "manage": ["R:ADMIN"], "accept": [], "write": ["R:ADMIN"], "owner": "administrator", "delete": ["R:ADMIN"]}

    /bags/default/tiddlers
    EOF

    twanager bag default <<EOF
    {"policy": {"read": [], "create": [], "manage": ["R:ADMIN"], "accept": [], "write": [], "owner": "administrator", "delete": []}}
    EOF

Finally, we can start the TiddlyWeb server

    twanager server

## Putting it all together

Once you have the TiddlyWeb server running, you can just go to wherever you're hosting the wiki html and it should work. You can try creating some posts, and the check mark on the sidebar should be red for a while and then turn black. Once that's done it's saved. You can refresh your browser and your posts should still be there.

At this point, you can start customizing your TiddlyWeb instance, by changing your store to something like a database, or adding authorization. You can also tweak the server setup so you won't need CORS anymore.

TiddlyWiki5 is still relatively new. I hope that eventually, support for server-side and the plugin ecosystem grows to be as great as the old TiddlyWiki.
