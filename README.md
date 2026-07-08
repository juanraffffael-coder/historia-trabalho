import { useState, useRef, useEffect } from "react";

// ─── Tokens ──────────────────────────────────────────────────────────────────
const T = {
  slateDeep: "#1B222B",
  slate: "#2E3A46",
  cream: "#F4EEE2",
  cream2: "#EFE7D8",
  terra: "#B8563D",
  terraS: "#C97A5F",
  graphite: "#241D1A",
  inkSoft: "#4A4038",
  line: "rgba(184,86,61,0.28)",
};

const serif = "'Libre Caslon Text', serif";
const sans = "'Inter', sans-serif";
const mono = "'JetBrains Mono', monospace";

// ─── Reveal on scroll ────────────────────────────────────────────────────────
function Reveal({ children, delay = 0 }) {
  const ref = useRef(null);
  const [visible, setVisible] = useState(false);

  useEffect(() => {
    const el = ref.current;
    if (!el) return;
    const obs = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setVisible(true);
          obs.disconnect();
        }
      },
      { threshold: 0.15 }
    );
    obs.observe(el);
    return () => obs.disconnect();
  }, []);

  return (
    <div
      ref={ref}
      className="reveal"
      style={{
        opacity: visible ? 1 : 0,
        transform: visible ? "translateY(0)" : "translateY(20px)",
        transition: `opacity 0.7s ease ${delay}s, transform 0.7s cubic-bezier(.2,.8,.2,1) ${delay}s`,
      }}
    >
      {children}
    </div>
  );
}

// ─── Nav ─────────────────────────────────────────────────────────────────────
function Nav() {
  const [scrolled, setScrolled] = useState(false);
  useEffect(() => {
    const h = () => setScrolled(window.scrollY > 10);
    window.addEventListener("scroll", h);
    return () => window.removeEventListener("scroll", h);
  }, []);

  return (
    <nav style={{
      position: "sticky", top: 0, zIndex: 50,
      background: scrolled ? "rgba(244,238,226,0.92)" : "rgba(244,238,226,0.86)",
      backdropFilter: "blur(10px)",
      borderBottom: `1px solid ${T.line}`,
      transition: "background 0.3s",
    }}>
      <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", padding: "18px 30px", maxWidth: 1080, margin: "0 auto" }}>
        <div style={{ fontFamily: serif, fontSize: "1.5rem", color: T.graphite }}>
          Ne<span style={{ color: T.terra, fontStyle: "italic" }}>x</span>o
        </div>
        <a
          href="https://wa.me/5511958642044?text=Ol%C3%A1%2C%20quero%20saber%20mais%20sobre%20o%20Nexo"
          target="_blank" rel="noopener"
          style={{
            fontFamily: sans, fontWeight: 600, fontSize: "0.88rem",
            background: T.slateDeep, color: T.cream,
            padding: "11px 22px", borderRadius: 100, textDecoration: "none",
            transition: "background 0.2s, transform 0.2s",
          }}
          onMouseEnter={e => { e.currentTarget.style.background = T.terra; e.currentTarget.style.transform = "translateY(-1px)"; }}
          onMouseLeave={e => { e.currentTarget.style.background = T.slateDeep; e.currentTarget.style.transform = "none"; }}
        >
          Falar no WhatsApp
        </a>
      </div>
    </nav>
  );
}

// ─── Chat Demo ────────────────────────────────────────────────────────────────
const CHAT_INITIAL = [
  { dir: "out", text: "Oi! Você está falando com a Barbearia Nexo 💈 — a maior barbearia do bairro. Funcionamos das 9h às 20h, de segunda a sábado. Corte R$45, corte + barba R$65.", time: "21:46" },
  { dir: "in",  text: "Oi, vocês abrem sábado?", time: "21:47" },
  { dir: "out", text: "Abrimos sim, das 9h às 13h. Quer já deixar um horário reservado?", time: "21:47" },
  { dir: "in",  text: "Quero, pode ser 10h", time: "21:48" },
  { dir: "out", text: "Fechado — 10h de sábado reservado no seu nome. Te aviso um dia antes ✓", time: "21:48" },
];

const BOT_REPLIES = {
  default: "Posso te ajudar com agendamento, preços ou informações. O que deseja?",
  preco: "Corte: R$45. Corte + barba: R$65. Coloração: R$80. Quer agendar?",
  agendar: "Claro! Qual horário você prefere? Temos disponibilidade amanhã às 10h, 14h e 16h.",
  horario: "Funcionamos de segunda a sábado, das 9h às 20h. Domingo fechado.",
  valor: "Corte: R$45. Corte + barba: R$65. Coloração: R$80. Quer agendar?",
};

function matchReply(input) {
  const q = input.toLowerCase();
  if (q.includes("prec") || q.includes("valor") || q.includes("cust") || q.includes("quanto")) return BOT_REPLIES.preco;
  if (q.includes("hor") || q.includes("funciona") || q.includes("abre")) return BOT_REPLIES.horario;
  if (q.includes("agenda") || q.includes("marcar") || q.includes("reserv")) return BOT_REPLIES.agendar;
  return BOT_REPLIES.default;
}

function TypingDots() {
  return (
    <div style={{ alignSelf: "flex-end", maxWidth: "80%" }}>
      <div style={{
        padding: "11px 15px", borderRadius: 12, borderTopRightRadius: 2,
        background: "#144D37", display: "flex", gap: 4, alignItems: "center",
      }}>
        {[0, 1, 2].map(i => (
          <span key={i} style={{
            width: 5, height: 5, borderRadius: "50%", background: "rgba(233,237,239,0.8)",
            animation: `typingBlink 1.2s infinite ${i * 0.18}s`,
          }} />
        ))}
      </div>
    </div>
  );
}

