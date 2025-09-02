import React, { useMemo, useState, useEffect } from "react";

// Rao Sahab 
// This MVP helps people discover legitimate, zero‑investment earning opportunities (freelance gigs,
// micro‑tasks, tutoring, referrals to free programs). It DOES NOT guarantee income. Avoid deceptive
// claims. Replace placeholder links with vetted ones before launching.

// ---- Mock data (replace with your vetted links) ----
const CATEGORIES = [
  {
    id: "freelance",
    title: "Freelance & Skills",
    desc: "Get paid for writing, design, video editing, voiceover, data entry, coding, etc.",
    items: [
      { name: "Beginner Writing Gigs", estPay: "₹400–₹1,200/task", time: "1–3 hrs", link: "#" },
      { name: "Logo/Thumbnail Design", estPay: "₹600–₹2,000/task", time: "2–4 hrs", link: "#" },
      { name: "Short Video Editing", estPay: "₹800–₹3,000/video", time: "3–6 hrs", link: "#" },
      { name: "No‑code Website Setup", estPay: "₹1,000–₹4,000/site", time: "1–2 days", link: "#" },
    ],
  },
  {
    id: "microtasks",
    title: "Micro‑tasks & Testing",
    desc: "Usability tests, transcription snippets, data labeling, bug bounties.",
    items: [
      { name: "Usability Testing (Beginner)", estPay: "₹600–₹1,200/test", time: "15–30 min", link: "#" },
      { name: "Transcription Small Jobs", estPay: "₹200–₹800/task", time: "30–60 min", link: "#" },
      { name: "Data Labeling (Simple)", estPay: "₹250–₹700/hr", time: "Flexible", link: "#" },
      { name: "Bug Bounty (Starter)", estPay: "₹1,000+ /find", time: "Varies", link: "#" },
    ],
  },
  {
    id: "tutoring",
    title: "Tutoring & Teaching",
    desc: "Teach English or school subjects online with free tools.",
    items: [
      { name: "1:1 Spoken English (Zoom)", estPay: "₹300–₹800/30 min", time: "30–60 min", link: "#" },
      { name: "Group Class (4–6 learners)", estPay: "₹1,200–₹3,000/hr", time: "60 min", link: "#" },
      { name: "Notes & Worksheets Store", estPay: "₹100–₹300/download", time: "Build once", link: "#" },
    ],
  },
  {
    id: "content",
    title: "Content & Assets",
    desc: "Create templates, captions, reels scripts, thumbnail packs.",
    items: [
      { name: "50 Reel Hooks Pack", estPay: "₹299–₹799/pack", time: "2–4 hrs", link: "#" },
      { name: "YouTube Thumbnail Pack", estPay: "₹399–₹1,299/pack", time: "3–6 hrs", link: "#" },
      { name: "Caption & Hashtag Sheets", estPay: "₹199–₹599/pack", time: "2–3 hrs", link: "#" },
    ],
  },
];

const TIPS = [
  "Never pay to get a job. If a website asks for deposits/registration fees, skip it.",
  "Use a fresh email for signups and enable two‑factor authentication.",
  "Collect sample work: 3 writing pieces, 2 thumbnails, 1 edited short — show in a simple portfolio.",
  "Start with smaller tasks to build ratings, then raise your rates week by week.",
  "Track your time. If a task pays too low per hour, replace it with a better one.",
];

// ---- Local storage helpers ----
const LS_KEY = "earnhub_progress_v1";

function useLocalStore() {
  const [data, setData] = useState(() => {
    try {
      const raw = localStorage.getItem(LS_KEY);
      return raw ? JSON.parse(raw) : { earnings: 0, tasks: [] };
    } catch (e) {
      return { earnings: 0, tasks: [] };
    }
  });
  useEffect(() => {
    localStorage.setItem(LS_KEY, JSON.stringify(data));
  }, [data]);
  return [data, setData];
}

function classNames(...s) { return s.filter(Boolean).join(" "); }

