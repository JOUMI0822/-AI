<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>GoMaster AI</title>
<style>
:root{--bg:#11151c;--panel:#1b222d;--panel2:#242e3b;--text:#eef3f8;--muted:#9eacbb;--accent:#72c7a2;--line:#344252;--wood:#d9ad70}
*{box-sizing:border-box}body{margin:0;background:var(--bg);color:var(--text);font-family:system-ui,-apple-system,"Noto Sans KR",sans-serif}
header{padding:18px 24px;border-bottom:1px solid var(--line);display:flex;justify-content:space-between;align-items:center;gap:12px;flex-wrap:wrap}
h1{margin:0;font-size:22px}header small{color:var(--muted)}main{max-width:1450px;margin:auto;padding:20px;display:grid;grid-template-columns:minmax(300px,1.3fr) minmax(300px,.9fr);gap:18px}
.panel{background:var(--panel);border:1px solid var(--line);border-radius:14px;padding:16px;min-width:0}.board-wrap{display:flex;justify-content:center;align-items:center;background:#c99b5d;border-radius:10px;padding:10px}
canvas{display:block;width:min(100%,720px);height:auto;touch-action:manipulation;cursor:pointer}
.controls{display:flex;gap:8px;flex-wrap:wrap;margin-top:12px;align-items:center}
button,select{background:var(--panel2);color:var(--text);border:1px solid var(--line);padding:9px 12px;border-radius:8px;font-size:14px}
button{cursor:pointer}button:hover{border-color:var(--accent)}button.primary{background:#27664e;border-color:#398566}
.status{display:flex;justify-content:space-between;gap:10px;flex-wrap:wrap;margin:10px 2px;color:var(--muted);font-size:14px}
h2{font-size:17px;margin:0 0 12px}.card{background:var(--panel2);padding:13px;border-radius:10px;margin:10px 0}.tag{display:inline-block;padding:3px 8px;border-radius:20px;background:#344d43;color:#a9e8c9;font-size:12px;margin-right:6px}
.muted{color:var(--muted)}.big{font-size:20px;font-weight:700}.grid2{display:grid;grid-template-columns:1fr 1fr;gap:8px}.metric{background:#18202a;padding:10px;border-radius:8px}.metric b{display:block;font-size:18px;margin-top:4px}
ol{padding-left:22px;max-height:210px;overflow:auto}.move{padding:5px 0;border-bottom:1px solid #344252;font-size:13px}.move button{padding:3px 7px;margin-left:5px;font-size:12px}
#feedback{line-height:1.65;font-size:14px;white-space:pre-line}.footer{max-width:1450px;margin:0 auto;padding:0 20px 25px;color:var(--muted);font-size:12px}
@media(max-width:850px){main{grid-template-columns:1fr;padding:12px}.panel{padding:12px}header{padding:14px}}
</style>
</head>
<body>
<header><div><h1>⚫ GoMaster AI</h1><small>대국하고, 근거를 확인하며 배우는 바둑 코치</small></div><div class="muted">프로토타입 v0.1 · 브라우저 실행</div></header>
<main>
<section class="panel">
 <h2>바둑판</h2>
 <div class="board-wrap"><canvas id="board" width="760" height="760" aria-label="바둑판"></canvas></div>
 <div class="status"><span id="turn">흑 차례</span><span id="count">0수</span><span id="captures">잡은 돌 — 흑 0 / 백 0</span></div>
 <div class="controls">
  <label>판 크기 <select id="size"><option>9</option><option>13</option><option selected>19</option></select></label>
  <label>내 색 <select id="human"><option value="B">흑</option><option value="W">백</option></select></label>
  <label>AI 수준 <select id="level"><option>입문</option><option selected>초급</option><option>중급</option><option>고급</option></select></label>
  <button id="undo">↶ 무르기</button><button id="pass">패스</button><button id="new" class="primary">새 대국</button>
 </div>
 <div class="controls"><button id="analyze">현재 국면 피드백</button><button id="review">대국 복기 요약</button><button id="save">SGF 저장</button><label>SGF 불러오기 <input id="load" type="file" accept=".sgf,text/plain" hidden></label><button id="loadBtn">파일 선택</button></div>
</section>
<aside>
 <section class="panel">
  <h2>🧠 착수 피드백</h2>
  <div class="card"><span class="tag">엔진 연결 상태</span><b id="engine">KataGo 미연결</b><p class="muted" style="margin-bottom:0">현재 버전은 바둑 규칙과 기초 전술을 처리합니다. 엔진 기반 승률·점수 평가는 아직 제공하지 않습니다.</p></div>
  <div class="grid2"><div class="metric"><span class="muted">현재 차례</span><b id="metricTurn">흑</b></div><div class="metric"><span class="muted">착수 수</span><b id="metricMoves">0</b></div></div>
  <div class="card" id="feedback">바둑판에 착수한 뒤 ‘현재 국면 피드백’을 눌러 보세요.\n\n피드백에는 확인 가능한 근거만 표시합니다.</div>
 </section>
 <section class="panel" style="margin-top:18px">
  <h2>📜 착수 기록</h2><div class="muted" id="historyLabel">아직 착수가 없습니다.</div><ol id="history"></ol>
 </section>
 <section class="panel" style="margin-top:18px">
  <h2>📘 사용 안내</h2>
  <p class="muted" style="font-size:13px;line-height:1.6;margin-bottom:0">바둑판을 클릭해 착수합니다. 기본적인 단수·따내기·자충수·패 규칙을 적용합니다. 한국식/중국식 계가와 복잡한 패 순환, 코·장생 등 고급 규칙은 아직 완전하지 않습니다. 실제 대국 분석을 위해서는 별도 KataGo 서버 연결이 필요합니다.</p>
 </section>
</aside>
</main>
<div class="footer">주의: 이 프로토타입의 평가는 교육용 기초 규칙 점검이며 프로 수준의 바둑 분석이 아닙니다.</div>
<script>
const cvs=document.getElementById('board'),ctx=cvs.getContext('2d');
const $=id=>document.getElementById(id);
let N=19, board=[], turn='B', moves=[], snapshots=[], captures={B:0,W:0}, passes=0, gameOver=false;
let lastMove=null, human='B';
const starPoints={9:[2,4,6],13:[3,6,9],19:[3,9,15]};
function empty(){return Array.from({length:N},()=>Array(N).fill(null))}
function clone(b){return b.map(r=>r.slice())}
function reset(){N=+$('size').value;human=$('human').value;board=empty();turn='B';moves=[];snapshots=[];captures={B:0,W:0};passes=0;gameOver=false;lastMove=null;render();update();$('feedback').textContent='새 대국을 시작했습니다. 바둑판에 착수해 보세요.'}
function xy(i){return String.fromCharCode(65+i+(i>=8?1:0))}
function coord(x,y){return xy(x)+(N-y)}
function neighbors(x,y){return [[x-1,y],[x+1,y],[x,y-1],[x,y+1]].filter(([a,b])=>a>=0&&a<N&&b>=0&&b<N)}
function group(b,x,y){let c=b[y][x], stack=[[x,y]], seen=new Set([x+','+y]), stones=[], libs=new Set();while(stack.length){let [a,d]=stack.pop();stones.push([a,d]);for(let [nx,ny] of neighbors(a,d)){if(!b[ny][nx])libs.add(nx+','+ny);else if(b[ny][nx]===c&&!seen.has(nx+','+ny)){seen.add(nx+','+ny);stack.push([nx,ny])}}}return {stones,libs}}
function tryMove(x,y,color,source='human'){
 if(gameOver||board[y][x])return {ok:false,reason:'이미 돌이 놓인 자리입니다.'};
 let b=clone(board);b[y][x]=color;let enemy=color==='B'?'W':'B',taken=[];
 for(let [nx,ny] of neighbors(x,y)){if(b[ny][nx]===enemy){let g=group(b,nx,ny);if(g.libs.size===0)taken.push(...g.stones)}}
 for(let [a,d] of taken)b[d][a]=null;
 if(group(b,x,y).libs.size===0)return {ok:false,reason:'자충수입니다. 착수 후 자신의 돌에 활로가 없습니다.'};
 // Simple ko: reject exact previous board position
 if(snapshots.length>=1&&same(b,snapshots[snapshots.length-1].board))return {ok:false,reason:'패 규칙: 직전 국면과 동일해지는 착수입니다.'};
 snapshots.push({board:clone(board),turn,captures:{...captures},lastMove,passes});
 board=b;captures[color]+=taken.length;turn=enemy;passes=0;lastMove={x,y,color};moves.push({x,y,color,taken:taken.length,board:clone(board)});
 return {ok:true,taken:taken.length};
}
function same(a,b){return a.every((r,y)=>r.every((v,x)=>v===b[y][x]))}
function play(x,y){if(gameOver)return;if(turn!==human){$('feedback').textContent='AI 차례입니다.';return}let result=tryMove(x,y,turn);if(!result.ok){$('feedback').textContent='착수할 수 없습니다: '+result.reason;return}update();render();if(result.taken)$('feedback').textContent=`돌 ${result.taken}개를 따냈습니다. 주변의 활로와 연결 상태를 확인해 보세요.`;if(!gameOver&&turn!==human)setTimeout(aiMove,280)}
function aiMove(){
 if(gameOver||turn===human)return;
 let candidates=[];
 // Simple legal heuristic: capture, liberties, adjacency; not an engine.
 for(let y=0;y<N;y++)for(let x=0;x<N;x++)if(!board[y][x]){
  let b=clone(board);b[y][x]=turn;let enemy=turn==='B'?'W':'B',taken=0;
  for(let [nx,ny] of neighbors(x,y))if(b[ny][nx]===enemy){let g=group(b,nx,ny);if(!g.libs.size){taken+=g.stones.length;for(let [a,d] of g.stones)b[d][a]=null}}
  if(group(b,x,y).libs.size===0)continue;
  if(snapshots.length&&same(b,snapshots[snapshots.length-1].board))continue;
  let own=0,opp=0,emptyN=0;
  for(let [nx,ny] of neighbors(x,y)){if(b[ny][nx]===turn)own++;else if(b[ny][nx]===enemy)opp++;else emptyN++}
  let center=(N-1)/2, centrality=-(Math.abs(x-center)+Math.abs(y-center))*0.03;
  let score=taken*5+own*0.7+opp*0.15+emptyN*0.08+centrality+Math.random()*0.2;
  candidates.push({x,y,score});
 }
 if(!candidates.length){passMove(true);return}
 candidates.sort((a,b)=>b.score-a.score);
 let pick=candidates[Math.floor(Math.random()*Math.min(candidates.length,Math.max(1,({입문:25,초급:12,중급:5,고급:2})[$('level').value])) )];
 let r=tryMove(pick.x,pick.y,turn,'ai');if(!r.ok){passMove(true);return}render();update();
 $('feedback').textContent=`AI가 ${coord(pick.x,pick.y)}에 착수했습니다.\n\n이 AI는 현재 기초 규칙 기반의 시범 상대입니다. KataGo의 탐색 결과가 아니므로 강도나 최선의 수를 보장하지 않습니다.`;
}
function passMove(fromAI=false){if(gameOver)return;if(turn!==human&&!fromAI)return;snapshots.push({board:clone(board),turn,captures:{...captures},lastMove,passes});moves.push({pass:true,color:turn,board:clone(board)});passes++;turn=turn==='B'?'W':'B';lastMove=null;if(passes>=2){gameOver=true;$('feedback').textContent='양측이 연속 패스했습니다. 대국이 종료되었습니다. 정확한 집 계산은 아직 지원하지 않습니다.'}render();update();if(!gameOver&&turn!==human)setTimeout(aiMove,250)}
function undo(){if(!snapshots.length)return;let steps=turn===human?1:Math.min(2,snapshots.length);while(steps--&&snapshots.length){let s=snapshots.pop();board=s.board;turn=s.turn;captures=s.captures;lastMove=s.lastMove;passes=s.passes;moves.pop()}gameOver=false;render();update();$('feedback').textContent='최근 착수를 무르고 이전 국면으로 돌아왔습니다.'}
function drawBoard(){let w=cvs.width,p=42,step=(w-2*p)/(N-1);ctx.clearRect(0,0,w,w);ctx.fillStyle='#d9ad70';ctx.fillRect(0,0,w,w);ctx.strokeStyle='#54391f';ctx.lineWidth=1.6;for(let i=0;i<N;i++){let q=p+i*step;ctx.beginPath();ctx.moveTo(p,q);ctx.lineTo(w-p,q);ctx.stroke();ctx.beginPath();ctx.moveTo(q,p);ctx.lineTo(q,w-p);ctx.stroke()}
 let stars=starPoints[N]||[];ctx.fillStyle='#54391f';for(let a of stars)for(let b of stars){ctx.beginPath();ctx.arc(p+a*step,p+b*step,4,0,Math.PI*2);ctx.fill()}
 for(let y=0;y<N;y++)for(let x=0;x<N;x++){let c=board[y][x];if(!c)continue;let px=p+x*step,py=p+y*step,r=step*.43;ctx.beginPath();ctx.arc(px+1.5,py+2.5,r,0,Math.PI*2);ctx.fillStyle='rgba(0,0,0,.25)';ctx.fill();ctx.beginPath();ctx.arc(px,py,r,0,Math.PI*2);ctx.fillStyle=c==='B'?'#101318':'#f7f7f2';ctx.fill();ctx.strokeStyle=c==='B'?'#000':'#aaa';ctx.lineWidth=1;ctx.stroke()}
 if(lastMove){let p0=42,st=(w-84)/(N-1);ctx.beginPath();ctx.arc(p0+lastMove.x*st,p0+lastMove.y*st,4,0,Math.PI*2);ctx.fillStyle=lastMove.color==='B'?'#fff':'#e33';ctx.fill()}
}
function render(){drawBoard()}
function update(){ $('turn').textContent=gameOver?'대국 종료':(turn==='B'?'흑 차례':'백 차례');$('count').textContent=moves.length+'수';$('captures').textContent=`잡은 돌 — 흑 ${captures.B} / 백 ${captures.W}`;$('metricTurn').textContent=turn==='B'?'흑':'백';$('metricMoves').textContent=moves.length;$('historyLabel').textContent=moves.length?`총 ${moves.length}수 · ${gameOver?'종료':'진행 중'}`:'아직 착수가 없습니다.';$('history').innerHTML=moves.map((m,i)=>`<li class="move">${i+1}. ${m.color==='B'?'흑':'백'} ${m.pass?'패스':coord(m.x,m.y)}${m.taken?` · ${m.taken}개 따냄`:''}</li>`).join('')}
function analyze(){let ownGroups=0,atari=[],isolated=0;let seen=new Set();for(let y=0;y<N;y++)for(let x=0;x<N;x++){let c=board[y][x],k=x+','+y;if(!c||seen.has(k))continue;let g=group(board,x,y);g.stones.forEach(([a,b])=>seen.add(a+','+b));ownGroups++;if(g.libs.size===1)atari.push({color:c,stones:g.stones.length,lib:[...g.libs][0]});if(g.stones.length===1&&g.libs.size<=2)isolated++}
 let text=`현재 국면의 확인 가능한 사실\n\n• 돌 무리 수: ${ownGroups}개\n• 단수(활로 1개인 무리): ${atari.length}개\n• 단독 돌 중 활로가 2개 이하인 돌: ${isolated}개\n• 누적 포획: 흑 ${captures.B}개 / 백 ${captures.W}개\n\n해석\n`;
 if(atari.length)text+=`• 단수 상태인 돌 무리가 ${atari.length}개 있습니다. 다음 수에 잡힐 수 있으므로 해당 활로를 메우거나 상대 돌을 먼저 잡을 수 있는지 검토하세요.\n`;
 else text+='• 현재 단수 상태인 돌 무리는 발견되지 않았습니다.\n';
 text+='• 이 정보는 현재 판의 연결·활로를 직접 계산한 결과입니다. 승률, 예상 집, 최선의 수는 KataGo가 연결되지 않아 판단하지 않습니다.\n\n추천 수를 확정하려면 엔진 분석이 필요합니다.';$('feedback').textContent=text}
function review(){if(!moves.length){$('feedback').textContent='복기할 착수가 없습니다.';return}let cap=moves.reduce((a,m)=>a+(m.taken||0),0),passesCount=moves.filter(m=>m.pass).length;$('feedback').textContent=`대국 기록 요약\n\n• 총 착수: ${moves.length}수\n• 흑 포획: ${captures.B}개\n• 백 포획: ${captures.W}개\n• 패스: ${passesCount}회\n• 현재 판 크기: ${N}×${N}\n\n이 버전은 각 수의 엔진 평가값을 저장하지 않으므로 '최선의 수', '실수', '승률 손실'을 사후 판정하지 않습니다. KataGo 연결 후에는 수순별 분석을 추가할 수 있습니다.`}
function sgf(){let s=`(;GM[1]FF[4]CA[UTF-8]AP[GoMasterAI:0.1]SZ[${N}]`;for(let m of moves){if(m.pass)s+=`;${m.color}[]`;else{s+=`;${m.color}[${String.fromCharCode(97+m.x)}${String.fromCharCode(97+m.y)}]`}}return s+')'}
function save(){let blob=new Blob([sgf()],{type:'application/x-go-sgf'}),a=document.createElement('a');a.href=URL.createObjectURL(blob);a.download='gomaster-game.sgf';a.click();URL.revokeObjectURL(a.href)}
function loadSGF(txt){let sz=txt.match(/SZ\[(\d+)\]/);if(!sz)throw Error('SGF 판 크기를 찾을 수 없습니다.');$('size').value=sz[1];reset();let re=/;([BW])\[([a-s]{0,2})\]/g,m;while((m=re.exec(txt))){if(!m[2]){passMove(turn!==human);continue}let x=m[2].charCodeAt(0)-97,y=m[2].charCodeAt(1)-97;if(x<N&&y<N){let oldHuman=human;human=turn;let r=tryMove(x,y,turn);human=oldHuman;if(!r.ok)throw Error('불러오기 중 잘못된 착수가 있습니다: '+r.reason)}}render();update();$('feedback').textContent='SGF 기록을 불러왔습니다. 일부 고급 SGF 속성은 지원하지 않습니다.'}
cvs.addEventListener('click',e=>{let rect=cvs.getBoundingClientRect(),scale=cvs.width/rect.width,xp=(e.clientX-rect.left)*scale,yp=(e.clientY-rect.top)*scale,p=42,step=(cvs.width-84)/(N-1),x=Math.round((xp-p)/step),y=Math.round((yp-p)/step);if(x>=0&&x<N&&y>=0&&y<N&&Math.hypot(xp-(p+x*step),yp-(p+y*step))<step*.48)play(x,y)});
$('size').onchange=reset;$('human').onchange=reset;$('new').onclick=reset;$('undo').onclick=undo;$('pass').onclick=()=>passMove();$('analyze').onclick=analyze;$('review').onclick=review;$('save').onclick=save;$('loadBtn').onclick=()=>$('load').click();$('load').onchange=async e=>{let f=e.target.files[0];if(!f)return;try{loadSGF(await f.text())}catch(err){alert(err.message)}e.target.value=''};
reset();
</script>
</body></html>
