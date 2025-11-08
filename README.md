# Responsive-Calculator-
import React, { useEffect, useState, useRef } from "react";
import { motion } from "framer-motion";
import { Backspace, DivideCircle, PlusCircle, MinusCircle, Percent, XCircle } from "lucide-react";

// ResponsiveCalculator - single-file React component
// TailwindCSS utility classes assumed. Exports default component.

export default function ResponsiveCalculator() {
  const [display, setDisplay] = useState("0");
  const [prevValue, setPrevValue] = useState(null);
  const [operation, setOperation] = useState(null);
  const [overwrite, setOverwrite] = useState(true);
  const displayRef = useRef(null);

  useEffect(() => {
    function handleKey(e) {
      const key = e.key;
      if ((/^[0-9]$/).test(key)) onDigit(key);
      if (key === ".") onDot();
      if (key === "+" || key === "-" || key === "*" || key === "/") onOperator(key);
      if (key === "Enter" || key === "=") onEquals();
      if (key === "Backspace") onBackspace();
      if (key === "Escape") onClear();
    }
    window.addEventListener("keydown", handleKey);
    return () => window.removeEventListener("keydown", handleKey);
  }, [display, prevValue, operation, overwrite]);

  // Helpers
  const formatNumber = (value) => {
    if (value === null || value === undefined) return "";
    // keep scientific/long numbers readable
    const num = Number(value);
    if (!isFinite(num)) return String(num);
    const [intPart, decPart] = String(value).split(".");
    const formattedInt = Number(intPart).toLocaleString();
    return decPart ? `${formattedInt}.${decPart}` : formattedInt;
  };

  const setToDisplay = (val) => {
    setDisplay(String(val));
    setOverwrite(false);
  };

  // Actions
  function onDigit(d) {
    if (overwrite) {
      setDisplay(d === "0" ? "0" : d);
      setOverwrite(false);
      return;
    }

    // prevent leading zeros
    if (display === "0") {
      setDisplay(d);
    } else {
      setDisplay((prev) => (prev + d).slice(0, 30));
    }
  }

  function onDot() {
    if (overwrite) {
      setDisplay("0.");
      setOverwrite(false);
      return;
    }
    if (!display.includes(".")) setDisplay((prev) => prev + ".");
  }

  function onClear() {
    setDisplay("0");
    setPrevValue(null);
    setOperation(null);
    setOverwrite(true);
  }

  function onBackspace() {
    if (overwrite) {
      setDisplay("0");
      setOverwrite(true);
      return;
    }
    setDisplay((prev) => {
      if (prev.length === 1) return "0";
      return prev.slice(0, -1);
    });
  }

  function onToggleSign() {
    setDisplay((prev) => (prev.startsWith("-") ? prev.slice(1) : `-${prev}`));
  }

  function onPercent() {
    setDisplay((prev) => String(Number(prev) / 100));
    setOverwrite(true);
  }

  function performOperation(a, b, op) {
    const x = Number(a);
    const y = Number(b);
    if (op === "+") return x + y;
    if (op === "-") return x - y;
    if (op === "*") return x * y;
    if (op === "/") return y === 0 ? "Error" : x / y;
    return y;
  }

  function onOperator(opKey) {
    const op = opKey === "*" ? "*" : opKey === "/" ? "/" : opKey; // keep symbol
    if (prevValue === null) {
      setPrevValue(display);
      setOperation(op);
      setOverwrite(true);
    } else {
      const result = performOperation(prevValue, display, operation);
      setPrevValue(String(result));
      setDisplay(String(result));
      setOperation(op);
      setOverwrite(true);
    }
  }

  function onEquals() {
    if (operation == null || prevValue == null) return;
    const result = performOperation(prevValue, display, operation);
    setDisplay(String(result));
    setPrevValue(null);
    setOperation(null);
    setOverwrite(true);
  }

  // Button config for grid rendering
  const buttons = [
    { label: "AC", action: onClear, className: "col-span-2 text-sm" },
    { label: <Backspace size={18} />, action: onBackspace },
    { label: <DivideCircle size={18} />, action: () => onOperator("/") },

    { label: "7", action: () => onDigit("7") },
    { label: "8", action: () => onDigit("8") },
    { label: "9", action: () => onDigit("9") },
    { label: <XCircle size={18} />, action: () => onOperator("*") },

    { label: "4", action: () => onDigit("4") },
    { label: "5", action: () => onDigit("5") },
    { label: "6", action: () => onDigit("6") },
    { label: <MinusCircle size={18} />, action: () => onOperator("-") },

    { label: "1", action: () => onDigit("1") },
    { label: "2", action: () => onDigit("2") },
    { label: "3", action: () => onDigit("3") },
    { label: <PlusCircle size={18} />, action: () => onOperator("+") },

    { label: "+/−", action: onToggleSign },
    { label: "0", action: () => onDigit("0") },
    { label: ".", action: onDot },
    { label: "=", action: onEquals, className: "bg-gradient-to-br from-indigo-500 to-sky-400 text-white" },
  ];

  return (
    <div className="min-h-screen flex items-center justify-center p-4 bg-gradient-to-br from-gray-50 to-white">
      <motion.div
        initial={{ opacity: 0, scale: 0.98 }}
        animate={{ opacity: 1, scale: 1 }}
        transition={{ duration: 0.18 }}
        className="w-full max-w-md"
      >
        <div className="shadow-lg rounded-2xl p-4 bg-white">
          <div className="flex items-center justify-between mb-4">
            <div>
              <h1 className="text-xl font-semibold">Calculator</h1>
              <p className="text-xs text-gray-500">Responsive • Keyboard friendly • Mobile-first</p>
            </div>
            <div className="text-sm text-gray-600">v1.0</div>
          </div>

          <div className="rounded-xl bg-gray-900 text-white p-4 mb-4">
            <div className="text-right text-xs text-gray-400 truncate">{operation ? `${formatNumber(prevValue)} ${operation}` : " "}</div>
            <div
              ref={displayRef}
              className="text-right font-mono text-3xl sm:text-4xl md:text-5xl break-words"
              aria-live="polite"
            >
              {formatNumber(display)}
            </div>
          </div>

          <div className="grid grid-cols-4 gap-3">
            {buttons.map((btn, idx) => (
              <motion.button
                key={idx}
                onClick={btn.action}
                whileTap={{ scale: 0.96 }}
                className={`select-none focus:outline-none focus:ring-2 focus:ring-indigo-300 transition rounded-xl py-3 px-3 flex items-center justify-center text-lg font-medium shadow-sm ${
                  btn.className || "bg-gray-100"
                }`}
                aria-label={typeof btn.label === "string" ? btn.label : `button-${idx}`}
              >
                {btn.label}
              </motion.button>
            ))}
          </div>

          <div className="mt-4 text-xs text-gray-500">Tip: use keyboard numbers, + - * /, Enter for equals, Esc to clear.</div>
        </div>
      </motion.div>
    </div>
  );
}
