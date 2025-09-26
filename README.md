
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>Air Strike - Bắn Máy Bay</title>
  <style>
    body { margin:0; background:#000; overflow:hidden; font-family: Arial, sans-serif; color:white; }
    canvas { background:#111; display:block; margin:0 auto; }
    #menu {
      position:absolute; top:0; left:0; width:100%; height:100%; background:#000;
      display:flex; flex-direction:column; justify-content:center; align-items:center;
    }
    #logo { font-size:48px; font-weight:bold; color:#f33; text-shadow:0 0 10px #fff; margin-bottom:20px; }
    #subtitle { font-size:20px; margin-bottom:30px; color:#ccc; }
    .btn { padding:10px 20px; margin:10px; font-size:20px; cursor:pointer; border:none; border-radius:10px; background:#333; color:white; }
    .btn:hover { background:#555; }
  </style>
</head>
<body>
  <div id="menu">
    <div id="logo">🚀 AIR STRIKE</div>
    <div id="subtitle">Game Bắn Máy Bay</div>
    <button class="btn" onclick="startGame('easy')">Easy</button>
    <button class="btn" onclick="startGame('normal')">Normal</button>
    <button class="btn" onclick="startGame('hard')">Hard</button>
  </div>
  <canvas id="gameCanvas" width="480" height="640"></canvas>

  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    let player, bullets = [], enemies = [], enemyBullets = [], keys = {}, score=0, level=1, gameLoop;
    let mode = "normal", inGame = false, spawnRate = 100;

    class Player {
      constructor(){
        this.x = canvas.width/2 - 20;
        this.y = canvas.height - 60;
        this.width = 40;
        this.height = 40;
        this.speed = 5;
      }
      draw(){
        ctx.fillStyle = "cyan";
        ctx.fillRect(this.x,this.y,this.width,this.height);
      }
    }

    class Bullet {
      constructor(x,y){ this.x=x; this.y=y; this.width=5; this.height=10; this.speed=7; }
      update(){ this.y -= this.speed; }
      draw(){ ctx.fillStyle="yellow"; ctx.fillRect(this.x,this.y,this.width,this.height); }
    }

    class Enemy {
      constructor(x,y,type="normal"){
        this.x=x; this.y=y; this.width=40; this.height=40; this.speed=2+level*0.5;
        this.type = type;
        this.hp = (type==="boss"? 50+level*20 : 1);
      }
      update(){ this.y += this.speed; }
      draw(){
        ctx.fillStyle = (this.type==="boss"? "red":"lime");
        ctx.fillRect(this.x,this.y,this.width,this.height);
      }
    }

    function startGame(selectedMode){
      mode = selectedMode;
      document.getElementById("menu").style.display="none";
      inGame=true;
      resetGame();
      gameLoop = setInterval(update,1000/60);
      // auto fire for mobile
      if(isMobile()){
        setInterval(()=>{
          if(inGame) bullets.push(new Bullet(player.x+player.width/2-2,player.y));
        }, 300);
      }
    }

    function resetGame(){
      player = new Player();
      bullets=[]; enemies=[]; enemyBullets=[]; score=0; level=1;
      if(mode==="easy") spawnRate=120;
      if(mode==="normal") spawnRate=80;
      if(mode==="hard") spawnRate=50;
    }

    function spawnEnemy(){
      if(level%3===0 && Math.random()<0.01){ // Boss every 3 levels
        enemies.push(new Enemy(Math.random()*(canvas.width-60), -60, "boss"));
      } else {
        enemies.push(new Enemy(Math.random()*(canvas.width-40), -40));
      }
    }

    function update(){
      ctx.clearRect(0,0,canvas.width,canvas.height);
      // Controls PC
      if(!isMobile()){
        if(keys["ArrowLeft"]||keys["a"]) player.x -= player.speed;
        if(keys["ArrowRight"]||keys["d"]) player.x += player.speed;
        if(keys["ArrowUp"]||keys["w"]) player.y -= player.speed;
        if(keys["ArrowDown"]||keys["s"]) player.y += player.speed;
      }
      // Boundaries
      if(player.x<0) player.x=0; if(player.x+player.width>canvas.width) player.x=canvas.width-player.width;
      if(player.y<0) player.y=0; if(player.y+player.height>canvas.height) player.y=canvas.height-player.height;

      // Shooting PC
      if(keys[" "] && !isMobile() && frame%15===0){
        bullets.push(new Bullet(player.x+player.width/2-2,player.y));
      }

      // Update bullets
      bullets.forEach((b,i)=>{ b.update(); if(b.y<0) bullets.splice(i,1); b.draw(); });

      // Spawn enemies
      if(frame%spawnRate===0) spawnEnemy();
      enemies.forEach((e,i)=>{
        e.update();
        if(e.y>canvas.height) enemies.splice(i,1);
        e.draw();
        // Collision with bullets
        bullets.forEach((b,bi)=>{
          if(b.x<b.x+b.width && b.x+ b.width>e.x && b.y<b.y+b.height && b.y+b.height>e.y){
            e.hp--; bullets.splice(bi,1);
            if(e.hp<=0){ enemies.splice(i,1); score+= (e.type==="boss"? 500:100); }
          }
        });
      });

      // Level up
      if(score> level*1000){ level++; }

      // Draw player
      player.draw();
      // UI
      ctx.fillStyle="white";
      ctx.fillText("Score: "+score,10,20);
      ctx.fillText("Level: "+level,10,40);
    }

    let frame=0;
    setInterval(()=>{ frame++; },1000/60);

    // Keyboard
    document.addEventListener("keydown", e=> keys[e.key]=true);
    document.addEventListener("keyup", e=> keys[e.key]=false);

    // Touch controls
    function isMobile(){ return /Mobi|Android/i.test(navigator.userAgent); }
    if(isMobile()){
      let touchId=null;
      canvas.addEventListener("touchstart",e=>{
        touchId=e.changedTouches[0].identifier;
      });
      canvas.addEventListener("touchmove",e=>{
        for(let t of e.changedTouches){
          if(t.identifier===touchId){
            player.x = t.clientX - player.width/2;
            player.y = t.clientY - player.height/2;
          }
        }
      });
    }
  </script>
</body>
</html>
function drawPlayer(ctx, x, y) {
  ctx.fillStyle = "cyan"; 
  ctx.beginPath();
  ctx.moveTo(x, y - 20);   // mũi máy bay
  ctx.lineTo(x - 15, y + 20); // cánh trái
  ctx.lineTo(x + 15, y + 20); // cánh phải
  ctx.closePath();
  ctx.fill();

  // Ô kính
  ctx.fillStyle = "white";
  ctx.fillRect(x - 5, y - 15, 10, 10);
}