function ChatDemo() {
  const [messages, setMessages] = useState(CHAT_INITIAL);
  const [input, setInput] = useState("");
  const [typing, setTyping] = useState(false);
  const chatRef = useRef(null);

  const send = () => {
    if (!input.trim()) return;
    const now = new Date();
    const time = `${now.getHours()}:${String(now.getMinutes()).padStart(2, "0")}`;
    const userMsg = { dir: "in", text: input, time };
    setMessages(m => [...m, userMsg]);
    setInput("");
    setTyping(true);
    setTimeout(() => {
      setTyping(false);
      const botMsg = { dir: "out", text: matchReply(userMsg.text), time };
      setMessages(m => [...m, botMsg]);
    }, 900);
  };

  useEffect(() => {
    if (chatRef.current) chatRef.current.scrollTop = chatRef.current.scrollHeight;
  }, [messages, typing]);

  return (
    <div style={{
      background: "#101720", borderRadius: 26, padding: 10,
      boxShadow: "0 30px 70px rgba(0,0,0,0.4)",
      maxWidth: 320, margin: "0 auto",
      border: "1px solid rgba(244,238,226,0.08)",
    }}>
      <div style={{ background: "#0B141A", borderRadius: 18, overflow: "hidden" }}>
        <div style={{ background: "#3E5C4A", color: "#fff", padding: "14px 16px", fontSize: 13, fontWeight: 600, display: "flex", alignItems: "center", gap: 10, fontFamily: sans }}>
          <div style={{ width: 8, height: 8, borderRadius: "50%", background: "#8FD4A8" }} />
          Nexo · online
        </div>
        <div ref={chatRef} style={{
          padding: "18px 14px", display: "flex", flexDirection: "column", gap: 10,
          minHeight: 260, maxHeight: 320, overflowY: "auto", background: "#0B141A",
          scrollbarWidth: "none",
        }}>
          {messages.map((m, i) => (
            <div key={i} style={{ alignSelf: m.dir === "out" ? "flex-start" : "flex-end", maxWidth: "80%" }}>
              <div style={{
                padding: "10px 13px", borderRadius: 12, fontSize: 12.5, lineHeight: 1.5,
                fontFamily: sans,
                background: m.dir === "out" ? "#1F2C34" : "#144D37",
                color: "#E9EDEF",
                borderTopLeftRadius: m.dir === "out" ? 2 : 12,
                borderTopRightRadius: m.dir === "in" ? 2 : 12,
              }}>
                {m.text}
                <div style={{ marginTop: 4, fontSize: 9.5, color: "rgba(233,237,239,0.45)", textAlign: "right" }}>{m.time}</div>
              </div>
            </div>
          ))}
          {typing && <TypingDots />}
        </div>
        <div style={{ display: "flex", gap: 8, padding: "10px 12px", background: "#101720", borderTop: "1px solid rgba(244,238,226,0.06)" }}>
          <input
            value={input}
            onChange={e => setInput(e.target.value)}
            onKeyDown={e => e.key === "Enter" && send()}
            placeholder="Pergunte algo, ex: qual o valor?"
            maxLength={120}
            style={{
              flex: 1, background: "#1F2C34", border: "1px solid rgba(244,238,226,0.08)",
              borderRadius: 100, padding: "9px 14px", fontFamily: sans,
              fontSize: 12.5, color: "#E9EDEF", outline: "none",
            }}
          />
          <button onClick={send} style={{
            width: 34, height: 34, flexShrink: 0, borderRadius: "50%", border: "none",
            background: "#3E5C4A", color: "#fff", fontSize: 14, cursor: "pointer",
            display: "flex", alignItems: "center", justifyContent: "center",
            transition: "transform 0.15s",
          }}
          onMouseEnter={e => (e.currentTarget.style.transform = "scale(1.08)")}
          onMouseLeave={e => (e.currentTarget.style.transform = "none")}
          >➤</button>
        </div>
      </div>
      <div style={{ fontSize: 10, color: "rgba(244,238,226,0.4)", textAlign: "center", marginTop: 10, fontFamily: mono }}>
        experimente digitar uma pergunta ↑
      </div>
    </div>
  );
}

// ─── Hero ─────────────────────────────────────────────────────────────────────
function Hero() {
  return (
    <section style={{
      background: T.slateDeep,
      color: T.cream,
      padding: "88px 0 100px",
      position: "relative",
      overflow: "hidden",
    }}>
      {/* Top terracotta line */}
      <div style={{ position: "absolute", top: 0, left: 0, width: "100%", height: 3, background: T.terra }} />
      {/* Radial accent */}
      <div style={{
        position: "absolute", right: -160, top: -160, width: 440, height: 440, borderRadius: "50%",
        background: "radial-gradient(circle, rgba(184,86,61,0.3), transparent 70%)",
        pointerEvents: "none",
      }} />

      <div style={{ maxWidth: 1080, margin: "0 auto", padding: "0 30px", position: "relative", zIndex: 1 }}>
        <div style={{ display: "grid", gridTemplateColumns: "1.1fr 0.9fr", gap: 60, alignItems: "center" }}
          className="hero-grid">
          <div>
            <div style={{ fontFamily: mono, fontSize: 11, letterSpacing: "0.18em", textTransform: "uppercase", color: T.terraS, marginBottom: 16, display: "flex", alignItems: "center", gap: 10 }}>
              <span style={{ width: 14, height: 1, background: T.terraS, display: "inline-block" }} />
              Automação com IA pra PMEs
            </div>
            <h1 style={{ fontFamily: serif, fontWeight: 400, fontSize: "clamp(2.1rem, 4.2vw, 3.2rem)", lineHeight: 1.18, letterSpacing: "-0.01em" }}>
              Coloque a IA pra trabalhar no seu negócio — sem precisar entender de tecnologia.
            </h1>
            <p style={{ marginTop: 22, fontSize: "1.05rem", color: "rgba(244,238,226,0.75)", maxWidth: 440, fontFamily: sans }}>
              O Nexo responde no WhatsApp, organiza seus dados num painel simples e cria seu conteúdo. Você continua tocando o negócio, só que com menos trabalho manual.
            </p>
            <div style={{ display: "flex", gap: 14, marginTop: 34, flexWrap: "wrap" }}>
              <a
                href="https://wa.me/5511958642044?text=Ol%C3%A1%2C%20quero%20contratar%20o%20Nexo"
                target="_blank" rel="noopener"
                style={{
                  fontFamily: sans, fontWeight: 600, fontSize: "0.95rem",
                  background: T.terra, color: T.cream, padding: "15px 28px",
                  borderRadius: 100, textDecoration: "none", display: "inline-block",
                  transition: "background 0.2s, transform 0.2s",
                }}
                onMouseEnter={e => { e.currentTarget.style.background = "#9F4A34"; e.currentTarget.style.transform = "translateY(-2px)"; }}
                onMouseLeave={e => { e.currentTarget.style.background = T.terra; e.currentTarget.style.transform = "none"; }}
              >
                Quero contratar
              </a>
              <button
                onClick={() => document.getElementById("como-funciona")?.scrollIntoView({ behavior: "smooth" })}
                style={{
                  fontFamily: sans, fontWeight: 600, fontSize: "0.95rem",
                  color: T.cream, padding: "15px 24px", borderRadius: 100,
                  border: "1px solid rgba(244,238,226,0.35)", background: "transparent",
                  cursor: "pointer", transition: "border-color 0.2s",
                }}
                onMouseEnter={e => (e.currentTarget.style.borderColor = T.cream)}
                onMouseLeave={e => (e.currentTarget.style.borderColor = "rgba(244,238,226,0.35)")}
              >
                Ver como funciona
              </button>
            </div>
            <div style={{ display: "flex", gap: 26, marginTop: 38, flexWrap: "wrap", fontFamily: mono, fontSize: 11.5, color: "rgba(244,238,226,0.55)" }}>
              {["Ativa em 1 semana", "Sem fidelidade", "Suporte incluso 15 dias"].map(t => (
                <span key={t} style={{ display: "flex", alignItems: "center", gap: 7 }}>
                  <span style={{ color: T.terraS }}>✓</span> {t}
                </span>
              ))}
            </div>
          </div>
          <div className="hero-phone">
            <ChatDemo />
          </div>
        </div>
      </div>
    </section>
  );
}

