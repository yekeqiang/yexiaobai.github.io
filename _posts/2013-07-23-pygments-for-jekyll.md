---
comments: true
date: 2013-07-23 15:00:00
layout: sampler
slug: all-builtin-pygments-styles 

title: Pygments for Jekyll
summary: "Sample of All Pygments Styles in Jekyll"
image: 'pygments/pygments.jpeg'
tags:
- jekyll
- python
- pygments
---

One of the greatest features of Jekyll is its ability to render
beautiful syntax highlighting in an effortless fashion.  Simply enable
the python library `pygments` syntax highlighter in your `_config.yml`
configuration and you're on your way.  Jekyll comes shipped with
all the goodies you need to start rendering gorgeous HTMLified code so
that all your viewers can read and totally immerse themselves in the
experience.  Once Jekyll is installed, all one must do is setup a
`highlight` block and you're good to go.

{% highlight html %}
{% raw %}{% highlight bash %}{% endraw %}
  #! /bin/bash
  echo You called me: $0
  echo With this arg: $1
  echo Thanks!
{% raw %}{% endhighlight %}{% endraw %}
{% endhighlight %}

Now, this will get you something that will render like so:

{% highlight bash %}
  #! /bin/bash
  echo You called me: $0
  echo With this arg: $1
  echo Thanks!
{% endhighlight %}

Isn't that just breath taking?  So much beauty, so little work required.
If only life could be so simple.

As I looked through the `pygments` library, I found there were also styles
associated with the highlighting.  All are pretty nice, but I was confused on
how to render the styles.  Time to dig in!

The python library `pygments` can generate a CSS for a particular style.  This can
be generated on the command line if pygments is installed on your machine.  I
choose to use the source, as I always recommend, and pulled it down from GitHub.
the standard `python setup.py install` works quite well as long as you have something
like Python 2.7 installed.  When finished you'll have an installation of `pygments` and
a script in the base directory of the source tree called `pygmentize` as well as in your
`/usr/local/bin` unless you specify otherwise.  This script makes it quite easy to generate
a CSS for use with Jekyll.  Here's a one liner to generate a CSS file:

{% highlight console %}
pygmentize -f html -S default > sampler.css
{% endhighlight %}

This CSS file can be included in the Jekyll `assets`,`css`, or similar, folder
and then referenced by a layout file.  The following could be included in your
main layout HTML file:

{% highlight html %}
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <link rel="stylesheet" href="/css/sampler.css">
{% endhighlight %}

This is great for likely 99.9% of all applications, however, wouldn't
you like to know what the other styles looked like?  Wouldn't you like
to see them side by side and decide which was best for your site?  Me too!
Even though I found a site which did post all of these, for some reason,
I couldn't let go of this problem until I solved it for myself.  I wanted
to make sure I understood the data flow from the markdown to the `pygments`
library.

Looking into the `pygments` library I found that a list of all the styles
available could be programatically determined.

{% highlight python %}
from pygments.styles import get_all_styles
styles = list(get_all_styles())
{% endhighlight %}

These style names could then be passed to the `pygmentize` script and a CSS
with all styles could be generated.  Each style can be made unique by using the
`-a name` option to prepend a `.name` selector to all style rules.  I wrote a
short script to handle the generation of the CSS and a test markdown file(don't
worry about the `pylight` tag, we'll get to that later).

{% highlight bash %}
#! /bin/bash
# get the list of all available styles for pygments
STYLES=$(python -c "from pygments.styles import get_all_styles; \
                    styles = list(get_all_styles()); \
										print ' '.join(styles)")
# create a file to store all the different styles
echo \/\* Pygments styles \*\/ > sampler.css
# create a test file to be included in a jekyll blog post
echo "" > pylight-sampler.md

for style in $STYLES;
  do
    echo Add the $style
    # generate a style sheet using the pygments command line
    # interface `pygmentize` and create a new selector of the
    # same name for all the style rules for this style
    pygmentize -f html -S $style -a .$style >> sampler.css
    echo \## $style >> pylight-sampler.md
    echo {% raw %}{% pylight ruby style=$style %}{% endraw %} >> pylight-sampler.md
    echo def print_hi\(name\) >> pylight-sampler.md
    echo '  puts "Hi, #{name}"' >> pylight-sampler.md
    echo end >> pylight-sampler.md
    echo print_hi\('Tom'\) >> pylight-sampler.md
    echo "#=> prints 'Hi, Tom' to STDOUT." >> pylight-sampler.md
    echo {% raw %}{% endpylight %}{% endraw %} >> pylight-sampler.md
    echo "" >> pylight-sampler.md
done
{% endhighlight %}

Now, we have a CSS with all the styles and some test markdown.  Next problem
is, how to render this thing?  Jekyll by default will render a code block
using a CSS `class` of `highlight` for the `div` and will create a `code` HTML
block with a CSS `class` of the language specified by the
`{% raw %}{% highlight langauge %}{% endraw %}` liquid tag.  So we could have
language specific CSS files, which is cool.  However, that's not how `pygments`
works.  It automagically sets up a specific syntax coloring scheme for each
langauge and runs it all from one stylesheet.  So, the language class doesn't
really do much for us.  Besides this, it also complicates the background color!
The class attributes for the `<div><pre><code>` triplet need to pickup the
background color from the syntax specification, but are easily overridden by other
well meaning CSS classes.  To solve this, I decided to try my hand at a little
plugin code.  It's been a good learning experience for me, and I encourage you
to try the same. :-)

