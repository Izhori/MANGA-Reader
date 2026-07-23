<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=no, viewport-fit=cover" />
<meta name="apple-mobile-web-app-capable" content="yes" />
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
<meta name="apple-mobile-web-app-title" content="マンガReader" />
<meta name="mobile-web-app-capable" content="yes" />
<meta name="theme-color" content="#0f1115" />
<title>マンガReader for iPhone</title>
<!-- ショートカット用アイコン -->
<link rel="icon" href="icons/favicon.svg" type="image/svg+xml" />
<link rel="icon" href="icons/favicon-32.png" sizes="32x32" type="image/png" />
<link rel="icon" href="icons/favicon-16.png" sizes="16x16" type="image/png" />
<link rel="apple-touch-icon" href="icons/apple-touch-icon.png" />
<link rel="manifest" href="manifest.webmanifest" />
<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
<style>
  :root{
    --bg:#0f1115; --surface:#1a1d24; --surface2:#232733; --border:#2e3440;
    --text:#e6e8ee; --dim:#9aa1b0; --accent:#e0405a; --reader:#000;
    --safe-t:env(safe-area-inset-top); --safe-b:env(safe-area-inset-bottom);
  }
  *{box-sizing:border-box; -webkit-tap-highlight-color:transparent; -webkit-touch-callout:none;}
  html,body{margin:0; height:100%; overscroll-behavior:none;}
  body{
    background:var(--bg); color:var(--text); overflow:hidden;
    font-family:-apple-system,BlinkMacSystemFont,"Hiragino Kaku Gothic ProN","Yu Gothic",Meiryo,sans-serif;
    -webkit-user-select:none; user-select:none; touch-action:none;
  }
  button{font-family:inherit;}

  /* ---------- ホーム ---------- */
  #home{position:fixed; inset:0; display:flex; flex-direction:column; align-items:center; justify-content:center;
    gap:20px; padding:calc(24px + var(--safe-t)) 22px calc(24px + var(--safe-b)); text-align:center;}
  #home h1{margin:0; font-size:27px; letter-spacing:.05em; font-weight:800;}
  #home h1 .m{color:var(--accent);}
  #home .sub{color:var(--dim); font-size:12px; letter-spacing:.14em; margin-top:-8px;}
  #home p{margin:0; color:var(--dim); font-size:13.5px; line-height:1.75; max-width:440px;}
  .drop{width:100%; max-width:440px; border:2px dashed var(--border); border-radius:18px; padding:30px 20px; background:var(--surface); display:flex; flex-direction:column; align-items:center; gap:16px;}
  .drop .ic{font-size:42px;}
  .btns{display:flex; flex-direction:column; gap:10px; width:100%;}
  .btn{background:var(--surface2); color:var(--text); border:1px solid var(--border); padding:14px 16px; border-radius:13px; font-size:15px; cursor:pointer; width:100%;}
  .btn:active{background:#2b3040;}
  .btn.primary{background:var(--accent); border-color:var(--accent); color:#fff; font-weight:700;}
  #recent{display:flex; flex-wrap:wrap; gap:8px; justify-content:center; width:100%; max-width:440px;}
  .chip{background:var(--surface); border:1px solid var(--border); color:var(--dim); padding:7px 13px; border-radius:999px; font-size:12px; max-width:100%; overflow:hidden; text-overflow:ellipsis; white-space:nowrap;}

  /* ---------- リーダー ---------- */
  #reader{position:fixed; inset:0; background:var(--reader); display:none; touch-action:none;}
  #stage{position:absolute; inset:0; overflow:hidden;}
  .layer{position:absolute; inset:0; display:flex; align-items:center; justify-content:center; will-change:transform; background:var(--reader);}
  .layer.idle{visibility:hidden;}
  /* 横幅フィット：画像の幅を必ず画面幅に合わせる（縦に長いページは縦パンで全体を読む） */
  .layer img{width:100%; height:auto; max-width:100%; max-height:none; display:block; will-change:transform; transform-origin:center center;}

  /* 上部バー */
  #topbar{position:absolute; left:0; right:0; top:0; z-index:20; display:flex; align-items:center; gap:10px;
    padding:calc(10px + var(--safe-t)) 14px 12px; background:linear-gradient(180deg,rgba(10,10,12,.9),rgba(10,10,12,0));
    transition:transform .22s, opacity .22s;}
  #topbar.hidden{transform:translateY(-108%); opacity:0;}
  #title{flex:1; min-width:0; font-size:14px; overflow:hidden; text-overflow:ellipsis; white-space:nowrap;}
  .tbtn{background:rgba(35,39,51,.9); border:1px solid var(--border); color:var(--text); padding:9px 13px; border-radius:11px; font-size:14px; cursor:pointer; white-space:nowrap;}
  .tbtn:active{background:var(--accent); border-color:var(--accent); color:#fff;}

  /* 下部バー */
  #botbar{position:absolute; left:0; right:0; bottom:0; z-index:20; display:flex; align-items:center; gap:12px;
    padding:12px 18px calc(14px + var(--safe-b)); background:linear-gradient(0deg,rgba(10,10,12,.9),rgba(10,10,12,0));
    transition:transform .22s, opacity .22s;}
  #botbar.hidden{transform:translateY(108%); opacity:0;}
  #counter{font-size:13px; min-width:70px; text-align:center; font-variant-numeric:tabular-nums;}
  #seek{flex:1; accent-color:var(--accent); height:26px;}

  /* ヒント（初回操作ガイド） */
  #hint{position:absolute; left:50%; top:50%; transform:translate(-50%,-50%); z-index:15; pointer-events:none;
    display:flex; flex-direction:column; align-items:center; gap:10px; color:#fff; opacity:0; transition:opacity .3s;
    text-shadow:0 1px 6px rgba(0,0,0,.8); font-size:14px;}
  #hint.show{opacity:.92;}
  #hint .a{font-size:34px; animation:bob 1.4s ease-in-out infinite;}
  @keyframes bob{0%,100%{transform:translateY(4px);}50%{transform:translateY(-6px);}}

  /* ズームインジケータ */
  #zbadge{position:absolute; z-index:16; right:14px; top:calc(56px + var(--safe-t)); background:rgba(0,0,0,.6);
    color:#fff; font-size:12px; padding:5px 9px; border-radius:8px; opacity:0; transition:opacity .2s; pointer-events:none;}
  #zbadge.show{opacity:1;}

  #loading{position:fixed; inset:0; z-index:50; display:none; align-items:center; justify-content:center; flex-direction:column; gap:14px; background:rgba(15,17,21,.9);}
  .spin{width:38px;height:38px;border:3px solid var(--border);border-top-color:var(--accent);border-radius:50%;animation:s 1s linear infinite;}
  @keyframes s{to{transform:rotate(360deg);}}
  #loadMsg{color:var(--dim); font-size:13px;}
  .sr{position:absolute; width:1px; height:1px; overflow:hidden; clip:rect(0 0 0 0);}

  /* 目次 */
  #toc{position:absolute; inset:0; z-index:40; display:none; background:rgba(12,13,17,0.97); flex-direction:column;}
  #toc.open{display:flex;}
  #tocHead{display:flex; align-items:center; gap:12px; padding:calc(14px + var(--safe-t)) 16px 12px; border-bottom:1px solid var(--border);}
  #tocHead h2{margin:0; font-size:19px; font-weight:800;}
  #tocHead h2 .m{color:var(--accent);}
  #tocSub{color:var(--dim); font-size:12px; flex:1;}
  #tocClose{background:var(--surface2); border:1px solid var(--border); color:var(--text); padding:9px 14px; border-radius:11px; font-size:14px;}
  #tocList{flex:1; overflow-y:auto; -webkit-overflow-scrolling:touch; touch-action:pan-y; padding:12px 14px calc(16px + var(--safe-b)); display:flex; flex-direction:column; gap:10px;}
  .tocItem{display:flex; align-items:center; justify-content:space-between; gap:10px; text-align:left;
    background:var(--surface); border:1px solid var(--border); color:var(--text); border-radius:13px; padding:16px; font-size:16px;}
  .tocItem:active{background:var(--surface2);}
  .tocItem.current{border-color:var(--accent); box-shadow:0 0 0 1px var(--accent) inset;}
  .tocItem .ep{font-weight:700;}
  .tocItem .meta{color:var(--dim); font-size:12px; white-space:nowrap;}