// ─── Dashboard Demo ───────────────────────────────────────────────────────────
const CHART_VALS = [28, 41, 35, 58, 72, 89];
const CHART_MONTHS = ["Jan", "Fev", "Mar", "Abr", "Mai", "Jun"];

function DashboardEmbed() {
  const [chartMode, setChartMode] = useState("bar");
  const maxVal = Math.max(...CHART_VALS);

  return (
    <div style={{ background: "#1B222B", borderRadius: 10, overflow: "hidden", border: "1px solid rgba(244,238,226,0.07)", fontFamily: sans }}>
      {/* Mini topbar */}
      <div style={{ background: "#232C37", borderBottom: "1px solid rgba(244,238,226,0.07)", padding: "10px 16px", display: "flex", alignItems: "center", gap: 10 }}>
        <div style={{ width: 8, height: 8, borderRadius: "50%", background: "#4ade80", boxShadow: "0 0 6px #4ade80" }} />
        <span style={{ fontSize: 12, fontWeight: 700, color: "#F4EEE2" }}>Dashboard · Nexo</span>
        <span style={{ marginLeft: "auto", fontSize: 10, color: "rgba(244,238,226,0.35)", fontFamily: mono }}>v3.0 · ao vivo</span>
      </div>
      <div style={{ padding: 16 }}>
        {/* KPI row */}
        <div style={{ display: "grid", gridTemplateColumns: "repeat(4,1fr)", gap: 10, marginBottom: 14 }}>
          {[
            { label: "Receita", val: "R$ 11.940", sub: "↑ 23%", subColor: "#4ade80" },
            { label: "Clientes", val: "30", sub: "↑ 18 novos", subColor: "#4ade80" },
            { label: "Tarefas", val: "7", sub: "4 pendentes", subColor: "#C97A5F" },
            { label: "Score", val: "9.2", sub: "Excelente", subColor: "#C97A5F", accent: true },
          ].map(k => (
            <div key={k.label} style={{
              background: k.accent ? "linear-gradient(135deg,rgba(184,86,61,.5),rgba(159,74,52,.2))" : "#232C37",
              border: k.accent ? "1px solid rgba(184,86,61,.35)" : "1px solid rgba(244,238,226,0.07)",
              borderRadius: 12, padding: "12px 14px",
            }}>
              <div style={{ fontSize: 9, textTransform: "uppercase", letterSpacing: "0.18em", color: "rgba(244,238,226,0.35)", marginBottom: 6 }}>{k.label}</div>
              <div style={{ fontSize: 18, fontWeight: 700, color: "#F4EEE2" }}>{k.val}</div>
              <div style={{ fontSize: 10, color: k.subColor, marginTop: 4 }}>{k.sub}</div>
            </div>
          ))}
        </div>
        {/* Chart */}
        <div style={{ background: "#232C37", border: "1px solid rgba(244,238,226,0.07)", borderRadius: 12, padding: "14px 16px", marginBottom: 12 }}>
          <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", marginBottom: 14 }}>
            <div>
              <div style={{ fontSize: 12, fontWeight: 600, color: "rgba(244,238,226,0.6)" }}>Receita mensal</div>
              <div style={{ fontSize: 10, color: "rgba(244,238,226,0.32)" }}>Jan – Jun 2025 (R$ mil)</div>
            </div>
            <div style={{ display: "flex", gap: 6 }}>
              {["bar", "line"].map(m => (
                <button key={m} onClick={() => setChartMode(m)} style={{
                  fontSize: 10, padding: "4px 10px", borderRadius: 7,
                  border: "1px solid rgba(244,238,226,0.15)",
                  background: chartMode === m ? "#B8563D" : "transparent",
                  borderColor: chartMode === m ? "#B8563D" : undefined,
                  color: chartMode === m ? "#F4EEE2" : "rgba(244,238,226,0.35)",
                  cursor: "pointer", fontFamily: sans,
                }}>
                  {m === "bar" ? "Barras" : "Linha"}
                </button>
              ))}
            </div>
          </div>
          {chartMode === "bar" ? (
            <div style={{ display: "flex", alignItems: "flex-end", gap: 7, height: 100 }}>
              {CHART_VALS.map((v, i) => (
                <div key={i} style={{ flex: 1, display: "flex", flexDirection: "column", justifyContent: "flex-end", alignItems: "center", height: "100%", gap: 4 }}>
                  <div style={{
                    width: "100%", borderRadius: "5px 5px 0 0",
                    height: Math.round((v / maxVal) * 90),
                    background: "linear-gradient(to top, #B8563D, #C97A5F)",
                    transition: "height 0.6s cubic-bezier(.34,1.2,.64,1)",
                  }} />
                  <div style={{ fontSize: 9, color: "rgba(244,238,226,0.32)" }}>{CHART_MONTHS[i]}</div>
                </div>
              ))}
            </div>
          ) : (
            <div>
              <svg viewBox="0 0 100 100" preserveAspectRatio="none" style={{ width: "100%", height: 90, overflow: "visible" }}>
                <polygon
                  points={`0,100 ${CHART_VALS.map((v, i) => `${i * (100 / (CHART_VALS.length - 1))},${100 - (v / maxVal) * 100}`).join(" ")} 100,100`}
                  fill="rgba(184,86,61,0.2)"
                />
                <polyline
                  points={CHART_VALS.map((v, i) => `${i * (100 / (CHART_VALS.length - 1))},${100 - (v / maxVal) * 100}`).join(" ")}
                  fill="none" stroke="#C97A5F" strokeWidth="2" vectorEffect="non-scaling-stroke"
                />
                {CHART_VALS.map((v, i) => (
                  <circle key={i}
                    cx={i * (100 / (CHART_VALS.length - 1))}
                    cy={100 - (v / maxVal) * 100}
                    r="3" fill="#C97A5F" vectorEffect="non-scaling-stroke"
                  />
                ))}
              </svg>
              <div style={{ display: "flex", gap: 7, marginTop: 4 }}>
                {CHART_MONTHS.map(m => (
                  <div key={m} style={{ flex: 1, textAlign: "center", fontSize: 9, color: "rgba(244,238,226,0.32)" }}>{m}</div>
                ))}
              </div>
            </div>
          )}
        </div>
        {/* Activity feed */}
        <div style={{ background: "#232C37", border: "1px solid rgba(244,238,226,0.07)", borderRadius: 12, padding: "12px 14px" }}>
          <div style={{ fontSize: 12, fontWeight: 600, color: "rgba(244,238,226,0.6)", marginBottom: 10 }}>Atividade recente</div>
          {[
            { color: "#4ade80", text: "Nova venda registrada · R$ 890", sub: "há 2 min" },
            { color: "#C97A5F", text: "Lead capturado via Instagram", sub: "há 5 min" },
            { color: "#B8563D", text: "Tarefa concluída · Envio de proposta", sub: "há 12 min" },
            { color: "#4ade80", text: "Pagamento confirmado · R$ 2.400", sub: "há 18 min" },
          ].map((ev, i) => (
            <div key={i} style={{ display: "flex", alignItems: "flex-start", gap: 10, padding: "8px 0", borderBottom: i < 3 ? "1px solid rgba(244,238,226,0.06)" : "none" }}>
              <div style={{ width: 7, height: 7, borderRadius: "50%", background: ev.color, flexShrink: 0, marginTop: 4 }} />
              <div>
                <div style={{ fontSize: 11, color: "rgba(244,238,226,0.6)", lineHeight: 1.4 }}>{ev.text}</div>
                <div style={{ fontSize: 10, color: "rgba(244,238,226,0.32)", marginTop: 2 }}>{ev.sub}</div>
              </div>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

// ─── Modules ──────────────────────────────────────────────────────────────────
function Modules() {
  const [openModule, setOpenModule] = useState(null);
  const [hovered, setHovered] = useState(null);

  const toggle = (i) => setOpenModule(openModule === i ? null : i);

  const modules = [
    {
      name: "Atendimento automatizado",
      desc: "Resposta automática no WhatsApp: tira dúvida, informa preço e agenda horário, mesmo quando você não está disponível.",
      example: <><strong style={{ color: T.terra }}>Cliente:</strong> qual o valor do corte?<br /><strong style={{ color: T.terra }}>Nexo:</strong> R$45 o corte, R$65 com barba. Quer agendar?</>,
    },
    {
      name: "Painel de gestão",
      desc: "Acesso ao painel com seus dados: controle financeiro, cadastro de clientes e relatório simples de como o negócio está indo.",
      dashboard: true,
      example: <><strong style={{ color: T.terra }}>Painel:</strong> "Essa semana: 34 atendimentos, 12 agendamentos, faturamento R$2.840."</>,
    },
    {
      name: "Motor de conteúdo",
      desc: "Cria posts, legendas e anúncios em minutos. O Nexo aprende sua voz e gera conteúdo alinhado com a sua marca.",
      example: <><strong style={{ color: T.terra }}>Nexo gerou:</strong> "Barbearia Nexo — onde o estilo encontra a tradição. Agende já o seu horário 💈 #barbearia #corte"</>,
    },
  ];

  return (
    <section style={{ position: "relative", zIndex: 1, padding: "76px 0", borderTop: `1px solid ${T.line}` }}>
      <div style={{ maxWidth: 1080, margin: "0 auto", padding: "0 30px" }}>
        <Reveal>
          <div style={{ maxWidth: 560, marginBottom: 44 }}>
            <div style={{ fontFamily: mono, fontSize: 11, letterSpacing: "0.18em", textTransform: "uppercase", color: T.terra, marginBottom: 12, display: "flex", alignItems: "center", gap: 10 }}>
              <span style={{ width: 14, height: 1, background: T.terra, display: "inline-block" }} />
              O que você recebe
            </div>
            <h2 style={{ fontFamily: serif, fontWeight: 400, fontSize: "2rem", letterSpacing: "-0.01em", color: T.graphite }}>Três propostas, prontas pra usar</h2>
            <p style={{ marginTop: 14, color: T.inkSoft, fontSize: "1.02rem", fontFamily: sans }}>Cada módulo resolve uma parte do dia a dia. Você pode contratar um, dois, ou os três juntos.</p>
          </div>
        </Reveal>
        <div style={{ display: "grid", gridTemplateColumns: "repeat(3,1fr)", gap: 26 }} className="modules-grid">
          {modules.map((m, i) => (
            <Reveal key={i} delay={i * 0.08}>
              <div
                onMouseEnter={() => setHovered(i)}
                onMouseLeave={() => setHovered(null)}
                style={{
                  background: T.cream2, borderRadius: 10, padding: 30,
                  border: `1px solid ${hovered === i ? T.terra : T.line}`,
                  transform: hovered === i ? "translateY(-5px)" : "none",
                  boxShadow: hovered === i ? "0 18px 34px rgba(36,29,26,0.14)" : "none",
                  transition: "transform 0.25s ease, box-shadow 0.25s ease, border-color 0.25s ease",
                  height: "100%",
                }}
              >
                <div style={{ fontFamily: serif, fontSize: "1.25rem", marginBottom: 10, color: T.graphite }}>{m.name}</div>
                <div style={{ color: T.inkSoft, fontSize: "0.95rem", fontFamily: sans }}>{m.desc}</div>
                <div style={{ borderTop: `1px solid ${T.line}`, marginTop: 16, paddingTop: 12 }}>
                  <button
                    onClick={() => toggle(i)}
                    style={{
                      display: "flex", alignItems: "center", gap: 8, cursor: "pointer",
                      fontFamily: sans, fontWeight: 600, fontSize: "0.85rem", color: T.terra,
                      background: "none", border: "none", padding: 0,
                    }}
                  >
                    <span style={{ fontSize: "0.75rem", display: "inline-block", transform: openModule === i ? "rotate(90deg)" : "none", transition: "transform 0.2s" }}>▸</span>
                    Visualizar ilustrativo
                  </button>
                  {openModule === i && (
                    <div style={{ marginTop: 12 }}>
                      {m.dashboard ? (
                        <DashboardEmbed />
                      ) : (
                        <div style={{
                          padding: "12px 14px", background: T.cream, borderLeft: `2px solid ${T.terra}`,
                          fontFamily: sans, fontSize: 13, lineHeight: 1.6, color: T.slate, borderRadius: "0 6px 6px 0",
                        }}>
                          {m.example}
                        </div>
                      )}
                    </div>
                  )}
                </div>
              </div>
            </Reveal>
          ))}
        </div>
      </div>
    </section>
  );
}

// ─── How it works ─────────────────────────────────────────────────────────────
function HowItWorks() {
  const steps = [
    { n: "1.", title: "Você nos conta sobre o seu negócio", desc: "Uma conversa de 30 minutos para entender o que você faz, quem são seus clientes e quais são as maiores dores do dia a dia." },
    { n: "2.", title: "A gente configura tudo pra você", desc: "Sem tecnês, sem planilhas. Nossa equipe monta o atendimento, o painel e os conteúdos com base no que você nos passou." },
    { n: "3.", title: "Ativa em até 1 semana", desc: "Em 7 dias o Nexo já está no ar respondendo clientes, organizando dados e publicando conteúdo. Você só aprova." },
    { n: "4.", title: "Suporte e ajustes inclusos", desc: "Os primeiros 15 dias vêm com suporte direto para qualquer dúvida ou ajuste fino. Você nunca fica sozinho." },
  ];

  return (
    <section id="como-funciona" style={{ position: "relative", zIndex: 1, padding: "76px 0", borderTop: `1px solid ${T.line}` }}>
      <div style={{ maxWidth: 1080, margin: "0 auto", padding: "0 30px" }}>
        <Reveal>
          <div style={{ maxWidth: 560, marginBottom: 44 }}>
            <div style={{ fontFamily: mono, fontSize: 11, letterSpacing: "0.18em", textTransform: "uppercase", color: T.terra, marginBottom: 12, display: "flex", alignItems: "center", gap: 10 }}>
              <span style={{ width: 14, height: 1, background: T.terra, display: "inline-block" }} />
              Como funciona
            </div>
            <h2 style={{ fontFamily: serif, fontWeight: 400, fontSize: "2rem", letterSpacing: "-0.01em", color: T.graphite }}>Do zero ao ar em 1 semana</h2>
            <p style={{ marginTop: 14, color: T.inkSoft, fontSize: "1.02rem", fontFamily: sans }}>Sem precisar contratar desenvolvedor, sem custo de implantação, sem surpresa.</p>
          </div>
        </Reveal>
        <div>
          {steps.map((s, i) => (
            <Reveal key={i} delay={i * 0.06}>
              <div style={{
                display: "flex", gap: 22, padding: "22px 0",
                borderTop: `1px solid ${T.line}`,
                borderBottom: i === steps.length - 1 ? `1px solid ${T.line}` : "none",
              }}>
                <div style={{ fontFamily: serif, fontStyle: "italic", color: T.terra, fontSize: "1.4rem", width: 32, flexShrink: 0 }}>{s.n}</div>
                <div>
                  <div style={{ fontWeight: 600, fontSize: "1.02rem", color: T.graphite, fontFamily: sans }}>{s.title}</div>
                  <div style={{ color: T.inkSoft, fontSize: "0.94rem", marginTop: 5, maxWidth: 480, fontFamily: sans }}>{s.desc}</div>
                </div>
              </div>
            </Reveal>
          ))}
        </div>
      </div>
    </section>
  );
}

// ─── Testimonials ─────────────────────────────────────────────────────────────
function Testimonials() {
  const items = [
    {
      quote: "Antes eu perdia cliente porque não dava conta de responder no WhatsApp à noite. Hoje o Nexo responde na hora e eu só confirmo o agendamento.",
      name: "Marcos T.",
      role: "Barbearia Vintage, São Paulo",
    },
    {
      quote: "O painel me mostrou que eu tava perdendo dinheiro num serviço que eu achava que era o mais lucrativo. Mudei o preço no mesmo mês.",
      name: "Renata A.",
      role: "Clínica de estética, Belo Horizonte",
    },
    {
      quote: "Não tenho tempo pra pensar em legenda de post. O Nexo escreve, eu só reviso e publico. Meu Instagram nunca ficou tanto tempo ativo.",
      name: "Diego S.",
      role: "Loja de roupas, Curitiba",
    },
  ];

  return (
    <section style={{ position: "relative", zIndex: 1, padding: "76px 0", borderTop: `1px solid ${T.line}`, background: T.cream2 }}>
      <div style={{ maxWidth: 1080, margin: "0 auto", padding: "0 30px" }}>
        <Reveal>
          <div style={{ maxWidth: 560, marginBottom: 44 }}>
            <div style={{ fontFamily: mono, fontSize: 11, letterSpacing: "0.18em", textTransform: "uppercase", color: T.terra, marginBottom: 12, display: "flex", alignItems: "center", gap: 10 }}>
              <span style={{ width: 14, height: 1, background: T.terra, display: "inline-block" }} />
              Quem já usa
            </div>
            <h2 style={{ fontFamily: serif, fontWeight: 400, fontSize: "2rem", letterSpacing: "-0.01em", color: T.graphite }}>Donos de negócio, não desenvolvedores</h2>
          </div>
        </Reveal>
        <div style={{ display: "grid", gridTemplateColumns: "repeat(3,1fr)", gap: 26 }} className="modules-grid">
          {items.map((t, i) => (
            <Reveal key={i} delay={i * 0.08}>
              <div style={{
                background: T.cream, borderRadius: 10, padding: "28px 26px",
                border: `1px solid ${T.line}`, height: "100%",
                display: "flex", flexDirection: "column", justifyContent: "space-between",
              }}>
                <div>
                  <div style={{ fontFamily: serif, fontStyle: "italic", fontSize: "2.2rem", color: T.terra, lineHeight: 0.5, marginBottom: 14 }}>"</div>
                  <p style={{ fontFamily: sans, fontSize: "0.95rem", color: T.inkSoft, lineHeight: 1.7 }}>{t.quote}</p>
                </div>
                <div style={{ marginTop: 22, paddingTop: 14, borderTop: `1px solid ${T.line}` }}>
                  <div style={{ fontFamily: sans, fontWeight: 600, fontSize: "0.9rem", color: T.graphite }}>{t.name}</div>
                  <div style={{ fontFamily: mono, fontSize: 10.5, color: T.inkSoft, marginTop: 2, letterSpacing: "0.03em" }}>{t.role}</div>
                </div>
              </div>
            </Reveal>
          ))}
        </div>
      </div>
    </section>
  );
}

// ─── Pricing ─────────────────────────────────────────────────────────────────
function Pricing() {
  const tiers = [
    { label: "Starter", sublabel: "Para começar", price: "R$", value: "79", cents: "/mês", note: "1 módulo à escolha. Suporte por e-mail. Ideal pra quem quer testar.", tag: "sem fidelidade", featured: false },
    { label: "Pro", sublabel: "O mais escolhido", price: "R$", value: "149", cents: "/mês", note: "Os 3 módulos completos. Suporte WhatsApp. Onboarding dedicado.", tag: "mais popular ★", featured: true },
    { label: "Business", sublabel: "Para crescer rápido", price: "", value: "Sob consulta", cents: "", note: "Módulos personalizados, integrações sob medida e conta dedicada.", tag: "fale conosco", featured: false },
  ];

  return (
    <section style={{ position: "relative", zIndex: 1, padding: "76px 0", borderTop: `1px solid ${T.line}` }}>
      <div style={{ maxWidth: 1080, margin: "0 auto", padding: "0 30px" }}>
        <Reveal>
          <div style={{ maxWidth: 560, marginBottom: 44 }}>
            <div style={{ fontFamily: mono, fontSize: 11, letterSpacing: "0.18em", textTransform: "uppercase", color: T.terra, marginBottom: 12, display: "flex", alignItems: "center", gap: 10 }}>
              <span style={{ width: 14, height: 1, background: T.terra, display: "inline-block" }} />
              Investimento
            </div>
            <h2 style={{ fontFamily: serif, fontWeight: 400, fontSize: "2rem", letterSpacing: "-0.01em", color: T.graphite }}>Preço fixo. Sem surpresa.</h2>
            <p style={{ marginTop: 14, color: T.inkSoft, fontSize: "1.02rem", fontFamily: sans }}>Cancele quando quiser. Sem contrato de fidelidade.</p>
          </div>
        </Reveal>

        <Reveal delay={0.05}>
          <div style={{
            background: T.slateDeep, color: T.cream, borderRadius: 10,
            padding: 44, position: "relative", overflow: "hidden",
          }}>
            <div style={{ position: "absolute", top: 0, left: 0, width: "100%", height: 3, background: T.terra }} />
            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr 1fr", gap: 0 }} className="invest-split">
              {tiers.map((t, i) => (
                <div key={i} className={t.featured ? "tier-featured" : undefined} style={{
                  padding: i > 0 ? "0 0 0 36px" : "0 36px 0 0",
                  borderLeft: i > 0 ? "1px solid rgba(244,238,226,0.15)" : "none",
                  position: "relative",
                  ...(t.featured ? {
                    background: "rgba(184,86,61,0.1)",
                    borderRadius: 14,
                    padding: "22px 30px",
                    margin: "-22px 0",
                    border: `1px solid rgba(184,86,61,0.45)`,
                  } : {}),
                }}>
                  <div style={{ fontFamily: mono, fontSize: 11, letterSpacing: "0.15em", textTransform: "uppercase", color: "rgba(244,238,226,0.55)" }}>{t.label}</div>
                  <div style={{ fontFamily: mono, fontSize: 9, letterSpacing: "0.1em", textTransform: "uppercase", color: "rgba(244,238,226,0.35)", marginTop: 2 }}>{t.sublabel}</div>
                  <div style={{ display: "flex", alignItems: "baseline", gap: 2, marginTop: 12 }}>
                    {t.price && <span style={{ fontFamily: serif, fontWeight: 400, fontSize: "1.8rem", color: T.cream }}>{t.price}</span>}
                    <span style={{ fontFamily: serif, fontWeight: 400, fontSize: t.cents ? "2.9rem" : "2rem", color: T.cream }}>{t.value}</span>
                    {t.cents && <span style={{ fontFamily: sans, fontSize: "1rem", color: T.terraS }}>{t.cents}</span>}
                  </div>
                  <div style={{ marginTop: 14, fontSize: 13.5, color: "rgba(244,238,226,0.75)", maxWidth: 220, fontFamily: sans, lineHeight: 1.6 }}>{t.note}</div>
                  <span style={{
                    display: "inline-block", marginTop: 12, fontFamily: mono, fontSize: 10.5,
                    letterSpacing: "0.08em", textTransform: "uppercase", color: t.featured ? T.cream : T.terraS,
                    background: t.featured ? T.terra : "transparent",
                    border: t.featured ? "none" : "1px solid rgba(201,122,95,0.4)", padding: "3px 9px", borderRadius: 100,
                  }}>{t.tag}</span>
                </div>
              ))}
            </div>

            <div style={{ marginTop: 34, display: "flex", alignItems: "center", gap: 20, flexWrap: "wrap" }}>
              <a
                href="https://wa.me/5511958642044?text=Ol%C3%A1%2C%20quero%20contratar%20o%20Nexo"
                target="_blank" rel="noopener"
                style={{
                  fontFamily: sans, fontWeight: 600, fontSize: "0.95rem",
                  background: T.terra, color: T.cream, padding: "15px 28px",
                  borderRadius: 100, textDecoration: "none", display: "inline-block",
                  transition: "background 0.2s, transform 0.2s",
                }}
                onMouseEnter={e => { e.currentTarget.style.background = "#9F4A34"; e.currentTarget.style.transform = "translateY(-2px)"; }}
                onMouseLeave={e => { e.currentTarget.style.background = T.terra; e.currentTarget.style.transform = "none"; }}
              >
                Começar agora
              </a>
              <span style={{ fontFamily: mono, fontSize: 11, color: "rgba(244,238,226,0.45)", letterSpacing: "0.1em" }}>
                Sem cartão de crédito · Ativa em 7 dias
              </span>
            </div>
          </div>
        </Reveal>
      </div>
    </section>
  );
}

// ─── FAQ ─────────────────────────────────────────────────────────────────────
function FAQ() {
  const [open, setOpen] = useState(null);

  const items = [
    { q: "Preciso entender de tecnologia para usar o Nexo?", a: "Não. O Nexo foi pensado para donos de negócio, não para desenvolvedores. Nossa equipe configura tudo e você recebe tudo pronto para usar." },
    { q: "Quanto tempo leva para o Nexo entrar no ar?", a: "Em média 5 a 7 dias úteis após a conversa de onboarding. Casos mais complexos podem levar até 2 semanas." },
    { q: "O que acontece depois dos 15 dias de suporte?", a: "Você continua com acesso ao suporte por e-mail incluído no plano. O suporte WhatsApp dedicado é exclusivo do plano Pro." },
    { q: "Posso cancelar a qualquer momento?", a: "Sim. Não há contrato de fidelidade. Você cancela quando quiser, sem multa." },
    { q: "O Nexo funciona para qualquer tipo de negócio?", a: "Funciona para a maioria das PMEs: barbearias, clínicas, lojas, restaurantes, escritórios, salões, construtoras e mais. Se tiver dúvida sobre o seu setor, fale com a gente." },
    { q: "Meus dados ficam seguros?", a: "Sim. Usamos infraestrutura segura com criptografia de dados e não compartilhamos informações com terceiros." },
  ];

  return (
    <section style={{ position: "relative", zIndex: 1, padding: "76px 0", borderTop: `1px solid ${T.line}` }}>
      <div style={{ maxWidth: 1080, margin: "0 auto", padding: "0 30px" }}>
        <Reveal>
          <div style={{ maxWidth: 560, marginBottom: 44 }}>
            <div style={{ fontFamily: mono, fontSize: 11, letterSpacing: "0.18em", textTransform: "uppercase", color: T.terra, marginBottom: 12, display: "flex", alignItems: "center", gap: 10 }}>
              <span style={{ width: 14, height: 1, background: T.terra, display: "inline-block" }} />
              Perguntas frequentes
            </div>
            <h2 style={{ fontFamily: serif, fontWeight: 400, fontSize: "2rem", letterSpacing: "-0.01em", color: T.graphite }}>Ainda com dúvida?</h2>
          </div>
        </Reveal>
        <Reveal delay={0.05}>
          <div>
            {items.map((item, i) => (
              <div key={i} style={{ borderTop: `1px solid ${T.line}`, ...(i === items.length - 1 ? { borderBottom: `1px solid ${T.line}` } : {}) }}>
                <button
                  onClick={() => setOpen(open === i ? null : i)}
                  style={{
                    display: "flex", justifyContent: "space-between", alignItems: "center",
                    width: "100%", padding: "22px 0", fontWeight: 600, fontSize: "1rem",
                    color: T.graphite, fontFamily: sans, background: "none", border: "none",
                    cursor: "pointer", gap: 20, textAlign: "left",
                  }}
                >
                  <span>{item.q}</span>
                  <span style={{ fontFamily: serif, fontSize: "1.4rem", color: T.terra, flexShrink: 0, display: "inline-block", transform: open === i ? "rotate(45deg)" : "none", transition: "transform 0.2s" }}>+</span>
                </button>
                {open === i && (
                  <p style={{ paddingBottom: 24, color: T.inkSoft, fontSize: "0.95rem", maxWidth: 640, fontFamily: sans, lineHeight: 1.7 }}>
                    {item.a}
                  </p>
                )}
              </div>
            ))}
          </div>
        </Reveal>
      </div>
    </section>
  );
}

// ─── Final CTA ────────────────────────────────────────────────────────────────
function FinalCTA() {
  return (
    <section style={{
      background: T.slateDeep, color: T.cream,
      padding: "76px 0", textAlign: "center",
      position: "relative", zIndex: 1,
    }}>
      <div style={{ position: "absolute", top: 0, left: 0, width: "100%", height: 3, background: T.terra }} />
      <div style={{ maxWidth: 1080, margin: "0 auto", padding: "0 30px" }}>
        <Reveal>
          <div style={{ fontFamily: mono, fontSize: 11, letterSpacing: "0.18em", textTransform: "uppercase", color: T.terraS, marginBottom: 12 }}>
            Pronto para começar?
          </div>
          <h2 style={{ fontFamily: serif, fontWeight: 400, fontSize: "clamp(1.8rem, 3.5vw, 2.8rem)", color: T.cream, letterSpacing: "-0.01em" }}>
            Automatize seu negócio.<br />Sem complicação.
          </h2>
          <p style={{ marginTop: 16, fontSize: "1.05rem", color: "rgba(244,238,226,0.7)", fontFamily: sans, maxWidth: 480, margin: "16px auto 0" }}>
            Em 1 semana o Nexo já está respondendo clientes, organizando dados e criando conteúdo pelo você.
          </p>
          <div style={{ display: "flex", gap: 14, marginTop: 30, justifyContent: "center", flexWrap: "wrap" }}>
            <a
              href="https://wa.me/5511958642044?text=Ol%C3%A1%2C%20quero%20contratar%20o%20Nexo"
              target="_blank" rel="noopener"
              style={{
                fontFamily: sans, fontWeight: 600, fontSize: "0.95rem",
                background: T.terra, color: T.cream, padding: "15px 28px",
                borderRadius: 100, textDecoration: "none",
                transition: "background 0.2s, transform 0.2s",
              }}
              onMouseEnter={e => { e.currentTarget.style.background = "#9F4A34"; e.currentTarget.style.transform = "translateY(-2px)"; }}
              onMouseLeave={e => { e.currentTarget.style.background = T.terra; e.currentTarget.style.transform = "none"; }}
            >
              Quero contratar agora
            </a>
            <a
              href="https://wa.me/5511958642044?text=Ol%C3%A1%2C%20tenho%20d%C3%BAvidas%20sobre%20o%20Nexo"
              target="_blank" rel="noopener"
              style={{
                fontFamily: sans, fontWeight: 600, fontSize: "0.95rem",
                color: T.cream, padding: "15px 24px", borderRadius: 100,
                border: "1px solid rgba(244,238,226,0.35)", textDecoration: "none",
                transition: "border-color 0.2s",
              }}
              onMouseEnter={e => (e.currentTarget.style.borderColor = T.cream)}
              onMouseLeave={e => (e.currentTarget.style.borderColor = "rgba(244,238,226,0.35)")}
            >
              Tirar dúvidas
            </a>
          </div>
        </Reveal>
      </div>
    </section>
  );
}

// ─── Footer ──────────────────────────────────────────────────────────────────
function Footer() {
  return (
    <footer style={{
      position: "relative", zIndex: 1, padding: "44px 0", textAlign: "center",
      fontFamily: mono, fontSize: 11, letterSpacing: "0.1em", textTransform: "uppercase",
      color: "rgba(46,58,70,0.45)",
      background: T.cream,
      borderTop: `1px solid ${T.line}`,
    }}>
      <div style={{ maxWidth: 1080, margin: "0 auto", padding: "0 30px" }}>
        <div style={{ marginBottom: 16 }}>
          <span style={{ fontFamily: serif, fontSize: "1.1rem", color: T.graphite, letterSpacing: "0.05em" }}>
            Ne<span style={{ color: T.terra, fontStyle: "italic" }}>x</span>o
          </span>
        </div>
        <div style={{ display: "flex", justifyContent: "center", gap: 32, flexWrap: "wrap", marginBottom: 20 }}>
          {["Plataforma", "Como funciona", "Preços", "Perguntas"].map(l => (
            <a key={l} href="#" style={{ color: "rgba(46,58,70,0.45)", textDecoration: "none", transition: "color 0.2s" }}
              onMouseEnter={e => (e.currentTarget.style.color = T.inkSoft)}
              onMouseLeave={e => (e.currentTarget.style.color = "rgba(46,58,70,0.45)")}>
              {l}
            </a>
          ))}
        </div>
        <div style={{ color: "rgba(46,58,70,0.3)", fontSize: 10 }}>
          ©️ {new Date().getFullYear()} Nexo · Automação Inteligente para PMEs · Todos os direitos reservados
        </div>
      </div>
    </footer>
  );
}

// ─── Grain overlay ───────────────────────────────────────────────────────────
function Grain() {
  return (
    <div style={{
      position: "fixed", inset: 0, pointerEvents: "none", zIndex: 0, opacity: 0.035, mixBlendMode: "multiply",
      backgroundImage: `url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='140' height='140'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='2' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E")`,
    }} />
  );
}

// ─── Floating WhatsApp button ─────────────────────────────────────────────────
function FloatingWhatsApp() {
  const [visible, setVisible] = useState(false);
  useEffect(() => {
    const h = () => setVisible(window.scrollY > 500);
    window.addEventListener("scroll", h);
    return () => window.removeEventListener("scroll", h);
  }, []);

  return (
    <a
      href="https://wa.me/5511958642044?text=Ol%C3%A1%2C%20quero%20saber%20mais%20sobre%20o%20Nexo"
      target="_blank" rel="noopener"
      aria-label="Falar no WhatsApp"
      style={{
        position: "fixed", bottom: 24, right: 24, zIndex: 60,
        width: 56, height: 56, borderRadius: "50%",
        background: T.terra, display: "flex", alignItems: "center", justifyContent: "center",
        boxShadow: "0 12px 28px rgba(184,86,61,0.45)",
        opacity: visible ? 1 : 0,
        transform: visible ? "scale(1)" : "scale(0.7)",
        pointerEvents: visible ? "auto" : "none",
        transition: "opacity 0.3s ease, transform 0.3s ease, background 0.2s",
      }}
      onMouseEnter={e => (e.currentTarget.style.background = "#9F4A34")}
      onMouseLeave={e => (e.currentTarget.style.background = T.terra)}
    >
      <span className="wa-pulse" />
      <svg width="26" height="26" viewBox="0 0 24 24" fill="none" style={{ position: "relative" }}>
        <path d="M12.04 2C6.58 2 2.13 6.45 2.13 11.91c0 1.75.46 3.45 1.32 4.95L2 22l5.28-1.38a9.9 9.9 0 0 0 4.76 1.21h.01c5.46 0 9.91-4.45 9.91-9.92C21.96 6.45 17.5 2 12.04 2Z" stroke="#F4EEE2" strokeWidth="0" fill="#F4EEE2" fillOpacity="0.0" />
        <path d="M9.1 7.05c-.2-.45-.4-.46-.6-.47h-.5c-.18 0-.46.07-.7.33-.24.26-.92.9-.92 2.2 0 1.29.94 2.54 1.07 2.72.13.18 1.83 2.94 4.52 4 2.24.88 2.7.7 3.18.66.49-.04 1.58-.64 1.8-1.27.23-.62.23-1.15.16-1.27-.07-.11-.25-.18-.5-.3-.27-.13-1.58-.78-1.82-.87-.25-.09-.43-.13-.6.14-.19.27-.7.87-.86 1.05-.16.18-.31.2-.58.07-.27-.14-1.13-.42-2.15-1.34-.8-.71-1.33-1.58-1.49-1.85-.15-.27-.02-.42.12-.55.12-.13.27-.32.4-.48.13-.16.18-.27.27-.46.09-.18.05-.34-.02-.48-.07-.14-.6-1.49-.83-2.04Z" fill={T.cream} />
        <path d="M12.04 2C6.58 2 2.13 6.45 2.13 11.91c0 1.75.46 3.45 1.32 4.95L2 22l5.28-1.38a9.9 9.9 0 0 0 4.76 1.21h.01c5.46 0 9.91-4.45 9.91-9.92C21.96 6.45 17.5 2 12.04 2Zm0 18.06h-.01a8.2 8.2 0 0 1-4.19-1.15l-.3-.18-3.13.82.84-3.05-.2-.31a8.24 8.24 0 0 1-1.27-4.28c0-4.55 3.71-8.26 8.27-8.26 2.21 0 4.28.86 5.84 2.42a8.2 8.2 0 0 1 2.42 5.84c0 4.56-3.71 8.15-8.27 8.15Z" fill={T.cream} />
      </svg>
    </a>
  );
}

// ─── App ─────────────────────────────────────────────────────────────────────
export default function App() {
  return (
    <div style={{ background: T.cream, color: T.graphite, fontFamily: sans, lineHeight: 1.6, WebkitFontSmoothing: "antialiased" }}>
      <style>{`
        ::selection { background: #B8563D; color: #F4EEE2; }
        * { box-sizing: border-box; }
        a:focus-visible, button:focus-visible { outline: 2px solid #B8563D; outline-offset: 3px; border-radius: 4px; }
        ::-webkit-scrollbar { width: 4px; }
        ::-webkit-scrollbar-thumb { background: rgba(184,86,61,0.2); border-radius: 4px; }
        @keyframes typingBlink { 0%, 80%, 100% { opacity: .25; } 40% { opacity: 1; } }
        @keyframes waPulse {
          0% { box-shadow: 0 0 0 0 rgba(184,86,61,0.55); }
          70% { box-shadow: 0 0 0 14px rgba(184,86,61,0); }
          100% { box-shadow: 0 0 0 0 rgba(184,86,61,0); }
        }
        .wa-pulse {
          position: absolute; inset: 0; border-radius: 50%;
          animation: waPulse 2.4s infinite;
        }
        @media (prefers-reduced-motion: reduce) {
          .reveal { opacity: 1 !important; transform: none !important; transition: none !important; }
          .wa-pulse { animation: none; }
        }
        @media (max-width: 860px) {
          .hero-grid { grid-template-columns: 1fr !important; }
          .hero-phone { margin-top: 20px; }
          .modules-grid { grid-template-columns: 1fr !important; }
          .invest-split { grid-template-columns: 1fr !important; }
          .invest-split > div { border-left: none !important; padding-left: 0 !important; border-top: 1px solid rgba(244,238,226,0.15) !important; padding-top: 28px !important; margin-top: 6px; }
          .invest-split > div:first-child { border-top: none !important; padding-top: 0 !important; }
          .tier-featured { margin: 6px 0 !important; }
        }
        @media (max-width: 520px) {
          .hero-grid { padding: 0 20px !important; }
        }
      `}</style>
      <Grain />
      <Nav />
      <Hero />
      <Modules />
      <HowItWorks />
      <Testimonials />
      <Pricing />
      <FAQ />
      <FinalCTA />
      <Footer />
      <FloatingWhatsApp />
    </div>
  );
}
