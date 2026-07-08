import { useState, useRef, useEffect, useCallback } from "react";
import { ChevronLeft, ChevronRight, Play, Info, Star, Volume2, VolumeX, Search, Bell, User, X } from "lucide-react";

// ─── Tokens ────────────────────────────────────────────────────────────────
const T = {
  bg: "#090D12",
  bgCard: "#131920",
  slate: "#1B222B",
  slate2: "#2E3A46",
  cream: "#F4EEE2",
  creamDim: "rgba(244,238,226,0.7)",
  creamFaint: "rgba(244,238,226,0.35)",
  terra: "#B8563D",
  terraS: "#C97A5F",
  terraGlow: "rgba(184,86,61,0.35)",
  line: "rgba(184,86,61,0.18)",
};

const serif = "'Playfair Display', Georgia, serif";
const sans = "'Inter', system-ui, sans-serif";
const mono = "'JetBrains Mono', monospace";

// ─── Reveal on scroll ──────────────────────────────────────────────────────
function Reveal({ children, delay = 0 }: { children: React.ReactNode; delay?: number }) {
  const ref = useRef<HTMLDivElement>(null);
  const [visible, setVisible] = useState(false);

  useEffect(() => {
    const el = ref.current;
    if (!el) return;
    const obs = new IntersectionObserver(
      ([entry]) => { if (entry.isIntersecting) { setVisible(true); obs.disconnect(); } },
      { threshold: 0.12 }
    );
    obs.observe(el);
    return () => obs.disconnect();
  }, []);

  return (
    <div
      ref={ref}
      style={{
        opacity: visible ? 1 : 0,
        transform: visible ? "translateY(0)" : "translateY(22px)",
        transition: `opacity 0.75s ease ${delay}s, transform 0.75s cubic-bezier(.2,.8,.2,1) ${delay}s`,
      }}
    >
      {children}
    </div>
  );
}

// ─── Nav ───────────────────────────────────────────────────────────────────
function Nav() {
  const [scrolled, setScrolled] = useState(false);
  const [muted, setMuted] = useState(true);

  useEffect(() => {
    const h = () => setScrolled(window.scrollY > 60);
    window.addEventListener("scroll", h);
    return () => window.removeEventListener("scroll", h);
  }, []);

  return (
    <nav style={{
      position: "fixed", top: 0, left: 0, right: 0, zIndex: 50,
      background: scrolled
        ? "linear-gradient(to bottom, rgba(9,13,18,0.98), rgba(9,13,18,0.92))"
        : "linear-gradient(to bottom, rgba(9,13,18,0.85) 0%, transparent 100%)",
      backdropFilter: scrolled ? "blur(12px)" : "none",
      borderBottom: scrolled ? `1px solid ${T.line}` : "none",
      transition: "all 0.4s ease",
      padding: "0 48px",
    }}>
      <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", height: 68, maxWidth: 1440, margin: "0 auto" }}>
        {/* Logo */}
        <div style={{ display: "flex", alignItems: "center", gap: 36 }}>
          <div style={{ fontFamily: serif, fontSize: "1.75rem", fontWeight: 700, color: T.cream, letterSpacing: "-0.02em" }}>
            Ne<span style={{ color: T.terra, fontStyle: "italic" }}>x</span>o
          </div>
          <div style={{ display: "flex", gap: 24, alignItems: "center" }} className="nav-links">
            {["Início", "Módulos", "Como funciona", "Planos"].map((l) => (
              <a
                key={l}
                href="#"
                onClick={(e) => {
                  e.preventDefault();
                  const id = l === "Como funciona" ? "como-funciona" : l === "Planos" ? "planos" : l === "Módulos" ? "modulos" : "";
                  if (id) document.getElementById(id)?.scrollIntoView({ behavior: "smooth" });
                }}
                style={{
                  fontFamily: sans, fontSize: "0.875rem", fontWeight: 500,
                  color: T.creamDim, textDecoration: "none",
                  transition: "color 0.2s",
                }}
                onMouseEnter={e => (e.currentTarget.style.color = T.cream)}
                onMouseLeave={e => (e.currentTarget.style.color = T.creamDim)}
              >
                {l}
              </a>
            ))}
          </div>
        </div>
        {/* Right */}
        <div style={{ display: "flex", alignItems: "center", gap: 18 }}>
          <button
            onClick={() => setMuted(m => !m)}
            style={{ background: "none", border: "none", color: T.creamDim, cursor: "pointer", padding: 4, display: "flex" }}
            title={muted ? "Ativar som" : "Silenciar"}
          >
            {muted ? <VolumeX size={18} /> : <Volume2 size={18} />}
          </button>
          <button style={{ background: "none", border: "none", color: T.creamDim, cursor: "pointer", padding: 4, display: "flex" }}>
            <Search size={18} />
          </button>
          <button style={{ background: "none", border: "none", color: T.creamDim, cursor: "pointer", padding: 4, display: "flex" }}>
            <Bell size={18} />
          </button>
          <a
            href="https://wa.me/5511958642044?text=Ol%C3%A1%2C%20quero%20saber%20mais%20sobre%20o%20Nexo"
            target="_blank" rel="noopener"
            style={{
              fontFamily: sans, fontWeight: 600, fontSize: "0.85rem",
              background: T.terra, color: T.cream,
              padding: "9px 20px", borderRadius: 6, textDecoration: "none",
              transition: "background 0.2s, transform 0.15s",
              display: "flex", alignItems: "center", gap: 8,
            }}
            onMouseEnter={e => { e.currentTarget.style.background = "#9F4A34"; e.currentTarget.style.transform = "translateY(-1px)"; }}
            onMouseLeave={e => { e.currentTarget.style.background = T.terra; e.currentTarget.style.transform = "none"; }}
          >
            Começar agora
          </a>
        </div>
      </div>
    </nav>
  );
}

// ─── Chat Demo ──────────────────────────────────────────────────────────────
const CHAT_INITIAL = [
  { dir: "out", text: "Oi! Você está falando com a Barbearia Nexo 💈 — funcionamos das 9h às 20h, de seg a sáb. Corte R$45, corte + barba R$65.", time: "21:46" },
  { dir: "in",  text: "Vocês abrem sábado?", time: "21:47" },
  { dir: "out", text: "Abrimos sim, das 9h às 13h. Quer reservar um horário?", time: "21:47" },
  { dir: "in",  text: "Pode ser às 10h", time: "21:48" },
  { dir: "out", text: "Fechado — 10h de sábado reservado. Te aviso um dia antes ✓", time: "21:48" },
];

const BOT_REPLIES: Record<string, string> = {
  default: "Posso te ajudar com agendamento, preços ou informações. O que deseja?",
  preco: "Corte: R$45. Corte + barba: R$65. Coloração: R$80. Quer agendar?",
  agendar: "Claro! Qual horário você prefere? Temos disponibilidade amanhã às 10h, 14h e 16h.",
  horario: "Funcionamos de segunda a sábado, das 9h às 20h. Domingo fechado.",
};