</style>
</head>
<body>

<!-- ========== ホーム ========== -->
<div id="home">
  <h1><span class="m">マンガ</span>Reader</h1>
  <div class="sub">for iPhone</div>
  <p>ZIP / CBZ、または写真・ファイルの画像を読み込んで縦画面で読めます。上下スワイプでページ送り、ピンチやダブルタップで拡大。ファイルは端末内だけで処理され、どこにもアップロードされません。</p>
  <div class="drop" id="drop">
    <div class="ic">📖</div>
    <div class="btns">
      <button class="btn primary" id="pickArchive">ZIP / CBZ を開く</button>
      <button class="btn" id="pickImages">画像を選ぶ</button>
    </div>
  </div>
  <div id="recent"></div>
  <input id="fileArchive" class="sr" type="file" accept=".zip,.cbz,application/zip" />
  <input id="fileImages" class="sr" type="file" accept="image/*" multiple />
</div>

<!-- ========== リーダー ========== -->
<div id="reader">
  <div id="stage">
    <div class="layer" id="layerA"><img alt="" /></div>
    <div class="layer" id="layerB"><img alt="" /></div>
  </div>

  <div id="zbadge">100%</div>

  <div id="hint">
    <div class="a">⇅</div>
    <div>上下スワイプでページ送り<br>ダブルタップ / ピンチで拡大</div>
  </div>

  <div id="topbar">
    <button class="tbtn" id="btnHome">☰</button>
    <button class="tbtn" id="btnToc">目次</button>
    <div id="title">—</div>
    <button class="tbtn" id="btnZoomReset">等倍</button>
  </div>

  <div id="botbar">
    <span id="counter">0 / 0</span>
    <input id="seek" type="range" min="0" max="0" value="0" />
  </div>

  <!-- 目次 -->
  <div id="toc">
    <div id="tocHead">
      <h2><span class="m">目次</span></h2>
      <span id="tocSub"></span>
      <button id="tocClose">閉じる</button>
    </div>
    <div id="tocList"></div>
  </div>
