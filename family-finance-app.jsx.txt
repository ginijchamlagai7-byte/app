import { useState, useEffect, useCallback } from "react";

// ============================================================
// BIKRAM SAMVAT CALENDAR UTILITY
// ============================================================
const BS_MONTHS = ["Baisakh","Jestha","Ashadh","Shrawan","Bhadra","Ashwin","Kartik","Mangsir","Poush","Magh","Falgun","Chaitra"];
const BS_MONTHS_NP = ["वैशाख","जेठ","असाढ","श्रावण","भाद्र","आश्विन","कार्तिक","मंसिर","पुष","माघ","फाल्गुन","चैत्र"];
const NEPALI_DIGITS = ["०","१","२","३","४","५","६","७","८","९"];

const toNepaliNum = (n) => String(n).split("").map(d => NEPALI_DIGITS[+d] ?? d).join("");

// BS to AD conversion data (year -> days in each month)
const BS_CALENDAR_DATA = {
  2080: [31,32,31,32,31,30,30,30,29,29,30,30],
  2081: [31,31,32,31,31,31,30,29,30,29,30,30],
  2082: [31,32,31,32,31,30,30,30,29,29,30,31],
  2083: [30,32,31,32,31,30,30,30,29,30,30,30],
};

function adToBS(adDate) {
  // Simple approximate conversion (for display purposes)
  const ad = new Date(adDate);
  const adYear = ad.getFullYear();
  const adMonth = ad.getMonth() + 1;
  const adDay = ad.getDate();
  // BS is ~56.7 years ahead; approximate
  let bsYear = adYear + 56;
  let bsMonth = adMonth + 8;
  let bsDay = adDay + 17;
  if (bsDay > 32) { bsDay -= 32; bsMonth += 1; }
  if (bsMonth > 12) { bsMonth -= 12; bsYear += 1; }
  return { year: bsYear, month: bsMonth, day: bsDay };
}

function getTodayBS() {
  return adToBS(new Date());
}

function formatBS(bs) {
  return `${bs.year}-${String(bs.month).padStart(2,"0")}-${String(bs.day).padStart(2,"0")}`;
}

function formatBSNepali(bs) {
  return `${toNepaliNum(bs.year)} ${BS_MONTHS_NP[bs.month-1]} ${toNepaliNum(bs.day)}`;
}

// ============================================================
// MOCK DATA
// ============================================================
const INCOME_CATEGORIES = ["Daily Sales","Product Sales","Service Income","Wholesale Income","Retail Income","Rent Income","Interest Income","Investment Income","Miscellaneous Income"];
const HOUSEHOLD_EXPENSE_CATS = ["Groceries","Vegetables","Fruits","Rice & Food Items","Gas Cylinder","Electricity","Water Bill","Internet","Mobile Recharge","Rent","Fuel","Vehicle Maintenance","Healthcare","Education","Clothing","Entertainment","Travel","Gifts","Household Maintenance","Miscellaneous"];
const BUSINESS_EXPENSE_CATS = ["Inventory Purchase","Employee Salary","Office Rent","Marketing","Transportation","Packaging","Equipment","Repairs","Utilities","Tax","Banking Charges","Miscellaneous Business"];
const PAYMENT_METHODS = ["Cash","eSewa","Khalti","Bank Transfer","Cheque","Card","Credit"];

const today = getTodayBS();

const INITIAL_TRANSACTIONS = [
  { id:1, type:"income", bsDate:formatBS(today), category:"Daily Sales", amount:15500, source:"Shop", customer:"Walk-in", payment:"Cash", description:"Morning sales", createdBy:"Admin" },
  { id:2, type:"income", bsDate:formatBS(today), category:"Product Sales", amount:8200, source:"Online", customer:"Ram Sharma", payment:"eSewa", description:"Electronics", createdBy:"Admin" },
  { id:3, type:"expense", bsDate:formatBS(today), category:"Groceries", amount:3200, vendor:"Bhatbhateni", payment:"Cash", description:"Monthly grocery", createdBy:"Admin" },
  { id:4, type:"expense", bsDate:formatBS(today), category:"Electricity", amount:1850, vendor:"NEA", payment:"eSewa", description:"Monthly bill", createdBy:"Admin" },
  { id:5, type:"income", bsDate:formatBS({...today, day: today.day-1}), category:"Service Income", amount:5000, source:"Repair", customer:"Sita Devi", payment:"Cash", description:"AC repair", createdBy:"Staff" },
  { id:6, type:"expense", bsDate:formatBS({...today, day: today.day-1}), category:"Fuel", amount:2500, vendor:"HP Petrol Pump", payment:"Cash", description:"Bike fuel", createdBy:"Staff" },
  { id:7, type:"expense", bsDate:formatBS({...today, day: today.day-1}), category:"Inventory Purchase", amount:12000, vendor:"Supplier Co.", payment:"Bank Transfer", description:"Stock replenishment", createdBy:"Admin" },
  { id:8, type:"income", bsDate:formatBS({...today, day: today.day-2}), category:"Wholesale Income", amount:25000, source:"Wholesale", customer:"ABC Traders", payment:"Cheque", description:"Bulk order", createdBy:"Admin" },
];

const INITIAL_BUDGETS = [
  { category:"Groceries", limit:20000, type:"household" },
  { category:"Electricity", limit:3000, type:"household" },
  { category:"Fuel", limit:10000, type:"household" },
  { category:"Internet", limit:2000, type:"household" },
  { category:"Healthcare", limit:5000, type:"household" },
  { category:"Inventory Purchase", limit:50000, type:"business" },
  { category:"Employee Salary", limit:80000, type:"business" },
  { category:"Marketing", limit:15000, type:"business" },
];

const CAT_COLORS = {
  "Daily Sales":"#10b981","Product Sales":"#3b82f6","Service Income":"#8b5cf6","Wholesale Income":"#f59e0b",
  "Retail Income":"#06b6d4","Rent Income":"#ec4899","Interest Income":"#14b8a6","Investment Income":"#6366f1",
  "Groceries":"#ef4444","Vegetables":"#22c55e","Electricity":"#eab308","Fuel":"#f97316",
  "Healthcare":"#e11d48","Education":"#7c3aed","Inventory Purchase":"#0ea5e9","Employee Salary":"#64748b",
  "Marketing":"#d946ef","Transportation":"#84cc16",
};

const getCatColor = (cat) => CAT_COLORS[cat] || "#6b7280";

