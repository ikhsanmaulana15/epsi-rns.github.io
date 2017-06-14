---
layout: post-sidemenu-wm
title:  "HerbstluftWM Tag Status in BASH using Dzen2 or Lemonbar"
date:   2017-06-02 17:35:15 +0700
categories: desktop
tags: [coding, bash, herbstluftwm]
author: epsi

excerpt:
  HersbtluftWM Tag Status using Dzen2 or Lemonbar.
  Modularized implementation in BASH script.

---

### Preface

> Goal: From Statusbar Parameters, until the Herbstclient Tag shown.

	Focusing in "herbstclient tag_status". 

![HerbstluftWM: Tag Status][image-hlwm-02-tag-status]{: .img-responsive }

This tutorial cover Lemonbar, and in order to use Dzen2,
any reader could use the source code in github.

-- -- --

#### Reading

Before you jump off to scripting,
you might desire to read this overview.

*	[HerbstluftWM Tag Status Overview][local-overview]

*	[Modularized HerbstluftWM in BASH][local-bash-config]

*	[Piping and Forking in BASH ][local-bash-pipe]

#### All The Source Code:

Impatient coder like me, like to open many tab on browser.

*	[github.com/.../dotfiles/.../bash/][dotfiles-bash-directory]

-- -- --

### HerbstluftWM Tag Status in Many Languages

This article is one part of a collection.
All integrated, on related to another.
So we can compare each other quickly.

Tutorial/ Guidance/ Article:
[[ Tag Status Overview ][local-overview]]
[[ BASH ][local-BASH]]
[[ Perl ][local-Perl]]
[[ Python ][local-python]]
[[ Ruby ][local-Ruby]]
[[ PHP ][local-PHP]]
[[ Lua ][local-Lua]]
[[ Haskell ][local-Haskell]]

Source Code Directory:
[[ BASH ][dotfiles-BASH]]
[[ Perl ][dotfiles-Perl]]
[[ Python ][dotfiles-python]]
[[ Ruby ][dotfiles-Ruby]]
[[ PHP ][dotfiles-PHP]]
[[ Lua ][dotfiles-Lua]]
[[ Haskell ][dotfiles-Haskell]]

-- -- --

### Screenshot

Since window manager is out of topic in this tutorial,
I present **only panel** HerbstluftWM screenshot.

#### Dzen2

![Statusbar: Dzen2 Screenshot][image-hlwm-ss-dzen2]{: .img-responsive }

#### Lemonbar

![Statusbar: Lemonbar Screenshot][image-hlwm-ss-lemon]{: .img-responsive }

-- -- --

### Directory Structure

Directory Structure has been explained in preface. 
For both Dzen2 and Lemonbar, the structure are the same.
This figure will explain how it looks
in <code>BASH script</code> directory.

![Statusbar: Directory Structure][image-01-tree-bash]{: .img-responsive }

Special customization can be done in output script,
without changing the whole stuff.

-- -- --

### Get Geometry

Let's have a look at <code class="code-file">helper.sh</code> in github.

#### View Source File:

*	[github.com/.../dotfiles/.../bash/helper.sh][dotfiles-bash-helper]

#### Get Script Argument

The original herbstluftwm  panel example,
contain statusbar for each monitor.
The default is using monitor 0,
although you can use other monitor as well.

{% highlight bash %}
$ ./panel.sh 0
{% endhighlight %}

I do not implement statusbar in multi monitor since I only ave my notebook.
But I'll pass the argument anyway for learning purpose.

The original herbstluftwm example handle this in oneliner fashion.

{% highlight bash %}
monitor=${1:-0}
{% endhighlight %}

Since we want to port to other language, we need a more clear code.
And for that reason, we wrapped them in function in module.

Here it is our code, with similar result.
I know BASH is cryptic. But we have good  manual in TLDP.
BASH do not have a variable return capability, 
so we have to use global variable <code>$monitor</code>.

