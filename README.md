<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>心のアプリ</title>
    
    <!-- スタイルシート (Tailwind CSS) と Google Fonts を読み込みます -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;500;700&display=swap" rel="stylesheet">
    
    <!-- Reactライブラリを読み込みます -->
    <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    
    <!-- JSX (HTMLのような記法) をブラウザで解釈するためのBabelを読み込みます -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    
    <style>
        body { font-family: 'Noto Sans JP', sans-serif; -webkit-tap-highlight-color: transparent; }
        .modal-enter-active { transition: all 200ms ease-in-out; }
        .modal-exit-active { transition: all 200ms ease-in-out; }
        .modal-enter { opacity: 0; transform: translateY(20px); }
        .modal-enter-active { opacity: 1; transform: translateY(0); }
        .modal-exit { opacity: 1; transform: translateY(0); }
        .modal-exit-active { opacity: 0; transform: translateY(20px); }
    </style>
</head>
<body class="bg-gray-100">
    <!-- Reactアプリケーションがこの div の中に描画されます -->
    <div id="root"></div>

    <!-- ここから下がアプリケーション本体のJavaScript (React) コードです -->
    <script type="text/babel">
        const { useState, useEffect, useMemo, useRef } = React;

        // --- 静的データ（本来はAPIから取得） ---
        const STATIC_DOCTORS = [
            { id: 'doc1', nickname: '佐藤 先生', specialty: '臨床心理士', bio: 'あなたの心に寄り添い、共に歩んでいきたいです。どんな小さなお悩みでも、お気軽にご相談ください。' },
            { id: 'doc2', nickname: '鈴木 先生', specialty: '精神科専門医', bio: '現代社会でのストレスは様々です。仕事の悩みを専門的な観点からサポートします。' },
            { id: 'doc3', nickname: '高橋 先生', specialty: '公認心理師', bio: '良い睡眠は心の健康の第一歩。あなたの睡眠サイクルを整えるお手伝いをします。' },
        ];

        // --- 定数とアイコン ---
        const MOODS = { '😄': { label: 'とても良い' }, '🙂': { label: '良い' }, '😐': { label: '普通' }, '😔': { label: '不調' }, '😭': { label: 'とても不調' } };
        const ACTIVITIES = { '🚶': '散歩', '💊': '服薬', '☀️': '日光浴', '📖': '読書', '💪': '運動', '💬': '会話' };
        const CalendarIcon = (props) => <svg {...props} fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" /></svg>;
        const SearchIcon = (props) => <svg {...props} fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" /></svg>;
        const VideoCameraIcon = (props) => <svg {...props} fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M15 10l4.553-2.276A1 1 0 0121 8.618v6.764a1 1 0 01-1.447.894L15 14M5 18h8a2 2 0 002-2V8a2 2 0 00-2-2H5a2 2 0 00-2 2v8a2 2 0 002 2z" /></svg>;
        const CloseIcon = (props) => <svg {...props} fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" /></svg>;
        const ChevronLeftIcon = (props) => <svg {...props} fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M15 19l-7-7 7-7" /></svg>;
        const ChevronRightIcon = (props) => <svg {...props} fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M9 5l7 7-7 7" /></svg>;

        // --- ローカルストレージ用カスタムフック ---
        const useLocalStorage = (key, initialValue) => {
            const [storedValue, setStoredValue] = useState(() => {
                try {
                    const item = window.localStorage.getItem(key);
                    return item ? JSON.parse(item) : initialValue;
                } catch (error) {
                    console.error(error);
                    return initialValue;
                }
            });
            const setValue = (value) => {
                try {
                    const valueToStore = value instanceof Function ? value(storedValue) : value;
                    setStoredValue(valueToStore);
                    window.localStorage.setItem(key, JSON.stringify(valueToStore));
                } catch (error) {
                    console.error(error);
                }
            };
            return [storedValue, setValue];
        };

        // --- メインアプリケーション ---
        const App = () => {
            const [screen, setScreen] = useState('CALENDAR');
            const [selectedDoctor, setSelectedDoctor] = useState(null);
            const [calendarRecords, setCalendarRecords] = useLocalStorage('calendarRecords', {});
            const [appointments, setAppointments] = useLocalStorage('appointments', []);

            const navigateToBooking = (doctor) => {
                setSelectedDoctor(doctor);
                setScreen('BOOKING');
            };

            const renderScreen = () => {
                switch(screen) {
                    case 'CALENDAR': return <CalendarScreen records={calendarRecords} setRecords={setCalendarRecords} />;
                    case 'DOCTOR_LIST': return <DoctorListScreen onSelectDoctor={navigateToBooking} />;
                    case 'BOOKING': return <BookingScreen doctor={selectedDoctor} appointments={appointments} setAppointments={setAppointments} onBack={() => setScreen('DOCTOR_LIST')} />;
                    case 'APPOINTMENTS': return <AppointmentsScreen appointments={appointments} />;
                    default: return <CalendarScreen records={calendarRecords} setRecords={setCalendarRecords} />;
                }
            };
            
            return (
                <div className="w-full h-screen max-w-lg mx-auto bg-gray-100 flex flex-col">
                    <main className="flex-1 overflow-y-auto">{renderScreen()}</main>
                    <Footer active={screen} onNavigate={setScreen} />
                </div>
            );
        };

        // --- フッター ---
        const Footer = ({ active, onNavigate }) => (
            <footer className="flex justify-around p-2 border-t bg-white shadow-inner sticky bottom-0">
                <button onClick={() => onNavigate('CALENDAR')} className={`w-1/3 flex flex-col items-center pt-1 pb-1 transition-colors duration-200 ${active === 'CALENDAR' ? 'text-blue-600' : 'text-gray-500'}`}><CalendarIcon className="h-6 w-6" /><span className="text-xs font-bold">カレンダー</span></button>
                <button onClick={() => onNavigate('DOCTOR_LIST')} className={`w-1/3 flex flex-col items-center pt-1 pb-1 transition-colors duration-200 ${['DOCTOR_LIST', 'BOOKING'].includes(active) ? 'text-blue-600' : 'text-gray-500'}`}><SearchIcon className="h-6 w-6" /><span className="text-xs font-bold">専門家を探す</span></button>
                <button onClick={() => onNavigate('APPOINTMENTS')} className={`w-1/3 flex flex-col items-center pt-1 pb-1 transition-colors duration-200 ${active === 'APPOINTMENTS' ? 'text-blue-600' : 'text-gray-500'}`}><VideoCameraIcon className="h-6 w-6" /><span className="text-xs font-bold">予約一覧</span></button>
            </footer>
        );

        // --- カレンダー関連コンポーネント ---
        const CalendarScreen = ({ records, setRecords }) => {
            const [currentDate, setCurrentDate] = useState(new Date());
            const [selectedDate, setSelectedDate] = useState(null);

            const handleSaveRecord = (newRecord) => {
                const recordId = selectedDate.toISOString().split('T')[0];
                setRecords(prev => ({ ...prev, [recordId]: newRecord }));
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
                                return (
                                    <div key={index} onClick={() => date && setSelectedDate(date)} className={`h-24 sm:h-28 rounded-xl flex flex-col justify-start items-center p-2 cursor-pointer transition duration-300 ${date ? 'bg-white shadow-sm hover:shadow-lg hover:-translate-y-1' : 'bg-transparent'}`}>
                                        {date && (
                                            <>
                                                <span className={`font-bold text-sm ${isToday ? 'text-white bg-blue-500 rounded-full h-6 w-6 flex items-center justify-center' : ''}`}>{date.getDate()}</span>
                                                <div className="mt-2 text-3xl flex-1 flex items-center justify-center">{record?.mood || ''}</div>
                                                {record?.activities?.length > 0 && <div className="text-xs text-gray-400 flex flex-wrap justify-center gap-1">{record.activities.map(a => <span key={a}>{a}</span>)}</div>}
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
            const [selectedActivities, setSelectedActivities] = useState(record?.activities || []);
            const [journal, setJournal] = useState(record?.journal || '');
            const [sleepHours, setSleepHours] = useState(record?.sleepHours ?? 7);
            
            return (
                <div className="fixed inset-0 bg-black bg-opacity-60 flex justify-center items-center z-50" onClick={onClose}>
                    <div className="bg-white rounded-2xl shadow-2xl w-11/12 max-w-md max-h-[90vh] flex flex-col modal-enter-active" onClick={e => e.stopPropagation()}>
                        <header className="p-4 border-b flex justify-between items-center">
                            <h2 className="font-bold text-lg text-gray-800">{date.toLocaleDateString('ja-JP', { month: 'long', day: 'numeric', weekday: 'short' })}の記録</h2>
                            <button onClick={onClose} className="p-1 rounded-full text-gray-500 hover:bg-gray-200 transition"><CloseIcon className="h-6 w-6" /></button>
                        </header>
                        <main className="p-5 space-y-6 overflow-y-auto">
                            <div className="p-4 bg-gray-50 rounded-xl">
                                <p className="font-semibold mb-3 text-gray-700">今日の気分</p>
                                <div className="flex justify-around">
                                    {Object.keys(MOODS).map((emoji) => (
                                        <button key={emoji} onClick={() => setMood(emoji)} className={`text-4xl p-2 rounded-full transition-transform duration-200 ${mood === emoji ? 'transform scale-125' : 'hover:transform scale-110'}`} title={MOODS[emoji].label}>{emoji}</button>
                                    ))}
                                </div>
                            </div>
                            <div className="p-4 bg-gray-50 rounded-xl">
                                <p className="font-semibold mb-3 text-gray-700">今日できたこと</p>
                                <div className="flex flex-wrap gap-3">
                                    {Object.keys(ACTIVITIES).map(activity => (
                                        <button key={activity} onClick={() => setSelectedActivities(prev => prev.includes(activity) ? prev.filter(a => a !== activity) : [...prev, activity])} className={`px-3 py-2 text-sm font-semibold rounded-full border-2 flex items-center gap-2 transition ${selectedActivities.includes(activity) ? 'bg-blue-500 text-white border-blue-500' : 'bg-white text-gray-700 border-gray-300 hover:border-blue-400'}`}>
                                            <span className="text-lg">{activity}</span> {ACTIVITIES[activity]}
                                        </button>
                                    ))}
                                </div>
                            </div>
                            <div className="p-4 bg-gray-50 rounded-xl">
                                <p className="font-semibold mb-2 text-gray-700">睡眠時間: <span className="font-bold text-blue-600">{sleepHours.toFixed(1)}</span>時間</p>
                                <input type="range" min="0" max="12" step="0.5" value={sleepHours} onChange={(e) => setSleepHours(parseFloat(e.target.value))} className="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500" />
                            </div>
                            <div className="p-4 bg-gray-50 rounded-xl">
                                <p className="font-semibold mb-2 text-gray-700">日記 (この端末だけに保存されます)</p>
                                <textarea value={journal} onChange={e => setJournal(e.target.value)} placeholder="感じたことや出来事を自由にメモ..." className="w-full h-28 p-3 bg-white border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-400 transition"></textarea>
                            </div>
                        </main>
                        <footer className="p-4 border-t bg-gray-50 rounded-b-2xl">
                            <button onClick={() => onSave({ mood, activities: selectedActivities, journal, sleepHours })} className="w-full bg-blue-600 text-white font-bold py-3 rounded-lg hover:bg-blue-700 transition-all duration-300 shadow-lg hover:shadow-xl focus:outline-none focus:ring-4 focus:ring-blue-300">記録を保存する</button>
                        </footer>
                    </div>
                </div>
            );
        };

        // --- 予約関連コンポーネント ---
        const DoctorListScreen = ({ onSelectDoctor }) => (
            <div className="p-4">
                <h1 className="text-2xl font-bold mb-4 text-gray-800">専門家を探す</h1>
                <div className="space-y-4">
                    {STATIC_DOCTORS.map(doctor => (
                        <div key={doctor.id} onClick={() => onSelectDoctor(doctor)} className="bg-white p-5 rounded-xl shadow-md cursor-pointer hover:shadow-lg hover:-translate-y-1 transition-all duration-300">
                            <h2 className="font-bold text-lg text-gray-900">{doctor.nickname}</h2>
                            <p className="text-sm text-blue-600 font-semibold my-1">{doctor.specialty}</p>
                            <p className="text-gray-600 mt-2">{doctor.bio}</p>
                        </div>
                    ))}
                </div>
            </div>
        );

        const BookingScreen = ({ doctor, appointments, setAppointments, onBack }) => {
            const [selectedSlot, setSelectedSlot] = useState(null);
            
            const availableSlots = useMemo(() => {
                const slots = [];
                const today = new Date();
                for (let i = 1; i < 8; i++) {
                    const date = new Date(today);
                    date.setDate(today.getDate() + i);
                    slots.push(new Date(date.setHours(10, 0, 0, 0)));
                    slots.push(new Date(date.setHours(14, 0, 0, 0)));
                }
                return slots.filter(slot => !appointments.some(app => new Date(app.appointmentTime).getTime() === slot.getTime() && app.doctorId === doctor.id));
            }, [doctor.id, appointments]);

            const handleBooking = () => {
                if (!selectedSlot) { alert("時間を選択してください。"); return; }
                const newAppointment = {
                    id: 'app' + Date.now(),
                    doctorId: doctor.id,
                    doctorName: doctor.nickname,
                    appointmentTime: selectedSlot.toISOString(),
                    status: 'booked'
                };
                setAppointments(prev => [...prev, newAppointment]);
                alert("予約が完了しました。");
                onBack();
            };

            return (
                <div className="p-4">
                    <button onClick={onBack} className="mb-4 font-bold text-blue-600 flex items-center"><ChevronLeftIcon className="h-5 w-5 mr-1" />専門家一覧に戻る</button>
                    <div className="bg-white p-5 rounded-xl shadow-md mb-6">
                        <h1 className="text-2xl font-bold text-gray-900">予約: {doctor.nickname}</h1>
                        <p className="text-gray-600">{doctor.specialty}</p>
                    </div>
                    <h2 className="font-bold mb-3 text-lg text-gray-800">ご希望の時間を選択</h2>
                    <div className="grid grid-cols-2 gap-3">
                        {availableSlots.map(slot => (
                            <button key={slot.toISOString()} onClick={() => setSelectedSlot(slot)} className={`p-3 border rounded-lg text-center transition ${selectedSlot?.getTime() === slot.getTime() ? 'bg-blue-600 text-white shadow-lg' : 'bg-white hover:bg-blue-50'}`}>
                                {slot.toLocaleString('ja-JP', { month: 'short', day: 'numeric', hour: '2-digit', minute: '2-digit' })}
                            </button>
                        ))}
                    </div>
                    <button onClick={handleBooking} disabled={!selectedSlot} className="w-full mt-6 bg-blue-600 text-white font-bold py-3 rounded-lg hover:bg-blue-700 transition disabled:bg-gray-400 disabled:cursor-not-allowed">この時間で予約する</button>
                </div>
            );
        };

        const AppointmentsScreen = ({ appointments }) => {
            const meetLink = "https://meet.google.com/vpz-agds-htd";
            return (
                <div className="p-4">
                    <h1 className="text-2xl font-bold mb-4 text-gray-800">予約一覧</h1>
                    <div className="space-y-4">
                        {appointments.length > 0 ? appointments.sort((a,b) => new Date(a.appointmentTime) - new Date(b.appointmentTime)).map(app => (
                            <div key={app.id} className="bg-white p-5 rounded-xl shadow-md">
                                <p className="font-bold text-lg text-gray-900">{app.doctorName}</p>
                                <p className="text-gray-600">{new Date(app.appointmentTime).toLocaleString('ja-JP', { year: 'numeric', month: 'long', day: 'numeric', hour: '2-digit', minute: '2-digit' })}</p>
                                <a href={meetLink} target="_blank" rel="noopener noreferrer" className="block text-center mt-4 w-full bg-green-500 text-white font-bold py-2 rounded-lg hover:bg-green-600 transition flex items-center justify-center gap-2">
                                    <VideoCameraIcon className="h-5 w-5"/>面談を開始する
                                </a>
                            </div>
                        )) : <p className="text-gray-500 text-center mt-16">予約はまだありません。</p>}
                    </div>
                </div>
            );
        };

        ReactDOM.createRoot(document.getElementById('root')).render(<App />);

    </script>
</body>
</html>