// ============================================================
// ICONS (SVG)
// ============================================================
const Icon = ({ name, size=20, color="currentColor" }) => {
  const icons = {
    dashboard: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><rect x="3" y="3" width="7" height="7" rx="1"/><rect x="14" y="3" width="7" height="7" rx="1"/><rect x="3" y="14" width="7" height="7" rx="1"/><rect x="14" y="14" width="7" height="7" rx="1"/></svg>,
    income: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><path d="M12 2v20M17 5H9.5a3.5 3.5 0 000 7h5a3.5 3.5 0 010 7H6"/></svg>,
    expense: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><path d="M3 3h18v4H3zM3 10h18v11H3z"/><path d="M8 10v11M16 10v11"/></svg>,
    budget: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><circle cx="12" cy="12" r="10"/><path d="M12 6v6l4 2"/></svg>,
    reports: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><path d="M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/><polyline points="10 9 9 9 8 9"/></svg>,
    settings: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><circle cx="12" cy="12" r="3"/><path d="M19.4 15a1.65 1.65 0 00.33 1.82l.06.06a2 2 0 010 2.83 2 2 0 01-2.83 0l-.06-.06a1.65 1.65 0 00-1.82-.33 1.65 1.65 0 00-1 1.51V21a2 2 0 01-4 0v-.09A1.65 1.65 0 009 19.4a1.65 1.65 0 00-1.82.33l-.06.06a2 2 0 01-2.83 0 2 2 0 010-2.83l.06-.06A1.65 1.65 0 004.68 15a1.65 1.65 0 00-1.51-1H3a2 2 0 010-4h.09A1.65 1.65 0 004.6 9a1.65 1.65 0 00-.33-1.82l-.06-.06a2 2 0 010-2.83 2 2 0 012.83 0l.06.06A1.65 1.65 0 009 4.68a1.65 1.65 0 001-1.51V3a2 2 0 014 0v.09a1.65 1.65 0 001 1.51 1.65 1.65 0 001.82-.33l.06-.06a2 2 0 012.83 0 2 2 0 010 2.83l-.06.06A1.65 1.65 0 0019.4 9a1.65 1.65 0 001.51 1H21a2 2 0 010 4h-.09a1.65 1.65 0 00-1.51 1z"/></svg>,
    plus: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2.5"><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg>,
    arrow_up: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><polyline points="18 15 12 9 6 15"/></svg>,
    arrow_down: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><polyline points="6 9 12 15 18 9"/></svg>,
    trend_up: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><polyline points="23 6 13.5 15.5 8.5 10.5 1 18"/><polyline points="17 6 23 6 23 12"/></svg>,
    search: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>,
    bell: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><path d="M18 8A6 6 0 006 8c0 7-3 9-3 9h18s-3-2-3-9M13.73 21a2 2 0 01-3.46 0"/></svg>,
    moon: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><path d="M21 12.79A9 9 0 1111.21 3 7 7 0 0021 12.79z"/></svg>,
    sun: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><circle cx="12" cy="12" r="5"/><line x1="12" y1="1" x2="12" y2="3"/><line x1="12" y1="21" x2="12" y2="23"/><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"/><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"/><line x1="1" y1="12" x2="3" y2="12"/><line x1="21" y1="12" x2="23" y2="12"/><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"/><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"/></svg>,
    x: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>,
    menu: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><line x1="3" y1="12" x2="21" y2="12"/><line x1="3" y1="6" x2="21" y2="6"/><line x1="3" y1="18" x2="21" y2="18"/></svg>,
    google: <svg width={size} height={size} viewBox="0 0 24 24"><path fill="#4285F4" d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92c-.26 1.37-1.04 2.53-2.21 3.31v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.09z"/><path fill="#34A853" d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z"/><path fill="#FBBC05" d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l2.85-2.22.81-.62z"/><path fill="#EA4335" d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z"/></svg>,
    wallet: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><path d="M20 12V22H4V12"/><path d="M22 7H2v5h20V7z"/><path d="M12 22V7"/><path d="M12 7H7.5a2.5 2.5 0 010-5C11 2 12 7 12 7z"/><path d="M12 7h4.5a2.5 2.5 0 000-5C13 2 12 7 12 7z"/></svg>,
    edit: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><path d="M11 4H4a2 2 0 00-2 2v14a2 2 0 002 2h14a2 2 0 002-2v-7"/><path d="M18.5 2.5a2.121 2.121 0 013 3L12 15l-4 1 1-4 9.5-9.5z"/></svg>,
    trash: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><polyline points="3 6 5 6 21 6"/><path d="M19 6v14a2 2 0 01-2 2H7a2 2 0 01-2-2V6m3 0V4a1 1 0 011-1h4a1 1 0 011 1v2"/></svg>,
    download: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><path d="M21 15v4a2 2 0 01-2 2H5a2 2 0 01-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/></svg>,
    filter: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><polygon points="22 3 2 3 10 12.46 10 19 14 21 14 12.46 22 3"/></svg>,
  };
  return icons[name] || null;
};

// ============================================================
// MINI BAR CHART
// ============================================================
function MiniBarChart({ data, colorKey = "income" }) {
  const max = Math.max(...data.map(d => d.value), 1);
  const colors = colorKey === "income" ? "#10b981" : "#ef4444";
  return (
    <div style={{ display:"flex", alignItems:"flex-end", gap:3, height:40 }}>
      {data.map((d, i) => (
        <div key={i} style={{ display:"flex", flexDirection:"column", alignItems:"center", flex:1 }}>
          <div style={{
            width:"100%", backgroundColor: colors,
            height: Math.max(3, (d.value / max) * 36),
            borderRadius:"3px 3px 0 0", opacity: i === data.length-1 ? 1 : 0.5,
            transition:"height 0.5s ease"
          }}/>
        </div>
      ))}
    </div>
  );
}

// ============================================================
// DONUT CHART
// ============================================================
function DonutChart({ segments, size = 120 }) {
  const total = segments.reduce((a, b) => a + b.value, 0);
  let cumulative = 0;
  const r = 45, cx = 60, cy = 60;
  const circumference = 2 * Math.PI * r;
  return (
    <svg width={size} height={size} viewBox="0 0 120 120">
      {segments.map((seg, i) => {
        const pct = seg.value / total;
        const offset = circumference - cumulative * circumference;
        const dash = pct * circumference;
        const el = (
          <circle key={i} cx={cx} cy={cy} r={r}
            fill="none" stroke={seg.color} strokeWidth="18"
            strokeDasharray={`${dash} ${circumference - dash}`}
            strokeDashoffset={offset - circumference * 0.25}
            style={{ transition:"stroke-dasharray 0.6s ease" }}
          />
        );
        cumulative += pct;
        return el;
      })}
      <circle cx={cx} cy={cy} r={r-9} fill="var(--bg-card)" />
    </svg>
  );
}

// ============================================================
// PROGRESS BAR
// ============================================================
function ProgressBar({ value, max, color = "#10b981" }) {
  const pct = Math.min(100, (value / max) * 100);
  const warn = pct > 80;
  return (
    <div style={{ height:6, background:"var(--bg-muted)", borderRadius:99, overflow:"hidden" }}>
      <div style={{
        width: `${pct}%`, height:"100%",
        background: warn ? "#ef4444" : color,
        borderRadius:99, transition:"width 0.6s ease"
      }}/>
    </div>
  );
}

// ============================================================
// MODAL
// ============================================================
function Modal({ open, onClose, title, children }) {
  if (!open) return null;
  return (
    <div style={{
      position:"fixed", inset:0, background:"rgba(0,0,0,0.6)",
      backdropFilter:"blur(4px)", zIndex:1000,
      display:"flex", alignItems:"flex-end", justifyContent:"center"
    }} onClick={onClose}>
      <div style={{
        background:"var(--bg-card)", borderRadius:"20px 20px 0 0",
        width:"100%", maxWidth:500, maxHeight:"90vh", overflow:"auto",
        padding:"20px 20px 40px", animation:"slideUp 0.3s ease"
      }} onClick={e => e.stopPropagation()}>
        <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:20 }}>
          <h2 style={{ fontSize:18, fontWeight:700, color:"var(--text-primary)", fontFamily:"'Playfair Display', serif" }}>{title}</h2>
          <button onClick={onClose} style={{ background:"var(--bg-muted)", border:"none", borderRadius:99, width:32, height:32, cursor:"pointer", display:"flex", alignItems:"center", justifyContent:"center", color:"var(--text-secondary)" }}>
            <Icon name="x" size={16} />
          </button>
        </div>
        {children}
      </div>
    </div>
  );
}

