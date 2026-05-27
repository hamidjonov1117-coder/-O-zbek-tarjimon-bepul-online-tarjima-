import { useState, useCallback } from "react";

const LANGUAGES = [
  { code: "uz", label: "🇺🇿 O'zbek" },
  { code: "en", label: "🇬🇧 English" },
  { code: "ru", label: "🇷🇺 Русский" },
  { code: "ar", label: "🇸🇦 العربية" },
  { code: "de", label: "🇩🇪 Deutsch" },
  { code: "fr", label: "🇫🇷 Français" },
  { code: "zh", label: "🇨🇳 中文" },
  { code: "ja", label: "🇯🇵 日本語" },
  { code: "tr", label: "🇹🇷 Türkçe" },
  { code: "ko", label: "🇰🇷 한국어" },
];

const LANG_NAMES = {
  uz: "O'zbek",
  en: "English",
  ru: "Russian",
  ar: "Arabic",
  de: "German",
  fr: "French",
  zh: "Chinese",
  ja: "Japanese",
  tr: "Turkish",
  ko: "Korean",
};

export default function TarjimonApp() {
  const [inputText, setInputText] = useState("");
  const [outputText, setOutputText] = useState("");
  const [fromLang, setFromLang] = useState("uz");
  const [toLang, setToLang] = useState("en");
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState("");
  const [copied, setCopied] = useState(false);
  const [charCount, setCharCount] = useState(0);

  const handleSwap = () => {
    setFromLang(toLang);
    setToLang(fromLang);
    setInputText(outputText);
    setOutputText(inputText);
    setCharCount(outputText.length);
  };

  const handleTranslate = useCallback(async () => {
    if (!inputText.trim()) return;
    setLoading(true);
    setError("");
    setOutputText("");

    try {
      const response = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-20250514",
          max_tokens: 1000,
          messages: [
            {
              role: "user",
              content: `Translate the following text from ${LANG_NAMES[fromLang]} to ${LANG_NAMES[toLang]}. Return ONLY the translated text, nothing else, no explanations, no quotation marks.\n\nText: ${inputText}`,
            },
          ],
        }),
      });

      const data = await response.json();
      const result = data.content?.map((b) => b.text || "").join("") || "";
      setOutputText(result.trim());
    } catch (err) {
      setError("Tarjima qilishda xatolik yuz berdi. Qayta urinib ko'ring.");
    } finally {
      setLoading(false);
    }
  }, [inputText, fromLang, toLang]);

  const handleCopy = () => {
    if (!outputText) return;
    navigator.clipboard.writeText(outputText);
    setCopied(true);
    setTimeout(() => setCopied(false), 2000);
  };

  const handleInputChange = (e) => {
    const val = e.target.value;
    if (val.length <= 2000) {
      setInputText(val);
      setCharCount(val.length);
    }
  };

  return (
    <div style={styles.root}>
      {/* Background blobs */}
      <div style={styles.blob1} />
      <div style={styles.blob2} />
      <div style={styles.blob3} />

      <div style={styles.container}>
        {/* Header */}
        <div style={styles.header}>
          <div style={styles.logoWrap}>
            <span style={styles.logoIcon}>⬡</span>
            <span style={styles.logoText}>Tarjimon</span>
          </div>
          <p style={styles.subtitle}>Sun'iy intellekt asosida tarjima</p>
        </div>

        {/* Lang selector row */}
        <div style={styles.langRow}>
          <select
            value={fromLang}
            onChange={(e) => setFromLang(e.target.value)}
            style={styles.select}
          >
            {LANGUAGES.map((l) => (
              <option key={l.code} value={l.code}>{l.label}</option>
            ))}
          </select>

          <button onClick={handleSwap} style={styles.swapBtn} title="Almashtirish">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2.2" strokeLinecap="round" strokeLinejoin="round">
              <path d="M7 16V4m0 0L3 8m4-4l4 4" />
              <path d="M17 8v12m0 0l4-4m-4 4l-4-4" />
            </svg>
          </button>

          <select
            value={toLang}
            onChange={(e) => setToLang(e.target.value)}
            style={styles.select}
          >
            {LANGUAGES.map((l) => (
              <option key={l.code} value={l.code}>{l.label}</option>
            ))}
          </select>
        </div>

        {/* Main panels */}
        <div style={styles.panels}>
          {/* Input */}
          <div style={styles.panel}>
            <div style={styles.panelHeader}>
              <span style={styles.panelLabel}>{LANGUAGES.find(l => l.code === fromLang)?.label}</span>
              <button
                onClick={() => { setInputText(""); setCharCount(0); setOutputText(""); }}
                style={styles.clearBtn}
                disabled={!inputText}
              >✕ Tozalash</button>
            </div>
            <textarea
              value={inputText}
              onChange={handleInputChange}
              placeholder="Matn kiriting..."
              style={styles.textarea}
              onKeyDown={(e) => {
                if (e.key === "Enter" && e.ctrlKey) handleTranslate();
              }}
            />
            <div style={styles.panelFooter}>
              <span style={{ ...styles.charCount, color: charCount > 1800 ? "#f87171" : "#94a3b8" }}>
                {charCount} / 2000
              </span>
              <button
                onClick={handleTranslate}
                disabled={loading || !inputText.trim()}
                style={{
                  ...styles.translateBtn,
                  opacity: loading || !inputText.trim() ? 0.5 : 1,
                  cursor: loading || !inputText.trim() ? "not-allowed" : "pointer",
                }}
              >
                {loading ? (
                  <span style={styles.spinner} />
                ) : (
                  <>
                    <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2.5" strokeLinecap="round" strokeLinejoin="round" style={{ marginRight: 6 }}>
                      <circle cx="12" cy="12" r="10" />
                      <path d="M12 8v4l3 3" />
                    </svg>
                    Tarjima qilish
                  </>
                )}
              </button>
            </div>
          </div>

          {/* Output */}
          <div style={{ ...styles.panel, ...styles.outputPanel }}>
            <div style={styles.panelHeader}>
              <span style={styles.panelLabel}>{LANGUAGES.find(l => l.code === toLang)?.label}</span>
              <button onClick={handleCopy} style={styles.copyBtn} disabled={!outputText}>
                {copied ? "✓ Nusxalandi!" : "⎘ Nusxalash"}
              </button>
            </div>
            <div style={styles.outputBox}>
              {loading ? (
                <div style={styles.loadingWrap}>
                  <div style={styles.dots}>
                    <span style={{ ...styles.dot, animationDelay: "0s" }} />
                    <span style={{ ...styles.dot, animationDelay: "0.2s" }} />
                    <span style={{ ...styles.dot, animationDelay: "0.4s" }} />
                  </div>
                  <p style={styles.loadingText}>Tarjima qilinmoqda...</p>
                </div>
              ) : error ? (
                <p style={styles.errorText}>{error}</p>
              ) : outputText ? (
                <p style={styles.outputText}>{outputText}</p>
              ) : (
                <p style={styles.placeholder}>Tarjima bu yerda ko'rinadi...</p>
              )}
            </div>
          </div>
        </div>

        {/* Hint */}
        <p style={styles.hint}>
          💡 <kbd style={styles.kbd}>Ctrl</kbd> + <kbd style={styles.kbd}>Enter</kbd> — tez tarjima
        </p>
      </div>

      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Sora:wght@300;400;500;600;700&family=Noto+Serif:ital@0;1&display=swap');
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { background: #080c14; }
        select option { background: #111827; color: #e2e8f0; }

        @keyframes bounce {
          0%, 80%, 100% { transform: translateY(0); opacity: 0.4; }
          40% { transform: translateY(-8px); opacity: 1; }
        }
        @keyframes spin {
          to { transform: rotate(360deg); }
        }
        @keyframes float1 {
          0%, 100% { transform: translate(0,0) scale(1); }
          50% { transform: translate(30px, -20px) scale(1.05); }
        }
        @keyframes float2 {
          0%, 100% { transform: translate(0,0) scale(1); }
          50% { transform: translate(-20px, 30px) scale(1.08); }
        }
        @keyframes float3 {
          0%, 100% { transform: translate(0,0) scale(1); }
          50% { transform: translate(15px, 15px) scale(0.96); }
        }
      `}</style>
    </div>
  );
}

const styles = {
  root: {
    minHeight: "100vh",
    background: "#080c14",
    fontFamily: "'Sora', sans-serif",
    position: "relative",
    overflow: "hidden",
    display: "flex",
    alignItems: "center",
    justifyContent: "center",
    padding: "24px 16px",
  },
  blob1: {
    position: "fixed", top: "-120px", left: "-100px",
    width: "500px", height: "500px", borderRadius: "50%",
    background: "radial-gradient(circle, rgba(99,102,241,0.25) 0%, transparent 70%)",
    filter: "blur(40px)", animation: "float1 8s ease-in-out infinite", pointerEvents: "none",
  },
  blob2: {
    position: "fixed", bottom: "-100px", right: "-80px",
    width: "450px", height: "450px", borderRadius: "50%",
    background: "radial-gradient(circle, rgba(20,184,166,0.2) 0%, transparent 70%)",
    filter: "blur(40px)", animation: "float2 10s ease-in-out infinite", pointerEvents: "none",
  },
  blob3: {
    position: "fixed", top: "40%", left: "50%",
    width: "300px", height: "300px", borderRadius: "50%",
    background: "radial-gradient(circle, rgba(236,72,153,0.1) 0%, transparent 70%)",
    filter: "blur(50px)", animation: "float3 12s ease-in-out infinite", pointerEvents: "none",
  },
  container: {
    width: "100%", maxWidth: "880px", position: "relative", zIndex: 1,
  },
  header: {
    textAlign: "center", marginBottom: "36px",
  },
  logoWrap: {
    display: "inline-flex", alignItems: "center", gap: "10px", marginBottom: "8px",
  },
  logoIcon: {
    fontSize: "32px", color: "#818cf8",
    textShadow: "0 0 20px rgba(129,140,248,0.8)",
  },
  logoText: {
    fontSize: "32px", fontWeight: "700", letterSpacing: "-0.5px",
    background: "linear-gradient(135deg, #a5b4fc 0%, #6ee7b7 100%)",
    WebkitBackgroundClip: "text", WebkitTextFillColor: "transparent",
  },
  subtitle: {
    color: "#64748b", fontSize: "14px", letterSpacing: "0.5px",
  },
  langRow: {
    display: "flex", alignItems: "center", justifyContent: "center",
    gap: "12px", marginBottom: "20px",
  },
  select: {
    flex: 1, maxWidth: "220px",
    background: "rgba(255,255,255,0.05)",
    border: "1px solid rgba(255,255,255,0.1)",
    color: "#e2e8f0", borderRadius: "12px",
    padding: "10px 14px", fontSize: "14px",
    fontFamily: "'Sora', sans-serif",
    cursor: "pointer", outline: "none",
    backdropFilter: "blur(10px)",
    transition: "border-color 0.2s",
  },
  swapBtn: {
    background: "rgba(99,102,241,0.15)",
    border: "1px solid rgba(99,102,241,0.3)",
    color: "#a5b4fc", borderRadius: "50%",
    width: "42px", height: "42px",
    display: "flex", alignItems: "center", justifyContent: "center",
    cursor: "pointer", transition: "all 0.2s", flexShrink: 0,
  },
  panels: {
    display: "grid", gridTemplateColumns: "1fr 1fr", gap: "16px",
    "@media (max-width: 600px)": { gridTemplateColumns: "1fr" },
  },
  panel: {
    background: "rgba(255,255,255,0.03)",
    border: "1px solid rgba(255,255,255,0.08)",
    borderRadius: "20px", overflow: "hidden",
    backdropFilter: "blur(20px)",
    display: "flex", flexDirection: "column",
    minHeight: "280px",
  },
  outputPanel: {
    background: "rgba(99,102,241,0.04)",
    border: "1px solid rgba(99,102,241,0.15)",
  },
  panelHeader: {
    display: "flex", alignItems: "center", justifyContent: "space-between",
    padding: "14px 18px", borderBottom: "1px solid rgba(255,255,255,0.06)",
  },
  panelLabel: {
    fontSize: "13px", fontWeight: "600", color: "#94a3b8", letterSpacing: "0.3px",
  },
  clearBtn: {
    background: "none", border: "none", color: "#475569",
    fontSize: "12px", cursor: "pointer", padding: "2px 6px", borderRadius: "6px",
    transition: "color 0.2s", fontFamily: "'Sora', sans-serif",
  },
  copyBtn: {
    background: "rgba(99,102,241,0.1)",
    border: "1px solid rgba(99,102,241,0.2)",
    color: "#a5b4fc", fontSize: "12px",
    cursor: "pointer", padding: "4px 10px", borderRadius: "8px",
    transition: "all 0.2s", fontFamily: "'Sora', sans-serif",
  },
  textarea: {
    flex: 1, background: "none", border: "none", outline: "none",
    color: "#f1f5f9", fontSize: "15.5px", lineHeight: "1.7",
    fontFamily: "'Noto Serif', serif",
    padding: "18px", resize: "none", minHeight: "160px",
  },
  panelFooter: {
    display: "flex", alignItems: "center", justifyContent: "space-between",
    padding: "12px 18px", borderTop: "1px solid rgba(255,255,255,0.05)",
  },
  charCount: {
    fontSize: "12px", transition: "color 0.2s",
  },
  translateBtn: {
    background: "linear-gradient(135deg, #6366f1, #4f46e5)",
    border: "none", color: "#fff",
    padding: "9px 20px", borderRadius: "12px",
    fontSize: "13px", fontWeight: "600",
    fontFamily: "'Sora', sans-serif",
    display: "flex", alignItems: "center",
    boxShadow: "0 4px 20px rgba(99,102,241,0.4)",
    transition: "all 0.2s",
  },
  spinner: {
    width: "16px", height: "16px",
    border: "2px solid rgba(255,255,255,0.3)",
    borderTopColor: "#fff", borderRadius: "50%",
    animation: "spin 0.7s linear infinite",
    display: "inline-block",
  },
  outputBox: {
    flex: 1, padding: "18px", overflowY: "auto",
    display: "flex", alignItems: "flex-start",
  },
  loadingWrap: {
    display: "flex", flexDirection: "column",
    alignItems: "center", justifyContent: "center",
    width: "100%", gap: "12px", paddingTop: "30px",
  },
  dots: {
    display: "flex", gap: "8px",
  },
  dot: {
    width: "8px", height: "8px", borderRadius: "50%",
    background: "#818cf8",
    display: "inline-block",
    animation: "bounce 1.2s ease-in-out infinite",
  },
  loadingText: {
    color: "#64748b", fontSize: "13px",
  },
  outputText: {
    color: "#e2e8f0", fontSize: "15.5px", lineHeight: "1.7",
    fontFamily: "'Noto Serif', serif",
  },
  placeholder: {
    color: "#334155", fontSize: "14px", fontStyle: "italic",
    marginTop: "8px",
  },
  errorText: {
    color: "#f87171", fontSize: "14px",
  },
  hint: {
    textAlign: "center", marginTop: "20px",
    color: "#334155", fontSize: "12px",
  },
  kbd: {
    background: "rgba(255,255,255,0.06)",
    border: "1px solid rgba(255,255,255,0.1)",
    borderRadius: "5px", padding: "1px 6px",
    fontSize: "11px", color: "#64748b",
    fontFamily: "monospace",
  },
};

