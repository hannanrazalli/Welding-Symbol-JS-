import React, { useState, useEffect, useMemo } from 'react';

// --- INLINE ICONS (Fixed: Accepting props for size & className) ---
const createIcon = (children) => ({ size = 16, className = "", ...props }) => (
  <svg 
    xmlns="http://www.w3.org/2000/svg" 
    width={size} 
    height={size} 
    viewBox="0 0 24 24" 
    fill="none" 
    stroke="currentColor" 
    strokeWidth="2" 
    strokeLinecap="round" 
    strokeLinejoin="round" 
    className={className} 
    {...props}
  >
    {children}
  </svg>
);

const IconInfo = createIcon(<><circle cx="12" cy="12" r="10"/><path d="M12 16v-4"/><path d="M12 8h.01"/></>);
const IconRuler = createIcon(<><path d="M2 12h20"/><path d="M6 12v-2"/><path d="M10 12v-3"/><path d="M14 12v-3"/><path d="M18 12v-2"/></>);
const IconAlertCircle = createIcon(<><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="12" y1="16" x2="12.01" y2="16"/></>);
const IconBox = createIcon(<><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/></>);
const IconLayers = createIcon(<><polygon points="12 2 2 7 12 12 22 7 12 2"/><polyline points="2 17 12 22 22 17"/><polyline points="2 12 12 17 22 12"/></>);
const IconArrowDownToLine = createIcon(<><path d="M12 17V3"/><path d="m6 11 6 6 6-6"/><path d="M19 21H5"/></>);
const IconTriangleAlert = createIcon(<><path d="m21.73 18-8-14a2 2 0 0 0-3.48 0l-8 14A2 2 0 0 0 4 21h16a2 2 0 0 0 1.73-3Z"/><line x1="12" y1="9" x2="12" y2="13"/><line x1="12" y1="17" x2="12.01" y2="17"/></>);
const IconActivity = createIcon(<polyline points="22 12 18 12 15 21 9 3 6 12 2 12"/>);
const IconShield = createIcon(<path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/>);
const IconEye = createIcon(<><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/><circle cx="12" cy="12" r="3"/></>);
const IconRefreshCw = createIcon(<><path d="M21 12a9 9 0 0 0-9-9 9.75 9.75 0 0 0-6.74 2.74L3 8"/><path d="M3 3v5h5"/><path d="M3 12a9 9 0 0 0 9 9 9.75 9.75 0 0 0 6.74-2.74L21 16"/><path d="M16 16h5v5"/></>);
const IconScanEye = createIcon(<><path d="M3 7V5a2 2 0 0 1 2-2h2"/><path d="M17 3h2a2 2 0 0 1 2 2v2"/><path d="M21 17v2a2 2 0 0 1-2 2h-2"/><path d="M7 21H5a2 2 0 0 1-2-2v-2"/><circle cx="12" cy="12" r="3"/><path d="M12 16a9 9 0 0 0-9-7"/><path d="M12 8a9 9 0 0 1 9 4"/></>);
const IconEyeOff = createIcon(<><path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-1.07a3 3 0 1 1-4.24-4.24"/><line x1="1" y1="1" x2="23" y2="23"/></>);
const IconHash = createIcon(<><line x1="4" y1="9" x2="20" y2="9"/><line x1="4" y1="15" x2="20" y2="15"/><line x1="10" y1="3" x2="8" y2="21"/><line x1="16" y1="3" x2="14" y2="21"/></>);
const IconFilter = createIcon(<polygon points="22 3 2 3 10 12.46 10 19 14 21 14 12.46 22 3"/>);
const IconTrain = createIcon(<path d="M15 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V4a2 2 0 0 0-2-2ZM10 21h4 M6 18h12 M6 4h12" />);

// --- WAREHOUSE DATA ---
const AVAILABLE_PLATES = [2, 3, 4, 5, 6, 8, 10, 12, 16, 20, 25, 30, 35, 40, 50, 60, 75, 115, 150, 200];
 
// --- EN 15085-3 CLASS DETAILS (Table 3) ---
// Updated with validPairs AND specific stressVal for each pair
const CLASS_DETAILS = {
    "CP A": {
        inspection: "CT 1",
        volumetric: "100%",
        surface: "100%",
        visual: "100%",
        desc: "Highest safety requirements. Full testing.",
        validPairs: [{ stress: "High", safety: "High", stressVal: "S ≥ 0.9" }]
    },
    "CP B1": {
        inspection: "CT 2",
        volumetric: "10%",
        surface: "10%",
        visual: "100%",
        desc: "High safety, medium stress.",
        validPairs: [{ stress: "Medium", safety: "High", stressVal: "0.75 ≤ S ≤ 0.9" }]
    },
    "CP B2": {
        inspection: "CT 2",
        volumetric: "10%",
        surface: "10%",
        visual: "100%",
        desc: "High stress, medium safety.",
        validPairs: [{ stress: "High", safety: "Medium", stressVal: "S ≥ 0.9" }]
    },
    "CP C1": {
        inspection: "CT 2",
        volumetric: "10%",
        surface: "10%",
        visual: "100%",
        desc: "High safety, low stress.",
        validPairs: [{ stress: "Low", safety: "High", stressVal: "S < 0.75" }]
    },
    "CP C2": {
        inspection: "CT 3",
        volumetric: "Not required",
        surface: "Not required",
        visual: "100%",
        desc: "General usage. Visual check only.",
        validPairs: [
            { stress: "High", safety: "Low", stressVal: "S ≥ 0.9" },
            { stress: "Medium", safety: "Medium", stressVal: "0.75 ≤ S ≤ 0.9" }
        ]
    },
    "CP C3": {
        inspection: "CT 4",
        volumetric: "Not required",
        surface: "Not required",
        visual: "100%",
        desc: "Low safety requirements.",
        validPairs: [
            { stress: "Medium", safety: "Low", stressVal: "0.75 ≤ S ≤ 0.9" },
            { stress: "Low", safety: "Medium", stressVal: "S < 0.75" }
        ]
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
        id: '3a_butt', 
        name: 'HV Weld (Single Bevel)',
        symbol: 'HV',
        min_t: 3,
        max_t: 15,
        availableClasses: ["CP A", "CP B1", "CP B2", "CP C1", "CP C2"],
        penetration: "Full Penetration",
        description: "Single Bevel preparation.",
        fig_type: "butt_hv" 
      },
      {
        id: '2b',
        name: 'Y-Weld (Single V with Root)',
        symbol: 'Y',
        min_t: 5,
        max_t: 20,
        availableClasses: ["CP C2", "CP C3"], 
        penetration: "Partial Penetration", 
        description: "Single V with root face, no gap.",
        fig_type: "butt_y"
      },
      {
        id: '11a_butt', 
        name: 'HY Weld',
        symbol: 'HY',
        min_t: 3,
        max_t: 15,
        availableClasses: ["CP C2", "CP C3"],
        penetration: "Partial Penetration",
        description: "Single bevel with large root face.",
        fig_type: "butt_hy" 
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
      },
      {
        id: '7_butt', 
        name: 'K-Weld (Double Bevel)',
        symbol: 'K',
        min_t: 12,
        max_t: 100,
        availableClasses: ["CP A", "CP B1", "CP B2", "CP C1", "CP C2"],
        penetration: "Full Penetration",
        description: "Double bevel preparation.",
        fig_type: "butt_k" 
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
        availableClasses: ["CP C2", "CP C3"], 
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
        availableClasses: ["CP C2", "CP C3"], 
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
        availableClasses: ["CP C1", "CP C2", "CP C3"],
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
        availableClasses: ["CP C2"], 
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
        if (t_min <= 17) {
            return `a${Math.ceil(0.7 * t_min)}`;
        } else if (t_min >= 18 && t_min <= 19) {
            return `a12`;
        } else {
            const angleInRadians = 30 * (Math.PI / 180);
            const calculatedValue = Math.tan(angleInRadians) * (t_min - 2);
            return `a${Math.ceil(calculatedValue)}`;
        }
    } else if (weldType.symbol === 'HY' || weldType.symbol === 'Y') {
        return `${Number(t1) - Number(rootFace)}`;
    } else if (['HV', 'V', 'K', 'X', 'I'].includes(weldType.symbol)) {
        return `${t1}`; 
    }
    return "";
};
 
// --- SUB-COMPONENTS ---
 
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
                <path d="M25,18 L35,18 L35,122 L25,122 L15,112 L15,28 Z" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                <line x1="15" y1="25" x2="25" y2="15" stroke="red" strokeDasharray="2,2"/>
                <line x1="15" y1="115" x2="25" y2="125" stroke="red" strokeDasharray="2,2"/>
                <path d="M125,18 L135,18 L145,28 L145,112 L135,122 L125,122 Z" fill={plate3Fill} stroke={getStroke('t3')} strokeWidth={getWidth('t3')} />
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
                    {fig === 'butt_sq' ? (
                        // Square Edge (Modified V with no root face)
                        <path d="M20,60 L80,60 L95,90 L20,90 Z" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    ) : fig === 'butt_half_sq' ? (
                        // Half Square (Modified HV with no root face) - Square side
                        <rect x="20" y="60" width="75" height="30" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    ) : fig === 'butt_v' ? (
                        <path d="M20,60 L80,60 L95,80 L95,90 L20,90 Z" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    ) : fig === 'butt_hv' ? (
                        // HV: Single Bevel - Left plate square (ends at 95), Right plate beveled
                         <rect x="20" y="60" width="75" height="30" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    ) : fig === 'butt_y' ? (
                        // Y-Weld: Similar to V but touching at bottom. Beveled plates.
                        <path d="M20,60 L80,60 L100,80 L100,90 L20,90 Z" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    ) : fig === 'butt_hy' ? (
                        // HY: Modified to look like HV but Rapat (Gap=0)
                        // Left Plate ends at 100
                         <rect x="20" y="60" width="80" height="30" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    ) : fig === 'butt_x' ? (
                        <path d="M20,60 L80,60 L95,75 L80,90 L20,90 Z" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    ) : fig === 'butt_k' ? (
                        // K-Weld Butt: Left square, Right Double Bevel.
                        // Left plate Square
                        <rect x="20" y="60" width="75" height="30" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    ) : (
                        <rect x="20" y="60" width="75" height="30" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    )}
 
                    {/* RIGHT PLATE */}
                    {fig === 'butt_sq' ? (
                        // Square Edge (Modified V with no root face)
                        <path d="M180,60 L120,60 L105,90 L180,90 Z" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                    ) : fig === 'butt_half_sq' ? (
                        // Half Square (Modified HV with no root face) - Beveled side
                        <path d="M180,60 L120,60 L105,90 L180,90 Z" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                    ) : fig === 'butt_v' ? (
                        <path d="M180,60 L120,60 L105,80 L105,90 L180,90 Z" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                    ) : fig === 'butt_hv' ? (
                         // HV: Right plate beveled. Matching V-weld right plate
                        <path d="M180,60 L120,60 L105,80 L105,90 L180,90 Z" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                    ) : fig === 'butt_y' ? (
                        <path d="M180,60 L120,60 L100,80 L100,90 L180,90 Z" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                    ) : fig === 'butt_hy' ? (
                         // HY: Beveled, Rapat with Left (starts at 100). 
                         // Maintain HV slope (15/20). Root at 100. Top at 115.
                         <path d="M180,60 L115,60 L100,80 L100,90 L180,90 Z" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                    ) : fig === 'butt_x' ? (
                        <path d="M180,60 L120,60 L105,75 L120,90 L180,90 Z" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                    ) : fig === 'butt_k' ? (
                        // K-Weld Butt: Right plate Double Bevel (K shape) WITH ROOT FACE.
                        // Top bevel: 120,60 to 105,72. Root Face: 105,72 to 105,78 (height 6). Bottom bevel: 105,78 to 120,90.
                        <path d="M180,60 L120,60 L105,72 L105,78 L120,90 L180,90 Z" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                    ) : (
                        <rect x="105" y="60" width="75" height="30" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                    )}
 
                    <text x="50" y="45" className="text-[10px]" fill={getStroke('t1')}>t1</text>
                    <text x="140" y="45" className="text-[10px]" fill={getStroke('t2')}>t2</text>
                    
                    {/* Dotted lines for V & Square Edge */}
                    {(fig === 'butt_v' || fig === 'butt_sq') && (
                        <>
                            <line x1="80" y1="60" x2="95" y2={fig === 'butt_sq' ? 90 : 80} stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
                            <line x1="120" y1="60" x2="105" y2={fig === 'butt_sq' ? 90 : 80} stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
                        </>
                    )}
 
                    {/* Dotted lines for HV & Half Square */}
                    {(fig === 'butt_hv' || fig === 'butt_half_sq') && (
                        <line x1="120" y1="60" x2="105" y2={fig === 'butt_half_sq' ? 90 : 80} stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
                    )}
 
                    {/* Dotted lines for Y */}
                    {fig === 'butt_y' && (
                        <>
                            <line x1="80" y1="60" x2="100" y2="80" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
                            <line x1="120" y1="60" x2="100" y2="80" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
                        </>
                    )}
 
                     {/* Dotted lines for HY - Modified to match HV slope but RAPAT */}
                     {fig === 'butt_hy' && <line x1="115" y1="60" x2="100" y2="80" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>}
 
                    {/* Dotted lines for X (Double V) */}
                    {fig === 'butt_x' && (
                        <>
                            <line x1="80" y1="60" x2="95" y2="75" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
                            <line x1="80" y1="90" x2="95" y2="75" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
                            <line x1="120" y1="60" x2="105" y2="75" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
                            <line x1="120" y1="90" x2="105" y2="75" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
                        </>
                    )}
 
                    {/* Dotted lines for K (Butt) - UPDATED FOR ROOT FACE */}
                    {fig === 'butt_k' && (
                        <>
                            {/* Top bevel dotted line */}
                            <line x1="120" y1="60" x2="105" y2="72" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
                            {/* Bottom bevel dotted line */}
                            <line x1="120" y1="90" x2="105" y2="78" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>
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
                    ) : fig.includes('hv') ? (
                         // HV: Single Bevel with Gap (y=95)
                         <path d="M90,20 L110,20 L110,95 L100,95 L90,80 Z" fill={plate2Fill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    ) : fig.includes('hy') ? (
                         // HY: Single Bevel NO Gap (y=100) - Touching plate
                         // Ensure web sits on flange at y=100
                         <path d="M90,20 L110,20 L110,100 L100,100 L90,80 L90,20 Z" fill={plate2Fill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    ) : (
                         // No Bevel (Square)
                         <rect x="90" y="20" width="20" height="80" fill={plate2Fill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                    )}
 
                    <text x="175" y="115" className="text-[10px]" fill={getStroke('t2')}>t2</text>
                    <text x="60" y="50" className="text-[10px]" textAnchor="end" fill={getStroke('t1')}>t1</text>
                    
                    {/* Red Dashed Lines indicating Weld Prep */}
                    {(fig === 't_hv') && <line x1="90" y1="80" x2="100" y2="95" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>}
                    {(fig === 't_hy') && <line x1="90" y1="80" x2="100" y2="100" stroke="red" strokeWidth="1" strokeDasharray="2,2" opacity="0.7"/>}
                    
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
                           {/* Option 1: Corner to Corner */}
                           <rect x="0" y="20" width="20" height="60" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                           <rect x="20" y="0" width="60" height="20" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                           <circle cx="20" cy="20" r="2" fill="red" opacity="0.5" />
                           <text x="-15" y="50" className="text-[10px]" textAnchor="end" fill={getStroke('t1')}>t1</text>
                           <text x="50" y="-10" className="text-[10px]" fill={getStroke('t2')}>t2</text>
                        </g>
                     ) : (
                        <g>
                           {/* Option 2: Overlap (Modified: 3/10 overlap) */}
                           <rect x="0" y="20" width="20" height="60" fill={plateFill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                           {/* Plate 2 shifted right to x=14 (overlaps 6px out of 20px width of P1) */}
                           <rect x="14" y="0" width="80" height="20" fill={plate2Fill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                           <text x="-15" y="50" className="text-[10px]" textAnchor="end" fill={getStroke('t1')}>t1</text>
                           <text x="60" y="-10" className="text-[10px]" fill={getStroke('t2')}>t2</text>
                        </g>
                     )}
                  </g>
                )}
                {jointType === 'lap' && (
                    <g transform="translate(90, 80)">
                        {/* Bottom Plate: Width 130 */}
                        <rect x="0" y="20" width="130" height="20" fill={plateFill} stroke={getStroke('t2')} strokeWidth={getWidth('t2')} />
                        {/* Top Plate: Width 100, Starts 30 */}
                        <rect x="30" y="0" width="100" height="20" fill={plate2Fill} stroke={getStroke('t1')} strokeWidth={getWidth('t1')} />
                        <text x="-10" y="35" className="text-[10px]" textAnchor="end" fill={getStroke('t2')}>t2</text>
                        <text x="140" y="15" className="text-[10px]" fill={getStroke('t1')}>t1</text>
                    </g>
                )}
                {jointType === 'three_member' && (
                    <g transform="translate(80, 60)">
                        <path d="M0,0 L70,0 L78,15 L0,15 Z" fill={plateFill} stroke="black" />
                        <path d="M82,15 L90,0 L160,0 L160,15 Z" fill={plate2Fill} stroke="black" />
                        {/* FIX: Updated t3 rect to use getStroke('t3') and getWidth('t3') and plate3Fill */}
                        <rect x="72" y="15" width="16" height="60" fill={plate3Fill} stroke="black" />
                        <line x1="70" y1="0" x2="78" y2="15" stroke="red" strokeWidth="1" strokeDasharray="1,1" opacity="0.6"/>
                        <line x1="90" y1="0" x2="82" y2="15" stroke="red" strokeWidth="1" strokeDasharray="1,1" opacity="0.6"/>
                        <text x="40" y="-5" className="text-[10px]" fill="black">t1</text>
                        <text x="120" y="-5" className="text-[10px]" fill="black">t2</text>
                        <text x="95" y="50" className="text-[10px]" fill="black">t3</text>
                    </g>
                )}
            </>
        )}
      </svg>
    </div>
  );
};
 
