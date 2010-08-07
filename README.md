Description
-----------

This is a Fuse-based perl script that allows you to create a mountpoint that 
allows you to access many web resources as a filesystem. It needs a lot of 
work w.r.t. caching, but you can play with it.

Requirements
------------

REQUIRED non-standard modules: 
* Fuse
* LWP::UserAgent
* URI

OPTIONAL modules: 
* XML::XPath 
* HTML::TreeBuilder
* JSON
* CSS

Setup
-----

    # usermod -G fuse <your username>
    # vi /etc/fuse.conf
    (enable the user_allow_other option, if you desire)

Examples
--------

    % mkdir ~/r
    % mount.restfs -d ~/r
    % cat '/r/bushong.net,dave,tmp,demo.html/links/Google REST Search/tree/responseData/results/0/unescapedUrl/links/RSS/tree/rss/channel/item[1]/title/text()'
    Blog reveals Afghanistan medic Karen Woo's dedication - BBC News
    % fusermount -u ~/r

That `cat` did the following: 

1. Pulled down the HTML web page at `http://bushong.net/dave/tmp/demo.html`
   (`bushong.net,dave,tmp,demo.html`)
2. Followed the link with the text "Google REST Search", which goes to a
   google REST API search result in JSON format (`links/Google REST Search`)
3. Traversed the JSON structure that it got from following that link to the 
   first result, and followed the url it found there
   (`tree/responseData/results/0/unescapedUrl`)
4. Clicked the first link with the text "RSS" to an RSS file
   (`links/RSS`)
5. Traversed the XML using xpath and gave you the text node's contents,
   thus retrieving the RSS "title" attribute of the first item
   (`tree/rss/channel/item[1]/title/text()`)

Specification
-------------

Assuming mount on `/mnt`, paths look like: `/mnt/<encoded url>/<info>`
where <encoded url> is a url with commas replaced with `%2C` then /s replaced 
with commas.

So, e.g. `/mnt/en.wikipedia.org,w,api.php?action=parse&page=Kittens`

(shell escaping is left as an exercise to the reader)
(you can leave off the `http:,,` if it's not HTTPS)

<info> consists of the following shared files:

* `content`: result of the request
* `url`: decoded url of the request for convenience
* `headers`: response headers

additionally, types text/plain and text/html have:

* `links/`: a directory of symlinks named (uniquely) after their related text

additionally, types text/html, text/xml, and application/json have:

* `tree/`<tree-path>

<tree-path> varies depending on the encoding, but for JSON each path
component corresponds to an object property (numbers for array indices),
and for html & xml it corresponds to an XPath

Right now the only way to unmount is "fusermount -u /path/to/mntpt" which
will also make the script exit; ^C or killing the pid won't unmount the fs!

TODO
----

* Threading 
* Cache improvements