// ============================================================
// FORM COMPONENTS
// ============================================================
function FormGroup({ label, children }) {
  return (
    <div style={{ marginBottom:14 }}>
      <label style={{ display:"block", fontSize:12, fontWeight:600, color:"var(--text-secondary)", marginBottom:5, textTransform:"uppercase", letterSpacing:"0.05em" }}>{label}</label>
      {children}
    </div>
  );
}

function Input({ ...props }) {
  return <input {...props} style={{ width:"100%", padding:"10px 12px", background:"var(--bg-input)", border:"1px solid var(--border)", borderRadius:10, color:"var(--text-primary)", fontSize:14, outline:"none", boxSizing:"border-box", fontFamily:"inherit", ...props.style }} />;
}

function Select({ children, ...props }) {
  return <select {...props} style={{ width:"100%", padding:"10px 12px", background:"var(--bg-input)", border:"1px solid var(--border)", borderRadius:10, color:"var(--text-primary)", fontSize:14, outline:"none", boxSizing:"border-box", fontFamily:"inherit", ...props.style }}>{children}</select>;
}

function Btn({ children, variant="primary", small, ...props }) {
  const styles = {
    primary: { background:"var(--accent)", color:"#fff" },
    secondary: { background:"var(--bg-muted)", color:"var(--text-primary)" },
    danger: { background:"#ef4444", color:"#fff" },
    ghost: { background:"transparent", color:"var(--accent)", border:"1px solid var(--accent)" },
  };
  return (
    <button {...props} style={{
      ...styles[variant],
      border:"none", borderRadius:10, padding: small ? "6px 12px" : "11px 20px",
      fontSize: small ? 12 : 14, fontWeight:600, cursor:"pointer",
      display:"inline-flex", alignItems:"center", gap:6, fontFamily:"inherit",
      transition:"opacity 0.15s", ...props.style
    }}>
      {children}
    </button>
  );
}

// ============================================================
// ADD TRANSACTION FORM
// ============================================================
function AddTransactionModal({ open, onClose, type, onAdd, dark }) {
  const [form, setForm] = useState({ category:"", amount:"", payment:"Cash", description:"", notes:"", source:"", customer:"", vendor:"", bsDate: formatBS(getTodayBS()) });
  const cats = type === "income" ? INCOME_CATEGORIES : [...HOUSEHOLD_EXPENSE_CATS, ...BUSINESS_EXPENSE_CATS];
  const set = (k, v) => setForm(f => ({...f, [k]: v}));

  const handleSubmit = () => {
    if (!form.category || !form.amount) return;
    onAdd({ ...form, type, amount: +form.amount, id: Date.now(), createdBy:"Admin" });
    setForm({ category:"", amount:"", payment:"Cash", description:"", notes:"", source:"", customer:"", vendor:"", bsDate: formatBS(getTodayBS()) });
    onClose();
  };

  return (
    <Modal open={open} onClose={onClose} title={type === "income" ? "➕ Add Income" : "➕ Add Expense"}>
      <FormGroup label="BS Date"><Input type="text" value={form.bsDate} onChange={e => set("bsDate", e.target.value)} placeholder="YYYY-MM-DD" /></FormGroup>
      <FormGroup label="Category">
        <Select value={form.category} onChange={e => set("category", e.target.value)}>
          <option value="">Select category…</option>
          {cats.map(c => <option key={c}>{c}</option>)}
        </Select>
      </FormGroup>
      <FormGroup label="Amount (NPR)"><Input type="number" value={form.amount} onChange={e => set("amount", e.target.value)} placeholder="0.00" /></FormGroup>
      {type === "income" ? <>
        <FormGroup label="Source"><Input value={form.source} onChange={e => set("source", e.target.value)} placeholder="Source" /></FormGroup>
        <FormGroup label="Customer"><Input value={form.customer} onChange={e => set("customer", e.target.value)} placeholder="Customer name" /></FormGroup>
      </> : <>
        <FormGroup label="Vendor"><Input value={form.vendor} onChange={e => set("vendor", e.target.value)} placeholder="Vendor name" /></FormGroup>
      </>}
      <FormGroup label="Payment Method">
        <Select value={form.payment} onChange={e => set("payment", e.target.value)}>
          {PAYMENT_METHODS.map(p => <option key={p}>{p}</option>)}
        </Select>
      </FormGroup>
      <FormGroup label="Description"><Input value={form.description} onChange={e => set("description", e.target.value)} placeholder="Brief description" /></FormGroup>
      <FormGroup label="Notes"><Input value={form.notes} onChange={e => set("notes", e.target.value)} placeholder="Additional notes" /></FormGroup>
      <div style={{ display:"flex", gap:10, marginTop:8 }}>
        <Btn variant="secondary" style={{ flex:1 }} onClick={onClose}>Cancel</Btn>
        <Btn style={{ flex:2, background: type === "income" ? "#10b981" : "#ef4444" }} onClick={handleSubmit}>
          <Icon name="plus" size={16} /> Save {type === "income" ? "Income" : "Expense"}
        </Btn>
      </div>
    </Modal>
  );
}

