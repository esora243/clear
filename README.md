<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="ストレスや心の不調について、専門家（臨床心理士・精神科専門医）に相談できるオンライン窓口です。簡単なセルフチェックや、日々の心の記録も行えます。">
    <title>こころの相談窓口</title>
    
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;500;700&display=swap" rel="stylesheet">
    
    <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    
    <style>
        body { font-family: 'Noto Sans JP', sans-serif; -webkit-tap-highlight-color: transparent; scroll-behavior: smooth; background-color: #F8F9FA; }
        .hero-bg { background: linear-gradient(to bottom, #EFF6FF, #F8F9FA); }
        .section-title { border-left: 5px solid #3B82F6; padding-left: 1rem; }
        .focus-ring:focus-within, input:focus, textarea:focus { box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.5); outline: none; }
        .card-hover { transition: transform 0.3s ease, box-shadow 0.3s ease; }
        .card-hover:hover { transform: translateY(-5px); box-shadow: 0 10px 20px rgba(0,0,0,0.08); }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useMemo, useRef } = React;

        // --- アイコン ---
        const CheckCircleIcon = (props) => <svg {...props} fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" /></svg>;
        const CalendarIcon = (props) => <svg {...props} fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" /></svg>;
        const HomeIcon = (props) => <svg {...props} fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6" /></svg>;
        const CloseIcon = (props) => <svg {...props} fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" /></svg>;
        const ChevronLeftIcon = (props) => <svg {...props} fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M15 19l-7-7 7-7" /></svg>;
        const ChevronRightIcon = (props) => <svg {...props} fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M9 5l7 7-7 7" /></svg>;
        const HeartIcon = (props) => <svg {...props} fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4.318 6.318a4.5 4.5 0 016.364 0L12 7.5l1.318-1.182a4.5 4.5 0 116.364 6.364L12 21l-7.682-7.682a4.5 4.5 0 010-6.364z" /></svg>;

        // --- 静的データ ---
        // ★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★
        // ★ ここを編集してください ★
        // GITHUBのユーザー名とリポジトリ名をあなたのものに書き換えてください
        const GITHUB_USER = "esora243"; // 例: "tanaka-ichiro"
        const GITHUB_REPO = "clear"; // 例: "kokoro-app-assets"
        const GITHUB_BRANCH = "main"; // 通常は "main" のままで問題ありません
        // ★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★

        const createImageUrl = (fileName) => `https://raw.githubusercontent.com/${GITHUB_USER}/${GITHUB_REPO}/${GITHUB_BRANCH}/${encodeURIComponent(fileName)}`;

        const STATIC_DOCTORS = [
            { id: 'doc1', nickname: '臨床心理士 高田先生', specialty: '人間関係の悩み・キャリアの不安', image: createImageUrl('女性医師.jpg') },
            { id: 'doc2', nickname: '精神科専門医 田辺先生', specialty: '気分の落ち込み・不眠・ストレス関連症状', image: 'https://placehold.co/128x128/E0E7FF/3730A3?text=田辺先生' },
        ];
        const CHECKLIST_ITEMS = ["最近、よく眠れない", "仕事や学業に集中できない", "理由もなく気分が落ち込む", "食欲がわかない、または食べ過ぎる", "何事も楽しめなくなった", "将来への不安が強い"];
        const MOODS = { '😄': '快調', '🙂': '良好', '😐': '普通', '😔': '不調', '😭': '絶不調' };

        // --- ローカルストレージ用カスタムフック ---
        const useLocalStorage = (key, initialValue) => {
            const [storedValue, setStoredValue] = useState(() => {
                try {
                    const item = window.localStorage.getItem(key);
                    return item ? JSON.parse(item) : initialValue;
                } catch (error) { return initialValue; }
            });
            const setValue = (value) => {
                try {
                    const valueToStore = value instanceof Function ? value(storedValue) : value;
                    setStoredValue(valueToStore);
                    window.localStorage.setItem(key, JSON.stringify(valueToStore));
                } catch (error) { console.error(error); }
            };
            return [storedValue, setValue];
        };

        // --- メインアプリケーション ---
        const App = () => {
            const [screen, setScreen] = useState('HOME'); // HOME, CALENDAR
            const [calendarRecords, setCalendarRecords] = useLocalStorage('calendarRecords', {});
            const [appointments, setAppointments] = useLocalStorage('appointments', []);

            const renderScreen = () => {
                switch(screen) {
                    case 'HOME': return <HomeScreen setAppointments={setAppointments} />;
                    case 'CALENDAR': return <CalendarScreen records={calendarRecords} setRecords={setCalendarRecords} appointments={appointments} />;
                    default: return <HomeScreen setAppointments={setAppointments} />;
                }
            };
            
            return (
                <div className="w-full max-w-4xl mx-auto bg-white shadow-xl">
                    <div className="min-h-screen pb-20"> {/* フッター分の余白 */}
                        {renderScreen()}
                    </div>
                    <Footer active={screen} onNavigate={setScreen} />
                </div>
            );
        };

        // --- フッターナビゲーション ---
        const Footer = ({ active, onNavigate }) => (
            <footer className="fixed bottom-0 left-0 right-0 max-w-4xl mx-auto flex justify-around p-2 border-t bg-white/90 backdrop-blur-sm shadow-inner z-10">
                <button onClick={() => onNavigate('HOME')} className={`w-1/2 flex flex-col items-center pt-1 pb-1 transition-colors duration-200 ${active === 'HOME' ? 'text-blue-600' : 'text-gray-500 hover:text-blue-600'}`}>
                    <HomeIcon className="h-6 w-6" /><span className="text-xs font-bold">相談・料金</span>
                </button>
                <button onClick={() => onNavigate('CALENDAR')} className={`w-1/2 flex flex-col items-center pt-1 pb-1 transition-colors duration-200 ${active === 'CALENDAR' ? 'text-blue-600' : 'text-gray-500 hover:text-blue-600'}`}>
                    <CalendarIcon className="h-6 w-6" /><span className="text-xs font-bold">セルフケア</span>
                </button>
            </footer>
        );

        // --- ホーム（相談）ページ ---
        const HomeScreen = ({ setAppointments }) => {
            const bookingFormRef = useRef(null);
            return (
                <div>
                    <header className="hero-bg text-center p-8 sm:p-16">
                        <img src={createImageUrl('ストレスのロゴ.jpg')} alt="こころの相談窓口 ロゴ" className="w-24 h-24 mx-auto mb-4 rounded-full shadow-lg" onError={(e) => e.target.src='https://placehold.co/96x96/E0E7FF/3730A3?text=ロゴ'}/>
                        <h1 className="text-3xl sm:text-5xl font-bold text-gray-800 leading-tight">ひとりで悩んでいませんか？</h1>
                        <p className="mt-4 text-lg text-gray-600 max-w-2xl mx-auto">経験豊富な専門家が、あなたの心に優しく寄り添います。<br/>安心して話せる場所が、ここにあります。</p>
                        <button onClick={() => bookingFormRef.current.scrollIntoView()} className="mt-8 bg-blue-600 text-white font-bold py-4 px-10 rounded-full text-lg hover:bg-blue-700 transition-all duration-300 shadow-lg hover:shadow-xl transform hover:scale-105">
                            今すぐ専門家に相談する
                        </button>
                    </header>
                    <main className="p-4 sm:p-10 space-y-20">
                        <section>
                            <h2 className="text-3xl font-bold text-gray-800 section-title">こんなお悩みありませんか？</h2>
                            <div className="mt-8 grid grid-cols-1 md:grid-cols-2 gap-10 items-center">
                                <div className="space-y-5">
                                    {CHECKLIST_ITEMS.map(item => (
                                        <div key={item} className="flex items-center bg-white p-4 rounded-lg shadow-sm">
                                            <CheckCircleIcon className="h-7 w-7 text-green-500 mr-4 flex-shrink-0" />
                                            <span className="text-gray-700 text-lg">{item}</span>
                                        </div>
                                    ))}
                                </div>
                                <img src={createImageUrl('チェックリスト.jpg')} alt="悩みを抱える人のイラスト" className="rounded-lg shadow-lg object-cover w-full h-full" onError={(e) => e.target.src='https://placehold.co/600x400/E2E8F0/4A5568?text=セルフチェック'}/>
                            </div>
                            <p className="mt-8 text-gray-600 text-center">ひとつでも当てはまったら、心が休息を求めているサインかもしれません。<br/>専門家に話すことで、気持ちが楽になることがあります。</p>
                        </section>
                        <section>
                            <h2 className="text-3xl font-bold text-gray-800 section-title">私たちの想い</h2>
                            <div className="mt-8 flex flex-col md:flex-row gap-10 items-center">
                                <img src={createImageUrl('ストレスの説明.png')} alt="リラックスしている人のイラスト" className="rounded-lg shadow-lg md:w-2/5" onError={(e) => e.target.src='https://placehold.co/400x400/E2E8F0/4A5568?text=ストレスの説明'}/>
                                <div className="md:w-3/5">
                                    <p className="text-gray-700 text-lg leading-relaxed">私たちは、誰もが心の健康を当たり前に大切にできる社会を目指しています。ストレスは特別なことではありません。身体の不調と同じように、心の不調にも専門家のサポートが必要です。<br/><br/>あなたのペースで、あなたの言葉で、ゆっくりとお話をお聞かせください。私たちは、あなたが自分らしい毎日を取り戻すためのお手伝いをします。</p>
                                </div>
                            </div>
                        </section>
                        <section>
                            <h2 className="text-3xl font-bold text-gray-800 section-title">専門家のご紹介</h2>
                            <div className="mt-8 grid grid-cols-1 md:grid-cols-2 gap-10">
                                {STATIC_DOCTORS.map(doctor => (
                                    <div key={doctor.id} className="bg-white p-8 rounded-xl shadow-lg text-center card-hover">
                                        <img src={doctor.image} alt={`${doctor.nickname}の顔写真`} className="w-32 h-32 mx-auto rounded-full mb-5 object-cover ring-4 ring-white shadow-md" onError={(e) => e.target.src='https://placehold.co/128x128/CBD5E0/4A5568?text=先生'}/>
                                        <h3 className="text-2xl font-bold text-gray-900">{doctor.nickname}</h3>
                                        <p className="text-blue-600 font-semibold my-2">{doctor.specialty}</p>
                                        <p className="text-gray-600 mt-4 text-left leading-relaxed">{doctor.bio}</p>
                                    </div>
                                ))}
                            </div>
                        </section>
                        <section>
                            <h2 className="text-3xl font-bold text-gray-800 section-title">ご利用料金</h2>
                            <PricingSection />
                        </section>
                        <section ref={bookingFormRef}>
                            <h2 className="text-3xl font-bold text-gray-800 section-title">オンライン相談 予約フォーム</h2>
                            <BookingForm setAppointments={setAppointments} />
                        </section>
                    </main>
                    <footer className="text-center p-8 bg-gray-800 text-white mt-20">
                        <p>&copy; 2025 こころの相談窓口. All Rights Reserved.</p>
                    </footer>
                </div>
            );
        };
        
        // --- 予約フォーム ---
        const BookingForm = ({ setAppointments }) => {
            const [name, setName] = useState('');
            const [phone, setPhone] = useState('');
            const [email, setEmail] = useState('');
            const [hope1, setHope1] = useState('');
            const [hope2, setHope2] = useState('');
            const [hope3, setHope3] = useState('');
            const [isSubmitted, setIsSubmitted] = useState(false);

            const handleSubmit = (e) => {
                e.preventDefault();
                if(!name || !phone || !email || !hope1) {
                    alert("氏名、電話番号、メールアドレス、および第1希望日時は必須です。");
                    return;
                }
                const newAppointments = [hope1, hope2, hope3].filter(Boolean).map(hopeDate => ({
                    id: `app_${Date.now()}_${Math.random()}`,
                    hopeDate: hopeDate
                }));
                setAppointments(prev => [...prev, ...newAppointments]);
                setIsSubmitted(true);
            };

            if (isSubmitted) {
                return (
                    <div className="bg-green-100 border-l-4 border-green-500 text-green-800 p-6 rounded-lg shadow-md mt-8 text-center">
                        <h3 className="text-xl font-bold">ご予約ありがとうございます</h3>
                        <p className="mt-2">担当者より折り返しご連絡いたしますので、今しばらくお待ちください。</p>
                    </div>
                );
            }

            return (
                <form onSubmit={handleSubmit} className="mt-8 bg-white p-8 rounded-xl shadow-lg space-y-6">
                    <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <div>
                            <label htmlFor="name" className="block text-sm font-medium text-gray-700 mb-1">お名前<span className="text-red-500">*</span></label>
                            <input type="text" id="name" value={name} onChange={e => setName(e.target.value)} required className="w-full p-3 border border-gray-300 rounded-lg focus-ring" />
                        </div>
                        <div>
                            <label htmlFor="phone" className="block text-sm font-medium text-gray-700 mb-1">電話番号<span className="text-red-500">*</span></label>
                            <input type="tel" id="phone" value={phone} onChange={e => setPhone(e.target.value)} required className="w-full p-3 border border-gray-300 rounded-lg focus-ring" />
                        </div>
                    </div>
                    <div>
                        <label htmlFor="email" className="block text-sm font-medium text-gray-700 mb-1">メールアドレス<span className="text-red-500">*</span></label>
                        <input type="email" id="email" value={email} onChange={e => setEmail(e.target.value)} required className="w-full p-3 border border-gray-300 rounded-lg focus-ring" />
                    </div>
                    <div>
                        <label className="block text-sm font-medium text-gray-700 mb-2">ご希望の日時</label>
                        <div className="space-y-3">
                            <input type="datetime-local" value={hope1} onChange={e => setHope1(e.target.value)} required className="w-full p-3 border border-gray-300 rounded-lg focus-ring" title="第1希望（必須）"/>
                            <input type="datetime-local" value={hope2} onChange={e => setHope2(e.target.value)} className="w-full p-3 border border-gray-300 rounded-lg focus-ring" title="第2希望"/>
                            <input type="datetime-local" value={hope3} onChange={e => setHope3(e.target.value)} className="w-full p-3 border border-gray-300 rounded-lg focus-ring" title="第3希望"/>
                        </div>
                    </div>
                    <button type="submit" className="w-full bg-blue-600 text-white font-bold py-3 px-8 rounded-lg text-lg hover:bg-blue-700 transition-all duration-300 shadow-lg hover:shadow-xl">
                        この内容で相談を予約する
                    </button>
                </form>
            );
        };

        // --- カレンダーページ ---
        const CalendarScreen = ({ records, setRecords, appointments }) => {
            const [currentDate, setCurrentDate] = useState(new Date());
            const [selectedDate, setSelectedDate] = useState(null);

            const handleSaveRecord = (newRecord) => {
                const recordId = selectedDate.toISOString().split('T')[0];
                setRecords(prev => ({ ...prev, [recordId]: newRecord }));
                setSelectedDate(null);
            };
            
            const calendarGrid = useMemo(() => {
                const year = currentDate.getFullYear();
                const month = currentDate.getMonth();
                const firstDayOfMonth = new Date(year, month, 1).getDay();
                const daysInMonth = new Date(year, month + 1, 0).getDate();
                const grid = [];
                let day = 1;
                for (let i = 0; i < 6; i++) {
                    const week = [];
                    for (let j = 0; j < 7; j++) {
                        if (i === 0 && j < firstDayOfMonth) { week.push(null); } 
                        else if (day > daysInMonth) { week.push(null); } 
                        else { week.push(new Date(year, month, day)); day++; }
                    }
                    grid.push(week);
                    if (day > daysInMonth) break;
                }
                return grid;
            }, [currentDate]);

            const appointmentDates = useMemo(() => {
                const dates = {};
                appointments.forEach(app => {
                    const dateStr = new Date(app.hopeDate).toISOString().split('T')[0];
                    if (!dates[dateStr]) dates[dateStr] = 0;
                    dates[dateStr]++;
                });
                return dates;
            }, [appointments]);

            return (
                <div className="p-4">
                    {selectedDate && <RecordModal date={selectedDate} record={records[selectedDate.toISOString().split('T')[0]]} onClose={() => setSelectedDate(null)} onSave={handleSaveRecord} />}
                    <header className="flex items-center justify-between p-2 mb-4">
                        <button onClick={() => setCurrentDate(new Date(currentDate.getFullYear(), currentDate.getMonth() - 1, 1))} className="p-2 rounded-full hover:bg-gray-200 transition"><ChevronLeftIcon className="h-6 w-6 text-gray-600" /></button>
                        <h1 className="text-2xl font-bold text-gray-800">{currentDate.getFullYear()}年 {currentDate.getMonth() + 1}月</h1>
                        <button onClick={() => setCurrentDate(new Date(currentDate.getFullYear(), currentDate.getMonth() + 1, 1))} className="p-2 rounded-full hover:bg-gray-200 transition"><ChevronRightIcon className="h-6 w-6 text-gray-600" /></button>
                    </header>
                    <main>
                        <div className="grid grid-cols-7 text-center font-bold text-sm text-gray-500 mb-2">
                            {['日', '月', '火', '水', '木', '金', '土'].map((day, i) => <div key={day} className={i === 0 ? 'text-red-500' : i === 6 ? 'text-blue-500' : ''}>{day}</div>)}
                        </div>
                        <div className="grid grid-cols-7 gap-2">
                            {calendarGrid.flat().map((date, index) => {
                                const recordId = date ? date.toISOString().split('T')[0] : null;
                                const record = recordId ? records[recordId] : null;
                                const isToday = date && new Date().toDateString() === date.toDateString();
                                const hasAppointment = recordId && appointmentDates[recordId];

                                return (
                                    <div key={index} onClick={() => date && setSelectedDate(date)} className={`h-24 sm:h-28 rounded-xl flex flex-col justify-start items-center p-1.5 cursor-pointer transition duration-300 relative ${date ? 'bg-white shadow-sm hover:shadow-md hover:-translate-y-0.5' : 'bg-transparent'}`}>
                                        {date && (
                                            <>
                                                {hasAppointment && <span className="absolute top-1.5 right-1.5 h-2.5 w-2.5 bg-blue-500 rounded-full" title="相談希望日"></span>}
                                                <span className={`font-semibold text-xs sm:text-sm ${isToday ? 'text-white bg-blue-500 rounded-full h-6 w-6 flex items-center justify-center' : ''}`}>{date.getDate()}</span>
                                                <div className="mt-1 text-2xl sm:text-3xl flex-1 flex items-center justify-center">{record?.mood || ''}</div>
                                            </>
                                        )}
                                    </div>
                                );
                            })}
                        </div>
                    </main>
                </div>
            );
        };

        const RecordModal = ({ date, record, onClose, onSave }) => {
            const [mood, setMood] = useState(record?.mood || null);
            return (
                <div className="fixed inset-0 bg-black bg-opacity-60 flex justify-center items-center z-50 p-4" onClick={onClose}>
                    <div className="bg-white rounded-2xl shadow-2xl w-full max-w-md max-h-[90vh] flex flex-col" onClick={e => e.stopPropagation()}>
                        <header className="p-4 border-b flex justify-between items-center">
                            <h2 className="font-bold text-lg text-gray-800">{date.toLocaleDateString('ja-JP', { month: 'long', day: 'numeric', weekday: 'short' })}の気分</h2>
                            <button onClick={onClose} className="p-1 rounded-full text-gray-500 hover:bg-gray-200 transition"><CloseIcon className="h-6 w-6" /></button>
                        </header>
                        <main className="p-5 flex-1 flex flex-col justify-center">
                            <div className="flex justify-around">
                                {Object.keys(MOODS).map((emoji) => (
                                    <button key={emoji} onClick={() => setMood(emoji)} className={`text-4xl p-2 rounded-full transition-transform duration-200 ${mood === emoji ? 'transform scale-125 bg-blue-100' : 'hover:transform scale-110'}`} title={MOODS[emoji]}>{emoji}</button>
                                ))}
                            </div>
                        </main>
                        <footer className="p-4 border-t bg-gray-50 rounded-b-2xl">
                            <button onClick={() => onSave({ mood })} className="w-full bg-blue-600 text-white font-bold py-3 rounded-lg hover:bg-blue-700 transition-all duration-300 shadow-lg hover:shadow-xl">気分を記録する</button>
                        </footer>
                    </div>
                </div>
            );
        };

        // --- 料金セクション ---
        const PricingSection = () => (
            <div className="mt-8 grid grid-cols-1 md:grid-cols-3 gap-8 text-center">
                <div className="bg-white p-8 rounded-xl shadow-lg border-t-4 border-blue-300 card-hover">
                    <h3 className="text-xl font-bold text-gray-800">お試し相談</h3>
                    <p className="mt-4 text-4xl font-bold text-gray-900">3,000<span className="text-lg font-medium">円</span></p>
                    <p className="text-gray-500">/ 30分</p>
                    <ul className="mt-6 space-y-2 text-gray-600">
                        <li><span className="text-green-500 mr-2">✔</span>初めての方におすすめ</li>
                        <li><span className="text-green-500 mr-2">✔</span>気軽に相談したい</li>
                    </ul>
                </div>
                <div className="bg-white p-8 rounded-xl shadow-xl border-t-4 border-blue-500 md:transform md:scale-105 relative card-hover">
                    <p className="absolute top-0 -translate-y-1/2 left-1/2 -translate-x-1/2 bg-blue-500 text-white text-sm font-bold px-3 py-1 rounded-full">一番人気</p>
                    <h3 className="text-xl font-bold text-gray-800">標準プラン</h3>
                    <p className="mt-4 text-4xl font-bold text-gray-900">5,000<span className="text-lg font-medium">円</span></p>
                    <p className="text-gray-500">/ 50分</p>
                    <ul className="mt-6 space-y-2 text-gray-600">
                        <li><span className="text-green-500 mr-2">✔</span>じっくり話したい方</li>
                        <li><span className="text-green-500 mr-2">✔</span>継続的なサポート</li>
                    </ul>
                </div>
                <div className="bg-white p-8 rounded-xl shadow-lg border-t-4 border-blue-300 card-hover">
                    <h3 className="text-xl font-bold text-gray-800">月額プラン</h3>
                    <p className="mt-4 text-4xl font-bold text-gray-900">18,000<span className="text-lg font-medium">円</span></p>
                    <p className="text-gray-500">/ 月4回 (50分/回)</p>
                    <ul className="mt-6 space-y-2 text-gray-600">
                        <li><span className="text-green-500 mr-2">✔</span>定期的なカウンセリング</li>
                        <li><span className="text-green-500 mr-2">✔</span>最もお得なプラン</li>
                    </ul>
                </div>
            </div>
        );

        ReactDOM.createRoot(document.getElementById('root')).render(<App />);
    </script>
</body>
</html>
F