function matchReply(input: string) {
  const q = input.toLowerCase();
  if (q.includes("prec") || q.includes("valor") || q.includes("quanto")) return BOT_REPLIES.preco;
  if (q.includes("hor") || q.includes("funciona") || q.includes("abre")) return BOT_REPLIES.horario;
  if (q.includes("agenda") || q.includes("marcar") || q.includes("reserv")) return BOT_REPLIES.agendar;
  return BOT_REPLIES.default;
}

function ChatDemo() {
  const [messages, setMessages] = useState(CHAT_INITIAL);
  const [input, setInput] = useState("");
  const [typing, setTyping] = useState(false);
  const chatRef = useRef<HTMLDivElement>(null);

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
      setMessages(m => [...m, { dir: "out", text: matchReply(userMsg.text), time }]);
    }, 900);
  };

  useEffect(() => {
    if (chatRef.current) chatRef.current.scrollTop = chatRef.current.scrollHeight;
  }, [messages, typing]);

  return (
    <div style={{
      background: "#101720", borderRadius: 20, padding: 8,
      boxShadow: "0 40px 80px rgba(0,0,0,0.6), 0 0 0 1px rgba(184,86,61,0.15)",
      width: 300,
    }}>
      <div style={{ background: "#0B141A", borderRadius: 14, overflow: "hidden" }}>
        <div style={{ background: "#3E5C4A", color: "#fff", padding: "12px 16px", fontSize: 13, fontWeight: 600, display: "flex", alignItems: "center", gap: 10, fontFamily: sans }}>
          <div style={{ width: 8, height: 8, borderRadius: "50%", background: "#8FD4A8" }} />
          Nexo · online
        </div>
        <div ref={chatRef} style={{
          padding: "16px 12px", display: "flex", flexDirection: "column", gap: 10,
          minHeight: 240, maxHeight: 280, overflowY: "auto", background: "#0B141A",
          scrollbarWidth: "none",
        }}>
          {messages.map((m, i) => (
            <div key={i} style={{ alignSelf: m.dir === "out" ? "flex-start" : "flex-end", maxWidth: "82%" }}>
              <div style={{
                padding: "9px 13px", borderRadius: 12, fontSize: 12.5, lineHeight: 1.5, fontFamily: sans,
                background: m.dir === "out" ? "#1F2C34" : "#144D37",
                color: "#E9EDEF",
                borderTopLeftRadius: m.dir === "out" ? 2 : 12,
                borderTopRightRadius: m.dir === "in" ? 2 : 12,
              }}>
                {m.text}
                <div style={{ marginTop: 3, fontSize: 9.5, color: "rgba(233,237,239,0.4)", textAlign: "right" }}>{m.time}</div>
              </div>
            </div>
          ))}
          {typing && (
            <div style={{ alignSelf: "flex-start", maxWidth: "80%" }}>
              <div style={{ padding: "10px 14px", borderRadius: 12, borderTopLeftRadius: 2, background: "#1F2C34", display: "flex", gap: 4 }}>
                {[0,1,2].map(i => (
                  <span key={i} style={{ width: 5, height: 5, borderRadius: "50%", background: "rgba(233,237,239,0.7)", animation: `typingBlink 1.2s infinite ${i * 0.18}s`, display: "block" }} />
                ))}
              </div>
            </div>
          )}
        </div>
        <div style={{ display: "flex", gap: 8, padding: "9px 10px", background: "#101720", borderTop: "1px solid rgba(244,238,226,0.06)" }}>
          <input
            value={input}
            onChange={e => setInput(e.target.value)}
            onKeyDown={e => e.key === "Enter" && send()}
            placeholder="Ex: qual o valor do corte?"
            maxLength={120}
            style={{
              flex: 1, background: "#1F2C34", border: "1px solid rgba(244,238,226,0.08)",
              borderRadius: 100, padding: "8px 13px", fontFamily: sans,
              fontSize: 12, color: "#E9EDEF", outline: "none",
            }}
          />
          <button onClick={send} style={{
            width: 32, height: 32, flexShrink: 0, borderRadius: "50%", border: "none",
            background: "#3E5C4A", color: "#fff", fontSize: 13, cursor: "pointer",
            display: "flex", alignItems: "center", justifyContent: "center",
          }}>➤</button>
        </div>
      </div>
      <div style={{ fontSize: 10, color: "rgba(244,238,226,0.3)", textAlign: "center", marginTop: 8, fontFamily: mono }}>
        experimente digitar uma pergunta ↑
      </div>
    </div>
  );
}

