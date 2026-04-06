# Simulador-el-trico-
Simulador educativo de elétrica residencial
import { useState, useEffect } from "react";

export default function App() {
  const [screen, setScreen] = useState("home");
  const [progress, setProgress] = useState({
    completedLevels: [],
    totalScore: 0,
    currentLevel: 1,
  });
  const [circuitState, setCircuitState] = useState({
    components: [],
    isCorrect: false,
  });

  // Carregar progresso do localStorage
  useEffect(() => {
    const saved = localStorage.getItem("simuladorProgress");
    if (saved) {
      setProgress(JSON.parse(saved));
    }
  }, []);

  // Salvar progresso
  const saveProgress = (newProgress) => {
    setProgress(newProgress);
    localStorage.setItem("simuladorProgress", JSON.stringify(newProgress));
  };

  const Container = ({ children }) => (
    <div
      style={{
        minHeight: "100vh",
        display: "flex",
        alignItems: "center",
        justifyContent: "center",
        background: "linear-gradient(135deg, #0f172a, #1e293b)",
        color: "white",
        fontFamily: "sans-serif",
        padding: "20px",
      }}
    >
      <div
        style={{
          width: "100%",
          maxWidth: "420px",
          background: "#020617",
          borderRadius: "20px",
          padding: "30px",
          boxShadow: "0 10px 30px rgba(0,0,0,0.5)",
        }}
      >
        {children}
      </div>
    </div>
  );

  const Button = ({ children, onClick, secondary, disabled }) => (
    <button
      onClick={onClick}
      disabled={disabled}
      style={{
        width: "100%",
        padding: "12px",
        marginTop: "10px",
        borderRadius: "12px",
        border: "none",
        background: disabled
          ? "#475569"
          : secondary
            ? "#334155"
            : "#2563eb",
        color: disabled ? "#94a3b8" : "white",
        fontWeight: "bold",
        cursor: disabled ? "not-allowed" : "pointer",
        opacity: disabled ? 0.6 : 1,
        transition: "all 0.3s ease",
      }}
    >
      {children}
    </button>
  );

  const levels = [
    {
      id: 1,
      name: "Lâmpada",
      icon: "🔌",
      description: "Monte um circuito simples com bateria e lâmpada",
      correctComponents: ["battery", "lamp"],
    },
    {
      id: 2,
      name: "Interruptor",
      icon: "💡",
      description: "Adicione um interruptor ao circuito",
      correctComponents: ["battery", "switch", "lamp"],
    },
    {
      id: 3,
      name: "Cômodo",
      icon: "🏠",
      description: "Controle múltiplas lâmpadas em um cômodo",
      correctComponents: ["battery", "switch", "lamp", "lamp"],
    },
    {
      id: 4,
      name: "Tomadas",
      icon: "🔒",
      description: "Instale tomadas residenciais",
      correctComponents: ["battery", "outlet"],
    },
    {
      id: 5,
      name: "Chuveiro",
      icon: "🔒",
      description: "Circuito de chuveiro elétrico",
      correctComponents: ["battery", "breaker", "heater"],
    },
    {
      id: 6,
      name: "Three Way",
      icon: "🔒",
      description: "Controle de dois pontos",
      correctComponents: ["battery", "switch3way", "switch3way", "lamp"],
    },
    {
      id: 7,
      name: "Four Way",
      icon: "🔒",
      description: "Controle de três ou mais pontos",
      correctComponents: ["battery", "switch4way", "lamp"],
    },
  ];

  // Home Screen
  if (screen === "home") {
    return (
      <Container>
        <h1 style={{ textAlign: "center", fontSize: "28px", marginBottom: "5px" }}>
          ⚡ Simulador Elétrico
        </h1>
        <p style={{ textAlign: "center", fontSize: "13px", opacity: 0.7, marginBottom: "30px" }}>
          Aprenda elétrica residencial de forma interativa
        </p>

        <div
          style={{
            background: "#1e293b",
            borderRadius: "12px",
            padding: "15px",
            marginBottom: "20px",
            textAlign: "center",
          }}
        >
          <p style={{ fontSize: "12px", opacity: 0.7, margin: "5px 0" }}>
            📊 Pontuação Total
          </p>
          <p style={{ fontSize: "24px", fontWeight: "bold", margin: "0" }}>
            {progress.totalScore} pts
          </p>
          <p style={{ fontSize: "12px", opacity: 0.7, margin: "5px 0" }}>
            Níveis completos: {progress.completedLevels.length}/7
          </p>
        </div>

        <Button onClick={() => setScreen("levels")}>Começar</Button>
        <Button secondary onClick={() => setScreen("tutorial")}>
          📚 Aprender
        </Button>
        <Button secondary>💰 Remover Anúncios</Button>
      </Container>
    );
  }

  // Levels Screen
  if (screen === "levels") {
    return (
      <Container>
        <h2 style={{ textAlign: "center", marginBottom: "20px" }}>Níveis</h2>

        {levels.map((level) => {
          const isCompleted = progress.completedLevels.includes(level.id);
          const isUnlocked = level.id === 1 || progress.completedLevels.includes(level.id - 1);

          return (
            <Button
              key={level.id}
              onClick={() => {
                if (isUnlocked) {
                  setProgress({ ...progress, currentLevel: level.id });
                  setCircuitState({ components: [], isCorrect: false });
                  setScreen("game");
                }
              }}
              disabled={!isUnlocked}
              secondary={!isCompleted && isUnlocked}
            >
              {isCompleted ? "✅" : level.icon} Nível {level.id} - {level.name}
            </Button>
          );
        })}

        <Button onClick={() => setScreen("home")} secondary style={{ marginTop: "20px" }}>
          Voltar
        </Button>
      </Container>
    );
  }

  // Game Screen
  if (screen === "game") {
    const currentLevelData = levels[progress.currentLevel - 1];

    const addComponent = (type) => {
      setCircuitState({
        ...circuitState,
        components: [...circuitState.components, { id: Date.now(), type }],
      });
    };

    const removeComponent = (id) => {
      setCircuitState({
        ...circuitState,
        components: circuitState.components.filter((c) => c.id !== id),
      });
    };

    const testCircuit = () => {
      const componentTypes = circuitState.components
        .map((c) => c.type)
        .sort()
        .join(",");
      const correctTypes = currentLevelData.correctComponents.sort().join(",");

      if (componentTypes === correctTypes) {
        const newProgress = {
          ...progress,
          completedLevels: [...new Set([...progress.completedLevels, progress.currentLevel])],
          totalScore: progress.totalScore + 100,
        };
        saveProgress(newProgress);
        setCircuitState({ ...circuitState, isCorrect: true });
        setScreen("result");
      } else {
        alert("❌ Circuito incorreto! Verifique os componentes.");
      }
    };

    const componentLabels = {
      battery: "🔋 Bateria",
      lamp: "💡 Lâmpada",
      switch: "🔘 Interruptor",
      switch3way: "🔀 3-Way",
      switch4way: "🔁 4-Way",
      outlet: "🔌 Tomada",
      breaker: "⚡ Disjuntor",
      heater: "🔥 Resistência",
    };

    return (
      <Container>
        <h2 style={{ textAlign: "center" }}>Nível {progress.currentLevel}</h2>
        <p style={{ fontSize: "13px", opacity: 0.7, textAlign: "center", marginBottom: "20px" }}>
          {currentLevelData.description}
        </p>

        <div
          style={{
            background: "#0f172a",
            borderRadius: "12px",
            padding: "15px",
            minHeight: "150px",
            marginBottom: "20px",
            border: "2px dashed #334155",
          }}
        >
          <p style={{ fontSize: "12px", opacity: 0.7, margin: "0 0 10px 0" }}>
            Componentes adicionados:
          </p>
          {circuitState.components.length === 0 ? (
            <p style={{ textAlign: "center", opacity: 0.5, margin: "30px 0" }}>
              Nenhum componente adicionado
            </p>
          ) : (
            <div style={{ display: "flex", flexWrap: "wrap", gap: "8px" }}>
              {circuitState.components.map((comp) => (
                <div
                  key={comp.id}
                  style={{
                    background: "#1e293b",
                    padding: "8px 12px",
                    borderRadius: "8px",
                    fontSize: "12px",
                    display: "flex",
                    alignItems: "center",
                    gap: "8px",
                  }}
                >
                  {componentLabels[comp.type]}
                  <button
                    onClick={() => removeComponent(comp.id)}
                    style={{
                      background: "#ef4444",
                      border: "none",
                      borderRadius: "4px",
                      color: "white",
                      cursor: "pointer",
                      padding: "2px 6px",
                      fontSize: "11px",
                    }}
                  >
                    ✕
                  </button>
                </div>
              ))}
            </div>
          )}
        </div>

        <p style={{ fontSize: "12px", opacity: 0.7, marginBottom: "10px" }}>
          Selecione componentes:
        </p>
        <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: "8px", marginBottom: "20px" }}>
          {Object.entries(componentLabels).map(([type, label]) => (
            <button
              key={type}
              onClick={() => addComponent(type)}
              style={{
                padding: "10px",
                borderRadius: "8px",
                border: "1px solid #334155",
                background: "#1e293b",
                color: "white",
                cursor: "pointer",
                fontSize: "12px",
                transition: "all 0.2s ease",
              }}
              onMouseEnter={(e) => (e.target.style.background = "#334155")}
              onMouseLeave={(e) => (e.target.style.background = "#1e293b")}
            >
              {label}
            </button>
          ))}
        </div>

        <div style={{ display: "flex", gap: "10px" }}>
          <Button onClick={testCircuit}>✓ Testar</Button>
          <Button secondary onClick={() => setScreen("help")}>
            💡 Dica
          </Button>
        </div>
        <Button secondary onClick={() => setScreen("levels")} style={{ marginTop: "10px" }}>
          Voltar
        </Button>
      </Container>
    );
  }

  // Help Screen
  if (screen === "help") {
    const currentLevelData = levels[progress.currentLevel - 1];
    const hints = {
      1: "Para uma lâmpada funcionar, você precisa de uma bateria (energia) e uma lâmpada. Tente adicionar esses dois componentes!",
      2: "Um interruptor controla o circuito! Adicione: Bateria → Interruptor → Lâmpada",
      3: "Para controlar múltiplas lâmpadas, você precisa de 1 bateria, 1 interruptor e 2 lâmpadas",
      4: "Tomadas residenciais conectam-se diretamente à bateria",
      5: "Um chuveiro precisa de disjuntor para proteção: Bateria → Disjuntor → Resistência",
      6: "Three-way permite controle de dois pontos. Use dois switches 3-way e uma lâmpada",
      7: "Four-way permite controle de múltiplos pontos. Combine switches 4-way corretamente",
    };

    return (
      <Container>
        <h2 style={{ textAlign: "center" }}>💡 Dica - Nível {progress.currentLevel}</h2>
        <div
          style={{
            background: "#1e293b",
            borderRadius: "12px",
            padding: "15px",
            marginTop: "20px",
            marginBottom: "20px",
            borderLeft: "4px solid #2563eb",
          }}
        >
          <p style={{ fontSize: "13px", lineHeight: "1.6", margin: "0" }}>
            {hints[progress.currentLevel]}
          </p>
        </div>

        <Button onClick={() => setScreen("game")}>Voltar ao Jogo</Button>
        <Button secondary onClick={() => setScreen("levels")}>
          Voltar aos Níveis
        </Button>
      </Container>
    );
  }

  // Result Screen
  if (screen === "result") {
    const nextLevel = progress.currentLevel + 1;
    const hasNextLevel = nextLevel <= levels.length;

    return (
      <Container>
        <div style={{ textAlign: "center" }}>
          <h1 style={{ fontSize: "48px", margin: "0 0 10px 0" }}>✅</h1>
          <h2 style={{ margin: "0 0 10px 0" }}>Parabéns!</h2>
          <p style={{ fontSize: "13px", opacity: 0.7, marginBottom: "20px" }}>
            Você completou o nível com sucesso e ganhou 100 pontos!
          </p>

          <div
            style={{
              background: "#1e293b",
              borderRadius: "12px",
              padding: "15px",
              marginBottom: "20px",
            }}
          >
            <p style={{ fontSize: "12px", opacity: 0.7, margin: "0" }}>
              Progresso Total
            </p>
            <p style={{ fontSize: "20px", fontWeight: "bold", margin: "5px 0 0 0" }}>
              {progress.completedLevels.length}/7 Níveis
            </p>
          </div>

          {hasNextLevel ? (
            <>
              <Button
                onClick={() => {
                  setProgress({ ...progress, currentLevel: nextLevel });
                  setCircuitState({ components: [], isCorrect: false });
                  setScreen("game");
                }}
              >
                Próximo Nível →
              </Button>
              <Button secondary onClick={() => setScreen("levels")}>
                Voltar aos Níveis
              </Button>
            </>
          ) : (
            <>
              <Button
                onClick={() => {
                  setProgress({
                    completedLevels: [],
                    totalScore: 0,
                    currentLevel: 1,
                  });
                  localStorage.clear();
                  setScreen("home");
                }}
              >
                🎉 Começar Novamente
              </Button>
              <Button secondary onClick={() => setScreen("home")}>
                Voltar ao Início
              </Button>
            </>
          )}
        </div>
      </Container>
    );
  }

  // Tutorial Screen
  if (screen === "tutorial") {
    return (
      <Container>
        <h2 style={{ textAlign: "center", marginBottom: "20px" }}>📚 Como Jogar</h2>

        <div style={{ marginBottom: "20px" }}>
          <h3 style={{ fontSize: "14px", marginBottom: "8px" }}>O que é um Circuito Elétrico?</h3>
          <p style={{ fontSize: "12px", opacity: 0.8, lineHeight: "1.6", margin: "0" }}>
            Um circuito elétrico é um caminho fechado por onde a corrente elétrica flui. Ele precisa de uma fonte de energia (bateria), um caminho condutor e um consumidor (lâmpada).
          </p>
        </div>

        <div style={{ marginBottom: "20px" }}>
          <h3 style={{ fontSize: "14px", marginBottom: "8px" }}>Componentes Básicos</h3>
          <ul style={{ fontSize: "12px", opacity: 0.8, margin: "0", paddingLeft: "20px" }}>
            <li>🔋 <strong>Bateria:</strong> Fornece energia</li>
            <li>💡 <strong>Lâmpada:</strong> Consome energia e acende</li>
            <li>🔘 <strong>Interruptor:</strong> Liga/desliga o circuito</li>
            <li>🔌 <strong>Tomada:</strong> Ponto de conexão</li>
          </ul>
        </div>

        <div style={{ marginBottom: "20px" }}>
          <h3 style={{ fontSize: "14px", marginBottom: "8px" }}>Como Jogar</h3>
          <ol style={{ fontSize: "12px", opacity: 0.8, margin: "0", paddingLeft: "20px" }}>
            <li>Selecione um nível</li>
            <li>Adicione os componentes corretos</li>
            <li>Clique em "Testar" para validar</li>
            <li>Ganhe pontos e desbloqueie novos níveis!</li>
          </ol>
        </div>

        <Button onClick={() => setScreen("home")}>Voltar ao Início</Button>
      </Container>
    );
  }

  return null;
}
