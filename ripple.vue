<template>
    <div class="sky" ref="sky">
        <canvas ref="canvas"></canvas>
        <div class="filter"></div>
    </div>
</template>
<script>
export default {
    mounted() {
        let _this = this;
        _this.$refs.sky.width = window.innerWidth;
        _this.$refs.sky.height = window.innerHeight;
        function Star(id, x, y) {
            this.id = id;
            this.x = x;
            this.y = y;
            this.r = Math.floor(Math.random() * 2) + 1;
            var alpha = (Math.floor(Math.random() * 10) + 1) / 10 / 2;
            this.color = "rgba(255, 255, 255," + alpha + ")"
        }
        Star.prototype.draw = function() {
            ctx.fillStyle = this.color; // 填充色
            ctx.shadowBlur = this.r * 2; // 阴影
            ctx.beginPath(); // 起始路径
            ctx.arc(this.x, this.y, this.r, 0, 2 * Math.PI, false); // 弧
            ctx.closePath(); // 闭合路径
            ctx.fill(); // 图型填充
        }
        Star.prototype.move = function() {
            this.y -= .5;
            if (this.y <= -10) this.y = _Height + 10;
            this.draw();
        }
        Star.prototype.die = function() {
            stars[this.id] = null;
            delete stars[this.id];
        }
        function Dot(id, x, y) {
            this.id = id;
            this.x = x;
            this.y = y;
            this.r = Math.floor(Math.random() * 5) + 1;
            this.speed = .5;
            this.a = .5;
            this.aReduction = .005;
            this.color = "rgba(255,255,255,"+this.a + ")";
            this.dir = Math.floor(Math.random()*140)+200;
        }
        
        Dot.prototype.draw = function() {
            ctx.fillStyle = this.color;
            ctx.shadowBlur = this.r * 2;
            ctx.beginPath();
            ctx.arc(this.x, this.y, this.r, 0, 2 * Math.PI, false);
            ctx.closePath();
            ctx.fill();
        }
        function getPreviousDot(id, stepback) {
            if (id == 0 || id - stepback < 0) return false;
            if (typeof dots[id - stepback] != "undefined") return dots[id - stepback];
            else return false;
        }
        Dot.prototype.move = function() {
            this.a -= this.aReduction;
            if (this.a <= 0) {
                this.die();
                return
            }
            this.color = "rgba(255,255,255," + this.a + ")";
            this.x = this.x + Math.cos(degToRad(this.dir))*this.speed;
            this.y = this.y + Math.sin(degToRad(this.dir))*this.speed;

            this.draw();
        }

        Dot.prototype.die = function() {
            dots[this.id] = null;
            delete dots[this.id];
        }

        function Ripple(id,x,y,r) {
            this.id=id;
            this.x=x;
            this.y=y;
            this.r=r;
            this.a = 1;
            this.b =25;
            this.aReduction = .2;
            this.color = "rgba(0,128,0,"+this.a + ")";
        }
        Ripple.prototype.draw =function(){
            ctx.strokeStyle = this.color;
            ctx.beginPath();
            ctx.arc(this.x,this.y,this.r,0,2*Math.PI,false);
            ctx.stroke();
            ctx.closePath();
        }
        Ripple.prototype.move =function(){
            this.a-=this.aReduction;
            this.r+=this.b;
            if(this.a<=0){
                this.die();
                return
            }
            this.color="rgba(0,128,0,"+this.a+")";
            this.draw();
        }
        Ripple.prototype.die = function(){
            ripples[this.id]=null;
            delete ripples[this.id];
        }
        var canvas = _this.$refs.canvas,
            ctx = canvas.getContext("2d"),
            _Width = _this.$refs.sky.width,
            _Height = _this.$refs.sky.height,
            mouseMoving = false,
            click = false,
            mouseMoveChecker,
            clickChecker,
            mouseX,
            mouseY,
            clickX,
            clickY,
	        dots = [],
            stars = [],
            ripples = [],
	        dotsMinDist = 2,
	        maxDistFromCursor = 50,
            initStarsPopulation = 200;
        setCanvasSize();
        __init__();
        function setCanvasSize() {
            canvas.setAttribute("width", _Width)
            canvas.setAttribute("height", _Height)
        }
        function __init__() {
            ctx.shadowColor = "white";
            for (var i = 0; i < initStarsPopulation; i++) {
                stars[i] = new Star(i, Math.floor(Math.random() * _Width), Math.floor(Math.random() * _Height));
            }
            ctx.shadowBlur = 0;
            animate();
        }
        function animate() {
            ctx.clearRect(0, 0, _Width, _Height);
            for (var i in stars) {
                stars[i].move();
            }
            for (var i in dots) {
                dots[i].move();
            }
            for (var i in ripples) {
                ripples[i].move();
            }
            
            drawIfMouseMoving();
            drawIfclick();
            setTimeout(function(){
                requestAnimationFrame(animate);
            },50);
            
        }
        window.onmousemove = function(e){
            mouseMoving = true;
            mouseX = e.clientX;
            mouseY = e.clientY;
            setTimeout(function() {
                mouseMoving = false;
            }, 100);
        }
        window.onclick = function(e){
            click = true;
            clickX = e.clientX;
            clickY = e.clientY;
            setTimeout(function(){
                click=false;
            },100);
        }
        function drawIfMouseMoving(){
            if (!mouseMoving) return;

            if (dots.length == 0) {
                dots[0] = new Dot(0, mouseX, mouseY);
                dots[0].draw();
                return;
            }

            var previousDot = getPreviousDot(dots.length, 1);
            var prevX = previousDot.x; 
            var prevY = previousDot.y; 

            var diffX = Math.abs(prevX - mouseX);
            var diffY = Math.abs(prevY - mouseY);

            if (diffX < dotsMinDist || diffY < dotsMinDist) return;
            dots[dots.length] = new Dot(dots.length, mouseX, mouseY);
            dots[dots.length-1].draw();
        }
        function drawIfclick(){
            if (!click) return;
            ripples[ripples.length] = new Ripple(ripples.length,clickX,clickY,15);
            ripples[ripples.length-1].draw();
        }

        function degToRad(deg) {
            return deg * (Math.PI / 180);
        }
    }
}
</script>
<style lang="less" scoped>
    .sky{
        background: radial-gradient(65% 105% at bottom center, #f7f7b6 10%, #e96f92 40%,#75517d  65%, #1b2947);
    }
    .filter{
        position: fixed;
        width: 100%;
        height: 100%;
        top:0;
        background: #f00000;
        mix-blend-mode: overlay;
        animation: colorChange 30s ease-in-out infinite;
        animation-fill-mode: both;
    }
    @keyframes colorChange {
        0%,
        100% {
            opacity: 0;
        }
        50% {
            opacity: 0.9;
        }
    }
</style>