{% highlight bash %}
# script arguments
function get_monitor() {
    local argument=("$@")
    local num_args=${#argument[@]}

    # ternary operator
    [[ $num_args > 0 ]] && monitor=${argument[0]} || monitor=0
}
{% endhighlight %}

And in main code we can call

{% highlight bash %}
get_monitor ${@}
echo $monitor
{% endhighlight %}

This will display <code>0</code> or else such as <code>1</code>,
depend on the script argument given.

{% highlight bash %}
0
{% endhighlight %}

#### Get Monitor Geometry

HerbstluftWM give this little tools to manage monitor geometry
by getting monitor rectangle.

{% highlight bash %}
herbstclient monitor_rect
{% endhighlight %}

This will show something similar to this.

{% highlight bash %}
0 0 1280 800
{% endhighlight %}

![HerbstluftWM: Monitor Rectangle][image-hlwm-02-monitor-rect]{: .img-responsive }

Consider wrap the code into function.
And use <code>$geometry</code> as global variable.

{% highlight bash %}
function get_geometry() {
    local monitor=$1;
    geometry=( $(herbstclient monitor_rect "$monitor") )
    if [ -z "$geometry" ] ;then
        echo "Invalid monitor $monitor"
        exit 1
    fi
}
{% endhighlight %}

Consider call this function from script later.
It is a little bit tricky,
because the variable <code>$geometry"</code> has space in it,
we have to wrap it in <code>"${geometry[@]}"</code>.

{% highlight bash %}
get_monitor ${@}
get_geometry $monitor
echo ${geometry[@]}
{% endhighlight %}

This will produce

{% highlight bash %}
0 0 1280 800
{% endhighlight %}

#### Get Panel Geometry

The Panel geometry is completely depend on the user flavor and taste.
You can put it, on top, or bottom, or hanging somewhere.
You can create gap on both left and right.

Consider this example:

{% highlight bash %}
function get_bottom_panel_geometry() {
   local panel_height=$1
   shift
   local geometry=("$@")
   
   # geometry has the format X Y W H
     xpos=$(( ${geometry[0]} + 24 ))
     ypos=$(( ${geometry[3]} - $panel_height ))
    width=$(( ${geometry[2]} - 48 ))
   height=$panel_height
}
{% endhighlight %}

We are going to use this <code>X Y W H</code>,
to get lemonbar parameter.

{% highlight bash %}
panel_height=24
get_monitor ${@}
get_geometry $monitor
get_bottom_panel_geometry $panel_height "${geometry[@]}"

echo "Lemonbar geometry: ${width}x${height}+${xpos}+${ypos}"
{% endhighlight %}

This will show something similar to this result,
depend on your monitor size.

{% highlight bash %}
Lemonbar geometry: 1280x24+0+776
{% endhighlight %}

#### Get Lemonbar Parameters

We almost done. 
This is the last step.
We wrap it all inside this function below.

{% highlight bash %}
function get_lemon_parameters() {  
    # parameter: function argument
    local monitor=$1
    local panel_height=$2

    # calculate geometry
    get_geometry $monitor
    get_top_panel_geometry $panel_height "${geometry[@]}"
    
    # geometry: -g widthxheight+x+y
    geom_res="${width}x${height}+${xpos}+${ypos}"
    
    # color, with transparency
    local bgcolor="#aa000000"
    local fgcolor="#ffffff"
    
    # XFT: require lemonbar_xft_git 
    local font_takaop="takaopgothic-9"
    local font_bottom="monospace-9"
    local font_symbol="PowerlineSymbols-11"
    local font_awesome="FontAwesome-9"

    # finally
    lemon_parameters="  -g $geom_res -u 2"
    lemon_parameters+=" -B $bgcolor -F $fgcolor" 
    lemon_parameters+=" -f $font_takaop -f $font_awesome -f $font_symbol" 
}
{% endhighlight %}

-- -- --

### Testing The Parameters

Consider this code <code class="code-file">01-testparams.sh</code>.
The script call the above function to get lemon parameters.

{% highlight bash %}
# libraries

DIR=$(dirname "$0")

. ${DIR}/gmc.sh
. ${DIR}/helper.sh
. ${DIR}/output.sh

# initialize

panel_height=24
get_monitor ${@}

get_lemon_parameters $monitor $panel_height
echo $lemon_parameters 
{% endhighlight %}

This will produce output
something similar to this result

{% highlight bash %}
-g 1280x24+0+0 -u 2 -B #aa000000 -F #ffffff 
-f takaopgothic-9 -f FontAwesome-9 -f PowerlineSymbols-11
{% endhighlight %}

#### View Source File:

*	[github.com/.../dotfiles/.../bash/01-testparams.sh][dotfiles-bash-testparams]

-- -- --

### Adjusting the Desktop

Since we want to use panel, we have to adjust the desktop gap,
giving space at the top and bottom.

{% highlight bash %}
$ herbstclient pad 0 24 0 24 0
{% endhighlight %}

For more information, do <code>$ man herbsluftclient</code>,
and type <code>\pad</code> to search what it means.

In script, it looks like this below.

{% highlight bash %}
herbstclient pad $monitor $panel_height 0 $panel_height 0
{% endhighlight %}

-- -- --

### Preparing Output

Let's have a look at <code class="code-file">output.sh</code> in github.
Before explain deep about Function,
we need to define two things that live in output module:

*	global variable: mutable state, must be initialized.

*	global constant: it is usually defined as a global variable,
	except that, the value won't be altered during script execution.
	The value is defined at the beginning of program.

#### View Source File:

*	[github.com/.../dotfiles/.../bash/output.sh][dotfiles-bash-output]

#### Mutable State: Segment Variable

The different between interval based and event based is that,
with interval based all panel segment are recalculated,
while with event based only recalculate the trigerred segment.

In this case, we only have two segment in panel.

*	Tag

*	Title

In script, we initialize the variable as below

{% highlight bash %}
segment_windowtitle=''; # empty string
tags_status=();         # empty array
{% endhighlight %}

Each segment buffered.
And will be called while rendering the panel.

#### Global Constant: Tag Name 

Assuming that herbstclient tag status
only consist of nine number element.

{% highlight bash %}
$ herbstclient tag_status
	#1	:2	:3	:4	:5	.6	.7	.8	.9	
{% endhighlight %}

We can manage custom tag names,
consist of nine string element.
We can also freely using *unicode* string instead of plain one.

{% highlight bash %}
readonly tag_shows=( "一 ichi" "二 ni" "三 san" "四 shi" 
  "五 go" "六 roku" "七 shichi" "八 hachi" "九 kyū" "十 jū")
{% endhighlight %}

#### Global Constant: Decoration

Decoration consist lemonbar formatting tag.

{% highlight bash %}
readonly separator="%{B-}%{F${color['yellow500']}}|%{B-}%{F-}"

# powerline symbol
readonly right_hard_arrow=""
readonly right_soft_arrow=""
readonly  left_hard_arrow=""
readonly  left_soft_arrow=""

# theme
readonly  pre_icon="%{F${color['yellow500']}}"
readonly post_icon="%{F-}
{% endhighlight %}

-- -- --

### Segment Variable

As response to herbstclient event idle,
these two function set the state of segment variable.

{% highlight bash %}
function set_tag_value() {
  IFS=$'\t' read -ra tags_status <<< "$(herbstclient tag_status $monitor)"
}
{% endhighlight %}

This IFS turn the tag status string into array of tags for later use.

{% highlight bash %}
function set_windowtitle() {
    local windowtitle=$1
    local icon="$pre_icon$post_icon"
    
    segment_windowtitle=" $icon %{B-}%{F${color['grey700']}} $windowtitle"
}
{% endhighlight %}

We will call these two functions later.

-- -- --

### Decorating: Window Title

This is self explanatory.
I put separator, just in case you want to add other segment.
And put the result in <code>$buffer</code>
because BASH can't return a string.

{% highlight bash %}
function output_by_title() {
    local text="$segment_windowtitle $separator  "
    buffer=$text
}
{% endhighlight %}

-- -- --

### Decorating: Tag Status

This transform each plain tag such as <code>.2</code>,
to decorated tag names such as <code>二 ni</code>.
Note that it only process one tag.
We process all tags in a loop in other function.

This has some parts:

*	Pre Text: Color setting for Main Text
	(Background, Foreground, Underline).
	Arrow before the text, only for active tag.

*	Main Text: Tag Name by number,
	and clickable area setting.

*	Post Text: 
	Arrow after the text, only for active tag.

*	Color Reset: <code>%{B-}%{F-}%{-u}</code>
	(Background, Foreground, Underline).


{% highlight bash %}
function output_by_tag() {
    local    monitor=$1    
    local tag_status=$2    
        
    local  tag_index=${tag_status:1:1}
    local   tag_mark=${tag_status:0:1}
    local   tag_name=${tag_shows[$tag_index - 1]}; # zero based

    # ----- pre tag

    local text_pre=''
    case $tag_mark in
        '#') text_pre+="%{B${color['blue500']}}%{F${color['black']}}"
             text_pre+="%{U${color['white']}}%{+u}$right_hard_arrow"
             text_pre+="%{B${color['blue500']}}%{F${color['white']}}"
             text_pre+="%{U${color['white']}}%{+u}"
        ;;
        '+') text_pre+="%{B${color['yellow500']}}%{F${color['grey400']}}"
        ;;
        ':') text_pre+="%{B-}%{F${color['white']}}"
             text_pre+="%{U${color['red500']}}%{+u}"
        ;;
        '!') text_pre+="%{B${color['red500']}}%{F${color['white']}}"
             text_pre+="%{U${color['white']}}%{+u}"
        ;;
        *)   text_pre+="%{B-}%{F${color['grey600']}}%{-u}"
        ;;
    esac

    # ----- tag by number
    
    # clickable tags
    local text_name=''
    text_name+="%{A:herbstclient focus_monitor \"$monitor\" && "
    text_name+="herbstclient use \"$tag_index\":} $tag_name %{A} "
  
    # non clickable tags
    # local text_name=" $tag_name "
    
    # ----- post tag

    local text_post=''
    if [ $tag_mark = '#' ]
    then        
        text_post+="%{B-}%{F${color['blue500']}}"
        text_post+="%{U${color['red500']}}%{+u}${right_hard_arrow}";
    fi
    
    text_clear='%{B-}%{F-}%{-u}'
     
    buffer="$text_pre$text_name$text_post$text_clear"
}
{% endhighlight %}

