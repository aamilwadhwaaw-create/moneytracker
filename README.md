import { useState, useMemo, useEffect } from "react";

const KEY = "mark_money_v2";
const load = () => { try { return JSON.parse(localStorage.getItem(KEY)) || {}; } catch { return {}; } };
const save = d => { try { localStorage.setItem(KEY, JSON.stringify(d)); } catch {} };

const DEFAULT_PIN = "1234";
const CURRENCIES = [
  {code:"INR",symbol:"₹"},{code:"USD",symbol:"$"},{code:"EUR",symbol:"€"},
  {code:"AED",symbol:"د.إ"},{code:"GBP",symbol:"£"},{code:"SAR",symbol:"﷼"},
];
const ENTRY_TYPES = [
  {key:"gave",      label:"I Gave",         emoji:"💸",color:"#f43f5e",bg:"#f43f5e18"},
  {key:"received",  label:"I Received",     emoji:"💰",color:"#10b981",bg:"#10b98118"},
  {key:"bill",      label:"Bill Submitted", emoji:"📄",color:"#f59e0b",bg:"#f59e0b18"},
  {key:"reimbursed",label:"Reimbursed",     emoji:"✅",color:"#3b82f6",bg:"#3b82f618"},
  {key:"expense",   label:"My Expense",     emoji:"🛒",color:"#8b5cf6",bg:"#8b5cf618"},
  {key:"income",    label:"My Income",      emoji:"🏦",color:"#06b6d4",bg:"#06b6d418"},
];
const EXPENSE_CATS = ["Food 🍔","Transport 🚗","Shopping 🛍","Bills 💡","Health 🏥","Entertainment 🎬","Education 📚","Other 📦"];
const PARTY_CATS  = ["Friend","Relative","Company","Colleague","Client","Other"];
const todayStr = () => new Date().toISOString().slice(0,10);
const fmt = (n, cur="INR") => {
  const s = CURRENCIES.find(c=>c.code===cur)?.symbol||"";
  return s+Number(n||0).toLocaleString("en-IN",{minimumFractionDigits:2,maximumFractionDigits:2});
};
const fmtDate = d => { try { return new Date(d+"T00:00:00").toLocaleDateString("en-IN",{day:"2-digit",month:"short",year:"numeric"}); } catch { return d; } };

// ── Pill ──
function Pill({color,children}){
  return <span style={{background:color+"22",color,fontSize:10,fontWeight:700,padding:"2px 8px",borderRadius:20}}>{children}</span>;
}

// ── Toast ──
function Toast({msg,type}){
  const colors={success:"#10b981",error:"#f43f5e",info:"#3b82f6"};
  return (
    <div style={{position:"fixed",top:64,left:"50%",transform:"translateX(-50%)",background:colors[type]||"#10b981",color:"#fff",padding:"9px 20px",borderRadius:30,fontSize:13,fontWeight:700,zIndex:9999,boxShadow:"0 4px 20px #0005",whiteSpace:"nowrap"}}>
      {msg}
    </div>
  );
}

// ── Modal ──
function Modal({title,onClose,children}){
  return (
    <div style={{position:"fixed",inset:0,background:"#000000bb",zIndex:500,display:"flex",alignItems:"flex-end",justifyContent:"center"}}
      onClick={e=>e.target===e.currentTarget&&onClose()}>
      <div style={{background:"#1a1a2e",borderRadius:"24px 24px 0 0",width:"100%",maxWidth:520,maxHeight:"92vh",overflowY:"auto",border:"1px solid #ffffff12",borderBottom:"none",animation:"slideUp .28s cubic-bezier(.34,1.56,.64,1)"}}>
        <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",padding:"18px 20px 14px",borderBottom:"1px solid #ffffff0d",position:"sticky",top:0,background:"#1a1a2e",zIndex:1}}>
          <span style={{fontWeight:800,fontSize:16,color:"#f1f5f9"}}>{title}</span>
          <button onClick={onClose} style={{background:"#ffffff12",border:"none",color:"#94a3b8",width:30,height:30,borderRadius:"50%",cursor:"pointer",fontSize:15}}>✕</button>
        </div>
        <div style={{padding:"0 20px 40px"}}>{children}</div>
      </div>
    </div>
  );
}

// ── Label ──
const Lbl = ({children}) => <p style={{color:"#475569",fontSize:11,fontWeight:700,letterSpacing:.8,marginBottom:5,marginTop:14}}>{children.toUpperCase()}</p>;
const Inp = (props) => <input {...props} style={{width:"100%",background:"#0f172a",border:"1px solid #ffffff14",borderRadius:12,padding:"11px 14px",color:"#f1f5f9",fontSize:14,outline:"none",display:"block",...props.style}}/>;
const Sel = ({children,...props}) => <select {...props} style={{width:"100%",background:"#0f172a",border:"1px solid #ffffff14",borderRadius:12,padding:"11px 14px",color:"#f1f5f9",fontSize:14,outline:"none",display:"block",...props.style}}>{children}</select>;
const PrimaryBtn = ({children,...props}) => <button {...props} style={{display:"block",width:"100%",background:"linear-gradient(135deg,#6366f1,#8b5cf6)",color:"#fff",border:"none",borderRadius:14,padding:"13px",fontSize:15,fontWeight:800,cursor:"pointer",marginTop:14,...props.style}}>{children}</button>;

