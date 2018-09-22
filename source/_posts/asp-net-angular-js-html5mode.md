---
title: 'ASP.NET, Angular.js & html5mode'
tags:
  - angular.js
  - asp.net
  - iis
  - routing
url: 3234.html
id: 3234
categories:
  - ASP.NET MVC
date: 2015-09-24 07:30:00
---

I’ve been looking at Angular.js recently.  I’ve already got enough of a project done in [MongoDB](//www.mongodb.org/) (with [Mongoose](//mongoosejs.com/)), [Express](//expressjs.com/), [Angular](//angularjs.org/) and [Node.js](//nodejs.org/) (MEAN) to be comfortable with how Angular works.  But I wanted to give it a try using ASP.NET as the back end.  I’m always learning.  Always improving. To start out, I just setup an index.html page to hold my basic form as I got the basic look and feel going.  But as I progressed, I wanted to make sure I progressed, I wanted to add in the capability of using Angular’s html5mode for the client side routing. For those of you who are new to Angular, Angular is a client side JavaScript framework that will allow you to create web applications where much of the processing happens on the client side instead of the server side.  That’s cool enough.  But it also adds the ability to handle client side routing, just like ASP.NET’s MVC handles server side routing.  This allows you to have a client side “master page” that can suck in the differences from the server as it needs them based on the url that is in the address bar.  In fact, there is an extension that will let you have sub routes as far down as you need. Out of the box, Angular, and most other frameworks that implement client side routing, using the hash symbol to specify the route.  For example

http:/index.html#/pathToRoute

This allows the routing to work on older browsers. ASP.NET, Angular.js & html5mode ![image](/uploads/2015/09/image3.png "image") ASP.NET, Angular.js & html5modeBut if you are working with newer browsers that support HTML5, you can avoid the hash tag and just create a route that looks like this:

http:/pathToRoute

Which you have to admit, looks a whole lot nicer. But here  is where the problems start. As soon as you implement html5mode on a site that is hosted in IIS or IIS express, you will get a 404 error because your initial request to the server is going to ask the server for a path that doesn’t exist. There are a few ways that you can take care of this.

Return a default view for every undefined server route.
-------------------------------------------------------

One of the first suggestions you are likely to find suggest creating a default view for all routes that start with “/angular/”. This is a great start.  But here are my issues with it.  If I really want to use Angular the way it was intended to be used, I would prefer to not have to use MVC on the server side at all.  While not a huge hit, writing a razor page just to get my initial angular page up seems to be a bit of overkill.  There must be a way to do this without creating a *.chshtml file.  I also don’t want to have a sub directory for my page.  Why can’t I just go to [http://blog.dmbcllc.com](//blog.dmbcllc.com) as my default route?  And why can’t I just return a plain old html file?! Well, it turns out you can.  A slight modification of the “Return a default view” method is to have your controller return your html page.

Return an HTML page direct from the controller.
-----------------------------------------------

If you dig a bit further, you’ll find that someone else has realized that you can just return your HTML directly from the controller.  The magic to this trick is all similar to what the guy in the original article did except for in the controller, instead of returning the view, he returns the html file that contains the main html.

public ActionResult Index()
{
    return File("~/yourstartpage.html", "text/html");
}

And his main MVC route looks like this:

routes.MapRoute(
      name: "Default",
      url: "{*.}",
      defaults: new
      {
        controller = "Home",
        action = "Index",
      }
  );

This implementation has the added benefit that I’m not tied to a specific sub directory because it just says, “Any URL that doesn’t have a real file behind it should resolve to this default route.” Of course, you may be thinking, but what about the WEB API route, or any other routes I want in my system.  Well, just make sure this route comes first and you have other routes to cover the real routes you want to be able to support. Now, this gets past the objection I had with the first solution.  I no longer have to have a route.  But, why should I need to call the controller?  This is just a static HTML file we are talking about.  I should be able to by pass ASP.NET handling this file and just have IIS serve it up directly to me.

Use the URL Rewrite Module
--------------------------

A little deeper digging on the search engines reminded me that  I could just setup the [URL Rewrite module](//www.iis.net/downloads/microsoft/url-rewrite) to return my main HTML page when no real page is available.  BTW, URL Rewrite is built into IIS Express, so it should work in your development environment if you are using IIS Express as well as under IIS with the module installed. The main step to getting this working is to add the following XML to your Web.config file:

<system.webServer>
    <rewrite>
        <rules>
            <rule name="angularjs routes"
                    stopProcessing="true">
                <match url=".*" />
                <conditions logicalGrouping="MatchAll">
                    <add input="{REQUEST_FILENAME}" 
                        matchType="IsFile" negate="true" />
                    <add input="{REQUEST_FILENAME}" 
                        matchType="IsDirectory" negate="true" />
                    <add input="{REQUEST_URI}" 
                        pattern="^/(api)" negate="true" />
                </conditions>
                <action type="Rewrite" url="/" />
            </rule>
        </rules>
    </rewrite>
</system.webServer>

You should already have a system.webServer section in your web.config file, so you just need the rewrite rule inside of it. Basically what this rule does is that it says, “If you can’t find the file, and the path you are looking for is not a subdirectory of the “api” directory, return the default file at the root.”  The part about the API directory allows your WEB API stuff to continue working. The only other thing you will need to do, which isn’t unique to ASP.NET or MVC, is that you will need to remember to add the base tag to the HEAD section of your HTML file.

<base href="/"/>

And all of your client side routing with HTML5 issues should be solved. Notice that no ASP.NET code has to run to get this working.  In fact, the only time you’ll need to run ASP.NET is to call the server for data.