</div>

<div id="loading"><div class="spin"></div><div id="loadMsg">読み込み中…</div></div>

<script>
"use strict";

/* ================= 状態 ================= */
const state = { pages:[], index:0, title:"", recent:[], chapters:[], pageChapterStart:null };
const IMG_RE = /\.(jpe?g|png|gif|webp|bmp|avif)$/i;
const len = ()=>state.pages.length;

/* ================= 要素 ================= */
const $ = id=>document.getElementById(id);
const home=$("home"), reader=$("reader"), stage=$("stage"),
      layerA=$("layerA"), layerB=$("layerB"),
      topbar=$("topbar"), botbar=$("botbar"), titleEl=$("title"),
      counter=$("counter"), seek=$("seek"), recentEl=$("recent"),
      loading=$("loading"), loadMsg=$("loadMsg"), zbadge=$("zbadge"), hint=$("hint"),
      btnToc=$("btnToc"), toc=$("toc"), tocList=$("tocList"), tocSub=$("tocSub"), tocClose=$("tocClose");
let curLayer=layerA, nextLayer=layerB;
const imgOf = layer => layer.firstElementChild;

/* ================= ユーティリティ ================= */
function naturalCompare(a,b){
  const ax=[], bx=[];
  a.replace(/(\d+)|(\D+)/g,(_,n,s)=>{ax.push([n||Infinity,s||""]);});
  b.replace(/(\d+)|(\D+)/g,(_,n,s)=>{bx.push([n||Infinity,s||""]);});
  while(ax.length&&bx.length){ const an=ax.shift(),bn=bx.shift();
    const c=(Number(an[0])-Number(bn[0]))||an[1].localeCompare(bn[1],"ja"); if(c) return c; }
  return ax.length-bx.length;
}
function showLoading(m){ loadMsg.textContent=m||"読み込み中…"; loading.style.display="flex"; }
function hideLoading(){ loading.style.display="none"; }
function revokeAll(){ state.pages.forEach(p=>p.url&&URL.revokeObjectURL(p.url)); }

