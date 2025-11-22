import React, { useState, useEffect, useRef } from "react";   
// EnglishExchangeApp.jsx // Single-file React component (TailwindCSS). Default export at bottom. // Notes for integration (in comments): // - Replace mock state with real backend (WebSocket / Firebase / Supabase) for realtime chat // - Add OAuth (Google / Facebook / Apple) for authentication // - Use WebRTC for audio/video calls or integrate Jitsi/Agora // - Use translation APIs for on-the-fly translation if desired

function sampleTopics() { return [ "Travel", "Culture", "News", "Tech", "Everyday Conversation", "Interview Practice", ]; }

function mockRooms() { return [ { id: "r1", name: "Travel Talk", topic: "Travel", lang: "English", users: 6 }, { id: "r2", name: "Tech News", topic: "Tech", lang: "English", users: 4 }, { id: "r3", name: "Casual Chat", topic: "Everyday Conversation", lang: "English", users: 10 }, ]; }

function Avatar({ name, className = "w-10 h-10 rounded-full bg-gray-200 flex items-center justify-center text-sm" }) { const initials = name .split(" ") .map((p) => p[0]) .slice(0, 2) .join(""); return <div className={${className}}>{initials}</div>; }

function RoomCard({ room, onJoin }) { return ( <div className="p-4 bg-white rounded-2xl shadow-sm border flex flex-col gap-2"> <div className="flex items-center justify-between"> <div> <h3 className="font-semibold">{room.name}</h3> <p className="text-xs text-gray-500">{room.topic} • {room.lang}</p> </div> <div className="text-sm text-gray-600">{room.users} online</div> </div> <div className="flex justify-end"> <button onClick={() => onJoin(room)} className="px-3 py-1 rounded-lg bg-indigo-600 text-white text-sm">Join</button> </div> </div> ); }

function ChatMessage({ msg, me }) { return ( <div className={flex ${me ? 'justify-end' : 'justify-start'}}> <div className={${me ? 'bg-indigo-600 text-white' : 'bg-gray-100 text-gray-900'} rounded-xl p-3 max-w-xs}> <div className="text-xs text-gray-500 mb-1">{!me && <strong className="mr-1">{msg.user}</strong>}</div> <div className="whitespace-pre-line">{msg.text}</div> <div className="text-[11px] text-gray-400 mt-1 text-right">{msg.time}</div> </div> </div> ); }

export default function EnglishExchangeApp() { const [loggedIn, setLoggedIn] = useState(false); const [email, setEmail] = useState(''); const [password, setPassword] = useState(''); const [user, setUser] = useState({ id: 'u_me', name: 'أنت' }); const [topics] = useState(sampleTopics()); const [rooms, setRooms] = useState(mockRooms()); const [selectedTopic, setSelectedTopic] = useState('All'); const [currentRoom, setCurrentRoom] = useState(null); const [messages, setMessages] = useState([]); const [input, setInput] = useState(''); const messagesEndRef = useRef(null);

if (!loggedIn) { return ( <div className="min-h-screen flex items-center justify-center bg-slate-50 p-4"> <div className="bg-white p-6 rounded-2xl shadow-md w-full max-w-sm"> <h2 className="text-xl font-bold text-center mb-4">تسجيل الدخول</h2> <input value={email} onChange={(e) => setEmail(e.target.value)} placeholder="البريد الإلكتروني" className="w-full px-3 py-2 rounded-lg border mb-3" /> <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="كلمة المرور" className="w-full px-3 py-2 rounded-lg border mb-3" /> <button onClick={() => setLoggedIn(true)} className="w-full py-2 bg-indigo-600 text-white rounded-lg" > دخول </button> <button className="w-full py-2 mt-2 bg-gray-100 rounded-lg">تسجيل جديد</button> </div> </div> ); }

return; const now = new Date(); const time = now.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }); const msg = { id: Date.now(), user: user.name, text: input.trim(), time }; // In production: emit to server setMessages((s) => [...s, msg]); setInput(''); }

// lightweight UI only; mobile-first responsive return ( <div className="min-h-screen bg-slate-50 p-4 md:p-10 font-sans"> <div className="max-w-6xl mx-auto grid grid-cols-1 md:grid-cols-3 gap-6"> {/* Left: Rooms / Create */} <aside className="col-span-1"> <div className="sticky top-6 p-4 bg-white rounded-2xl shadow-sm"> <div className="flex items-center gap-3"> <Avatar name={user.name} /> <div> <div className="text-sm font-semibold">{user.name}</div> <div className="text-xs text-gray-500">Practice English • Learn about the world</div> </div> </div>