// ─── Hero ───────────────────────────────────────────────────────────────────
function Hero() {
  const [showDemo, setShowDemo] = useState(false);

  return (
    <section style={{ position: "relative", height: "100vh", minHeight: 640, overflow: "hidden" }}>
      {/* Background cinematic image */}
      <div style={{
        position: "absolute", inset: 0,
        backgroundImage: "url('https://images.unsplash.com/photo-1536440136628-849c177e76a1?w=1800&h=900&fit=crop&auto=format')",
        backgroundSize: "cover",
        backgroundPosition: "center 30%",
        transform: "scale(1.04)",
      }} />
      {/* Cinematic gradient overlays */}
      <div style={{ position: "absolute", inset: 0, background: "linear-gradient(to right, rgba(9,13,18,0.95) 40%, rgba(9,13,18,0.5) 75%, rgba(9,13,18,0.2) 100%)" }} />
      <div style={{ position: "absolute", inset: 0, background: "linear-gradient(to top, rgba(9,13,18,1) 0%, transparent 50%)" }} />
      <div style={{ position: "absolute", inset: 0, background: "linear-gradient(to bottom, rgba(9,13,18,0.5) 0%, transparent 20%)" }} />
      {/* Terra radial accent */}
      <div style={{ position: "absolute", top: "20%", left: "38%", width: 600, height: 600, borderRadius: "50%", background: "radial-gradient(circle, rgba(184,86,61,0.12), transparent 70%)", pointerEvents: "none" }} />

      {/* Content */}
      <div style={{ position: "relative", zIndex: 2, height: "100%", display: "flex", flexDirection: "column", justifyContent: "flex-end", padding: "0 60px 120px", maxWidth: 1440, margin: "0 auto" }}>
        {/* Badge */}
        <div style={{ display: "flex", alignItems: "center", gap: 12, marginBottom: 20 }}>
          <div style={{ display: "flex", alignItems: "center", gap: 6, background: T.terra, padding: "4px 12px", borderRadius: 4 }}>
            <Star size={11} fill={T.cream} color={T.cream} />
            <span style={{ fontFamily: mono, fontSize: 11, fontWeight: 500, color: T.cream, letterSpacing: "0.12em", textTransform: "uppercase" }}>Destaque</span>
          </div>
          <span style={{ fontFamily: mono, fontSize: 11, color: T.creamFaint, letterSpacing: "0.1em", textTransform: "uppercase" }}>Automação · IA · PME</span>
        </div>

        {/* Title */}
        <h1 style={{
          fontFamily: serif, fontWeight: 900, color: T.cream,
          fontSize: "clamp(2.8rem, 6vw, 5.5rem)",
          lineHeight: 1.05, letterSpacing: "-0.02em",
          maxWidth: 700, marginBottom: 20,
          textShadow: "0 4px 30px rgba(0,0,0,0.5)",
        }}>
          Coloque a IA pra trabalhar no seu negócio
        </h1>

        {/* Description */}
        <p style={{
          fontFamily: sans, fontSize: "clamp(1rem, 1.3vw, 1.2rem)",
          color: T.creamDim, maxWidth: 520, lineHeight: 1.7, marginBottom: 34,
        }}>
          O Nexo responde no WhatsApp, organiza seus dados num painel simples e cria seu conteúdo. Você continua tocando o negócio — só com menos trabalho manual.
        </p>

        {/* CTA Buttons */}
        <div style={{ display: "flex", gap: 14, flexWrap: "wrap", alignItems: "center", marginBottom: 36 }}>
          <button
            onClick={() => setShowDemo(true)}
            style={{
              display: "flex", alignItems: "center", gap: 10,
              background: T.cream, color: T.bg,
              fontFamily: sans, fontWeight: 700, fontSize: "1rem",
              padding: "14px 28px", borderRadius: 8, border: "none", cursor: "pointer",
              transition: "background 0.2s, transform 0.15s",
              boxShadow: "0 8px 24px rgba(0,0,0,0.4)",
            }}
            onMouseEnter={e => { e.currentTarget.style.background = "#EEE6D4"; e.currentTarget.style.transform = "translateY(-2px)"; }}
            onMouseLeave={e => { e.currentTarget.style.background = T.cream; e.currentTarget.style.transform = "none"; }}
          >
            <Play size={18} fill={T.bg} color={T.bg} /> Ver demo ao vivo
          </button>
          <a
            href="https://wa.me/5511958642044?text=Ol%C3%A1%2C%20quero%20contratar%20o%20Nexo"
            target="_blank" rel="noopener"
            style={{
              display: "flex", alignItems: "center", gap: 10,
              background: "rgba(244,238,226,0.15)", color: T.cream,
              fontFamily: sans, fontWeight: 600, fontSize: "1rem",
              padding: "14px 28px", borderRadius: 8, textDecoration: "none",
              border: "1px solid rgba(244,238,226,0.3)",
              backdropFilter: "blur(8px)",
              transition: "background 0.2s, transform 0.15s",
            }}
            onMouseEnter={e => { e.currentTarget.style.background = "rgba(244,238,226,0.25)"; e.currentTarget.style.transform = "translateY(-2px)"; }}
            onMouseLeave={e => { e.currentTarget.style.background = "rgba(244,238,226,0.15)"; e.currentTarget.style.transform = "none"; }}
          >
            <Info size={18} /> Quero contratar
          </a>
        </div>

        {/* Meta pills */}
        <div style={{ display: "flex", gap: 20, flexWrap: "wrap" }}>
          {["✓ Ativa em 1 semana", "✓ Sem fidelidade", "✓ Suporte incluso 15 dias"].map(t => (
            <span key={t} style={{ fontFamily: mono, fontSize: 11.5, color: T.creamFaint, letterSpacing: "0.05em" }}>{t}</span>
          ))}
        </div>
      </div>

      {/* Demo Modal */}
      {showDemo && (
        <div
          style={{
            position: "fixed", inset: 0, zIndex: 100,
            background: "rgba(9,13,18,0.88)",
            backdropFilter: "blur(14px)",
            display: "flex", alignItems: "center", justifyContent: "center",
          }}
          onClick={() => setShowDemo(false)}
        >
          <div onClick={e => e.stopPropagation()} style={{ position: "relative" }}>
            <button
              onClick={() => setShowDemo(false)}
              style={{
                position: "absolute", top: -40, right: 0,
                background: "none", border: "none", color: T.creamDim,
                cursor: "pointer", display: "flex", alignItems: "center", gap: 6,
                fontFamily: sans, fontSize: 14,
              }}
            >
              <X size={18} /> Fechar
            </button>
            <ChatDemo />
          </div>
        </div>
      )}
    </section>
  );
}

// ─── Carousel ───────────────────────────────────────────────────────────────
interface CarouselCard {
  title: string;
  subtitle: string;
  tag: string;
  img: string;
  badge?: string;
  rating?: string;
  desc: string;
  cta?: string;
  href?: string;
  onClick?: () => void;
}