// ============================================================
// DASHBOARD PAGE
// ============================================================
function DashboardPage({ transactions, dark }) {
  const todayStr = formatBS(getTodayBS());
  const todayTx = transactions.filter(t => t.bsDate === todayStr);
  const todayIncome = todayTx.filter(t => t.type === "income").reduce((a, t) => a + t.amount, 0);
  const todayExpense = todayTx.filter(t => t.type === "expense").reduce((a, t) => a + t.amount, 0);
  const monthlyIncome = transactions.filter(t => t.type === "income").reduce((a, t) => a + t.amount, 0);
  const monthlyExpense = transactions.filter(t => t.type === "expense").reduce((a, t) => a + t.amount, 0);
  const balance = monthlyIncome - monthlyExpense;

  const expenseByCategory = HOUSEHOLD_EXPENSE_CATS.concat(BUSINESS_EXPENSE_CATS).map(cat => ({
    cat, value: transactions.filter(t => t.type === "expense" && t.category === cat).reduce((a, t) => a + t.amount, 0)
  })).filter(x => x.value > 0).sort((a,b) => b.value - a.value).slice(0,5);

  const incomeByCategory = INCOME_CATEGORIES.map(cat => ({
    cat, value: transactions.filter(t => t.type === "income" && t.category === cat).reduce((a, t) => a + t.amount, 0)
  })).filter(x => x.value > 0);

  const last7 = Array.from({length:7}, (_, i) => {
    const d = new Date(); d.setDate(d.getDate() - (6-i));
    const bs = adToBS(d);
    const str = formatBS(bs);
    return { label: bs.day, value: transactions.filter(t => t.bsDate === str && t.type === "income").reduce((a,t)=>a+t.amount,0) };
  });

  const fmt = (n) => n >= 1000 ? `NPR ${(n/1000).toFixed(1)}K` : `NPR ${n}`;

  const StatCard = ({ label, value, icon, color, sub }) => (
    <div style={{ background:"var(--bg-card)", borderRadius:16, padding:"16px", border:"1px solid var(--border)", position:"relative", overflow:"hidden" }}>
      <div style={{ position:"absolute", top:-10, right:-10, width:60, height:60, borderRadius:"50%", background:color+"22" }} />
      <div style={{ display:"flex", justifyContent:"space-between", alignItems:"flex-start" }}>
        <div>
          <p style={{ fontSize:11, color:"var(--text-secondary)", fontWeight:600, textTransform:"uppercase", letterSpacing:"0.07em", marginBottom:6 }}>{label}</p>
          <p style={{ fontSize:20, fontWeight:800, color:"var(--text-primary)", fontFamily:"'Playfair Display', serif" }}>{value}</p>
          {sub && <p style={{ fontSize:11, color:"var(--text-secondary)", marginTop:3 }}>{sub}</p>}
        </div>
        <div style={{ background:color+"22", borderRadius:12, padding:10, color:color }}>{icon}</div>
      </div>
    </div>
  );

  return (
    <div style={{ padding:"0 0 20px" }}>
      {/* Header */}
      <div style={{ padding:"16px 0 20px" }}>
        <p style={{ color:"var(--text-secondary)", fontSize:12, fontWeight:500 }}>
          {formatBSNepali(getTodayBS())} • {new Date().toLocaleDateString("en-US",{weekday:"long"})}
        </p>
        <h1 style={{ fontSize:24, fontWeight:900, color:"var(--text-primary)", fontFamily:"'Playfair Display', serif", marginTop:2 }}>
          नमस्ते! 🙏
        </h1>
        <p style={{ color:"var(--text-secondary)", fontSize:13 }}>Here's your financial overview</p>
      </div>

      {/* Balance Hero */}
      <div style={{ background: dark ? "linear-gradient(135deg, #1a2f23 0%, #0d1f17 100%)" : "linear-gradient(135deg, #10b981 0%, #059669 100%)", borderRadius:20, padding:"20px", marginBottom:16, position:"relative", overflow:"hidden" }}>
        <div style={{ position:"absolute", top:-20, right:-20, width:120, height:120, borderRadius:"50%", background:"rgba(255,255,255,0.05)" }} />
        <div style={{ position:"absolute", bottom:-30, left:-10, width:90, height:90, borderRadius:"50%", background:"rgba(255,255,255,0.05)" }} />
        <p style={{ color: dark ? "#10b981" : "rgba(255,255,255,0.8)", fontSize:12, fontWeight:600, textTransform:"uppercase", letterSpacing:"0.08em" }}>Current Balance</p>
        <p style={{ color: dark ? "#fff" : "#fff", fontSize:32, fontWeight:900, fontFamily:"'Playfair Display', serif", margin:"6px 0 12px" }}>
          NPR {balance.toLocaleString("en-IN")}
        </p>
        <div style={{ display:"flex", gap:20 }}>
          <div>
            <p style={{ color: dark ? "#10b981" : "rgba(255,255,255,0.75)", fontSize:11 }}>Monthly Income</p>
            <p style={{ color: dark ? "#fff" : "#fff", fontWeight:700, fontSize:14 }}>NPR {monthlyIncome.toLocaleString("en-IN")}</p>
          </div>
          <div>
            <p style={{ color: dark ? "#ef4444" : "rgba(255,255,255,0.75)", fontSize:11 }}>Monthly Expenses</p>
            <p style={{ color: dark ? "#fff" : "#fff", fontWeight:700, fontSize:14 }}>NPR {monthlyExpense.toLocaleString("en-IN")}</p>
          </div>
        </div>
      </div>

      {/* Stat Grid */}
      <div style={{ display:"grid", gridTemplateColumns:"1fr 1fr", gap:12, marginBottom:16 }}>
        <StatCard label="Today's Income" value={fmt(todayIncome)} color="#10b981" icon={<Icon name="arrow_up" size={18} color="#10b981"/>} sub={`${todayTx.filter(t=>t.type==="income").length} transactions`} />
        <StatCard label="Today's Expense" value={fmt(todayExpense)} color="#ef4444" icon={<Icon name="arrow_down" size={18} color="#ef4444"/>} sub={`${todayTx.filter(t=>t.type==="expense").length} transactions`} />
        <StatCard label="Profit/Loss" value={fmt(monthlyIncome - monthlyExpense)} color="#6366f1" icon={<Icon name="trend_up" size={18} color="#6366f1"/>} sub="This month" />
        <StatCard label="Transactions" value={transactions.length} color="#f59e0b" icon={<Icon name="wallet" size={18} color="#f59e0b"/>} sub="Total records" />
      </div>

      {/* Income Chart */}
      <div style={{ background:"var(--bg-card)", borderRadius:16, padding:16, border:"1px solid var(--border)", marginBottom:12 }}>
        <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:12 }}>
          <h3 style={{ fontSize:14, fontWeight:700, color:"var(--text-primary)" }}>Income — Last 7 Days</h3>
          <span style={{ fontSize:11, color:"var(--text-secondary)" }}>{today.year} BS</span>
        </div>
        <MiniBarChart data={last7} colorKey="income" />
        <div style={{ display:"flex", justifyContent:"space-between", marginTop:4 }}>
          {last7.map((d,i) => <span key={i} style={{ fontSize:10, color:"var(--text-secondary)", flex:1, textAlign:"center" }}>{d.label}</span>)}
        </div>
      </div>

      {/* Top Expenses + Donut */}
      {expenseByCategory.length > 0 && (
        <div style={{ background:"var(--bg-card)", borderRadius:16, padding:16, border:"1px solid var(--border)", marginBottom:12 }}>
          <h3 style={{ fontSize:14, fontWeight:700, color:"var(--text-primary)", marginBottom:12 }}>Top Expense Categories</h3>
          <div style={{ display:"flex", gap:16, alignItems:"center" }}>
            <DonutChart segments={expenseByCategory.map(e => ({ value:e.value, color: getCatColor(e.cat) }))} />
            <div style={{ flex:1 }}>
              {expenseByCategory.map((e,i) => (
                <div key={i} style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:8 }}>
                  <div style={{ display:"flex", alignItems:"center", gap:8 }}>
                    <div style={{ width:8, height:8, borderRadius:"50%", background:getCatColor(e.cat), flexShrink:0 }} />
                    <span style={{ fontSize:12, color:"var(--text-secondary)" }}>{e.cat}</span>
                  </div>
                  <span style={{ fontSize:12, fontWeight:700, color:"var(--text-primary)" }}>NPR {e.value.toLocaleString()}</span>
                </div>
              ))}
            </div>
          </div>
        </div>
      )}

      {/* Recent Transactions */}
      <div style={{ background:"var(--bg-card)", borderRadius:16, padding:16, border:"1px solid var(--border)" }}>
        <h3 style={{ fontSize:14, fontWeight:700, color:"var(--text-primary)", marginBottom:12 }}>Recent Transactions</h3>
        {transactions.slice(0, 5).map(tx => (
          <div key={tx.id} style={{ display:"flex", alignItems:"center", justifyContent:"space-between", padding:"10px 0", borderBottom:"1px solid var(--border)" }}>
            <div style={{ display:"flex", alignItems:"center", gap:12 }}>
              <div style={{ width:36, height:36, borderRadius:12, background: getCatColor(tx.category) + "22", display:"flex", alignItems:"center", justifyContent:"center", color:getCatColor(tx.category) }}>
                <Icon name={tx.type === "income" ? "arrow_up" : "arrow_down"} size={16} color={getCatColor(tx.category)} />
              </div>
              <div>
                <p style={{ fontSize:13, fontWeight:600, color:"var(--text-primary)" }}>{tx.category}</p>
                <p style={{ fontSize:11, color:"var(--text-secondary)" }}>{tx.bsDate} • {tx.payment}</p>
              </div>
            </div>
            <span style={{ fontSize:14, fontWeight:700, color: tx.type === "income" ? "#10b981" : "#ef4444" }}>
              {tx.type === "income" ? "+" : "-"}NPR {tx.amount.toLocaleString()}
            </span>
          </div>
        ))}
      </div>
    </div>
  );
}

