
import React, { useState, useMemo } from 'react';

// --- SEO対策のための推奨事項 ---
// 実際にWebサイトとして公開する際は、以下の<head>タグ内情報をindex.htmlに設定してください。
/*
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>繊細さんお悩み相談窓口 | HSP・発達障害の傾向をセルフチェック</title>
  <meta name="description" content="「もしかしてHSP？」「生きづらい…」と感じるあなたのためのセルフチェックツール。簡単な質問からHSP、ADHD、ASDなどの傾向を把握し、自分を理解する第一歩に。専門家への相談もサポートします。" />
  <meta name="keywords" content="HSP, 繊細さん, ADHD, ASD, 発達障害, うつ, 不安, 生きづらさ, セルフチェック, 相談" />
  
  <!-- Open Graph / Facebook -->
  <meta property="og:type" content="website" />
  <meta property="og:url" content="（ここにサイトのURLを記載）" />
  <meta property="og:title" content="繊細さんお悩み相談窓口 | HSP・発達障害の傾向をセルフチェック" />
  <meta property="og:description" content="「生きづらい…」と感じるあなたのためのセルフチェックツール。簡単な質問からHSP、ADHD、ASDなどの傾向を把握し、自分を理解する第一歩に。" />
  <meta property="og:image" content="（ここにOGP画像のURLを記載）" />
</head>
*/

// --- アイコンコンポーネント ---
const CheckCircleIcon = () => (
  <svg xmlns="http://www.w3.org/2000/svg" className="h-6 w-6 mr-2 inline-block" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
    <path strokeLinecap="round" strokeLinejoin="round" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
  </svg>
);

const AlertTriangleIcon = () => (
  <svg xmlns="http://www.w3.org/2000/svg" className="h-6 w-6 mr-2 inline-block" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
    <path strokeLinecap="round" strokeLinejoin="round" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
  </svg>
);

const ExternalLinkIcon = () => (
    <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5 ml-2 inline-block" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
        <path strokeLinecap="round" strokeLinejoin="round" d="M10 6H6a2 2 0 00-2 2v10a2 2 0 002 2h10a2 2 0 002-2v-4M14 4h6m0 0v6m0-6L10 14" />
    </svg>
);


// --- 質問データ ---
const questions = [
  // カテゴリA：HSP（繊細さ・感覚過敏）
  { id: 1, text: 'かすかな匂いや物音など、他の人が気づかないような些細な刺激によく気づくほうだ。', category: 'HSP' },
  { id: 2, text: '美術や音楽、美しい景色などに深く感動しやすい。', category: 'HSP' },
  { id: 3, text: '人の気分に左右されやすいと感じる。', category: 'HSP' },
  { id: 4, text: '忙しい日々が続くと、一人になれる空間に引きこもりたくなる。', category: 'HSP' },
  { id: 5, text: '一度にたくさんのことを頼まれると混乱してしまう。', category: 'HSP' },
  { id: 6, text: '大きな音や、サイレンの音などが非常に不快に感じられる。', category: 'HSP' },

  // カテゴリB：ADHD（不注意・多動性・衝動性）
  { id: 7, text: '物事を順序立てて行うのが苦手だ。', category: 'ADHD' },
  { id: 8, text: '約束ややるべきことを忘れやすい。', category: 'ADHD' },
  { id: 9, text: '仕事や勉強で、不注意な間違いが多い。', category: 'ADHD' },
  { id: 10, text: '静かに座っているのが苦痛で、そわそわしてしまう。', category: 'ADHD' },
  { id: 11, text: '相手の質問が終わる前に、出し抜くように答えてしまうことがある。', category: 'ADHD' },
  { id: 12, text: 'よく考えずに、思いつきで行動してしまうことが多い。', category: 'ADHD' },

  // カテゴリC：ASD（対人関係・こだわり・感覚）
  { id: 13, text: '人との雑談や世間話が苦手だと感じる。', category: 'ASD' },
  { id: 14, text: '「常識的に考えて」「空気を読んで」と言われるのが苦手だ。', category: 'ASD' },
  { id: 15, text: '決まった手順や日課が崩れると、強いストレスを感じる。', category: 'ASD' },
  { id: 16, text: '自分の興味がある分野については、非常に詳しく、膨大な知識を持っている。', category: 'ASD' },
  { id: 17, text: '予期せぬ変化や、突然の予定変更に対応するのが苦手だ。', category: 'ASD' },
  { id: 18, text: '特定の音、光、味、匂いなどを、極端に嫌がることがある。', category: 'ASD' },

  // カテゴリD：うつ病・気分障害
  { id: 19, text: 'ここ最近、ほとんど毎日、気分が落ち込んでいる。', category: 'Depression' },
  { id: 20, text: 'これまで楽しめていたことに興味がわかない、または楽しめない。', category: 'Depression' },
  { id: 21, text: 'いつもより疲れやすく、気力がないと感じる。', category: 'Depression' },
  { id: 22, text: '自分には価値がない、あるいは過剰な罪悪感を感じる。', category: 'Depression' },

  // カテゴリE：不安障害
  { id: 23, text: '様々な出来事について、過剰に心配してしまうことがコントロールできない。', category: 'Anxiety' },
  { id: 24, text: '理由もなく突然、強い恐怖感に襲われ、動悸や息苦しさを感じることがある。', category: 'Anxiety' },
  { id: 25, text: '人からどう思われているか、常に気になってしまう。', category: 'Anxiety' },
];

