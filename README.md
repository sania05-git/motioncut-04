import React, { useState, useCallback, useEffect } from 'react';
import { Coins, RotateCcw, Play } from 'lucide-react';

interface TossResult {
  outcome: 'heads' | 'tails';
  timestamp: Date;
}

function App() {
  const [isFlipping, setIsFlipping] = useState(false);
  const [results, setResults] = useState<TossResult[]>([]);
  const [multipleFlips, setMultipleFlips] = useState(1);
  const [animatingCoin, setAnimatingCoin] = useState(false);

  const flipCoin = useCallback(() => {
    setAnimatingCoin(true);
    const outcome = Math.random() < 0.5 ? 'heads' : 'tails';
    return { outcome, timestamp: new Date() };
  }, []);

  const handleSingleFlip = useCallback(() => {
    if (isFlipping) return;
    setIsFlipping(true);
    setTimeout(() => {
      const result = flipCoin();
      setResults(prev => [...prev, result]);
      setIsFlipping(false);
    }, 1000);
  }, [flipCoin, isFlipping]);

  const handleMultipleFlips = useCallback(() => {
    if (isFlipping) return;
    setIsFlipping(true);
    let count = 0;
    const interval = setInterval(() => {
      const result = flipCoin();
      setResults(prev => [...prev, result]);
      count++;
      if (count >= multipleFlips) {
        clearInterval(interval);
        setIsFlipping(false);
      }
    }, 1000);
  }, [flipCoin, isFlipping, multipleFlips]);

  const resetResults = useCallback(() => {
    setResults([]);
  }, []);

  useEffect(() => {
    if (animatingCoin) {
      const timer = setTimeout(() => setAnimatingCoin(false), 1000);
      return () => clearTimeout(timer);
    }
  }, [animatingCoin]);

  const stats = {
    heads: results.filter(r => r.outcome === 'heads').length,
    tails: results.filter(r => r.outcome === 'tails').length,
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-500 to-purple-600 flex items-center justify-center p-4">
      <div className="bg-white rounded-xl shadow-2xl p-8 w-full max-w-md">
        <div className="text-center mb-8">
          <h1 className="text-3xl font-bold text-gray-800 mb-2">Virtual Coin Toss</h1>
          <p className="text-gray-600">Flip a coin and track your results!</p>
        </div>

        <div className="flex justify-center mb-8">
          <div 
            className={`relative w-32 h-32 rounded-full bg-gradient-to-r from-yellow-400 to-yellow-300 flex items-center justify-center shadow-lg
              ${animatingCoin ? 'animate-[spin_1s_ease-in-out]' : ''}`}
          >
            <Coins className="w-16 h-16 text-yellow-600" />
          </div>
        </div>

        <div className="space-y-4 mb-8">
          <div className="flex items-center gap-4">
            <input
              type="number"
              min="1"
              max="100"
              value={multipleFlips}
              onChange={(e) => setMultipleFlips(Math.min(100, Math.max(1, parseInt(e.target.value) || 1)))}
              className="flex-1 px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
            <button
              onClick={handleMultipleFlips}
              disabled={isFlipping}
              className="px-6 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 disabled:opacity-50 disabled:cursor-not-allowed flex items-center gap-2"
            >
              <Play className="w-4 h-4" />
              Flip {multipleFlips > 1 ? `${multipleFlips} times` : ''}
            </button>
          </div>

          <button
            onClick={handleSingleFlip}
            disabled={isFlipping}
            className="w-full px-6 py-3 bg-purple-500 text-white rounded-lg hover:bg-purple-600 disabled:opacity-50 disabled:cursor-not-allowed"
          >
            Single Flip
          </button>

          <button
            onClick={resetResults}
            className="w-full px-6 py-3 border-2 border-gray-300 text-gray-700 rounded-lg hover:bg-gray-50 flex items-center justify-center gap-2"
          >
            <RotateCcw className="w-4 h-4" />
            Reset Results
          </button>
        </div>

        <div className="bg-gray-50 rounded-lg p-4">
          <h2 className="text-xl font-semibold mb-4">Statistics</h2>
          <div className="grid grid-cols-2 gap-4">
            <div className="bg-blue-100 p-4 rounded-lg">
              <p className="text-blue-800 font-medium">Heads</p>
              <p className="text-2xl font-bold text-blue-900">{stats.heads}</p>
              <p className="text-blue-700">
                {results.length > 0
                  ? `${((stats.heads / results.length) * 100).toFixed(1)}%`
                  : '0%'}
              </p>
            </div>
            <div className="bg-purple-100 p-4 rounded-lg">
              <p className="text-purple-800 font-medium">Tails</p>
              <p className="text-2xl font-bold text-purple-900">{stats.tails}</p>
              <p className="text-purple-700">
                {results.length > 0
                  ? `${((stats.tails / results.length) * 100).toFixed(1)}%`
                  : '0%'}
              </p>
            </div>
          </div>
        </div>

        {results.length > 0 && (
          <div className="mt-8">
            <h2 className="text-xl font-semibold mb-4">History</h2>
            <div className="max-h-48 overflow-y-auto">
              {[...results].reverse().map((result, index) => (
                <div
                  key={index}
                  className="flex items-center justify-between py-2 border-b last:border-b-0"
                >
                  <span className="font-medium capitalize">{result.outcome}</span>
                  <span className="text-sm text-gray-500">
                    {result.timestamp.toLocaleTimeString()}
                  </span>
                </div>
              ))}
            </div>
          </div>
        )}
      </div>
    </div>
  );
}

export default App;