// ============================================================
// TRANSACTIONS PAGE
// ============================================================
function TransactionsPage({ transactions, onAdd, onDelete, type }) {
  const [search, setSearch] = useState("");
  const [filterCat, setFilterCat] = useState("All");
  const [showAdd, setShowAdd] = useState(false);

  const filtered = transactions.filter(t =>
    t.type === type &&
    (filterCat === "All" || t.category === filterCat) &&
    (t.description?.toLowerCase().includes(search.toLowerCase()) || t.category.toLowerCase().includes(search.toLowerCase()) || (t.customer||t.vendor||"").toLowerCase().includes(search.toLowerCase()))
  );

  const cats = type === "income" ? ["All", ...INCOME_CATEGORIES] : ["All", ...HOUSEHOLD_EXPENSE_CATS, ...BUSINESS_EXPENSE_CATS];
  const total = filtered.reduce((a, t) => a + t.amount, 0);

  return (
    <div>
      <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", paddingBottom:16 }}>
        <div>
          <h2 style={{ fontSize:20, fontWeight:800, color:"var(--text-primary)", fontFamily:"'Playfair Display', serif" }}>{type === "income" ? "Income" : "Expenses"}</h2>
          <p style={{ fontSize:12, color:"var(--text-secondary)" }}>{filtered.length} records • NPR {total.toLocaleString()}</p>
        </div>
        <Btn onClick={() => setShowAdd(true)} style={{ background: type === "income" ? "#10b981" : "#ef4444" }}>
          <Icon name="plus" size={16} /> Add
        </Btn>
      </div>

      {/* Search */}
      <div style={{ position:"relative", marginBottom:12 }}>
        <div style={{ position:"absolute", left:12, top:"50%", transform:"translateY(-50%)", color:"var(--text-secondary)" }}><Icon name="search" size={16} /></div>
        <Input value={search} onChange={e => setSearch(e.target.value)} placeholder="Search transactions…" style={{ paddingLeft:38 }} />
      </div>

      {/* Category filter chips */}
      <div style={{ display:"flex", gap:8, overflowX:"auto", paddingBottom:12, scrollbarWidth:"none" }}>
        {["All", ...( type === "income" ? INCOME_CATEGORIES : HOUSEHOLD_EXPENSE_CATS.slice(0,8) )].map(c => (
          <button key={c} onClick={() => setFilterCat(c)} style={{
            flexShrink:0, padding:"5px 14px", borderRadius:99, fontSize:12, fontWeight:600,
            border: filterCat === c ? "none" : "1px solid var(--border)",
            background: filterCat === c ? "var(--accent)" : "var(--bg-muted)",
            color: filterCat === c ? "#fff" : "var(--text-secondary)",
            cursor:"pointer", fontFamily:"inherit"
          }}>{c}</button>
        ))}
      </div>

      {/* List */}
      <div style={{ background:"var(--bg-card)", borderRadius:16, border:"1px solid var(--border)", overflow:"hidden" }}>
        {filtered.length === 0 ? (
          <div style={{ padding:"32px", textAlign:"center", color:"var(--text-secondary)" }}>
            <p style={{ fontSize:32 }}>📭</p>
            <p style={{ marginTop:8 }}>No {type} records found</p>
          </div>
        ) : filtered.map((tx, i) => (
          <div key={tx.id} style={{ display:"flex", alignItems:"center", padding:"12px 16px", borderBottom: i < filtered.length-1 ? "1px solid var(--border)" : "none" }}>
            <div style={{ width:40, height:40, borderRadius:12, background:getCatColor(tx.category)+"22", display:"flex", alignItems:"center", justifyContent:"center", marginRight:12, flexShrink:0 }}>
              <span style={{ fontSize:18 }}>{type === "income" ? "💰" : "🧾"}</span>
            </div>
            <div style={{ flex:1, minWidth:0 }}>
              <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center" }}>
                <p style={{ fontSize:13, fontWeight:700, color:"var(--text-primary)", whiteSpace:"nowrap", overflow:"hidden", textOverflow:"ellipsis" }}>{tx.category}</p>
                <span style={{ fontSize:14, fontWeight:800, color: type === "income" ? "#10b981" : "#ef4444", marginLeft:8, flexShrink:0 }}>
                  {type === "income" ? "+" : "-"}NPR {tx.amount.toLocaleString()}
                </span>
              </div>
              <div style={{ display:"flex", gap:8, marginTop:2, flexWrap:"wrap" }}>
                <span style={{ fontSize:11, color:"var(--text-secondary)" }}>{tx.bsDate}</span>
                {(tx.customer || tx.vendor) && <span style={{ fontSize:11, color:"var(--text-secondary)" }}>• {tx.customer || tx.vendor}</span>}
                <span style={{ fontSize:11, background:"var(--bg-muted)", borderRadius:99, padding:"1px 7px", color:"var(--text-secondary)" }}>{tx.payment}</span>
              </div>
              {tx.description && <p style={{ fontSize:11, color:"var(--text-secondary)", marginTop:2, opacity:0.7 }}>{tx.description}</p>}
            </div>
          </div>
        ))}
      </div>

      <AddTransactionModal open={showAdd} onClose={() => setShowAdd(false)} type={type} onAdd={onAdd} />
    </div>
  );
}

