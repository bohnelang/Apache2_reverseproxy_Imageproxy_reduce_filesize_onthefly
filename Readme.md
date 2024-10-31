# On-the-fly image converting, file size reduction and caching by apache2 

By reducing image quality one can reduce image size enormously. When using a content management system there is 
the disadventage that users often can upload images from camera or smartphone with original size directly.  
Thus for a 1024x800 pixel image on the webpage a 9MB image is upload without resizing or other image processing.

To save mobile bandwith you can reduce image file size by shrinking image quality. Normally users do not notice a reduced image quality. 

For this porpos you can use the imageproxy https://github.com/willnorris/imageproxy (written in go) that runs as a standalone application on linux. The imageproxy is cabable of interacting with apache2 by url encoded paramters.

The imageproxy runs on localhost and takes its paramter by url: imageproxy URLs are of the form 
```
http://localhost/{options}/{remote_url}'
```

The imageproxy cames as a linux service:
```
# service  imageproxy status
# service  imageproxy stop
# service  imageproxy start
# /usr/local/go/src/willnorris.com/go/imageproxy/etc/imageproxy.service
```

The workflow to control image size is defined by some apache2 rewrite rule:
```
If a image ends with the suffix .jpg, .gif or .png  
AND the request is not from localhost or webserver's IP 
AND the webbrowser is mobile (or a maybe a spider)
THEN the request is redirected to imageproxy

ELSE the image is on the server and pass through to the imageproxy  
```

```
# For requests from mobile devices:
RewriteCond %{REQUEST_URI}      ^(.*)\.(jpg|gif|png)$   [NC]
RewriteCond %{REMOTE_ADDR}      !^147\.142\.106\.210$
RewriteCond %{REMOTE_ADDR}      !^127\.0\.0\.1$
RewriteCond %{HTTP_USER_AGENT} (android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge\ |maemo|midp|mmp|mobile.+firefox|netfront|opera\ m(ob|in)i|palm(\ os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows\ ce|xda|xiino                             [NC]
RewriteRule ^(.*)$  http://localhost:4593/q50/https://www.mycompany.test%{REQUEST_URI} [P,L]
```

Theses are the workflow steps:
1. The user asks for an image from web server - e.g.: GET 'https://www.mycompany.test/images/hello.jpg' (IP from outside)
   The rule set notice that the request is from outside and redirect this request to imageproxy with setting quality to 50%: 'http://localhost:4593/q50/https://www.mycompany.test/images/hello.jpg'
2. Imageproxy takes URL from paramater list and request image again: 'https://www.mycompany.test/images/hello.jpg' (from localhost). Now the request is passed to the web server directory.

3. Imageproxy gets image (step 2), converts it and answers the processed image to the user (step 2). 

**Warning**: Rendering the images take some time on the first time of the image access. Thus test the load of your system before going live. 