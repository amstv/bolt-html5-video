# Bolt HTML5 Video Extension

A Bolt Extension that allows you to play local videos either through ```/files/``` or from your own CDN. Also lets you say goodbye to heavy GIFs and allows you to use new video codec sweetness for short animations in Bolt!  
 
 For a few usage examples with videos and a gif like animation you can see [Bolt HTML5 Videos Demo](https://corydowdy.com/demo/bolt-html5-videos) on my site. 

Quick Navigation:

* [Extension Set Up](#set-up)
* [ Quick Usage With Defaults ](#quick-usage-with-defaults)
* [Using Your Own CDN](#using-your-own-cdn)
* [Adding Video Attributes](#video-attributes)
* [Controlling Number of Video Sources](#controlling-video-sources)
* [Adding Tracks and Subtitles](#adding-tracks-and-subtitles)
* [Controlling Start and End Times](#controlling-start-and-end-times)
* [Advanced Usage - over ride settings](#advanced-usage)
* [Saving Data with Client Hints ](#save-data-option-and-client-hints)
* [Uploading Video Files](#uploading-video-files)
* [Using Both ``file`` & ``filelist`` types](#using-both-file-and-filelist-types)

Has current support for:

  * HTML5 Video Attributes
    * autoplay - please don't use this for videos :)
    * controls
    * muted
    * loop
    * playsinline - this is used particularly for ios 10 
  * preload options - metadata | auto | none
  * video posters
  * video width and height
  * custom classes and id's
  * Tracks & subtitles
  * Media Fragments (currently only 't' or the time)
  * single or multiple video sources
  
  
## NOTICE:  
a recent change to Bolt may strip certain tags from the rendered output. You'll need to add these to your Bolt Config. [Here is the relevant section in the config](https://github.com/bolt/bolt/blob/868e36f2961a98745131c1f0b2b13f711deb6345/app/config/config.yml.dist#L221-L223)  

```html 
htmlcleaner:
  allowed_tags: [video, source, track, ... other tags ]
```   

You'll also need to allow these attributes so the too will not be removed:    

```html 
htmlcleaner:
  allowed_tags: [video, source, track, ... other tags ]  
  allowed_attributes: [ preload, controls, muted, autoplay, playsinline, loop, poster, type, label, kind, srclang, .. other attributes here  ]
``` 

## Set Up

To start off you'll need to have a field in your contenttypes that accepts/uses video. There are two types right off the bat you can use. The``file`` type and/or ``filelist`` type.

**If You Plan To Upload Videos through the 'Edit' Option of the Backend With 'Upload File' You'll Need To Use the ``filelist`` Type**. This is because Bolt's backend will place the video in a directory. This extension assumes each file will be named the same. So an MP4 file will have the same name and file path as a Webm or OGG.

Regular File upload through 'upload files'

 ```yaml
# your contenttypes.yml file
entries:
  name: Entries
  singular_name: Entry
  fields:
    # other fields here
    video: # our video field to call in the template
        type: file # the type of field we are using.
```

Uploading Files through the record's edit screen:

 ```yaml
# your contenttypes.yml file
entries:
  name: Entries
  singular_name: Entry
  fields:
    # other fields here
    video:
        type: filelist
```



If you're not using the default settings then create a new settings group in the extensions config file.

```yaml
# extensions config file
blogVideos: # the name of our settings group!
```

Then follow the same structure in the *default* settings. Any setting left out of your new settings group will fall back to whatever is set in ``default``.


```yaml
blogVideos:
  use_cdn: false
  save_data: false
  video_poster: 'path/to/poster.png'
  attributes: [ 'controls', 'muted' ]
  preload: 'metadata'
  width_height: [ 400, 400 ]
  multiple_source: true
  video_types: [ 'webm', 'mp4' ]
```

Now in your template ( example: record.twig ) place this tag with your named settings group wherever you want a video!

```twig
{{ html5video(record.video, 'blogVideos' ) }}
```
or for the filelist type:

```twig
{{ html5video(record.video.0, 'blogVideos' ) }}
```



## Quick Usage With Defaults

Placing this twig tag in your template:

```twig
{{ html5video(record.video) }}
```
Will use the defaults found in the extensions config file

```yaml
default:
  use_cdn: false
  save_data: false
  attributes: [ 'controls']
  preload: 'metadata'
  multiple_source: true
  video_types: [ 'webm', 'mp4' ]
```

and produce a video tag in your template.

```html
<video controls preload="metadata">
  <source src="/files/your-video.webm" type="video/webm" >
  <source src="/files/your-video.mp4" type="video/mp4" >
</video>
```

## Using Your Own CDN

There are two (2) ways to use a CDN. The first is to add your CDN URL to the extensions config setting of ``cdn_url``

```yaml
cdn_url: https://your-cdn.com/path/to/files/
```

Then in your templates where you want the video use the file name:

```twig
{{ html5video( 'your-file.webm' ) }}
```

This will produce in the rendered HTML

```html
<video controls preload="metadata">
  <source src="https://your-cdn.com/path/to/files/your-file.webm" type="video/webm" >
  <source src="https://your-cdn.com/path/to/files/your-file.mp4" type="video/mp4" >
</video>
```

The second way to do this is leave the ``cdn_url`` setting empty and place the __full URL__ to your video in the tag:

```twig
{{ html5video( 'https://your-cdn.com/path/to/videos/second-cdn-example.webm' ) }}
```

This will produce in the rendered HTML

```html
<video controls preload="metadata">
  <source src="https://your-cdn.com/path/to/videos/second-cdn-example.webm" type="video/webm" >
  <source src="https://your-cdn.com/path/to/videos/second-cdn-example.mp4" type="video/mp4" >
</video>
```  

## Video Attributes {#video-attributes}
Video attributes are boolean. If you want them you add them to the config array. Options are:  

* autoplay - immediately play the video as soon as it can  
* controls - give the user playback controls ie: play, pause, seek 
* muted    
* loop - continuously loop the video. Much like gif's. 

And since iOS is a snowflake and all snowflakes are unique :) you can also add:  

* playsinline  

which is telling iOS Safari 10+ that you want to play the video "inline" in the document by default, within the dimensions set by the video element, instead of being displayed fullscreen or in an independent resizable window.  

example with all Attributes:  

```yaml
blogVideos:
  attributes: [ 'autoplay', 'controls', 'muted', 'loop', 'playsinline'  ]
```  

HTML output:  

```html
<video autoplay controls muted loop playsinline>
  <source src="/files/your=file.webm" type="video/webm" >
  <source src="/files/your=file.mp4" type="video/mp4" >
</video>
```  

## Adding Width's and Heights  
When adding width and heights the format is as follows:  

```yaml
blogVideos:
  width_height: [ your-width, your-height ]
```  

If you want to use a percentage: ie 100%, you **MUST** put it in quotes:  

```yaml
blogVideos:
  width_height: [ 400, '100%' ]
```  

This will produce in the rendered HTML

```html
<video width="400" height="100%">
  <source src="/files/your=file.webm" type="video/webm" >
  <source src="/files/your=file.mp4" type="video/mp4" >
</video>
```  



## Controlling Video Sources

When the config has ``multiple_source`` set to *true* as in the default settings this extension will use the file types specified in ``video_types``. Typically these are [MP4 - caniuse.com](http://caniuse.com/#search=mp4) and [WEBM - caniuse.com](http://caniuse.com/#search=webm) types. This will serve two (2) files.

```yaml
# The config file
default:
  use_cdn: false
  save_data: false
  attributes: [ 'controls']
  preload: 'metadata'
  multiple_source: true
  video_types: [ 'webm', 'mp4' ]
```

```html
<video controls preload="metadata">
  <source src="/files/your-video.webm" type="video/webm" >
  <source src="/files/your-video.mp4" type="video/mp4" >
</video>
```

If you would prefer to only serve one (1) file set ``multiple_source`` to false and then pass either a CDN url or the video attached to the record you want. The file that will be served is whatever you pass in the template or the one you've uploaded to files or your record.

One File with a record's video:

```twig
{# Your Twig Template ie. 'record.twig` #}
{{ html5video(record.video ) }}
```
```html
<!-- The Rendered HTML in your page -->
<video controls preload="metadata" src="/files/your-video.mp4"></video>
```

One file with your own CDN:

```twig
{# Your Twig Template ie. 'record.twig` #}
{{ html5video('https://your-cdn.com/videos/your-video.webm') }}
```
```html
<!-- The Rendered HTML in your page -->
<video controls preload="metadata" src="https://your-cdn.com/videos/your-video.webm"></video>
```


## Adding Tracks and Subtitles

You add subtitles and tracks to your videos by adding a section to your named config titled *tracks*


```yaml
blogVideos:
  # other config settings here
  tracks:
```

It is recommended to prefix each subsection with the language of the particular subtitle or captions. Each of these subsections will need to have

* kind: what kind of track are you providing? Values to place here are 'subtitles' or 'captions'
* srclang: What is the source language of the provided file.
* label: For English language subtitles or captions give it a descriptive label like - 'English Subtitles'
* src: the path to the file. if the provided file is in your theme directory give that path, ie: '/theme/base-2016/your-video-subtitles.vtt'
* default: if this is the default file mark this as true. Otherwise leave it out of the section.

Here is a completed example of multiple language subtitles and English Captions:

```yaml
blogVideos:
  # other settings here
  tracks:
   en_subtitles:
     kind: 'subtitles'
     srclang: 'en'
     label: 'English subtitles'
     src: '/theme/base-2016/your-video-subtitles.vtt'
     default: true
   es_subtitles:
      kind: 'subtitles'
      srclang: 'es'
      label: 'Español subtitles'
      src: '/theme/base-2016/your-video-subtitles-es.vtt'
   en_captions:
      kind: 'captions'
      srclang: 'en'
      label: 'English Captions'
      src: '/theme/base-2016/captions.vtt'
```

This will give you in your rendered page:

```html
<video controls preload="metadata" poster="/path/to/poser.png" width="400" height="400">
  <source src="/files/your-video.webm" type="video/webm" >
  <source src="/files/your-video.mp4" type="video/mp4" >
  <track label="English subtitles" kind="subtitles" srclang="en" src="/theme/base-2016/your-video-subtitles.vtt"  default >
  <track label="Español subtitles" kind="subtitles" srclang="es" src="/theme/base-2016/your-video-subtitles-es.vtt" >
  <track label="English Captions" kind="captions" srclang="en" src="/theme/base-2016/captions.vtt" >
</video>
```

For a deeper dive into WebVTT have a look at the following links:

* [HTML5 Doctor: Video Subtitling and WebVTT](http://html5doctor.com/video-subtitling-and-webvtt/)
* [Mozilla Developer Network: Introduction to WebVTT](https://developer.mozilla.org/en-US/docs/Web/API/Web_Video_Text_Tracks_Format)  

Also see the Note on [VTT Mime Type Console Warnings](#gotchas-and-fyis)

## Controlling Start and End Times

Through the ``media_fragment: [ ]`` config setting you can start the video or end the video at a specific time.

The example below starts the video at five seconds and will stop playing at twelve seconds.

```yaml
blogVideos:
  #other settings here
  media_fragment: [ 5, 12 ]
```  
 
```html
<video controls preload="metadata">
  <source src="/files/your-video.webm#t=5,12" type="video/webm" >
  <source src="/files/your-video.mp4#t=5,12" type="video/mp4" >  
</video>
```

For more examples on this see [Mozilla Developer Network - HTML5 Video Specifying Playback Range ](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Using_HTML5_audio_and_video#Specifying_playback_range)

## Advanced Usage
You can over-ride default config settings on a per usage basis in your templates.

Settings that can be currently over-ridden are:

* video_poster
* use_cdn
* class
* preload
* width_height
* media_fragment
* multiple_source

To over-ride these in a template place the tag in the template along with the named config:

```twig
{{ html5video( record.video, 'blogVideos' ) }}
```

After your named config place a comma then your custom config setting(s).

```twig
{{ html5video( record.video, 'blogVideos', { preload: 'auto' } )  }}
```

or to add an ID for a JavaScript hook:

```twig
{{ html5video( record.video, 'blogVideos', { video_id: 'js-id' } ) }}
```


For multiple over-rides I would suggest placing them on a new line like so:

```twig
{{ html5video( record.video, 'blogVideos', {
  video_poster: 'path/to/custom/poster.png',
  use_cdn: true,
  preload: 'auto',
  width_height: [ 600, 400 ],
  media_fragment: [ 0, 60 ],
  multiple_source: false
  } )
}}
```  

## Save Data Option and Client Hints  

With the ``save_data: true`` option set in the Extensions config there will be a check for the Client-Hint header of 'Save-Data: On'.  
Currently these browsers will advertise this header:  

* Chrome
* Opera 
* Yandex  

For more information on how a user can enable data-saving in Chrome for Android or iOS, or through desktop Chrome see [Reduce Data Usage in Chrome](https://developer.chrome.com/multidevice/data-compression)

With this option set a message and a button will be rendered instead of downloading and rendering the video. You can control the message that is delivered to the user, CSS styles for both the message, poster-image placeholder and/or button.  
 
```yaml  
save_data_options:  
  message: 'The Save Data Header is Present. To Play and Load the Video click the image or button below.'
  message_class: [ 'your-paragraph', 'classes' ]  
  use_poster: true
  img_placeholder_class: []
  button_class: [ 'button' , 'primary' ]
  wrapping_div: true
  wrapping_div_class: [ 'class', 'that-wraps', 'the-paragraph-and-button' ]  
```  

With these set here is how it will look instead of a video.  

```html  
<div class="class that-wraps the-paragraph-and-button">  
  <p class="your-paragraph classes">
   The Save Data Header is Present. To Play and Load the Video click the button below.
  </p>  
  <!-- if using a poster placeholder an image will be used -->
  <img src="poster-img.png" 
        class=" whatever classes are in img_placeholder_class" 
        alt="click to load video"
        width="video-width"
        height="video-height" 
        <!-- video data here --> />
  <!-- if no poster a button will be used -->
  <button class="button primary"  
        <!-- video data here --> >Load Video</button>  
</div>        
```  
Here is how it would look on the page and clicking through the buttons.  

![Save Data Enabled and Clicking load video button to load the video](https://raw.githubusercontent.com/cdowdy/bolt-html5-video/master/screenshots/save-data-enabled-placehold.gif)

## Uploading Video Files

**Uploading through Bolt's Admin backend**:

1). Under the ``Settings`` navigation area click ``File Management > Uploaded Files ``

2). Either create a folder for your videos or choose ``select file``
  * When uploading choose all the video types you would like to serve. That is webm, mp4 or ogg.

3). Click upload file.

Uploading your files this way will allow you to use ``record.video`` portion in your twig tag.

```twig
{{ html5video( record.video) }}
```

**Uploading through a record / contenttypes creation or edit page**:

1). You'll need to use ``filelist`` in your contenttypes video field.

 ```yaml
# your contenttypes.yml file
entries:
  name: Entries
  singular_name: Entry
  fields:
    # other fields here
    videolist:
        type: filelist
```

2). When Editing your contenttype choose the field you're using for videos ( this example above is 'videolist' ) and select 'upload file'

3). Chose all the filetypes you wish to serve for that particular record, ie webm, mp4 or ogg and upload those video files.

4). In your template you can now use:

```twig
{{ html5video( record.videolist.0 ) }}
```

## Using Both file and filelist types:

Structure your contenttype to use both ``file`` and ``filelist``

```yaml
# your contenttypes.yml file
entries:
  name: Entries
  singular_name: Entry
  fields:
    video:
      type: file
    videolist:
      type: filelist
```

Then in your template use an ``if/else`` statement similar to the one below:

```twig
{% if record.videolist %}
  {{ html5video( record.videolist.0, 'blogVideos') }}
{% endif %}
{% if record.video %}
  {{ html5video( record.video, 'blogVideos' ) }}
{% endif %}

{# of you could use one like this #}
{% if record.videolist %}
  {{ html5video( record.videolist.0, 'blogVideos') }}
{% else %}
  {{ html5video( record.video, 'blogVideos' ) }}
{% endif %}
```



<!--## Config Settings:-->

<!--__cdn_url__:-->
<!--set CDN url if its set and the config option of "use_cdn" is true then use this path for the video.-->
<!--example:-->

<!--```yaml-->
<!--cdn_url: 'https://awesome-cdn.com/path/to/videos/'-->
<!--```-->

<!--__use_cdn__:-->
<!--If you want to use a CDN. Defaults to false - or no you don't want to.-->

<!--```yaml-->
<!--use_cdn:  [true | false ]-->
<!--```-->

<!--If the above two are set then you can use just the filename in your templates to get the full path to the video. If there is no ``cdn_url`` set and ``use_cdn`` is true then you need to put the full url in your template.-->
<!--EX:-->

<!--```twig-->
<!--{{ html5video( 'https://your-cdn.com/path/to/videos/example.webm' ) }}-->
<!--```-->

## Gotchas and FYI'S

1. The video files should be the same name. When using multiple sources the filename will be appended with the types you provide in ``video_types``
2. If uploading through the contenttypes record edit/creation page you should use the filelist type instead of the file type  

3. If you get a Console Error of ``Resource interpreted as TextTrack but transferred with MIME type text/plain:`` when including text tracks (vtt files) you need to add the correct mime type for your server:  
  * Apache:  
   ```  
   <IfModule mod_mime.c>
     AddType text/vtt    vtt
   </IfModule>  
   ``` 
   * NGINX:  
   ```  
   types {
   #others here
       text/vtt                              vtt;
   }  
   ```  
   
4. When using a CDN and if your site is using HTTPS and you have have ``enforce_ssl: true`` in your Bolt main config file then this extension will prefix your CDN url with HTTPS to prevent Mixed Content which most browsers now will block. This may cause issues if your CDN doesn't offer HTTPS or you haven't enabled it through your CDN. If this does cause to many issues please file a bug and I'll think about removing this portion of the extension if the reasons to do so are valid and extensive enough. 