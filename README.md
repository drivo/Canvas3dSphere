## How to make a 3D sphere with HTML5 Canvas

A simple project which shows how to use the HTML5 Canvas element and its API. You'll find the JavaScript code inside "sphere.html" file.

## Usage

Open HTML file with a compatible browser like Firefox 4, Chrome or Safari.


## Detailed information

One of the most interesting news is the introduction of the HTML5 Canvas element. Originally introduced by Apple in its engine Webkit, the canvas allow us to draw 2D (and 3D) in a specific region of HTML with JavaScript programming language.

The canvas element is defined as follows:

<canvas id="sphere3d" width="100" height="100">
    Text to alert visitors that the Canvas
    is not supported by your browser.
</canvas>

Where width and height are respectively the area of the canvas. The browser will place the rendering area in the exact spot where the item is defined in the HTML page.

In JavaScript, to obtain the rendering context (Rendering Context) you should do this:
var canvas = document.getElementById('sphere3d');
               
if(canvas.getContext){
  // 2D Context found
  var ctx = canvas.getContext('2d');
} else {
  // 2D Context not found
}

The rendering context gives us access to all the APIs for drawing:

- Drawing shapes
- Drawing pictures
- Apply the design styles and gradients
- Make changes to translation, rotation and scale.
- Effects of composition and clipping

In summary, the skeleton of a page that defines a HTML5 canvas could be summarized as follows:
``` html
<html>
  <head>
    <script type="text/javascript">
      function init() {

      var canvas = document.getElementById('sphere3d');

      if (canvas.getContext) {
        var ctx = canvas.getContext('2d');
 
         // Here the implementation
         // ...
       }
     }
    </script>
 </head>
 
 <body onload="init();">
   <canvas id="sphere3d" width="500" height="500"></canvas>
 </body>

</html>
```

Riding the excitement of discovering these APIs, I thought there was nothing better than to bring one of my old application written in C and Assembler in HTML5. A demo written according to the canons of the old school of 3D. A jump long over 15 years that I thought I did well to understand what has changed in so long.
Let’s see what came out!

First of all, I defined the classic primitive functions to handle 3D objects:

``` html
function Point3D() {
  this.x = 0;
  this.y = 0;
  this.z = 0;
}

function Sphere3D(radius) {
 this.point = new Array();
 this.color = "rgb(100,0,255)"
 this.radius = (typeof(radius) == "undefined") ? 20.0 : radius;
 this.radius = (typeof(radius) != "number") ? 20.0 : radius;
 this.numberOfVertexes = 0;

 // Loop from 0 to 360 degrees with a pitch of 10 degrees ...
  for(alpha = 0; alpha <= 6.28; alpha += 0.17) {
   p = this.point[this.numberOfVertexes] = new Point3D();
     
   p.x = Math.cos(alpha) * this.radius;
   p.y = 0;
   p.z = Math.sin(alpha) * this.radius;

   this.numberOfVertexes++;
 }

 // Loop from 0 to 90 degrees with a pitch of 10 degrees ...
 // (direction = 1)
 
 // Loop from 0 to 90 degrees with a pitch of 10 degrees ...
 // (direction = -1)

 for(var direction = 1; direction >= -1; direction -= 2) {
   for(var beta = 0.17; beta < 1.445; beta += 0.17) {
       
     var radius = Math.cos(beta) * this.radius;
     var fixedY = Math.sin(beta) * this.radius * direction;
     
     for(var alpha = 0; alpha < 6.28; alpha += 0.17) {
       p = this.point[this.numberOfVertexes] = new Point3D();

       p.x = Math.cos(alpha) * radius;
       p.y = fixedY;
       p.z = Math.sin(alpha) * radius;

       this.numberOfVertexes++;
     }
   }
 }
}
```

The Point3D class defines a 3D point. The function Sphere3D with simple calculations defines the 3D points in the space of a sphere. There are no limits to your imagination to build other nice things with the functions of sine and cosine.
The 3D engine

This piece is truly “old school”. Thinking of today’s engines, OpenGL based, this might seem awkward. Our engine, despite appearances, has only four functions:

``` html
function rotateX(point, radians) {
   var y = point.y;
   point.y = (y * Math.cos(radians)) + (point.z * Math.sin(radians) * -1.0);
   point.z = (y * Math.sin(radians)) + (point.z * Math.cos(radians));
}

function rotateY(point, radians) {
   var x = point.x;
   point.x = (x * Math.cos(radians)) + (point.z * Math.sin(radians) * -1.0);
   point.z = (x * Math.sin(radians)) + (point.z * Math.cos(radians));
}

function rotateZ(point, radians) {
   var x = point.x;
   point.x = (x * Math.cos(radians)) + (point.y * Math.sin(radians) * -1.0);
   point.y = (x * Math.sin(radians)) + (point.y * Math.cos(radians));
}

function projection(xy, z, xyOffset, zOffset, distance) {
   return ((distance * xy) / (z - zOffset)) + xyOffset;
}
```

In practice we have functions to rotate a single point (implementation of the identity matrix) and a function that gives us a 2D projection of our 3D point.
Animation

In practice, we ask the browser to call a function function init() at the end of HTML5 body load. Inside the function init() we set a timer with a rate of 30fps, and we tell him to call the render() each time he needs to perform the triggering.

``` html
function init(){
   // Set framerate to 30 fps
   setInterval(render, 1000/30);
}
...
...
<body onload="init();">
Il render loop
```

This is the heart of a classical 3D simple application

``` html
function render() {
...
   for(i = 0; i < sphere.numberOfVertexes; i++) {
   
     p.x = sphere.point[i].x;
     p.y = sphere.point[i].y;
     p.z = sphere.point[i].z;

     rotateX(p, rotation);
     rotateY(p, rotation);
     rotateZ(p, rotation);

     x = projection(p.x, p.z, width/2.0, 100.0, distance);
     y = projection(p.y, p.z, height/2.0, 100.0, distance);

     if((x >= 0) && (x < width)) {
       if((y >= 0) && (y < height)) {
         if(p.z < 0) {
           drawPoint(ctx, x, y, 4, "rgba(200,200,200,0.6)");
         } else {
           drawPointWithGradient(ctx, x, y, 10, "rgb(0,200,0)", 0.8);
         }
       }
     }                    
   }
...
}
```

For each step we apply a rotation Sphere3D by “r”, we compute the 2D projection, then we draw the point on the rendering area. Depending on the position on the Z (depth), we decide whether to render a colored dot (with a smart gradient) or not.
Drawing on rending context

All drawing functions make use of our demo function save() and restore(). These functions save and restore the graphics context which contains the following elements:

- Transformations applied
- Styles applied
- Areas of clipping

Using these functions, the drawing method (like drawPoint()) won’t alter the state of the rending context.

``` html
function drawPoint(ctx, x, y, size, color) {
  ctx.save();
  ctx.beginPath();
  ctx.fillStyle = color;
  ctx.arc(x, y, size, 0, 2*Math.PI, true);
  ctx.fill();
  ctx.restore();
}
```

To draw a point (a colored circle) the area we use the function beginPath(), arc() and fill().
3D sphere is Ready!

Simple routine, synthetic programming, for a simple result.

