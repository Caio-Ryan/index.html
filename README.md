import React, { useState, useRef } from "react";
import { motion } from "framer-motion";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import confetti from "canvas-confetti";

function rolarDados(qtd, faces) {
  const dados = [];
  for (let i = 0; i < qtd; i++) {
    dados.push(Math.floor(Math.random() * faces) + 1);
  }
  return dados;
}

function media(valores) {
  return valores.reduce((a, b) => a + b, 0) / valores.length;
}

export default function JogoEstatisticas() {
  const [aposta, setAposta] = useState(3);
  const [numDados, setNumDados] = useState(3);
  const [faces, setFaces] = useState(6);
  const [limiar, setLimiar] = useState(0.5);
  const [resultado, setResultado] = useState(null);
  const [pontuacao, setPontuacao] = useState(0);
  const [round, setRound] = useState(1);
  const [historico, setHistorico] = useState([]);
  const [animando, setAnimando] = useState(false);

  const audioRef = useRef({});

  const tocarSom = (tipo) => {
    if (audioRef.current[tipo]) {
      audioRef.current[tipo].currentTime = 0;
      audioRef.current[tipo].play();
    }
  };

  const jogar = () => {
    setAnimando(true);
    setResultado(null);
    tocarSom("rolar");

    setTimeout(() => {
      const dados = rolarDados(numDados, faces);
      const m = media(dados);
      const diff = Math.abs(m - aposta);
      const acertou = diff < limiar;
      if (acertou) {
        setPontuacao((p) => p + 1);
        tocarSom("vitoria");
        confetti({ particleCount: 100, spread: 70, origin: { y: 0.6 } });
      } else {
        tocarSom("derrota");
      }
      setRound((r) => r + 1);

      const r = { dados, media: m, aposta, diff, limiar, acertou };
      setResultado(r);
      setHistorico((h) => [{ ...r, round }, ...h]);
      setAnimando(false);
    }, 1500);
  };

  const simular = () => {
    let wins = 0;
    for (let i = 0; i < 1000; i++) {
      const dados = rolarDados(numDados, faces);
      const m = media(dados);
      if (Math.abs(m - aposta) < limiar) wins++;
    }
    alert(`Taxa de acerto estimada: ${(wins / 10).toFixed(1)}%`);
  };

  return (
    <div className="min-h-screen bg-gradient-to-b from-blue-50 to-blue-200 p-6 flex flex-col items-center">
      <audio ref={(el) => (audioRef.current["rolar"] = el)} src="/sounds/roll.mp3" preload="auto" />
      <audio ref={(el) => (audioRef.current["vitoria"] = el)} src="/sounds/win.mp3" preload="auto" />
      <audio ref={(el) => (audioRef.current["derrota"] = el)} src="/sounds/lose.mp3" preload="auto" />

      <motion.h1
        initial={{ opacity: 0, y: -20 }}
        animate={{ opacity: 1, y: 0 }}
        className="text-3xl font-bold mb-6"
      >
        🎲 Jogo de Estatísticas com Dados
      </motion.h1>

      <Card className="w-full max-w-xl mb-6">
        <CardContent className="grid gap-4 p-4">
          <label>
            Aposta:
            <input
              type="number"
              step="0.1"
              value={aposta}
              onChange={(e) => setAposta(parseFloat(e.target.value))}
              className="border rounded p-1 w-20 ml-2"
            />
          </label>
          <label>
            Número de dados:
            <input
              type="number"
              value={numDados}
              onChange={(e) => setNumDados(parseInt(e.target.value))}
              className="border rounded p-1 w-20 ml-2"
            />
          </label>
          <label>
            Faces dos dados:
            <input
              type="number"
              value={faces}
              onChange={(e) => setFaces(parseInt(e.target.value))}
              className="border rounded p-1 w-20 ml-2"
            />
          </label>
          <label>
            Limiar:
            <input
              type="number"
              step="0.1"
              value={limiar}
              onChange={(e) => setLimiar(parseFloat(e.target.value))}
              className="border rounded p-1 w-20 ml-2"
            />
          </label>

          <div className="flex gap-4 mt-4">
            <Button onClick={jogar} disabled={animando}>
              {animando ? "Rolando..." : "Rolar Dados"}
            </Button>
            <Button variant="secondary" onClick={simular}>
              Simular 1000 rodadas
            </Button>
          </div>
        </CardContent>
      </Card>

      {animando && (
        <div className="flex gap-4 mb-6">
          {[...Array(numDados)].map((_, i) => (
            <motion.div
              key={i}
              className="w-12 h-12 bg-white rounded-xl shadow-lg flex items-center justify-center text-2xl font-bold"
              animate={{ rotate: [0, 360] }}
              transition={{ repeat: Infinity, duration: 0.6, ease: "linear" }}
            >
              🎲
            </motion.div>
          ))}
        </div>
      )}

      {resultado && !animando && (
        <motion.div
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          className="w-full max-w-xl"
        >
          <Card className="mb-6">
            <CardContent className="p-4">
              <h2 className="text-xl font-semibold mb-2">Resultado da Rodada {round - 1}</h2>
              <p>Aposta: {resultado.aposta.toFixed(2)}</p>
              <div className="flex gap-2 my-2">
                {resultado.dados.map((d, i) => (
                  <motion.div
                    key={i}
                    initial={{ scale: 0 }}
                    animate={{ scale: 1 }}
                    transition={{ duration: 0.3, delay: i * 0.2 }}
                    className="w-12 h-12 bg-white rounded-xl shadow-md flex items-center justify-center text-xl font-bold"
                  >
                    {d}
                  </motion.div>
                ))}
              </div>
              <p>Média: {resultado.media.toFixed(2)}</p>
              <p>
                Diferença: {resultado.diff.toFixed(2)} (limiar {resultado.limiar})
              </p>
              {resultado.acertou ? (
                <p className="text-green-600 font-bold mt-2">✅ Você venceu!</p>
              ) : (
                <p className="text-red-600 font-bold mt-2">❌ Não foi dessa vez.</p>
              )}
            </CardContent>
          </Card>
        </motion.div>
      )}

      <Card className="w-full max-w-xl">
        <CardContent className="p-4">
          <h2 className="text-xl font-semibold mb-2">Pontuação</h2>
          <p>Rodada atual: {round}</p>
          <p>Pontuação: {pontuacao}</p>
          <div className="w-full bg-gray-200 h-4 rounded mt-2">
            <motion.div
              className="bg-green-500 h-4 rounded"
              initial={{ width: 0 }}
              animate={{ width: `${(pontuacao / round) * 100}%` }}
              transition={{ duration: 0.5 }}
            />
          </div>
        </CardContent>
      </Card>

      <Card className="w-full max-w-xl mt-6">
        <CardContent className="p-4">
          <h2 className="text-xl font-semibold mb-2">Histórico</h2>
          {historico.length === 0 && <p>Nenhuma rodada ainda.</p>}
          {historico.map((h, i) => (
            <div key={i} className="border-b py-2">
              <p>
                <strong>Rodada {h.round}:</strong> Dados {h.dados.join(", ")},
                média {h.media.toFixed(2)} → {h.acertou ? "✅" : "❌"}
              </p>
            </div>
          ))}
        </CardContent>
      </Card>
    </div>
  );
}
