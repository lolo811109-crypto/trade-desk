/* Trade Desk UI widgets: pipeline strip, position Gantt, night/day header. Real paper-trade data only. */
(function(){
const TZ='Asia/Shanghai',H=3600;
const css=`.tdx{background:#0D1117;color:#E6EDF3;border-radius:18px;padding:14px 14px 12px;margin-bottom:12px;border:1px solid #21262D;font-family:Inter,-apple-system,"PingFang SC",sans-serif;position:relative;overflow:hidden}
.tdx h4{font-size:14px;font-weight:700;display:flex;justify-content:space-between;align-items:center;margin-bottom:10px}
.tdx .m{font:500 10px "JetBrains Mono",monospace;color:#7D8590;letter-spacing:.06em;text-transform:uppercase}
.tdx .nodes{display:flex;justify-content:space-between;position:relative;margin:0 2px}
.tdx .line{position:absolute;left:18px;right:18px;top:17px;height:2px;background:linear-gradient(90deg,var(--x),#30363D);opacity:.7}
.tdx .line:after{content:"";position:absolute;top:-2px;width:30px;height:6px;border-radius:3px;background:var(--x);filter:blur(3px);animation:tdflow 2.6s linear infinite}
@keyframes tdflow{from{left:0}to{left:100%}}
.tdx .nd{position:relative;z-index:1;display:flex;flex-direction:column;align-items:center;width:20%}
.tdx .ic{width:36px;height:36px;border-radius:12px;background:#161B22;border:1.5px solid var(--x);display:grid;place-items:center;font-size:16px;box-shadow:0 0 12px color-mix(in srgb,var(--x) 40%,transparent)}
.tdx .nd b{font:700 15px "JetBrains Mono",monospace;margin-top:5px}
.tdx .nd span{font-size:11px;color:#C9D1D9}.tdx .nd small{font:500 9px "JetBrains Mono",monospace;color:#7D8590}
.tdx .pills{display:flex;gap:6px;overflow:hidden;margin-top:12px;height:24px;mask-image:linear-gradient(90deg,#000 80%,transparent)}
.tdx .pl{flex:none;font:700 10px "JetBrains Mono",monospace;padding:5px 8px;border-radius:99px;white-space:nowrap;animation:tdpop .5s cubic-bezier(.3,1.6,.5,1) both}
.tdx .pl.ok{background:#0F2E22;color:#3FB950}.tdx .pl.k{background:#3A1619;color:#F85149}.tdx .pl.n{background:#1F2630;color:#C9D1D9}.tdx .pl.w{background:#2E2410;color:#E3B341}
@keyframes tdpop{from{transform:translateY(8px) scale(.6);opacity:0}}
.tdx .gt{position:relative;margin-top:4px}
.tdx .ax{position:relative;height:16px;margin-left:30%;font:500 9px "JetBrains Mono",monospace;color:#7D8590}
.tdx .ax i{position:absolute;transform:translateX(-50%);font-style:normal}
.tdx .gr{display:flex;align-items:center;height:20px}
.tdx .gl{width:30%;font:500 10px "JetBrains Mono",monospace;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;padding-right:6px;color:#C9D1D9}
.tdx .gb{position:relative;flex:1;height:100%;border-left:1px solid #21262D}
.tdx .bar{position:absolute;top:4px;height:12px;border-radius:4px;min-width:3px}
.tdx .bar.op{box-shadow:0 0 8px currentColor}
.tdx .bar em{position:absolute;left:100%;margin-left:4px;font:700 9px "JetBrains Mono",monospace;font-style:normal;white-space:nowrap;top:-1px}
.tdx .now{position:absolute;top:0;bottom:0;width:2px;background:var(--x);box-shadow:0 0 8px var(--x);z-index:1;animation:tdbl 1.6s ease-in-out infinite}
.tdx .now:before{content:"NOW";position:absolute;top:-1px;left:4px;font:700 8px "JetBrains Mono",monospace;color:var(--x)}
@keyframes tdbl{50%{opacity:.45}}
.tdx .grid{position:absolute;top:16px;bottom:0;width:1px;background:#161B22}
.tdx .emp{font-size:12px;color:#7D8590;padding:8px 0}`;
const st=document.createElement('style');st.textContent=css;document.head.appendChild(st);
const tail=t=>t?'·'+String(t).replace(/pump$/,'').slice(-4):'';
const esc=s=>String(s??'').replace(/[<>&"]/g,c=>({'<':'&lt;','>':'&gt;','&':'&amp;','"':'&quot;'}[c]));
const ymd=ts=>new Date(ts*1000).toLocaleDateString('sv-SE',{timeZone:TZ});
const hm=ts=>new Date(ts*1000).toLocaleTimeString('zh-CN',{hour12:false,timeZone:TZ,hour:'2-digit',minute:'2-digit'});
const name=t=>t.sym!=null?(t.sym=='water'&&String(t.token||'').toLowerCase().startsWith('0x6567c4')?'#矿泉水 water':t.sym):(t.title||'');
function reason(r){r=(r&&r[0])||'';const M=[[/LP/i,'LP未锁'],[/top10|holder/i,'前十集中'],[/tax/i,'税率'],[/mint/i,'增发'],[/freez/i,'冻结'],[/honeypot/i,'貔貅'],[/dev/i,'开发者持仓']];for(const[a,b]of M)if(a.test(r))return b;return r.slice(0,10)||'风控'}
async function j(u){try{const r=await fetch(u+(u.includes('?')?'&':'?')+'t='+Date.now());return r.ok?await r.json():null}catch(e){return null}}
/* Normalise trades of one account into pipeline events (today, UTC+8) */
function events(s,tag){if(!s||!Array.isArray(s.trades))return null;const today=ymd(Date.now()/1000),poly=!!s.wallets&&!s.pos;
 const c={sig:0,pass:0,kill:0,copy:0,tpsl:0,settle:0,pnl:0},ev=[],cost={};
 for(const t of s.trades){const nm=name(t),tl=poly?'':' '+tail(t.token),pre=tag?tag+' ':'',td=ymd(t.ts)==today;
  if(t.side=='BUY'&&t.action=='buy'){const k=t.wallet+':'+(t.token||t.title);cost[k]=(cost[k]||0)+(t.stake||10)}
  if(!td||t.side=='FILTER')continue;
  if(t.side=='BUY'){c.sig++;
   if(t.action=='buy'){c.pass++;c.copy++;ev.push({ts:t.ts,c:'ok',x:pre+'PASS '+esc(nm.slice(0,18))+tl})}
   else if(t.action=='kill-risk'||(poly&&t.why=='rule')){c.kill++;ev.push({ts:t.ts,c:'k',x:pre+'KILL '+esc(reason(t.reasons||[t.why]))+tl})}
   else if(/liq/.test(t.action||'')){c.kill++;ev.push({ts:t.ts,c:'k',x:pre+'KILL 流动性'+tl})}
  }else if(t.side=='SELL'&&t.action){const k=t.wallet+':'+(t.token||t.title),bs=cost[k];let p=t.pnl;
   const pct=(p!=null&&t.proceeds!=null&&t.proceeds-p>0)?p/(t.proceeds-p):null,ps=pct==null?'':(pct>=0?'+':'')+Math.round(pct*100)+'% ';
   if(p!=null)c.pnl+=p;
   if(/stop|take|tp|time|sl/i.test(t.action)){c.tpsl++;const lab=/take|tp/i.test(t.action)?'止盈':/time/i.test(t.action)?'时间止损':'止损';ev.push({ts:t.ts,c:p>=0?'ok':'k',x:pre+ps+lab+' '+esc(nm.slice(0,14))+tl})}
   else{c.settle++;ev.push({ts:t.ts,c:p==null?'n':p>=0?'ok':'k',x:pre+ps+'跟卖 '+esc(nm.slice(0,14))+tl})}}}
 return{c,ev}}
function merge(list){const o={c:{sig:0,pass:0,kill:0,copy:0,tpsl:0,settle:0,pnl:0},ev:[]};let any=false;for(const e of list){if(!e)continue;any=true;for(const k in o.c)o.c[k]+=e.c[k];o.ev.push(...e.ev)}return any?o:null}
function pipe(el,e,color,title){if(!el)return;el.className='tdx';el.style.setProperty('--x',color);
 if(!e){el.innerHTML=`<h4>今日流程 <span class="m">PIPELINE</span></h4><div class="emp">暂无数据（数据文件读取失败）</div>`;return}
 const c=e.c,N=[['📡','信号',c.sig,'SIGNAL'],['🛡️','风控',c.pass+'/'+c.kill,'PASS/KILL'],['🤝','跟单',c.copy,'COPY'],['🎯','止盈止损',c.tpsl,'TP/SL'],['🏁','结算',c.settle,(c.pnl>=0?'+':'−')+'$'+Math.abs(c.pnl).toFixed(2)]];
 const ev=e.ev.sort((a,b)=>b.ts-a.ts).slice(0,14);
 el.innerHTML=`<h4>${title||'今日流程'} <span class="m">PIPELINE · TODAY UTC+8</span></h4><div class="nodes"><div class="line"></div>${N.map(n=>`<div class="nd"><div class="ic">${n[0]}</div><b>${n[2]}</b><span>${n[1]}</span><small>${n[3]}</small></div>`).join('')}</div>
 <div class="pills">${ev.length?ev.map((p,i)=>`<span class="pl ${p.c}" style="animation-delay:${i*.08}s" title="${hm(p.ts)}">${hm(p.ts)} ${p.x}</span>`).join(''):'<span class="pl n">今天还没有事件</span>'}</div>`}
/* Gantt of open + recent closed positions over last 24h */
function bars(s){if(!s||!Array.isArray(s.trades))return null;const now=Date.now()/1000,poly=!!s.wallets&&!s.pos,open={},out=[];
 for(const t of s.trades){const k=t.wallet+':'+(t.token||(t.title+'|'+t.outcome));
  if(t.side=='BUY'&&t.action=='buy'){if(!open[k])open[k]={k,t0:t.ts,nm:name(t),tok:t.token,pnl:0,cost:0};open[k].cost+=t.stake||(poly?0:10)}
  else if(t.side=='SELL'&&t.action&&open[k]){const o=open[k];o.pnl+=t.pnl||0;o.t1=t.ts}}
 const live=new Set();
 if(s.pos)for(const[k,p]of Object.entries(s.pos)){live.add(k);const o=open[k]||{k,t0:p.opened,nm:p.sym,tok:p.tok,pnl:0};o.t0=p.opened||o.t0;o.cost=p.cost;o.pnl=(o.pnl||0)+(p.mark!=null&&p.qty?p.qty*p.mark-p.cost:0);o.t1=null;o.open=true;out.push(o)}
 if(s.wallets)for(const[w,W]of Object.entries(s.wallets))for(const p of Object.values(W.pos||{})){const k=w+':'+p.title+'|'+p.outcome;live.add(k);const o=open[k]||{k,t0:null,nm:p.title};o.nm=(p.title||'')+' · '+(p.outcome||'');o.cost=p.cost;o.pnl=p.mark!=null?p.shares*p.mark-p.cost:0;o.t1=null;o.open=true;if(o.t0)out.push(o)}
 for(const o of Object.values(open))if(!live.has(o.k)&&o.t1)out.push(o);
 return out.filter(o=>(o.t1||now)>now-24*H).sort((a,b)=>(b.open-a.open)||((b.t1||now)-(a.t1||now))).slice(0,24).sort((a,b)=>a.t0-b.t0)}
function gantt(el,s,color){if(!el)return;el.className='tdx';el.style.setProperty('--x',color);const B=bars(s);
 const hd=`<h4>持仓时间线 <span class="m">${B?B.filter(b=>b.open).length+' OPEN · '+B.filter(b=>!b.open).length+' CLOSED · 24H':'24H'}</span></h4>`;
 if(!B){el.innerHTML=hd+'<div class="emp">暂无数据（数据文件读取失败）</div>';return}
 if(!B.length){el.innerHTML=hd+'<div class="emp">过去 24 小时没有持仓</div>';return}
 const draw=()=>{const now=Date.now()/1000,t0=now-24*H,X=t=>Math.max(0,Math.min(100,(t-t0)/(24*H)*100));
  let ax='',gd='';for(let h=0;h<=24;h+=6){const t=t0+h*H;ax+=`<i style="left:${h/24*100}%">${h==24?'':hm(t)}</i>`}
  for(let h=6;h<24;h+=6)gd+=`<div class="grid" style="left:calc(30% + ${h/24*70}%)"></div>`;
  const rows=B.map(b=>{const pct=b.cost?b.pnl/b.cost:0,cl=pct>=0.005?'#3FB950':pct<=-0.005?'#F85149':'#8B949E',a=Math.max(b.t0,t0),l=X(a),w=Math.max(.6,X(b.t1||now)-l);
   return `<div class="gr"><div class="gl" title="${esc(b.nm)}">${b.open?'●':'○'} ${esc(String(b.nm).slice(0,22))} ${b.tok?tail(b.tok):''}</div><div class="gb"><div class="bar ${b.open?'op':''}" style="left:${l}%;width:${w}%;background:${cl};color:${cl};opacity:${b.open?1:.55}">${l+w<78?`<em>${pct>=0?'+':''}${(pct*100).toFixed(0)}%</em>`:''}</div></div></div>`}).join('');
  el.innerHTML=hd+`<div class="gt">${gd}<div class="ax">${ax}</div>${rows}<div class="now" style="left:calc(30% + 70% - 2px)"></div></div><div class="m" style="margin-top:6px">● 持仓中 ○ 已平仓 · 颜色=盈亏 · 时间 UTC+8</div>`};
 draw()}
/* Equity delta over current shift window; sum of accounts */
function shift(){const p=new Intl.DateTimeFormat('en-GB',{timeZone:TZ,hour:'numeric',minute:'2-digit',hourCycle:'h23'}).formatToParts(new Date()),h=+p.find(x=>x.type=='hour').value,m=+p.find(x=>x.type=='minute').value;
 const night=h>=22||h<8,now=Date.now()/1000,sinceH=night?(h>=22?h-22:h+2):h-8;return{night,start:now-(sinceH*H+m*60),h,m}}
function eqAt(hist,t){let v=null;for(const p of hist){if(p[0]<=t)v=p[1];else{if(v==null)v=p[1];break}}return v}
function shiftPnl(accts,start){const now=Date.now()/1000,hs=accts.map(a=>(a&&a.history||[]).filter(p=>p&&p.length>1)).filter(h=>h.length);if(!hs.length)return null;
 const pts=[];for(let i=0;i<=40;i++){const t=start+(now-start)*i/40;let s=0;for(const h of hs){const b=eqAt(h,start),v=eqAt(h,t);if(b!=null&&v!=null)s+=v-b}pts.push([t,s])}return{pnl:pts.at(-1)[1],pts,n:hs.length}}
function sparkSvg(pts,c){if(!pts||pts.length<2)return'';const ys=pts.map(p=>p[1]),lo=Math.min(0,...ys),hi=Math.max(0,...ys),r=hi-lo||1,Y=v=>22-(v-lo)/r*20;
 return `<svg viewBox="0 0 100 24" preserveAspectRatio="none" style="width:90px;height:24px"><line x1="0" x2="100" y1="${Y(0)}" y2="${Y(0)}" stroke="#555" stroke-dasharray="2 2" vector-effect="non-scaling-stroke"/><path d="M${pts.map((p,i)=>(i/(pts.length-1)*100).toFixed(1)+','+Y(p[1]).toFixed(1)).join('L')}" fill="none" stroke="${c}" stroke-width="1.8" vector-effect="non-scaling-stroke"/></svg>`}
window.TDUI={j,events,merge,pipe,gantt,shift,shiftPnl,sparkSvg,hm};
/* Auto-mount on market pages: <div id="tdpipe" data-src="..." data-color="..."> */
async function mount(){const P=document.getElementById('tdpipe'),G=document.getElementById('tdgantt');if(!P&&!G)return;const src=(P||G).dataset.src,c=(P||G).dataset.color;
 const s=await j(src);pipe(P,events(s),c);gantt(G,s,c)}
if(document.readyState=='loading')addEventListener('DOMContentLoaded',mount);else mount();setInterval(mount,60000);
})();
