import React, { useState, useEffect } from 'react';
import { Calendar, MapPin, Clock, Heart, Music, Phone, Info, ChevronRight, Sparkles, Send, Map, Loader2, MessageSquare } from 'lucide-react';

const App = () => {
  const [timeLeft, setTimeLeft] = useState({ days: 0, hours: 0, minutes: 0, seconds: 0 });
  const [blessingInput, setBlessingInput] = useState("");
  const [aiBlessing, setAiBlessing] = useState("");
  const [travelGuide, setTravelGuide] = useState("");
  const [isLoading, setIsLoading] = useState({ blessing: false, travel: false });
  const [chatMessage, setChatMessage] = useState("");
  const [chatResponse, setChatResponse] = useState("");
  const [isChatLoading, setIsChatLoading] = useState(false);

  // Event Date: Feb 5, 2026
  const eventDate = new Date('February 5, 2026 10:00:00').getTime();

  useEffect(() => {
    const timer = setInterval(() => {
      const now = new Date().getTime();
      const distance = eventDate - now;

      if (distance < 0) {
        clearInterval(timer);
      } else {
        setTimeLeft({
          days: Math.floor(distance / (1000 * 60 * 60 * 24)),
          hours: Math.floor((distance % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60)),
          minutes: Math.floor((distance % (1000 * 60 * 60)) / (1000 * 60)),
          seconds: Math.floor((distance % (1000 * 60)) / 1000),
        });
      }
    }, 1000);
    return () => clearInterval(timer);
  }, []);

  const geminiCall = async (prompt, systemInstruction, useSearch = false) => {
    const apiKey = ""; // Provided by environment
    let retries = 0;
    const maxRetries = 5;

    const payload = {
      contents: [{ parts: [{ text: prompt }] }],
      systemInstruction: { parts: [{ text: systemInstruction }] }
    };

    if (useSearch) {
      payload.tools = [{ "google_search": {} }];
    }

    const attempt = async (delay) => {
      try {
        const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });
        if (!response.ok) throw new Error('API request failed');
        const result = await response.json();
        return result.candidates?.[0]?.content?.parts?.[0]?.text;
      } catch (error) {
        if (retries < maxRetries) {
          retries++;
          await new Promise(res => setTimeout(res, delay));
          return attempt(delay * 2);
        }
        throw error;
      }
    };

    return attempt(1000);
  };

  const handleRefineBlessing = async () => {
    if (!blessingInput.trim()) return;
    setIsLoading(prev => ({ ...prev, blessing: true }));
    try {
      const response = await geminiCall(
        blessingInput,
        "You are an elegant Indian wedding assistant. Refine the user's blessing into a poetic, respectful, and heartwarming message for the couple Niharika and Pankaj. Use a mix of English and Hindi (written in Latin script) if appropriate."
      );
      setAiBlessing(response);
    } catch (e) {
      console.error(e);
    } finally {
      setIsLoading(prev => ({ ...prev, blessing: false }));
    }
  };

  const handleGetTravelGuide = async () => {
    setIsLoading(prev => ({ ...prev, travel: true }));
    try {
      const response = await geminiCall(
        "I am visiting Deoria, Uttar Pradesh for Niharika and Pankaj's wedding at Sukh Vatika Lawn. What are some local food specialties I should try and best places nearby to visit in Feb 2026?",
        "You are a local travel expert for Deoria, UP. Provide helpful, grounded advice for wedding guests.",
        true
      );
      setTravelGuide(response);
    } catch (e) {
      console.error(e);
    } finally {
      setIsLoading(prev => ({ ...prev, travel: false }));
    }
  };

  const handleChat = async () => {
    if (!chatMessage.trim()) return;
    setIsChatLoading(true);
    try {
      const systemPrompt = `You are the wedding assistant for Niharika and Pankaj's wedding.
      Event Details: 
      - Haldi: Feb 5, 2026, 10 AM.
      - Wedding: Feb 5, 2026, Shubh Lagna.
      - Vidai: Feb 6, 2026, 6 AM.
      - Venue: Sukh Vatika Lawn, Chakiyawa Road, New Colony, Deoria.
      - Hosts: Ashok Kumar Srivastava & Anamika Srivastava.
      Keep answers short and celebratory.`;
      
      const response = await geminiCall(chatMessage, systemPrompt);
      setChatResponse(response);
    } catch (e) {
      console.error(e);
    } finally {
      setIsChatLoading(false);
    }
  };

  const ScheduleItem = ({ title, date, time, icon: Icon }) => (
    <div className="flex items-start space-x-4 p-4 bg-white/80 rounded-2xl shadow-sm border border-orange-100 mb-4 transition-transform hover:scale-[1.02]">
      <div className="bg-orange-100 p-3 rounded-xl">
        <Icon className="text-orange-600 w-6 h-6" />
      </div>
      <div>
        <h4 className="font-bold text-gray-900">{title}</h4>
        <p className="text-sm text-gray-600">{date}</p>
        <p className="text-sm font-semibold text-orange-600">{time}</p>
      </div>
    </div>
  );

  const openMap = () => {
    window.open("https://maps.app.goo.gl/skAqF3SNNGcBtU159", "_blank");
  };

  return (
    <div className="min-h-screen bg-orange-50 font-serif text-gray-800 pb-10">
      {/* Hero Section */}
      <header className="relative h-[70vh] flex flex-col items-center justify-center text-center px-6 overflow-hidden">
        <div className="absolute inset-0 z-0 opacity-20 bg-[radial-gradient(circle_at_center,_var(--tw-gradient-stops))] from-orange-300 via-orange-50 to-orange-50"></div>
        
        <div className="z-10 max-w-2xl animate-in fade-in slide-in-from-bottom-8 duration-1000">
          <p className="text-orange-600 tracking-[0.3em] uppercase mb-4 text-xs font-bold">।। श्री गणेशाय नमः ।।</p>
          <h2 className="text-xl md:text-2xl text-orange-800 mb-6 italic">With the blessings of Almighty...</h2>
          
          <div className="space-y-2 mb-8">
            <h1 className="text-6xl md:text-8xl font-bold text-orange-950 tracking-tight">Niharika</h1>
            <p className="text-3xl font-light text-orange-400">&</p>
            <h1 className="text-6xl md:text-8xl font-bold text-orange-950 tracking-tight">Pankaj</h1>
          </div>
        </div>
      </header>

      {/* Countdown Section */}
      <section className="py-10 bg-white/40 backdrop-blur-sm -mt-20 relative z-20 shadow-lg mx-4 rounded-3xl border border-white/50">
        <div className="max-w-4xl mx-auto flex justify-around items-center text-center">
          {Object.entries(timeLeft).map(([label, value]) => (
            <div key={label} className="flex flex-col">
              <span className="text-3xl md:text-5xl font-bold text-orange-800">{value}</span>
              <span className="text-[10px] uppercase tracking-widest text-gray-400 font-sans">{label}</span>
            </div>
          ))}
        </div>
      </section>

      {/* AI Interactive Section */}
      <section className="max-w-6xl mx-auto px-6 py-16">
        <div className="mb-12 text-center">
          <h2 className="text-3xl font-bold text-orange-950 flex items-center justify-center gap-3">
            <Sparkles className="text-orange-500 animate-pulse" />
            Guest Concierge
          </h2>
          <p className="text-gray-600 mt-2">Personalized assistance for our dear guests</p>
        </div>

        <div className="grid md:grid-cols-3 gap-8">
          {/* AI Blessing Refiner */}
          <div className="bg-white p-6 rounded-3xl shadow-md border border-orange-100 flex flex-col h-full">
            <h3 className="text-lg font-bold text-orange-900 mb-4 flex items-center gap-2">
              <Heart size={18} className="text-orange-500" /> ✨ Refine Blessing
            </h3>
            <textarea
              className="w-full flex-grow p-4 bg-orange-50/50 rounded-xl border border-orange-100 focus:outline-none focus:ring-2 focus:ring-orange-200 text-sm mb-4 resize-none"
              placeholder="Type your message for the couple..."
              value={blessingInput}
              onChange={(e) => setBlessingInput(e.target.value)}
            />
            {aiBlessing && (
              <div className="p-4 bg-orange-100 rounded-xl mb-4 text-xs italic text-orange-900 leading-relaxed border-l-4 border-orange-400 animate-in fade-in slide-in-from-top-2">
                {aiBlessing}
              </div>
            )}
            <button
              onClick={handleRefineBlessing}
              disabled={isLoading.blessing || !blessingInput}
              className="w-full bg-orange-600 hover:bg-orange-700 disabled:opacity-50 text-white py-3 rounded-xl font-bold flex items-center justify-center gap-2 transition-all"
            >
              {isLoading.blessing ? <Loader2 className="animate-spin" size={18} /> : <Sparkles size={18} />}
              Make it Poetic ✨
            </button>
          </div>

          {/* AI Local Guide */}
          <div className="bg-orange-900 text-white p-6 rounded-3xl shadow-xl flex flex-col h-full relative overflow-hidden">
             <div className="absolute top-0 right-0 -mr-4 -mt-4 opacity-10">
                <Map size={120} />
             </div>
            <h3 className="text-lg font-bold text-orange-300 mb-4 flex items-center gap-2">
              <Map size={18} /> ✨ Deoria Guide
            </h3>
            <p className="text-xs text-orange-100/80 mb-6 leading-relaxed">
              Traveling to Deoria? Let our AI suggest local food and hidden spots to explore.
            </p>
            {travelGuide && (
              <div className="mb-6 p-4 bg-white/10 rounded-xl text-xs overflow-y-auto max-h-48 custom-scrollbar">
                {travelGuide}
              </div>
            )}
            <button
              onClick={handleGetTravelGuide}
              disabled={isLoading.travel}
              className="mt-auto w-full bg-orange-500 hover:bg-orange-400 disabled:opacity-50 text-white py-3 rounded-xl font-bold flex items-center justify-center gap-2 transition-all shadow-lg"
            >
              {isLoading.travel ? <Loader2 className="animate-spin" size={18} /> : <Sparkles size={18} />}
              Explore Local ✨
            </button>
          </div>

          {/* AI Wedding Assistant */}
          <div className="bg-white p-6 rounded-3xl shadow-md border border-orange-100 flex flex-col h-full">
            <h3 className="text-lg font-bold text-orange-900 mb-4 flex items-center gap-2">
              <MessageSquare size={18} className="text-orange-500" /> ✨ Wedding Assistant
            </h3>
            <div className="flex-grow flex flex-col">
              <div className="flex-grow bg-gray-50 rounded-xl p-4 text-xs overflow-y-auto max-h-48 mb-4 border border-gray-100">
                {chatResponse ? (
                  <div className="animate-in fade-in">
                    <p className="font-bold text-orange-800 mb-1 italic">Assistant:</p>
                    <p className="text-gray-700 leading-relaxed">{chatResponse}</p>
                  </div>
                ) : (
                  <p className="text-gray-400 italic text-center mt-10">Ask me anything about the venue, timings, or family!</p>
                )}
              </div>
              <div className="flex gap-2">
                <input
                  type="text"
                  className="flex-grow p-3 bg-gray-50 border border-gray-100 rounded-xl text-xs focus:outline-none focus:ring-2 focus:ring-orange-200"
                  placeholder="What is the dress code?"
                  value={chatMessage}
                  onChange={(e) => setChatMessage(e.target.value)}
                  onKeyPress={(e) => e.key === 'Enter' && handleChat()}
                />
                <button
                  onClick={handleChat}
                  disabled={isChatLoading || !chatMessage}
                  className="bg-orange-100 p-3 rounded-xl text-orange-600 hover:bg-orange-200 transition-all disabled:opacity-50"
                >
                  {isChatLoading ? <Loader2 className="animate-spin" size={18} /> : <Send size={18} />}
                </button>
              </div>
            </div>
          </div>
        </div>
      </section>

      {/* Details Grid */}
      <main className="max-w-6xl mx-auto px-6 py-10 grid md:grid-cols-2 gap-16">
        {/* Left Column: Schedule */}
        <div className="space-y-8">
          <div className="flex items-center space-x-2 border-b-2 border-orange-200 pb-2 mb-8">
            <Heart className="text-orange-600 fill-orange-600" size={20} />
            <h3 className="text-2xl font-bold text-orange-950">Wedding Program</h3>
          </div>
          
          <ScheduleItem 
            title="Haldi Ceremony" 
            date="Thursday, 5th February 2026" 
            time="10:00 AM Onwards" 
            icon={Music} 
          />
          <ScheduleItem 
            title="Shubh Vivah" 
            date="Thursday, 5th February 2026" 
            time="As per Shubh Lagna" 
            icon={Heart} 
          />
          <ScheduleItem 
            title="Vidai" 
            date="Friday, 6th February 2026" 
            time="06:00 AM" 
            icon={Info} 
          />
        </div>

        {/* Right Column: Venue & Host */}
        <div className="space-y-12">
          <div className="bg-orange-950 text-white p-8 rounded-3xl shadow-2xl relative overflow-hidden group">
            <div className="absolute top-0 right-0 -m-8 opacity-10 transition-transform group-hover:rotate-12 duration-500">
               <MapPin size={200} />
            </div>
            <MapPin className="mb-4 text-orange-400" size={32} />
            <h3 className="text-2xl font-bold mb-4">Venue</h3>
            <p className="text-xl font-medium mb-1">"Sukh Vatika Lawn"</p>
            <address className="not-italic text-orange-100 text-sm leading-relaxed mb-6">
              Chakiyawa Road, New Colony<br />
              Near Dr. Shamsul Azam Clinic<br />
              Deoria, Uttar Pradesh
            </address>
            <button 
              onClick={openMap}
              className="w-full bg-orange-600 hover:bg-orange-500 text-white py-3 rounded-xl font-bold transition-all flex items-center justify-center space-x-2 shadow-lg active:scale-95"
            >
              <span>View Location on Map</span>
              <ChevronRight size={18} />
            </button>
          </div>

          <div className="bg-white p-8 rounded-3xl border border-orange-100 shadow-xl">
             <h3 className="text-xl font-bold text-orange-950 mb-4">Hosting with Love</h3>
             <p className="text-gray-600 mb-4 leading-relaxed">
               <span className="font-bold text-gray-800">Smt. Anamika & Shri Ashok Kumar Srivastava</span><br />
               Request your presence to bless the couple.
             </p>
             <div className="flex items-center space-x-3 text-orange-600 font-bold bg-orange-50 p-4 rounded-2xl w-fit">
               <Phone size={20} />
               <span>+91 8004788630</span>
             </div>
          </div>
        </div>
      </main>

      {/* Footer */}
      <footer className="py-12 text-center text-gray-400 text-[10px] uppercase tracking-[0.4em]">
        Celebrating the Union of Niharika & Pankaj • 2026
      </footer>

      <style dangerouslySetInnerHTML={{ __html: `
        .custom-scrollbar::-webkit-scrollbar { width: 4px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: transparent; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: rgba(0,0,0,0.1); border-radius: 10px; }
      ` }} />
    </div>
  );
};

export default App;