// ============================================================
// BUDGET PAGE
// ============================================================
function BudgetPage({ transactions, budgets, setBudgets }) {
  const [showEdit, setShowEdit] = useState(null);
  const [newLimit, setNewLimit] = useState("");

  const getBudgetUsed = (cat) => transactions.filter(t => t.type === "expense" && t.category === cat).reduce((a, t) => a + t.amount, 0);
  const totalHouseholdBudget = budgets.filter(b => b.type === "household").reduce((a, b) => a + b.limit, 0);
  const totalBusinessBudget = budgets.filter(b => b.type === "business").reduce((a, b) => a + b.limit, 0);
  const totalUsed = transactions.filter(t => t.type === "expense").reduce((a, t) => a + t.amount, 0);

  const saveEdit = () => {
    setBudgets(bs => bs.map(b => b.category === showEdit ? { ...b, limit: +newLimit } : b));
    setShowEdit(null);
  };

  const BudgetCard = ({ b }) => {
    const used = getBudgetUsed(b.category);
    const pct = Math.min(100, (used / b.limit) * 100);
    const over = used > b.limit;
    return (
      <div style={{ background:"var(--bg-card)", borderRadius:14, padding:"14px 16px", border: over ? "1px solid #ef444444" : "1px solid var(--border)", marginBottom:10 }}>
        <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:8 }}>
          <div style={{ display:"flex", alignItems:"center", gap:8 }}>
            <div style={{ width:8, height:8, borderRadius:"50%", background:getCatColor(b.category) }} />
            <span style={{ fontSize:13, fontWeight:600, color:"var(--text-primary)" }}>{b.category}</span>
            {over && <span style={{ fontSize:10, background:"#ef444422", color:"#ef4444", padding:"2px 7px", borderRadius:99, fontWeight:700 }}>OVER</span>}
          </div>
          <button onClick={() => { setShowEdit(b.category); setNewLimit(b.limit); }} style={{ background:"none", border:"none", cursor:"pointer", color:"var(--text-secondary)" }}>
            <Icon name="edit" size={14} />
          </button>
        </div>
        <ProgressBar value={used} max={b.limit} color={getCatColor(b.category)} />
        <div style={{ display:"flex", justifyContent:"space-between", marginTop:8 }}>
          <span style={{ fontSize:11, color:"var(--text-secondary)" }}>NPR {used.toLocaleString()} / {b.limit.toLocaleString()}</span>
          <span style={{ fontSize:11, fontWeight:700, color: over ? "#ef4444" : "var(--text-secondary)" }}>{pct.toFixed(0)}%</span>
        </div>
      </div>
    );
  };

  return (
    <div>
      <h2 style={{ fontSize:20, fontWeight:800, color:"var(--text-primary)", fontFamily:"'Playfair Display', serif", marginBottom:4 }}>Budget Tracker</h2>
      <p style={{ fontSize:12, color:"var(--text-secondary)", marginBottom:16 }}>Baisakh {today.year} BS</p>

      {/* Summary */}
      <div style={{ display:"grid", gridTemplateColumns:"1fr 1fr", gap:12, marginBottom:20 }}>
        {[
          { label:"Household Budget", val:`NPR ${totalHouseholdBudget.toLocaleString()}`, sub:"Total allocated", color:"#10b981" },
          { label:"Business Budget", val:`NPR ${totalBusinessBudget.toLocaleString()}`, sub:"Total allocated", color:"#6366f1" },
          { label:"Total Spent", val:`NPR ${totalUsed.toLocaleString()}`, sub:"This month", color:"#ef4444" },
          { label:"Remaining", val:`NPR ${(totalHouseholdBudget+totalBusinessBudget-totalUsed).toLocaleString()}`, sub:"Available", color:"#f59e0b" },
        ].map((s,i) => (
          <div key={i} style={{ background:"var(--bg-card)", borderRadius:14, padding:"14px", border:"1px solid var(--border)" }}>
            <p style={{ fontSize:11, color:"var(--text-secondary)", fontWeight:600, textTransform:"uppercase", letterSpacing:"0.06em" }}>{s.label}</p>
            <p style={{ fontSize:17, fontWeight:800, color:s.color, fontFamily:"'Playfair Display', serif", marginTop:4 }}>{s.val}</p>
            <p style={{ fontSize:11, color:"var(--text-secondary)" }}>{s.sub}</p>
          </div>
        ))}
      </div>

      <h3 style={{ fontSize:14, fontWeight:700, color:"var(--text-primary)", marginBottom:10 }}>🏠 Household Budgets</h3>
      {budgets.filter(b => b.type === "household").map(b => <BudgetCard key={b.category} b={b} />)}

      <h3 style={{ fontSize:14, fontWeight:700, color:"var(--text-primary)", margin:"16px 0 10px" }}>🏢 Business Budgets</h3>
      {budgets.filter(b => b.type === "business").map(b => <BudgetCard key={b.category} b={b} />)}

      <Modal open={!!showEdit} onClose={() => setShowEdit(null)} title={`Edit Budget: ${showEdit}`}>
        <FormGroup label="Monthly Limit (NPR)">
          <Input type="number" value={newLimit} onChange={e => setNewLimit(e.target.value)} />
        </FormGroup>
        <div style={{ display:"flex", gap:10 }}>
          <Btn variant="secondary" style={{ flex:1 }} onClick={() => setShowEdit(null)}>Cancel</Btn>
          <Btn style={{ flex:2 }} onClick={saveEdit}>Save Budget</Btn>
        </div>
      </Modal>
    </div>
  );
}

// ============================================================
// REPORTS PAGE
// ============================================================
function ReportsPage({ transactions }) {
  const [period, setPeriod] = useState("monthly");
  const totalIncome = transactions.filter(t => t.type === "income").reduce((a, t) => a + t.amount, 0);
  const totalExpense = transactions.filter(t => t.type === "expense").reduce((a, t) => a + t.amount, 0);
  const netProfit = totalIncome - totalExpense;
  const savingsRate = totalIncome > 0 ? ((netProfit / totalIncome) * 100).toFixed(1) : 0;

  const incomeByCat = INCOME_CATEGORIES.map(cat => ({
    cat, value: transactions.filter(t => t.type === "income" && t.category === cat).reduce((a,t)=>a+t.amount,0)
  })).filter(x => x.value > 0);

  const expByCat = [...HOUSEHOLD_EXPENSE_CATS, ...BUSINESS_EXPENSE_CATS].map(cat => ({
    cat, value: transactions.filter(t => t.type === "expense" && t.category === cat).reduce((a,t)=>a+t.amount,0)
  })).filter(x => x.value > 0).sort((a,b) => b.value - a.value);

  return (
    <div>
      <h2 style={{ fontSize:20, fontWeight:800, color:"var(--text-primary)", fontFamily:"'Playfair Display', serif", marginBottom:4 }}>Reports</h2>
      <p style={{ fontSize:12, color:"var(--text-secondary)", marginBottom:16 }}>Financial analytics & summaries</p>

      {/* Period selector */}
      <div style={{ display:"flex", gap:8, marginBottom:20 }}>
        {["daily","weekly","monthly","yearly"].map(p => (
          <button key={p} onClick={() => setPeriod(p)} style={{
            flex:1, padding:"8px 0", borderRadius:10, fontSize:12, fontWeight:600,
            background: period === p ? "var(--accent)" : "var(--bg-muted)",
            color: period === p ? "#fff" : "var(--text-secondary)",
            border:"none", cursor:"pointer", textTransform:"capitalize", fontFamily:"inherit"
          }}>{p}</button>
        ))}
      </div>

      {/* Summary Cards */}
      <div style={{ display:"grid", gridTemplateColumns:"1fr 1fr", gap:12, marginBottom:20 }}>
        {[
          { label:"Total Income", val:`NPR ${totalIncome.toLocaleString()}`, color:"#10b981", icon:"💚" },
          { label:"Total Expense", val:`NPR ${totalExpense.toLocaleString()}`, color:"#ef4444", icon:"🔴" },
          { label:"Net Profit", val:`NPR ${netProfit.toLocaleString()}`, color: netProfit >= 0 ? "#10b981" : "#ef4444", icon:"📊" },
          { label:"Savings Rate", val:`${savingsRate}%`, color:"#6366f1", icon:"💰" },
        ].map((s,i) => (
          <div key={i} style={{ background:"var(--bg-card)", borderRadius:14, padding:"14px", border:"1px solid var(--border)", textAlign:"center" }}>
            <p style={{ fontSize:24, marginBottom:4 }}>{s.icon}</p>
            <p style={{ fontSize:16, fontWeight:800, color:s.color, fontFamily:"'Playfair Display', serif" }}>{s.val}</p>
            <p style={{ fontSize:11, color:"var(--text-secondary)", marginTop:2 }}>{s.label}</p>
          </div>
        ))}
      </div>

      {/* Income by Category */}
      {incomeByCat.length > 0 && (
        <div style={{ background:"var(--bg-card)", borderRadius:16, padding:16, border:"1px solid var(--border)", marginBottom:12 }}>
          <h3 style={{ fontSize:14, fontWeight:700, color:"var(--text-primary)", marginBottom:12 }}>Income Breakdown</h3>
          {incomeByCat.map((e,i) => (
            <div key={i} style={{ marginBottom:10 }}>
              <div style={{ display:"flex", justifyContent:"space-between", marginBottom:4 }}>
                <span style={{ fontSize:12, color:"var(--text-primary)", fontWeight:600 }}>{e.cat}</span>
                <span style={{ fontSize:12, fontWeight:700, color:"#10b981" }}>NPR {e.value.toLocaleString()}</span>
              </div>
              <ProgressBar value={e.value} max={totalIncome} color="#10b981" />
            </div>
          ))}
        </div>
      )}

      {/* Expense by Category */}
      {expByCat.length > 0 && (
        <div style={{ background:"var(--bg-card)", borderRadius:16, padding:16, border:"1px solid var(--border)", marginBottom:12 }}>
          <h3 style={{ fontSize:14, fontWeight:700, color:"var(--text-primary)", marginBottom:12 }}>Expense Breakdown</h3>
          {expByCat.map((e,i) => (
            <div key={i} style={{ marginBottom:10 }}>
              <div style={{ display:"flex", justifyContent:"space-between", marginBottom:4 }}>
                <span style={{ fontSize:12, color:"var(--text-primary)", fontWeight:600 }}>{e.cat}</span>
                <span style={{ fontSize:12, fontWeight:700, color:"#ef4444" }}>NPR {e.value.toLocaleString()}</span>
              </div>
              <ProgressBar value={e.value} max={totalExpense} color={getCatColor(e.cat)} />
            </div>
          ))}
        </div>
      )}

      {/* Export */}
      <div style={{ background:"var(--bg-card)", borderRadius:16, padding:16, border:"1px solid var(--border)" }}>
        <h3 style={{ fontSize:14, fontWeight:700, color:"var(--text-primary)", marginBottom:12 }}>Export Report</h3>
        <div style={{ display:"flex", gap:10 }}>
          {["PDF","Excel","CSV"].map(fmt => (
            <button key={fmt} style={{ flex:1, padding:"10px", borderRadius:10, background:"var(--bg-muted)", border:"1px solid var(--border)", cursor:"pointer", fontSize:12, fontWeight:700, color:"var(--text-secondary)", display:"flex", flexDirection:"column", alignItems:"center", gap:4, fontFamily:"inherit" }}>
              <Icon name="download" size={18} color="var(--text-secondary)" />
              {fmt}
            </button>
          ))}
        </div>
      </div>
    </div>
  );
}