// --- NEW: PARAMETER DIAGRAM COMPONENT ---
const ParameterDiagram = ({ jointType, fig }) => {
    // Show only if not a fillet/lap where params don't apply visually in the same way
    if (['t_fillet', 'corner_fillet', 'lap'].includes(fig)) {
        return (
            <div className="flex flex-col h-full bg-slate-50 items-center justify-center border-l border-slate-200">
                <div className="text-xs text-gray-400 text-center p-4">
                    <IconInfo size={24} className="mx-auto mb-2 opacity-50"/>
                    Parameters (Gap, Root, Angle)<br/>not typically defined for Fillet welds.
                </div>
            </div>
        );
    }
 
    const isT = jointType === 't_joint' || jointType === 'corner';
 
    return (
        <div className="flex flex-col h-full border-l border-slate-200">
            <div className="bg-slate-50 px-3 py-2 border-b text-xs font-bold text-gray-500 uppercase tracking-wider">Param Definition</div>
            <div className="flex-1 relative flex items-center justify-center bg-white">
                <svg viewBox="0 0 200 150" className="w-full h-full">
                    <defs>
                        <marker id="arrowRed" markerWidth="6" markerHeight="6" refX="3" refY="3" orient="auto" markerUnits="strokeWidth">
                            <path d="M0,0 L0,6 L6,3 z" fill="#dc2626" />
                        </marker>
                    </defs>
 
                    {isT ? (
                        // T-JOINT SCHEMATIC
                        <g transform="translate(50, 20)">
                            {/* Horizontal Plate */}
                            <rect x="0" y="80" width="100" height="15" fill="#e2e8f0" stroke="#334155" />
                            {/* Vertical Plate with Single Bevel */}
                            <path d="M40,0 L60,0 L60,70 L50,80 L40,80 Z" fill="#e2e8f0" stroke="#334155" />
                            
                            {/* Gap (b) */}
                            <line x1="50" y1="80" x2="65" y2="80" stroke="#94a3b8" strokeDasharray="2" />
                            <line x1="50" y1="95" x2="65" y2="95" stroke="#94a3b8" strokeDasharray="2" />
                            <line x1="70" y1="80" x2="70" y2="95" stroke="#dc2626" markerStart="url(#arrowRed)" markerEnd="url(#arrowRed)" />
                            <text x="75" y="90" fontSize="10" fill="#dc2626" fontWeight="bold">b (Gap)</text>
 
                            {/* Root Face (c) */}
                            <line x1="60" y1="70" x2="80" y2="70" stroke="#94a3b8" strokeDasharray="2" />
                            <line x1="50" y1="80" x2="30" y2="80" stroke="#94a3b8" strokeDasharray="2" /> {/* Ref line for bottom of web */}
                            <line x1="60" y1="70" x2="60" y2="80" stroke="transparent" /> {/* Invisible line for measurement */}
                            
                            {/* Angle (alpha) */}
                            <path d="M60,0 L60,50" stroke="#94a3b8" strokeDasharray="2"/>
                            <path d="M60,40 Q55,40 54,30" fill="none" stroke="#dc2626" /> 
                            {/* Simplified Arc */}
                            <text x="35" y="35" fontSize="10" fill="#dc2626" fontWeight="bold">α</text>
 
                            {/* Root Face pointer */}
                            <line x1="60" y1="75" x2="85" y2="75" stroke="#dc2626" strokeWidth="1"/>
                            <text x="90" y="78" fontSize="10" fill="#dc2626" fontWeight="bold">c (Root)</text>
                        </g>
                    ) : (
                        // BUTT JOINT SCHEMATIC (V-Prep)
                        <g transform="translate(10, 30)">
                            {/* Left Plate */}
                            <path d="M10,20 L80,20 L90,60 L90,90 L80,90 L10,90 Z" fill="#e2e8f0" stroke="#334155" />
                            {/* Right Plate */}
                            <path d="M170,20 L100,20 L90,60 L90,90 L100,90 L170,90 Z" fill="#e2e8f0" stroke="#334155" transform="translate(10,0)"/>
                            
                            {/* Root Face (c) */}
                            <line x1="90" y1="60" x2="70" y2="60" stroke="#94a3b8" strokeDasharray="2"/>
                            <line x1="90" y1="90" x2="70" y2="90" stroke="#94a3b8" strokeDasharray="2"/>
                            <line x1="75" y1="60" x2="75" y2="90" stroke="#dc2626" markerStart="url(#arrowRed)" markerEnd="url(#arrowRed)"/>
                            <text x="65" y="80" textAnchor="end" fontSize="10" fill="#dc2626" fontWeight="bold">c (Root)</text>
 
                            {/* Gap (b) */}
                            {/* Using transform translate 10 on right plate means gap is 10 */}
                            <line x1="90" y1="90" x2="90" y2="110" stroke="#94a3b8" strokeDasharray="2"/>
                            <line x1="100" y1="90" x2="100" y2="110" stroke="#94a3b8" strokeDasharray="2"/>
                            <line x1="90" y1="105" x2="100" y2="105" stroke="#dc2626" markerStart="url(#arrowRed)" markerEnd="url(#arrowRed)"/>
                            <text x="95" y="120" textAnchor="middle" fontSize="10" fill="#dc2626" fontWeight="bold">b (Gap)</text>
 
                            {/* Angle (alpha) */}
                            <path d="M90,60 L90,20" stroke="#94a3b8" strokeDasharray="2"/>
                            <path d="M90,40 Q85,40 82,30" fill="none" stroke="#dc2626"/>
                            <text x="80" y="25" textAnchor="end" fontSize="10" fill="#dc2626" fontWeight="bold">α</text>
                        </g>
                    )}
                </svg>
            </div>
        </div>
    );
}
 
