import React, { useState, useEffect } from 'react';
import { 
  Info, Settings, Ruler, CheckCircle, AlertCircle, RefreshCw, 
  MessageSquare, Sparkles, FileText, X, Box, Layers, 
  ShieldCheck, ArrowDownToLine, TriangleAlert, RotateCw, Eye, Activity, Shield, ScanEye
} from 'lucide-react';

// --- GEMINI API CONFIG ---
// Sila masukkan API Key anda di sini
const apiKey = ""; 

// --- WAREHOUSE DATA ---
const AVAILABLE_PLATES = [2, 3, 4, 5, 6, 8, 10, 12, 16, 20, 25, 30, 35, 40, 50, 60, 75, 115, 150, 200];

// --- EN 15085-3 CLASS DETAILS (Table 3) ---
const CLASS_DETAILS = {
    "CP A": {
        stress: "High",
        safety: "High",
        inspection: "CT 1",
        volumetric: "100%",
        surface: "100%",
        visual: "100%",
        desc: "Highest safety requirements. Full testing."
    },
    "CP B1": {
        stress: "Medium",
        safety: "High",
        inspection: "CT 2",
        volumetric: "10%",
        surface: "10%",
        visual: "100%",
        desc: "High safety, medium stress."
    },
    "CP B2": {
        stress: "High",
        safety: "Medium",
        inspection: "CT 2",
        volumetric: "10%",
        surface: "10%",
        visual: "100%",
        desc: "High stress, medium safety."
    },
    "CP C1": {
        stress: "Low",
        safety: "High",
        inspection: "CT 2",
        volumetric: "10%",
        surface: "10%",
        visual: "100%",
        desc: "High safety, low stress."
    },
    "CP C2": {
        stress: "High / Medium",
        safety: "Low / Medium",
        inspection: "CT 3",
        volumetric: "Not required",
        surface: "Not required",
        visual: "100%",
        desc: "General usage. Visual check only."
    }
};

// --- DATA & STANDARDS (EN 15085-3) ---
const WELD_DATA = {
  butt: {
    label: "Butt Joint",
    types: [
      {
        id: '2a',
        name: 'V-Weld (Single V)',
        symbol: 'V',
        min_t: 3,
        max_t: 15,
        availableClasses: ["CP A", "CP B1", "CP B2", "CP C1", "CP C2"],
        penetration: "Full Penetration",
        description: "Single V preparation.",
        fig_type: "butt_v"
      },
      {
        id: '2b',
        name: 'Y-Weld (Single V with Root)',
        symbol: 'Y',
        min_t: 5,
        max_t: 20,
        availableClasses: ["CP C2"], 
        penetration: "Partial Penetration", 
        description: "Single V with root face, no gap.",
        fig_type: "butt_y"
      },
      {
        id: '6',
        name: 'X-Weld (Double V)',
        symbol: 'X',
        min_t: 12,
        max_t: 100,
        availableClasses: ["CP A", "CP B1", "CP B2", "CP C1", "CP C2"],
        penetration: "Full Penetration",
        description: "Double V preparation.",
        fig_type: "butt_x"
      }
    ]
  },
  t_joint: {
    label: "T-Joint",
    hasBoxOption: true,
    types: [
      {
        id: '13a',
        name: 'Fillet Weld',
        symbol: 'F',
        min_t: 2, 
        max_t: 50,
        availableClasses: ["CP C2"], 
        penetration: "Partial Penetration",
        description: "No preparation required.",
        fig_type: "t_fillet"
      },
      {
        id: '3a',
        name: 'HV Weld (Single Bevel)',
        symbol: 'HV',
        min_t: 3,
        max_t: 15,
        availableClasses: ["CP A", "CP B1", "CP B2", "CP C1", "CP C2"],
        penetration: "Full Penetration",
        description: "Single bevel on web.",
        fig_type: "t_hv",
        allowBox: true 
      },
      {
        id: '11a',
        name: 'HY Weld',
        symbol: 'HY',
        min_t: 3,
        max_t: 15,
        availableClasses: ["CP C2"], 
        penetration: "Partial Penetration",
        description: "Single bevel with large root face.",
        fig_type: "t_hy"
      },
      {
        id: '7',
        name: 'DHV Weld (Double Bevel/K)',
        symbol: 'K',
        min_t: 12,
        max_t: 100,
        availableClasses: ["CP A", "CP B1", "CP B2", "CP C1", "CP C2"],
        penetration: "Full Penetration",
        description: "Double bevel (K-Weld) on web.",
        fig_type: "t_k"
      }
    ]
  },
  corner: {
    label: "Corner Joint",
    hasOptions: true, 
    types: [
      {
        id: '13c',
        name: 'Corner Fillet',
        symbol: 'F',
        min_t: 2,
        max_t: 20,
        availableClasses: ["CP C1", "CP C2"],
        penetration: "Partial Penetration",
        description: "Outside & Inside fillet weld.",
        fig_type: "corner_fillet"
      },
      {
        id: '13d',
        name: 'Corner Seam Weld (V)',
        symbol: 'V',
        min_t: 3,
        max_t: 20,
        availableClasses: ["CP A", "CP B1", "CP B2", "CP C1", "CP C2"],
        penetration: "Full Penetration",
        description: "Corner with V preparation.",
        fig_type: "corner_v"
      }
    ]
  },
  lap: {
    label: "Lap Joint",
    types: [
      {
        id: '13e',
        name: 'Lap Seam Weld',
        symbol: 'F',
        min_t: 2,
        max_t: 15,
        availableClasses: ["CP C1", "CP C2"],
        penetration: "Partial Penetration",
        description: "Standard lap fillet.",
        fig_type: "lap"
      }
    ]
  },
  three_member: {
    label: "3-Member Joint",
    hasT3: true, 
    types: [
      {
        id: '12',
        name: 'Joint between 3 Members',
        symbol: 'V',
        min_t: 4,
        max_t: 20,
        availableClasses: ["CP A", "CP B1", "CP B2", "CP C1", "CP C2"],
        penetration: "Full Penetration",
        description: "T-Butt combination.",
        fig_type: "three_member"
      }
    ]
  }
};

// --- HELPER FUNCTIONS ---
const calculateWeldSize = (weldType, t1, t2, rootFace) => {
    const t_min = Math.min(Number(t1), Number(t2));
    
    if (weldType.symbol === 'F') {
        // Fillet Logic: Maintain "a" prefix
        if (t_min <= 17) {
            return `a${Math.ceil(0.7 * t_min)}`;
        } else if (t_min >= 18 && t_min <= 19) {
            return `a12`;
        } else {
            // New logic for thickness >= 20mm
            // Formula: tan(30deg) * (thickness - 2), rounded up
            const angleInRadians = 30 * (Math.PI / 180);
            const calculatedValue = Math.tan(angleInRadians) * (t_min - 2);
            return `a${Math.ceil(calculatedValue)}`;
        }
    } else if (weldType.symbol === 'HY' || weldType.symbol === 'Y') {
        // HY/Y Logic: Thickness - Root Face
        return `${Number(t1) - Number(rootFace)}`;
    } else if (weldType.symbol === 'HV' || weldType.symbol === 'V' || weldType.symbol === 'K' || weldType.symbol === 'X') {
        // HV/Groove Logic: Value = plate thickness
        return `${t1}`; 
    }
    return "";
};

