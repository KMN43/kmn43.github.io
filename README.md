import { useState } from "react";

const stops = [
  { id: 0,  day: "1–4", label: "NEW YORK",        code: "NYC", x: 570, y: 100, color: "#FF4D4D", emoji: "🗽",
    desc: "4 días en la Gran Manzana: Times Square, Brooklyn Bridge, MoMA, High Line, Central Park, Harlem y el ferry gratuito a la Estatua de la Libertad." },
  { id: 1,  day: "5",   label: "PHILADELPHIA",    code: "PHL", x: 520, y: 200, color: "#FF9A3C", emoji: "🔔",
    desc: "Liberty Bell, Independence Hall y los Rocky Steps. Come en el Reading Terminal Market, el mejor mercado cubierto del noreste." },
  { id: 2,  day: "6",   label: "WASHINGTON D.C.", code: "D.C.", x: 470, y: 300, color: "#FFD700", emoji: "🏛️",
    desc: "National Mall completo: Lincoln Memorial, Washington Monument, Vietnam Memorial. Smithsonian gratis. Cena en Georgetown junto al canal." },
  { id: 3,  day: "7",   label: "SHENANDOAH NP",   code: "SNP", x: 390, y: 380, color: "#7FD67F", emoji: "⛰️",
    desc: "Skyline Drive por los Blue Ridge Mountains. Paradas en los overlooks y caminata a cascadas. Noche en Luray, cerca de las famosas grutas." },
  { id: 4,  day: "8",   label: "ASHEVILLE",        code: "AVL", x: 310, y: 460, color: "#A78BFA", emoji: "🍺",
    desc: "La ciudad más bohemia del sur: Biltmore Estate, River Arts District y la mayor concentración de cervecerías artesanales del país. Música en vivo en Lexington Ave." },
  { id: 5,  day: "9",   label: "GREAT SMOKY MTS.", code: "GSM", x: 230, y: 540, color: "#34D399", emoji: "🐻",
    desc: "El parque más visitado de EE.UU.: niebla mágica, osos negros y Clingmans Dome (2025 m). Baja a Gatlinburg para cenar entre souvenirs y pancakes." },
  { id: 6,  day: "10",  label: "CHARLESTON",       code: "CHS", x: 230, y: 660, color: "#FBBF24", emoji: "🌸",
    desc: "La ciudad más bella del sur: Rainbow Row, French Quarter colonial, Magnolia Plantation. Imprescindible: shrimp & grits y she-crab soup." },
  { id: 7,  day: "11",  label: "OUTER BANKS",      code: "OBX", x: 570, y: 580, color: "#38BDF8", emoji: "🏖️",
    desc: "Playas salvajes, Cape Hatteras (el faro más alto de EE.UU.), dunas de Jockey's Ridge y caballos salvajes. Memorial de los hermanos Wright en Kill Devil Hills." },
  { id: 8,  day: "12",  label: "ANNAPOLIS",        code: "ANP", x: 510, y: 440, color: "#60A5FA", emoji: "⚓",
    desc: "Puerto colonial, capitolio estatal más antiguo del país en uso continuo, Academia Naval y el cangrejo azul del Chesapeake: la experiencia gastronómica definitiva de Maryland." },
  { id: 9,  day: "13",  label: "HUDSON VALLEY",    code: "HV",  x: 540, y: 320, color: "#F9A8D4", emoji: "🏞️",
    desc: "Dia Beacon (arte contemporáneo en una fábrica junto al río), Rhinebeck con encanto bohemio, los Catskills y vistas al Hudson envueltas en colinas." },
  { id: 10, day: "14",  label: "NEW YORK",          code: "NYC", x: 570, y: 200, color: "#FF4D4D", emoji: "🌆",
    desc: "Regreso a la Gran Manzana. Chelsea Market, atardecer en el High Line, cena de celebración y devolución del coche. Fin del road trip." },
];

// Segments using only H, V, 45° lines. Points define the polyline.
const segments = [
  // NYC internal connector (vertical, red)
  { color: "#FF4D4D", points: [[570,100],[570,200]] },
  // NYC → PHL (diagonal 45° then vertical)
  { color: "#FF9A3C", points: [[570,100],[570,150],[520,200]] },
  // PHL → DC (diagonal)
  { color: "#FFD700", points: [[520,200],[520,250],[470,300]] },
  // DC → SNP (diagonal)
  { color: "#7FD67F", points: [[470,300],[390,380]] },
  // SNP → AVL (diagonal)
  { color: "#A78BFA", points: [[390,380],[310,460]] },
  // AVL → GSM (diagonal)
  { color: "#34D399", points: [[310,460],[230,540]] },
  // GSM → CHS (vertical)
  { color: "#FBBF24", points: [[230,540],[230,660]] },
  // CHS → OBX (H then diagonal)
  { color: "#38BDF8", points: [[230,660],[230,620],[570,620],[570,580]] },
  // OBX → ANP (vertical then H)
  { color: "#60A5FA", points: [[570,580],[570,440],[510,440]] },
  // ANP → HV (diagonal then H)
  { color: "#F9A8D4", points: [[510,440],[510,320],[540,320]] },
  // HV → NYC return (H then vertical)
  { color: "#FF4D4D", points: [[540,320],[570,320],[570,200]] },
];