// ============================================================
// SETTINGS PAGE
// ============================================================
function SettingsPage({ dark, setDark }) {
  const [sheetsId, setSheetsId] = useState("");
  const [syncing, setSyncing] = useState(false);

  const mockSync = () => {
    setSyncing(true);
    setTimeout(() => setSyncing(false), 2000);
  };

  return (
    <div>
      <h2 style={{ fontSize:20, fontWeight:800, color:"var(--text-primary)", fontFamily:"'Playfair Display', serif", marginBottom:16 }}>Settings</h2>

      {/* Google Account */}
      <div style={{ background:"var(--bg-card)", borderRadius:16, padding:16, border:"1px solid var(--border)", marginBottom:12 }}>
        <h3 style={{ fontSize:14, fontWeight:700, color:"var(--text-primary)", marginBottom:12 }}>Google Account</h3>
        <div style={{ display:"flex", alignItems:"center", gap:12, padding:"12px", background:"var(--bg-muted)", borderRadius:12, marginBottom:12 }}>
          <div style={{ width:40, height:40, borderRadius:"50%", background:"linear-gradient(135deg,#10b981,#059669)", display:"flex", alignItems:"center", justifyContent:"center", color:"#fff", fontWeight:800, fontSize:16 }}>A</div>
          <div>
            <p style={{ fontSize:14, fontWeight:700, color:"var(--text-primary)" }}>Admin User</p>
            <p style={{ fontSize:12, color:"var(--text-secondary)" }}>admin@family.com</p>
          </div>
          <span style={{ marginLeft:"auto", fontSize:11, background:"#10b98122", color:"#10b981", padding:"3px 10px", borderRadius:99, fontWeight:700 }}>ADMIN</span>
        </div>
        <Btn variant="ghost" style={{ width:"100%" }}>
          <Icon name="google" size={16} /> Sign in with Google
        </Btn>
      </div>

      {/* Google Sheets */}
      <div style={{ background:"var(--bg-card)", borderRadius:16, padding:16, border:"1px solid var(--border)", marginBottom:12 }}>
        <h3 style={{ fontSize:14, fontWeight:700, color:"var(--text-primary)", marginBottom:12 }}>Google Sheets Integration</h3>
        <FormGroup label="Spreadsheet ID">
          <Input value={sheetsId} onChange={e => setSheetsId(e.target.value)} placeholder="1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgVE2upms" />
        </FormGroup>
        <div style={{ display:"flex", gap:10 }}>
          <Btn variant="secondary" style={{ flex:1 }} onClick={mockSync}>{syncing ? "Syncing…" : "Test Connection"}</Btn>
          <Btn style={{ flex:2 }}>Connect Sheet</Btn>
        </div>
        <div style={{ marginTop:12, padding:"10px 12px", background:"#10b98111", borderRadius:10, border:"1px solid #10b98133" }}>
          <p style={{ fontSize:12, color:"#10b981", fontWeight:600 }}>✓ Sheets: Income, Expenses, Budget, Users</p>
        </div>
      </div>

      {/* Appearance */}
      <div style={{ background:"var(--bg-card)", borderRadius:16, padding:16, border:"1px solid var(--border)", marginBottom:12 }}>
        <h3 style={{ fontSize:14, fontWeight:700, color:"var(--text-primary)", marginBottom:12 }}>Appearance</h3>
        <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center" }}>
          <div>
            <p style={{ fontSize:14, color:"var(--text-primary)", fontWeight:600 }}>Dark Mode</p>
            <p style={{ fontSize:12, color:"var(--text-secondary)" }}>Switch between light and dark</p>
          </div>
          <button onClick={() => setDark(!dark)} style={{ width:52, height:28, borderRadius:99, background: dark ? "var(--accent)" : "var(--bg-muted)", border:"none", cursor:"pointer", position:"relative", transition:"background 0.2s" }}>
            <div style={{ position:"absolute", top:3, left: dark ? 27 : 3, width:22, height:22, borderRadius:"50%", background:"#fff", transition:"left 0.2s", display:"flex", alignItems:"center", justifyContent:"center" }}>
              {dark ? <Icon name="moon" size={12} color="#1a1a2e" /> : <Icon name="sun" size={12} color="#f59e0b" />}
            </div>
          </button>
        </div>
      </div>

      {/* App Info */}
      <div style={{ background:"var(--bg-card)", borderRadius:16, padding:16, border:"1px solid var(--border)" }}>
        <h3 style={{ fontSize:14, fontWeight:700, color:"var(--text-primary)", marginBottom:12 }}>About</h3>
        <div style={{ fontSize:13, color:"var(--text-secondary)", lineHeight:1.8 }}>
          <p>📱 Family Business Finance Manager</p>
          <p>🗓️ Bikram Samvat Support</p>
          <p>💾 Google Sheets Database</p>
          <p>🔒 Google OAuth Authentication</p>
          <p>📊 PWA — works offline</p>
          <p style={{ marginTop:8, fontSize:11, opacity:0.6 }}>Version 1.0.0 • Built for Nepal 🇳🇵</p>
        </div>
      </div>
    </div>
  );
}

