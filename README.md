import React, { useState, useRef } from 'react';

const INITIAL_CARDS = [
  { id: 1, name: "ジャンボン・ペルシェ", nameEn: "Jambon Persillé" },
  { id: 2, name: "真鯛のカルパッチョ", nameEn: "Carpaccio de Daurade" },
  { id: 3, name: "オマール海老のビスク", nameEn: "Bisque de Homard" },
  { id: 4, name: "国産牛頬肉の赤ワイン煮", nameEn: "Joue de Bœuf Bourguignonne" },
  { id: 5, name: "鴨のロースト", nameEn: "Canard Rôti" },
  { id: 6, name: "季節のキッシュ", nameEn: "Quiche de Saison" },
  { id: 7, name: "サーモンのミ・キュイ", nameEn: "Saumon Mi-cuit" },
  { id: 8, name: "フォアグラのポワレ", nameEn: "Foie Gras Poêlé" },
  { id: 9, name: "クレーム・ブリュレ", nameEn: "Crème Brûlée" },
  { id: 10, name: "オペラ", nameEn: "Opéra" },
];

const CardTemplate = ({ card }) => (
  <div className="w-[91mm] h-[55mm] bg-white flex flex-col justify-center items-center relative overflow-hidden box-border shadow-sm print:shadow-none print:border-none border border-gray-200" style={{ padding: '8mm' }}>
    <div className="absolute top-3 w-12 border-t-[0.5px] border-gray-400"></div>
    <div className="flex-grow flex flex-col justify-center items-center text-center w-full">
      <h2 className="text-[14px] text-gray-900 leading-snug tracking-widest mb-2" style={{ fontFamily: "'Noto Serif JP', 'Yu Mincho', 'Klee One', serif", fontFeatureSettings: '"palt" 1' }}>
        {card.name}
      </h2>
      {card.nameEn && (
        <p className="text-[9px] text-gray-500 tracking-widest uppercase" style={{ fontFamily: "'Cinzel', 'Times New Roman', serif" }}>
          {card.nameEn}
        </p>
      )}
    </div>
    <div className="absolute bottom-2 text-center w-full">
       <p className="text-[5px] text-gray-400 uppercase tracking-[0.3em]" style={{fontFamily: "'Cinzel', serif"}}>Kobe Kitano Terrace</p>
    </div>
  </div>
);

