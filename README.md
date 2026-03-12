<!DOCTYPE html>
<html>
<head>
<title>Goal Based Agent - Real Maze Solver</title>
<style>
body{
  background:#0f2027;
  font-family:Arial;
  color:white;
  text-align:center;
}
.container{
  background:#1b3a4b;
  width:540px;
  margin:20px auto;
  padding:20px;
  border-radius:15px;
  box-shadow:0 0 25px #00ffd5;
}
h1{color:#00ffd5;}
input,button{
  margin:8px;
  padding:10px;
  border-radius:6px;
  border:none;
}
button{
  background:#00ffd5;
  font-weight:bold;
  cursor:pointer;
}
canvas{
  border:3px solid #00ffd5;
  margin-top:10px;
}
#result{
  margin-top:10px;
  font-size:18px;
  color:#ffeb3b;
}
</style>
</head>

<body>

<div class="container">
<h1>Goal Based Agent</h1>
<p>Upload Maze Image (Black walls, White path)</p>

<input type="file" id="mazeInput" accept="image/*"><br>
<button onclick="solveMaze()">Find Path</button><br>

<canvas id="mazeCanvas" width="300" height="300"></canvas>
<h3 id="result"></h3>
</div>

<script>
const canvas = document.getElementById("mazeCanvas");
const ctx = canvas.getContext("2d");

let grid=[], start=null, goal=null;
let size=60, cell=5;

document.getElementById("mazeInput").addEventListener("change", loadMaze);

function loadMaze(e){
  const file=e.target.files[0];
  if(!file) return;
  const img=new Image();
  img.onload=()=>{
    ctx.clearRect(0,0,300,300);
    ctx.drawImage(img,0,0,300,300);
    buildGrid();
    document.getElementById("result").innerText="Maze loaded. Ready to solve 🧠";
  };
  img.src=URL.createObjectURL(file);
}

function buildGrid(){
  const data = ctx.getImageData(0,0,300,300).data;

  // --- detect maze area (ignore white background) ---
  let minX=300,minY=300,maxX=0,maxY=0;

  for(let y=0;y<300;y++){
    for(let x=0;x<300;x++){
      let i=(y*300+x)*4;
      let b=(data[i]+data[i+1]+data[i+2])/3;
      if(b<240){ // not white background
        minX=Math.min(minX,x);
        minY=Math.min(minY,y);
        maxX=Math.max(maxX,x);
        maxY=Math.max(maxY,y);
      }
    }
  }

  let mazeW=maxX-minX;
  let mazeH=maxY-minY;

  grid=[];
  start=null;
  goal=null;

  for(let y=0;y<size;y++){
    let row=[];
    for(let x=0;x<size;x++){
      let px = Math.floor(minX + (x/size)*mazeW);
      let py = Math.floor(minY + (y/size)*mazeH);
      let i=(py*300+px)*4;
      let b=(data[i]+data[i+1]+data[i+2])/3;

      if(b>200){
        row.push(0); // path
        if(!start) start=[x,y];
        goal=[x,y];
      }else{
        row.push(1); // wall
      }
    }
    grid.push(row);
  }
}

function solveMaze(){
  if(!start||!goal){
    alert("Maze not detected properly");
    return;
  }

  let q=[start];
  let visited=Array.from({length:size},()=>Array(size).fill(false));
  let parent={};
  visited[start[1]][start[0]]=true;
  let found=false;

  while(q.length){
    let [x,y]=q.shift();
    if(x===goal[0]&&y===goal[1]){found=true;break;}
    for(let [dx,dy] of [[1,0],[-1,0],[0,1],[0,-1]]){
      let nx=x+dx, ny=y+dy;
      if(nx>=0&&ny>=0&&nx<size&&ny<size&&!visited[ny][nx]&&grid[ny][nx]===0){
        visited[ny][nx]=true;
        parent[`${nx},${ny}`]=[x,y];
        q.push([nx,ny]);
      }
    }
  }

  if(found){
    drawPath(parent);
    document.getElementById("result").innerText="Goal Reached! Real Path Found 🎯🧠";
  }else{
    document.getElementById("result").innerText="No Path Found ❌";
  }
}

function drawPath(parent){
  ctx.strokeStyle="red";
  ctx.lineWidth=2;
  ctx.beginPath();

  let cur=goal;
  while(cur && !(cur[0]===start[0]&&cur[1]===start[1])){
    ctx.lineTo(cur[0]*cell,cur[1]*cell);
    cur=parent[`${cur[0]},${cur[1]}`];
  }
  ctx.stroke();

  // start & goal
  ctx.fillStyle="lime";
  ctx.fillRect(start[0]*cell,start[1]*cell,cell,cell);
  ctx.fillStyle="yellow";
  ctx.fillRect(goal[0]*cell,goal[1]*cell,cell,cell);
}
</script>

</body>
</html>