function Carousel({ title, cards, wide = false }: { title: string; cards: CarouselCard[]; wide?: boolean }) {
  const scrollRef = useRef<HTMLDivElement>(null);
  const [canLeft, setCanLeft] = useState(false);
  const [canRight, setCanRight] = useState(true);
  const [hoveredIdx, setHoveredIdx] = useState<number | null>(null);

  const checkScroll = useCallback(() => {
    const el = scrollRef.current;
    if (!el) return;
    setCanLeft(el.scrollLeft > 10);
    setCanRight(el.scrollLeft < el.scrollWidth - el.clientWidth - 10);
  }, []);

  const scroll = (dir: "left" | "right") => {
    const el = scrollRef.current;
    if (!el) return;
    const amount = wide ? 700 : 500;
    el.scrollBy({ left: dir === "left" ? -amount : amount, behavior: "smooth" });
  };

  const cardW = wide ? 380 : 220;
  const cardH = wide ? 214 : 310;

  return (
    <div style={{ marginBottom: 48, position: "relative" }}>
      {/* Row header */}
      <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", marginBottom: 16, padding: "0 48px" }}>
        <h2 style={{ fontFamily: serif, fontWeight: 700, fontSize: "1.4rem", color: T.cream, letterSpacing: "-0.01em" }}>{title}</h2>
        <div style={{ display: "flex", gap: 8 }}>
          <button
            onClick={() => scroll("left")}
            disabled={!canLeft}
            style={{
              width: 36, height: 36, borderRadius: "50%", border: `1px solid ${canLeft ? T.line : "rgba(184,86,61,0.08)"}`,
              background: canLeft ? "rgba(184,86,61,0.15)" : "rgba(255,255,255,0.03)",
              color: canLeft ? T.cream : T.creamFaint,
              cursor: canLeft ? "pointer" : "not-allowed", display: "flex", alignItems: "center", justifyContent: "center",
              transition: "all 0.2s",
            }}
          >
            <ChevronLeft size={18} />
          </button>
          <button
            onClick={() => scroll("right")}
            disabled={!canRight}
            style={{
              width: 36, height: 36, borderRadius: "50%", border: `1px solid ${canRight ? T.line : "rgba(184,86,61,0.08)"}`,
              background: canRight ? "rgba(184,86,61,0.15)" : "rgba(255,255,255,0.03)",
              color: canRight ? T.cream : T.creamFaint,
              cursor: canRight ? "pointer" : "not-allowed", display: "flex", alignItems: "center", justifyContent: "center",
              transition: "all 0.2s",
            }}
          >
            <ChevronRight size={18} />
          </button>
        </div>
      </div>

      {/* Scrollable track */}
      <div
        ref={scrollRef}
        onScroll={checkScroll}
        style={{
          display: "flex", gap: 14,
          overflowX: "auto", overflowY: "visible",
          scrollbarWidth: "none",
          padding: "8px 48px 24px",
          scrollSnapType: "x mandatory",
        }}
      >
        {cards.map((card, i) => (
          <div
            key={i}
            onMouseEnter={() => setHoveredIdx(i)}
            onMouseLeave={() => setHoveredIdx(null)}
            onClick={card.onClick}
            style={{
              flexShrink: 0,
              width: cardW,
              height: cardH,
              borderRadius: 10,
              overflow: "hidden",
              position: "relative",
              cursor: card.onClick || card.href ? "pointer" : "default",
              scrollSnapAlign: "start",
              transform: hoveredIdx === i ? "scale(1.05) translateY(-4px)" : "scale(1)",
              transition: "transform 0.35s cubic-bezier(.2,.8,.2,1), box-shadow 0.35s ease",
              boxShadow: hoveredIdx === i
                ? "0 24px 50px rgba(0,0,0,0.7), 0 0 0 1px rgba(184,86,61,0.3)"
                : "0 6px 20px rgba(0,0,0,0.4)",
              zIndex: hoveredIdx === i ? 5 : 1,
            }}
          >
            {/* Image */}
            <img
              src={card.img}
              alt={card.title}
              style={{ width: "100%", height: "100%", objectFit: "cover", display: "block" }}
            />

            {/* Base gradient */}
            <div style={{ position: "absolute", inset: 0, background: "linear-gradient(to top, rgba(9,13,18,0.95) 0%, rgba(9,13,18,0.4) 50%, transparent 100%)" }} />

            {/* Badge top-left */}
            {card.badge && (
              <div style={{
                position: "absolute", top: 10, left: 10,
                background: T.terra, color: T.cream,
                fontFamily: mono, fontSize: 9, fontWeight: 500,
                padding: "3px 8px", borderRadius: 4,
                letterSpacing: "0.12em", textTransform: "uppercase",
              }}>
                {card.badge}
              </div>
            )}

            {/* Bottom info */}
            <div style={{ position: "absolute", bottom: 0, left: 0, right: 0, padding: "0 14px 14px" }}>
              <div style={{ fontFamily: mono, fontSize: 9.5, color: T.terra, letterSpacing: "0.15em", textTransform: "uppercase", marginBottom: 4 }}>
                {card.tag}
              </div>
              <div style={{ fontFamily: serif, fontWeight: 700, fontSize: wide ? "1.1rem" : "0.95rem", color: T.cream, lineHeight: 1.2, marginBottom: 4 }}>
                {card.title}
              </div>

              {/* Hover info */}
              <div style={{
                overflow: "hidden",
                maxHeight: hoveredIdx === i ? 120 : 0,
                opacity: hoveredIdx === i ? 1 : 0,
                transition: "max-height 0.3s ease, opacity 0.25s ease",
              }}>
                <div style={{ fontFamily: sans, fontSize: 11.5, color: T.creamDim, lineHeight: 1.5, marginBottom: 10, marginTop: 4 }}>
                  {card.desc}
                </div>
                {card.rating && (
                  <div style={{ display: "flex", alignItems: "center", gap: 4, marginBottom: 10 }}>
                    {[1,2,3,4,5].map(s => (
                      <Star key={s} size={10} fill={s <= parseFloat(card.rating!) ? T.terra : "transparent"} color={T.terra} />
                    ))}
                    <span style={{ fontFamily: mono, fontSize: 10, color: T.creamFaint, marginLeft: 4 }}>{card.rating}</span>
                  </div>
                )}
                {card.cta && (
                  <a
                    href={card.href || "#"}
                    target={card.href ? "_blank" : undefined}
                    rel="noopener"
                    onClick={e => { if (card.onClick) { e.preventDefault(); card.onClick!(); } }}
                    style={{
                      display: "inline-flex", alignItems: "center", gap: 6,
                      background: T.terra, color: T.cream,
                      fontFamily: sans, fontWeight: 600, fontSize: 11,
                      padding: "7px 14px", borderRadius: 6, textDecoration: "none",
                      transition: "background 0.2s",
                    }}
                    onMouseEnter={e => (e.currentTarget.style.background = "#9F4A34")}
                    onMouseLeave={e => (e.currentTarget.style.background = T.terra)}
                  >
                    <Play size={11} fill={T.cream} color={T.cream} /> {card.cta}
                  </a>
                )}
              </div>

              {!hoveredIdx && <div style={{ fontFamily: sans, fontSize: 11, color: T.creamFaint }}>{card.subtitle}</div>}
            </div>
          </div>
        ))}
      </div>

      {/* Edge fades */}
      <div style={{ position: "absolute", left: 0, top: 0, bottom: 0, width: 60, background: `linear-gradient(to right, ${T.bg}, transparent)`, pointerEvents: "none", zIndex: 3 }} />
      <div style={{ position: "absolute", right: 0, top: 0, bottom: 0, width: 60, background: `linear-gradient(to left, ${T.bg}, transparent)`, pointerEvents: "none", zIndex: 3 }} />
    </div>
  );
}

// ─── Dashboard KPI embed ─────────────────────────────────────────────────────
const CHART_VALS = [28, 41, 35, 58, 72, 89];
const CHART_MONTHS = ["Jan", "Fev", "Mar", "Abr", "Mai", "Jun"];