/* ================= 読み込み ================= */
async function openArchive(file){
  showLoading("アーカイブを展開中…");
  try{
    const zip=await JSZip.loadAsync(file); const entries=[];
    zip.forEach((path,entry)=>{ if(!entry.dir && IMG_RE.test(path) && !path.split("/").pop().startsWith(".")) entries.push(entry); });
    if(!entries.length){ hideLoading(); alert("画像が見つかりませんでした。"); return; }
    entries.sort((a,b)=>naturalCompare(a.name,b.name));
    const pages=[];
    for(let i=0;i<entries.length;i++){ loadMsg.textContent=`画像を読み込み中… ${i+1}/${entries.length}`;
      const blob=await entries[i].async("blob"); pages.push({name:entries[i].name,url:URL.createObjectURL(blob)}); }
    startReading(pages, file.name.replace(/\.(zip|cbz)$/i,""));
  }catch(e){ console.error(e); hideLoading(); alert("ファイルを開けませんでした。"); }
}
function openImages(fileList){
  const files=Array.from(fileList).filter(f=>IMG_RE.test(f.name)||(f.type||"").startsWith("image/"));
  if(!files.length){ alert("画像が選ばれていません。"); return; }
  files.sort((a,b)=>naturalCompare(a.name,b.name));
  const pages=files.map(f=>({name:f.name,url:URL.createObjectURL(f)}));
  startReading(pages, files.length===1?files[0].name:`画像 ${files.length} 枚`);
}
function startReading(pages,title){
  revokeAll();
  state.pages=pages; state.index=0; state.title=title||"無題";
  parseChapters();
  addRecent(title); hideLoading();
  home.style.display="none"; reader.style.display="block";
  titleEl.textContent=state.title; seek.max=pages.length-1;
  setLayerImage(curLayer, 0); curLayer.style.transform="translateY(0)"; curLayer.style.zIndex=2;
  curLayer.classList.remove("idle");
  nextLayer.style.zIndex=1; nextLayer.style.transform="translateY(0)"; nextLayer.classList.add("idle");
  viewReset(false);
  updateCounter();
  btnToc.style.display = (state.chapters.length>1) ? "" : "none";
  if(state.chapters.length>1){ openTOC(); } else { showUI(true); maybeHint(); }
}

/* ================= 話（チャプター）解析＋目次 ================= */
function parseChapters(){
  const re=/第\s*0*(\d+)\s*話/;
  const chapters=[]; let prevEp=null;
  state.pages.forEach((p,i)=>{
    const baseName=(p.name||"").split("/").pop();
    const m=re.exec(baseName); const ep=m?parseInt(m[1],10):null;
    if(i===0){ chapters.push({ep,start:i}); prevEp=ep; }
    else if(ep!=null && ep!==prevEp){ chapters.push({ep,start:i}); prevEp=ep; }
  });
  const pcs=new Array(state.pages.length); let ci=0;
  for(let i=0;i<state.pages.length;i++){ while(ci+1<chapters.length && chapters[ci+1].start<=i) ci++; pcs[i]=chapters[ci].start; }
  chapters.forEach((c,k)=>{ c.end=(k+1<chapters.length?chapters[k+1].start:state.pages.length)-1;
    c.count=c.end-c.start+1; c.title=c.ep!=null?`第${c.ep}話`:"本編"; });
  state.chapters=chapters; state.pageChapterStart=pcs;
}
function curChapterStart(){ const a=state.pageChapterStart; return (a&&a[state.index]!=null)?a[state.index]:0; }
function buildTOC(){
  tocSub.textContent=`全${state.chapters.length}話・${len()}ページ`;
  const curCs=curChapterStart();
  tocList.innerHTML="";
  state.chapters.forEach(c=>{
    const it=document.createElement("button"); it.className="tocItem"+(c.start===curCs?" current":"");
    it.innerHTML=`<span class="ep">${c.title}</span><span class="meta">${c.count}ページ・P${c.start+1}〜</span>`;
    it.onclick=()=>jumpToChapter(c.start);
    tocList.appendChild(it);
  });
}
function openTOC(){ buildTOC(); toc.classList.add("open"); }
function closeTOC(){ toc.classList.remove("open"); }
function jumpToChapter(startIdx){ closeTOC(); goTo(startIdx); showUI(false); }