// ============================================================
// MAIN APP
// ============================================================
export default function App() {
  const [dark, setDark] = useState(true);
  const [page, setPage] = useState("dashboard");
  const [transactions, setTransactions] = useState(INITIAL_TRANSACTIONS);
  const [budgets, setBudgets] = useState(INITIAL_BUDGETS);
  const [sidebarOpen, setSidebarOpen] = useState(false);

  const addTransaction = (tx) => setTransactions(t => [tx, ...t]);
  const deleteTransaction = (id) => setTransactions(t => t.filter(x => x.id !== id));

  const theme = dark ? {
    "--bg": "#0d1117", "--bg-card": "#161b22", "--bg-muted": "#21262d",
    "--bg-input": "#0d1117", "--border": "#30363d",
    "--text-primary": "#e6edf3", "--text-secondary": "#7d8590",
    "--accent": "#10b981",
  } : {
    "--bg": "#f6f8fa", "--bg-card": "#ffffff", "--bg-muted": "#f0f2f5",
    "--bg-input": "#ffffff", "--border": "#e1e4e8",
    "--text-primary": "#1c2128", "--text-secondary": "#6e7781",
    "--accent": "#059669",
  };

  const cssVars = Object.entries(theme).map(([k,v]) => `${k}:${v}`).join(";");

  const navItems = [
    { id:"dashboard", label:"Dashboard", icon:"dashboard" },
    { id:"income", label:"Income", icon:"income" },
    { id:"expense", label:"Expenses", icon:"expense" },
    { id:"budget", label:"Budget", icon:"budget" },
    { id:"reports", label:"Reports", icon:"reports" },
    { id:"settings", label:"Settings", icon:"settings" },
  ];

  const pageComponents = {
    dashboard: <DashboardPage transactions={transactions} dark={dark} />,
    income: <TransactionsPage transactions={transactions} onAdd={addTransaction} onDelete={deleteTransaction} type="income" />,
    expense: <TransactionsPage transactions={transactions} onAdd={addTransaction} onDelete={deleteTransaction} type="expense" />,
    budget: <BudgetPage transactions={transactions} budgets={budgets} setBudgets={setBudgets} />,
    reports: <ReportsPage transactions={transactions} />,
    settings: <SettingsPage dark={dark} setDark={setDark} />,
  };

  return (
    <>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700;800;900&family=DM+Sans:wght@400;500;600;700;800&display=swap');
        * { margin:0; padding:0; box-sizing:border-box; }
        body { font-family:'DM Sans',sans-serif; background:var(--bg); color:var(--text-primary); }
        ::-webkit-scrollbar { display:none; }
        @keyframes slideUp { from { transform:translateY(100%); opacity:0; } to { transform:translateY(0); opacity:1; } }
        @keyframes fadeIn { from { opacity:0; transform:translateY(8px); } to { opacity:1; transform:translateY(0); } }
        .page-content { animation: fadeIn 0.25s ease; }
      `}</style>
      <div style={{ ...Object.fromEntries(Object.entries(theme)), minHeight:"100vh", background:"var(--bg)", fontFamily:"'DM Sans',sans-serif" }}>
        {/* Inject CSS vars via style attribute trick */}
        <div style={{ cssText: cssVars } as any}>
        
        {/* Sidebar Overlay */}
        {sidebarOpen && (
          <div style={{ position:"fixed", inset:0, background:"rgba(0,0,0,0.5)", zIndex:90 }} onClick={() => setSidebarOpen(false)}>
            <div style={{ width:260, height:"100%", background:"var(--bg-card)", padding:"20px 16px", display:"flex", flexDirection:"column", gap:4 }} onClick={e => e.stopPropagation()}>
              <div style={{ display:"flex", alignItems:"center", gap:10, marginBottom:20, paddingBottom:16, borderBottom:"1px solid var(--border)" }}>
                <div style={{ width:36, height:36, borderRadius:10, background:"var(--accent)", display:"flex", alignItems:"center", justifyContent:"center", fontSize:18 }}>💼</div>
                <div>
                  <p style={{ fontSize:14, fontWeight:800, color:"var(--text-primary)" }}>ParivaarKhata</p>
                  <p style={{ fontSize:11, color:"var(--text-secondary)" }}>परिवार खाता</p>
                </div>
              </div>
              {navItems.map(n => (
                <button key={n.id} onClick={() => { setPage(n.id); setSidebarOpen(false); }} style={{
                  display:"flex", alignItems:"center", gap:12, padding:"12px 14px",
                  borderRadius:12, background: page === n.id ? "var(--accent)" : "transparent",
                  color: page === n.id ? "#fff" : "var(--text-secondary)",
                  border:"none", cursor:"pointer", fontSize:14, fontWeight:600, fontFamily:"inherit",
                  transition:"all 0.15s"
                }}>
                  <Icon name={n.icon} size={18} color={page === n.id ? "#fff" : "var(--text-secondary)"} />
                  {n.label}
                </button>
              ))}
            </div>
          </div>
        )}

        {/* Top Bar */}
        <div style={{ position:"sticky", top:0, zIndex:50, background:"var(--bg-card)", borderBottom:"1px solid var(--border)", padding:"12px 16px", display:"flex", justifyContent:"space-between", alignItems:"center" }}>
          <div style={{ display:"flex", alignItems:"center", gap:10 }}>
            <button onClick={() => setSidebarOpen(true)} style={{ background:"none", border:"none", cursor:"pointer", color:"var(--text-primary)", display:"flex" }}>
              <Icon name="menu" size={22} />
            </button>
            <div>
              <p style={{ fontSize:15, fontWeight:800, color:"var(--text-primary)", fontFamily:"'Playfair Display', serif" }}>ParivaarKhata</p>
              <p style={{ fontSize:10, color:"var(--text-secondary)" }}>{formatBSNepali(getTodayBS())}</p>
            </div>
          </div>
          <div style={{ display:"flex", gap:8, alignItems:"center" }}>
            <button onClick={() => setDark(!dark)} style={{ background:"var(--bg-muted)", border:"none", borderRadius:99, width:34, height:34, display:"flex", alignItems:"center", justifyContent:"center", cursor:"pointer", color:"var(--text-secondary)" }}>
              <Icon name={dark ? "sun" : "moon"} size={16} />
            </button>
            <button style={{ background:"var(--bg-muted)", border:"none", borderRadius:99, width:34, height:34, display:"flex", alignItems:"center", justifyContent:"center", cursor:"pointer", color:"var(--text-secondary)", position:"relative" }}>
              <Icon name="bell" size={16} />
              <div style={{ position:"absolute", top:6, right:6, width:7, height:7, borderRadius:"50%", background:"#ef4444", border:"2px solid var(--bg-card)" }} />
            </button>
          </div>
        </div>

        {/* Page Content */}
        <div style={{ padding:"0 16px 80px", maxWidth:500, margin:"0 auto" }}>
          <div className="page-content" key={page}>
            {pageComponents[page]}
          </div>
        </div>

        {/* Bottom Nav */}
        <div style={{ position:"fixed", bottom:0, left:0, right:0, background:"var(--bg-card)", borderTop:"1px solid var(--border)", padding:"8px 0 12px", display:"flex", justifyContent:"space-around", zIndex:50 }}>
          {navItems.slice(0,5).map(n => (
            <button key={n.id} onClick={() => setPage(n.id)} style={{ display:"flex", flexDirection:"column", alignItems:"center", gap:3, background:"none", border:"none", cursor:"pointer", flex:1, color: page === n.id ? "var(--accent)" : "var(--text-secondary)" }}>
              <Icon name={n.icon} size={20} color={page === n.id ? "var(--accent)" : "var(--text-secondary)"} />
              <span style={{ fontSize:10, fontWeight: page === n.id ? 700 : 500, fontFamily:"inherit" }}>{n.label}</span>
            </button>
          ))}
        </div>
        </div>
      </div>
    </>
  );
}