function DashboardModal({ onClose }: { onClose: () => void }) {
  const [chartMode, setChartMode] = useState<"bar" | "line">("bar");
  const maxVal = Math.max(...CHART_VALS);

  return (
    <div
      style={{ position: "fixed", inset: 0, zIndex: 100, background: "rgba(9,13,18,0.9)", backdropFilter: "blur(14px)", display: "flex", alignItems: "center", justifyContent: "center", padding: 24 }}
      onClick={onClose}
    >
      <div onClick={e => e.stopPropagation()} style={{ width: "100%", maxWidth: 640, position: "relative" }}>
        <button onClick={onClose} style={{ position: "absolute", top: -40, right: 0, background: "none", border: "none", color: T.creamDim, cursor: "pointer", display: "flex", alignItems: "center", gap: 6, fontFamily: sans, fontSize: 14 }}>
          <X size={18} /> Fechar
        </button>
        <div style={{ background: "#1B222B", borderRadius: 12, overflow: "hidden", border: `1px solid ${T.line}`, fontFamily: sans }}>
          <div style={{ background: "#232C37", borderBottom: `1px solid ${T.line}`, padding: "12px 18px", display: "flex", alignItems: "center", gap: 10 }}>
            <div style={{ width: 8, height: 8, borderRadius: "50%", background: "#4ade80", boxShadow: "0 0 6px #4ade80" }} />
            <span style={{ fontSize: 13, fontWeight: 700, color: T.cream }}>Dashboard · Nexo</span>
            <span style={{ marginLeft: "auto", fontSize: 10, color: "rgba(244,238,226,0.35)", fontFamily: mono }}>ao vivo</span>
          </div>
          <div style={{ padding: 18 }}>
            <div style={{ display: "grid", gridTemplateColumns: "repeat(4,1fr)", gap: 10, marginBottom: 16 }}>
              {[
                { label: "Receita", val: "R$ 11.940", sub: "↑ 23%", c: "#4ade80" },
                { label: "Clientes", val: "30", sub: "↑ 18 novos", c: "#4ade80" },
                { label: "Tarefas", val: "7", sub: "4 pendentes", c: T.terraS },
                { label: "Score", val: "9.2", sub: "Excelente", c: T.terraS, accent: true },
              ].map(k => (
                <div key={k.label} style={{
                  background: (k as any).accent ? "linear-gradient(135deg,rgba(184,86,61,.5),rgba(159,74,52,.2))" : "#232C37",
                  border: `1px solid ${(k as any).accent ? "rgba(184,86,61,.35)" : T.line}`,
                  borderRadius: 10, padding: "12px 14px",
                }}>
                  <div style={{ fontSize: 9, textTransform: "uppercase", letterSpacing: "0.18em", color: "rgba(244,238,226,0.35)", marginBottom: 6 }}>{k.label}</div>
                  <div style={{ fontSize: 18, fontWeight: 700, color: T.cream }}>{k.val}</div>
                  <div style={{ fontSize: 10, color: k.c, marginTop: 4 }}>{k.sub}</div>
                </div>
              ))}
            </div>
            <div style={{ background: "#232C37", border: `1px solid ${T.line}`, borderRadius: 10, padding: "14px 16px" }}>
              <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 14 }}>
                <div>
                  <div style={{ fontSize: 12, fontWeight: 600, color: "rgba(244,238,226,0.6)" }}>Receita mensal</div>
                  <div style={{ fontSize: 10, color: "rgba(244,238,226,0.32)" }}>Jan – Jun 2025 (R$ mil)</div>
                </div>
                <div style={{ display: "flex", gap: 6 }}>
                  {(["bar","line"] as const).map(m => (
                    <button key={m} onClick={() => setChartMode(m)} style={{
                      fontSize: 10, padding: "4px 10px", borderRadius: 6,
                      border: `1px solid ${chartMode === m ? T.terra : T.line}`,
                      background: chartMode === m ? T.terra : "transparent",
                      color: chartMode === m ? T.cream : "rgba(244,238,226,0.35)",
                      cursor: "pointer", fontFamily: sans,
                    }}>
                      {m === "bar" ? "Barras" : "Linha"}
                    </button>
                  ))}
                </div>
              </div>
              {chartMode === "bar" ? (
                <div style={{ display: "flex", alignItems: "flex-end", gap: 8, height: 90 }}>
                  {CHART_VALS.map((v, i) => (
                    <div key={i} style={{ flex: 1, display: "flex", flexDirection: "column", justifyContent: "flex-end", alignItems: "center", height: "100%", gap: 4 }}>
                      <div style={{ width: "100%", borderRadius: "4px 4px 0 0", height: Math.round((v / maxVal) * 80), background: `linear-gradient(to top, ${T.terra}, ${T.terraS})`, transition: "height 0.6s cubic-bezier(.34,1.2,.64,1)" }} />
                      <div style={{ fontSize: 9, color: "rgba(244,238,226,0.32)" }}>{CHART_MONTHS[i]}</div>
                    </div>
                  ))}
                </div>
              ) : (
                <div>
                  <svg viewBox="0 0 100 100" preserveAspectRatio="none" style={{ width: "100%", height: 80, overflow: "visible" }}>
                    <polygon points={`0,100 ${CHART_VALS.map((v, i) => `${i*(100/(CHART_VALS.length-1))},${100-(v/maxVal)*100}`).join(" ")} 100,100`} fill="rgba(184,86,61,0.2)" />
                    <polyline points={CHART_VALS.map((v, i) => `${i*(100/(CHART_VALS.length-1))},${100-(v/maxVal)*100}`).join(" ")} fill="none" stroke={T.terraS} strokeWidth="2" vectorEffect="non-scaling-stroke" />
                    {CHART_VALS.map((v, i) => <circle key={i} cx={i*(100/(CHART_VALS.length-1))} cy={100-(v/maxVal)*100} r="3" fill={T.terraS} vectorEffect="non-scaling-stroke" />)}
                  </svg>
                  <div style={{ display: "flex" }}>
                    {CHART_MONTHS.map(m => <div key={m} style={{ flex: 1, textAlign: "center", fontSize: 9, color: "rgba(244,238,226,0.32)", marginTop: 4 }}>{m}</div>)}
                  </div>
                </div>
              )}
            </div>
          </div>
        </div>
      </div>
    </div>
  );
}

// ─── FAQ Accordion ──────────────────────────────────────────────────────────
function FAQ() {
  const [open, setOpen] = useState<number | null>(null);

  const items = [
    { q: "Preciso entender de tecnologia para usar o Nexo?", a: "Não. O Nexo foi pensado para donos de negócio, não para desenvolvedores. Nossa equipe configura tudo e você recebe tudo pronto para usar." },
    { q: "Quanto tempo leva para o Nexo entrar no ar?", a: "Em média 5 a 7 dias úteis após a conversa de onboarding. Casos mais complexos podem levar até 2 semanas." },
    { q: "O que acontece depois dos 15 dias de suporte?", a: "Você continua com acesso ao suporte por e-mail incluído no plano. O suporte WhatsApp dedicado é exclusivo do plano Pro." },
    { q: "Posso cancelar a qualquer momento?", a: "Sim. Não há contrato de fidelidade. Você cancela quando quiser, sem multa." },
    { q: "O Nexo funciona para qualquer tipo de negócio?", a: "Funciona para a maioria das PMEs: barbearias, clínicas, lojas, restaurantes, escritórios, salões e mais. Se tiver dúvida sobre o seu setor, fale com a gente." },
  ];

  return (
    <section style={{ padding: "80px 48px", maxWidth: 900, margin: "0 auto" }}>
      <Reveal>
        <div style={{ display: "flex", alignItems: "center", gap: 14, marginBottom: 40 }}>
          <div style={{ width: 3, height: 28, background: T.terra, borderRadius: 2 }} />
          <h2 style={{ fontFamily: serif, fontWeight: 700, fontSize: "1.8rem", color: T.cream }}>Perguntas Frequentes</h2>
        </div>
      </Reveal>
      {items.map((item, i) => (
        <Reveal key={i} delay={i * 0.05}>
          <div style={{ borderBottom: `1px solid ${T.line}` }}>
            <button
              onClick={() => setOpen(open === i ? null : i)}
              style={{
                display: "flex", justifyContent: "space-between", alignItems: "center",
                width: "100%", padding: "20px 0", background: "none", border: "none",
                cursor: "pointer", gap: 20, textAlign: "left",
              }}
            >
              <span style={{ fontFamily: sans, fontWeight: 600, fontSize: "1rem", color: T.cream }}>{item.q}</span>
              <span style={{ fontFamily: serif, fontSize: "1.5rem", color: T.terra, flexShrink: 0, display: "inline-block", transform: open === i ? "rotate(45deg)" : "none", transition: "transform 0.2s" }}>+</span>
            </button>
            <div style={{ overflow: "hidden", maxHeight: open === i ? 200 : 0, transition: "max-height 0.35s ease", opacity: open === i ? 1 : 0 }}>
              <p style={{ paddingBottom: 22, color: T.creamDim, fontSize: "0.95rem", fontFamily: sans, lineHeight: 1.7 }}>{item.a}</p>
            </div>
          </div>
        </Reveal>
      ))}
    </section>
  );
}