/* ================= 最近開いた ================= */
function addRecent(t){ state.recent=state.recent.filter(r=>r!==t); state.recent.unshift(t); state.recent=state.recent.slice(0,6); renderRecent(); }
function renderRecent(){
  if(!state.recent.length){ recentEl.innerHTML=""; return; }
  recentEl.innerHTML=`<div style="width:100%;color:var(--dim);font-size:12px">最近開いた（この画面を閉じるまで）</div>`;
  state.recent.forEach(t=>{ const c=document.createElement("div"); c.className="chip"; c.textContent=t; recentEl.appendChild(c); });
}

/* ================= ページ設定・カウンタ ================= */
// 縦長ページを上端表示にするためのtyを算出（横幅フィット時の余剰高さの半分）
function topYFor(im){
  const SW=stage.clientWidth, SH=stage.clientHeight;
  const iw=im.naturalWidth||0, ih=im.naturalHeight||0;
  const bh = iw ? SW*ih/iw : (im.clientHeight||0);
  return Math.max(0,(bh-SH)/2);
}
function setLayerImage(layer, idx){
  const im=imgOf(layer);
  im.onload=()=>{ if(layer===curLayer && zoom.scale<=1.01){ zoom.ty=topYFor(im); zoom.tx=0; clampPan(); applyZoom(false); } };
  im.src=state.pages[idx].url;
  im.style.transform=`translate(0px,${topYFor(im)}px) scale(1)`;
}
function updateCounter(){ counter.textContent=`${state.index+1} / ${len()}`; seek.value=state.index; stage.dataset.index=state.index; }
function preload(i){ if(i<0||i>=len()) return; const im=new Image(); im.src=state.pages[i].url; }

/* ================= ズーム ================= */
const zoom={ scale:1, tx:0, ty:0, min:1, max:4.5 };
function applyZoom(animate){
  const im=imgOf(curLayer);
  im.style.transition = animate ? "transform .22s ease-out" : "none";
  im.style.transform = `translate(${zoom.tx}px,${zoom.ty}px) scale(${zoom.scale})`;
  stage.dataset.scale = zoom.scale.toFixed(3);
  $("zbadge").textContent = Math.round(zoom.scale*100)+"%";
  if(zoom.scale>1.01){ zbadge.classList.add("show"); } else { zbadge.classList.remove("show"); }
}
function baseSize(){ const im=imgOf(curLayer); return { w:im.clientWidth, h:im.clientHeight }; }
function clampPan(){
  const {w,h}=baseSize(); const SW=stage.clientWidth, SH=stage.clientHeight;
  const maxX=Math.max(0,(w*zoom.scale-SW)/2), maxY=Math.max(0,(h*zoom.scale-SH)/2);
  zoom.tx=Math.max(-maxX,Math.min(maxX,zoom.tx));
  zoom.ty=Math.max(-maxY,Math.min(maxY,zoom.ty));
}
function resetZoom(animate){ zoom.scale=1; zoom.tx=0; zoom.ty=0; applyZoom(animate); }
// 等倍に戻す（縦長ページは上端表示）
function viewReset(animate){ const im=imgOf(curLayer); zoom.scale=1; zoom.tx=0; zoom.ty=topYFor(im); clampPan(); applyZoom(animate); }
// 現在のパン可動域
function panRoom(){ const {w,h}=baseSize(); const SW=stage.clientWidth,SH=stage.clientHeight;
  return {x:Math.max(0,(w*zoom.scale-SW)/2), y:Math.max(0,(h*zoom.scale-SH)/2)}; }