-- -- --

### Combine The Segments

Now it is time to combine all segments to compose one panel.
Lemonbar using <code>%{l}</code> to align left segment,
and <code>%{r}</code> to align right segment.
All tags processed in a loop.

{% highlight bash %}
function get_statusbar_text() {
    local monitor=$1
    local text=''

    # draw tags
    text+='%{l}'
    for tag_status in "${tags_status[@]}"
    do
        output_by_tag $monitor $tag_status
        text+=$buffer
    done
    
    # draw window title
    text+='%{r}'
    output_by_title    
    text+=$buffer
    
    buffer=$text
}
{% endhighlight %}

-- -- --

### Testing The Output

Consider this code <code class="code-file">02-testoutput.sh</code>.
The script using pipe as feed to lemonbar.

We append <code>-p</code> parameter to make the panel persistent.


{% highlight bash %}
# libraries

DIR=$(dirname "$0")

. ${DIR}/gmc.sh
. ${DIR}/helper.sh
. ${DIR}/output.sh

# process handler

function test_lemon() { 
    monitor=$1
    shift
    parameters=$@
    
    command_out="lemonbar $parameters -p"
    
    {
      # initialize statusbar
      set_tag_value $monitor
      set_windowtitle 'test'

      get_statusbar_text $monitor
      echo $buffer
    } | $command_out

}