// --- RESULT RENDERER (STABLE - NO DIMS) ---
const ResultRenderer = ({ data, inputs, isBoxSection, hasBacking, hasSealingRun, hasAdditionalFillet, weldSide, selectedClass }) => {
  const { jointType, cornerOption, angle, gap, rootFace } = inputs;
  const fig = data?.fig_type || "unknown";
  const s = data.symbol;
 
  const plateFill = "url(#hatch)";
  const plate2Fill = "url(#hatch)";
  const plate3Fill = "url(#hatch)";
  const plate4Fill = "url(#hatch)";
  const weldFill = "#fca5a5";
  const backingColor = "#475569";
 
  const showLegend = !['t_fillet', 'corner_fillet', 'lap'].includes(fig);
  const showGap = !fig.includes('hy') && fig !== 'butt_y'; 
  
  // Determine if specific legends should be shown based on weld type (s) and jointType
  // For Corner Joint, never show these legends as per instructions.
  const isCorner = jointType === 'corner';
  const showAngle = !isCorner && ['V', 'HV', 'Y', 'HY', 'X', 'K'].includes(s);
  const showRoot = !isCorner && ['V', 'HV', 'Y', 'HY', 'X', 'K'].includes(s);
  const showGapVal = !isCorner && !['F', 'Y', 'HY'].includes(s); 
 
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
                                {(fig === 'butt_sq' || fig === 'butt_half_sq') ? (
                                    <>
                                        {/* Square Edge (Modified V with no root face) */}
                                        <path d="M20,60 L80,60 L95,90 L20,90 Z" fill={plateFill} stroke="black" />
                                        <path d="M180,60 L120,60 L105,90 L180,90 Z" fill={plateFill} stroke="black" />
                                        <path d="M80,60 L120,60 L105,90 L95,90 Z" fill={weldFill} stroke="black" />
                                    </>
                                ) : (
                                    <>
                                        {fig === 'butt_v' ? <path d="M20,60 L80,60 L95,80 L95,90 L20,90 Z" fill={plateFill} stroke="black" /> : <rect x="20" y="60" width="75" height="30" fill={plateFill} stroke="black" />}
                                        {fig === 'butt_v' ? <path d="M180,60 L120,60 L105,80 L105,90 L180,90 Z" fill={plateFill} stroke="black" /> : <rect x="105" y="60" width="75" height="30" fill={plateFill} stroke="black" />}
                                        {fig === 'butt_v' && <path d="M80,60 L120,60 L105,80 L105,90 L95,90 L95,80 Z" fill={weldFill} stroke="black" />}
                                        
                                        {fig === 'butt_hv' ? <rect x="20" y="60" width="75" height="30" fill={plateFill} stroke="black" /> : null}
                                        {fig === 'butt_hv' ? <path d="M180,60 L105,60 L95,90 L180,90 Z" fill={plateFill} stroke="black" /> : null}
                                        {fig === 'butt_hv' && <path d="M95,60 L120,60 L105,80 L105,90 L95,90 Z" fill={weldFill} stroke="black" />}
        
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
        
                                        {fig === 'butt_hy' && (
                                            <>
                                                {/* Left Plate: Square (Ends at 100) */}
                                                <rect x="20" y="60" width="80" height="30" fill={plateFill} stroke="black" />
                                                {/* Right Plate: Bevel with root face, RAPAT (Starts at 100) */}
                                                <path d="M180,60 L115,60 L100,80 L100,90 L180,90 Z" fill={plateFill} stroke="black" />
                                                {/* Weld Fill */}
                                                <path d="M100,60 L115,60 L100,80 Z" fill={weldFill} stroke="black" />
                                                {/* REMOVED REINFORCEMENT */}
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
                                        
                                        {/* K-Weld Butt (Double Bevel) */}
                                        {fig === 'butt_k' && (
                                            <>
                                                {/* Left Plate: Square */}
                                                <rect x="20" y="60" width="75" height="30" fill={plateFill} stroke="black" />
                                                {/* Right Plate: Double Bevel WITH ROOT FACE (Same as Input Guide) */}
                                                {/* M180,60 L120,60 L105,72 L105,78 L120,90 L180,90 Z */}
                                                <path d="M180,60 L120,60 L105,72 L105,78 L120,90 L180,90 Z" fill={plateFill} stroke="black" />
                                                
                                                {/* Weld Fills - Matching Top/Bottom Bevels */}
                                                {/* Top: Connects 95,60 -> 120,60 -> 105,72 -> 95,72 */}
                                                <path d="M95,60 L120,60 L105,72 L95,72 Z" fill={weldFill} stroke="black" />
                                                {/* Bottom: Connects 95,90 -> 120,90 -> 105,78 -> 95,78 */}
                                                <path d="M95,90 L120,90 L105,78 L95,78 Z" fill={weldFill} stroke="black" />
                                                
                                                {/* Gap Fill - Between x=95 and x=105 for the Root Face height */}
                                                <rect x="95" y="72" width="10" height="6" fill={weldFill} stroke="none" opacity="0.5"/>
                                            </>
                                        )}
                                    </>
                                )}
 
                                {hasBacking && fig !== 'butt_x' && fig !== 'butt_y' && fig !== 'butt_hy' && fig !== 'butt_k' && <rect x="85" y="90" width="30" height="5" fill={backingColor} stroke="black"/>}
                                
                                {/* Sealing Run - Butt (V & HV) */}
                                {hasSealingRun && (fig === 'butt_v' || s === 'HV') && (
                                    <path d="M90,90 Q100,95 110,90" fill="none" stroke={weldFill} strokeWidth="3" />
                                )}
                            </g>
                        )}
                        {jointType === 't_joint' && (
                            <g transform="translate(50, 30)">
                                <rect x="40" y="100" width="120" height="20" fill={plateFill} stroke="black" />
                                
                                {fig === 't_k' ? (
                                     // Double Bevel (K) with Visible Root Face & Gap
                                     // Web bottom at 95 (gap of 5). Root face width 6 (center 100).
                                     <path d="M90,20 L110,20 L110,80 L103,95 L97,95 L90,80 Z" fill={plate2Fill} stroke="black" />
                                ) : fig.includes('hv') ? (
                                     // HV: Single Bevel with Gap (y=95)
                                     <path d="M90,20 L110,20 L110,95 L100,95 L90,80 Z" fill={plate2Fill} stroke="black" />
                                ) : fig.includes('hy') ? (
                                     // HY: Single Bevel NO Gap (y=100) - Touching plate
                                     // Ensure web sits on flange at y=100
                                     <path d="M90,20 L110,20 L110,100 L100,100 L90,80 L90,20 Z" fill={plate2Fill} stroke="black" />
                                ) : (
                                     // Square
                                     <rect x="90" y="20" width="20" height="80" fill={plate2Fill} stroke="black" />
                                )}
 
                                {fig === 't_fillet' && (
                                    <>
                                        <path d="M90,100 L90,80 L70,100 Z" fill={weldFill} stroke="black" />
                                        {weldSide === '2-sided' && <path d="M110,100 L110,80 L130,100 Z" fill={weldFill} stroke="black" />}
                                    </>
                                )}
                                
                                {/* Standard HV/HY Welds - FLUSH with surface (x=90) */}
                                {fig.includes('hv') && weldSide !== '2-sided' && <path d="M90,80 L100,95 L110,95 L110,100 L90,100 Z" fill={weldFill} stroke="black" /> }
                                
                                {fig.includes('hy') && <path d="M90,80 L100,100 L90,100 Z" fill={weldFill} stroke="black" />}
                                
                                {fig === 't_k' && (
                                    <>
                                        {/* Left Weld: Bevel + Gap Penetration - FLUSH */}
                                        <path d="M90,80 L97,95 L97,100 L90,100 Z" fill={weldFill} stroke="black" />
                                        {/* Right Weld - FLUSH */}
                                        <path d="M110,80 L103,95 L103,100 L110,100 Z" fill={weldFill} stroke="black" />
                                        {/* Center Gap Fill */}
                                        <rect x="97" y="95" width="6" height="5" fill={weldFill} stroke="none" opacity="0.5"/>
                                    </>
                                )}
                                
                                {/* MODIFIED: Backing Bar for HV - Vertical behind web (x=110), sitting on flange (y=100) */}
                                {hasBacking && fig === 't_hv' && <rect x="110" y="80" width="5" height="20" fill={backingColor} stroke="black" />}
                                
                                {/* SPECIAL CASE: T-JOINT HV 2-SIDED (Flush Left + Fillet Right) */}
                                {fig === 't_hv' && weldSide === '2-sided' && (
                                    <>
                                        {/* Left Weld (Flush with Web x=90) */}
                                        <path d="M90,80 L100,95 L110,95 L110,100 L90,100 Z" fill={weldFill} stroke="black" />
                                        
                                        {/* Right Weld (Standard Fillet) */}
                                        <path d="M110,80 L110,100 L130,100 Z" fill={weldFill} stroke="black" />
                                    </>
                                )}
 
                                {/* Sealing Run - T-Joint (HV) - usually on the other side of the bevel? For T-HV, sealing run is typically on the other side (fillet-like or just bead) */}
                                {/* Only show if NOT 2-sided case handled above */}
                                {hasSealingRun && fig === 't_hv' && weldSide !== '2-sided' && (
                                    <path d="M110,100 L115,95 L115,105 Z" fill={weldFill} stroke="black" opacity="0.8"/>
                                )}
                                
                                {/* Additional Fillet (Convex shape on top of HV/HY weld) */}
                                {hasAdditionalFillet && (fig.includes('hv') || fig.includes('hy')) && weldSide === '1-sided' && (
                                    // Draw a curve from top of bevel to bottom of bevel on the weld face
                                    <path d="M90,80 L70,100 L90,100 Z" fill={weldFill} stroke="black" />
                                )}
                                
                                {/* HV Dimension - Hide if 2-sided */}
                                {fig.includes('hv') && weldSide !== '2-sided' && (
                                    <g>
                                       {/* Existing weld path */}
                                       
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
                                        {/* Corner to Corner */}
                                        <rect x="0" y="20" width="20" height="60" fill={plateFill} stroke="black" />
                                        <rect x="20" y="0" width="60" height="20" fill={plateFill} stroke="black" />
                                        
                                        {/* Existing Outside Weld (V) */}
                                        <path d="M0,20 L20,20 L20,0 Z" fill={weldFill} stroke="black" />
                                        
                                        {/* NEW: Opposite Inside Fillet Weld */}
                                        <path d="M20,20 L20,30 L30,20 Z" fill={weldFill} stroke="black" />
                                    </g>
                                 ) : (
                                    <g>
                                        {/* Overlap (3/10) */}
                                        <rect x="0" y="20" width="20" height="60" fill={plateFill} stroke="black" />
                                        <rect x="14" y="0" width="80" height="20" fill={plateFill} stroke="black" />
                                        
                                        {/* Weld filling the step (Fillet) */}
                                        {/* Step is from x=0 to x=14 on top of P1 (y=20) */}
                                        {/* Weld triangle: (0,20) -> (14,20) -> (14,0) approx */}
                                        <path d="M0,20 L14,20 L14,0 Z" fill={weldFill} stroke="black"/> 
                                    </g>
                                 )}
                            </g>
                        )}
                        {jointType === 'lap' && (
                             <g transform="translate(90, 80)">
                                <rect x="0" y="20" width="130" height="20" fill={plateFill} stroke="black" />
                                <rect x="30" y="0" width="100" height="20" fill={plateFill} stroke="black" />
                                <path d="M30,20 L30,0 L10,20 Z" fill={weldFill} stroke="black"/>
                                {/* Right Weld Removed */}
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
            </svg>
             {/* Dynamic Legend at Bottom of Welding View */}
             <div className="absolute bottom-2 left-0 w-full flex justify-center gap-6 text-xs font-bold font-mono pointer-events-none bg-white/50 py-1">
                {showAngle && <span><span className="text-red-600">Angle(α):</span> <span className="text-black">{angle}°</span></span>}
                {showGapVal && gap !== undefined && gap !== null && <span><span className="text-red-600">Gap(b):</span> <span className="text-black">{gap}mm</span></span>}
                {showRoot && rootFace !== undefined && rootFace !== null && <span><span className="text-red-600">Root(c):</span> <span className="text-black">{rootFace}mm</span></span>}
            </div>
        </div>
    </div>
  );
};
 
// --- SYMBOL RENDERER ---
const SymbolRenderer = ({ data, inputs, isBoxSection, hasBacking, weldLength, isWeldAllAround, t1, t2, hasSealingRun, hasAdditionalFillet, selectedClass, weldSide }) => {
    const s = data.symbol;
    const weldSizeText = calculateWeldSize(data, t1, t2, inputs.rootFace);
    const isDouble = s === 'X' || s === 'K';
    
    // NDT Logic for Tail
    let ndtText = null;
    const currentClassDetails = CLASS_DETAILS[selectedClass];
    if (currentClassDetails) {
        // Simple logic based on inspection class
        // CP A -> 100% -> "UT / MT"
        if (selectedClass === "CP A") {
            ndtText = "UT / MT";
        }
        // CP B1, B2, C1 -> 10% -> "UT / MT\n10%"
        else if (["CP B1", "CP B2", "CP C1"].includes(selectedClass)) {
            ndtText = "UT / MT\n10%";
        }
        // CP C2 -> None
    }

    return (
       <svg viewBox="0 0 300 100" className="w-full h-full">
           {/* Reference Line */}
           <line x1="20" y1="50" x2="280" y2="50" stroke="black" strokeWidth="2"/>
           
           {/* Tail Fork at the end (x=280) if NDT text exists */}
           {ndtText && (
               <g>
                    {/* NEW: TYP Text Above UT/MT */}
                    <text x="310" y="30" fontSize="8" fontWeight="bold" textAnchor="middle" fill="black">TYP (if applicable)</text>
                    
                   {/* Upper Branch (45 deg) */}
                   <line x1="280" y1="50" x2="295" y2="35" stroke="black" strokeWidth="2" />
                   {/* Lower Branch (-45 deg) */}
                   <line x1="280" y1="50" x2="295" y2="65" stroke="black" strokeWidth="2" />
                   {/* Text in between */}
                   <text x="300" y="50" fontSize="8" fontWeight="bold" dominantBaseline="middle">
                       {ndtText.split('\n').map((t, i) => (
                           <tspan x="300" dy={i === 0 ? "-3" : "10"} key={i}>{t}</tspan>
                       ))}
                   </text>
               </g>
           )}
           
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
               {/* MODIFIED: If Square/Half Square, draw 'I' symbol */}
               {(s === 'I') && (
                   <path d="M0,0 L0,-15" stroke="black" strokeWidth="2" fill="none"/> // Basic Vertical Line for Square
               )}
 
               {/* MODIFIED: V-Weld 1-Sided (and 3-member) style: _\_/ */}
               {s === 'V' && inputs.jointType !== 'corner' && (
                   <g>
                        {/* 1-Sided Butt V >= 16mm logic uses s='I' via activeWeld transform, 
                            so this block runs for 3-member OR V-weld <16mm.
                            Wait, user requested "1-sided weld butt joint V weld utk plate thickness >= 16mm, symbol sama mcm 3-member joint".
                            And 3-member joint uses _\_/.
                            So for V >= 16mm (which is transformed to s='I', fig='butt_sq'), we need _\_/.
                            That is handled in the `data.fig_type === 'butt_sq'` block below.
                            
                            So here, we handle V < 16mm. 
                            Does V < 16mm also need _\_/ ? 
                            "1-sided weld butt joint V weld utk plate thickness >= 16mm, symbol sama mcm 3-member joint".
                            Implies < 16mm uses standard V.
                            
                            Standard V: M0,0 L-10,-15 M0,0 L10,-15
                            _\_/ : M-10,-15 L-3,0  M3,0 L10,-15
                        */}
                        
                        {(inputs.jointType === 'three_member') ? (
                            // 3-Member Joint Symbol: _\_/
                            <>
                                <path d="M-10,-15 L-3,0" stroke="black" strokeWidth="2" fill="none"/>
                                <path d="M3,0 L10,-15" stroke="black" strokeWidth="2" fill="none"/>
                            </>
                        ) : (
                            // Standard V-Weld (< 16mm)
                            <path d="M0,0 L-10,-15 M0,0 L10,-15" stroke="black" strokeWidth="2" fill="none"/>
                        )}
                   </g>
               )}
               
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
               {/* MODIFIED: Fillet F Symbol - Closed Triangle Z */}
               {s === 'F' && inputs.jointType !== 'corner' && <path d="M0,0 L0,-15 L15,0 Z" stroke="black" strokeWidth="2" fill="none"/>}
               
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
               
               {/* MODIFIED: Backing Bar M symbol moved BELOW dotted line for HV, V, and Square Edge only */}
               {(hasBacking || isBoxSection) && (s === 'HV' || s === 'V' || s === 'I') && (
                   <g transform="translate(0, 20)"> {/* Positive Y moves down. Dotted line is at y=60 (relative +10 from center). +20 puts it safely below. */}
                       <rect x="-5" y="-5" width="10" height="10" stroke="black" fill="none" strokeWidth="1"/>
                       <text x="0" y="3" textAnchor="middle" fontSize="8" fontWeight="bold">M</text>
                   </g>
               )}
 
               {/* Sealing Run Symbol:
                   Standard: Small curve on opposite side.
                   2-Sided T-HV special case: Fillet symbol on opposite side (dotted line).
               */}
               {hasSealingRun && (
                   <>
                       {(inputs.jointType === 't_joint' && s === 'HV') ? (
                           // Special case for T-Joint 2-Sided HV: Fillet symbol on dotted line
                           // Dotted line is at y=60. Fillet should be mirrored.
                           // Standard fillet top is |__ (0,0 L0,-15 L15,0).
                           // Bottom (mirrored) would be |`` (0,10 L0,25 L15,10).
                           <path d="M0,10 L0,25 L15,10 Z" stroke="black" strokeWidth="2" fill="none" />
                       ) : (
                           // Standard sealing run curve - semicircle
                           // Starts from dotted line (y=10 relative to g)
                           <path d="M-5,10 A5,5 0 0,0 5,10" fill="none" stroke="black" strokeWidth="2" />
                       )}
                   </>
               )}
 
               {/* Additional Fillet Symbol - Triangle overlaid on the groove symbol */}
               {hasAdditionalFillet && (
                   // Stack ON TOP of existing symbol.
                   // Existing symbols (HV/HY) go from y=0 to y=-15.
                   // We want to stack this fillet physically above it.
                   // Move it up by 15 units? so it goes from -15 to -30?
                   // Or just draw it "on" the bevel line?
                   // The user requested: "tak overlap... letak atas symbol HV".
                   // Let's translate it up by -18 to give a small gap.
                   <g transform="translate(0, -18)">
                        <path d="M0,0 L0,-15 L15,0 Z" stroke="black" strokeWidth="2" fill="none" /> 
                   </g>
               )}

               {/* SPECIAL CASE: 1-Sided Butt HV (Thickness >= 16mm) - Symbol |_/_ */}
               {/* This is handled by activeWeld symbol being 'I' and fig_type 'butt_half_sq' */}
               {/* But wait, 'I' symbol is just a line. We need |_/_ */}
               {/* Let's intercept it based on data.fig_type */}
               {data.fig_type === 'butt_half_sq' && (
                    <g>
                        {/* Remove the generic 'I' if it was rendered by s==='I' block above? 
                            The s==='I' block renders `M0,0 L0,-15`.
                            We need `|_/_`.
                            Vertical: M0,0 L0,-15
                            Bottom: M0,0 L5,0
                            Bevel: M5,0 L10,-15
                        */}
                         <path d="M0,0 L5,0 L15,-15" stroke="black" strokeWidth="2" fill="none"/>
                    </g>
               )}
               
               {/* SPECIAL CASE: 1-Sided Butt V (Thickness >= 16mm) - Symbol _\_/ */}
               {/* Handled by s='I' and butt_sq logic */}
               {data.fig_type === 'butt_sq' && (
                    <g>
                        {/* Removed the center vertical line per request */}
                        <path d="M-10,-15 L-3,0" stroke="black" strokeWidth="2" fill="none"/>
                        <path d="M3,0 L10,-15" stroke="black" strokeWidth="2" fill="none"/>
                    </g>
               )}
           </g>
           
           {/* Dimensions (a, z) - LEFT of symbol - Top Value */}
           <text x="115" y="45" textAnchor="end" className="text-xs font-medium font-mono">
             {weldSizeText}
           </text>
 
           {/* Dimensions (a, z) - LEFT of symbol - Bottom Value for Double Sided */}
           {/* MODIFIED: Show bottom value for standard double sided OR T-Joint 2-Sided Fillet */}
           {(isDouble || (inputs.jointType === 't_joint' && s === 'F' && weldSide === '2-sided') || (inputs.jointType === 'corner' && inputs.cornerOption === '1')) && (
               <text x="115" y="75" textAnchor="end" className="text-xs font-medium font-mono">
                 {weldSizeText}
               </text>
           )}
           
           {/* Weld Length - RIGHT of symbol */}
           {weldLength && !isWeldAllAround && (
                <>
                    {/* Changed font-bold to font-normal per request */}
                    <text x="165" y="45" textAnchor="start" className="text-xs font-normal font-mono">
                        {weldLength}
                    </text>
                    {/* MODIFIED: Show bottom length for standard double sided OR T-Joint 2-Sided Fillet */}
                    {(isDouble || (inputs.jointType === 't_joint' && s === 'F' && weldSide === '2-sided') || (inputs.jointType === 'corner' && inputs.cornerOption === '1')) && (
                        <text x="165" y="80" textAnchor="start" className="text-xs font-normal font-mono">
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
           
           {/* Special Corner Joint Logic for Symbols */}
           {inputs.jointType === 'corner' && (
               <g transform="translate(140, 50)">
                   {inputs.cornerOption === '1' ? (
                       // Corner to Corner: Double Fillet (Top and Bottom)
                       <>
                           <path d="M0,0 L0,-15 L15,0 Z" stroke="black" strokeWidth="2" fill="none" />
                           <path d="M0,10 L0,25 L15,10 Z" stroke="black" strokeWidth="2" fill="none" />
                       </>
                   ) : (
                       // Overlap: Single Fillet Top
                       <path d="M0,0 L0,-15 L15,0 Z" stroke="black" strokeWidth="2" fill="none" />
                   )}
               </g>
           )}

           {/* Special T-Joint Double Fillet Logic (13b) */}
           {/* Check for T-Joint & 2-Sided & Fillet */}
           {inputs.jointType === 't_joint' && data.symbol === 'F' && weldSide === '2-sided' && (
               <g transform="translate(140, 50)">
                   {/* Top Fillet (already drawn by s='F' block? No, s='F' block draws top only. We need bottom too.) */}
                   {/* But wait, the s='F' block is generic. We need to ADD the bottom one here. */}
                   {/* Bottom Fillet (Mirrored on dotted line y=60? No, standard dotted line is y=60 relative to svg? Wait. 
                       Reference line is y=50. Dotted line is y=60.
                       Top symbol sits on y=50 (M0,0).
                       Bottom symbol should sit on y=60 (M0,10).
                   */}
                   <path d="M0,10 L0,25 L15,10 Z" stroke="black" strokeWidth="2" fill="none" />
               </g>
           )}

           {/* 3-Member Joint Symbol Override */}
           {/* Handled in 'V' block with condition */}

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
  const [mainAssembly, setMainAssembly] = useState('All');
  
  // NEW STATES
  const [weldLength, setWeldLength] = useState("");
  const [isWeldAllAround, setIsWeldAllAround] = useState(false);
  
  const [hasBacking, setHasBacking] = useState(false);
  const [activeField, setActiveField] = useState(null);
  
  // New State for Weld Side Option (for Butt/T-Joint)
  const [weldSide, setWeldSide] = useState('1-sided');
  const [hasSealingRun, setHasSealingRun] = useState(false); 
  const [hasAdditionalFillet, setHasAdditionalFillet] = useState(false);
  
  // NEW STATE: Inspection Access
  const [inspectionAccess, setInspectionAccess] = useState('yes');

  // New States for Stress/Safety Filtering
  const [stressCat, setStressCat] = useState('All');
  const [safetyCat, setSafetyCat] = useState('All');
 
  // --- NEW: WAREHOUSE PLATES CHECK ---
  const availablePlates = [2, 3, 4, 5, 6, 8, 10, 12, 16, 20, 25, 30, 35, 40, 50, 60, 75, 115, 150, 200];
  const [invalidPlates, setInvalidPlates] = useState([]);
  
  // --- NEW STATE: SELECTED WELD CLASS ---
  const [selectedClass, setSelectedClass] = useState("CP A");
 
  // DYNAMIC FILTERING OF WELD TYPES BASED ON SIDE
  const allWeldTypes = WELD_DATA[jointType].types;
  
  // Updated filteredWeldTypes to use validPairs for strict filtering
  const filteredWeldTypes = useMemo(() => {
    let types = WELD_DATA[jointType].types;

    // Filter by Side (1-Sided vs 2-Sided)
    if (jointType === 'butt') {
        if (weldSide === '1-sided') {
            types = types.filter(t => ['V', 'HV'].includes(t.symbol)); 
        } else {
            types = types.filter(t => ['V', 'HV', 'X', 'K'].includes(t.symbol));
        }
    } else if (jointType === 't_joint') {
         if (weldSide === '1-sided') {
             types = types.filter(t => ['HV', 'HY', 'F'].includes(t.symbol));
         } else {
             types = types.filter(t => ['K', 'HV', 'F'].includes(t.symbol));
         }
    }

    // Filter by Stress/Safety Category using new strict pairing logic
    // Even in basic mode, we respect Main Assembly safety constraints
    types = types.filter(t => {
         const validClassesForType = t.availableClasses.filter(cls => {
             const details = CLASS_DETAILS[cls];
             if (!details) return false;
             
             const pairs = details.validPairs || [];
             
             // Check pairing match with user selection
             const matchingPairs = pairs.filter(p => {
                 const sMatch = mode !== 'advanced' || stressCat === 'All' || p.stress === stressCat;
                 const saMatch = mode !== 'advanced' || safetyCat === 'All' || p.safety === safetyCat;
                 return sMatch && saMatch;
             });

             if (matchingPairs.length === 0) return false;

             // Check Main Assembly Safety Constraint
             if (mainAssembly !== 'All') {
                 // Must have at least one matching pair with Safety High or Medium
                 if (!matchingPairs.some(p => p.safety === 'High' || p.safety === 'Medium')) return false;
             }

             return true;
         });

         return validClassesForType.length > 0;
    });

    return types;
  }, [jointType, weldSide, stressCat, safetyCat, mode, mainAssembly]);

  // Ensure weldIndex is valid after filtering
  useEffect(() => {
      if (weldIndex >= filteredWeldTypes.length) {
          setWeldIndex(0);
      }
  }, [filteredWeldTypes.length]);

  const currentWeld = filteredWeldTypes[weldIndex] || filteredWeldTypes[0] || WELD_DATA[jointType].types[0]; 
  const canHaveBox = WELD_DATA[jointType]?.hasBoxOption && currentWeld?.allowBox;
  const isT3Enabled = WELD_DATA[jointType]?.hasT3 || isBoxSection;
  const isT4Enabled = isBoxSection;
  
  const showSideOption = jointType === 'butt' || jointType === 't_joint';
 
  // FILTER AVAILABLE CLASSES FOR BUTTONS
  const maxPlateThickness = Math.max(Number(t1), Number(t2));

  // Updated visibleClasses to use validPairs for strict filtering
  const visibleClasses = useMemo(() => {
    if (!currentWeld?.availableClasses) return [];
    
    let classes = currentWeld.availableClasses;

    // 1. Thickness Constraint: If t <= 8, remove CP A
    if (maxPlateThickness <= 8) {
        classes = classes.filter(c => c !== "CP A");
    }

    // 2. Strict Filter based on Stress/Safety/MainAssembly Pairs
    classes = classes.filter(cls => {
        const details = CLASS_DETAILS[cls];
        if (!details) return false;

        const pairs = details.validPairs || [];

        // Find pairs matching current Stress/Safety selection
        const matchingPairs = pairs.filter(p => {
             const sMatch = mode !== 'advanced' || stressCat === 'All' || p.stress === stressCat;
             const saMatch = mode !== 'advanced' || safetyCat === 'All' || p.safety === safetyCat;
             return sMatch && saMatch;
        });

        if (matchingPairs.length === 0) return false;

        // Main Assembly Constraint
        if (mainAssembly !== 'All') {
             // Must have at least one matching pair with Safety High or Medium
             const isSafeEnough = matchingPairs.some(p => p.safety === 'High' || p.safety === 'Medium');
             if (!isSafeEnough) return false;
        }

        return true;
    });

    return classes;
  }, [currentWeld, stressCat, safetyCat, mode, maxPlateThickness, mainAssembly]);

  // Auto-select valid class if current selection is invalid
  useEffect(() => {
      if (visibleClasses.length > 0 && !visibleClasses.includes(selectedClass)) {
          setSelectedClass(visibleClasses[0]);
      }
  }, [visibleClasses, selectedClass]);
 
  // Auto-adjust angle, gap, rootFace for specific joints
  useEffect(() => {
      if (jointType === 'three_member') {
          setAngle(30);
          setGap(4);
          setRootFace(0);
      } else {
          // Existing logic for others
          const maxThickness = Math.max(Number(t1), Number(t2), isT3Enabled ? Number(t3) : 0, isT4Enabled ? Number(t4) : 0);
          if (maxThickness >= 20) { setAngle(30); } else { setAngle(50); }

           if (jointType !== 'three_member') {
               if (rootFace === 0) setRootFace(2);
               if (gap === 4) setGap(2); 
           }
      }
  }, [t1, t2, t3, t4, isT3Enabled, isT4Enabled, jointType]);
 
  // Auto-clear weldLength
  useEffect(() => {
      if (isWeldAllAround) setWeldLength("");
  }, [isWeldAllAround]);
 
  const currentClassDetails = CLASS_DETAILS[selectedClass] || {};

  // --- NEW LOGIC: FILTERED DISPLAY DETAILS ---
  // Calculates the Stress/Safety text to display based on the selected class AND user filters/main assembly.
  const getFilteredDetails = () => {
      const details = CLASS_DETAILS[selectedClass];
      if (!details) return { stress: "-", safety: "-", stressVal: "-" };

      let pairs = details.validPairs || [];

      // 1. Filter by Main Assembly (Restricted assemblies remove Low Safety)
      if (['Bogie', 'Underframe', 'Superstructure'].includes(mainAssembly)) {
          pairs = pairs.filter(p => p.safety === 'High' || p.safety === 'Medium');
      }

      // 2. Filter by Advanced Mode Selection
      // If user selected specific categories in Advanced Mode, filter pairs to match.
      if (mode === 'advanced') {
          if (stressCat !== 'All') {
              pairs = pairs.filter(p => p.stress === stressCat);
          }
          if (safetyCat !== 'All') {
              pairs = pairs.filter(p => p.safety === safetyCat);
          }
      }

      // 3. Deduplicate results for display (e.g., if multiple pairs result in "High", show just "High")
      const uniqueStress = [...new Set(pairs.map(p => p.stress))];
      const uniqueSafety = [...new Set(pairs.map(p => p.safety))];
      const uniqueStressVal = [...new Set(pairs.map(p => p.stressVal))];

      return {
          stress: uniqueStress.join(" / "),
          safety: uniqueSafety.join(" / "),
          stressVal: uniqueStressVal.join(" / ")
      };
  };

  const displayDetails = getFilteredDetails();

  // Helper function to render stacked text if contains "/"
  const renderStackedText = (text) => {
    if (!text) return null;
    if (text.includes('/')) {
        const parts = text.split('/').map(s => s.trim());
        return (
            <div className="flex flex-col items-start leading-tight">
                {parts.map((part, i) => (
                    <span key={i} className="block">{part}</span>
                ))}
            </div>
        );
    }
    return text;
  };
 
  // Logic for showing options
  const showSealingRunOption = weldSide === '2-sided' && jointType === 'butt' && (currentWeld?.symbol === 'V' || currentWeld?.symbol === 'HV');
  const showBackingOption = weldSide === '1-sided' && jointType !== 'corner' && jointType !== 'three_member' && (currentWeld?.symbol === 'V' || currentWeld?.symbol === 'HV');
  const isBackingAuto = Math.max(Number(t1), Number(t2)) >= 16;
  
  // NEW: Additional Fillet Option
  const showAdditionalFilletOption = (jointType === 't_joint' && weldSide === '1-sided' && (currentWeld?.symbol === 'HV' || currentWeld?.symbol === 'HY')) || isBoxSection;
  
  // New helper for display name
  const getWeldDisplayName = () => {
      let activeWeldName = currentWeld.name;
      const maxThickness = Math.max(Number(t1), Number(t2));
      
      if (jointType === 'butt' && weldSide === '1-sided' && maxThickness >= 16) {
          if (currentWeld.symbol === 'V') {
              activeWeldName = "Square Edge Weld (I)";
          } else if (currentWeld.symbol === 'HV') {
              activeWeldName = "Half Square Edge Weld";
          }
      }

      let name = isBoxSection ? "Box Girder (4x HV)" : activeWeldName;
      const extras = [];
      
      if (hasBacking) extras.push("Backing Bar");
      
      if (jointType === 't_joint' && weldSide === '2-sided' && currentWeld.symbol === 'HV') {
           extras.push("Fillet as sealing run");
      } else if (hasSealingRun) {
           extras.push("Sealing Run");
      }

      if (hasAdditionalFillet) extras.push("Additional Fillet");

      if (extras.length > 0) {
          return `${name} with ${extras.join(" & ")}`;
      }
      return name;
  };
 
  // --- WELD NUMBER LOGIC ---
  const getWeldNumber = () => {
    if (isBoxSection) return "10b, 10c & 10e";
 
    if (jointType === 'lap') return "13e";
 
    if (jointType === 'corner') {
        return cornerOption === '2' ? "13c" : "13d";
    }
 
    if (jointType === 'three_member') return "12";
 
    const maxThickness = Math.max(Number(t1), Number(t2));
    let isSquare = false;
    let isHalfSquare = false;
    
    if (jointType === 'butt' && weldSide === '1-sided' && maxThickness >= 16) {
        if (currentWeld.symbol === 'V') isSquare = true;
        if (currentWeld.symbol === 'HV') isHalfSquare = true;
    }
 
    if (jointType === 'butt') {
        if (weldSide === '2-sided') {
            if (currentWeld.symbol === 'V') return "2c";
            if (currentWeld.symbol === 'HV') return "3c";
            if (currentWeld.symbol === 'X') return "6";
            if (currentWeld.symbol === 'K') return "7"; 
        } else {
            if (isSquare && hasBacking) return "2d";
            if (isHalfSquare && hasBacking) return "3d";
            
            if (currentWeld.symbol === 'V') {
                return hasBacking ? "2b" : "2a";
            }
            if (currentWeld.symbol === 'HV') {
                return hasBacking ? "3b" : "3a";
            }
        }
    }
 
    if (jointType === 't_joint') {
        if (weldSide === '2-sided') {
            if (currentWeld.symbol === 'K') return "7";
            if (currentWeld.symbol === 'HV') return "10b";
            if (currentWeld.symbol === 'F') return "13b";
        } else {
            if (currentWeld.symbol === 'F') return "13a";
            if (currentWeld.symbol === 'HV') {
                if (hasAdditionalFillet) return "10c";
                if (hasBacking) return "10e";
                return "10a";
            }
            if (currentWeld.symbol === 'HY') {
                if (hasAdditionalFillet) return "11c";
                return "11a";
            }
        }
    }
 
    return "N/A";
  };
 
  // Prepare Active Weld Object for Renderers
  let activeWeld = { ...currentWeld };
  const maxThickness = Math.max(Number(t1), Number(t2));
  
  if (jointType === 'butt' && weldSide === '1-sided' && maxThickness >= 16) {
      if (currentWeld.symbol === 'V') {
          activeWeld = { ...currentWeld, symbol: 'I', fig_type: 'butt_sq', name: "Square Edge Weld" };
      } else if (currentWeld.symbol === 'HV') {
          activeWeld = { ...currentWeld, symbol: 'I', fig_type: 'butt_half_sq', name: "Half Square Edge Weld" }; 
      }
  }

  return (
    <div className="min-h-screen bg-slate-100 font-sans text-slate-800 p-4">
      <div className="max-w-7xl mx-auto bg-white rounded-lg shadow-lg overflow-hidden relative">
        <div className="bg-slate-900 text-white p-4 flex items-center justify-between">
            <div>
                <h1 className="text-xl font-bold flex items-center gap-2"><IconRuler className="text-yellow-400"/> Joint Preparation of Welds <span className="text-xs bg-yellow-500 text-black px-1 rounded">V1.0</span></h1>
                <div className="flex items-center gap-2 text-xs text-gray-400"><IconBox size={14} /> EN 15085-3:2023</div>
            </div>
            <div className="flex items-center gap-2">
                <span className={`text-xs uppercase font-bold ${mode==='basic' ? 'text-white' : 'text-gray-500'}`}>Basic</span>
                <button onClick={() => setMode(mode === 'basic' ? 'advanced' : 'basic')} className={`w-12 h-6 rounded-full p-1 transition-colors duration-300 ${mode === 'advanced' ? 'bg-yellow-500' : 'bg-slate-700'}`}><div className={`w-4 h-4 rounded-full bg-white transform transition-transform duration-300 ${mode === 'advanced' ? 'translate-x-6' : 'translate-x-0'}`} /></button>
                <span className={`text-xs uppercase font-bold ${mode==='advanced' ? 'text-yellow-400' : 'text-gray-500'}`}>Advanced</span>
            </div>
        </div>
 
        <div className="grid grid-cols-1 md:grid-cols-12 gap-0">
            <div className="md:col-span-5 p-5 bg-slate-50 border-r border-slate-200">

                {/* MAIN ASSEMBLY SELECTION */}
                <div className="mb-4 bg-gray-100 p-2 rounded border border-gray-200">
                    <label className="block text-[10px] font-bold uppercase text-gray-600 mb-1 flex items-center gap-1"><IconTrain size={12}/> Main Assembly</label>
                    <select 
                        value={mainAssembly} 
                        onChange={(e) => {
                            setMainAssembly(e.target.value);
                            // Reset Safety Cat if switching to restricted assembly and current is Low
                            if (['Bogie', 'Underframe', 'Superstructure'].includes(e.target.value)) {
                                if (safetyCat === 'Low') setSafetyCat('All');
                            }
                        }}
                        className="w-full p-1 text-xs border rounded bg-white font-semibold"
                    >
                        <option value="All">All</option>
                        <option value="Bogie">Bogie</option>
                        <option value="Underframe">Underframe</option>
                        <option value="Superstructure">Superstructure</option>
                    </select>
                </div>
                
                {/* WELD SIDE SELECTION */}
                {showSideOption && (
                    <div className="mb-4 bg-indigo-50 p-2 rounded border border-indigo-100">
                        <label className="block text-[10px] font-bold uppercase text-indigo-800 mb-1">Weld Side</label>
                        <div className="flex gap-2 mb-2">
                            <button 
                                onClick={() => setWeldSide('1-sided')}
                                className={`flex-1 text-xs border rounded py-1 font-bold ${weldSide === '1-sided' ? 'bg-indigo-600 text-white border-indigo-600' : 'bg-white text-indigo-600 border-indigo-200'}`}
                            >
                                1-Sided Weld
                            </button>
                            <button 
                                onClick={() => setWeldSide('2-sided')}
                                className={`flex-1 text-xs border rounded py-1 font-bold ${weldSide === '2-sided' ? 'bg-indigo-600 text-white border-indigo-600' : 'bg-white text-indigo-600 border-indigo-200'}`}
                            >
                                2-Sided Weld
                            </button>
                        </div>

                        {/* INSPECTION ACCESS SELECTION */}
                        <label className="block text-[10px] font-bold uppercase text-indigo-800 mb-1">Inspection Access</label>
                        <div className="flex gap-2 mb-1">
                            <button 
                                onClick={() => setInspectionAccess('yes')}
                                className={`flex-1 text-xs border rounded py-1 font-bold ${inspectionAccess === 'yes' ? 'bg-indigo-600 text-white border-indigo-600' : 'bg-white text-indigo-600 border-indigo-200'}`}
                            >
                                Possible
                            </button>
                            <button 
                                onClick={() => setInspectionAccess('no')}
                                className={`flex-1 text-xs border rounded py-1 font-bold ${inspectionAccess === 'no' ? 'bg-indigo-600 text-white border-indigo-600' : 'bg-white text-indigo-600 border-indigo-200'}`}
                            >
                                Not Possible
                            </button>
                        </div>
                        <p className="text-[9px] text-indigo-600 italic">
                            {inspectionAccess === 'no' ? "No access for inspection in production & maintenance." : "Access available for inspection in production & maintenance."}
                        </p>
                    </div>
                )}
                
                <div className="mb-4"><label className="block text-xs font-bold uppercase text-slate-500 mb-2">Joint Type</label><div className="flex flex-wrap gap-2">{Object.entries(WELD_DATA).map(([key, val]) => (<button key={key} onClick={() => setJointType(key)} className={`px-3 py-1.5 text-xs rounded border font-semibold ${jointType === key ? 'bg-blue-600 text-white border-blue-600' : 'bg-white text-slate-600 hover:bg-slate-200'}`}>{val.label}</button>))}</div></div>
                
                <div className="mb-4">
                    <label className="block text-xs font-bold uppercase text-slate-500 mb-2">Weld Type</label>
                    <select className="w-full p-2 text-sm border rounded bg-white font-semibold" value={weldIndex} onChange={(e) => setWeldIndex(Number(e.target.value))}>
                        {filteredWeldTypes.map((t, idx) => (<option key={t.id} value={idx}>{t.symbol} - {t.name}</option>))}
                    </select>
                    
                    {/* BACKING BAR OPTION (1-Sided V/HV) OR BOX SECTION (Mandatory) */}
                    {(showBackingOption || isBoxSection) && (
                        <div className="mt-2 flex items-center gap-2">
                             <input 
                                type="checkbox" 
                                id="backing"
                                checked={hasBacking} 
                                disabled={isBackingAuto || isBoxSection} // Disabled if auto-selected by thickness OR box section
                                onChange={(e) => setHasBacking(e.target.checked)} 
                                className="w-4 h-4 text-blue-600 rounded disabled:opacity-50"
                            />
                            <label htmlFor="backing" className="text-xs font-bold text-slate-600">
                                {isBoxSection ? "With Backing Bar (Mandatory)" : (isBackingAuto ? "Automatic Backing (Thickness ≥ 16mm)" : "With Backing Bar")}
                            </label>
                        </div>
                    )}
 
                    {/* SEALING RUN OPTION (2-Sided V/HV) */}
                    {showSealingRunOption && (
                        <div className="mt-2 flex items-center gap-2">
                             <input 
                                type="checkbox" 
                                id="sealing"
                                checked={hasSealingRun} 
                                disabled={true} // Since showSealingRunOption is only true for 2-sided Butt V/HV, which the user wants mandatory.
                                onChange={(e) => setHasSealingRun(e.target.checked)} 
                                className="w-4 h-4 text-blue-600 rounded disabled:opacity-50"
                            />
                            <label htmlFor="sealing" className="text-xs font-bold text-slate-600">With Sealing Run (Mandatory)</label>
                        </div>
                    )}
 
                    {/* ADDITIONAL FILLET OPTION (1-Sided T-Joint HV/HY) OR BOX SECTION (Mandatory) */}
                    {showAdditionalFilletOption && (
                         <div className="mt-2 flex items-center gap-2">
                            <input 
                               type="checkbox" 
                               id="addFillet"
                               checked={hasAdditionalFillet} 
                               disabled={isBoxSection} // Disabled if mandatory for box section
                               onChange={(e) => setHasAdditionalFillet(e.target.checked)} 
                               className="w-4 h-4 text-blue-600 rounded disabled:opacity-50"
                           />
                           <label htmlFor="addFillet" className="text-xs font-bold text-slate-600">
                               {isBoxSection ? "With Additional Fillet (Mandatory)" : "With Additional Fillet"}
                           </label>
                       </div>
                    )}
                </div>
 
                {WELD_DATA[jointType].hasOptions && !isBoxSection && (<div className="mb-4 bg-yellow-50 p-2 rounded border border-yellow-200"><label className="block text-[10px] font-bold uppercase text-yellow-800 mb-1">Corner Config</label><div className="flex gap-2"><button onClick={()=>setCornerOption('1')} className={`flex-1 text-xs border rounded py-1 ${cornerOption==='1' ? 'bg-yellow-400 font-bold' : 'bg-white'}`}>Corner to Corner</button><button onClick={()=>setCornerOption('2')} className={`flex-1 text-xs border rounded py-1 ${cornerOption==='2' ? 'bg-yellow-400 font-bold' : 'bg-white'}`}>Overlap</button></div></div>)}
                {canHaveBox && (<div className="mb-4 bg-blue-50 p-3 rounded border border-blue-200"><label className="flex items-center gap-2 cursor-pointer"><input type="checkbox" checked={isBoxSection} onChange={(e) => setIsBoxSection(e.target.checked)} className="w-4 h-4 text-blue-600 rounded"/><span className="text-xs font-bold text-blue-800 flex items-center gap-2"><IconLayers size={14}/> Box Section Assembly</span></label></div>)}
                <div className="mb-4"><label className="block text-xs font-bold uppercase text-slate-500 mb-1">Input Guide</label><JointVisualizer jointType={jointType} weldTypeData={activeWeld} t1={t1} t2={t2} t3={t3} t4={t4} cornerOption={cornerOption} activeField={activeField} isBoxSection={isBoxSection}/></div>
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
                {mode === 'advanced' && (
                    <div className="mt-4 p-3 bg-yellow-50 border border-yellow-200 rounded animate-fade-in">
                        {/* FILTER SECTION */}
                        <div className="mb-4 pb-3 border-b border-yellow-200">
                             <div className="flex items-center gap-2 mb-2 text-[10px] font-bold text-slate-500 uppercase">
                                <IconFilter size={12}/> Filter Welding Category
                             </div>
                             <div className="grid grid-cols-2 gap-2">
                                <div>
                                    <label className="block text-[9px] font-bold text-slate-500 mb-1">Stress Category</label>
                                    <select value={stressCat} onChange={(e) => setStressCat(e.target.value)} className="w-full p-1 text-xs border rounded bg-white">
                                        <option value="All">All</option>
                                        <option value="High">High</option>
                                        <option value="Medium">Medium</option>
                                        <option value="Low">Low</option>
                                    </select>
                                </div>
                                <div>
                                    <label className="block text-[9px] font-bold text-slate-500 mb-1">Safety Category</label>
                                    <select value={safetyCat} onChange={(e) => setSafetyCat(e.target.value)} className="w-full p-1 text-xs border rounded bg-white">
                                        <option value="All">All</option>
                                        <option value="High">High</option>
                                        <option value="Medium">Medium</option>
                                        {/* Hide Low option if main assembly implies High/Medium only */}
                                        { !['Bogie', 'Underframe', 'Superstructure'].includes(mainAssembly) && <option value="Low">Low</option> }
                                    </select>
                                </div>
                             </div>
                        </div>

                        {/* ADVANCED PARAMETERS */}
                        <div className="grid grid-cols-3 gap-2">
                            <div><label className="text-[10px] font-bold text-slate-500">Angle (°)</label><input type="number" value={angle} onChange={(e) => setAngle(Number(e.target.value))} disabled={jointType === 'three_member'} className="w-full p-1 border rounded text-sm font-mono disabled:bg-slate-100 disabled:text-slate-400"/></div>
                            <div><label className="text-[10px] font-bold text-slate-500">Gap (mm)</label><input type="number" value={gap} onChange={(e) => setGap(Number(e.target.value))} disabled={jointType === 'three_member' || jointType === 'corner'} className="w-full p-1 border rounded text-sm font-mono disabled:bg-slate-100 disabled:text-slate-400"/></div>
                            <div><label className="text-[10px] font-bold text-slate-500">Root (mm)</label><input type="number" value={rootFace} onChange={(e) => setRootFace(Number(e.target.value))} disabled={jointType === 'three_member' || jointType === 'corner'} className="w-full p-1 border rounded text-sm font-mono disabled:bg-slate-100 disabled:text-slate-400"/></div>
                        </div>
                    </div>
                )}
            </div>
 
            <div className="md:col-span-7 p-5 bg-white">
                <h2 className="text-md font-bold text-slate-800 mb-3 flex items-center gap-2"><IconInfo size={16} /> Technical Specification</h2>
                <div className={`p-3 mb-4 rounded-r border-l-4 ${hasBacking ? 'bg-amber-50 border-amber-500' : 'bg-blue-50 border-blue-500'}`}>
                    <div>
                        <div className="flex justify-between items-start">
                            <div>
                                <h3 className="font-bold text-sm text-slate-900">{getWeldDisplayName()}</h3>
                                <p className="text-xs text-slate-600 mt-1">{currentWeld.description}</p>
                            </div>
                            {/* WELD CLASS SELECTOR */}
                             <div className="flex flex-col items-end">
                                <span className="text-[10px] font-bold text-slate-400 uppercase mb-1">Performance Class</span>
                                <div className="flex gap-1">
                                    {visibleClasses.map((cls) => (
                                        <button 
                                            key={cls}
                                            onClick={() => setSelectedClass(cls)}
                                            className={`text-[10px] px-2 py-1 rounded border ${selectedClass === cls ? 'bg-blue-600 text-white border-blue-600' : 'bg-white text-slate-600 border-slate-200 hover:bg-slate-100'}`}
                                        >
                                            {cls}
                                        </button>
                                    ))}
                                </div>
                            </div>
                        </div>
                        
                        {/* WELD NO INFO (ADDED) */}
                        <div className="mt-2 text-xs font-bold text-slate-700 bg-slate-100 px-2 py-1 rounded inline-flex items-center gap-2">
                             <IconHash size={12}/> Weld No. (EN 15085-3 Annex B): <span className="text-blue-600">{getWeldNumber()}</span>
                        </div>
 
                        {/* CLASS DETAILS PANEL - Always Shown */}
                        {true && (
                            <div className="mt-3 bg-white/50 rounded-lg border border-blue-100 p-2 text-xs">
                                <div className="grid grid-cols-2 md:grid-cols-4 gap-2 mb-2">
                                    <div className="bg-blue-50 p-1.5 rounded">
                                        <span className="block text-[9px] text-blue-400 font-bold uppercase">Stress</span>
                                        <div className="font-bold text-blue-900 flex flex-col items-start gap-0.5">
                                            {/* UPDATED: Use displayDetails which dynamically filters pairs */}
                                            <div className="flex items-center gap-1"><IconActivity size={10}/> {renderStackedText(displayDetails.stress)}</div>
                                            <span className="text-[9px] font-normal opacity-80">{renderStackedText(displayDetails.stressVal)}</span>
                                        </div>
                                    </div>
                                    <div className="bg-green-50 p-1.5 rounded">
                                        <span className="block text-[9px] text-green-400 font-bold uppercase">Safety</span>
                                        <div className="font-bold text-green-900 flex flex-col gap-0.5"><div className="flex items-center gap-1"><IconShield size={10}/> {renderStackedText(displayDetails.safety)}</div></div>
                                    </div>
                                    <div className="bg-purple-50 p-1.5 rounded">
                                        <span className="block text-[9px] text-purple-400 font-bold uppercase">Volumetric Test</span>
                                        <div className="font-bold text-purple-900 flex items-center gap-1"><IconEye size={10}/> {currentClassDetails.volumetric}</div>
                                    </div>
                                    <div className="bg-orange-50 p-1.5 rounded">
                                        <span className="block text-[9px] text-orange-400 font-bold uppercase">Surface Test</span>
                                        <div className="font-bold text-orange-900 flex items-center gap-1"><IconRefreshCw size={10}/> {currentClassDetails.surface}</div>
                                    </div>
                                </div>
                                <div className="text-[10px] text-slate-500 italic border-t border-blue-100 pt-1 mt-1">
                                    Inspect: {currentClassDetails.inspection} | {currentClassDetails.desc}
                                </div>
                            </div>
                        )}
 
                        <div className="flex flex-col gap-2 mt-3">
                           <div className="flex flex-wrap gap-2">
                                <span className="inline-flex items-center gap-2 text-xs md:text-sm bg-indigo-100 text-indigo-900 px-3 py-1.5 rounded-md font-bold border border-indigo-200 w-fit shadow-sm"><IconArrowDownToLine size={16}/> {isBoxSection ? 'Full Penetration' : currentWeld.penetration}</span>
                                {/* ACCESSIBILITY BADGE - MODIFIED LOGIC */}
                                <span className={`inline-flex items-center gap-2 text-xs md:text-sm px-3 py-1.5 rounded-md font-bold border w-fit shadow-sm ${inspectionAccess === 'no' ? "bg-red-50 text-red-700 border-red-200" : "bg-emerald-100 text-emerald-900 border-emerald-200"}`}>
                                    {inspectionAccess === 'no' ? <IconEyeOff size={16}/> : <IconScanEye size={16}/>}
                                    {inspectionAccess === 'no' ? "No Access for Inspection" : "Access Available for Inspection"}
                                </span>
                           </div>
                           
                           {/* CP B2 / CP C1 Special Note */}
                           {["CP B2", "CP C1"].includes(selectedClass) && (
                                <div className="mt-2 flex items-start gap-1 text-[10px] font-bold text-blue-800 bg-blue-100 px-2 py-1 rounded">
                                    <IconInfo size={12} className="mt-0.5 flex-shrink-0"/> 
                                    <span>Applicable for welds where volumetric NDT is not possible. Increased surface testing is required.</span>
                                </div>
                           )}
                        </div>
                        {hasBacking && (<div className="mt-3 flex items-center gap-1 text-[10px] font-bold text-amber-800 bg-amber-100 px-2 py-1 rounded inline-block"><IconAlertCircle size={12}/> Backing Bar Required (Thickness ≥ 16mm or Box)</div>)}
                        
                        {/* WARNING: INVALID PLATE THICKNESS */}
                        {invalidPlates.length > 0 && (
                            <div className="mt-2 flex items-center gap-1 text-[10px] font-bold text-red-800 bg-red-100 px-2 py-1 rounded inline-block">
                                <IconTriangleAlert size={12}/> Warning: Plate thickness ({invalidPlates.join(', ')}mm) not available in house.
                            </div>
                        )}
                    </div>
                </div>
                <div className="border rounded bg-white p-3 shadow-sm mb-4">
                     <h4 className="text-xs font-bold text-gray-500 mb-2 border-b">Fig 1: Cross Section</h4>
                     <div className="h-64 flex items-center justify-center bg-gray-50"><ResultRenderer data={activeWeld} inputs={{ angle, gap, rootFace, jointType, cornerOption, t3, t4 }} isBoxSection={isBoxSection} hasBacking={hasBacking} hasSealingRun={hasSealingRun} hasAdditionalFillet={hasAdditionalFillet} weldSide={weldSide} selectedClass={selectedClass}/></div>
                </div>
                <div className="border rounded bg-white p-3 shadow-sm mb-4">
                     <h4 className="text-xs font-bold text-gray-500 mb-2 border-b">Fig 2: Symbol</h4>
                     <div className="h-32 flex items-center justify-center bg-gray-50"><SymbolRenderer data={activeWeld} inputs={{ angle, gap, rootFace, jointType, cornerOption, t3, t4 }} isBoxSection={isBoxSection} hasBacking={hasBacking} weldLength={weldLength} isWeldAllAround={isWeldAllAround} t1={t1} t2={t2} hasSealingRun={hasSealingRun} hasAdditionalFillet={hasAdditionalFillet} selectedClass={selectedClass} weldSide={weldSide}/></div>
                </div>
                <div className="mt-4 text-[10px] text-gray-400 pt-2 border-t">*Ref: EN 15085-3 / ISO 9692. Always verify with approved WPS.</div>
            </div>
        </div>
      </div>
    </div>
  )
}