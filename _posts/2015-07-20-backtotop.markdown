---
layout: post
title: "Create a 'Back to Top' button in Webpages"
date: 2015-07-20 23:00:00
categories: [Web]
---

When people visit webpages, especially those with long articles, the function of instantly going back to page top is needed. Although the Home key in keyboard is a ready-built solution, some people don't know this function and prefer using mouse. Besides, there are machines without Home key, like Macbook. Therefore, to make a more user-friendly webpage, it is useful to put a 'back to top' button.

Just in this page you are visiting there is such one. Using CSS and jQuery, the button is designed so that it doesn't appear until the reader scrolls for some distance from the beginning of the webpage, and when the button is clicked on, the webpage automatically scrolls back to the page top. Below is my code to achieve such effect.

First, define an id for the shape and style of the button in the .css file:

    #top {
        position: fixed;
        right: 2em;
        bottom: 2em;
        width: 50px;
        height: 50px;
        background: lightblue url(top-arrow.svg) no-repeat center 50%;
        opacity: 0.8;
        display: none;
        z-index: 888;
    }

Here the position of the button is set as fixed in the screen, with distance of 2 em (length of 2 latin letters) to the right border and the bottom respectively. The `background` line includes a .svg file which defines the button shape:

    <?xml version="1.0" encoding="utf-8"?>
    <!-- Generator: Adobe Illustrator 17.1.0, SVG Export Plug-In . SVG Version: 6.00 Build 0)  -->
    <!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
    <svg version="1.1" id="Layer_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
         width="16px" height="16px" viewBox="0 0 16 16" enable-background="new 0 0 16 16" xml:space="preserve">
    <polygon fill="#FFFFFF" points="8,2.8 16,10.7 13.6,13.1 8.1,7.6 2.5,13.2 0,10.7 "/>
    </svg>


Then we can show the button in our webpage just by introducing a link in the .html file and including this id:

    <a id="top"></a>

Next, to make the button responsive to reader's operation, write functions in the .js file:

    $(document).ready(function() {
        categoryDisplay();
        generateContent();
        backToTop();
    });

    function categoryDisplay(){/*implementation hidden*/}
    function generateContent(){/*implementation hidden*/}

    function backToTop() {
        $(window).scroll(function() {
            if ($(window).scrollTop() > 100) {
                $("#top").fadeIn(500);
            } else {
                $("#top").fadeOut(500);
            }
        });
        $("#top").on('click', function(event) {
            $("body,html").animate({
                scrollTop: 0,
            });
        });
        $(function() {
            $('[data-toggle="tooltip"]').tooltip();
        });
    }

Here the `backToTop` function defines the required behaviours of the button. The first method tells the button when to appear (after scrolling for 100 pixels) and when to disappear. The second creates an event of scrolling to top when the button is clicked. The third one is not very important, just for showing tips. Note that to achieve these functions, jQuery is essential.

And that's all work required to create a back-to-top button, enjoy!

---

PS: to make the webpage scrolling to top, there is one more easy method. We know that a url with the '#' symbol in the end targets to the page top, so you can just implement the link in your .html like this:

    <a href="#" id="top"></a>

However, this results in an unsatisfactory effect. After you click on the button, actually you jump to a page with different url (since a '#' is appended):

![banner]({{ site.baseurl }}/images/beforeclick.png)
![banner]({{ site.baseurl }}/images/afterclick.png)

Although we are still in the same page, the browser doesn't think so. 
When you need to go back to the previous webpage you have visited, you have to click twice on the Back button (the first time just for jumping to the location before you click on the back-to-top button), which is kind of annoying. In contrast, using jQuery function can prevent this problem. You can try the back-to-top button in this page, it won't lead you to a different url with '#' symbol.