export default function App() {
  const [cards, setCards] = useState(INITIAL_CARDS);
  const [translatingId, setTranslatingId] = useState(null);
  const [globalMessage, setGlobalMessage] = useState({ text: "", type: "" });
  const printRef = useRef(null);

  const handlePrint = () => {
    window.print();
  };

  const updateCard = (id, field, value) => {
    setCards(cards.map(card => card.id === id ? { ...card, [field]: value } : card));
  };

  const handleTranslate = async (id, japaneseName) => {
    if (!japaneseName) return;
    setTranslatingId(id);
    setGlobalMessage({ text: "", type: "" });
    
    const apiKey = ""; // 実行環境から提供されます
    const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;
    const prompt = `高級フレンチレストラン「Kobe Kitano Terrace」のメニューに載せるための、以下の日本語の料理名にふさわしい欧文表記（英語またはフランス語）を出力してください。単なる直訳ではなく、例えば「ジャンボン・ペルシェ」なら「Jambon Persillé」のように、美食家がメニューとして認知する正式な料理名（フランス料理由来のものはフランス語表記を優先）にしてください。出力は料理名のみ（余計な説明や記号は一切不要）としてください。大文字小文字のスタイルはTitle Case（単語の先頭を大文字）にしてください。\n料理名: ${japaneseName}`;
    
    const payload = { contents: [{ parts: [{ text: prompt }] }] };

    const fetchWithRetry = async (url, options, retries = 5) => {
      const delays = [1000, 2000, 4000, 8000, 16000];
      for (let i = 0; i < retries; i++) {
        try {
          const response = await fetch(url, options);
          if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
          return await response.json();
        } catch (error) {
          if (i === retries - 1) throw error;
          await new Promise(res => setTimeout(res, delays[i]));
        }
      }
    };

    try {
      const result = await fetchWithRetry(url, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });
      const translatedText = result.candidates?.[0]?.content?.parts?.[0]?.text;
      
      if (translatedText) {
        updateCard(id, 'nameEn', translatedText.trim());
      } else {
         setGlobalMessage({ text: "翻訳結果を取得できませんでした。", type: "error" });
      }
    } catch (error) {
      console.error("Translation error:", error);
      setGlobalMessage({ text: "通信エラーが発生しました。しばらく経ってから再度お試しください。", type: "error" });
    } finally {
      setTranslatingId(null);
    }
  };

  return (
    <div className="min-h-screen bg-gray-100 font-sans flex flex-col lg:flex-row print:bg-white print:block">
      {/* 編集パネル（印刷時非表示） */}
      <div className="w-full lg:w-5/12 bg-white p-6 shadow-md overflow-y-auto h-screen print:hidden border-r border-gray-200">
        <h1 className="text-xl font-bold mb-2 text-gray-800 tracking-wider">シンプル料理名カード</h1>
        <p className="text-xs text-gray-500 mb-6">A4サイズ（10面）名刺横型。余白とフォントの美しさを重視したデザインです。</p>
        
        {globalMessage.text && (
          <div className={`mb-4 p-3 text-sm rounded ${globalMessage.type === 'error' ? 'bg-red-50 text-red-700 border border-red-200' : 'bg-blue-50 text-blue-700 border border-blue-200'}`}>
            {globalMessage.text}
          </div>
        )}

        <button 
          onClick={handlePrint}
          className="w-full bg-slate-800 hover:bg-slate-900 text-white text-sm font-semibold py-3 px-4 rounded mb-8 transition duration-200 flex justify-center items-center"
        >
          <svg className="w-4 h-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M17 17h2a2 2 0 002-2v-4a2 2 0 00-2-2H5a2 2 0 00-2 2v4a2 2 0 002 2h2m2 4h6a2 2 0 002-2v-4a2 2 0 00-2-2H9a2 2 0 00-2 2v4a2 2 0 002 2zm8-12V5a2 2 0 00-2-2H9a2 2 0 00-2 2v4h10z"></path></svg>
          A4用紙に印刷する
        </button>

        <div className="space-y-4">
          {cards.map((card, index) => (
            <div key={card.id} className="border-b border-gray-100 pb-4 relative">
              <div className="flex items-center mb-2">
                <span className="bg-slate-200 text-slate-700 w-5 h-5 rounded-full flex items-center justify-center text-[10px] font-bold mr-2">{index + 1}</span>
              </div>
              <div className="space-y-2">
                <div>
                  <label className="block text-[10px] font-medium text-gray-500 mb-1 uppercase tracking-wider">Japanese Name</label>
                  <input type="text" value={card.name} onChange={(e) => updateCard(card.id, 'name', e.target.value)} className="w-full p-2 border border-gray-200 rounded text-sm focus:ring-1 focus:ring-slate-500 focus:border-slate-500 transition-colors" />
                </div>
                <div>
                  <div className="flex justify-between items-center mb-1">
                    <label className="block text-[10px] font-medium text-gray-500 uppercase tracking-wider">English Name</label>
                    <button 
                      onClick={() => handleTranslate(card.id, card.name)}
                      disabled={translatingId === card.id || !card.name}
                      className="text-[10px] text-blue-600 hover:text-blue-800 disabled:text-gray-400 flex items-center transition-colors"
                      title="AIで英語表記を自動生成します"
                    >
                      {translatingId === card.id ? (
                        <><svg className="animate-spin w-3 h-3 mr-1" viewBox="0 0 24 24"><circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4" fill="none"></circle><path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>翻訳中...</>
                      ) : (
                        <><svg className="w-3 h-3 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M3 5h12M9 3v2m1.048 9.5A18.022 18.022 0 016.412 9m6.088 9h7M11 21l5-10 5 10M12.751 5C11.783 10.77 8.07 15.61 3 18.129"></path></svg>AI翻訳</>
                      )}
                    </button>
                  </div>
                  <input type="text" value={card.nameEn} onChange={(e) => updateCard(card.id, 'nameEn', e.target.value)} className="w-full p-2 border border-gray-200 rounded text-sm focus:ring-1 focus:ring-slate-500 focus:border-slate-500 transition-colors" />
                </div>
              </div>

              {/* 個別カードのリアルタイムプレビュー */}
              <div className="mt-4 bg-gray-50 rounded p-4 flex justify-center items-center border border-gray-100 overflow-hidden relative">
                <span className="absolute top-2 left-2 text-[8px] text-gray-400 font-bold uppercase tracking-widest">Preview</span>
                <div style={{ transform: 'scale(0.7)', transformOrigin: 'center', height: '38.5mm' }}>
                  <CardTemplate card={card} />
                </div>
              </div>

            </div>
          ))}
        </div>
      </div>

      {/* プレビュー・印刷領域 */}
      <div className="w-full lg:w-7/12 flex justify-center items-start p-8 overflow-x-auto print:w-full print:p-0 print:bg-white print:overflow-visible h-screen overflow-y-auto bg-gray-200">
        {/* A4キャンバス */}
        <div 
          ref={printRef}
          className="bg-white shadow-2xl print:shadow-none mx-auto relative print:m-0"
          style={{ 
            width: '210mm', 
            height: '297mm',
            padding: '11mm 14mm',
            boxSizing: 'border-box'
          }}
        >
          {/* センターガイドライン */}
          <div className="absolute top-0 bottom-0 left-[105mm] border-l border-dashed border-gray-200 print:border-gray-200 z-0"></div>
          
          <div className="grid grid-cols-2 grid-rows-5 h-full w-full relative z-10">
            {cards.slice(0, 10).map((card, index) => (
              <div 
                key={card.id} 
                className="border border-dashed border-gray-200 print:border-gray-100 flex items-center justify-center p-0 relative"
                style={{ width: '91mm', height: '55mm' }}
              >
                {/* コンポーネント化したカードデザインを呼び出し */}
                <CardTemplate card={card} />
              </div>
            ))}
          </div>
        </div>
      </div>
      
      {/* 印刷用スタイル */}
      <style dangerouslySetInnerHTML={{__html: `
        @import url('https://fonts.googleapis.com/css2?family=Cinzel:wght@400;500&family=Noto+Serif+JP:wght@300;400&display=swap');
        @media print {
          @page { margin: 0; size: A4 portrait; }
          body { margin: 0; padding: 0; background: white; -webkit-print-color-adjust: exact; print-color-adjust: exact; }
          html, body { width: 210mm; height: 297mm; }
        }
      `}} />
    </div>
  );
}