// ─── Footer ─────────────────────────────────────────────────────────────────
function Footer() {
  return (
    <footer style={{ borderTop: `1px solid ${T.line}`, padding: "44px 48px", textAlign: "center" }}>
      <div style={{ maxWidth: 1440, margin: "0 auto" }}>
        <div style={{ fontFamily: serif, fontSize: "1.4rem", fontWeight: 700, color: T.cream, marginBottom: 20 }}>
          Ne<span style={{ color: T.terra, fontStyle: "italic" }}>x</span>o
        </div>
        <div style={{ display: "flex", justifyContent: "center", gap: 32, flexWrap: "wrap", marginBottom: 28 }}>
          {["Plataforma", "Como funciona", "Preços", "Perguntas"].map(l => (
            <a key={l} href="#" style={{ fontFamily: sans, fontSize: "0.85rem", color: T.creamFaint, textDecoration: "none", transition: "color 0.2s" }}
              onMouseEnter={e => (e.currentTarget.style.color = T.cream)}
              onMouseLeave={e => (e.currentTarget.style.color = T.creamFaint)}>
              {l}
            </a>
          ))}
        </div>
        <div style={{ fontFamily: mono, fontSize: 11, color: "rgba(244,238,226,0.2)", letterSpacing: "0.08em" }}>
          © {new Date().getFullYear()} Nexo · Automação Inteligente para PMEs
        </div>
      </div>
    </footer>
  );
}

// ─── Floating WhatsApp ───────────────────────────────────────────────────────
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
        position: "fixed", bottom: 28, right: 28, zIndex: 60,
        width: 54, height: 54, borderRadius: "50%",
        background: T.terra, display: "flex", alignItems: "center", justifyContent: "center",
        boxShadow: `0 12px 28px ${T.terraGlow}`,
        opacity: visible ? 1 : 0,
        transform: visible ? "scale(1)" : "scale(0.7)",
        pointerEvents: visible ? "auto" : "none",
        transition: "opacity 0.3s ease, transform 0.3s ease, background 0.2s",
      }}
      onMouseEnter={e => (e.currentTarget.style.background = "#9F4A34")}
      onMouseLeave={e => (e.currentTarget.style.background = T.terra)}
    >
      <span style={{ position: "absolute", inset: 0, borderRadius: "50%", animation: "waPulse 2.4s infinite" }} />
      <svg width="26" height="26" viewBox="0 0 24 24" fill="none" style={{ position: "relative" }}>
        <path d="M9.1 7.05c-.2-.45-.4-.46-.6-.47h-.5c-.18 0-.46.07-.7.33-.24.26-.92.9-.92 2.2 0 1.29.94 2.54 1.07 2.72.13.18 1.83 2.94 4.52 4 2.24.88 2.7.7 3.18.66.49-.04 1.58-.64 1.8-1.27.23-.62.23-1.15.16-1.27-.07-.11-.25-.18-.5-.3-.27-.13-1.58-.78-1.82-.87-.25-.09-.43-.13-.6.14-.19.27-.7.87-.86 1.05-.16.18-.31.2-.58.07-.27-.14-1.13-.42-2.15-1.34-.8-.71-1.33-1.58-1.49-1.85-.15-.27-.02-.42.12-.55.12-.13.27-.32.4-.48.13-.16.18-.27.27-.46.09-.18.05-.34-.02-.48-.07-.14-.6-1.49-.83-2.04Z" fill={T.cream} />
        <path d="M12.04 2C6.58 2 2.13 6.45 2.13 11.91c0 1.75.46 3.45 1.32 4.95L2 22l5.28-1.38a9.9 9.9 0 0 0 4.76 1.21h.01c5.46 0 9.91-4.45 9.91-9.92C21.96 6.45 17.5 2 12.04 2Zm0 18.06h-.01a8.2 8.2 0 0 1-4.19-1.15l-.3-.18-3.13.82.84-3.05-.2-.31a8.24 8.24 0 0 1-1.27-4.28c0-4.55 3.71-8.26 8.27-8.26 2.21 0 4.28.86 5.84 2.42a8.2 8.2 0 0 1 2.42 5.84c0 4.56-3.71 8.15-8.27 8.15Z" fill={T.cream} />
      </svg>
    </a>
  );
}

