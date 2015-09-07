Source for Philly Area Scala Enthusiasts [PHASE][] presentation on
[Apache Spark](http://spark.apache.org) Streaming, 10 September, 2015.

Requires the following software:

* [Ruby][]
* The Ruby `bundler` gem.
* The `lessc` command, to process the [LESS][] stylesheets.

Once you have all that stuff handy, simply run:

    $ bundle install
    $ rake

to build `dist/index.html`, a [Reveal.js][] slide show. Just
open the file in your browser, and away you go.

To create PDF versions of the slides, open the HTML slides in Chrome or Chromium. Then,
tack `?print-pdf` on the end of the URL, and print the result. See the Reveal.js documentation
for details.

[Ruby]: http://www.ruby-lang.org/
[Rake]: http://rake.rubyforge.org/
[Bundler]: http://gembundler.com/
[LESS]: http://lesscss.org/
[Reveal.js]: https://github.com/hakimel/reveal.js
