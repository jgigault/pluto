# html-template - old-style classic HTML template language / engine like it's 1999 (incl. back-to-the-future ERB converter)


* home  :: [github.com/feedreader/pluto](https://github.com/feedreader/pluto)
* bugs  :: [github.com/feedreader/pluto/issues](https://github.com/feedreader/pluto/issues)
* gem   :: [rubygems.org/gems/html-template](https://rubygems.org/gems/html-template)
* rdoc  :: [rubydoc.info/gems/html-template](http://rubydoc.info/gems/html-template)
* forum :: [groups.google.com/group/wwwmake](http://groups.google.com/group/wwwmake)




## Intro

Note: The original [`HTML::Template`](https://metacpan.org/pod/HTML::Template) package was written by Sam Tregar et al
(in Perl with a first 1.0 release in 1999!)
and the documentation here
is mostly a copy from the original
with some changes / adaptions for
the ruby version.



First you make a template - this is just a normal HTML file with a few extra tags, the simplest being `<TMPL_VAR>`

For example, `test.html.tmpl`:

```
<html>
<head><title>Test Template</title></head>
<body>
My Home Directory is <TMPL_VAR home>
<p>
My Path is set to <TMPL_VAR path>
</body>
</html>
```

Now you can use it in your script:

``` ruby
require 'html/template'

template = HtmlTemplate.new( filename: 'test.html.tmpl' )

puts template.render( home: ENV['HOME'],
                      path: ENV['PATH'] )
```

If all is well in the universe this should output something like this:

``` html
<html>
<head><title>Test Template</title></head>
<body>
My Home Directory is /home/alice
<p>
My Path is set to /bin;/usr/bin
</body>
</html>
```


## Usage


This library attempts to make using HTML templates simple and natural.
It extends standard HTML with a few new HTML-esque tags - `<TMPL_VAR>` `<TMPL_LOOP>`, `<TMPL_IF>`, `<TMPL_ELSE>` and `<TMPL_UNLESS>`. The file written with HTML and these new tags is called a template. It is usually saved separate from your script - possibly even created by someone else! Using this module you fill in the values for the variables, loops and branches declared in the template. This allows you to separate design - the HTML - from the data, which you generate in the ~~Perl~~ Ruby script.

### The Tags

`TMPL_VAR` • `TMPL_LOOP` • `TMPL_IF` • `TMPL_ELSE` • `TMPL_UNLESS`


####  `TMPL_VAR`

    <TMPL_VAR name>

The `<TMPL_VAR>` tag is very simple.
When the template is output - for each `<TMPL_VAR>` tag you
use in the template the `<TMPL_VAR>` is replaced with the value text you specified.
If you don't set a value it just gets skipped in the output.

**Attributes**

The following "attributes" can also be specified in template var tags:

`ESCAPE`

This allows you to escape the value before it's put into the output.

This is useful when you want to use a `TMPL_VAR` in a context where those characters would cause trouble. For example:

    <input name=param type=text value="<TMPL_VAR name>">

If you use a value like `sam"my` you'll get in trouble with HTML's
idea of a double-quote. On the other hand,
if you use `ESCAPE=HTML`, like this:

    <input name=param type=text value="<TMPL_VAR name ESCAPE=HTML>">

You'll get what you wanted no matter what value happens
to be passed in.

The following escape values are supported:

`HTML`

Replaces the following characters with their
HTML entity equivalent: `&`, `"`, `'`, `<`, `>`

`NONE`

Performs no escaping. This is the default.




#### `TMPL_LOOP`

    <TMPL_LOOP name> ... </TMPL_LOOP>

The `<TMPL_LOOP>` tag is a bit more complicated than `<TMPL_VAR>`. The `<TMPL_LOOP>` tag allows you to delimit a section of text and give it a name. Inside this named loop you place `<TMPL_VAR>`s. Now you pass in a list (an array) of values for this loop. The loop iterates over the list and produces output from the text block for each pass. Here's an example:

In the template:

    <TMPL_LOOP employees>
      <p>
      Name: <TMPL_VAR name><br>
      Job:  <TMPL_VAR job>  
      </p>
    </TMPL_LOOP>

In your Ruby code:

``` ruby
puts template.result( employees: [ { name: 'Sam',   job: 'programmer' },
                                   { name: 'Steve', job: 'soda jerk' }
                                 ]
                    )
```

The output is:

``` html
<p>
Name: Sam<br>
Job: programmer
</p>

<p>
Name: Steve<br>
Job: soda jerk
</p>
```

As you can see above the `<TMPL_LOOP>` takes a list of values
and then iterates over the loop body producing output.


**Loop Context Variables**

Inside a loop extra variables that depend on the loop's context
are made available if the `loop_vars` option is set to true (yes, true by default). These are:

`__FIRST__`

Value that is true for the first iteration of the loop
and false every other time.

`__LAST__`

Value that is true for the last iteration of the loop and false every other time.

`__INNER__`

Value that is true for the every iteration of the loop except for the first and last.

`__OUTER__`

Value that is true for the first and last iterations of the loop.

`__ODD__`

Value that is true for the every odd iteration of the loop.

`__EVEN__`

Value that is true for the every even iteration of the loop.

`__COUNTER__`

An integer (starting from 1) whose value increments for each iteration of the loop.

`__INDEX__`

An integer (starting from 0) whose value increments
for each iteration of the loop.


Just like any other `TMPL_VAR`s these variables can be used in `<TMPL_IF>`, `<TMPL_UNLESS>` and `<TMPL_ELSE>` to control how a loop is output.

Example:

```
<TMPL_LOOP foos>
  <TMPL_IF __FIRST__>
    This only outputs on the first pass.
  </TMPL_IF>

  <TMPL_IF __ODD__>
    This outputs every other pass, on the odd passes.
  </TMPL_IF>

  <TMPL_UNLESS __ODD__>
    This outputs every other pass, on the even passes.
  </TMPL_UNLESS>

  <TMPL_IF __INNER__>
    This outputs on passes that are neither first nor last.
  </TMPL_IF>

  This is pass number <TMPL_VAR __COUNTER__>.

  <TMPL_IF __LAST__>
    This only outputs on the last pass.
  </TMPL_IF>
</TMPL_LOOP>
```

One use of this feature is to provide a "separator" similar in effect to the ruby method `join()`. Example:

```
<TMPL_LOOP fruits>
  <TMPL_IF __LAST__> and </TMPL_IF>
  <TMPL_VAR name><TMPL_UNLESS __LAST__>, <TMPL_ELSE>.</TMPL_UNLESS>
</TMPL_LOOP>
```

Would output something like:

    Apples, Oranges, Brains, Toes, and Kiwi.


NOTE: A loop with only a single pass will get both `__FIRST__` and `__LAST__`
set to true, but not `__INNER__`.



#### `TMPL_IF`

    <TMPL_IF name> ... </TMPL_IF>

The `<TMPL_IF>` tag allows you to include or not include a block of the template based on the value of a given name. If the name is given a value that is true for Ruby - like `true`   - then the block is included in the output. If it is not defined, or given a false value - like `false` or `nil` - then it is skipped. The names are specified the same way as with `<TMPL_VAR>`.

Example Template:

    <TMPL_IF bool>
      Some text that is ouptut only if bool is true!
    </TMPL_IF>

Now if you call `template.result( bool: true )` then the above block will be included by output.

`<TMPL_IF> </TMPL_IF>` blocks can include any valid HTML template construct - `VAR`s and `LOOP`s and other `IF/ELSE` blocks.



####  `TMPL_ELSE`

    <TMPL_IF name> ... <TMPL_ELSE> ... </TMPL_IF>

You can include an alternate block in your `<TMPL_IF>` block by using `<TMPL_ELSE>`. NOTE: You still end the block with `</TMPL_IF>`, not `</TMPL_ELSE>`!

Example:

    <TMPL_IF bool>
      Some text that is output only if bool is true.
    <TMPL_ELSE>
      Some text that is output only if bool is false.
    </TMPL_IF>


#### `TMPL_UNLESS`

    <TMPL_UNLESS name> ... </TMPL_UNLESS>

This tag is the opposite of `<TMPL_IF>`. The block is output if the name is set false or not defined. You can use `<TMPL_ELSE>` with `<TMPL_UNLESS>` just as you can with `<TMPL_IF>`.

Example:

    <TMPL_UNLESS bool>
      Some text that is output only if bool is false.
    <TMPL_ELSE>
      Some text that is output only if bool is true.
    </TMPL_UNLESS>





## Back to the Future - Convert HTML Templates to Embedded Ruby (ERB)

The HTML library always converts classic
HTML Template to Embedded Ruby (ERB) style.
Use the `text` attribute to get the converted template source.

Example:

``` ruby
puts HtmlTemplate.new( <<TXT ).text
<opml version="1.1">
  <head>
    <title><TMPL_VAR name ESCAPE="HTML"></title>
    <dateModified><TMPL_VAR date_822></dateModified>
    <ownerName><TMPL_VAR owner_name></ownerName>
    <ownerEmail><TMPL_VAR owner_email></ownerEmail>
  </head>

  <body>
    <TMPL_LOOP Channels>
    <outline type="rss"
             text="<TMPL_VAR name ESCAPE="HTML">"
             xmlUrl="<TMPL_VAR url ESCAPE="HTML">"
             <TMPL_IF channel_link> htmlUrl="<TMPL_VAR channel_link ESCAPE="HTML">"</TMPL_IF> />
    </TMPL_LOOP>
  </body>
</opml>
TXT
```

will print if debugging is turned on (e.g. `HtmlTemplate.config.debug = true`):

```
line 4 - match <TMPL_VAR name ESCAPE="HTML"> replacing with: <%=h name %>
line 5 - match <TMPL_VAR date_822> replacing with: <%= date_822 %>
line 6 - match <TMPL_VAR owner_name> replacing with: <%= owner_name %>
line 7 - match <TMPL_VAR owner_email> replacing with: <%= owner_email %>
line 11 - match <TMPL_LOOP Channels> replacing with: <% Channels.each_with_loop do |channel, channel_loop| %>
line 13 - match <TMPL_VAR name ESCAPE="HTML"> replacing with: <%=h channel.name %>
line 14 - match <TMPL_VAR url ESCAPE="HTML"> replacing with: <%=h channel.url %>
line 15 - match <TMPL_IF channel_link> replacing with: <% if channel.channel_link %>
line 15 - match <TMPL_VAR channel_link ESCAPE="HTML"> replacing with: <%=h channel.channel_link %>
line 15 - match </TMPL_IF> replacing with: <% end %>
line 16 - match </TMPL_LOOP> replacing with: <% end %>
```

and result in:

``` erb
<opml version="1.1">
  <head>
    <title><%=h name %></title>
    <dateModified><%= date_822 %></dateModified>
    <ownerName><%= owner_name %></ownerName>
  </head>

  <body>
    <% Channels.each_with_loop do |channel, channel_loop| %>
    <outline type="rss"
             text="<%=h channel.name %>"
             xmlUrl="<%=h channel.url %>"
             <% if channel.channel_link %> htmlUrl="<%=h channel.channel_link %>"<% end %> />
    <% end %>
  </body>
</opml>
```



## License

![](https://publicdomainworks.github.io/buttons/zero88x31.png)

The `html-template` scripts are dedicated to the public domain.
Use it as you please with no restrictions whatsoever.

## Questions? Comments?

Send them along to the [wwwmake Forum/Mailing List](http://groups.google.com/group/wwwmake).
Thanks!
