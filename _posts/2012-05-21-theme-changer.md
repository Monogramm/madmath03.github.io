---
title: "Theme changer"
description: "Alternative CSS theme"
published: true
---

# Alternative themes

Alternate themes are a (non-standard?) way to manage multiple themes for your website.
The principle consists in assigning a title via the `title` attribute of the `<link>` tag to identify a theme. A theme can be made up of several _CSS_ files and there is no limit to the number of themes that can be created.

**Warning**

It is still necessary to avoid having too many themes in a page!
At a minimum, 1 theme is equivalent to a _CSS_ file and, even if only one theme is applied at a time, all _CSS_ files are still loaded and **parsed** by the browser (significant increase in the loading time of the page).

The problem is that this functionality is only managed by _Firefox_ (display, style of the page), and it is not even persistent (as soon as we change the page, we go back to the theme by fault).
I therefore propose here to implement _JavaScript_ code to manage alternative themes regardless of the browser (as long as _JavaScript_ is activated), plus a concrete application for the use of [_cookies_](/2012/05/14/ cookies) for persistence of choice.

## Presentation of the _HTML_ code

A main style can be declared as follows (requires `title` attribute):
```html
<link href="main.css" title="Main" rel="stylesheet" type="text/css"/>
```

An alternate style can be declared as follows (requires `title` attribute and `rel` attribute with `alternate` value):
```html
<link href="alternative.css" title="Alternate" rel="alternate stylesheet" type="text/css"/>`
```

A permanent style (affecting whatever the current theme) can be declared as follows:
```html
<link href="permanent.css" rel="stylesheet" type="text/css"/>
```

## Management of alternative themes in _JavaScript_

Here is a function/class in _JavaScript_ to manage alternative themes (functionality present under _Firefox_). The function/class allows to retrieve the list of themes of the page, check that a theme is present and change the current theme.
The class calls the _cookies_ manipulation functions seen in a [here](/2012/05/14/cookies).

```javascript
/**
 * @brief Function to create a easy-to-use theme changer.
 * 
 * Pseudo class for handling theme changes through Javascript cookies.
 * 
 * @author madmath03
 */
function themeChanger() {
	// Theme Cookie Parameters
	this.themeCookieName = "theme";
	this.themeCookieDaysDuration = 30;
	
	this.listThemes = new Array();
	/**
	 * @brief Function to get the list of themes available in the page
	 * @return array of themes (DOM 'link' Objects)
	 */
	this.getListThemes = function() {
		this.listThemes = new Array();
		var listLinks = document.getElementsByTagName("link");
		
		for (var i = 0, n = listLinks.length ; i < n ; i++) {
			if (listLinks[i].rel && (listLinks[i].rel.indexOf("style") !== -1) && listLinks[i].title) {
				this.listThemes.push(listLinks[i]);
			}
		}
		
		return this.listThemes;
	};
	
	/**
	 * @brief Function to check if a given theme exists
	 * @param themeName name of the theme
	 * @returns true if present in the list of theme, false otherwise
	 */
	this.check = function(themeName) {
		if (themeName == null || themeName.length <= 0)
			return false;
		
		var themeExists = false;
		for (var i = 0, n = this.listThemes.length ; i < n && !themeExists ; i++) {
			if (this.listThemes[i].title) {
				if (this.listThemes[i].title === themeName)
					themeExists = true;
			}
		}
		return themeExists;
	};
	
	/**
	 * @brief Function to switch to a given theme
	 * @param themeName name of the theme
	 */
	this.change = function(themeName) {
		this.getListThemes();
		// if the theme is not set or does not exist
		if (!this.check(themeName)) {
			unsetCookie(this.themeCookieName);
			return;
		}
		
		// loop through the themes
		for (var i = 0, n = this.listThemes.length ; i < n ; i++) {
			if (this.listThemes[i].title) {
				// disable all themes
				this.listThemes[i].disabled = true;
				// enable the proper one
				if (this.listThemes[i].title === themeName)
					this.listThemes[i].disabled = false;
			}
		}
		var path = getUrlPath().split('/');
		setCookie(this.themeCookieName, themeName, this.themeCookieDaysDuration, null, '/'+path[0]+'/');
	};
	
	/**
	 * @brief Function to retrieve and switch to the current theme
	 * @returns the name of the current theme
	 */
	this.getCurrentTheme = function() {
		var themeCookie = getCookie(this.themeCookieName);
		this.change(themeCookie);
		return themeCookie;
	};
	
	// Init
	this.getCurrentTheme();
}
```

## Managing alternate themes with _jQuery_

Here is the equivalent function/class in _jQuery_:

```javascript
/**
 * Function/class to create a easy-to-use theme changer.
 * 
 * Pseudo class for handling theme changes through JavaScript cookies.
 * 
 * This advanced theme changer uses jQuery.
 * @see https://jquery.com/
 * 
 * @author madmath03
 * 
 * @param $ jQuery instance
 */
function $themeChanger($) {
	// Theme Cookie Parameters
	this.themeCookieName = "theme";
	this.themeCookieDaysDuration = 30;
	
	/**
	 * @brief Function to get the list of themes available in the page
	 * @return jQuery array of themes
	 */
	this.getListThemes = function() {
		return $('html head link[rel*="style"][title]');
	};
	
	/**
	 * @brief Function to check if a given theme exists
	 * @param themeName name of the theme
	 * @returns true if present in the list of theme, false otherwise
	 */
	this.check = function(themeName) {
		if (themeName == null || !(themeName.length > 0))
			return false;
		
		return ($('html head link[rel*="style"][title="'+themeName+'"]').length > 0);
	};

	/**
	 * @brief Function to switch to a given theme
	 * @param themeName name of the theme
	 */
	this.change = function(themeName) {
		// if the theme is set and exists
		if (this.check(themeName)) {
			// for each theme CSS
			this.getListThemes().each(function() {
				this.disabled = true;
				if (this.title === themeName)
					this.disabled = false;
			});
			var path = getUrlPath().split('/');
			setCookie(this.themeCookieName, themeName, this.themeCookieDaysDuration, null, '/'+path[0]+'/');
		}
		else
			unsetCookie(this.themeCookieName);
	};

	/**
	 * @brief Function to retrieve and switch to the current theme
	 * @returns the name of the current theme
	 */
	this.getCurrentTheme = function() {
		var themeCookie = getCookie(this.themeCookieName);
		this.change(themeCookie);
		return themeCookie;
	};
	
	// Init
	this.getCurrentTheme();
}
```
