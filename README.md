<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">

<title>Hand Neon square</title>

<style>
*{
    margin:0;
    padding:0;
    box-sizing:border-box;
}

body{
    overflow:hidden;
    background:#000;
}

canvas{
    position:fixed;
    inset:0;
}

#camera{
    position:fixed;
    right:10px;
    bottom:10px;
    width:100px;
    height:130px;
    object-fit:cover;
    border:1px solid #00eaff;
    border-radius:10px;
    transform:scaleX(-1);
    opacity:.65;
    z-index:5;
}

#status{
    position:fixed;
    top:15px;
    width:100%;
    text-align:center;
    color:#00eaff;
    font:16px Arial;
    z-index:10;
    text-shadow:0 0 8px #00eaff;
}
</style>
</head>

<body>

<div id="status">✋ Show Your Hand</div>

<video id="camera"
       autoplay
       playsinline
       muted></video>

<canvas id="canvas"></canvas>

<script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js"></script>

<script>

const canvas=document.getElementById("canvas");
const ctx=canvas.getContext("2d");

const video=document.getElementById("camera");
const status=document.getElementById("status");

let W=innerWidth;
let H=innerHeight;

canvas.width=W;
canvas.height=H;


/* =========================
   NEON SQUARE
========================= */

let square={
    x:W/2,
    y:H/2,
    size:100,
    targetX:W/2,
    targetY:H/2
};


/* =========================
   RESIZE
========================= */

addEventListener("resize",()=>{

    W=innerWidth;
    H=innerHeight;

    canvas.width=W;
    canvas.height=H;

});


/* =========================
   UPDATE
========================= */

function update(){

    /*
       Smooth movement
    */

    square.x +=
        (square.targetX-square.x)*0.15;

    square.y +=
        (square.targetY-square.y)*0.15;

}


/* =========================
   DRAW NEON SQUARE
========================= */

function draw(){

    ctx.fillStyle="rgba(0,0,0,.20)";
    ctx.fillRect(0,0,W,H);

    const x=square.x;
    const y=square.y;
    const s=square.size;

    /*
       Glow
    */

    ctx.shadowColor="#00eaff";
    ctx.shadowBlur=30;

    ctx.strokeStyle="#00eaff";
    ctx.lineWidth=4;

    ctx.strokeRect(
        x-s/2,
        y-s/2,
        s,
        s
    );

    /*
       Extra bright border
    */

    ctx.shadowBlur=10;
    ctx.lineWidth=2;

    ctx.strokeStyle="#ffffff";

    ctx.strokeRect(
        x-s/2,
        y-s/2,
        s,
        s
    );

    ctx.shadowBlur=0;

}


/* =========================
   ANIMATION
========================= */

function animate(){

    update();
    draw();

    requestAnimationFrame(animate);
}

animate();


/* =========================
   MEDIAPIPE HANDS
========================= */

const hands=new Hands({

    locateFile:file=>
        `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`

});


hands.setOptions({

    maxNumHands:1,

    modelComplexity:0,

    minDetectionConfidence:.55,

    minTrackingConfidence:.55

});


/* =========================
   HAND RESULT
========================= */

hands.onResults(results=>{

    if(
        results.multiHandLandmarks &&
        results.multiHandLandmarks.length
    ){

        const hand=
            results.multiHandLandmarks[0];

        /*
           Index finger tip
           Landmark 8
        */

        const finger=hand[8];

        square.targetX=
            (1-finger.x)*W;

        square.targetY=
            finger.y*H;

        status.textContent=
            "👆 Move Your Hand";

    }
    else{

        status.textContent=
            "✋ Show Your Hand";

    }

});


/* =========================
   CAMERA
========================= */

async function startCamera(){

    try{

        const stream=
            await navigator.mediaDevices.getUserMedia({

                video:{
                    facingMode:"user",
                    width:320,
                    height:240,

                    frameRate:{
                        ideal:20,
                        max:24
                    }
                },

                audio:false

            });


        video.srcObject=stream;

        await video.play();


        let last=0;

        async function track(){

            const now=performance.now();

            if(now-last>50){

                last=now;

                await hands.send({
                    image:video
                });

            }

            requestAnimationFrame(track);

        }

        track();

    }

    catch(e){

        status.textContent=
            "Camera Permission Required";

        console.log(e);

    }

}

startCamera();

</script>

</body>
</html>