function zoomAtPoint(px,py,newScale){
  const SW=stage.clientWidth, SH=stage.clientHeight; const C={x:SW/2,y:SH/2};
  newScale=Math.max(zoom.min,Math.min(zoom.max,newScale));
  // 焦点(px,py)固定：局所座標 L=(focal-C-T)/scale → T'=focal-C-L*newScale
  const Lx=(px-C.x-zoom.tx)/zoom.scale, Ly=(py-C.y-zoom.ty)/zoom.scale;
  zoom.scale=newScale; zoom.tx=px-C.x-Lx*newScale; zoom.ty=py-C.y-Ly*newScale;
  clampPan(); applyZoom(true);
}

/* ================= ページ送り（縦スライド） ================= */
let animating=false;
function goTo(i){ // アニメなしジャンプ（スライダー用）
  if(animating) return;
  i=Math.max(0,Math.min(len()-1,i)); if(i===state.index){ return; }
  state.index=i; setLayerImage(curLayer,i);
  curLayer.style.transition="none"; curLayer.style.transform="translateY(0)";
  viewReset(false); updateCounter();
}
// dir:'next'(上スワイプ) | 'prev'(下スワイプ)
function turn(dir){
  if(animating) return;
  const target = dir==="next" ? state.index+1 : state.index-1;
  if(target<0 || target>=len()) { bounce(dir); return; }
  animating=true; hideHint();
  if(zoom.scale>1.01) viewReset(false);   // 拡大中に送る場合は等倍へ
  setLayerImage(nextLayer, target);
  const H=stage.clientHeight;
  const fromNext = dir==="next" ? H : -H;     // 次は下から、前は上から
  const toCur    = dir==="next" ? -H : H;      // 現在は上へ/下へ退場
  nextLayer.classList.remove("idle");          // 入場レイヤーを表示
  nextLayer.style.transition="none";
  nextLayer.style.transform=`translateY(${fromNext}px)`;
  nextLayer.style.zIndex=3; curLayer.style.zIndex=2;
  // 次フレームでアニメ開始
  requestAnimationFrame(()=>requestAnimationFrame(()=>{
    const dur=".3s";
    curLayer.style.transition=`transform ${dur} ease-in-out`;
    nextLayer.style.transition=`transform ${dur} ease-in-out`;
    curLayer.style.transform=`translateY(${toCur}px)`;
    nextLayer.style.transform="translateY(0)";
    let finished=false;
    const done=()=>{
      if(finished) return; finished=true; clearTimeout(guard);
      nextLayer.removeEventListener("transitionend",done);
      // レイヤー入替：退場した旧currentは待機（非表示）にして重なりを防ぐ
      const t=curLayer; curLayer=nextLayer; nextLayer=t;
      curLayer.style.zIndex=2; curLayer.classList.remove("idle");
      nextLayer.style.transition="none"; nextLayer.style.transform="translateY(0)";
      nextLayer.style.zIndex=1; nextLayer.classList.add("idle");
      state.index=target; viewReset(false); updateCounter();
      preload(dir==="next"?target+1:target-1);
      animating=false;
    };
    const guard=setTimeout(done, 480); // transitionend が来ない場合の保険
    nextLayer.addEventListener("transitionend",done);
  }));
}
function bounce(dir){ // 端で軽く跳ねる
  const d = dir==="next" ? -26 : 26;
  curLayer.style.transition="transform .12s ease-out";
  curLayer.style.transform=`translateY(${d}px)`;
  setTimeout(()=>{ curLayer.style.transition="transform .16s ease-out"; curLayer.style.transform="translateY(0)"; },130);
}

/* ================= UI表示/非表示・ヒント ================= */
let uiVisible=true, uiTimer=null;
function showUI(persist){ uiVisible=true; topbar.classList.remove("hidden"); botbar.classList.remove("hidden");
  clearTimeout(uiTimer); if(!persist) uiTimer=setTimeout(hideUI,2600); }
function hideUI(){ uiVisible=false; topbar.classList.add("hidden"); botbar.classList.add("hidden"); }
function toggleUI(){ uiVisible?hideUI():showUI(false); }
function goHome(){ reader.style.display="none"; home.style.display="flex"; }
let hintShown=false;
function maybeHint(){ if(hintShown) return; hintShown=true; hint.classList.add("show"); setTimeout(hideHint,2600); }
function hideHint(){ hint.classList.remove("show"); }