// --- API HELPER ---
async function generateGeminiContent(prompt, systemInstruction = "") {
  try {
    // Basic check if API key is empty
    if (!apiKey) {
        console.warn("API Key is missing. Please set the apiKey variable.");
        return "Sila masukkan API Key Gemini anda dalam kod untuk menggunakan ciri ini.";
    }

    const response = await fetch(
      `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-exp:generateContent?key=${apiKey}`,
      {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          contents: [{ parts: [{ text: prompt }] }],
          systemInstruction: { parts: [{ text: systemInstruction }] },
        }),
      }
    );
    if (!response.ok) throw new Error(`API Error: ${response.status}`);
    const data = await response.json();
    return data.candidates?.[0]?.content?.parts?.[0]?.text || "Maaf, tiada respons.";
  } catch (error) {
    console.error("Gemini Error:", error);
    return "Ralat menyambung ke AI.";
  }
}

// --- SUB-COMPONENTS ---

const AIChatConsultant = () => {
    const [query, setQuery] = useState("");
    const [response, setResponse] = useState("");
    const [isLoading, setIsLoading] = useState(false);
    const [isOpen, setIsOpen] = useState(false);

    const handleAsk = async () => {
        if (!query.trim()) return;
        setIsLoading(true);
        const sysPrompt = "You are an Expert Welding Engineer in EN 15085 standard for locomotives. Answer concisely, technically, and professionally.";
        const result = await generateGeminiContent(query, sysPrompt);
        setResponse(result);
        setIsLoading(false);
    };

    if (!isOpen) {
        return (
            <button onClick={() => setIsOpen(true)} className="fixed bottom-6 right-6 bg-indigo-600 text-white p-4 rounded-full shadow-lg hover:bg-indigo-700 transition-all flex items-center gap-2 z-50 animate-bounce">
                <Sparkles size={20} /> Ask AI
            </button>
        );
    }

    return (
        <div className="fixed bottom-6 right-6 w-80 bg-white rounded-xl shadow-2xl border border-indigo-100 flex flex-col z-50 overflow-hidden h-96">
            <div className="bg-indigo-600 text-white p-3 flex justify-between items-center">
                <h3 className="text-sm font-bold flex items-center gap-2"><Sparkles size={16}/> AI Welding Consultant</h3>
                <button onClick={() => setIsOpen(false)}><X size={16}/></button>
            </div>
            <div className="p-4 flex-1 overflow-y-auto bg-slate-50 text-sm">
                {response ? (
                    <div className="bg-white p-3 rounded-lg border border-slate-200 text-sm text-slate-700 shadow-sm">
                        <p className="font-bold text-indigo-600 text-xs mb-1">AI Answer:</p>
                        {response}
                    </div>
                ) : (
                    <p className="text-gray-400 text-center mt-4">Tanya tentang EN 15085, kecacatan kimpalan, atau bahan...</p>
                )}
            </div>
            <div className="p-3 border-t bg-white flex gap-2">
                <input type="text" value={query} onChange={(e) => setQuery(e.target.value)} className="flex-1 border rounded px-2 py-1 text-sm" onKeyPress={(e) => e.key === 'Enter' && handleAsk()}/>
                <button onClick={handleAsk} disabled={isLoading} className="bg-indigo-600 text-white p-2 rounded">{isLoading ? <RefreshCw className="animate-spin" size={16}/> : <MessageSquare size={16}/>}</button>
            </div>
        </div>
    );
};

