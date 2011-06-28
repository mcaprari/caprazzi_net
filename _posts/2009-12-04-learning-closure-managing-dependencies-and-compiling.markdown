--- 
wordpress_id: 154
layout: post
title: "Learning Closure: managing dependencies and compiling"
date: 2009-12-04 10:42:36 +00:00
wordpress_url: http://caprazzi.net/?p=154
---

Google's [closure library](http://code.google.com/closure/library/index.html "Closure Library") offers semantics and tools to manage dependencies between modules. This is specially useful when building single-page javascript applications.

To find out how smoothly this works, I'll build a simple javascript application that given a string, displays its MD5 hash. We'll
- create a simple component
- crate a template with [closure templates](http://code.google.com/closure/templates/ "Closure Templates")
- import a third party library  ([jshash library](http://pajhome.org.uk/crypt/md5/scripts.html "jshash")) to do the md5 hashing
- build a dependency tree that includes all of the aobove using the [Dependency Calculation Script](http://code.google.com/closure/library/docs/calcdeps.html "Dependency Calculation Script")
- concatenate all dependencies in one script
<div class="xbox">
If you have used jQuery you'll find that this code is verbose and painfully similar to java.

To ease your mind, think that structured libraries like Closure and [YUI](http://developer.yahoo.com/yui/ "Yahoo User Interface Library") tend to promote more readable and maintainable code (at the expense of coolness).
 
Closure and YUI are a better fit when the goal is to build a complex application.
</div>## Try this at home
See the [demo](http://caprazzi.net/demos/closure-dependencies-example/index_live.html) of the sample application discussed below.You can browse [browse](http://github.com/mcaprari/closure-dependencies-example) or  [download](http://github.com/mcaprari/closure-dependencies-example/zipball/master) all source code discussed here, including all 3rd party libraries. I welcome all comments and corrections.## Project setup

Create an empty directory (i called mine goog-dependencies-example), enter it and download all the stuff
- Download the closure library<pre style="display:inline">svn export http://closure-library.googlecode.com/svn/trunk/ closure-library</pre>
- Download [closure template tools](http://closure-templates.googlecode.com/files/closure-templates-for-javascript-latest.zip "Closure Templates Latest") and expand it in closure-templates/
- [Download the jshash library](http://pajhome.org.uk/crypt/md5/jshash-2.2.zip) and unzip it
You should now have a structure like this:<pre>goog-dependencies-example\  |-- closure-templates\  |-- closure-library\  |-- jshash-2.2\</pre>
Publish this directory on a web server.
 
During development, I use[lighttpd](http://www.lighttpd.net/) with this [minimal configuration file](http://caprazzi.net/wp-content/uploads/2009/12/lhttpd-minimal.txt "lighttpd minimal config") to quickly publish just a directory. Save the file in the directory you want to publish, run **lighttpd -D -f lhttpd-minimal.txt** and browse to http://localhost:3030/. Lighttpd is available on ubuntu, macos (via macports) and windows (via cygwin).
## UI: template
We need a ui component with a textarea to input some text, a button to execute and and a textarea to see the results. Using closure template is definitely overkill here, but it's useful to demonstrate how to load templates a dependencies.Create the file md5.ui.template.soy<pre class="code">``{namespace net.caprazzi.md5.ui.template}/** * md5 ui template * To retrieve its contents, invoke the function * net.caprazzi.md5.template.ui.main() */{template .main}&lt;div class=&quot;md5-ui&quot;&gt;&lt;textarea class=&quot;md5-input&quot;&gt;&lt;/textarea&gt;&lt;br/&gt;&lt;button class=&quot;md5-action&quot;&gt;Hash It!&lt;/button&gt;&lt;br/&gt;&lt;textarea class=&quot;md5-output&quot;&gt;&lt;/textarea&gt;&lt;br/&gt;&lt;/div&gt;{/template}``</pre>
And execute this command to compile it to a javascript file:
<pre class="term">$ java -jar closure-templates/SoyToJsSrcCompiler.jar \--shouldProvideRequireSoyNamespaces \--outputPathFormat **js/md5.ui.template.js** md5.ui.template.soy</pre><pre class="code">``// This file was automatically generated from md5.ui.template.soy.// Please don't edit this file by hand.goog.provide('net.caprazzi.md5.ui.template');goog.require('soy');goog.require('soy.StringBuilder');net.caprazzi.md5.ui.template.main = function(opt_data, opt_sb) {  var output = opt_sb || new soy.StringBuilder();  output.append('&lt;div class=&quot;md5-ui&quot;&gt;&lt;textarea class=&quot;md5-input&quot;&gt;&lt;/textarea&gt;&lt;br/&gt;&lt;button class=&quot;md5-action&quot;&gt;Hash It!&lt;/button&gt;&lt;br/&gt;&lt;textarea class=&quot;md5-output&quot;&gt;&lt;/textarea&gt;&lt;br/&gt;&lt;/div&gt;');  if (!opt_sb) return output.toString();};``</pre>## UI: javascript
Now let's write some code that hashes the contents of the first textarea when the button is clicked.Name the file js/md5.ui.js.<pre class="code">``goog.provide('net.caprazzi.md5.ui');// NOTE the inclusion of our templategoog.require('net.caprazzi.md5.ui.template');// NOTE the inclusion of jshash// as defined in third_party_deps.jsgoog.require('jshash.md5');goog.require('goog.events');goog.require('goog.dom');net.caprazzi.md5.Ui = function(parent) {this.parent = parent;}net.caprazzi.md5.Ui.prototype.render = function() {var html = net.caprazzi.md5.ui.template.main();this.parent.innerHTML = html;this.input = goog.dom.$$('textarea', 'md5-input', this.parent)[0];this.button = goog.dom.$$('button', 'md5-action', this.parent)[0];this.output = goog.dom.$$('textarea', 'md5-output', this.parent)[0];var self = this;goog.events.listen(this.button,goog.events.EventType.CLICK,function() { self.onButton_()});}net.caprazzi.md5.Ui.prototype.onButton_ = function() {this.output.value = hex_md5(this.input.value);}``</pre>## Application entry point

This file exposes the function main(), that will be executed at page load. It only requires the two modules that uses directly
<pre class="code">``goog.provide('net.caprazzi.md5.application');goog.require('net.caprazzi.md5.ui');goog.require('goog.dom');net.caprazzi.md5.application.main = function() {var container = document.getElementById('md5-ui-container');var ui = new net.caprazzi.md5.Ui(container);ui.render();}``</pre>## deps.js: the dependencies file 

Calcdeps.py is a script that comes with closure library. It parses the source files in search of **goog.require()** and **goog.provide()** declarations. It then uses those declarations to build a dependency tree.

The dependency tree is stored in a file as a list of [goog.addDependency](http://closure-library.googlecode.com/svn/trunk/closure/goog/docs/closure_goog_base.js.source.html#line218 "addDependency documentation") calls. Each calls describes a file and the modules it provieds and requires. This line declares that md5.component.js will provide 'net.caprazzi.md5' and require 'net.caprazzi.md5.template' and others:
<pre>``goog.addDependency('js/md5.component.js',    ['net.caprazzi.md5'],    ['net.caprazzi.md5.template', 'goog.events', 'goog.dom', 'jshash']);``</pre>
Run this command in the root of the project to generate the deps.js all the dependecies (including google's). This may take a while.
<pre class="term">$ python closure-library/closure/bin/calcdeps.py \    -p closure-library/closure \    -p closure-library/third_party \    -p closure-templates \    -p js \    -i js /md5.* \    -o deps \> deps.js</pre>
The option '-o deps' tells calcdeps.py to output a dependency tree. The other options specify the source directories and files. Spend a moment to review the contents of the resulting file.
## Managing the third party library 'md5.js'

As I explained above, calcdeps uses special declarations in the javascript files to make sense of the dependencies. We know that by 'jshash.md5' i mean 'jshash-2.2/md5.js' but calcdeps could only figure this out if the file included a correct goog.require statement.

At this point we may just add the line to the file and go on with our lives. But we all agree that editing 3rd party libraries is not a good practice.

I maintain a text file 'extradeps.txt' where each line associates a module to a file, in this case the contents are
<pre>``jshash.md5 jshash-2.2/md5-min.js``</pre> 
Then a simple python script parses that file and generates the missing addDependency() statements:
<pre class="code">``import sysfor mod, path in [  line.strip().split(' ') for line in file(sys.argv[1]) ]:print "goog.addDependency('%s',['%s'],[]);" % (path, mod)``</pre>
Execute the script to see what the output looks like. Then remember to concatenate its output to deps.js each time your rebuild:
<pre class="term">$ python fix_deps_file.py extradeps.txt >> deps.js</pre>## in dev: index_dev.html

The last piece of the puzzle is the one that holds everything together: the html
<pre class="code">``&lt;html&gt;&lt;head&gt;&lt;title&gt;Hash me, hash me&lt;/title&gt;&lt;script&gt;CLOSURE_NO_DEPS=true;CLOSURE_BASE_PATH=&quot;./&quot;;&lt;/script&gt;&lt;script src=&quot;closure-library/closure/goog/base.js&quot;&gt;&lt;/script&gt;&lt;script src=&quot;deps.js&quot;&gt;&lt;/script&gt;&lt;script&gt;goog.require(&#x27;net.caprazzi.md5.application&#x27;);&lt;/script&gt;&lt;/head&gt;&lt;body onload=&quot;net.caprazzi.md5.application.main();&quot;&gt;&lt;h3&gt;Hash me, hash me&lt;/h3&gt;&lt;div id=&quot;md5-ui-container&quot;&gt;&lt;/div&gt;&lt;/body&gt;&lt;/html&gt;``</pre>
Note the two global directives before the inclusion of base.js: - **CLOSURE_NO_DEPS** stops closure from loading its own deps.js
- **CLOSURE_BASE_PATH** tells closure file loader how to build the script urls.
Note that the html file only imports closure's base.js and deps.js then invokes directly code that comes from md5.ui.js**Navigate to index.html, the application should be working.**Use firebug or another network monitor, to see how the browser is downloading many different files to satisfy all the dependencies. This is the perfect behaviour for development, because separate files are easier to debug.## going live: the single js file and index_live.html
Loading many small javascript files is an evil practice in production.We definitely want to have all our javascript files concatenated in one big blob.I previously used calcdeps.py with the option '-o deps' to build a dependency tree. Calcdeps can also concatenate all your dependancies right away, using the option '-o script'. Unfortunately if you try to run it against our code, it will terminate with an exception (Exception: Missing provider for (jshash.md5). Clearly, again, the dependency script doesn't know which files provides that modulePython to the rescue again: we'll reuse the extradeps.txt file with a new python script that takes all the external deps and puts them in a file along with the correct goog.provide() statements.``import sysmods = [  line.strip().split(' ') for line in file(sys.argv[1]) ];for mod, path in mods:print "goog.provide('%s');" % (mod)for mod, path in mods:for line in file(path):print line`` Running this script produces a calcdeps-friendly javascript file:$ mkdir tmp/$ python concat_extra_deps.py extradeps.txt > **tmp**/extra.js We can now tell calcdeps to look into tmp/ to find what's needed to build our big target file:$ python closure-library/closure/bin/calcdeps.py \-p closure-library \-p **tmp** \-p js \-p closure-templates \ -i js/md5.application.js \    -o script \> **md5_application.js**Now we can produce the final artifact and go live with index_live.html:``<html><head><title>Hash me, hash me</title><script src="md5_application.js"></script></head><body onload="net.caprazzi.md5.application.main();"><h3>Hash me, hash me</h3><div id="md5-ui-container"></div></body></html>``## Conclusions
I've done this exercise to understand if the closure's dependency management would be usable in a production environment.There was some initial misunderstanding between calcdeps and me and I had to look at base.js to understand what was going on and to find out about the global directives that influence the loader.The  syntax is neat and serves the double purpose of managing namespaces and dependencies.My impression is that at least the goog.require/provide syntax and the calcdeps.py script can be used in production with confidence.Closure dosen't allow to specify css files as dependencies, while it's possible with YUI3 (that sports a cool dependency management mechanism of his own).As an exercise you can think about adding a compilation step using google closure compiler.-teo