// --- カテゴリ定義 ---
const categories = {
  HSP: { name: 'HSP（繊細さ）の傾向', color: 'bg-teal-500', description: '感覚が鋭敏で、刺激を受けやすい気質です。物事を深く考える一方、疲れやすい側面もあります。' },
  ADHD: { name: 'ADHD（不注意・多動）の傾向', color: 'bg-sky-500', description: '集中し続けることや、じっとしていることが苦手な特性です。忘れ物や衝動的な行動が見られることがあります。'},
  ASD: { name: 'ASD（こだわり・対人関係）の傾向', color: 'bg-amber-500', description: '対人関係やコミュニケーションに独特のスタイルを持ち、特定のことに強いこだわりを持つ特性です。'},
  Depression: { name: '気分の落ち込みの傾向', color: 'bg-indigo-500', description: '憂うつな気分や、興味・喜びの喪失が続く状態です。気力や集中力の低下が見られることもあります。'},
  Anxiety: { name: '不安を感じやすい傾向', color: 'bg-purple-500', description: '様々なことに対して過剰な心配や恐怖を感じ、心や体に影響が出ることがあります。'},
};

// 閾値の設定（このスコア以上で受診を推奨）
const thresholds = {
  HSP: 4,
  ADHD: 4,
  ASD: 4,
  Depression: 3,
  Anxiety: 2,
};

// --- メインコンポーネント ---
export default function App() {
  const [screen, setScreen] = useState('welcome'); // 'welcome', 'quiz', 'results'
  const [answers, setAnswers] = useState({});
  const [currentQuestionIndex, setCurrentQuestionIndex] = useState(0);

  const handleStart = () => {
    setScreen('quiz');
    setAnswers({});
    setCurrentQuestionIndex(0);
  };

  const handleAnswer = (questionId, answer) => {
    const newAnswers = { ...answers, [questionId]: answer };
    setAnswers(newAnswers);

    if (currentQuestionIndex < questions.length - 1) {
      setCurrentQuestionIndex(currentQuestionIndex + 1);
    } else {
      setScreen('results');
    }
  };

  const scores = useMemo(() => {
    if (screen !== 'results') return {};
    
    const initialScores = Object.keys(categories).reduce((acc, key) => ({ ...acc, [key]: 0 }), {});
    return questions.reduce((acc, question) => {
      if (answers[question.id] === 'yes' && acc[question.category] !== undefined) {
        acc[question.category]++;
      }
      return acc;
    }, initialScores);
  }, [screen, answers]);

  const renderScreen = () => {
    switch (screen) {
      case 'quiz':
        return (
          <main>
            <QuizScreen question={questions[currentQuestionIndex]} onAnswer={handleAnswer} progress={(currentQuestionIndex + 1) / questions.length} />
          </main>
        );
      case 'results':
        return (
          <main>
            <ResultsScreen scores={scores} onRestart={handleStart} />
          </main>
        );
      case 'welcome':
      default:
        return (
          <main>
            <WelcomeScreen onStart={handleStart} />
          </main>
        );
    }
  };

  return (
    <div className="bg-gray-50 min-h-screen font-sans flex items-center justify-center p-4">
      <div className="w-full max-w-3xl mx-auto">
        {renderScreen()}
      </div>
    </div>
  );
}

// --- 画面コンポーネント ---