// ── PIN Screen ──
function PinScreen({onUnlock}){
  const [pin,setPin]=useState("");
  const [err,setErr]=useState("");
  const correct=(load().pin)||DEFAULT_PIN;

  function tryUnlock(p){
    if(p===correct) onUnlock();
    else{ setErr("Wrong PIN"); setPin(""); }
  }
  function press(d){
    if(pin.length>=correct.length) return;
    const np=pin+d; setPin(np); setErr("");
    if(np.length===correct.length) setTimeout(()=>tryUnlock(np),80);
  }

  return (
    <div style={{minHeight:"100vh",background:"linear-gradient(160deg,#0f0f1a,#1a1a3e,#0f0f1a)",display:"flex",alignItems:"center",justifyContent:"center",flexDirection:"column",padding:20}}>
      <style>{`@keyframes slideUp{from{transform:translateY(100%);opacity:0}to{transform:translateY(0);opacity:1}}`}</style>
      <div style={{textAlign:"center",marginBottom:36}}>
        <div style={{fontSize:52,marginBottom:10}}>💳</div>
        <h1 style={{fontSize:26,fontWeight:900,color:"#f1f5f9",letterSpacing:-0.5,margin:0}}>Money Tracker</h1>
        <p style={{fontSize:11,color:"#6366f1",fontWeight:700,letterSpacing:2,margin:"2px 0 0"}}>BY MARK</p>
      </div>
      <div style={{background:"#ffffff08",border:"1px solid #ffffff14",borderRadius:24,padding:"28px 24px",width:"min(320px,100%)",textAlign:"center"}}>
        <p style={{color:"#64748b",fontSize:13,marginBottom:20}}>Enter your PIN</p>
        <div style={{display:"flex",gap:10,justifyContent:"center",marginBottom:6}}>
          {Array.from({length:correct.length},(_,i)=>(
            <div key={i} style={{width:13,height:13,borderRadius:"50%",border:"2px solid",borderColor:i<pin.length?"#6366f1":"#ffffff25",background:i<pin.length?"#6366f1":"transparent",transition:"all .15s"}}/>
          ))}
        </div>
        {err && <p style={{color:"#f43f5e",fontSize:12,margin:"6px 0"}}>{err}</p>}
        <div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:8,marginTop:18}}>
          {[1,2,3,4,5,6,7,8,9,"",0,"⌫"].map((k,i)=>(
            <button key={i} onClick={()=>k==="⌫"?setPin(p=>{setErr("");return p.slice(0,-1);}):k!==""?press(String(k)):null}
              style={{height:54,borderRadius:12,border:"1px solid #ffffff10",background:k===""?"transparent":"#ffffff08",color:k==="⌫"?"#f43f5e":"#f1f5f9",fontSize:k==="⌫"?18:20,fontWeight:600,cursor:k===""?"default":"pointer"}}>
              {k}
            </button>
          ))}
        </div>
        <p style={{color:"#334155",fontSize:11,marginTop:16}}>Default PIN: <b style={{color:"#6366f1"}}>1234</b></p>
      </div>
    </div>
  );
}