/* ================= ジェスチャ（ズーム＋縦スワイプ） ================= */
let g=null;                 // 現在のジェスチャ状態
let lastTap=0, lastTapX=0, lastTapY=0, tapTimer=null;
function dist(t0,t1){ return Math.hypot(t0.clientX-t1.clientX, t0.clientY-t1.clientY); }
function mid(t0,t1){ return {x:(t0.clientX+t1.clientX)/2, y:(t0.clientY+t1.clientY)/2}; }

stage.addEventListener("touchstart",e=>{
  if(animating){ return; }
  if(e.touches.length===2){
    const [a,b]=e.touches; const m=mid(a,b);
    g={mode:"pinch", startDist:dist(a,b)||1, startScale:zoom.scale,
       C:{x:stage.clientWidth/2,y:stage.clientHeight/2},
       Lx:(m.x-stage.clientWidth/2-zoom.tx)/zoom.scale, Ly:(m.y-stage.clientHeight/2-zoom.ty)/zoom.scale};
    imgOf(curLayer).style.transition="none";
  }else if(e.touches.length===1){
    const t=e.touches[0]; const pr=panRoom();
    g={mode:"one", x0:t.clientX, y0:t.clientY, tx0:zoom.tx, ty0:zoom.ty, t0:Date.now(), moved:false, dx:0, dy:0,
       panY: pr.y>0.5, atTop: zoom.ty>=pr.y-1, atBottom: zoom.ty<=-pr.y+1};
    imgOf(curLayer).style.transition="none";
  }
},{passive:false});

stage.addEventListener("touchmove",e=>{
  if(!g) return;
  e.preventDefault();
  if(g.mode==="pinch" && e.touches.length>=2){
    const [a,b]=e.touches; const m=mid(a,b);
    let ns=g.startScale*(dist(a,b)/g.startDist);
    ns=Math.max(zoom.min,Math.min(zoom.max,ns));
    zoom.scale=ns;
    zoom.tx=m.x-g.C.x-g.Lx*ns; zoom.ty=m.y-g.C.y-g.Ly*ns;
    clampPan(); applyZoom(false);
  }else if(g.mode==="one"){
    const t=e.touches[0]; g.dx=t.clientX-g.x0; g.dy=t.clientY-g.y0;
    if(Math.abs(g.dx)>6||Math.abs(g.dy)>6) g.moved=true;
    if(zoom.scale>1.01 || g.panY){
      // パン（拡大中、または縦長ページの縦スクロール）
      zoom.tx=g.tx0+g.dx; zoom.ty=g.ty0+g.dy; clampPan(); applyZoom(false);
    }else{
      // 等倍・全体が収まるページ：縦スワイプで指に追従（軽く）
      if(Math.abs(g.dy)>Math.abs(g.dx)){
        const follow=Math.max(-140,Math.min(140,g.dy*0.5));
        curLayer.style.transition="none"; curLayer.style.transform=`translateY(${follow}px)`;
      }
    }
  }
},{passive:false});