<div className="mt-4">
          <label className="text-xs text-gray-500">Filter by topic</label>
          <div className="flex flex-wrap gap-2 mt-2">
            <button onClick={() => setSelectedTopic('All')} className={`px-2 py-1 rounded-md text-sm ${selectedTopic === 'All' ? 'bg-indigo-600 text-white' : 'bg-gray-100'}`}>All</button>
            {topics.map((t) => (
              <button key={t} onClick={() => setSelectedTopic(t)} className={`px-2 py-1 rounded-md text-sm ${selectedTopic === t ? 'bg-indigo-600 text-white' : 'bg-gray-100'}`}>{t}</button>
            ))}
          </div>
        </div>

        <div className="mt-4">
          <h4 className="text-sm font-semibold mb-2">Rooms</h4>
          <div className="flex flex-col gap-3">
            {rooms.filter(r => selectedTopic === 'All' ? true : r.topic === selectedTopic).map(r => (
              <RoomCard key={r.id} room={r} onJoin={joinRoom} />
            ))}
          </div>
        </div>

        <CreateRoomForm onCreate={createRoom} />
      </div>
    </aside>

    {/* Middle: Chat Area */}
    <main className="md:col-span-2 bg-white p-4 rounded-2xl shadow-sm flex flex-col min-h-[60vh]">
      {!currentRoom ? (
        <div className="flex-1 flex flex-col items-center justify-center text-center p-8">
          <h2 className="text-2xl font-bold">انضم لغرفة أو أنشئ واحدة للتحدث بالإنجليزية</h2>
          <p className="text-gray-500 mt-3">اختر موضوعًا وتعرّف على متحدثين أصليين أو متعلمين آخرين من حول العالم.</p>
          <div className="mt-6">
            <button onClick={() => setSelectedTopic('Travel')} className="px-4 py-2 rounded-lg bg-indigo-600 text-white">جرب غرفة السفر</button>
          </div>
        </div>
      ) : (
        <div className="flex flex-col h-full">
          <div className="flex items-center justify-between border-b pb-3">
            <div>
              <h3 className="font-semibold">{currentRoom.name}</h3>
              <div className="text-xs text-gray-500">{currentRoom.topic} • {currentRoom.lang}</div>
            </div>
            <div className="flex items-center gap-3">
              <button className="text-sm px-3 py-1 bg-gray-100 rounded-md">Report</button>
              <button className="text-sm px-3 py-1 bg-green-100 rounded-md">Start Voice</button>
            </div>
          </div>

          <div className="flex-1 overflow-auto py-4 space-y-3">
            {messages.length === 0 && (
              <div className="text-center text-gray-400">لا توجد رسائل بعد — قل مرحبًا 👋</div>
            )}
            {messages.map(m => (
              <ChatMessage key={m.id} msg={m} me={m.user === user.name} />
            ))}
            <div ref={messagesEndRef} />
          </div>

          <div className="mt-3">
            <div className="flex gap-2">
              <input value={input} onChange={(e) => setInput(e.target.value)} onKeyDown={(e)=>{ if(e.key==='Enter') sendMessage(); }} placeholder="اكتب رسالة..." className="flex-1 px-4 py-2 rounded-xl border" />
              <button onClick={sendMessage} className="px-4 py-2 rounded-xl bg-indigo-600 text-white">Send</button>
            </div>
            <div className="text-xs text-gray-400 mt-2">نصيحة: اذكر من أين أنت وموضوع تريد التحدث عنه.</div>
          </div>
        </div>
      )}
    </main>
  </div>

  <footer className="max-w-6xl mx-auto mt-6 text-center text-sm text-gray-500">© {new Date().getFullYear()} English Exchange — تعلم وتحدث مع العالم</footer>
</div>

); }

function CreateRoomForm({ onCreate }) { const [name, setName] = useState(''); const [topic, setTopic] = useState('Travel');

function submit(e) { e.preventDefault(); if (!name.trim()) return; onCreate(name.trim(), topic); setName(''); }

return ( <form onSubmit={submit} className="mt-4"> <h4 className="text-sm font-semibold mb-2">أنشئ غرفة جديدة</h4> <input value={name} onChange={(e) => setName(e.target.value)} placeholder="اسم الغرفة (مثال: Travel Buddies)" className="w-full px-3 py-2 rounded-lg border" /> <div className="flex items-center gap-2 mt-2"> <select value={topic} onChange={(e) => setTopic(e.target.value)} className="px-3 py-2 rounded-lg border"> <option>Travel</option> <option>Culture</option> <option>News</option> <option>Tech</option> <option>Everyday Conversation</option> <option>Interview Practice</option> </select> <button type="submit" className="px-3 py-2 bg-indigo-600 text-white rounded-lg">Create</button> </div> </form> ); }
