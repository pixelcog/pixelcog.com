---
layout: post
title:  Convert Gmail Messages to PDF with Google Apps Script
tags:   google-apps-script javascript gmail automation productivity
image:  gmail-to-pdf.png
---

Recently I had need of a method to quickly and easily convert Gmail messages into PDFs.  In the process, I created a small library of helpful tools for use with [Google Apps Script](https://developers.google.com/apps-script/) that I believe others may benefit from.  What follows is a description of the library and some example usage.

{{ more }}

## TL;DR

I've created a Google Apps Script library to automate the heavy-lifting involved with Gmail message to PDF conversion.  If you aren't interested in the nitty gritty of what goes into the process, you can dive right into the [example project](https://script.google.com/d/1qdkT9ShXl4VWO9XvKefcxmH_oRJe31MPDyIDsOKyGidKr-GHBpULLtvx/edit?usp=sharing), fork it, and play with it yourself.  I've put the library on [GitHub](https://github.com/pixelcog/gmail-to-pdf), of course, and you can find the project IDs needed to include the library in your own project there.

## Converting Emails to PDF

![Gmail to PDF](/img/posts/gmail-to-pdf.jpg)

When I sought out solutions on Google, I was initially led to a [clever idea](http://www.oxhow.com/automatically-backup-emails-as-pdf/) which utilized [IFTTT](https://ifttt.com/) and Gmail filters to send your email to an external service like [pdfconvert.me](http://pdfconvert.me/) for conversion.  In practice, however, the IFTTT recipes ended up being inflexible and buggy.  Also from a security standpoint I really didn't like the idea of sharing my email with two new third party services, no matter how benign their privacy policies may be.

I decided to keep the process in-house, so I looked into [Google Apps Script](https://developers.google.com/apps-script/).  Google, after all, is already hosting my email so I am not involving any new parties.  If you've never used Google Apps Script before, I urge you to check it out.  It is an indispensable tool for automating processes involving any of Google's cloud services, and all that is needed is some basic familiarity with JavaScript.

Looking around, I found a [basic example](http://ctrlq.org/code/19117-save-gmail-as-pdf) [or two](http://stackoverflow.com/questions/16946168/automatically-convert-emails-with-a-gmail-label-to-pdf-and-send-it-to-an-email-a) of Gmail-to-PDF scripts, but they have a few shortcomings:

* Inline images and embeddable attachments are not included.
* Google Drive's HTML renderer does not load remote images.
* The resulting PDF uses webkit's "print" styling which removes all background styling and messes with the colors.
* Unnecessary temporary files are created, then deleted on Google Drive.

### Rendering Images

Turns out we can get around Google's unwillingness to load remote images by downloading them ourselves with `UrlFetchApp.fetch()` and embedding them directly in the HTML as [data URI strings](http://en.wikipedia.org/wiki/Data_URI_scheme).  This means turning all `<img src="http://..." />` into `<img src="data:image.jpg;base64,..." />`.  Here's a simple method to convert an image to a base64-encoded data uri:


    function renderDataUri_(image) {
      if (typeof image == 'string' && !(image = fetchRemoteFile_(image))) {
        return null;
      }
      if (image.toString() == 'Blob' || image.toString() == 'GmailAttachment') {
        var type = image.getContentType().toLowerCase();
        var data = Utilities.base64Encode(image.getBytes());
        if (type.indexOf('image') == 0) {
          return 'data:' + type + ';base64,' + data;
        }
      }
      return null;
    }

    function fetchRemoteFile_(url) {
      var response = UrlFetchApp.fetch(url, {'muteHttpExceptions': true})
      return response.getResponseCode() == 200 ? response.getBlob() : null;
    }

> **Note**: the `instanceof` keyword does not work on Google Apps Script built-in classes, so `toString()` appears to be the only way to identify them.

With the ability to generate data uris for our images, we can now find and replace all image urls within `<img>` tags, `style` attributes, and css definitions using regular expressions (thanks to [Steven Levithan's blog](http://blog.stevenlevithan.com/archives/match-quoted-string) for help with the regex):

    function embedHtmlImages_(html) {
      // process all img tags
      html = html.replace(/(<img[^>]+src=)(["'])((?:(?!\2)[^\\]|\\.)*)\2/gi, function(m, tag, q, src) {
        return tag + q + (renderDataUri_(src) || src) + q;
      });

      // process all style attributes
      html = html.replace(/(<[^>]+style=)(["'])((?:(?!\2)[^\\]|\\.)*)\2/gi, function(m, tag, q, style) {
        style = style.replace(/url\((\\?["']?)([^\)]*)\1\)/gi, function(m, q, url) {
          return 'url(' + q + (renderDataUri_(url) || url) + q + ')';
        });
      return tag + q + style + q;
      });

      // process all style tags
      html = html.replace(/(<style[^>]*>)(.*?)(?:<\/style>)/gi, function(m, tag, style, end) {
        style = style.replace(/url\((["']?)([^\)]*)\1\)/gi, function(m, q, url) {
          return 'url(' + q + (renderDataUri_(url) || url) + q + ')';
        });
        return tag + style + end;
      });
      return html;
    }


### Extracting Inline Images

For some reason, Google has not provided a straight forward way to access attached images which are embedded in a message body.  This glaring omission has found its way into [several](https://code.google.com/p/google-apps-script-issues/issues/detail?id=1659) [bug](https://code.google.com/p/google-apps-script-issues/issues/detail?id=2810) [reports](https://code.google.com/p/google-apps-script-issues/issues/detail?id=3532) but has yet to receive any attention from the powers that be.  Luckily, there is a [somewhat hacky solution](http://stackoverflow.com/a/17267965/1447303) available to us by extracting the data from the raw message body with `message.getRawContent()` (thanks to [fooby](http://stackoverflow.com/a/17267965/1447303) on stack overflow for this):

    function embedInlineImages_(html, raw) {
      var images = [];

      // locate all inline content ids
      raw.replace(/<img[^>]+src=(?:3D)?(["'])cid:((?:(?!\1)[^\\]|\\.)*)\1/gi, function(m, q, cid) {
        images.push(cid);
        return m;
      });

      // extract all inline images
      images = images.map(function(cid) {
        var cidIndex = raw.search(new RegExp("Content-ID ?:.*?" + cid, 'i'));
        if (cidIndex === -1) return null;

        var prevBoundaryIndex = raw.lastIndexOf("\r\n--", cidIndex);
        var nextBoundaryIndex = raw.indexOf("\r\n--", prevBoundaryIndex+1);
        var part = raw.substring(prevBoundaryIndex, nextBoundaryIndex);

        var encodingLine = part.match(/Content-Transfer-Encoding:.*?\r\n/i)[0];
        var encoding = encodingLine.split(":")[1].trim();
        if (encoding != "base64") return null;

        var contentTypeLine = part.match(/Content-Type:.*?\r\n/i)[0];
        var contentType = contentTypeLine.split(":")[1].split(";")[0].trim();

        var startOfBlob = part.indexOf("\r\n\r\n");
        var blobText = part.substring(startOfBlob).replace("\r\n","");

        return Utilities.newBlob(Utilities.base64Decode(blobText), contentType, cid);
      }).filter(function(i){return i});

      // process all img tags which reference "attachments"
      return html.replace(/(<img[^>]+src=)(["'])(\?view=att(?:(?!\2)[^\\]|\\.)*)\2/gi, function(m, tag, q, src) {
        return tag + q + (renderDataUri_(images.shift()) || src) + q;
      });
    }


### Some CSS Magic

If we place the line `<script>document.write(navigator.userAgent);</script>` into an html document and convert it to a PDF using Google Apps Script, you'll get the following user agent string:

    Mozilla/5.0 (en­US) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/27.0.1453 Safari/537.36 +Google­Document­Conversion

Since they are using webkit as their rendering engine, we can get away with a few tricks to assist in the PDF rendering process.

    /* prevent the background from disappearing and other odd text color issues */
    body { -webkit-print-color-adjust: exact; }

    /* add a page break before each email header, so multiple message pdfs each have their own page */
    body>.email-meta { page-break-before: always; }
    body>.email-meta:first-child { page-break-before: auto; }

I then went on to create a simple message header with CSS to display the _from:_, _to:_, _subject:_, and _date:_ meta data, and I threw in the from address's [gravatar](http://gravatar.com/) image (when available) as an extra touch:

![Message header example](/img/posts/gmail-to-pdf-header.jpg)


### Performing The Conversion

A lot of scripts floating around out there seem to think you need to create and save an HTML document to Google Drive before a conversion can be made. Actually, there is no need to involve Google Drive at all.  Google Apps Script has a special class called a ["Blob"](https://developers.google.com/apps-script/reference/base/blob) which acts as a disembodied file object on which we can perform many operations without involving a file system.  Simply invoke it like so:

    var html = Utilities.newBlob(html, 'text/html', 'email.html');
    var pdf  = file.getAs('application/pdf');

You can then send the blob as an email attachment, upload it to another service, or whatever, and you never need to request permission to access the Google Drive API or manage temporary files to do so.


## Using The Library

If you want to use this library, I've made it publicly available on [GitHub](https://github.com/pixelcog/gmail-to-pdf/) and [Google Apps Script](https://script.google.com/d/1V9HLEXHv4-7muXGhMS0XC-Lon5WX1CQhtfmjmaVp7WSHoeswwfkq1-90/edit?usp=sharing). To include it as a library in your own Google Apps Script project, go to **Resources > Libraries...** and use the project ID `MsE3tErxE9G0z6EMfGmUGqVVaKzeOjMwH`.

Simply get a GmailMessage and pass it to the `messageToHtml()` method ([source](https://github.com/pixelcog/gmail-to-pdf/blob/3560a55/GmailUtils.gs#L146)), like so:

    // convert the first message in your inbox to pdf,
    // and save it to the root directory of Google Drive
    var threads = GmailApp.search('in:inbox');
    var message = threads[0].getMessages()[0];
    var pdf = GmailUtils.messageToPdf(message);
    DriveApp.createFile(pdf);

I hope others will find this library useful.

