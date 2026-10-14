
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>My TV & Movies</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#101014;color:white;font-family:Arial,sans-serif}
header{padding:20px;background:#191920}
h1{margin:0;color:#ff4545;font-size:25px}
p{color:#bbb}
nav{display:flex;gap:10px;padding:12px;position:sticky;top:0;background:#101014}
button{background:#e93443;color:white;border:0;border-radius:8px;padding:12px;cursor:pointer}
nav button{flex:1}
main{padding:15px}
input{width:100%;padding:13px;margin:7px 0;background:#24242d;color:white;border:1px solid #444;border-radius:8px}
video{width:100%;background:black;border-radius:10px;margin:10px 0}
.card{background:#202028;padding:14px;border-radius:10px;margin:10px 0}
.small{font-size:13px;color:#aaa}
</style>
</head>
<body>
<header>
<h1>▶ MY TV & MOVIES</h1>
<p>Your personal streaming space</p>
</header>
<nav>
<button onclick="show('tv')">📺 Live TV</button>
<button onclick="show('movies')">🎬 Movies</button>
</nav>
<main>
<input id="search" placeholder="Search channels or movies..." oninput="render()">
<div id="tv" class="section">
<h2>📺 Live TV</h2>
<p class="small">Add a stream URL that you are authorized to watch.</p>
<input id="channel" placeholder="Channel name">
<input id="tvurl" placeholder="Stream URL (M3U8 or MP4)">
<button onclick="add('tv')">+ Add Channel</button>
<div id="tvlist"></div>
</div>
<div id="movies" class="section" hidden>
<h2>🎬 Movies</h2>
<p class="small">Add your own videos or authorized movie links.</p>
<input id="movie" placeholder="Movie name">
<input id="movieurl" placeholder="Video URL (MP4 or supported stream)">
<button onclick="add('movies')">+ Add Movie</button>
<div id="movielist"></div>
</div>
<h2 id="now">Video Player</h2>
<video id="player" controls playsinline></video>
<p id="status" class="small">Choose a channel or movie to play.</p>
</main>
<script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
<script>
let items=[];
try{items=JSON.parse(localStorage.getItem('myMedia')||'[]')}catch(e){}
let currentType='tv',hls;
function show(type){
currentType=type;
document.getElementById('tv').hidden=type!=='tv';
document.getElementById('movies').hidden=type!=='movies';
render();
}
function add(type){
let name=document.getElementById(type==='tv'?'channel':'movie').value.trim();
let url=document.getElementById(type==='tv'?'tvurl':'movieurl').value.trim();
if(!name||!url){alert('Enter both a name and a video URL.');return}
items.push({type,name,url});
try{localStorage.setItem('myMedia',JSON.stringify(items))}
catch(e){alert('Could not save. Your phone storage may be full.')}
document.getElementById(type==='tv'?'channel':'movie').value='';
document.getElementById(type==='tv'?'tvurl':'movieurl').value='';
render();
}
function play(item){
let video=document.getElementById('player');
if(hls){hls.destroy();hls=null}
video.pause();video.removeAttribute('src');video.load();
document.getElementById('now').textContent='▶ '+item.name;
document.getElementById('status').textContent='Loading video...';
if(item.url.includes('.m3u8')&&Hls.isSupported()){
hls=new Hls();hls.loadSource(item.url);hls.attachMedia(video);
hls.on(Hls.Events.MANIFEST_PARSED,()=>video.play().catch(()=>{}));
hls.on(Hls.Events.ERROR,(e,data)=>{if(data.fatal)document.getElementById('status').textContent='Stream unavailable or unsupported.'});
}else{
video.src=item.url;
video.play().then(()=>document.getElementById('status').textContent='Playing').catch(()=>document.getElementById('status').textContent='Tap Play. If it fails, check the URL and format.');
}
}
function removeItem(index){
items.splice(index,1);
try{localStorage.setItem('myMedia',JSON.stringify(items))}catch(e){}
render();
}
function render(){
['tv','movies'].forEach(type=>{
let list=document.getElementById(type+'list');
list.innerHTML='';
let q=document.getElementById('search').value.toLowerCase();
items.forEach((item,index)=>{
if(item.type!==type||!item.name.toLowerCase().includes(q))return;
let card=document.createElement('div');card.className='card';
let title=document.createElement('strong');title.textContent=(type==='tv'?'📺 ':'🎬 ')+item.name;
let playBtn=document.createElement('button');playBtn.textContent='▶ Play';
playBtn.style.margin='10px 8px 0 0';playBtn.onclick=()=>play(item);
let delBtn=document.createElement('button');delBtn.textContent='Remove';
delBtn.style.background='#444';delBtn.onclick=()=>removeItem(index);
card.append(title,document.createElement('br'),playBtn,delBtn);list.appendChild(card);
});
});
}
render();
</script>
</body>
</html>

