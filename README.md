// AnswerCard.tsx
import { useState, useEffect } from "react";
import { Answer } from "@shared/schema";
import { useGameContext } from "@/contexts/GameContext";
import useGameSounds from "@/hooks/useGameSounds";
import { BadgeCheck, X } from "lucide-react";

interface AnswerCardProps {
  answer: Answer;
  displayOrder: number;
  activeTeam?: 1 | 2;
}

export default function AnswerCard({ answer, displayOrder, activeTeam = 1 }: AnswerCardProps) {
  const { isAnswerRevealed, revealAnswer, updateTeamScore, soundEnabled } = useGameContext();
  const [flipped, setFlipped] = useState(false);
  const [answerStatus, setAnswerStatus] = useState<'correct' | 'wrong' | null>(null);
  const { playFlipSound, playCorrectSound, playWrongSound } = useGameSounds();

  useEffect(() => {
    const revealed = isAnswerRevealed(answer.id);
    setFlipped(revealed);
  }, [answer.id, isAnswerRevealed]);

  const handleFlip = () => {
    if (!flipped) {
      setFlipped(true);
      revealAnswer(answer.id);
      if (soundEnabled) {
        playFlipSound();
      }
    }
  };

  const handleCorrectAnswer = (e: React.MouseEvent) => {
    e.stopPropagation();
    if (flipped && answerStatus !== 'correct') {
      setAnswerStatus('correct');
      if (soundEnabled) playCorrectSound();
      updateTeamScore(activeTeam, answer.points);
    }
  };

  const handleWrongAnswer = (e: React.MouseEvent) => {
    e.stopPropagation();
    if (flipped && answerStatus !== 'wrong') {
      setAnswerStatus('wrong');
      if (soundEnabled) playWrongSound();
    }
  };

  return (
    <div className={`perspective-1000 h-24 md:h-28 w-full cursor-pointer relative`} onClick={handleFlip}>
      <div className={`relative w-full h-full transition-transform duration-600 transform-style-preserve-3d ${flipped ? 'rotate-y-180' : ''}`}>
        <div className="absolute w-full h-full backface-hidden">
          <div className="bg-white/30 flex items-center justify-center rounded-xl h-full w-full shadow-lg">
            <span className="text-white text-5xl md:text-6xl font-bold">{displayOrder}</span>
          </div>
        </div>
        <div className="absolute w-full h-full backface-hidden rotate-y-180">
          <div className={`bg-white/90 flex justify-between items-center p-4 rounded-xl h-full w-full shadow-lg ${answerStatus === 'correct' ? 'ring-4 ring-green-500' : answerStatus === 'wrong' ? 'ring-4 ring-red-500 opacity-50' : ''}`}>
            <span className="text-primary text-xl md:text-2xl font-bold">{answer.text}</span>
            <span className="text-white text-2xl md:text-3xl font-mono bg-primary px-3 py-1 rounded-lg shadow">{answer.points}</span>
          </div>
        </div>
      </div>
      {flipped && !answerStatus && (
        <div className="absolute top-0 left-0 right-0 bottom-0 flex items-center justify-center z-10 backdrop-blur-sm bg-black/30 rounded-xl">
          <div className="bg-white/90 p-4 rounded-lg shadow-lg flex flex-col items-center gap-3">
            <p className="text-primary font-bold mb-1">هل الإجابة صحيحة؟</p>
            <div className="flex gap-3">
              <button onClick={handleCorrectAnswer} className="bg-green-500 px-4 py-2 rounded-lg hover:bg-green-600 text-white font-bold flex items-center gap-1">
                <BadgeCheck className="h-5 w-5" />صحيحة
              </button>
              <button onClick={handleWrongAnswer} className="bg-red-500 px-4 py-2 rounded-lg hover:bg-red-600 text-white font-bold flex items-center gap-1">
                <X className="h-5 w-5" />خاطئة
              </button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}

// GameStart.tsx
import { useState } from "react";
import { useLocation } from "wouter";
import { useGameContext } from "@/contexts/GameContext";
import { Button } from "@/components/ui/button";

export default function GameStart() {
  const [isLoading, setIsLoading] = useState(false);
  const { initializeGame } = useGameContext();
  const [, setLocation] = useLocation();

  const handleStartGame = () => {
    setIsLoading(true);
    initializeGame();
    setTimeout(() => {
      setLocation("/game");
    }, 1000);
  };

  return (
    <div className="bg-game-gradient min-h-screen flex flex-col items-center justify-center p-4">
      <div className="text-center mb-8">
        <h1 className="text-5xl md:text-7xl font-bold mb-4 text-white game-title-shadow">فاميلي فيود</h1>
        <h2 className="text-3xl md:text-5xl font-bold text-gradient">مع أبو شاكر</h2>
      </div>
      <div className="bg-white/90 p-8 rounded-xl shadow-2xl max-w-md w-full">
        <h2 className="text-2xl font-bold mb-6 text-center text-primary">قواعد اللعبة</h2>
        <ul className="list-disc list-inside space-y-3 mb-8 text-right">
          <li>سيتم عرض سؤال ومجموعة من الإجابات المخفية</li>
          <li>اضغط على الإجابة للكشف عنها</li>
          <li>الفريق الذي يكشف عن إجابة صحيحة يحصل على نقاط تلك الإجابة</li>
          <li>إذا كانت الإجابة خاطئة، انقر على زر "إجابة خاطئة"</li>
          <li>بعد الانتهاء من جميع الإجابات، انتقل إلى السؤال التالي</li>
        </ul>
        <Button onClick={handleStartGame} disabled={isLoading} className="w-full py-6 text-xl font-bold bg-primary hover:bg-primary/90 text-white">
          {isLoading ? "جاري التحميل..." : "ابدأ اللعبة!"}
        </Button>
      </div>
    </div>
  );
}

/* CSS Styles for index.css */
.perspective-1000 { perspective: 1000px; }
.transform-style-preserve-3d { transform-style: preserve-3d; }
.backface-hidden { backface-visibility: hidden; }
.rotate-y-180 { transform: rotateY(180deg); }
.duration-600 { transition-duration: 600ms; }
.bg-game-gradient { background: linear-gradient(135deg, hsl(12, 76%, 61%) 0%, hsl(32, 95%, 44%) 100%); }
.text-gradient {
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-image: linear-gradient(45deg, #FF5722, #FFB74D);
}
.game-title-shadow { text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3); }
