---
layout: post
title:  "TIL: Alternative to CSRF defense: SameSite cookies"
date:   2017-05-28 18:50:33 +0300
categories: TIL
---

Yesterday I saw [a talk](https://www.youtube.com/watch?v=wAS2M_VrigQ) about the security practices used in Angular2. One thing that was completely new to me is an alternative method of defense against [CSRF attacks](https://en.wikipedia.org/wiki/Cross-site_request_forgery): `SameSite` attribute for a cookie.

This new attribute supports two modes:
* `strict` - the browser will never attach the cookie to a cross-site requests
  * this will stop all CSRF attacks
* `lax` - the cookie will be present on save top-level navigations
  * e.g: the cookie will be sent on a `GET` request 	that	results	in	a	navigation	of	the	context
  * this will stop most CSRF attacks: unless	the	attack	can	be	launched	with	a	GET	request

The default setting for the `SameSite` attribute is `strict` mode.

`Set-Cookie: SSID=1234; SameSite=Strict`

This attribute is [currently supported](http://caniuse.com/#search=samesite) by Chrome and Opera, but there is no reason to not use it since other browsers will just ignore unknown attributes and you will be protected on Chrome and Opera, and of course on other browsers when they'll support `SameSite` attribute.
