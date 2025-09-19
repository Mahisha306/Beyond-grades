📂 beyond-grades-learning-platform/
│
├── package.json              // root scripts to run frontend + backend
├── README.md
│
├── frontend/                 // React app
│   ├── package.json
│   ├── vite.config.js
│   ├── index.html
│   └── src/
│       ├── main.jsx
│       ├── App.jsx
│       └── components/
│           ├── ChatBot.jsx
│           └── Dashboard.jsx
│
├── backend/                  // Express API
│   ├── package.json
│   └── server.js
│
└── docs/
    ├── Beyond_Grades_Submission.pdf
    └── Beyond_Grades_Submission_Visual.pdf

// ===== package.json (root) =====
{
  "name": "beyond-grades-learning-platform",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "start": "npm run dev --workspace frontend & npm run start --workspace backend",
    "install-all": "npm install --workspaces"
  },
  "workspaces": ["frontend", "backend"]
}

// ===== backend/package.json =====
{
  "name": "backend",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "express": "^4.18.2"
  }
}

// ===== backend/server.js =====
const express = require("express");
const cors = require("cors");
const app = express();
app.use(cors());
app.use(express.json());

// Mock AI mentor endpoint
app.post("/api/mentor", (req, res) => {
  const { question } = req.body;
  res.json({
    answer: `Good question! Let's think about: ${question}. Why do you think it happens this way?`
  });
});

// Mock dashboard data
app.get("/api/dashboard", (req, res) => {
  res.json({
    skills: [
      { name: "Critical Thinking", level: 80 },
      { name: "Problem Solving", level: 70 },
      { name: "Creativity", level: 65 },
      { name: "Collaboration", level: 75 }
    ]
  });
});

app.listen(5000, () => console.log("Backend running on http://localhost:5000"));

// ===== frontend/src/main.jsx =====
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

// ===== frontend/src/App.jsx =====
import React, { useState } from 'react';
import ChatBot from './components/ChatBot';
import Dashboard from './components/Dashboard';

export default function App() {
  const [view, setView] = useState('chat');
  return (
    <div className="p-6 font-sans">
      <h1 className="text-2xl font-bold mb-4">🚀 Beyond Grades</h1>
      <div className="space-x-4 mb-6">
        <button onClick={() => setView('chat')} className="px-4 py-2 bg-blue-500 text-white rounded">AI Mentor</button>
        <button onClick={() => setView('dashboard')} className="px-4 py-2 bg-green-500 text-white rounded">Dashboard</button>
      </div>
      {view === 'chat' ? <ChatBot /> : <Dashboard />}
    </div>
  );
}

// ===== frontend/src/components/ChatBot.jsx =====
import React, { useState } from 'react';

export default function ChatBot() {
  const [input, setInput] = useState('');
  const [messages, setMessages] = useState([]);

  const sendMessage = async () => {
    const res = await fetch('http://localhost:5000/api/mentor', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ question: input })
    });
    const data = await res.json();
    setMessages([...messages, { user: input, bot: data.answer }]);
    setInput('');
  };

  return (
    <div>
      <h2 className="text-xl mb-2">💬 AI Mentor</h2>
      <div className="border p-2 h-60 overflow-y-auto mb-2">
        {messages.map((m, i) => (
          <div key={i} className="mb-2">
            <p><strong>You:</strong> {m.user}</p>
            <p><strong>AI:</strong> {m.bot}</p>
          </div>
        ))}
      </div>
      <input value={input} onChange={(e) => setInput(e.target.value)} className="border px-2 py-1 mr-2" />
      <button onClick={sendMessage} className="bg-blue-500 text-white px-3 py-1 rounded">Send</button>
    </div>
  );
}

// ===== frontend/src/components/Dashboard.jsx =====
import React, { useEffect, useState } from 'react';

export default function Dashboard() {
  const [skills, setSkills] = useState([]);

  useEffect(() => {
    fetch('http://localhost:5000/api/dashboard')
      .then(res => res.json())
      .then(data => setSkills(data.skills));
  }, []);

  return (
    <div>
      <h2 className="text-xl mb-2">📊 Educator Dashboard</h2>
      <ul>
        {skills.map((s, i) => (
          <li key={i} className="mb-2">{s.name}: <strong>{s.level}%</strong></li>
        ))}
      </ul>
    </div>
  );
}