stage.addEventListener("touchend",e=>{
  if(!g){ return; }
  if(g.mode==="pinch"){
    if(zoom.scale<=1.02){ viewReset(true); } else { clampPan(); applyZoom(true); }
    g=null; return;
  }
  // one-finger 終了
  const wasZoomed = zoom.scale>1.01;
  const {dx,dy,moved,t0,panY,atTop,atBottom}=g; const dt=Date.now()-t0; g=null;

  if(moved){
    const vertical = Math.abs(dy)>Math.abs(dx);
    const isSwipe = vertical && (Math.abs(dy)>70 || (Math.abs(dy)>40 && dt<260));
    if(wasZoomed){ clampPan(); applyZoom(true); return; }   // 拡大中のドラッグ=パン確定（送らない）
    if(panY){
      // 縦長ページ：縦スクロール。端を越えるスワイプでページ送り
      clampPan(); applyZoom(true);
      if(isSwipe){
        if(dy<0 && atBottom) turn("next");        // 下端でさらに上スワイプ→次
        else if(dy>0 && atTop) turn("prev");      // 上端でさらに下スワイプ→前
      }
      return;
    }
    // 全体が収まるページ：縦スワイプでページ送り
    curLayer.style.transition="transform .18s ease-out"; curLayer.style.transform="translateY(0)"; // 追従を戻す
    if(isSwipe) turn(dy<0 ? "next" : "prev");   // 上スワイプ=次 / 下スワイプ=前
    return;
  }

  // 移動なし = タップ（拡大中でも有効）
  const now=Date.now();
  const px=e.changedTouches[0].clientX, py=e.changedTouches[0].clientY;
  const near = Math.abs(px-lastTapX)<28 && Math.abs(py-lastTapY)<28;
  if(now-lastTap<300 && near){
    clearTimeout(tapTimer); lastTap=0;
    if(zoom.scale>1.01) viewReset(true); else zoomAtPoint(px,py,2.6);   // ダブルタップでズーム切替
  }else{
    lastTap=now; lastTapX=px; lastTapY=py;
    tapTimer=setTimeout(()=>{ toggleUI(); }, 270);   // シングルタップ確定でUI切替
  }
},{passive:false});

// iOS Safari自身のピンチズームを抑止
["gesturestart","gesturechange","gestureend"].forEach(ev=>document.addEventListener(ev,e=>e.preventDefault()));
stage.addEventListener("dblclick",e=>e.preventDefault());

/* ================= 入力（ファイル/ボタン/キー） ================= */
$("pickArchive").onclick=()=>$("fileArchive").click();
$("pickImages").onclick =()=>$("fileImages").click();
$("fileArchive").onchange=e=>{ if(e.target.files[0]) openArchive(e.target.files[0]); e.target.value=""; };
$("fileImages").onchange =e=>{ if(e.target.files.length) openImages(e.target.files); e.target.value=""; };

const drop=$("drop");
["dragenter","dragover","drop"].forEach(ev=>drop.addEventListener(ev,e=>e.preventDefault()));
window.addEventListener("dragover",e=>e.preventDefault());
window.addEventListener("drop",e=>e.preventDefault());
drop.addEventListener("drop",e=>{ const files=e.dataTransfer.files; if(!files||!files.length) return;
  const arc=Array.from(files).find(f=>/\.(zip|cbz)$/i.test(f.name)); if(arc) openArchive(arc); else openImages(files); });

$("btnHome").onclick=goHome;
btnToc.onclick=()=>{ toc.classList.contains("open") ? closeTOC() : openTOC(); };
tocClose.onclick=closeTOC;
$("btnZoomReset").onclick=()=>{ viewReset(true); showUI(false); };
seek.oninput=e=>{ goTo(parseInt(e.target.value,10)); showUI(false); };
seek.addEventListener("touchstart",e=>e.stopPropagation());

window.addEventListener("keydown",e=>{
  if(reader.style.display==="none") return;
  if(toc.classList.contains("open")){ if(e.key==="Escape"||e.key==="t"||e.key==="T") closeTOC(); return; }
  if(e.key==="t"||e.key==="T"){ btnToc.click(); return; }
  if(e.key==="ArrowDown"||e.key===" "||e.key==="PageDown"){ turn("next"); e.preventDefault(); }
  else if(e.key==="ArrowUp"||e.key==="PageUp"){ turn("prev"); e.preventDefault(); }
  else if(e.key==="Home"){ goTo(0); }
  else if(e.key==="End"){ goTo(len()-1); }
  else if(e.key==="+"||e.key==="="){ zoomAtPoint(stage.clientWidth/2,stage.clientHeight/2, zoom.scale*1.4); }
  else if(e.key==="-"){ zoomAtPoint(stage.clientWidth/2,stage.clientHeight/2, zoom.scale/1.4); }
  else if(e.key==="0"){ viewReset(true); }
  else if(e.key==="Escape"){ goHome(); }
});

window.addEventListener("resize",()=>{ if(reader.style.display!=="none"){ clampPan(); applyZoom(false); } });
window.addEventListener("orientationchange",()=>{ setTimeout(()=>{ clampPan(); applyZoom(false); },200); });

renderRecent();
</script>
</body>
</html>