# initialize

panel_height=24
get_monitor ${@}
get_lemon_parameters $monitor $panel_height

herbstclient pad $monitor $panel_height 0 $panel_height 0

# test
test_lemon $monitor $lemon_parameters
{% endhighlight %}

This will produce a panel on top.

![Statusbar: Lemonbar Screenshot][image-hlwm-ss-lemon]{: .img-responsive }

The panel only contain the initialized version of the text.
It does not really interact with the Herbstluft event.

You can also click the clickable area to see it's result.
It only show text, not executed yet.

{% highlight bash %}
herbstclient focus_monitor "0" && herbstclient use "2"
herbstclient focus_monitor "0" && herbstclient use "3"
{% endhighlight %}


#### View Source File:

*	[github.com/.../dotfiles/.../bash/01-testoutput.sh][dotfiles-bash-testoutput]

-- -- --

### Continue on Next Tutorial

It is already a long tutorial.
It is time to take a break.

We are going to continue on next tutorial
to cover interaction between the script process
and HerbstluftWM idle event.

-- -- --

Enjoy the statusbar !


[//]: <> ( -- -- -- links below -- -- -- )

{% assign asset_path = site.url | append: '/assets/posts/desktop/2017/06' %}
{% assign dotfiles_path = 'https://github.com/epsi-rns/dotfiles/blob/master/standalone/dzen2-hlwm' %}

[image-hlwm-02-tag-status]:   {{ asset_path }}/herbstclient-02-tag-status.png
[image-hlwm-02-monitor-rect]: {{ asset_path }}/herbstclient-03-monitor-rect.png
[image-hlwm-ss-dzen2]: {{ asset_path }}/hlwm-dzen2-ss.png
[image-hlwm-ss-lemon]: {{ asset_path }}/hlwm-lemon-ss.png

[image-01-tree-bash]:  {{ asset_path }}/hlwm-statusbar-01-tree-bash.png

[local-bash-config]: {{ site.url }}/desktop/2017/05/02/herbstlustwm-modularized-bash.html
[local-bash-pipe]:   {{ site.url }}/code/2017/04/15/bash-pipe-and-fork.html

[local-overview]: {{ site.url }}/desktop/2017/06/01/herbstlustwm-tag-status-overview.html
[local-bash]:     {{ site.url }}/desktop/2017/06/02/herbstlustwm-tag-status-bash.html
[local-perl]:     {{ site.url }}/desktop/2017/06/03/herbstlustwm-tag-status-perl.html
[local-python]:   {{ site.url }}/desktop/2017/06/04/herbstlustwm-tag-status-python.html
[local-ruby]:     {{ site.url }}/desktop/2017/06/05/herbstlustwm-tag-status-ruby.html
[local-php]:      {{ site.url }}/desktop/2017/06/06/herbstlustwm-tag-status-php.html
[local-lua]:      {{ site.url }}/desktop/2017/06/07/herbstlustwm-tag-status-lua.html
[local-haskell]:  {{ site.url }}/desktop/2017/06/08/herbstlustwm-tag-status-haskell.html

[dotfiles-BASH]:    {{ dotfiles_path }}/bash
[dotfiles-Perl]:    {{ dotfiles_path }}/perl
[dotfiles-python]:  {{ dotfiles_path }}/python
[dotfiles-Ruby]:    {{ dotfiles_path }}/ruby
[dotfiles-PHP]:     {{ dotfiles_path }}/php
[dotfiles-Lua]:     {{ dotfiles_path }}/lua
[dotfiles-Haskell]: {{ dotfiles_path }}/haskell

[dotfiles-bash-directory]:   https://github.com/epsi-rns/dotfiles/tree/master/herbstluftwm/bash
[dotfiles-bash-testparams]:  {{ dotfiles_path }}/bash/01-testparams.sh
[dotfiles-bash-testoutput]:  {{ dotfiles_path }}/bash/02-testoutput.sh
[dotfiles-bash-panel]:       {{ dotfiles_path }}/bash/panel.sh
[dotfiles-bash-gmc]:         {{ dotfiles_path }}/bash/assets/gmc.sh
[dotfiles-bash-helper]:      {{ dotfiles_path }}/bash/helper.sh
[dotfiles-bash-output]:      {{ dotfiles_path }}/bash/output.sh
[dotfiles-bash-pipehandler]: {{ dotfiles_path }}/bash/pipehandler.sh