// ─── App ───────────────────────────────────────────────────────────────────
export default function App() {
  const [showDashboard, setShowDashboard] = useState(false);

  const modulesCards: CarouselCard[] = [
    {
      title: "Atendimento Automatizado",
      subtitle: "WhatsApp · IA",
      tag: "Módulo Principal",
      badge: "Novo",
      img: "https://images.unsplash.com/photo-1611746872915-64382b5c76da?w=400&h=600&fit=crop&auto=format",
      rating: "4.9",
      desc: "Responde no WhatsApp, informa preços e agenda horários — mesmo quando você está ocupado ou dormindo.",
      cta: "Ver demo",
      onClick: () => document.getElementById("chat-demo")?.scrollIntoView({ behavior: "smooth" }),
    },
    {
      title: "Painel de Gestão",
      subtitle: "Dashboard · Dados",
      tag: "Módulo Pro",
      badge: "Popular",
      img: "https://images.unsplash.com/photo-1551288049-bebda4e38f71?w=400&h=600&fit=crop&auto=format",
      rating: "4.8",
      desc: "Controle financeiro, cadastro de clientes e relatórios simples de como o negócio está indo.",
      cta: "Ver painel",
      onClick: () => setShowDashboard(true),
    },
    {
      title: "Motor de Conteúdo",
      subtitle: "IA · Redes Sociais",
      tag: "Módulo Creator",
      img: "https://images.unsplash.com/photo-1432821596592-e2c18b78144f?w=400&h=600&fit=crop&auto=format",
      rating: "4.7",
      desc: "Cria posts, legendas e anúncios em minutos. O Nexo aprende sua voz e gera conteúdo alinhado com sua marca.",
      cta: "Saiba mais",
      href: "https://wa.me/5511958642044?text=Ol%C3%A1%2C%20quero%20saber%20mais%20sobre%20o%20Nexo",
    },
    {
      title: "Integração Completa",
      subtitle: "API · Automação",
      tag: "Business",
      img: "https://images.unsplash.com/photo-1558494949-ef010cbdcc31?w=400&h=600&fit=crop&auto=format",
      rating: "5.0",
      desc: "Módulos personalizados, integrações sob medida e conta dedicada para empresas em crescimento acelerado.",
      cta: "Falar com equipe",
      href: "https://wa.me/5511958642044?text=Ol%C3%A1%2C%20quero%20saber%20mais%20sobre%20o%20Nexo+Business",
    },
    {
      title: "Relatórios Inteligentes",
      subtitle: "Analytics · IA",
      tag: "Em breve",
      img: "https://images.unsplash.com/photo-1460925895917-afdab827c52f?w=400&h=600&fit=crop&auto=format",
      rating: "4.9",
      desc: "Insights automáticos sobre seu negócio, gerados por IA toda semana no seu WhatsApp.",
      cta: "Avise-me",
      href: "https://wa.me/5511958642044",
    },
  ];

  const testimonialsCards: CarouselCard[] = [
    {
      title: '"Meus clientes não ficam sem resposta"',
      subtitle: "Marcos T. · Barbearia, SP",
      tag: "Depoimento",
      img: "https://images.unsplash.com/photo-1503951914875-452162b0f3f1?w=600&h=340&fit=crop&auto=format",
      rating: "5.0",
      desc: "Antes eu perdia cliente porque não dava conta de responder no WhatsApp à noite. Hoje o Nexo responde na hora.",
    },
    {
      title: '"Mudei meu preço no mesmo mês"',
      subtitle: "Renata A. · Clínica, BH",
      tag: "Depoimento",
      img: "https://images.unsplash.com/photo-1629425702073-9f59e8ad1e0f?w=600&h=340&fit=crop&auto=format",
      rating: "4.9",
      desc: "O painel me mostrou que eu tava perdendo dinheiro num serviço que achava que era o mais lucrativo.",
    },
    {
      title: '"Instagram ativo toda semana"',
      subtitle: "Diego S. · Loja, Curitiba",
      tag: "Depoimento",
      img: "https://images.unsplash.com/photo-1441986300917-64674bd600d8?w=600&h=340&fit=crop&auto=format",
      rating: "4.8",
      desc: "Não tenho tempo pra pensar em legenda de post. O Nexo escreve, eu só reviso e publico.",
    },
    {
      title: '"Configurado em 4 dias"',
      subtitle: "Ana L. · Salão, Rio",
      tag: "Depoimento",
      img: "https://images.unsplash.com/photo-1560066984-138dadb4c035?w=600&h=340&fit=crop&auto=format",
      rating: "5.0",
      desc: "Achei que ia ser complicado mas a equipe do Nexo configurou tudo e já estava funcionando na semana.",
    },
  ];

  const stepsCards: CarouselCard[] = [
    {
      title: "Conversa Inicial",
      subtitle: "30 minutos · Online",
      tag: "Passo 1",
      img: "https://images.unsplash.com/photo-1600880292203-757bb62b4baf?w=600&h=340&fit=crop&auto=format",
      desc: "Você nos conta sobre o seu negócio: o que faz, quem são seus clientes e as maiores dores do dia a dia.",
      cta: "Agendar agora",
      href: "https://wa.me/5511958642044?text=Ol%C3%A1%2C%20quero%20contratar%20o%20Nexo",
    },
    {
      title: "Configuração Completa",
      subtitle: "Feita por nós · Sem tecnês",
      tag: "Passo 2",
      img: "https://images.unsplash.com/photo-1517245386807-bb43f82c33c4?w=600&h=340&fit=crop&auto=format",
      desc: "Nossa equipe monta o atendimento, o painel e os conteúdos com base no que você nos passou. Sem planilhas.",
    },
    {
      title: "Ativa em 1 Semana",
      subtitle: "7 dias · Garantido",
      tag: "Passo 3",
      img: "https://images.unsplash.com/photo-1553877522-43269d4ea984?w=600&h=340&fit=crop&auto=format",
      desc: "Em 7 dias o Nexo já está no ar respondendo clientes, organizando dados e publicando conteúdo. Você só aprova.",
    },
    {
      title: "Suporte & Ajustes",
      subtitle: "15 dias · Incluso",
      tag: "Passo 4",
      img: "https://images.unsplash.com/photo-1521791136064-7986c2920216?w=600&h=340&fit=crop&auto=format",
      desc: "Os primeiros 15 dias vêm com suporte direto para qualquer dúvida ou ajuste fino. Você nunca fica sozinho.",
    },
  ];

  const planosCards: CarouselCard[] = [
    {
      title: "Starter — R$79/mês",
      subtitle: "1 módulo · E-mail support",
      tag: "Plano",
      img: "https://images.unsplash.com/photo-1507003211169-0a1dd7228f2d?w=400&h=600&fit=crop&auto=format",
      desc: "1 módulo à escolha. Suporte por e-mail. Ideal pra quem quer testar o Nexo sem compromisso.",
      cta: "Começar",
      href: "https://wa.me/5511958642044?text=Ol%C3%A1%2C%20quero%20o%20plano%20Starter",
    },
    {
      title: "Pro — R$149/mês",
      subtitle: "3 módulos · WhatsApp support",
      tag: "Mais Popular",
      badge: "★ Top",
      img: "https://images.unsplash.com/photo-1519389950473-47ba0277781c?w=400&h=600&fit=crop&auto=format",
      desc: "Os 3 módulos completos. Suporte WhatsApp. Onboarding dedicado. O mais escolhido pelas PMEs.",
      cta: "Escolher Pro",
      href: "https://wa.me/5511958642044?text=Ol%C3%A1%2C%20quero%20o%20plano%20Pro",
    },
    {
      title: "Business — Sob consulta",
      subtitle: "Custom · Conta dedicada",
      tag: "Enterprise",
      img: "https://images.unsplash.com/photo-1486406146926-c627a92ad1ab?w=400&h=600&fit=crop&auto=format",
      desc: "Módulos personalizados, integrações sob medida e conta dedicada para negócios em crescimento acelerado.",
      cta: "Falar com a equipe",
      href: "https://wa.me/5511958642044?text=Ol%C3%A1%2C%20quero%20saber%20sobre%20o%20Business",
    },
  ];

  return (
    <div style={{ background: T.bg, color: T.cream, fontFamily: sans, lineHeight: 1.6, minHeight: "100vh", WebkitFontSmoothing: "antialiased" }}>
      <style>{`
        ::selection { background: #B8563D; color: #F4EEE2; }
        * { box-sizing: border-box; }
        ::-webkit-scrollbar { width: 3px; height: 3px; }
        ::-webkit-scrollbar-thumb { background: rgba(184,86,61,0.3); border-radius: 3px; }
        a:focus-visible, button:focus-visible { outline: 2px solid #B8563D; outline-offset: 3px; border-radius: 4px; }
        @keyframes typingBlink { 0%, 80%, 100% { opacity: .25; } 40% { opacity: 1; } }
        @keyframes waPulse {
          0% { box-shadow: 0 0 0 0 rgba(184,86,61,0.55); }
          70% { box-shadow: 0 0 0 14px rgba(184,86,61,0); }
          100% { box-shadow: 0 0 0 0 rgba(184,86,61,0); }
        }
        @media (prefers-reduced-motion: reduce) {
          * { animation: none !important; transition: none !important; }
        }
        @media (max-width: 900px) {
          .nav-links { display: none !important; }
        }
        @media (max-width: 640px) {
          nav { padding: 0 20px !important; }
        }
      `}</style>

      <Nav />
      <Hero />

      {/* Carousels */}
      <div style={{ paddingTop: 48 }}>
        <div id="modulos">
          <Carousel title="Módulos em Destaque" cards={modulesCards} />
        </div>

        {/* Chat demo inline */}
        <section id="chat-demo" style={{ padding: "20px 48px 60px" }}>
          <Reveal>
            <div style={{ display: "flex", alignItems: "center", gap: 14, marginBottom: 32 }}>
              <div style={{ width: 3, height: 28, background: T.terra, borderRadius: 2 }} />
              <h2 style={{ fontFamily: serif, fontWeight: 700, fontSize: "1.4rem", color: T.cream }}>Experimente ao Vivo</h2>
            </div>
          </Reveal>
          <div style={{ display: "flex", flexWrap: "wrap", gap: 48, alignItems: "center" }}>
            <ChatDemo />
            <div style={{ flex: 1, minWidth: 260 }}>
              <Reveal>
                <div style={{ fontFamily: mono, fontSize: 11, letterSpacing: "0.15em", textTransform: "uppercase", color: T.terra, marginBottom: 14 }}>
                  Atendimento 24h
                </div>
                <h3 style={{ fontFamily: serif, fontWeight: 700, fontSize: "2rem", color: T.cream, lineHeight: 1.15, marginBottom: 16 }}>
                  Responde no WhatsApp enquanto você dorme
                </h3>
                <p style={{ color: T.creamDim, fontSize: "1rem", lineHeight: 1.7, marginBottom: 24, maxWidth: 440 }}>
                  O Nexo tira dúvidas, informa preços, agenda horários e até confirma pagamentos. Tudo automático, com a sua voz.
                </p>
                <div style={{ display: "flex", flexDirection: "column", gap: 12 }}>
                  {[
                    "Agendamento automático via WhatsApp",
                    "Respostas personalizadas para o seu negócio",
                    "Confirmações e lembretes enviados pelo bot",
                  ].map(f => (
                    <div key={f} style={{ display: "flex", alignItems: "center", gap: 10, fontFamily: sans, fontSize: "0.9rem", color: T.creamDim }}>
                      <span style={{ color: T.terra, fontSize: "1rem" }}>✓</span> {f}
                    </div>
                  ))}
                </div>
              </Reveal>
            </div>
          </div>
        </section>

        <div id="como-funciona">
          <Carousel title="Como Funciona — 4 Passos" cards={stepsCards} wide />
        </div>

        <Carousel title="O Que Nossos Clientes Dizem" cards={testimonialsCards} wide />

        <div id="planos">
          <Carousel title="Escolha Seu Plano" cards={planosCards} />
        </div>
      </div>

      {/* FAQ */}
      <FAQ />

      {/* Final CTA */}
      <section style={{
        margin: "0 48px 60px", borderRadius: 16, overflow: "hidden", position: "relative",
        background: "linear-gradient(135deg, #1B222B 0%, #131920 60%)",
        border: `1px solid ${T.line}`,
      }}>
        <div style={{ position: "absolute", inset: 0, backgroundImage: "url('https://images.unsplash.com/photo-1524178232363-1fb2b075b655?w=1200&h=400&fit=crop&auto=format')", backgroundSize: "cover", backgroundPosition: "center", opacity: 0.12 }} />
        <div style={{ position: "absolute", top: 0, left: 0, right: 0, height: 3, background: T.terra }} />
        <div style={{ position: "relative", zIndex: 1, padding: "64px 60px", textAlign: "center" }}>
          <Reveal>
            <div style={{ fontFamily: mono, fontSize: 11, letterSpacing: "0.2em", textTransform: "uppercase", color: T.terraS, marginBottom: 16 }}>
              Pronto para começar?
            </div>
            <h2 style={{ fontFamily: serif, fontWeight: 900, fontSize: "clamp(2rem, 4vw, 3.2rem)", color: T.cream, lineHeight: 1.1, letterSpacing: "-0.02em", marginBottom: 18 }}>
              Automatize seu negócio.<br />
              <span style={{ color: T.terra, fontStyle: "italic" }}>Sem complicação.</span>
            </h2>
            <p style={{ fontFamily: sans, fontSize: "1.05rem", color: T.creamDim, maxWidth: 440, margin: "0 auto 32px" }}>
              Em 1 semana o Nexo já está respondendo clientes, organizando dados e criando conteúdo por você.
            </p>
            <div style={{ display: "flex", gap: 14, justifyContent: "center", flexWrap: "wrap" }}>
              <a
                href="https://wa.me/5511958642044?text=Ol%C3%A1%2C%20quero%20contratar%20o%20Nexo"
                target="_blank" rel="noopener"
                style={{
                  display: "inline-flex", alignItems: "center", gap: 10,
                  background: T.terra, color: T.cream,
                  fontFamily: sans, fontWeight: 700, fontSize: "1rem",
                  padding: "15px 30px", borderRadius: 8, textDecoration: "none",
                  transition: "background 0.2s, transform 0.15s",
                  boxShadow: `0 8px 24px ${T.terraGlow}`,
                }}
                onMouseEnter={e => { e.currentTarget.style.background = "#9F4A34"; e.currentTarget.style.transform = "translateY(-2px)"; }}
                onMouseLeave={e => { e.currentTarget.style.background = T.terra; e.currentTarget.style.transform = "none"; }}
              >
                <Play size={18} fill={T.cream} color={T.cream} /> Quero contratar agora
              </a>
              <a
                href="https://wa.me/5511958642044?text=Ol%C3%A1%2C%20tenho%20d%C3%BAvidas%20sobre%20o%20Nexo"
                target="_blank" rel="noopener"
                style={{
                  display: "inline-flex", alignItems: "center", gap: 10,
                  color: T.cream, fontFamily: sans, fontWeight: 600, fontSize: "1rem",
                  padding: "15px 26px", borderRadius: 8, textDecoration: "none",
                  border: `1px solid rgba(244,238,226,0.3)`,
                  transition: "border-color 0.2s, transform 0.15s",
                }}
                onMouseEnter={e => { e.currentTarget.style.borderColor = T.cream; e.currentTarget.style.transform = "translateY(-2px)"; }}
                onMouseLeave={e => { e.currentTarget.style.borderColor = "rgba(244,238,226,0.3)"; e.currentTarget.style.transform = "none"; }}
              >
                Tirar dúvidas
              </a>
            </div>
          </Reveal>
        </div>
      </section>

      <Footer />
      <FloatingWhatsApp />

      {showDashboard && <DashboardModal onClose={() => setShowDashboard(false)} />}
    </div>
  );
}