const JointVisualizer = ({ jointType, weldTypeData, cornerOption, activeField, isBoxSection }) => {
  const strokeColor = "#334155"; 
  const plateFill = "#cbd5e1"; 
  const plate2Fill = "#94a3b8";
  const plate3Fill = "#e2e8f0"; 
  const plate4Fill = "#cbd5e1"; 
  const getStroke = (field) => activeField === field ? "#ef4444" : strokeColor; 
  const getWidth = (field) => activeField === field ? "3" : "2";
  const fig = weldTypeData?.fig_type || "unknown";

  return (
    <div className="w-full h-64 bg-gray-50 border border-gray-200 rounded-lg flex items-center justify-center overflow-hidden relative">
      <svg viewBox="0 0 300 200" className="w-full h-full">
        {isBoxSection ? (
            <g transform="translate(70, 30)">
                <rect x="0" y="0" width="160" height="15" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                <rect x="0" y="125" width="160" height="15" fill={plate4Fill} stroke={getStroke('t4')} strokeWidth={getWidth('t4')} />
                <path d="M25,15 L35,15 L35,125 L25,125 L15,115 L15,25 Z" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                <line x1="15" y1="25" x2="25" y2="15" stroke="red" strokeDasharray="2,2"/>
                <line x1="15" y1="115" x2="25" y2="125" stroke="red" strokeDasharray="2,2"/>
                <path d="M125,15 L135,15 L145,25 L145,115 L135,125 L125,125 Z" fill={plate3Fill} stroke={getStroke('t3')} strokeWidth={getWidth('t3')} />
                <line x1="135" y1="15" x2="145" y2="25" stroke="red" strokeDasharray="2,2"/>
                <line x1="145" y1="115" x2="135" y2="125" stroke="red" strokeDasharray="2,2"/>
                <text x="80" y="-5" textAnchor="middle" className="text-[10px]" fill={getStroke('t1')}>t1 (Top)</text>
                <text x="80" y="150" textAnchor="middle" className="text-[10px]" fill={getStroke('t4')}>t4 (Bottom)</text>
                <text x="0" y="70" textAnchor="end" className="text-[10px]" fill={getStroke('t2')}>t2 (Left)</text>
                <text x="160" y="70" textAnchor="start" className="text-[10px]" fill={getStroke('t3')}>t3 (Right)</text>
            </g>
        ) : (
            <>
                {jointType === 'butt' && (
                  <g transform="translate(50, 30)">
                    {/* LEFT PLATE */}
                    {fig === 'butt_v' ? (
                        <path d="M20,60 L80,60 L95,80 L95,90 L20,90 Z" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    ) : fig === 'butt_y' ? (
                        // Y-Weld: Similar to V but touching at bottom (Root face). Beveled plates.
                        <path d="M20,60 L80,60 L100,80 L100,90 L20,90 Z" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    ) : fig === 'butt_x' ? (
                        <path d="M20,60 L80,60 L95,75 L80,90 L20,90 Z" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    ) : (
                        <rect x="20" y="60" width="75" height="30" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    )}

                    {/* RIGHT PLATE */}
                    {fig === 'butt_v' ? (
                        <path d="M180,60 L120,60 L105,80 L105,90 L180,90 Z" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                    ) : fig === 'butt_y' ? (
                        <path d="M180,60 L120,60 L100,80 L100,90 L180,90 Z" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                    ) : fig === 'butt_x' ? (
                        <path d="M180,60 L120,60 L105,75 L120,90 L180,90 Z" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                    ) : (
                        <rect x="105" y="60" width="75" height="30" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                    )}

                    <text x="50" y="45" className="text-[10px]" fill={getStroke('t1')}>t1</text>
                    <text x="140" y="45" className="text-[10px]" fill={getStroke('t2')}>t2</text>
                    
                    {/* Dotted lines for V */}
                    {fig === 'butt_v' && <line x1="80" y1="60" x2="95" y2="80" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>}
                    {fig === 'butt_v' && <line x1="120" y1="60" x2="105" y2="80" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>}

                    {/* Dotted lines for Y */}
                    {fig === 'butt_y' && (
                        <>
                            <line x1="80" y1="60" x2="100" y2="80" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
                            <line x1="120" y1="60" x2="100" y2="80" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
                        </>
                    )}

                    {/* Dotted lines for X (Double V) */}
                    {fig === 'butt_x' && (
                        <>
                            <line x1="80" y1="60" x2="95" y2="75" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
                            <line x1="80" y1="90" x2="95" y2="75" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
                            <line x1="120" y1="60" x2="105" y2="75" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
                            <line x1="120" y1="90" x2="105" y2="75" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
                        </>
                    )}
                  </g>
                )}
                {jointType === 't_joint' && (
                  <g transform="translate(50, 30)">
                    {/* Horizontal Plate (Flange) */}
                    <rect x="40" y="100" width="120" height="20" fill={plateFill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                    
                    {/* Vertical Plate (Web) */}
                    {fig === 't_k' ? (
                         // Double Bevel (K)
                         <path d="M90,20 L110,20 L110,80 L100,100 L90,80 Z" fill={plate2Fill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    ) : fig.includes('hv') || fig.includes('hy') ? (
                         // Single Bevel
                         <path d="M90,20 L110,20 L110,95 L100,95 L90,80 Z" fill={plate2Fill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    ) : (
                         // No Bevel (Square)
                         <rect x="90" y="20" width="20" height="80" fill={plate2Fill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    )}

                    <text x="175" y="115" className="text-[10px]" fill={getStroke('t2')}>t2</text>
                    <text x="60" y="50" className="text-[10px]" textAnchor="end" fill={getStroke('t1')}>t1</text>
                    
                    {/* Red Dashed Lines indicating Weld Prep */}
                    {(fig === 't_hv' || fig === 't_hy') && <line x1="90" y1="80" x2="100" y2="100" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>}
                    
                    {fig === 't_k' && (
                        <>
                            <line x1="90" y1="80" x2="100" y2="100" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
                            <line x1="110" y1="80" x2="100" y2="100" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
                        </>
                    )}
                  </g>
                )}
                {jointType === 'corner' && (
                   <g transform="translate(100, 70)">
                     {cornerOption === '1' ? (
                        <g>
                           <rect x="0" y="20" width="20" height="60" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                           <rect x="20" y="0" width="60" height="20" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                           <circle cx="20" cy="20" r="2" fill="red" opacity="0.5" />
                           <text x="-15" y="50" className="text-[10px]" textAnchor="end" fill={getStroke('t1')}>t1</text>
                           <text x="50" y="-10" className="text-[10px]" fill={getStroke('t2')}>t2</text>
                        </g>
                     ) : (
                        <g>
                           <rect x="0" y="20" width="20" height="60" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                           <rect x="10" y="0" width="80" height="20" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                           <text x="-15" y="50" className="text-[10px]" textAnchor="end" fill={getStroke('t1')}>t1</text>
                           <text x="60" y="-10" className="text-[10px]" fill={getStroke('t2')}>t2</text>
                           <line x1="10" y1="20" x2="20" y2="20" stroke="red" strokeWidth="1" strokeDasharray="1,1" opacity="0.5"/>
                        </g>
                     )}
                   </g>
                )}
                {jointType === 'lap' && (
                    <g transform="translate(90, 80)">
                        <rect x="0" y="20" width="100" height="20" fill={plateFill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                        <rect x="30" y="0" width="100" height="20" fill={plate2Fill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                        <text x="-10" y="35" className="text-[10px]" textAnchor="end" fill={getStroke('t2')}>t2</text>
                        <text x="140" y="15" className="text-[10px]" fill={getStroke('t1')}>t1</text>
                    </g>
                )}
                {jointType === 'three_member' && (
                    <g transform="translate(80, 60)">
                        <path d="M0,0 L70,0 L78,15 L0,15 Z" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                        <path d="M82,15 L90,0 L160,0 L160,15 Z" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                        {/* FIX: Updated t3 rect to use getStroke('t3') and getWidth('t3') and plate3Fill */}
                        <rect x="72" y="15" width="16" height="60" fill={plate3Fill} stroke={getStroke('t3')} strokeWidth={getWidth('t3')} />
                        <line x1="70" y1="0" x2="78" y2="15" stroke="red" strokeWidth="1" strokeDasharray="1,1" opacity="0.6"/>
                        <line x1="90" y1="0" x2="82" y2="15" stroke="red" strokeWidth="1" strokeDasharray="1,1" opacity="0.6"/>
                        <text x="40" y="-5" className="text-[10px]" fill={getStroke('t1')}>t1</text>
                        <text x="120" y="-5" className="text-[10px]" fill={getStroke('t2')}>t2</text>
                        <text x="95" y="50" className="text-[10px]" fill={getStroke('t3')}>t3</text>
                    </g>
                )}
            </>
        )}
      </svg>
    </div>
  );
};

// --- RESULT RENDERER (STABLE - NO DIMS) ---
const ResultRenderer = ({ data, inputs, isBoxSection, hasBacking }) => {
  const { jointType, cornerOption, angle, gap, rootFace } = inputs;
  const fig = data?.fig_type || "unknown";

  const plateFill = "url(#hatch)";
  const plate2Fill = "url(#hatch)";
  const plate3Fill = "url(#hatch)";
  const plate4Fill = "url(#hatch)";
  const weldFill = "#fca5a5";
  const backingColor = "#475569";

  const showLegend = !['t_fillet', 'corner_fillet', 'lap'].includes(fig);
  const showGap = !fig.includes('hy') && fig !== 'butt_y'; 

  return (
    <div className="flex flex-col h-full">
        <div className="bg-slate-50 px-3 py-2 border-b text-xs font-bold text-gray-500 uppercase tracking-wider">Welding View / Result</div>
        <div className="flex-1 relative flex items-center justify-center bg-gray-50">
            <svg viewBox="0 0 300 200" className="w-full h-full">
                <defs>
                    <pattern id="hatch" patternUnits="userSpaceOnUse" width="4" height="4"><path d="M-1,1 l2,-2 M0,4 l4,-4 M3,5 l2,-2" stroke="#a1a1aa" strokeWidth="1"/></pattern>
                    <marker id="arrowRed" markerWidth="6" markerHeight="6" refX="3" refY="3" orient="auto" markerUnits="strokeWidth">
                        <path d="M0,0 L0,6 L6,3 z" fill="#dc2626" />
                    </marker>
                </defs>
                
                {isBoxSection ? (
                    <g transform="translate(70, 30)">
                        <rect x="0" y="0" width="160" height="15" fill={plateFill} stroke="black" />
                        <rect x="0" y="125" width="160" height="15" fill={plateFill} stroke="black" />
                        <path d="M25,15 L35,15 L35,125 L25,125 L15,115 L15,25 Z" fill={plateFill} stroke="black" />
                        <path d="M125,15 L135,15 L145,25 L145,115 L135,125 L125,125 Z" fill={plateFill} stroke="black" />
                        {hasBacking && <><rect x="35" y="15" width="5" height="15" fill={backingColor} /><rect x="120" y="15" width="5" height="15" fill={backingColor} /></>}
                        <path d="M25,15 L15,25 L15,15 Z" fill={weldFill} stroke="black" /> 
                        <path d="M15,15 L5,15 L15,25 Z" fill={weldFill} stroke="black" opacity="0.7"/> 
                        <path d="M25,125 L15,115 L15,125 Z" fill={weldFill} stroke="black" />
                        <path d="M15,125 L5,125 L15,115 Z" fill={weldFill} stroke="black" opacity="0.7"/>
                        <path d="M35,125 L35,115 L45,125 Z" fill={weldFill} stroke="black" /> 
                        <path d="M135,15 L145,25 L145,15 Z" fill={weldFill} stroke="black" />
                        <path d="M145,15 L155,15 L145,25 Z" fill={weldFill} stroke="black" opacity="0.7"/>
                        <path d="M135,125 L145,115 L145,125 Z" fill={weldFill} stroke="black" />
                        <path d="M145,125 L155,125 L145,115 Z" fill={weldFill} stroke="black" opacity="0.7"/>
                        <path d="M125,125 L125,115 L115,125 Z" fill={weldFill} stroke="black" /> 
                    </g>
                ) : (
                    <>
                        {jointType === 'butt' && (
                            <g transform="translate(50, 30)">
                                {fig === 'butt_v' ? <path d="M20,60 L80,60 L95,80 L95,90 L20,90 Z" fill={plateFill} stroke="black" /> : <rect x="20" y="60" width="75" height="30" fill={plateFill} stroke="black" />}
                                {fig === 'butt_v' ? <path d="M180,60 L120,60 L105,80 L105,90 L180,90 Z" fill={plateFill} stroke="black" /> : <rect x="105" y="60" width="75" height="30" fill={plateFill} stroke="black" />}
                                {fig === 'butt_v' && <path d="M80,60 L120,60 L105,80 L95,80 Z" fill={weldFill} stroke="black" />}
                                {fig === 'butt_v' && <path d="M80,60 L120,60 L100,55 Z" fill={weldFill} stroke="black" opacity="0.5" />}
                                
                                {fig === 'butt_y' && (
                                    <>
                                        {/* Y-Weld: No Gap, Touching at root */}
                                        {/* Left Plate: Root Face at bottom */}
                                        <path d="M20,60 L80,60 L100,80 L100,90 L20,90 Z" fill={plateFill} stroke="black" />
                                        {/* Right Plate */}
                                        <path d="M180,60 L120,60 L100,80 L100,90 L180,90 Z" fill={plateFill} stroke="black" />
                                        {/* Weld Fill */}
                                        <path d="M80,60 L120,60 L100,80 Z" fill={weldFill} stroke="black" />
                                    </>
                                )}
                                
                                {/* X-Weld (Double V) - Visual Gap & Root Face Update */}
                                {fig === 'butt_x' && (
                                    <>
                                        {/* Left Plate: Gap 2px from center (100), Root 3px high from mid (75) */}
                                        <path d="M20,60 L80,60 L98,72 L98,78 L80,90 L20,90 Z" fill={plateFill} stroke="black" />
                                        {/* Right Plate */}
                                        <path d="M180,60 L120,60 L102,72 L102,78 L120,90 L180,90 Z" fill={plateFill} stroke="black" />
                                        {/* Weld Fills */}
                                        <path d="M80,60 L120,60 L102,72 L98,72 Z" fill={weldFill} stroke="black" />
                                        <path d="M80,90 L120,90 L102,78 L98,78 Z" fill={weldFill} stroke="black" />
                                        {/* Gap Fill */}
                                        <rect x="98" y="72" width="4" height="6" fill={weldFill} stroke="none" opacity="0.5"/>
                                    </>
                                )}
                                {hasBacking && fig !== 'butt_x' && fig !== 'butt_y' && <rect x="85" y="90" width="30" height="5" fill={backingColor} stroke="black"/>}
                            </g>
                        )}
                        {jointType === 't_joint' && (
                            <g transform="translate(50, 30)">
                                <rect x="40" y="100" width="120" height="20" fill={plateFill} stroke="black" />
                                
                                {fig === 't_k' ? (
                                     // Double Bevel (K) with Visible Root Face & Gap
                                     // Web bottom at 95 (gap of 5). Root face width 6 (center 100).
                                     <path d="M90,20 L110,20 L110,80 L103,95 L97,95 L90,80 Z" fill={plateFill} stroke="black" />
                                ) : fig.includes('hv') ? (
                                     // HV: Single Bevel with Gap (y=95)
                                     <path d="M90,20 L110,20 L110,95 L100,95 L90,80 Z" fill={plateFill} stroke="black" />
                                ) : fig.includes('hy') ? (
                                     // HY: Single Bevel NO Gap (y=100) - Touching plate
                                     <path d="M90,20 L110,20 L110,100 L100,100 L90,80 Z" fill={plateFill} stroke="black" />
                                ) : (
                                     // Square
                                     <rect x="90" y="20" width="20" height="80" fill={plateFill} stroke="black" />
                                )}

                                {fig === 't_fillet' && <><path d="M90,100 L90,80 L70,100 Z" fill={weldFill} stroke="black" /><path d="M110,100 L110,80 L130,100 Z" fill={weldFill} stroke="black" /></>}
                                
                                {fig.includes('hv') && <path d="M90,80 L100,95 L110,95 L110,100 L70,100 Z" fill={weldFill} stroke="black" />}
                                
                                {fig.includes('hy') && <path d="M90,80 L100,100 L70,100 Z" fill={weldFill} stroke="black" />}
                                
                                {fig === 't_k' && (
                                    <>
                                        {/* Left Weld: Bevel + Gap Penetration */}
                                        <path d="M90,80 L97,95 L97,100 L70,100 Z" fill={weldFill} stroke="black" />
                                        {/* Right Weld */}
                                        <path d="M110,80 L103,95 L103,100 L130,100 Z" fill={weldFill} stroke="black" />
                                        {/* Center Gap Fill */}
                                        <rect x="97" y="95" width="6" height="5" fill={weldFill} stroke="none" opacity="0.5"/>
                                    </>
                                )}
                                
                                {/* MODIFIED: Backing Bar for HV - Vertical behind web (x=110), sitting on flange (y=100) */}
                                {hasBacking && fig === 't_hv' && <rect x="110" y="80" width="5" height="20" fill={backingColor} stroke="black" />}
                                
                                {/* HV Dimension */}
                                {fig.includes('hv') && (
                                    <g>
                                       {/* Existing weld path */}
                                       <path d="M90,80 L100,95 L110,95 L110,100 L70,100 Z" fill={weldFill} stroke="black" />
                                       
                                       {/* NEW: Dimension b */}
                                       <line x1="112" y1="95" x2="122" y2="95" stroke="#ef4444" strokeWidth="0.5" />
                                       <line x1="112" y1="100" x2="122" y2="100" stroke="#ef4444" strokeWidth="0.5" />
                                       <line x1="120" y1="95" x2="120" y2="100" stroke="#ef4444" strokeWidth="0.5" />
                                       <text x="124" y="100" fontSize="9" fill="#ef4444" fontWeight="bold">b</text>
                                    </g>
                                )}
                            </g>
                        )}
                        {jointType === 'corner' && (
                            <g transform="translate(100, 70)">
                                 {cornerOption === '1' ? (
                                    <g>
                                         <rect x="0" y="20" width="20" height="60" fill={plateFill} stroke="black" />
                                         <rect x="20" y="0" width="60" height="20" fill={plateFill} stroke="black" />
                                         <path d="M0,20 L20,20 L20,0 Z" fill={weldFill} stroke="black" />
                                         <path d="M0,20 L-5,25 L25,-5 L20,0 Z" fill={weldFill} stroke="none" opacity="0.5" />
                                         <path d="M20,20 L20,30 L30,20 Z" fill={weldFill} stroke="black" />
                                    </g>
                                 ) : (
                                    <g>
                                        <rect x="0" y="20" width="20" height="60" fill={plateFill} stroke="black"/>
                                        <rect x="10" y="0" width="80" height="20" fill={plateFill} stroke="black"/>
                                        <path d="M0,20 L10,20 L10,0 Z" fill={weldFill} stroke="black"/> 
                                        <path d="M0,20 L-5,25 L15,-5 L10,0 Z" fill={weldFill} stroke="none" opacity="0.5"/> 
                                    </g>
                                 )}
                            </g>
                        )}
                        {jointType === 'lap' && (
                             <g transform="translate(90, 80)">
                                <rect x="0" y="20" width="100" height="20" fill={plateFill} stroke="black" />
                                <rect x="30" y="0" width="100" height="20" fill={plateFill} stroke="black" />
                                <path d="M30,20 L30,0 L10,20 Z" fill={weldFill} stroke="black"/>
                                <path d="M100,20 L130,20 L100,40 Z" fill={weldFill} stroke="black"/>
                             </g>
                        )}
                        {jointType === 'three_member' && (
                             <g transform="translate(80, 60)">
                                <path d="M0,0 L70,0 L78,15 L0,15 Z" fill={plateFill} stroke="black" />
                                <path d="M82,15 L90,0 L160,0 L160,15 Z" fill={plate2Fill} stroke="black" />
                                {/* FIX: Updated t3 rect to use getStroke('t3') and getWidth('t3') and plate3Fill */}
                                <rect x="72" y="15" width="16" height="60" fill={plate3Fill} stroke="black" />
                                {/* MODIFIED: Only top V-groove weld is shown in red */}
                                <path d="M70,0 L78,15 L82,15 L90,0 Z" fill={weldFill} stroke="black" />
                                <line x1="70" y1="0" x2="78" y2="15" stroke="red" strokeWidth="1" strokeDasharray="1,1" opacity="0.6"/>
                                <line x1="90" y1="0" x2="82" y2="15" stroke="red" strokeWidth="1" strokeDasharray="1,1" opacity="0.6"/>
                                <text x="40" y="-5" className="text-[10px]" fill="#334155">t1</text>
                                <text x="120" y="-5" className="text-[10px]" fill="#334155">t2</text>
                                <text x="95" y="50" className="text-[10px]" fill="#334155">t3</text>
                            </g>
                        )}
                    </>
                )}
                {showLegend && (
                    <g transform="translate(10, 175)">
                        <rect x="0" y="0" width="280" height="20" rx="4" fill="white" stroke="#94a3b8" strokeWidth="1" opacity="0.9" />
                        <text x="10" y="14" fontSize="10" fill="#334155" fontWeight="bold">α, Angle = <tspan fill="#dc2626">{angle}°</tspan></text>
                        {showGap ? (
                            <>
                                <text x="90" y="14" fontSize="10" fill="#334155" fontWeight="bold">b, Gap = <tspan fill="#dc2626">{gap} mm</tspan></text>
                                <text x="170" y="14" fontSize="10" fill="#334155" fontWeight="bold">c, Root face = <tspan fill="#dc2626">{rootFace} mm</tspan></text>
                            </>
                        ) : (
                            <text x="90" y="14" fontSize="10" fill="#334155" fontWeight="bold">c, Root face = <tspan fill="#dc2626">{rootFace} mm</tspan></text>
                        )}
                    </g>
                )}
            </svg>
        </div>
    </div>
  );
};

// --- SYMBOL RENDERER ---
const SymbolRenderer = ({ data, inputs, isBoxSection, hasBacking, weldLength, isWeldAllAround, t1, t2 }) => {
    const s = data.symbol;
    const weldSizeText = calculateWeldSize(data, t1, t2, inputs.rootFace);
    const isDouble = s === 'X' || s === 'K';
    
    return (
       <svg viewBox="0 0 300 100" className="w-full h-full">
           {/* Reference Line */}
           <line x1="20" y1="50" x2="280" y2="50" stroke="black" strokeWidth="2"/>
           
           {/* Arrow Head */}
           <line x1="20" y1="50" x2="10" y2="80" stroke="black" strokeWidth="2"/>
           <polygon points="10,80 5,70 15,70" fill="black"/>
           
           {/* Identification Line (Dashed) */}
           <line x1="20" y1="60" x2="280" y2="60" stroke="black" strokeWidth="1" strokeDasharray="4"/>
           
           {/* Weld All Around Circle */}
           {isWeldAllAround && (
                <circle cx="20" cy="50" r="4" fill="none" stroke="black" strokeWidth="1.5" />
           )}

           {/* Symbol Group */}
           <g transform="translate(140, 50)">
               {s === 'V' && <path d="M0,0 L-10,-15 M0,0 L10,-15" stroke="black" strokeWidth="2" fill="none"/>}
               {/* Y-Weld Symbol: Like V but with vertical stem */}
               {s === 'Y' && (
                  <g>
                      <path d="M0,0 L0,-5" stroke="black" strokeWidth="2" fill="none"/>
                      <path d="M0,-5 L-10,-15" stroke="black" strokeWidth="2" fill="none"/>
                      <path d="M0,-5 L10,-15" stroke="black" strokeWidth="2" fill="none"/>
                  </g>
               )}
               {/* MODIFIED: Corrected HV symbol to |/ shape, same for HY */}
               {s === 'HV' && <path d="M0,0 L0,-15 M0,0 L10,-15" stroke="black" strokeWidth="2" fill="none"/>}
               {/* MODIFIED: Corrected HY symbol to |/ shape, with a vertical extension that DOES NOT cross the dotted line */}
               {s === 'HY' && (
                  <g>
                      <path d="M0,0 L0,-15" stroke="black" strokeWidth="2" fill="none"/>
                      <path d="M0,-5 L10,-15" stroke="black" strokeWidth="2" fill="none"/>
                      {/* Shortened to 5 to avoid touching the dotted line at y=10 */}
                      <path d="M0,0 L0,5" stroke="black" strokeWidth="2" fill="none"/>
                  </g>
               )}
               {s === 'F' && <path d="M0,0 L0,-15 L15,0" stroke="black" strokeWidth="2" fill="none"/>}
               
               {/* MODIFIED: X Weld (Double V) - Mirrored at dotted line */}
               {s === 'X' && (
                    <>
                        {/* Top V (Arrow Side) - on Solid Line (y=0) */}
                        <path d="M0,0 L-10,-15 M0,0 L10,-15" stroke="black" strokeWidth="2" fill="none"/>
                        {/* Bottom V (Other Side) - on Dotted Line (y=10) */}
                        <path d="M0,10 L-10,25 M0,10 L10,25" stroke="black" strokeWidth="2" fill="none"/>
                    </>
               )}

               {/* MODIFIED: K Weld (Double Bevel) - Mirrored at dotted line */}
               {s === 'K' && (
                    <>
                        {/* Top Bevel (Arrow Side) - on Solid Line (y=0) */}
                        <path d="M0,0 L0,-15 M0,0 L10,-15" stroke="black" strokeWidth="2" fill="none"/>
                        {/* Bottom Bevel (Other Side) - on Dotted Line (y=10) */}
                        <path d="M0,10 L0,25 M0,10 L10,25" stroke="black" strokeWidth="2" fill="none"/>
                    </>
               )}
               
               {/* MODIFIED: Backing Bar M symbol moved BELOW dotted line for HV and V only */}
               {(hasBacking || isBoxSection) && (s === 'HV' || s === 'V') && (
                   <g transform="translate(0, 20)"> {/* Positive Y moves down. Dotted line is at y=60 (relative +10 from center). +20 puts it safely below. */}
                       <rect x="-5" y="-5" width="10" height="10" stroke="black" fill="none" strokeWidth="1"/>
                       <text x="0" y="3" textAnchor="middle" fontSize="8" fontWeight="bold">M</text>
                   </g>
               )}
           </g>
           
           {/* Dimensions (a, z) - LEFT of symbol - Top Value */}
           <text x="115" y="45" textAnchor="end" className="text-xs font-medium font-mono">
             {weldSizeText}
           </text>

           {/* Dimensions (a, z) - LEFT of symbol - Bottom Value for Double Sided */}
           {isDouble && (
               <text x="115" y="80" textAnchor="end" className="text-xs font-medium font-mono">
                 {weldSizeText}
               </text>
           )}
           
           {/* Weld Length - RIGHT of symbol */}
           {weldLength && !isWeldAllAround && (
                <>
                    <text x="165" y="45" textAnchor="start" className="text-xs font-bold font-mono">
                        {weldLength}
                    </text>
                    {isDouble && (
                        <text x="165" y="80" textAnchor="start" className="text-xs font-bold font-mono">
                            {weldLength}
                        </text>
                    )}
                </>
           )}

           {/* Tail Text (TYP) */}
           {isBoxSection && (
               <g>
                   <polyline points="270,40 280,50 270,60" fill="none" stroke="black" strokeWidth="2" />
                   <text x="265" y="45" textAnchor="end" className="text-xs font-bold text-blue-600">TYP 4 Places</text>
               </g>
           )}
       </svg>
    )
};

// --- MAIN APP ---
export default function App() {
  const [jointType, setJointType] = useState('t_joint');
  const [weldIndex, setWeldIndex] = useState(0);
  const [cornerOption, setCornerOption] = useState('1'); 
  const [t1, setT1] = useState(10);
  const [t2, setT2] = useState(10);
  const [t3, setT3] = useState(10); 
  const [t4, setT4] = useState(10); 
  const [gap, setGap] = useState(2);
  const [angle, setAngle] = useState(50);
  const [rootFace, setRootFace] = useState(2);
  const [isBoxSection, setIsBoxSection] = useState(false);
  const [mode, setMode] = useState('basic'); 
  
  // NEW STATES
  const [weldLength, setWeldLength] = useState("");
  const [isWeldAllAround, setIsWeldAllAround] = useState(false);
  
  const [hasBacking, setHasBacking] = useState(false);
  const [activeField, setActiveField] = useState(null);
  const [report, setReport] = useState("");
  const [loadingReport, setLoadingReport] = useState(false);

  // --- NEW: WAREHOUSE PLATES CHECK ---
  // List of available thicknesses
  const availablePlates = [2, 3, 4, 5, 6, 8, 10, 12, 16, 20, 25, 30, 35, 40, 50, 60, 75, 115, 150, 200];
  const [invalidPlates, setInvalidPlates] = useState([]);
  
  // --- NEW STATE: SELECTED WELD CLASS ---
  const [selectedClass, setSelectedClass] = useState("CP A");
  const [showClassDetails, setShowClassDetails] = useState(false);

  const currentWeld = WELD_DATA[jointType].types[weldIndex] || WELD_DATA[jointType].types[0];
  const canHaveBox = WELD_DATA[jointType]?.hasBoxOption && currentWeld.allowBox;
  const isT3Enabled = WELD_DATA[jointType]?.hasT3 || isBoxSection;
  const isT4Enabled = isBoxSection;

  useEffect(() => { setWeldIndex(0); setReport(""); setIsBoxSection(false); }, [jointType]);
  useEffect(() => { if (!currentWeld.allowBox) setIsBoxSection(false); }, [currentWeld]);

  // COMBINED LOGIC FOR HASBACKING
  useEffect(() => {
      const maxThickness = Math.max(Number(t1), Number(t2), isT3Enabled ? Number(t3) : 0, isT4Enabled ? Number(t4) : 0);
      
      // Determine if weld type supports/requires backing bar logic
      // Exclude: Fillets, Double Sided (X, K), HY, Y, Lap, Corner
      const fig = currentWeld.fig_type;
      
      const noBackingTypes = [
          't_fillet', 
          't_hy', 
          't_k', 
          'butt_x', 
          'butt_y', 
          'lap', 
          'corner_fillet', 
          'corner_v' // User said "corner joint", implying the category
      ];

      if (noBackingTypes.includes(fig)) {
          setHasBacking(false);
      } else {
          setHasBacking(isBoxSection || maxThickness >= 16);
      }
      
  }, [t1, t2, t3, t4, isBoxSection, isT3Enabled, isT4Enabled, currentWeld]);

  // Check plate availability logic
  useEffect(() => {
      const invalid = [];
      const check = (val, name) => {
          if (!availablePlates.includes(Number(val))) invalid.push(`${name} (${val}mm)`);
      };
      
      check(t1, "t1");
      check(t2, "t2");
      if (isT3Enabled) check(t3, "t3");
      if (isT4Enabled) check(t4, "t4");

      setInvalidPlates(invalid);
  }, [t1, t2, t3, t4, isT3Enabled, isT4Enabled]);

  // RESET SELECTED CLASS ON WELD CHANGE
  useEffect(() => {
      if(currentWeld.availableClasses && currentWeld.availableClasses.length > 0) {
          setSelectedClass(currentWeld.availableClasses[0]);
      }
  }, [currentWeld]);


  // Auto-adjust angle based on thickness (>= 20mm -> 30deg, else 50deg)
  useEffect(() => {
      const maxThickness = Math.max(Number(t1), Number(t2), isT3Enabled ? Number(t3) : 0, isT4Enabled ? Number(t4) : 0);
      if (maxThickness >= 20) {
          setAngle(30);
      } else {
          setAngle(50);
      }
  }, [t1, t2, t3, t4, isT3Enabled, isT4Enabled]);

  // Auto-clear weldLength if isWeldAllAround is checked
  useEffect(() => {
      if (isWeldAllAround) {
          setWeldLength("");
      }
  }, [isWeldAllAround]);


  const generateReport = async () => {
      setLoadingReport(true);
      const prompt = `Write a short technical justification (EN 15085). Joint: ${currentWeld.name}. Plates: ${t1}mm/${t2}mm. Class: ${selectedClass}`;
      const result = await generateGeminiContent(prompt);
      setReport(result);
      setLoadingReport(false);
  }

  const currentClassDetails = CLASS_DETAILS[selectedClass] || {};

  return (
    <div className="min-h-screen bg-slate-100 font-sans text-slate-800 p-4">
      <div className="max-w-7xl mx-auto bg-white rounded-lg shadow-lg overflow-hidden relative">
        <div className="bg-slate-900 text-white p-4 flex items-center justify-between">
            <div>
                <h1 className="text-xl font-bold flex items-center gap-2"><Ruler className="text-yellow-400"/> LocoWeld Mate <span className="text-xs bg-yellow-500 text-black px-1 rounded">V6.9 Restored</span></h1>
                <div className="flex items-center gap-2 text-xs text-gray-400"><Box size={14} /> Enhanced for Heavy Structures</div>
            </div>
            <div className="flex items-center gap-2">
                <span className={`text-xs uppercase font-bold ${mode==='basic' ? 'text-white' : 'text-gray-500'}`}>Basic</span>
                <button onClick={() => setMode(mode === 'basic' ? 'advanced' : 'basic')} className={`w-12 h-6 rounded-full p-1 transition-colors duration-300 ${mode === 'advanced' ? 'bg-yellow-500' : 'bg-slate-700'}`}><div className={`w-4 h-4 rounded-full bg-white transform transition-transform duration-300 ${mode === 'advanced' ? 'translate-x-6' : 'translate-x-0'}`} /></button>
                <span className={`text-xs uppercase font-bold ${mode==='advanced' ? 'text-yellow-400' : 'text-gray-500'}`}>Advanced</span>
            </div>
        </div>

        <div className="grid grid-cols-1 md:grid-cols-12 gap-0">
            <div className="md:col-span-5 p-5 bg-slate-50 border-r border-slate-200">
                <div className="mb-4"><label className="block text-xs font-bold uppercase text-slate-500 mb-2">Joint Type</label><div className="flex flex-wrap gap-2">{Object.entries(WELD_DATA).map(([key, val]) => (<button key={key} onClick={() => setJointType(key)} className={`px-3 py-1.5 text-xs rounded border font-semibold ${jointType === key ? 'bg-blue-600 text-white border-blue-600' : 'bg-white text-slate-600 hover:bg-slate-200'}`}>{val.label}</button>))}</div></div>
                <div className="mb-4"><label className="block text-xs font-bold uppercase text-slate-500 mb-2">Weld Type</label><select className="w-full p-2 text-sm border rounded bg-white font-semibold" value={weldIndex} onChange={(e) => setWeldIndex(Number(e.target.value))}>{WELD_DATA[jointType].types.map((t, idx) => (<option key={t.id} value={idx}>{t.symbol} - {t.name}</option>))}</select></div>
                {WELD_DATA[jointType].hasOptions && !isBoxSection && (<div className="mb-4 bg-yellow-50 p-2 rounded border border-yellow-200"><label className="block text-[10px] font-bold uppercase text-yellow-800 mb-1">Corner Config</label><div className="flex gap-2"><button onClick={()=>setCornerOption('1')} className={`flex-1 text-xs border rounded py-1 ${cornerOption==='1' ? 'bg-yellow-400 font-bold' : 'bg-white'}`}>Corner to Corner</button><button onClick={()=>setCornerOption('2')} className={`flex-1 text-xs border rounded py-1 ${cornerOption==='2' ? 'bg-yellow-400 font-bold' : 'bg-white'}`}>Overlap</button></div></div>)}
                {canHaveBox && (<div className="mb-4 bg-blue-50 p-3 rounded border border-blue-200"><label className="flex items-center gap-2 cursor-pointer"><input type="checkbox" checked={isBoxSection} onChange={(e) => setIsBoxSection(e.target.checked)} className="w-4 h-4 text-blue-600 rounded"/><span className="text-xs font-bold text-blue-800 flex items-center gap-2"><Layers size={14}/> Box Section Assembly</span></label></div>)}
                <div className="mb-4"><label className="block text-xs font-bold uppercase text-slate-500 mb-1">Input Guide</label><JointVisualizer jointType={jointType} weldTypeData={currentWeld} t1={t1} t2={t2} t3={t3} t4={t4} cornerOption={cornerOption} activeField={activeField} isBoxSection={isBoxSection}/></div>
                <div className="grid grid-cols-2 gap-3 mb-4">
                    <div><label className="text-[10px] font-bold text-slate-500">Plate t1 (mm)</label><input type="number" value={t1} onChange={e=>setT1(e.target.value)} onFocus={()=>setActiveField('t1')} onBlur={()=>setActiveField(null)} className="w-full p-1 border rounded text-sm font-mono"/></div>
                    <div><label className="text-[10px] font-bold text-slate-500">Plate t2 (mm)</label><input type="number" value={t2} onChange={e=>setT2(e.target.value)} onFocus={()=>setActiveField('t2')} onBlur={()=>setActiveField(null)} className="w-full p-1 border rounded text-sm font-mono"/></div>
                    {isT3Enabled && <div><label className="text-[10px] font-bold text-slate-500">Plate t3 (mm)</label><input type="number" value={t3} onChange={e=>setT3(e.target.value)} onFocus={()=>setActiveField('t3')} onBlur={()=>setActiveField(null)} className="w-full p-1 border rounded text-sm font-mono"/></div>}
                    {isT4Enabled && <div><label className="text-[10px] font-bold text-slate-500">Plate t4 (mm)</label><input type="number" value={t4} onChange={e=>setT4(e.target.value)} onFocus={()=>setActiveField('t4')} onBlur={()=>setActiveField(null)} className="w-full p-1 border rounded text-sm font-mono"/></div>}
                    
                    {/* NEW INPUT: Weld Length */}
                    <div>
                        <label className="text-[10px] font-bold text-slate-500">Weld Length (mm)</label>
                        <input type="number" placeholder="Optional" value={weldLength} onChange={(e) => setWeldLength(e.target.value)} className="w-full p-1 border rounded text-sm font-mono disabled:opacity-50" disabled={isWeldAllAround}/>
                    </div>
                    {/* NEW INPUT: Weld All Around */}
                    <div className="flex items-center pt-4">
                        <label className="flex items-center gap-2 cursor-pointer select-none group">
                            <div className={`w-4 h-4 border rounded flex items-center justify-center transition-colors ${isWeldAllAround ? 'bg-blue-600 border-blue-600' : 'bg-white border-slate-300'}`}>
                                {isWeldAllAround && <svg width="10" height="10" viewBox="0 0 24 24" fill="none" stroke="white" strokeWidth="3" strokeLinecap="round" strokeLinejoin="round"><path d="M21 12a9 9 0 1 1-9-9c2.52 0 4.93 1 6.74 2.74L21 8" /><path d="M21 3v5h-5" /></svg>}
                            </div>
                            <input type="checkbox" checked={isWeldAllAround} onChange={(e) => setIsWeldAllAround(e.target.checked)} className="hidden"/>
                            <span className="text-[10px] font-bold text-slate-500 group-hover:text-blue-600">Weld All Around</span>
                        </label>
                    </div>
                </div>
                {mode === 'advanced' && (<div className="mt-4 p-3 bg-yellow-50 border border-yellow-200 rounded animate-fade-in"><div className="grid grid-cols-3 gap-2"><div><label className="text-[10px] font-bold text-slate-500">Angle (°)</label><input type="number" value={angle} onChange={(e) => setAngle(Number(e.target.value))} className="w-full p-1 border rounded text-sm font-mono"/></div><div><label className="text-[10px] font-bold text-slate-500">Gap (mm)</label><input type="number" value={gap} onChange={(e) => setGap(Number(e.target.value))} className="w-full p-1 border rounded text-sm font-mono"/></div><div><label className="text-[10px] font-bold text-slate-500">Root (mm)</label><input type="number" value={rootFace} onChange={(e) => setRootFace(Number(e.target.value))} className="w-full p-1 border rounded text-sm font-mono"/></div></div></div>)}
            </div>

            <div className="md:col-span-7 p-5 bg-white">
                <h2 className="text-md font-bold text-slate-800 mb-3 flex items-center gap-2"><Info size={16} /> Technical Specification</h2>
                <div className={`p-3 mb-4 rounded-r border-l-4 ${hasBacking ? 'bg-amber-50 border-amber-500' : 'bg-blue-50 border-blue-500'}`}>
                    <div>
                        <div className="flex justify-between items-start">
                            <div>
                                <h3 className="font-bold text-sm text-slate-900">{isBoxSection ? "Box Girder (4x HV)" : currentWeld.name}</h3>
                                <p className="text-xs text-slate-600 mt-1">{currentWeld.description}</p>
                            </div>
                            {/* WELD CLASS SELECTOR */}
                             <div className="flex flex-col items-end">
                                <span className="text-[10px] font-bold text-slate-400 uppercase mb-1">Performance Class</span>
                                <div className="flex gap-1">
                                    {currentWeld.availableClasses?.map((cls) => (
                                        <button 
                                            key={cls}
                                            onClick={() => { setSelectedClass(cls); setShowClassDetails(true); }}
                                            className={`text-[10px] px-2 py-1 rounded border ${selectedClass === cls ? 'bg-blue-600 text-white border-blue-600' : 'bg-white text-slate-600 border-slate-200 hover:bg-slate-100'}`}
                                        >
                                            {cls}
                                        </button>
                                    ))}
                                </div>
                            </div>
                        </div>

                        {/* CLASS DETAILS PANEL */}
                        {showClassDetails && (
                            <div className="mt-3 bg-white/50 rounded-lg border border-blue-100 p-2 text-xs">
                                <div className="grid grid-cols-2 md:grid-cols-4 gap-2 mb-2">
                                    <div className="bg-blue-50 p-1.5 rounded">
                                        <span className="block text-[9px] text-blue-400 font-bold uppercase">Stress</span>
                                        <div className="font-bold text-blue-900 flex items-center gap-1"><Activity size={10}/> {currentClassDetails.stress}</div>
                                    </div>
                                    <div className="bg-green-50 p-1.5 rounded">
                                        <span className="block text-[9px] text-green-400 font-bold uppercase">Safety</span>
                                        <div className="font-bold text-green-900 flex items-center gap-1"><Shield size={10}/> {currentClassDetails.safety}</div>
                                    </div>
                                    <div className="bg-purple-50 p-1.5 rounded">
                                        <span className="block text-[9px] text-purple-400 font-bold uppercase">Volumetric</span>
                                        <div className="font-bold text-purple-900 flex items-center gap-1"><Eye size={10}/> {currentClassDetails.volumetric}</div>
                                    </div>
                                    <div className="bg-orange-50 p-1.5 rounded">
                                        <span className="block text-[9px] text-orange-400 font-bold uppercase">Surface</span>
                                        <div className="font-bold text-orange-900 flex items-center gap-1"><RefreshCw size={10}/> {currentClassDetails.surface}</div>
                                    </div>
                                </div>
                                <div className="text-[10px] text-slate-500 italic border-t border-blue-100 pt-1 mt-1">
                                    Inspect: {currentClassDetails.inspection} | {currentClassDetails.desc}
                                </div>
                            </div>
                        )}

                        <div className="flex flex-col gap-2 mt-3">
                           <div className="flex flex-wrap gap-2">
                                <span className="inline-flex items-center gap-2 text-xs md:text-sm bg-indigo-100 text-indigo-900 px-3 py-1.5 rounded-md font-bold border border-indigo-200 w-fit shadow-sm"><ArrowDownToLine size={16}/> {isBoxSection ? 'Full Penetration' : currentWeld.penetration}</span>
                                {/* ACCESSIBILITY BADGE */}
                                <span className={`inline-flex items-center gap-2 text-xs md:text-sm px-3 py-1.5 rounded-md font-bold border w-fit shadow-sm ${["CP A", "CP B1"].includes(selectedClass) ? "bg-emerald-100 text-emerald-900 border-emerald-200" : "bg-gray-100 text-gray-600 border-gray-200"}`}>
                                    <ScanEye size={16}/> 
                                    {["CP A", "CP B1"].includes(selectedClass) ? "Full Accessibility Required for Inspection in production & maintenance" : "Full Accessibility Not Req."}
                                </span>
                           </div>
                           
                           {/* CP B2 / CP C1 Special Note */}
                           {["CP B2", "CP C1"].includes(selectedClass) && (
                                <div className="mt-2 flex items-start gap-1 text-[10px] font-bold text-blue-800 bg-blue-100 px-2 py-1 rounded">
                                    <Info size={12} className="mt-0.5 flex-shrink-0"/> 
                                    <span>Applicable for welds where volumetric NDT is not possible. Increased surface testing is required.</span>
                                </div>
                           )}
                        </div>
                        {hasBacking && (<div className="mt-3 flex items-center gap-1 text-[10px] font-bold text-amber-800 bg-amber-100 px-2 py-1 rounded inline-block"><AlertCircle size={12}/> Backing Bar Required (Thickness ≥ 16mm or Box)</div>)}
                        
                        {/* WARNING: INVALID PLATE THICKNESS */}
                        {invalidPlates.length > 0 && (
                            <div className="mt-2 flex items-center gap-1 text-[10px] font-bold text-red-800 bg-red-100 px-2 py-1 rounded inline-block">
                                <TriangleAlert size={12}/> Warning: Plate thickness ({invalidPlates.join(', ')}mm) not available in house.
                            </div>
                        )}
                    </div>
                </div>
                <div className="border rounded bg-white p-3 shadow-sm mb-4">
                     <h4 className="text-xs font-bold text-gray-500 mb-2 border-b">Fig 1: Cross Section</h4>
                     <div className="h-64 flex items-center justify-center bg-gray-50"><ResultRenderer data={currentWeld} inputs={{ angle, gap, rootFace, jointType, cornerOption, t3, t4 }} isBoxSection={isBoxSection} hasBacking={hasBacking}/></div>
                </div>
                <div className="border rounded bg-white p-3 shadow-sm mb-4">
                     <h4 className="text-xs font-bold text-gray-500 mb-2 border-b">Fig 2: Symbol</h4>
                     <div className="h-32 flex items-center justify-center bg-gray-50"><SymbolRenderer data={currentWeld} inputs={{ angle, gap, rootFace, jointType, cornerOption, t3, t4 }} isBoxSection={isBoxSection} hasBacking={hasBacking} weldLength={weldLength} isWeldAllAround={isWeldAllAround} t1={t1} t2={t2}/></div>
                </div>
                <div className="mt-6 border-t pt-4"><div className="flex justify-between items-center mb-2"><h3 className="text-sm font-bold flex items-center gap-1 text-slate-700"><Sparkles size={14} className="text-indigo-500"/> AI Technical Justification</h3><button onClick={generateReport} disabled={loadingReport} className="text-xs bg-indigo-50 text-indigo-600 px-3 py-1 rounded border border-indigo-200 hover:bg-indigo-100 flex items-center gap-1 disabled:opacity-50">{loadingReport ? 'Generating...' : '✨ Generate Report'}</button></div>{report && (<div className="bg-slate-50 p-3 rounded text-xs text-slate-600 italic border border-slate-200 leading-relaxed animate-fade-in">"{report}"</div>)}</div>
                <div className="mt-4 text-[10px] text-gray-400 pt-2 border-t">*Ref: EN 15085-3 / ISO 9692. Always verify with approved WPS.</div>
            </div>
        </div>
        <AIChatConsultant />
      </div>
    </div>
  )
}