I cloned the `highlight.rb` from Jekyll and modified the regular expression match
to render for the `pygment` library.  I added the ability to pass in a `style`
parameter through the tag and now I can have one CSS with many styles!  However, it's
utility is primarily academic since I'll likely only want to use one style.  Here's
the plugin that needs to go in the `_plugins` directory:


{% highlight ruby %}
module Jekyll
  module Tags
    class PylightBlock < Liquid::Block
      include Liquid::StandardFilters

      # The regular expression syntax checker. Start with the language specifier.
      # Follow that by zero or more space separated options that take one of two
      # forms:
      #
      # 1. name
      # 2. name=value
      SYNTAX = /^([a-zA-Z0-9.+#-]+)((\s+\w+(=\w+)?)*)$/

      def initialize(tag_name, markup, tokens)
        super
        if markup.strip =~ SYNTAX
          @lang = $1
          @options = {}
          if defined?($2) && $2 != ''
            $2.split.each do |opt|
              key, value = opt.split('=')
              if value.nil?
                if key == 'linenos'
                  value = 'inline'
                else
                  value = true
                end
              end
              @options[key] = value
            end
          end
        else
          raise SyntaxError.new <<-eos
Syntax Error in tag 'highlight' while parsing the following markup:

  #{markup}

Valid syntax: highlight <lang> [linenos]
eos
        end
      end

      def render(context)
        if context.registers[:site].pygments
          render_pygments(context, super)
        else
          render_codehighlighter(context, super)
        end
      end

      def render_pygments(context, code)
        require 'pygments'

        @options[:encoding] = 'utf-8'

        output = add_code_tags(
          Pygments.highlight(code, :lexer => @lang, :options => @options),
          @lang
        )

        output = context["pygments_prefix"] + output if context["pygments_prefix"]
        output = output + context["pygments_suffix"] if context["pygments_suffix"]
        output
      end

      def render_codehighlighter(context, code)
        #The div is required because RDiscount has serious issues
        <<-HTML
  <div>
    <pre><code class='#{@lang}'>#{h(code).strip}</code></pre>
  </div>
        HTML
      end

      def add_code_tags(code, lang)
        # Add nested <code> tags to code blocks
				style = @options["style"]
        code = code.sub(/<div class="highlight"><pre>/,'<div class="' + style  + '"><pre class="hll"><code class="hll">')
        code = code.sub(/<\/pre>/,"</code></pre>")
      end
    end
  end
end

Liquid::Template.register_tag('pylight', Jekyll::Tags::PylightBlock)

{% endhighlight %}

I added a new tag called `pylight` to a class called `PyLightBlock`, cute huh?
I didn't want to force the browser to always download the all encompassing CSS file
so I created a new layout for a post.  The only difference being that the new layout
includes the CSS file which has all the styles.  I would have just included the
CSS file in the blog post, but Jekyll wanted to put `<p></p>`'s around them in the
worst way.  Seeing as how that just felt wrong, I decided on the separate layout
to confine the larger CSS file to posts with that layout.  It only saves a few
kilobytes, I'm sure, but it felt cleaner and I felt better knowing that the complete
syntax style set would not be forced upon all who tread upon these pages.  Then I
included my `pylight-sampler.md` into my post and, bingo!

{% include post/pylight-sampler.md %}