const coastlinePoints = [
  [638,60],[648,110],[658,165],[652,220],[645,280],[638,340],
  [636,390],[642,435],[648,475],[646,515],[648,555],[652,595],
  [656,635],[648,675],[638,715],[618,750],[588,780],[548,805],
  [498,818],[440,810],[380,800]
];

const hudsonRiver = [[558,60],[560,100],[558,140],[552,180],[548,210]];
const chesRiver   = [[485,290],[488,330],[486,370],[482,410],[478,450],[474,490]];

export default function MetroRoadTrip() {
  const [selected, setSelected] = useState(null);
  const sel = selected !== null ? stops[selected] : null;

  const polyStr = (pts) => pts.map(p => p.join(",")).join(" ");
  const segPath = (pts) => pts.map((p,i)=>(i===0?`M${p[0]},${p[1]}`:`L${p[0]},${p[1]}`)).join(" ");

  return (
    <div style={{
      fontFamily: "'Courier New', monospace",
      background: "#070b0f",
      minHeight: "100vh",
      color: "#e0e8f0",
      display: "flex",
      flexDirection: "column",
      alignItems: "center",
      padding: "20px 12px",
    }}>

      <div style={{ textAlign:"center", marginBottom:"16px" }}>
        <div style={{ fontSize:"0.6rem", letterSpacing:"0.35em", color:"#38BDF8", textTransform:"uppercase", marginBottom:"3px" }}>
          East Coast · 14 días · ~3500 km
        </div>
        <h1 style={{ fontSize:"1.5rem", fontWeight:900, letterSpacing:"0.1em", color:"#fff", margin:0, textTransform:"uppercase" }}>
          New York & The South
        </h1>
        <div style={{ fontSize:"0.55rem", letterSpacing:"0.3em", color:"#1e3a50", marginTop:"3px" }}>
          TRANSIT MAP — SIMPLIFIED GEOGRAPHY
        </div>
      </div>

      <div style={{ display:"flex", gap:"18px", width:"100%", maxWidth:"1060px", alignItems:"flex-start", flexWrap:"wrap", justifyContent:"center" }}>

        {/* MAP */}
        <div style={{
          background:"#0b111a", border:"1px solid #152030",
          borderRadius:"3px", overflow:"hidden", flexShrink:0,
          boxShadow:"0 0 80px #000a"
        }}>
          <svg width="700" height="840" viewBox="0 0 700 840" style={{display:"block"}}>
            <defs>
              <pattern id="grid" width="40" height="40" patternUnits="userSpaceOnUse">
                <path d="M40 0L0 0 0 40" fill="none" stroke="#0f1a24" strokeWidth="0.5"/>
              </pattern>
              <filter id="glow" x="-50%" y="-50%" width="200%" height="200%">
                <feGaussianBlur stdDeviation="3" result="b"/>
                <feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge>
              </filter>
            </defs>

            <rect width="700" height="840" fill="#0b111a"/>
            <rect width="700" height="840" fill="url(#grid)"/>

            {/* Ocean fill */}
            <polygon
              points={[...coastlinePoints,[700,840],[700,60]].map(p=>p.join(",")).join(" ")}
              fill="#08131e" opacity="0.9"
            />

            {/* Coastline */}
            <polyline points={polyStr(coastlinePoints)} fill="none" stroke="#0e2840" strokeWidth="1.5" strokeDasharray="5 4"/>

            {/* Rivers */}
            <polyline points={polyStr(hudsonRiver)} fill="none" stroke="#0e2840" strokeWidth="1.2" opacity="0.7"/>
            <text x="564" y="152" fontSize="7" fill="#0e2840" letterSpacing="1.5" transform="rotate(-75,564,152)">HUDSON</text>
            <polyline points={polyStr(chesRiver)} fill="none" stroke="#0e2840" strokeWidth="1.2" opacity="0.6"/>
            <text x="492" y="380" fontSize="7" fill="#0e2840" letterSpacing="1" transform="rotate(-75,492,380)">CHESAPEAKE</text>

            {/* State labels */}
            {[
              [575, 72, "NEW YORK"],
              [450, 160, "NEW JERSEY"],
              [430, 240, "PENNSYLVANIA"],
              [390, 330, "VIRGINIA"],
              [290, 410, "W. VIRGINIA"],
              [210, 490, "TENNESSEE"],
              [190, 620, "SOUTH CAROLINA"],
              [540, 510, "NORTH CAROLINA"],
              [490, 360, "MARYLAND"],
            ].map(([x, y, name]) => (
              <text key={name} x={x} y={y} fontSize="7" fill="#0f2030"
                textAnchor="middle" letterSpacing="1.5" style={{userSelect:"none"}}>
                {name}
              </text>
            ))}

            {/* Atlantic */}
            <text x="670" y="430" fontSize="8" fill="#081520" textAnchor="middle"
              transform="rotate(90,670,430)" letterSpacing="3" style={{userSelect:"none"}}>
              ATLANTIC OCEAN
            </text>

            {/* ROUTE LINES */}
            {segments.map((seg, i) => (
              <path key={i} d={segPath(seg.points)}
                fill="none" stroke={seg.color} strokeWidth="4"
                strokeLinecap="square" opacity="0.82"/>
            ))}

            {/* STATIONS */}
            {stops.map((stop, i) => {
              const isSel = selected === i;
              const isNYC = i === 0 || i === 10;
              const r = isNYC ? 10 : 8;
              const onLeft = stop.x > 400;
              const lx = onLeft ? stop.x - 15 : stop.x + 15;
              const anchor = onLeft ? "end" : "start";

              return (
                <g key={i} onClick={() => setSelected(selected===i?null:i)} style={{cursor:"pointer"}}>
                  {isSel && <circle cx={stop.x} cy={stop.y} r="22" fill={stop.color} opacity="0.12"/>}
                  {/* White square tick */}
                  <rect x={stop.x - r} y={stop.y - r} width={r*2} height={r*2}
                    fill={isSel ? stop.color : "#0b111a"}
                    stroke={stop.color} strokeWidth={isSel ? 2.5 : 2}
                    filter={isSel ? "url(#glow)" : ""}
                  />
                  {isNYC && (
                    <rect x={stop.x-4} y={stop.y-4} width="8" height="8" fill={isSel?"#fff":stop.color}/>
                  )}
                  {!isNYC && (
                    <circle cx={stop.x} cy={stop.y} r="3" fill={isSel?"#fff":stop.color}/>
                  )}

                  {/* Tick line */}
                  <line x1={stop.x} y1={stop.y}
                    x2={onLeft ? stop.x-11 : stop.x+11} y2={stop.y}
                    stroke={stop.color} strokeWidth="1.5" opacity="0.6"/>

                  {/* Labels */}
                  <text x={lx} y={stop.y - 5} textAnchor={anchor}
                    fontSize="9.5" fontWeight="900" fill={isSel ? stop.color : "#b8d0e4"}
                    letterSpacing="0.06em" style={{userSelect:"none"}}>
                    {stop.label}
                  </text>
                  <text x={lx} y={stop.y + 8} textAnchor={anchor}
                    fontSize="7.5" fill="#1e4060" letterSpacing="0.04em"
                    style={{userSelect:"none"}}>
                    día {stop.day} · {stop.code}
                  </text>
                </g>
              );
            })}

            {/* LEGEND */}
            <rect x="12" y="744" width="210" height="84" rx="2" fill="#080e14" stroke="#152030" strokeWidth="1"/>
            <text x="22" y="761" fontSize="7.5" fill="#38BDF8" letterSpacing="2.5" fontWeight="700">LEYENDA DE LÍNEAS</text>
            {[
              ["#FF4D4D","NYC — Inicio y Regreso"],
              ["#FF9A3C","NYC → Washington"],
              ["#7FD67F","Appalachian Route"],
              ["#38BDF8","Coastal Return (sur)"],
            ].map(([col, name], i) => (
              <g key={i}>
                <line x1="20" y1={772+i*15} x2="44" y2={772+i*15}
                  stroke={col} strokeWidth="3.5" strokeLinecap="square"/>
                <text x="50" y={776+i*15} fontSize="8" fill="#4a7090">{name}</text>
              </g>
            ))}
            <text x="12" y="840" fontSize="0" fill="none"/>

            {/* Watermark */}
            <text x="688" y="832" textAnchor="end" fontSize="6.5" fill="#0f2030" letterSpacing="1.5">
              NYC EAST COAST ROAD TRIP · 14 DÍAS
            </text>
          </svg>
        </div>

        {/* PANEL */}
        <div style={{ flex:"1", minWidth:"260px", maxWidth:"330px" }}>

          <div style={{
            background:"#0b111a",
            border:`1px solid ${sel ? sel.color+"66" : "#152030"}`,
            borderRadius:"3px", padding:"18px", marginBottom:"12px",
            minHeight:"190px",
            boxShadow: sel ? `0 0 40px ${sel.color}14` : "none",
            transition:"all 0.3s"
          }}>
            {!sel ? (
              <div style={{ color:"#152a40", fontSize:"0.82rem", lineHeight:1.9, textAlign:"center", paddingTop:"34px" }}>
                <div style={{ fontSize:"2rem", marginBottom:"10px", color:"#0e2840" }}>◈</div>
                Selecciona una parada<br/>en el mapa.
              </div>
            ) : (
              <>
                <div style={{ display:"flex", alignItems:"center", gap:"10px", marginBottom:"12px" }}>
                  <div style={{
                    width:"34px", height:"34px",
                    background:sel.color+"22", border:`2px solid ${sel.color}`,
                    display:"flex", alignItems:"center", justifyContent:"center",
                    fontSize:"1rem", flexShrink:0
                  }}>
                    {sel.emoji}
                  </div>
                  <div>
                    <div style={{ fontSize:"0.58rem", color:sel.color, letterSpacing:"0.22em", fontWeight:700, textTransform:"uppercase" }}>
                      DÍA {sel.day} · {sel.code}
                    </div>
                    <div style={{ fontSize:"0.95rem", fontWeight:900, color:"#fff", letterSpacing:"0.05em" }}>
                      {sel.label}
                    </div>
                  </div>
                </div>
                <p style={{ fontSize:"0.78rem", color:"#5a8aaa", lineHeight:1.8, margin:0 }}>
                  {sel.desc}
                </p>
                <div style={{ display:"flex", gap:"8px", marginTop:"14px" }}>
                  <button onClick={() => setSelected(Math.max(0, selected-1))}
                    disabled={selected===0}
                    style={{ flex:1, padding:"7px", background:"#0f1a24", border:"1px solid #1e3040",
                      color:selected>0?"#7aa0b8":"#1e3040", cursor:selected>0?"pointer":"default",
                      fontSize:"0.75rem", letterSpacing:"0.05em", borderRadius:"2px" }}>
                    ← PREV
                  </button>
                  <button onClick={() => setSelected(Math.min(stops.length-1, selected+1))}
                    disabled={selected===stops.length-1}
                    style={{ flex:1, padding:"7px",
                      background:selected<stops.length-1 ? sel.color : "#0f1a24",
                      border:"none", color:"#fff",
                      cursor:selected<stops.length-1?"pointer":"default",
                      fontSize:"0.75rem", fontWeight:700, letterSpacing:"0.05em", borderRadius:"2px" }}>
                    NEXT →
                  </button>
                </div>
              </>
            )}
          </div>

          {/* Stop list */}
          <div style={{ background:"#0b111a", border:"1px solid #152030", borderRadius:"3px", padding:"12px" }}>
            <div style={{ fontSize:"0.58rem", color:"#0e2840", letterSpacing:"0.3em", marginBottom:"8px", fontWeight:700 }}>
              TODAS LAS PARADAS
            </div>
            {stops.map((s, i) => (
              <div key={i} onClick={() => setSelected(selected===i?null:i)}
                style={{
                  display:"flex", alignItems:"center", gap:"7px",
                  padding:"5px 7px", marginBottom:"2px",
                  background:selected===i ? s.color+"18" : "transparent",
                  borderLeft:`3px solid ${selected===i ? s.color : "#0f1a24"}`,
                  cursor:"pointer", transition:"all 0.15s"
                }}>
                <span style={{ fontSize:"0.7rem" }}>{s.emoji}</span>
                <span style={{ fontSize:"0.7rem", fontWeight:700, color:selected===i ? s.color:"#2a5a78", letterSpacing:"0.06em" }}>
                  D{s.day}
                </span>
                <span style={{ fontSize:"0.68rem", color:"#1a3a54", flex:1, overflow:"hidden", whiteSpace:"nowrap", textOverflow:"ellipsis" }}>
                  {s.label}
                </span>
                <span style={{ width:"5px", height:"5px", borderRadius:"50%", background:s.color, flexShrink:0, opacity:0.6 }}/>
              </div>
            ))}
          </div>
        </div>
      </div>
    </div>
  );
}