function WelcomeScreen({ onStart }) {
  return (
    <div className="bg-white p-8 md:p-12 rounded-2xl shadow-lg text-center animate-fade-in">
      <header>
        <h1 className="text-3xl md:text-4xl font-bold text-gray-800 mb-4">繊細さんお悩み相談窓口</h1>
        <p className="text-gray-600 mb-8 max-w-xl mx-auto">「もしかしてHSP？」「なんだか生きづらい…」と感じるあなたのためのセルフチェックツールです。</p>
      </header>
      <section className="bg-blue-50 border-l-4 border-blue-500 text-gray-700 p-4 rounded-lg text-left mb-10">
        <h2 className="font-bold text-lg flex items-center"><AlertTriangleIcon />【重要：必ずお読みください】</h2>
        <ul className="list-disc list-inside mt-3 text-sm space-y-1 pl-2">
          <li>このツールは医学的な診断を行うものではありません。</li>
          <li>結果はあくまでご自身の状態を把握するための参考情報です。</li>
          <li>心身に不調を感じる場合や、ご自身の状態について深く知りたい場合は、必ず精神科や心療内科などの専門医療機関にご相談ください。</li>
        </ul>
      </section>
      <button onClick={onStart} className="w-full max-w-md mx-auto bg-blue-600 hover:bg-blue-700 text-white font-bold py-4 px-8 rounded-full text-xl transition-all duration-300 transform hover:scale-105 shadow-md hover:shadow-lg">
        セルフチェックをはじめる
      </button>
    </div>
  );
}

function QuizScreen({ question, onAnswer, progress }) {
  return (
    <div className="bg-white p-8 md:p-10 rounded-2xl shadow-lg w-full animate-fade-in">
      <header className="mb-8">
        <div className="flex justify-between items-center mb-2">
          <span className="text-sm font-semibold text-gray-600">質問 {question.id} / {questions.length}</span>
          <span className="text-sm font-semibold text-blue-600">{Math.round(progress * 100)}%</span>
        </div>
        <div className="w-full bg-gray-200 rounded-full h-3">
          <div className="bg-blue-500 h-3 rounded-full transition-all duration-500" style={{ width: `${progress * 100}%` }}></div>
        </div>
      </header>
      <section className="bg-gray-50 p-6 rounded-lg mb-8 min-h-[150px] flex items-center justify-center">
        <p className="text-xl md:text-2xl text-gray-800 text-center font-medium leading-relaxed">{question.text}</p>
      </section>
      <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
        <button onClick={() => onAnswer(question.id, 'yes')} className="bg-blue-500 hover:bg-blue-600 text-white font-bold py-4 px-6 rounded-lg text-xl transition-all duration-200 transform hover:scale-105 shadow-sm">
          はい
        </button>
        <button onClick={() => onAnswer(question.id, 'no')} className="bg-gray-200 hover:bg-gray-300 text-gray-700 font-bold py-4 px-6 rounded-lg text-xl transition-all duration-200 transform hover:scale-105 shadow-sm">
          いいえ
        </button>
      </div>
    </div>
  );
}

// --- 結果画面のコンポーネント ---

// 結果に応じた誘導メッセージを表示するコンポーネント
function GuidanceSection({ scores }) {
  const needsConsultation = Object.keys(scores).some(key => scores[key] >= thresholds[key]);
  const someTendencies = !needsConsultation && Object.values(scores).some(score => score > 0);
  const guidanceLink = "https://esora243.github.io/CreaMed/";

  // パターンA: 顕著な傾向があり、受診推奨
  if (needsConsultation) {
    return (
      <section className="bg-red-50 border-2 border-red-200 text-red-800 p-6 rounded-lg text-left mb-10">
        <h3 className="font-bold text-xl flex items-center mb-3"><AlertTriangleIcon />専門家への相談を推奨します</h3>
        <p className="mb-4">今回の結果から、専門家への相談を検討する価値がある可能性が示唆されました。生きづらさやお悩みを抱えている場合、一人で抱え込まず、専門機関にご相談ください。</p>
        <a href={guidanceLink} target="_blank" rel="noopener noreferrer" className="w-full block text-center bg-red-500 hover:bg-red-600 text-white font-bold py-3 px-6 rounded-lg text-lg transition-all duration-300 transform hover:scale-105">
          医療機関・相談窓口を探す <ExternalLinkIcon />
        </a>
      </section>
    );
  }

  // パターンB: いくつかの傾向が見られる
  if (someTendencies) {
    return (
      <section className="bg-blue-50 border-2 border-blue-200 text-blue-800 p-6 rounded-lg text-left mb-10">
        <h3 className="font-bold text-xl flex items-center mb-3"><CheckCircleIcon />自分を理解するための一歩</h3>
        <p className="mb-4">今回の結果は、ご自身の特性を理解するための一つのヒントになります。もし、さらに詳しく知りたい、または日々の生活で気になることがある場合は、以下のサイトも参考になります。</p>
        <a href={guidanceLink} target="_blank" rel="noopener noreferrer" className="w-full block text-center bg-blue-500 hover:bg-blue-600 text-white font-bold py-3 px-6 rounded-lg text-lg transition-all duration-300 transform hover:scale-105">
          心の健康に関する情報を見る <ExternalLinkIcon />
        </a>
      </section>
    );
  }

  // パターンC: 特に顕著な傾向はない
  return (
    <section className="bg-green-50 border-2 border-green-200 text-green-800 p-6 rounded-lg text-left mb-10">
      <h3 className="font-bold text-xl flex items-center mb-3"><CheckCircleIcon />チェックお疲れ様でした</h3>
      <p className="mb-4">特に顕著な傾向は見られませんでしたが、心の健康を保つことは誰にとっても大切です。ご自身の状態について知る良い機会になったのであれば幸いです。</p>
      <a href={guidanceLink} target="_blank" rel="noopener noreferrer" className="w-full block text-center bg-green-500 hover:bg-green-600 text-white font-bold py-3 px-6 rounded-lg text-lg transition-all duration-300 transform hover:scale-105">
        心の健康に関する情報を見る <ExternalLinkIcon />
      </a>
    </section>
  );
}