// ── MAIN APP ──
export default function App(){
  const [unlocked,setUnlocked]=useState(false);
  const [data,setData]=useState(()=>{
    const s=load();
    return{pin:s.pin||DEFAULT_PIN,parties:s.parties||[],entries:s.entries||[],budget:s.budget||{},lastBackup:s.lastBackup||null,defaultCurrency:s.defaultCurrency||"INR"};
  });
  const [tab,setTab]=useState("home");
  const [modal,setModal]=useState(null);
  const [toast,setToast]=useState(null);
  const [filterParty,setFilterParty]=useState(null);
  const [filterType,setFilterType]=useState(null);
  const [search,setSearch]=useState("");
  const [period,setPeriod]=useState("month");

  const [eForm,setEForm]=useState({type:"gave",party:"",amount:"",currency:"INR",desc:"",date:todayStr(),ref:"",cat:""});
  const [pForm,setPForm]=useState({name:"",category:"Friend",phone:"",note:""});
  const [pinForm,setPinForm]=useState({current:"",next:"",confirm:""});
  const [pinErr,setPinErr]=useState("");
  const [budgetVal,setBudgetVal]=useState("");

  function showToast(msg,type="success"){ setToast({msg,type}); setTimeout(()=>setToast(null),2200); }
  function persist(next){ setData(next); save({...load(),...next}); }
  function closeModal(){ setModal(null); setPinErr(""); }

  function addEntry(){
    if(!eForm.amount||isNaN(parseFloat(eForm.amount))){ showToast("Enter a valid amount","error"); return; }
    const needsParty=["gave","received","bill","reimbursed"].includes(eForm.type);
    if(needsParty&&!eForm.party){ showToast("Select a party","error"); return; }
    const entry={id:Date.now(),...eForm,amount:parseFloat(eForm.amount),createdAt:new Date().toISOString()};
    persist({...data,entries:[entry,...data.entries]});
    setEForm({type:eForm.type,party:"",amount:"",currency:data.defaultCurrency,desc:"",date:todayStr(),ref:"",cat:""});
    closeModal(); showToast("Entry saved ✓");
  }
  function deleteEntry(id){ if(window.confirm("Delete this entry?")){ persist({...data,entries:data.entries.filter(e=>e.id!==id)}); showToast("Deleted","error"); } }
  function addParty(){
    if(!pForm.name.trim()){ showToast("Enter a name","error"); return; }
    persist({...data,parties:[...data.parties,{id:Date.now(),...pForm,name:pForm.name.trim()}]});
    setPForm({name:"",category:"Friend",phone:"",note:""}); closeModal(); showToast("Party added ✓");
  }
  function changePin(){
    if(pinForm.current!==data.pin){ setPinErr("Current PIN is wrong"); return; }
    if(pinForm.next.length<4){ setPinErr("At least 4 digits required"); return; }
    if(pinForm.next!==pinForm.confirm){ setPinErr("PINs don't match"); return; }
    persist({...data,pin:pinForm.next});
    setPinForm({current:"",next:"",confirm:""}); closeModal(); showToast("PIN updated ✓");
  }
  function exportBackup(){
    const blob=new Blob([JSON.stringify({...data,exportedAt:new Date().toISOString()},null,2)],{type:"application/json"});
    const a=document.createElement("a"); a.href=URL.createObjectURL(blob);
    a.download=`moneytracker_${todayStr()}.json`; a.click();
    persist({...data,lastBackup:new Date().toISOString()}); showToast("Backup downloaded ✓");
  }
  function importBackup(e){
    const file=e.target.files[0]; if(!file) return;
    const r=new FileReader();
    r.onload=ev=>{ try{ const d=JSON.parse(ev.target.result); persist({...data,...d}); showToast("Backup restored ✓"); closeModal(); }catch{ showToast("Invalid file","error"); }};
    r.readAsText(file);
  }

  const periodStart = useMemo(()=>{
    if(period==="week"){ const d=new Date(),day=d.getDay(),diff=d.getDate()-day+(day===0?-6:1); return new Date(new Date().setDate(diff)).toISOString().slice(0,10); }
    if(period==="month") return new Date().toISOString().slice(0,8)+"01";
    return "2000-01-01";
  },[period]);

  const filtered = useMemo(()=>data.entries.filter(e=>{
    if(e.date<periodStart) return false;
    if(filterParty&&e.party!==filterParty) return false;
    if(filterType&&e.type!==filterType) return false;
    if(search&&!e.party?.toLowerCase().includes(search.toLowerCase())&&!e.desc?.toLowerCase().includes(search.toLowerCase())) return false;
    return true;
  }),[data.entries,periodStart,filterParty,filterType,search]);

  const ledger   = useMemo(()=>filtered.filter(e=>["gave","received","bill","reimbursed"].includes(e.type)),[filtered]);
  const expenses = useMemo(()=>filtered.filter(e=>["expense","income"].includes(e.type)),[filtered]);

  const summary = useMemo(()=>{
    const t={lent:0,returned:0,expenses:0,income:0};
    filtered.forEach(e=>{
      if(e.currency!==data.defaultCurrency) return;
      if(e.type==="gave"||e.type==="bill") t.lent+=e.amount;
      if(e.type==="received"||e.type==="reimbursed") t.returned+=e.amount;
      if(e.type==="expense") t.expenses+=e.amount;
      if(e.type==="income") t.income+=e.amount;
    });
    return t;
  },[filtered,data.defaultCurrency]);

  const partyBal = useMemo(()=>{
    const m={};
    data.entries.forEach(e=>{
      if(!e.party) return;
      if(!m[e.party]) m[e.party]={};
      const c=e.currency||"INR";
      if(!m[e.party][c]) m[e.party][c]=0;
      if(e.type==="gave"||e.type==="bill") m[e.party][c]+=e.amount;
      else if(e.type==="received"||e.type==="reimbursed") m[e.party][c]-=e.amount;
    });
    return m;
  },[data.entries]);

  const catSpend = useMemo(()=>{
    const m={};
    expenses.filter(e=>e.type==="expense").forEach(e=>{ const c=e.cat||"Other 📦"; if(!m[c])m[c]=0; m[c]+=e.amount; });
    return Object.entries(m).sort((a,b)=>b[1]-a[1]);
  },[expenses]);

  const totalExp=catSpend.reduce((s,[,v])=>s+v,0);
  const monthBudget=data.budget?.total||0;
  const budgetPct=monthBudget>0?Math.min(100,(totalExp/monthBudget)*100):0;
  const cur=data.defaultCurrency;
  const net=summary.lent-summary.returned;

  if(!unlocked) return <PinScreen onUnlock={()=>setUnlocked(true)}/>;

  // ── Entry Row ──
  function EntryRow({e}){
    const et=ENTRY_TYPES.find(t=>t.key===e.type)||ENTRY_TYPES[0];
    const [open,setOpen]=useState(false);
    return (
      <div onClick={()=>setOpen(x=>!x)} style={{background:"#0f172a",border:"1px solid #ffffff0d",borderRadius:14,marginBottom:8,overflow:"hidden",cursor:"pointer"}}>
        <div style={{display:"flex",gap:10,alignItems:"center",padding:"11px 13px"}}>
          <div style={{width:36,height:36,borderRadius:10,background:et.bg,display:"flex",alignItems:"center",justifyContent:"center",fontSize:17,flexShrink:0}}>{et.emoji}</div>
          <div style={{flex:1,minWidth:0}}>
            <div style={{display:"flex",justifyContent:"space-between",alignItems:"center"}}>
              <span style={{color:"#f1f5f9",fontWeight:700,fontSize:13,overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap"}}>{e.party||e.cat||et.label}</span>
              <span style={{color:et.color,fontWeight:800,fontSize:13,flexShrink:0,marginLeft:8}}>{fmt(e.amount,e.currency||cur)}</span>
            </div>
            <div style={{display:"flex",justifyContent:"space-between",marginTop:1}}>
              <span style={{color:"#475569",fontSize:11}}>{et.label}{e.desc?` · ${e.desc}`:""}</span>
              <span style={{color:"#334155",fontSize:11}}>{fmtDate(e.date)}</span>
            </div>
          </div>
        </div>
        {open && (
          <div style={{padding:"0 13px 11px",borderTop:"1px solid #ffffff08"}}>
            <div style={{display:"flex",gap:6,flexWrap:"wrap",margin:"8px 0"}}>
              {e.ref&&<Pill color="#6366f1">Ref: {e.ref}</Pill>}
              {e.cat&&<Pill color="#8b5cf6">{e.cat}</Pill>}
              <Pill color={et.color}>{et.label}</Pill>
            </div>
            <button onClick={ev=>{ev.stopPropagation();deleteEntry(e.id);}}
              style={{background:"#f43f5e18",border:"1px solid #f43f5e44",color:"#f43f5e",padding:"5px 12px",borderRadius:8,fontSize:12,fontWeight:700,cursor:"pointer"}}>
              🗑 Delete
            </button>
          </div>
        )}
      </div>
    );
  }

  function Empty({icon,text,sub}){
    return <div style={{textAlign:"center",padding:"36px 16px",color:"#334155"}}><div style={{fontSize:38,marginBottom:8}}>{icon}</div><p style={{color:"#64748b",fontWeight:600}}>{text}</p>{sub&&<p style={{fontSize:12,marginTop:4}}>{sub}</p>}</div>;
  }

  function SearchBar({ph="Search…"}){
    return <div style={{padding:"10px 12px 6px"}}><input value={search} onChange={e=>setSearch(e.target.value)} placeholder={ph} style={{width:"100%",background:"#0f172a",border:"1px solid #ffffff0d",borderRadius:12,padding:"9px 13px",color:"#f1f5f9",fontSize:13,outline:"none"}}/></div>;
  }

  function TypeFilter({types}){
    return (
      <div style={{display:"flex",gap:5,padding:"0 12px 8px",overflowX:"auto"}}>
        <button onClick={()=>setFilterType(null)} style={{padding:"4px 11px",borderRadius:20,border:"1px solid",borderColor:!filterType?"#6366f1":"#ffffff10",background:!filterType?"#6366f118":"transparent",color:!filterType?"#6366f1":"#475569",fontSize:11,fontWeight:700,cursor:"pointer",whiteSpace:"nowrap"}}>All</button>
        {types.map(t=>{ const et=ENTRY_TYPES.find(x=>x.key===t);
          return <button key={t} onClick={()=>setFilterType(filterType===t?null:t)} style={{padding:"4px 11px",borderRadius:20,border:"1px solid",borderColor:filterType===t?et.color:"#ffffff10",background:filterType===t?et.color+"18":"transparent",color:filterType===t?et.color:"#475569",fontSize:11,fontWeight:700,cursor:"pointer",whiteSpace:"nowrap"}}>{et.emoji} {et.label}</button>;
        })}
      </div>
    );
  }

  function PeriodBar(){
    return (
      <div style={{display:"flex",gap:6,padding:"10px 12px 6px"}}>
        {["week","month","all"].map(p=>(
          <button key={p} onClick={()=>setPeriod(p)} style={{flex:1,padding:"6px 0",borderRadius:10,border:"1px solid",borderColor:period===p?"#6366f1":"#ffffff10",background:period===p?"#6366f118":"transparent",color:period===p?"#6366f1":"#475569",fontSize:11,fontWeight:700,cursor:"pointer"}}>
            {p==="all"?"All Time":p==="month"?"This Month":"This Week"}
          </button>
        ))}
      </div>
    );
  }

  const needsParty=["gave","received","bill","reimbursed"].includes(eForm.type);

  const TABS=[
    {key:"home",icon:"🏠",label:"Home"},
    {key:"ledger",icon:"📒",label:"Ledger"},
    {key:"expenses",icon:"💳",label:"Expenses"},
    {key:"parties",icon:"👥",label:"Parties"},
    {key:"settings",icon:"⚙️",label:"Settings"},
  ];

  return (
    <div style={{maxWidth:520,margin:"0 auto",minHeight:"100vh",background:"#0f0f1a",color:"#f1f5f9",fontFamily:"'Segoe UI',system-ui,sans-serif"}}>
      <style>{`@keyframes slideUp{from{transform:translateY(100%);opacity:0}to{transform:translateY(0);opacity:1}} @keyframes pulse{0%,100%{box-shadow:0 0 0 0 rgba(99,102,241,.4)}50%{box-shadow:0 0 0 8px rgba(99,102,241,0)}}`}</style>

      {/* Header */}
      <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",padding:"14px 14px 10px",background:"#0f0f1a",position:"sticky",top:0,zIndex:50,borderBottom:"1px solid #ffffff08"}}>
        <div><p style={{fontWeight:900,fontSize:16,color:"#f1f5f9",margin:0}}>Money Tracker</p><p style={{fontSize:9,color:"#6366f1",fontWeight:700,letterSpacing:2,margin:0}}>BY MARK</p></div>
        <div style={{display:"flex",gap:6}}>
          <button onClick={()=>setPeriod(p=>p==="month"?"week":p==="week"?"all":"month")} style={{background:"#ffffff08",border:"none",color:"#94a3b8",padding:"5px 10px",borderRadius:8,fontSize:11,cursor:"pointer",fontWeight:700}}>
            {period==="month"?"Month":period==="week"?"Week":"All"}
          </button>
          <button onClick={()=>setUnlocked(false)} style={{background:"#ffffff08",border:"none",color:"#94a3b8",padding:"5px 9px",borderRadius:8,fontSize:14,cursor:"pointer"}}>🔒</button>
        </div>
      </div>

      {/* Body */}
      <div style={{paddingBottom:85}}>

        {/* ── HOME ── */}
        {tab==="home" && (
          <div>
            <div style={{background:"linear-gradient(135deg,#1e1b4b,#312e81)",margin:"10px 12px 12px",borderRadius:18,padding:"20px",border:"1px solid #4338ca44"}}>
              <p style={{color:"#a5b4fc",fontSize:11,fontWeight:700,letterSpacing:1.5,margin:"0 0 3px"}}>NET OUTSTANDING</p>
              <p style={{color:net>0?"#f43f5e":"#10b981",fontSize:32,fontWeight:900,letterSpacing:-1,margin:"0 0 2px"}}>{fmt(Math.abs(net),cur)}</p>
              <p style={{color:"#818cf8",fontSize:11,margin:0}}>{net>0?"You are owed":"You owe"} this amount</p>
              <div style={{display:"flex",gap:12,marginTop:14,paddingTop:14,borderTop:"1px solid #ffffff12"}}>
                <div style={{flex:1}}><p style={{color:"#94a3b8",fontSize:10,margin:"0 0 2px"}}>Total Lent</p><p style={{color:"#f43f5e",fontWeight:700,fontSize:14,margin:0}}>{fmt(summary.lent,cur)}</p></div>
                <div style={{flex:1}}><p style={{color:"#94a3b8",fontSize:10,margin:"0 0 2px"}}>Total Recovered</p><p style={{color:"#10b981",fontWeight:700,fontSize:14,margin:0}}>{fmt(summary.returned,cur)}</p></div>
              </div>
            </div>

            <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:8,padding:"0 12px",marginBottom:10}}>
              <div style={{background:"#0f172a",border:"1px solid #ffffff0d",borderRadius:14,padding:"13px"}}>
                <p style={{color:"#94a3b8",fontSize:10,margin:"0 0 4px"}}>💸 Spent</p>
                <p style={{color:"#8b5cf6",fontWeight:800,fontSize:16,margin:0}}>{fmt(summary.expenses,cur)}</p>
                {monthBudget>0&&<p style={{color:"#475569",fontSize:10,margin:"3px 0 0"}}>{budgetPct.toFixed(0)}% of budget</p>}
              </div>
              <div style={{background:"#0f172a",border:"1px solid #ffffff0d",borderRadius:14,padding:"13px"}}>
                <p style={{color:"#94a3b8",fontSize:10,margin:"0 0 4px"}}>🏦 Income</p>
                <p style={{color:"#06b6d4",fontWeight:800,fontSize:16,margin:0}}>{fmt(summary.income,cur)}</p>
                <p style={{color:summary.income-summary.expenses>=0?"#10b981":"#f43f5e",fontSize:10,margin:"3px 0 0"}}>{summary.income-summary.expenses>=0?"▲":"▼"} {fmt(Math.abs(summary.income-summary.expenses),cur)} net</p>
              </div>
            </div>

            {monthBudget>0&&(
              <div style={{background:"#0f172a",border:"1px solid #ffffff0d",borderRadius:14,padding:"14px",margin:"0 12px 10px"}}>
                <div style={{display:"flex",justifyContent:"space-between",marginBottom:6}}><span style={{color:"#94a3b8",fontSize:11}}>📊 Monthly Budget</span><span style={{color:budgetPct>90?"#f43f5e":"#f1f5f9",fontSize:11,fontWeight:700}}>{fmt(totalExp,cur)} / {fmt(monthBudget,cur)}</span></div>
                <div style={{height:7,background:"#ffffff10",borderRadius:6,overflow:"hidden"}}><div style={{height:"100%",width:budgetPct+"%",background:budgetPct>90?"linear-gradient(90deg,#f43f5e,#be123c)":"linear-gradient(90deg,#6366f1,#8b5cf6)",borderRadius:6,transition:"width .5s"}}/></div>
                {budgetPct>90&&<p style={{color:"#f43f5e",fontSize:11,margin:"5px 0 0"}}>⚠️ Budget almost exhausted!</p>}
              </div>
            )}

            <PeriodBar/>

            <div style={{padding:"4px 12px 0"}}>
              <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:8}}>
                <p style={{color:"#475569",fontSize:11,fontWeight:700,letterSpacing:1,margin:0}}>RECENT ACTIVITY</p>
                <button onClick={()=>setTab("ledger")} style={{background:"none",border:"none",color:"#6366f1",fontSize:12,cursor:"pointer",fontWeight:700}}>View all →</button>
              </div>
              {data.entries.slice(0,6).map(e=><EntryRow key={e.id} e={e}/>)}
              {data.entries.length===0&&<Empty icon="📭" text="No entries yet" sub="Tap + to add your first entry"/>}
            </div>
          </div>
        )}

        {/* ── LEDGER ── */}
        {tab==="ledger"&&(
          <div>
            <SearchBar ph="Search ledger…"/>
            <TypeFilter types={["gave","received","bill","reimbursed"]}/>
            <PeriodBar/>
            <div style={{padding:"0 12px"}}>
              {ledger.map(e=><EntryRow key={e.id} e={e}/>)}
              {ledger.length===0&&<Empty icon="📒" text="No ledger entries" sub="Track money you lend or borrow"/>}
            </div>
          </div>
        )}

        {/* ── EXPENSES ── */}
        {tab==="expenses"&&(
          <div>
            <SearchBar ph="Search expenses…"/>
            <TypeFilter types={["expense","income"]}/>
            <PeriodBar/>
            {catSpend.length>0&&(
              <div style={{background:"#0f172a",border:"1px solid #ffffff0d",borderRadius:14,margin:"0 12px 10px",padding:"14px"}}>
                <p style={{color:"#475569",fontSize:11,fontWeight:700,letterSpacing:1,margin:"0 0 10px"}}>CATEGORY BREAKDOWN</p>
                {catSpend.map(([cat,amt])=>{
                  const pct=totalExp>0?(amt/totalExp*100):0;
                  return (
                    <div key={cat} style={{marginBottom:8}}>
                      <div style={{display:"flex",justifyContent:"space-between",marginBottom:3}}><span style={{color:"#e2e8f0",fontSize:12}}>{cat}</span><span style={{color:"#8b5cf6",fontSize:12,fontWeight:700}}>{fmt(amt,cur)}</span></div>
                      <div style={{height:4,background:"#ffffff0a",borderRadius:4,overflow:"hidden"}}><div style={{height:"100%",width:pct+"%",background:"linear-gradient(90deg,#6366f1,#8b5cf6)",borderRadius:4}}/></div>
                    </div>
                  );
                })}
              </div>
            )}
            <div style={{padding:"0 12px"}}>
              {expenses.map(e=><EntryRow key={e.id} e={e}/>)}
              {expenses.length===0&&<Empty icon="🛒" text="No expense entries" sub="Track your daily spending"/>}
            </div>
          </div>
        )}

        {/* ── PARTIES ── */}
        {tab==="parties"&&(
          <div>
            <SearchBar ph="Search parties…"/>
            <div style={{padding:"0 12px"}}>
              <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",margin:"6px 0 10px"}}>
                <p style={{color:"#475569",fontSize:11,fontWeight:700,letterSpacing:1,margin:0}}>PARTY BALANCES</p>
                <button onClick={()=>setModal("party")} style={{background:"#6366f118",border:"1px solid #6366f1",color:"#6366f1",padding:"5px 12px",borderRadius:8,fontSize:12,fontWeight:700,cursor:"pointer"}}>+ Add Party</button>
              </div>
              {data.parties.filter(p=>!search||p.name.toLowerCase().includes(search.toLowerCase())).map(p=>{
                const bal=partyBal[p.name]||{};
                return (
                  <div key={p.id} style={{background:"#0f172a",border:"1px solid #ffffff0d",borderRadius:14,padding:"13px",marginBottom:8,display:"flex",alignItems:"center",gap:11}}>
                    <div style={{width:40,height:40,borderRadius:"50%",background:"linear-gradient(135deg,#6366f1,#8b5cf6)",display:"flex",alignItems:"center",justifyContent:"center",fontWeight:800,fontSize:15,color:"#fff",flexShrink:0}}>{p.name[0].toUpperCase()}</div>
                    <div style={{flex:1}}>
                      <p style={{color:"#f1f5f9",fontWeight:700,fontSize:13,margin:0}}>{p.name}</p>
                      <p style={{color:"#475569",fontSize:11,margin:0}}>{p.category}{p.phone?` · ${p.phone}`:""}</p>
                    </div>
                    <div style={{textAlign:"right"}}>
                      {Object.entries(bal).length>0?Object.entries(bal).map(([c,b])=>(
                        <p key={c} style={{color:b>0?"#f43f5e":"#10b981",fontWeight:700,fontSize:12,margin:0}}>{fmt(Math.abs(b),c)}</p>
                      )):<p style={{color:"#334155",fontSize:11,margin:0}}>No entries</p>}
                    </div>
                    <button onClick={()=>{ if(window.confirm("Remove "+p.name+"?")) persist({...data,parties:data.parties.filter(x=>x.id!==p.id)}); }}
                      style={{background:"none",border:"none",color:"#334155",fontSize:13,cursor:"pointer",padding:"4px"}}>🗑</button>
                  </div>
                );
              })}
              {data.parties.length===0&&<Empty icon="👥" text="No parties yet" sub="Add people to track balances"/>}
            </div>
          </div>
        )}

        {/* ── SETTINGS ── */}
        {tab==="settings"&&(
          <div style={{padding:"0 12px"}}>
            {[
              {title:"General",rows:[
                {icon:"💱",label:"Default Currency",right:<span style={{color:"#6366f1",fontSize:13,fontWeight:700}}>{cur}</span>,onClick:()=>setModal("currency")},
                {icon:"📊",label:"Set Monthly Budget",onClick:()=>{ setBudgetVal(String(monthBudget||"")); setModal("budget"); }},
              ]},
              {title:"Security",rows:[
                {icon:"🔐",label:"Change PIN",onClick:()=>setModal("pin")},
                {icon:"🔒",label:"Lock App",onClick:()=>setUnlocked(false)},
              ]},
              {title:"Data",rows:[
                {icon:"⬆️",label:"Export Backup",onClick:exportBackup},
                {icon:"⬇️",label:"Import Backup",right:<label style={{color:"#6366f1",fontSize:13,fontWeight:700,cursor:"pointer"}}>Choose File<input type="file" accept=".json" style={{display:"none"}} onChange={importBackup}/></label>},
              ]},
            ].map(sec=>(
              <div key={sec.title} style={{marginBottom:18}}>
                <p style={{color:"#334155",fontSize:10,fontWeight:700,letterSpacing:1.5,margin:"18px 2px 8px"}}>{sec.title.toUpperCase()}</p>
                <div style={{background:"#0f172a",border:"1px solid #ffffff0d",borderRadius:14,overflow:"hidden"}}>
                  {sec.rows.map((r,i)=>(
                    <div key={i} onClick={r.onClick} style={{display:"flex",alignItems:"center",gap:12,padding:"13px 14px",borderBottom:i<sec.rows.length-1?"1px solid #ffffff06":"none",cursor:r.onClick?"pointer":"default"}}>
                      <span style={{fontSize:17}}>{r.icon}</span>
                      <span style={{flex:1,color:"#e2e8f0",fontSize:13}}>{r.label}</span>
                      {r.right||( r.onClick&&<span style={{color:"#334155"}}>›</span>)}
                    </div>
                  ))}
                </div>
              </div>
            ))}
            <div style={{background:"#0f172a",border:"1px solid #ffffff0d",borderRadius:14,padding:"16px",marginBottom:20,textAlign:"center"}}>
              <p style={{color:"#f1f5f9",fontWeight:900,fontSize:16,margin:"0 0 2px"}}>Money Tracker</p>
              <p style={{color:"#6366f1",fontSize:10,fontWeight:700,letterSpacing:2,margin:"0 0 8px"}}>BY MARK</p>
              <p style={{color:"#334155",fontSize:12,margin:0}}>All data saved on your device · Works offline</p>
              {data.lastBackup&&<p style={{color:"#1e293b",fontSize:11,marginTop:6}}>Last backup: {new Date(data.lastBackup).toLocaleString()}</p>}
            </div>
          </div>
        )}
      </div>

      {/* FAB */}
      <button onClick={()=>{ setEForm({type:"gave",party:"",amount:"",currency:cur,desc:"",date:todayStr(),ref:"",cat:""}); setModal("entry"); }}
        style={{position:"fixed",bottom:74,right:16,width:52,height:52,borderRadius:"50%",background:"linear-gradient(135deg,#6366f1,#8b5cf6)",color:"#fff",fontSize:26,border:"none",cursor:"pointer",boxShadow:"0 4px 20px #6366f155",zIndex:60,display:"flex",alignItems:"center",justifyContent:"center",animation:"pulse 2s infinite"}}>
        +
      </button>

      {/* Tab Bar */}
      <div style={{position:"fixed",bottom:0,left:"50%",transform:"translateX(-50%)",width:"min(520px,100vw)",background:"#0a0a14",borderTop:"1px solid #ffffff0d",display:"flex",zIndex:50}}>
        {TABS.map(t=>(
          <button key={t.key} onClick={()=>{ setTab(t.key); setSearch(""); setFilterParty(null); setFilterType(null); }}
            style={{flex:1,padding:"9px 0 13px",background:"none",border:"none",cursor:"pointer",display:"flex",flexDirection:"column",alignItems:"center",gap:2}}>
            <span style={{fontSize:17}}>{t.icon}</span>
            <span style={{fontSize:9,fontWeight:tab===t.key?700:500,color:tab===t.key?"#6366f1":"#334155"}}>{t.label}</span>
          </button>
        ))}
      </div>

      {toast&&<Toast msg={toast.msg} type={toast.type}/>}

      {/* ── MODALS ── */}
      {modal==="entry"&&(
        <Modal title="New Entry" onClose={closeModal}>
          <div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:6,marginTop:14}}>
            {ENTRY_TYPES.map(t=>(
              <button key={t.key} onClick={()=>setEForm(f=>({...f,type:t.key}))}
                style={{padding:"10px 4px",borderRadius:12,border:"2px solid",borderColor:eForm.type===t.key?t.color:"#ffffff10",background:eForm.type===t.key?t.bg:"transparent",cursor:"pointer",display:"flex",flexDirection:"column",alignItems:"center",gap:2}}>
                <span style={{fontSize:19}}>{t.emoji}</span>
                <span style={{fontSize:9,color:eForm.type===t.key?t.color:"#475569",fontWeight:700,textAlign:"center",lineHeight:1.2}}>{t.label}</span>
              </button>
            ))}
          </div>
          {needsParty&&<><Lbl>Party</Lbl><Sel value={eForm.party} onChange={e=>setEForm(f=>({...f,party:e.target.value}))}><option value="">Select…</option>{data.parties.map(p=><option key={p.id} value={p.name}>{p.name}</option>)}</Sel>{data.parties.length===0&&<p style={{color:"#f59e0b",fontSize:11,margin:"-8px 0 4px"}}>⚠️ Add parties first from the Parties tab</p>}</>}
          {(eForm.type==="expense"||eForm.type==="income")&&<><Lbl>Category</Lbl><div style={{display:"flex",flexWrap:"wrap",gap:5,marginBottom:4}}>{EXPENSE_CATS.map(c=><button key={c} onClick={()=>setEForm(f=>({...f,cat:c}))} style={{padding:"4px 9px",borderRadius:20,border:"1px solid",borderColor:eForm.cat===c?"#8b5cf6":"#ffffff10",background:eForm.cat===c?"#8b5cf618":"transparent",color:eForm.cat===c?"#8b5cf6":"#475569",fontSize:10,fontWeight:700,cursor:"pointer"}}>{c}</button>)}</div></>}
          <div style={{display:"flex",gap:8}}>
            <div style={{flex:2}}><Lbl>Amount</Lbl><Inp type="number" min="0" step="0.01" placeholder="0.00" value={eForm.amount} onChange={e=>setEForm(f=>({...f,amount:e.target.value}))}/></div>
            <div style={{flex:1}}><Lbl>Currency</Lbl><Sel value={eForm.currency} onChange={e=>setEForm(f=>({...f,currency:e.target.value}))}>{CURRENCIES.map(c=><option key={c.code} value={c.code}>{c.symbol} {c.code}</option>)}</Sel></div>
          </div>
          <Lbl>Description</Lbl><Inp placeholder="Optional note…" value={eForm.desc} onChange={e=>setEForm(f=>({...f,desc:e.target.value}))}/>
          <div style={{display:"flex",gap:8}}>
            <div style={{flex:1}}><Lbl>Date</Lbl><Inp type="date" value={eForm.date} onChange={e=>setEForm(f=>({...f,date:e.target.value}))}/></div>
            {needsParty&&<div style={{flex:1}}><Lbl>Ref No.</Lbl><Inp placeholder="Optional" value={eForm.ref} onChange={e=>setEForm(f=>({...f,ref:e.target.value}))}/></div>}
          </div>
          <PrimaryBtn onClick={addEntry}>Save Entry ✓</PrimaryBtn>
        </Modal>
      )}

      {modal==="party"&&(
        <Modal title="Add Party" onClose={closeModal}>
          <Lbl>Full Name</Lbl><Inp placeholder="Name or company…" value={pForm.name} onChange={e=>setPForm(f=>({...f,name:e.target.value}))}/>
          <Lbl>Category</Lbl>
          <div style={{display:"flex",flexWrap:"wrap",gap:6,marginBottom:4}}>
            {PARTY_CATS.map(c=><button key={c} onClick={()=>setPForm(f=>({...f,category:c}))} style={{padding:"5px 11px",borderRadius:20,border:"1px solid",borderColor:pForm.category===c?"#6366f1":"#ffffff10",background:pForm.category===c?"#6366f118":"transparent",color:pForm.category===c?"#6366f1":"#475569",fontSize:11,fontWeight:700,cursor:"pointer"}}>{c}</button>)}
          </div>
          <Lbl>Phone (optional)</Lbl><Inp placeholder="+91 98765 43210" value={pForm.phone} onChange={e=>setPForm(f=>({...f,phone:e.target.value}))} type="tel"/>
          <Lbl>Note (optional)</Lbl><Inp placeholder="Extra info…" value={pForm.note} onChange={e=>setPForm(f=>({...f,note:e.target.value}))}/>
          <PrimaryBtn onClick={addParty}>Add Party ✓</PrimaryBtn>
        </Modal>
      )}

      {modal==="pin"&&(
        <Modal title="Change PIN" onClose={closeModal}>
          <Lbl>Current PIN</Lbl><Inp type="password" inputMode="numeric" maxLength={6} value={pinForm.current} onChange={e=>{ setPinErr(""); setPinForm(f=>({...f,current:e.target.value})); }}/>
          <Lbl>New PIN</Lbl><Inp type="password" inputMode="numeric" maxLength={6} value={pinForm.next} onChange={e=>{ setPinErr(""); setPinForm(f=>({...f,next:e.target.value})); }}/>
          <Lbl>Confirm New PIN</Lbl><Inp type="password" inputMode="numeric" maxLength={6} value={pinForm.confirm} onChange={e=>{ setPinErr(""); setPinForm(f=>({...f,confirm:e.target.value})); }}/>
          {pinErr&&<p style={{color:"#f43f5e",fontSize:12,margin:"4px 0"}}>{pinErr}</p>}
          <PrimaryBtn onClick={changePin}>Update PIN ✓</PrimaryBtn>
        </Modal>
      )}

      {modal==="currency"&&(
        <Modal title="Default Currency" onClose={closeModal}>
          <div style={{paddingTop:8}}>
            {CURRENCIES.map(c=>(
              <div key={c.code} onClick={()=>{ persist({...data,defaultCurrency:c.code}); closeModal(); showToast("Currency → "+c.code); }}
                style={{display:"flex",alignItems:"center",gap:14,padding:"13px 0",borderBottom:"1px solid #ffffff06",cursor:"pointer",background:data.defaultCurrency===c.code?"#6366f10a":"transparent"}}>
                <span style={{fontSize:20,width:36,textAlign:"center"}}>{c.symbol}</span>
                <span style={{flex:1,color:"#f1f5f9",fontWeight:600,fontSize:13}}>{c.code}</span>
                {data.defaultCurrency===c.code&&<span style={{color:"#6366f1",fontWeight:700}}>✓</span>}
              </div>
            ))}
          </div>
        </Modal>
      )}

      {modal==="budget"&&(
        <Modal title="Monthly Budget" onClose={closeModal}>
          <Lbl>Total Monthly Limit ({cur})</Lbl>
          <Inp type="number" placeholder="e.g. 50000" value={budgetVal} onChange={e=>setBudgetVal(e.target.value)}/>
          <p style={{color:"#334155",fontSize:12,margin:"4px 0 8px"}}>Get a warning when you're close to the limit.</p>
          <PrimaryBtn onClick={()=>{ persist({...data,budget:{total:parseFloat(budgetVal)||0}}); closeModal(); showToast("Budget saved ✓"); }}>Save Budget ✓</PrimaryBtn>
        </Modal>
      )}
    </div>
  );
}
