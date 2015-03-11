---
title: jQuery CSS Clip Animation Plugin
layout: post
categories: css jquery
description: "I needed to animate the CSS clip property using jQuery. I found a plugin but had to update it for jQuery 1.8.0+."
---

Yesterday I needed to animate the CSS [clip property](https://developer.mozilla.org/en-US/docs/CSS/clip) using jQuery.
[`jQuery.animate()`](http://api.jquery.com/animate/) doesn't support this natively so I went searching for a plugin.
I found exactly what I needed in [a simple plugin by Jim Palmer][1], unfortunately it hasn't been updated to work with jQuery 1.8.0+.

So here you go, a forked and updated copy of the plugin which will work with jQuery 1.8.0+.

Changes include:

* Support for jQuery 1.8.0+
* Support for the element's initial clip value to be specified via a stylesheet instead of an inline style
* Support for decimal clip values (thanks to a comment by Manoel Quirino Neto)
* Type checking `fx.end` to guard against errors in some scenarios
* Support for IE8

The element you are animating **must** have a `clip` style specified. Otherwise, usage is as you would expect.

<pre data-language="html">
&lt;div class="element" style="clip: rect(0px 155px 31px 0px);"&gt;
    content to be clipped
&lt;/div&gt;
</pre>

<pre data-language="javascript">
$('.element').animate({
    clip: 'rect(0px 155px 31px 146px)'
}, 500);
</pre>

Refer [the original plugin][1] for a demo or to give credit to the author.

<pre data-language="javascript">
/*
* jquery.animate.clip.js
*
* jQuery css clip animation support -- Joshua Poehls
* version 0.1.4

* forked from Jim Palmer&#39;s plugin http://www.overset.com/2008/08/07/jquery-css-clip-animation-plugin/
* idea spawned from jquery.color.js by John Resig
* Released under the MIT license.
*/
(function (jQuery) {

    function getStyle(elem, name) {
        return (elem.currentStyle &amp;&amp; elem.currentStyle[name]) || elem.style[name];
    }

    function getClip(elem) {
        var cssClip = jQuery(elem).css(&#39;clip&#39;) || &#39;&#39;;

        if (!cssClip) {
            // Try to get the clip rect another way for IE8.
            // This is a workaround for jQuery&#39;s css(&#39;clip&#39;) returning undefined
            // when the clip is defined in an external stylesheet in IE8. -JPOEHLS
            var pieces = {
                top: getStyle(elem, &#39;clipTop&#39;),
                right: getStyle(elem, &#39;clipRight&#39;),
                bottom: getStyle(elem, &#39;clipBottom&#39;),
                left: getStyle(elem, &#39;clipLeft&#39;)
            };

            if (pieces.top &amp;&amp; pieces.right &amp;&amp; pieces.bottom &amp;&amp; pieces.left) {
                cssClip = &#39;rect(&#39; + pieces.top + &#39; &#39; + pieces.right + &#39; &#39; + pieces.bottom + &#39; &#39; + pieces.left + &#39;)&#39;;
            }
        }

        // Strip commas and return.
        return cssClip.replace(/,/g, &#39; &#39;);
    }

    jQuery.fx.step.clip = function (fx) {
        if (fx.pos === 0) {
            var cRE = /rect\(([0-9\.]{1,})(px|em)[,]?\s+([0-9\.]{1,})(px|em)[,]?\s+([0-9\.]{1,})(px|em)[,]?\s+([0-9\.]{1,})(px|em)\)/;

            fx.start = cRE.exec(getClip(fx.elem));
            if (typeof fx.end === &#39;string&#39;) {
                fx.end = cRE.exec(fx.end.replace(/,/g, &#39; &#39;));
            }
        }
        if (fx.start &amp;&amp; fx.end) {
            var sarr = new Array(), earr = new Array(), spos = fx.start.length, epos = fx.end.length,
                emOffset = fx.start[ss + 1] == &#39;em&#39; ? (parseInt($(fx.elem).css(&#39;fontSize&#39;)) * 1.333 * parseInt(fx.start[ss])) : 1;
            for (var ss = 1; ss &lt; spos; ss += 2) { sarr.push(parseInt(emOffset * fx.start[ss])); }
            for (var es = 1; es &lt; epos; es += 2) { earr.push(parseInt(emOffset * fx.end[es])); }
            fx.elem.style.clip = &#39;rect(&#39; +
                parseInt((fx.pos * (earr[0] - sarr[0])) + sarr[0]) + &#39;px &#39; +
                parseInt((fx.pos * (earr[1] - sarr[1])) + sarr[1]) + &#39;px &#39; +
                parseInt((fx.pos * (earr[2] - sarr[2])) + sarr[2]) + &#39;px &#39; +
                parseInt((fx.pos * (earr[3] - sarr[3])) + sarr[3]) + &#39;px)&#39;;
        }
    }
})(jQuery);
</pre>

### Updated to 0.1.4

I updated the plugin to support IE8. This entailed adding a fallback for when `$('...').css('clip')` returns `undefined` which it does in IE8 when the `clip` style is defined in an external stylesheet.

[1]: http://www.overset.com/2008/08/07/jquery-css-clip-animation-plugin/
