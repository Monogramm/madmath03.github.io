---
title: "Smooth Scrolling"
description: "How to scroll smoothly"
published: true
---

# Smooth Docking

## Presentation

If you've ever used anchors on a section of your page (`<a href="#id">`), you might have found yourself thinking that "_brutal_" access to said section is pretty ugly. :sweat:...
So, I'll briefly show you how to change that with _JavaScript_ and _jQuery_.

## _JavaScript_ functions

Retrieving the first "_scrollable_" item from the list of items passed to the function (variable arguments):

```javascript
/**
 * Function to get the first scrollable element
 * 
 * Function the get the first scrollable element among the arguments of the 
 * function.
 * 
 * @author brunotm
 * 
 * @param $ jQuery instance
 * @param els dummy parameter
 */
function scrollableElement($, els) {
    for (var i = 1, argLength = arguments.length; i < argLength; i++) {
        var el = arguments[i], $scrollElement = $(el);
        if ($scrollElement.scrollTop() > 0) {
            return el;
        }
        else {
            $scrollElement.scrollTop(1);
            var isScrollable = $scrollElement.scrollTop() > 0;
            $scrollElement.scrollTop(0);
            if (isScrollable) {
                return el;
            }
        }
    }
    return [];
}
```

Function to go to a given portion of the page:

```javascript
/**
 * Function to smoothly scroll to an element of the page
 * 
 * Once the animation is ended, the location hash will be replaced by the 
 * one that we moved to (in order to reproduce the default behavior).
 * 
 * @author brunotm
 * 
 * @param $ jQuery instance
 * @param id Id of the element to scroll to
 */
function goToByScroll($, id, time){
	$(scrollableElement($, 'html', 'body'))
	.animate({scrollTop: $('#'+id).offset().top}, 
		(time != null ? time : 800), 
		function () {
		    location.hash = '#'+id;
		}
	);
}
```

And finally, a function that will parse your page and replace the anchors (`#id`) with a _JavaScript_ function that will move the _scrollbar_ to the desired section:

```javascript
/**
 * Function to add a smmoth scrolling to every anchor in the page
 * 
 * This function will parse the page, looking for every anchor on an existing
 * section of the current page and replace the default behavior (goto section)
 * by a 'smooth scrolling to section' animation.
 * 
 * It is highly advised to launch this function when the page is LOADED (i.e. 
 * not ready), meaning:  &lt;body onload="javascript: autoSmoothScrolling(jQuery)"&gt;
 * 
 * @author brunotm
 * 
 * @param $ jQuery instance
 */
function autoSmoothScrolling($) {
	$.browser.chrome = /chrome/.test(navigator.userAgent.toLowerCase());
	$.browser.ipad   = /ipad/.test(navigator.userAgent.toLowerCase());
	
    var locationPath = getUrlPath();
    
	// the ipad already smoothly scrolls and does not detect the scrollable
    // element if top=0; as such we disable this behaviour for the iPad
    if (!$.browser.ipad) {
    	// for each anchor in the page
        $('a[href*=#]').each(function () {
            var thisPath = filterPath(this.pathname) || locationPath;
            // if anchor to the same page 
            if (locationPath == thisPath && (location.hostname == this.hostname || !this.hostname) && this.hash.replace(/#/, '')) {
                var $target = $(this.hash), target = this.hash;
                // if target exists
                if ($target.length > 0) {
                    $(this).click(function (event) {
                        event.preventDefault(); // prevent the 'goto target' behavior
                        goToByScroll($, target.substr(1));
                    });
                }
            }
        });
    }
}
```

I would like to thank Patate _[Aurelie DHENRY](https://www.linkedin.com/in/aurelie-dhenry)_ for giving me the idea of this evolution :smiley:

## CSS

Nowadays, you can globally enable this behavior with simple _CSS_:

```css
html {
  scroll-behavior: smooth;
}
```

See <https://www.w3schools.com/howto/howto_css_smooth_scroll.asp> for details.