export default function EarnHubApp() {
  const [tab, setTab] = useState("freelance");
  const [data, setData] = useLocalStore();
  const [dailyTarget, setDailyTarget] = useState(5000);
  const [taskName, setTaskName] = useState("");
  const [taskAmount, setTaskAmount] = useState("");

  const progress = useMemo(() => {
    const doneTodayAmt = data.tasks.filter(t => isToday(t.date)).reduce((s, t) => s + t.amount, 0);
    const pct = Math.min(100, Math.round((doneTodayAmt / dailyTarget) * 100));
    return { doneTodayAmt, pct };
  }, [data.tasks, dailyTarget]);

  function addTask() {
    const amt = Number(taskAmount);
    if (!taskName || !amt || amt <= 0) return;
    const t = { id: crypto.randomUUID(), name: taskName, amount: amt, date: new Date().toISOString() };
    setData(d => ({ ...d, earnings: d.earnings + amt, tasks: [t, ...d.tasks] }));
    setTaskName("");
    setTaskAmount("");
  }

  function removeTask(id) {
    setData(d => {
      const t = d.tasks.find(x => x.id === id);
      const rest = d.tasks.filter(x => x.id !== id);
      return { ...d, earnings: t ? d.earnings - t.amount : d.earnings, tasks: rest };
    });
  }

  return (
    <div className="min-h-screen bg-gray-50 text-gray-900">
      <header className="sticky top-0 z-20 backdrop-blur bg-white/80 border-b">
        <div className="max-w-6xl mx-auto p-4 flex items-center justify-between">
          <div className="flex items-center gap-3">
            <div className="w-9 h-9 rounded-2xl bg-black text-white grid place-items-center font-bold">EH</div>
            <div>
              <h1 className="font-extrabold text-xl">EarnHub</h1>
              <p className="text-xs text-gray-500">Zero‑investment opportunities • India‑friendly</p>
            </div>
          </div>
          <a href="#safety" className="text-sm underline">Safety & Disclaimer</a>
        </div>
      </header>

      <main className="max-w-6xl mx-auto p-4 grid md:grid-cols-[1fr_340px] gap-6">
        {/* LEFT: Opportunities */}
        <section>
          <div className="mb-4 flex flex-wrap gap-2">
            {CATEGORIES.map(c => (
              <button
                key={c.id}
                onClick={() => setTab(c.id)}
                className={classNames(
                  "px-3 py-2 rounded-2xl text-sm border shadow-sm",
                  tab === c.id ? "bg-gray-900 text-white" : "bg-white hover:bg-gray-100"
                )}
              >
                {c.title}
              </button>
            ))}
          </div>

          {CATEGORIES.map(c => (
            <div key={c.id} className={tab === c.id ? "block" : "hidden"}>
              <div className="bg-white rounded-2xl p-4 shadow-sm border">
                <h2 className="font-bold text-lg">{c.title}</h2>
                <p className="text-sm text-gray-600 mb-4">{c.desc}</p>
                <ul className="grid sm:grid-cols-2 gap-3">
                  {c.items.map((it, i) => (
                    <li key={i} className="border rounded-2xl p-3 bg-gray-50">
                      <div className="flex items-start justify-between gap-2">
                        <div>
                          <h3 className="font-semibold">{it.name}</h3>
                          <p className="text-xs text-gray-600">Est. pay: {it.estPay} • Time: {it.time}</p>
                        </div>
                        <a
                          className="text-sm px-3 py-1 rounded-full border bg-white hover:bg-gray-100"
                          href={it.link}
                          target="_blank"
                          rel="noreferrer"
                        >Explore</a>
                      </div>
                    </li>
                  ))}
                </ul>
              </div>
            </div>
          ))}

          {/* Tips */}
          <div id="safety" className="mt-6 bg-white rounded-2xl p-4 shadow-sm border">
            <h2 className="font-bold text-lg">Safety, Ethics & Legal</h2>
            <ul className="list-disc pl-5 text-sm text-gray-700 space-y-2 mt-2">
              {TIPS.map((t, i) => (
                <li key={i}>{t}</li>
              ))}
              <li>
                <b>Important:</b> This site lists opportunities and tools only. We do not guarantee any
                income or outcome. Your earnings depend on your skill, time, demand, and marketplace rules.
              </li>
            </ul>
          </div>
        </section>

        {/* RIGHT: Tracker & Calculator */}
        <aside className="space-y-6">
          <div className="bg-white rounded-2xl p-4 shadow-sm border">
            <h3 className="font-bold">Daily Target</h3>
            <p className="text-sm text-gray-600 mb-2">Plan your way to a realistic target (e.g., ₹5,000/day)</p>
            <div className="flex items-center gap-2 mb-3">
              <span className="text-sm">₹</span>
              <input
                type="number"
                value={dailyTarget}
                onChange={(e) => setDailyTarget(Math.max(0, Number(e.target.value)))}
                className="w-full px-3 py-2 rounded-xl border bg-gray-50"
                min={0}
              />
            </div>
            <div className="w-full bg-gray-100 rounded-xl h-3 overflow-hidden">
              <div
                className="bg-black h-3"
                style={{ width: `${progress.pct}%` }}
                aria-label="progress"
              />
            </div>
            <p className="text-xs text-gray-600 mt-2">Today: ₹{progress.doneTodayAmt.toLocaleString()} / ₹{dailyTarget.toLocaleString()} ({progress.pct}%)</p>
          </div>

          <div className="bg-white rounded-2xl p-4 shadow-sm border">
            <h3 className="font-bold">Add Completed Task</h3>
            <div className="grid grid-cols-1 gap-2 mt-2">
              <input
                className="px-3 py-2 rounded-xl border bg-gray-50"
                placeholder="Task name (e.g., 2 captions pack)"
                value={taskName}
                onChange={(e) => setTaskName(e.target.value)}
              />
              <input
                type="number"
                className="px-3 py-2 rounded-xl border bg-gray-50"
                placeholder="Amount earned (₹)"
                value={taskAmount}
                onChange={(e) => setTaskAmount(e.target.value)}
                min={0}
              />
              <button onClick={addTask} className="px-3 py-2 rounded-xl bg-black text-white hover:opacity-90">Add</button>
            </div>
          </div>

          <div className="bg-white rounded-2xl p-4 shadow-sm border">
            <h3 className="font-bold">Today's Tasks</h3>
            {data.tasks.filter(t => isToday(t.date)).length === 0 ? (
              <p className="text-sm text-gray-600">No tasks added yet.</p>
            ) : (
              <ul className="space-y-2 mt-2">
                {data.tasks.filter(t => isToday(t.date)).map(t => (
                  <li key={t.id} className="flex items-center justify-between border rounded-xl p-2">
                    <div>
                      <p className="text-sm font-medium">{t.name}</p>
                      <p className="text-xs text-gray-600">₹{t.amount.toLocaleString()}</p>
                    </div>
                    <button onClick={() => removeTask(t.id)} className="text-xs px-2 py-1 rounded-lg border hover:bg-gray-50">Remove</button>
                  </li>
                ))}
              </ul>
            )}
          </div>

          <div className="bg-white rounded-2xl p-4 shadow-sm border">
            <h3 className="font-bold">Portfolio Starters (Free)</h3>
            <ul className="list-disc pl-5 text-sm text-gray-700 space-y-1 mt-2">
              <li>Make 3 sample designs (thumbnails, logos) and host images on Google Drive (public link).</li>
              <li>Write 3 short articles (300–500 words) on trending topics. Export to PDF and link.</li>
              <li>Record a 30‑sec voiceover and a 15‑sec video edit as proof of skill.</li>
            </ul>
          </div>

          <div className="bg-white rounded-2xl p-4 shadow-sm border">
            <h3 className="font-bold">No‑Investment Tools</h3>
            <ul className="text-sm text-gray-700 space-y-1 mt-2">
              <li>Docs/Sheets for proposals & invoices</li>
              <li>Drive for portfolio hosting</li>
              <li>Meet/Zoom for classes & calls</li>
              <li>UPI for payments</li>
            </ul>
          </div>
        </aside>
      </main>

      <footer className="max-w-6xl mx-auto p-6 text-xs text-gray-500">
        <p>
          © {new Date().getFullYear()} EarnHub. This website is an informational platform. It does not
          sell jobs, charge fees, or guarantee earnings. Replace placeholder links with your own
          vetted listings before launching.
        </p>
      </footer>
    </div>
  );
}

function isToday(isoDate) {
  const d = new Date(isoDate);
  const now = new Date();
  return d.getDate() === now.getDate() && d.getMonth() === now.getMonth() && d.getFullYear() === now.getFullYear();
}
