<HTML>
<Head>
<Title> GOWA Empire -- Under construction </Title>
<link rel="stylesheet"href="style.css">
</Head>
<Body>
<!-- div that will hold our WebGL canvas -->
<div id="canvas"></div>

<!-- div used to create our plane -->
<div class="plane">

<!-- image that will be used as a texture by our plane -->
<img src="https://www.martin-laxenaire.fr/csstricks/images/second-example-texture.jpg" crossorigin="anonymous" />
</div>

<script id="plane-vs" type="x-shader/x-vertex">
#ifdef GL_ES
precision mediump float;
#endif

// those are the mandatory attributes that the lib sets
attribute vec3 aVertexPosition;
attribute vec2 aTextureCoord;

// those are mandatory uniforms that the lib sets and that contain our model view and projection matrix
uniform mat4 uMVMatrix;
uniform mat4 uPMatrix;

// our texture matrix uniform (this is the lib default name, but it could be changed)
uniform mat4 uTextureMatrix0;

// our time uniform
uniform float uTime;

// our mouse position uniform
uniform vec2 uMousePosition;

// our mouse strength
uniform float uMouseStrength;

// if you want to pass your vertex and texture coords to the fragment shader
varying vec3 vVertexPosition;
varying vec2 vTextureCoord;

void main() {
vec3 vertexPosition = aVertexPosition;

// get the distance between our vertex and the mouse position
float distanceFromMouse = distance(uMousePosition, vec2(vertexPosition.x, vertexPosition.y));

// this will define how close the ripples will be from each other. The bigger the number, the more ripples you'll get
float rippleFactor = 6.0;
// calculate our ripple effect
float rippleEffect = cos(rippleFactor * (distanceFromMouse - (uTime / 120.0)));

// calculate our distortion effect
float distortionEffect = rippleEffect * uMouseStrength;

// apply it to our vertex position
vertexPosition += distortionEffect / 30.0;

gl_Position = uPMatrix * uMVMatrix * vec4(vertexPosition, 1.0);

// varyings
// thanks to the texture matrix we will be able to calculate accurate texture coords
// so that our texture will always fit our plane without being distorted
vTextureCoord = (uTextureMatrix0 * vec4(aTextureCoord, 0.0, 1.0)).xy;
vVertexPosition = vertexPosition;
}
</script>
<script id="plane-fs" type="x-shader/x-fragment">
#ifdef GL_ES
precision mediump float;
#endif

// get our varyings
varying vec3 vVertexPosition;
varying vec2 vTextureCoord;

// our texture sampler (this is the lib default name, but it could be changed)
uniform sampler2D uSampler0;

void main() {
// get our texture coords
vec2 textureCoords = vTextureCoord;

// apply our texture
vec4 finalColor = texture2D(uSampler0, textureCoords);

// fake shadows based on vertex position along Z axis
finalColor.rgb -= clamp(-vVertexPosition.z, 0.0, 1.0);
// fake lights based on vertex position along Z axis
finalColor.rgb += clamp(vVertexPosition.z, 0.0, 1.0);

// handling premultiplied alpha (useful if we were using a png with transparency)
finalColor = vec4(finalColor.rgb * finalColor.a, finalColor.a);

gl_FragColor = finalColor;
}
</script>


<script src="https://www.curtainsjs.com/build/curtains.min.js" type="text/javascript"></script>
</HTML>