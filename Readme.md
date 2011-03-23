# Formaline for NodeJS 

> __formaline__ is a new ([nodejs](http://nodejs.org/)) module for handling forms ( **HTTP POST** ) and for fast parsing of file uploads, 
> it is also ready to use with [connect middleware](https://github.com/senchalabs/connect).  



 Installation
--------------
     
with npm:
    $ npm install formaline

with git:
    $ git clone git://github.com/rootslab/formaline.git

>if you want to use nodeJS, only for testing purpose, together with Apache, a simple way to do this is to enable apache *mod-proxy* and add this lines to your apache virtualhost:


    ProxyPass /test/ http://localhost:3000/test/
    ProxyPassReverse /test/ http://localhost:3000/test/


>change the path and the port with yours. 



 Features
----------

> - Real-time parsing of file uploads, also supports the "multiple" attribute, for HTML5 capable browsers .
> - It is possible to create module instances with a configuration object.
> - some useful configuration parameters ( listeners, uploadThreshold, logging .. ).
> - exceptions handling is fluid.
> - Many events for control of the module execution. 
> - Very Fast and Simple Parser (see parser-benchmarks).
> - It is possible to preserve or remove uploaded files if they are not completed, due to exceeding of the upload total threshold. 
> - It easily integrates with connect middleware.
> - Works!
> - etc..




 Simple Usage
--------------

    var formaline = require('formaline'),
        form = new formaline( { } );           // <-- empty config object
   
   *add events listener:*

    ...
    form.on( 'filereceived', function( filename, filedir, filetype, filesize, filefield ){ .. }  )  
    ...
 
  ** the listed params ( filename, filedir, .. ) are already attached to the function callback!** 

>  see below for a complete list of listeners signatures!
      
    

   *parse request:*    


    form.parse( req, res, next ); // next is a callback  function( .. ){ .. }
    

 Configuration Options
-----------------------

You could create a formaline instance with some configuration options : 

> - **'uploadRootDir'** : ( *string* ) the default root directory for files uploads is '/tmp/'.
>   - it is the root directory for file uploads, must already exist!
>   - a new sub-directory with a random name is created for every upload request.

> - **'uploadThreshold'** : ( *integer* ) default value is 1024 * 1024 * 1024 bytes (1GB).
>   - it indicates the upload threshold in bytes for file uploads (multipart/form-data) before of stopping  writing to disk,
>   - it also limits data received with serialized fields (x-www-urlencoded). 

> - **'emitDataProgress'** : ( *boolean or integer > 1* ) the default value is false.
>    - when it is true, it emits a 'dataprogress' event on every chunk. If you need to change the emitting factor ,( you could specify an integer > 1 ). 
>    - If you set it for example to  an integer k,  'dataprogress' is emitted every k data chunks received, starting from the first. ( it emits events on indexes: *1 + ( 0 * k )*, *1 + ( 1 * k )*, *1 + ( 2 * k )*, *1 + ( 3 * k )*, etc.. );  


> - **'checkContentLength'** : ( *boolean* ) the default value is false.
>   - formaline doesn't stop if ( Content-Length > uploadThreshold ), It will try to receive all data for request, and write to disk the bytes received, until it reaches the upload threshold. 
>   - if value is set to true, if  the header Content-Length exceeds uploadThreshold, It stops receiving data,

> - **'removeIncompleteFiles'** : ( *boolean* ) the default value is  true.
>   - if true, formaline auto-removes files not completed because of exceeded upload threshold limit, then it emits a 'fileremoved' event, 
>   - if false, no event is emitted, but the incomplete files list is passed to the 'end' listener in the form of an array of paths. 


> - **'logging'** : ( *string* ) the default value is 'debug:off,1:on,2:on,3:on'.
>   - it enables various logging levels, it is possible to switch on or  off one or more level at the same time. 
>   - debug: 'off' turns off logging, to see parser stats you have to enable the 2nd level.
            
> - **'listeners'** : ( *config object* ) It is possible to specify here a configuration object for listeners or adding them in normal way, with 'addListener' / 'on' . 
>    - **See below**




           
 Events & Listeners
--------

#### Type of events:
 
 

> - *'fatal' exceptions* : headersexception, pathexception, exception (the data transmission is interrupted). 
> - *informational* : filereceived, field, dataprogress, end 
> - *warning* : fileremoved, warning 

 
#### Listeners are called with following listed parameters , them are  already attached to the callbacks : 


> - **'warning'**: `function( msg ){ ... }`,
 
> - **'headersexception'**: `function ( isUpload, errmsg, res, next ) { .. }`,
 
> - **'exception'**: `function ( isUpload, errmsg, res, next ) { .. }`,
 
> - **'pathexception'**: `function ( path, errmsg, res, next ) { .. }`,
 
> - **'field'**: `function ( fname, fvalue ) { .. }`,
 
> - **'filereceived'**: `function ( filename, filedir, filetype, filesize, filefield ) { .. }`,
 
> - **'fileremoved'**: `function ( filename, filedir, filetype, filesize, filefield ) { .. }`,
 
> - **'dataprogress'**: `function ( bytesReceived, chunksReceived ) { .. }`,
 
> - **'end'**: `function ( incompleteFiles, response, next ) { .. }`
 
 



  Advanced Usage
------------------


*require the module:*


    var formaline = require('formaline');
    

*build a config object:*

    
    var config = { 
    
        uploadRootDir:    '/var/www/upload/',
            
        checkContentLength:   false,
            
        uploadThreshold:    3949000,  
          
        removeIncompleteFiles:    true,
            
        emitDataProgress:    false, 
            
        logging:    'debug:on,1:on,2:on,3:off'
            
        listeners: {
                
            'warning': function(msg){
                ...
            },
            'headersexception': function ( isUpload, errmsg, res, next ) {
                ...
                next();               
            },
            'exception': function ( isUpload, errmsg, res, next ) {
                ...
                next();
            },
            'pathexception': function ( path, errmsg, res, next ) {
                ...
                next();
            },
            'field': function ( fname, fvalue ) { 
                ...
            },
            'filereceived': function ( filename, filedir, filetype, filesize, filefield ) { 
                ... 
            },
            'fileremoved': function ( filename, filedir, filetype, filesize, filefield ) { 
                ...
            },
            'dataprogress': function ( bytesReceived, chunksReceived ) {
                ...
            },
            'end': function ( incompleteFiles, res, next ) {
                ...
                res.writeHead(200, {'content-type': 'text/plain'});
                res.end();
                //next();
            }
            
        }//end listener config
    };
        

*create an instance with config, then parse request:*
   

    new formaline( config ).parse( req, res, next );
    

  *or*


    var form = new formaline(config); 
    form.parse( req, res, next);
    
    

 **See Also :**


> - [examples](https://github.com/rootslab/formaline/tree/master/examples) 
> - [parser-benchmarks](https://github.com/rootslab/formaline/tree/master/parser-benchmarks), for parser speed tests (data-rate) 
 

 File Creation 
---------------
 
When a file is founded in the data stream:
 
 - this is directly writed to disk chunk per chunk, until end of file is reached.

 - a directory with a random integer name is created in the upload path directory (default is /tmp/), for example:  */tmp/123456789098/*,
   it assures no file name collisions for every different post.

 - when two files with the same name are uploaded through the same form post action, the file that causes the collision is renamed with a prefix equal to current time in millis; 
   >**for example**: 
   >we are uploading two files with same name, like *hello.jpg*, the first one is received and writed to disk with its original name, 
   >the second one is received but its name causes a collision, then it is also writed to disk, but with a name something like *1300465416185_hello.jpg*. 
   >It assures that the first file is not overwritten.

 - when a file reaches the max bytes allowed:
   > - if *removeIncompleteFiles === true*, the file is auto-removed and a **'fileremoved'** event is emitted; 
   > - if *removeIncompleteFiles === false*, the file is kept in the filesystem, **'end'** event is emitted and an array of  paths ( that lists incomplete files ), is passed to callback.

 - when a file is totally received, a **'filereceived'** event  is emitted. 

 - the **filereceived** and **fileremoved** events are emiited together with this params: *filename*, *filedir*, *filetype*, *filesize*, *filefield*.

 Parser
--------

###A Note about Parsing Data Rate vs Network Throughput
---------------------------------------------------------------------------------------

Overall parsing data-rate depends on many factors, it is generally possible to reach __700 MB/s and more__ ( searching a basic ~60 bytes boundary string, like Firefox uses ) with a *real* data Buffer totally loaded in RAM, but in my opinion, this parsing test emulates more a network with an high-level Throughput, than a real case. 

Unfortunately, sending data over the cloud is sometime a long-time task, the data is chunked, and the **chunk size may change because of underneath TCP flow control ( typically chunk size is ~ 8K to ~ 1024K )**. Now, the point is that the parser is called for every chunk of data received, the total delay of calling the method becomes more perceptible with a lot of chunks. 

I try to explain me:

>__In the world of Fairies, using a super-fast Booyer-Moore parser :__
 
>  - data is not chunked, 
>  - there is a low pattern repetition in data received, ( this get the result of n/m comparison )
> - network throughput == network bandwidth (wow),
 
 reaches a time complexity (in the best case) of : 

    O( ( data chunk length ) / ( pattern length ) ) * ( time to do a single comparison ) = T
      or  for simplicity  
     O( n / m ) * t = T

(for the purists, O stands for Theta). 

>__In real world, Murphy Laws assures that the best case doesn't exists:__ :O 
 
>  - data is chunked,
>  - in some cases (very large CSV file) there is a big number of char comparisons ( it decreases the parser data rate ), however, for optimism and simplicity, we use previous time result T = O( n / m ) * t. 
>  - network throughput < network bandwidth,
>  - time 't' to do a single comparison, depends on how the comparison is implemented,

 the time complexity becomes to look something like:

    ( T ) *  ( number of chunks ) * ( average number of parser calls per chunk * average delay time of a single call )  
      or
    ( T ) * ( k * d ) => ( O( n / m ) * t ) * ( c * k * d ) 

When the number k of chunks increases, the value  ( c *  k * d ) becomes to have a considerable weigth in terms of time consumption; I think it's obvious that, for the system, calling a function 10^4 times, is an heavier job than calling it only 1 time. 

`A single GB of data transferred, with a http chunk size of 40K, is typically splitted (on average) in ~ 26000 chunks!`

However, in a general case, 
 
 - we can do very little about reducing time delay of parser calls, and for reducing the number of chunks ( or manually increasing their size ), these don't totally depend on us. 
 - we could minimize the number of parser calls **'c'**, a single call for every chunk, c = 1.
 - we could minimize the time **'t'** to do a single char comparison , it obviously reduces the overall execution time.

For this reasons: 
 
 - I try to not use long *switch( .. ){ .. }* statements or a long chain of *if(..){..} else {..}*,
 - instead of building a complex state-machine, I have written a simple implementation of QuickSearch algorithm, using only high performance for-cycles,
 - for miminizing the time 't' to do a comparison, I have used two simple char lookup tables, 255 bytes long, implemented with nodeJS Buffers. (one for boundary pattern to match, one for CRLFCRLF sequence). 

The only limit in this implementation is that it doesn't support a boundary length more than 254 bytes, **for now it doesn't seem a real problem with all major browsers I have tested**, they are all using a boundary totally made of ASCII chars, typically ~60bytes in length.



 Future Releases
-----------------

 - add some other server-side security checks, and write about it . *( like weird chars in files name, name length.. )*
 - some code performance modifications in quickSearch.js and formaline.js .
 - some code variables cleaning in formaline.js .
 - change the core parser with a custom one .
 - check some weird boundary types .
 - Restify?
 - switch createDelegate to ecmascript5 bind ?
 - be happy?  

## License 

(The MIT License)

Copyright (c) 2011 Guglielmo Ferri &lt;44gatti@gmail.com&gt;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