// 電話相談窓口を表示するコンポーネント
function EmergencyContact() {
    const phoneNumber = "08052866835";
    const formattedPhoneNumber = "080-5286-6835";
    return (
        <section className="text-center border-t border-b border-gray-200 py-6 my-10">
            <p className="text-gray-600 text-lg mb-2">もし、今すぐ誰かに話を聞いてほしいと感じる方は</p>
            <p className="text-gray-800 font-bold text-xl">
                お辛い方はこちらまで: <a href={`tel:${phoneNumber}`} className="text-blue-600 hover:underline text-2xl tracking-wider">{formattedPhoneNumber}</a>
            </p>
            <p className="text-xs text-gray-500 mt-2">※発信には通話料金がかかります。</p>
        </section>
    );
}

// 結果表示画面本体
function ResultsScreen({ scores, onRestart }) {
  return (
    <div className="bg-white p-8 md:p-12 rounded-2xl shadow-lg animate-fade-in">
      <header className="text-center mb-10">
        <h2 className="text-3xl font-bold text-gray-800 mb-2">セルフチェック結果</h2>
        <p className="text-gray-500">ご自身の特性を理解するための一つの参考としてご覧ください。</p>
      </header>
      
      <GuidanceSection scores={scores} />
      
      <EmergencyContact />

      <section className="space-y-6">
        <h3 className="text-xl font-bold text-gray-700 text-center mb-4">あなたの傾向</h3>
        {Object.keys(scores).map(key => {
          const category = categories[key];
          const score = scores[key];
          const maxScore = questions.filter(q => q.category === key).length;
          const percentage = maxScore > 0 ? (score / maxScore) * 100 : 0;

          return (
            <div key={key} className="bg-gray-50 p-4 rounded-lg">
              <h4 className="text-lg font-semibold text-gray-800 mb-2">{category.name}</h4>
              <p className="text-sm text-gray-600 mb-3">{category.description}</p>
              <div className="w-full bg-gray-200 rounded-full h-6 relative">
                <div 
                  className={`${category.color} h-6 rounded-full flex items-center justify-end pr-3 text-white text-sm font-bold transition-all duration-1000`}
                  style={{ width: `${percentage}%` }}
                >
                </div>
                 <span className="absolute inset-0 flex items-center justify-center text-sm font-bold text-gray-700">{score} / {maxScore}</span>
              </div>
            </div>
          );
        })}
      </section>

      <footer className="mt-10 text-center">
         <p className="text-gray-500 mb-6 text-xs">※この結果は医学的診断に代わるものではありません。あくまで参考としてご活用ください。</p>
        <button onClick={onRestart} className="bg-gray-600 hover:bg-gray-700 text-white font-bold py-3 px-6 rounded-full text-lg transition-all duration-300 transform hover:scale-105">
          もう一度チェックする
        </button>
      </footer>
    </div>
  );
}

// CSS in JS for animations
const style = document.createElement('style');
style.innerHTML = `
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700;800&family=Noto+Sans+JP:wght@400;500;700&display=swap');
  body {
    font-family: 'Inter', 'Noto Sans JP', sans-serif;
  }
  @keyframes fade-in {
    from { opacity: 0; transform: translateY(-10px); }
    to { opacity: 1; transform: translateY(0); }
  }
  .animate-fade-in {
    animation: fade-in 0.6s ease-out forwards;
  }
`;
document.head.appendChild(style);
