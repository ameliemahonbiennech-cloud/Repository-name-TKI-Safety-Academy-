TKI-Safety-Academy-
Interactive Laboratory Safety TrainingInteractive Laboratory Safety Training


Je commence par lire chaque passage signalé, pour distinguer les vrais problèmes des fausses alertes de l'outil d'analyse.
Action: read({"request":{"path":"/app/frontend/src/pages/ClassroomPage.tsx"}})
Observation: /app/frontend/src/pages/ClassroomPage.tsx:
1|import { useCallback, useEffect, useRef, useState } from "react";
2|import { CheckCircle2, ChevronRight, Eye, Maximize, Minimize, Minus, Pause, Play, Plus, Presentation, RotateCcw, Timer, X } from "lucide-react";
3|import { Button } from "@/components/ui/button";
4|import { COURSES, PASS_MARK, type Course, type Question } from "@/data/courses";
5|import { useI18n } from "@/lib/i18n";
6|import { cn } from "@/lib/utils";
7|import { MascotImage } from "@/components/Mascot";
8|
9|const LETTERS = ["A", "B", "C"];
10|const TIMER = 30;
11|
12|interface Item {
13|  course: Course;
14|  q: Question;
15|}
16|type Result = { votes: number[]; correct: number };
17|
18|export default function ClassroomPage() {
19|  const { t, tr } = useI18n();
20|  const [choice, setChoice] = useState<string>("all");
21|  const [items, setItems] = useState<Item[] | null>(null);
22|
23|  const start = () => {
24|    const list = COURSES.filter((c) => choice === "all" || c.id === choice).flatMap((c) => c.questions.map((q) => ({ course: c, q })));
25|    setItems(list);
26|    void document.documentElement.requestFullscreen?.().catch(() => undefined);
27|  };
28|
29|  if (items) return <Projection items={items} onExit={() => setItems(null)} />;
30|
31|  return (
32|    <div className="mx-auto max-w-4xl space-y-8">
33|      <header className="space-y-2">
34|        <p className="font-mono text-[11px] uppercase tracking-[0.18em] text-teal">{t("navClassroom")}</p>
35|        <h1 className="font-heading text-3xl font-bold sm:text-4xl">{t("classTitle")}</h1>
36|        <p className="text-mist">{t("classSub")}</p>
37|      </header>
38|      <div className="grid gap-3 sm:grid-cols-2">
39|        {[{ id: "all", label: t("classAll"), n: COURSES.reduce((a, c) => a + c.questions.length, 0) }, ...COURSES.map((c) => ({ id: c.id, label: `0${c.number} · ${tr(c.title)}`, n: c.questions.length }))].map((o) => (
40|          <button
41|            key={o.id}
42|            type="button"
43|            onClick={() => setChoice(o.id)}
44|            className={cn(
45|              "flex min-h-16 items-center justify-between rounded-2xl border px-5 py-4 text-left transition-colors duration-150",
46|              choice === o.id ? "border-teal bg-teal/10" : "border-white/10 bg-navy/60 hover:border-teal/40",
47|            )}
48|            aria-pressed={choice === o.id}
49|            data-testid={`classroom-choice-${o.id}`}
50|          >
51|            <span className="font-heading text-lg font-semibold">{o.label}</span>
52|            <span className="font-mono text-xs text-mist">
53|              {o.n} {t("questions")}
54|            </span>
55|          </button>
56|        ))}
57|      </div>
58|      <div className="rounded-2xl border border-white/10 bg-navy/50 p-5 text-sm text-mist">
59|        <p className="mb-2 font-mono text-[11px] uppercase tracking-[0.18em] text-teal">{t("classKeys")}</p>
60|        <p>{t("classKeysDesc")}</p>
61|      </div>
62|      <Button size="lg" onClick={start} className="h-12 gap-2 px-6 font-semibold" data-testid="classroom-start-button">
63|        <Presentation className="size-5" /> {t("classStart")}
64|      </Button>
65|    </div>
66|  );
67|}
68|
69|function Projection({ items, onExit }: { items: Item[]; onExit: () => void }) {
70|  const { t, tr } = useI18n();
71|  const [idx, setIdx] = useState(0);
72|  const [votes, setVotes] = useState<number[]>([0, 0, 0]);
73|  const [revealed, setRevealed] = useState(false);
74|  const [results, setResults] = useState<Result[]>([]);
75|  const [left, setLeft] = useState(TIMER);
76|  const [running, setRunning] = useState(true);
77|  const [fs, setFs] = useState(!!document.fullscreenElement);
78|  const done = idx >= items.length;
79|  const item = items[Math.min(idx, items.length - 1)];
80|  const total = votes.reduce((a, b) => a + b, 0);
81|  const timerRef = useRef<number | null>(null);
82|
83|  useEffect(() => {
84|    const onFs = () => setFs(!!document.fullscreenElement);
85|    document.addEventListener("fullscreenchange", onFs);
86|    return () => document.removeEventListener("fullscreenchange", onFs);
87|  }, []);
88|
89|  useEffect(() => {
90|    if (!running || revealed || done) return;
91|    timerRef.current = window.setInterval(() => setLeft((l) => (l <= 1 ? 0 : l - 1)), 1000);
92|    return () => {
93|      if (timerRef.current) window.clearInterval(timerRef.current);
94|    };
95|  }, [running, revealed, done, idx]);
96|
97|  const vote = useCallback((i: number, d: number) => {
98|    if (revealed) return;
99|    setVotes((v) => v.map((x, j) => (j === i ? Math.max(0, x + d) : x)));
100|  }, [revealed]);
101|
102|  const reveal = useCallback(() => {
103|    if (revealed || done) return;
104|    setRevealed(true);
105|    setResults((r) => [...r, { votes, correct: item.q.correct }]);
106|  }, [revealed, done, votes, item]);
107|
108|  const next = useCallback(() => {
109|    if (!revealed) return;
110|    setIdx((i) => i + 1);
111|    setVotes([0, 0, 0]);
112|    setRevealed(false);
113|    setLeft(TIMER);
114|    setRunning(true);
115|  }, [revealed]);
116|
117|  const exit = useCallback(() => {
118|    if (document.fullscreenElement) void document.exitFullscreen().catch(() => undefined);
119|    onExit();
120|  }, [onExit]);
121|
122|  const toggleFs = () => {
123|    if (document.fullscreenElement) void document.exitFullscreen();
124|    else void document.documentElement.requestFullscreen?.().catch(() => undefined);
125|  };
126|
127|  useEffect(() => {
128|    const onKey = (e: KeyboardEvent) => {
129|      const k = e.key.toLowerCase();
130|      const map: Record<string, number> = { a: 0, b: 1, c: 2, "1": 0, "2": 1, "3": 2 };
131|      if (k in map) vote(map[k], e.shiftKey ? -1 : 1);
132|      else if (k === "r" || k === " ") {
133|        e.preventDefault();
134|        if (revealed) next();
135|        else reveal();
136|      } else if (k === "arrowright" || k === "enter") next();
137|      else if (k === "f") toggleFs();
138|    };
139|    window.addEventListener("keydown", onKey);
140|    return () => window.removeEventListener("keydown", onKey);
141|  }, [vote, reveal, next, revealed]);
142|
143|  const roomScore = results.length ? Math.round((results.reduce((a, r) => a + (r.votes.reduce((x, y) => x + y, 0) ? r.votes[r.correct] / r.votes.reduce((x, y) => x + y, 0) : 0), 0) / results.length) * 100) : 0;
144|
145|  return (
146|    <div className="blueprint fixed inset-0 z-50 flex flex-col overflow-y-auto" data-testid="classroom-projection">
147|      <div className="flex items-center justify-between gap-3 border-b border-teal/15 bg-deep/80 px-6 py-3 backdrop-blur">
148|        <p className="font-heading text-lg font-bold tracking-[0.14em]">
149|          SAFETY <span className="text-teal">ACADEMY</span> <span className="ml-2 font-mono text-xs font-normal tracking-widest text-mist">· {t("navClassroom")}</span>
150|        </p>
151|        <div className="flex items-center gap-2">
152|          {!done && (
153|            <span className="font-mono text-sm text-mist" data-testid="classroom-progress">
154|              {idx + 1} / {items.length}
155|            </span>
156|          )}
157|          <Button variant="outline" size="icon" onClick={toggleFs} aria-label="fullscreen" data-testid="classroom-fullscreen-toggle">
158|            {fs ? <Minimize className="size-4" /> : <Maximize className="size-4" />}
159|          </Button>
160|          <Button variant="outline" size="icon" onClick={exit} aria-label="exit" data-testid="classroom-exit-button">
161|            <X className="size-4" />
162|          </Button>
163|        </div>
164|      </div>
165|
166|      {done ? (
167|        <div className="mx-auto flex w-full max-w-6xl flex-1 flex-col justify-center gap-8 p-8" data-testid="classroom-summary">
168|          <div className="grid items-center gap-8 md:grid-cols-[1fr_auto]">
169|            <div className="space-y-3">
170|              <h2 className="font-heading text-5xl font-bold">{t("classSummary")}</h2>
171|              <p className="text-xl text-mist">{t("classRoomScore")}</p>
172|              <p className={cn("font-heading text-7xl font-bold", roomScore >= PASS_MARK ? "text-ok" : "text-warn")} data-testid="classroom-room-score">
173|                {roomScore}%
174|              </p>
175|            </div>
176|            <div className="h-64 w-48 overflow-hidden rounded-3xl bg-ice">
177|              <MascotImage pose={roomScore >= PASS_MARK ? "celebrate" : "caution"} className="h-full w-full object-cover object-top" />
178|            </div>
179|          </div>
180|          <div className="grid gap-2 sm:grid-cols-2 lg:grid-cols-3">
181|            {results.map((r, i) => {
182|              const n = r.votes.reduce((a, b) => a + b, 0);
183|              const pct = n ? Math.round((r.votes[r.correct] / n) * 100) : 0;
184|              return (
185|                <div key={i} className="rounded-xl border border-white/10 bg-navy/70 p-3" data-testid={`classroom-summary-item-${i}`}>
186|                  <p className="line-clamp-2 text-sm">{tr(items[i].q.q)}</p>
187|                  <div className="mt-2 h-2 rounded-full bg-white/10">
188|                    <div className={cn("h-full rounded-full", pct >= PASS_MARK ? "bg-ok" : "bg-warn")} style={{ width: `${pct}%` }} />
189|                  </div>
190|                  <p className="mt-1 font-mono text-xs text-mist">
191|                    {pct}% · {n} {t("classVotes")}
192|                  </p>
193|                </div>
194|              );
195|            })}
196|          </div>
197|          <div className="flex gap-3">
198|            <Button size="lg" onClick={() => { setIdx(0); setResults([]); setVotes([0, 0, 0]); setRevealed(false); setLeft(TIMER); }} className="h-12 gap-2" data-testid="classroom-restart-button">
199|              <RotateCcw className="size-4" /> {t("classRestart")}
200|            </Button>
201|            <Button size="lg" variant="outline" onClick={exit} className="h-12">
202|              {t("back")}
203|            </Button>
204|          </div>
205|        </div>
206|      ) : (
207|        <div key={idx} className="mx-auto flex w-full max-w-7xl flex-1 flex-col gap-6 p-6 animate-fade-up lg:p-10">
208|          <div className="flex flex-wrap items-center justify-between gap-4">
209|            <p className="font-mono text-sm uppercase tracking-[0.2em]" style={{ color: item.course.accent }}>
210|              0{item.course.number} · {tr(item.course.title)}
211|            </p>
212|            <div className="flex items-center gap-3">
213|              <span
214|                className={cn("inline-flex items-center gap-2 rounded-full px-4 py-2 font-mono text-2xl font-bold tabular-nums", left <= 5 && !revealed ? "bg-danger/20 text-danger" : "bg-white/10")}
215|                data-testid="classroom-timer"
216|              >
217|                <Timer className="size-5" /> {left}s
218|              </span>
219|              <Button variant="outline" size="icon" onClick={() => setRunning((r) => !r)} aria-label="timer" data-testid="classroom-timer-toggle">
220|                {running ? <Pause className="size-4" /> : <Play className="size-4" />}
221|              </Button>
222|            </div>
223|          </div>
224|          <h2 className="font-heading text-3xl font-bold leading-tight sm:text-5xl" data-testid="classroom-question">
225|            {tr(item.q.q)}
226|          </h2>
227|          <div className="grid gap-4 lg:grid-cols-3">
228|            {item.q.options.map((o, i) => {
229|              const pct = total ? Math.round((votes[i] / total) * 100) : 0;
230|              const right = revealed && i === item.q.correct;
231|              const wrong = revealed && i !== item.q.correct;
232|              return (
233|                <div
234|                  key={i}
235|                  className={cn(
236|                    "relative flex flex-col gap-4 overflow-hidden rounded-3xl border-2 p-6 transition-[border-color,opacity] duration-300",
237|                    right ? "border-ok bg-ok/10" : wrong ? "border-white/10 bg-navy/60 opacity-60" : "border-teal/30 bg-navy/80",
238|                  )}
239|                  data-testid={`classroom-option-${i}`}
240|                >
241|                  {revealed && <div className={cn("absolute inset-y-0 left-0 transition-[width] duration-700", right ? "bg-ok/15" : "bg-white/5")} style={{ width: `${pct}%` }} />}
242|                  <div className="relative flex items-start gap-4">
243|                    <span className={cn("grid size-14 shrink-0 place-items-center rounded-2xl font-heading text-3xl font-bold", right ? "bg-ok text-deep" : "bg-teal text-deep")}>{LETTERS[i]}</span>
244|                    <p className="font-heading text-2xl font-semibold leading-snug">{tr(o)}</p>
245|                    {right && <CheckCircle2 className="ml-auto size-8 shrink-0 text-ok" />}
246|                  </div>
247|                  <div className="relative mt-auto flex items-center justify-between">
248|                    <div className="flex items-center gap-2">
249|                      <Button variant="outline" size="icon" onClick={() => vote(i, -1)} disabled={revealed} aria-label="minus" data-testid={`classroom-vote-minus-${i}`}>
250|                        <Minus className="size-4" />
251|                      </Button>
252|                      <span className="min-w-12 text-center font-heading text-4xl font-bold tabular-nums" data-testid={`classroom-vote-count-${i}`}>
253|                        {votes[i]}
254|                      </span>
255|                      <Button size="icon" onClick={() => vote(i, 1)} disabled={revealed} aria-label="plus" data-testid={`classroom-vote-plus-${i}`}>
256|                        <Plus className="size-4" />
257|                      </Button>
258|                    </div>
259|                    {revealed && <span className="font-heading text-3xl font-bold" data-testid={`classroom-vote-pct-${i}`}>{pct}%</span>}
260|                  </div>
261|                </div>
262|              );
263|            })}
264|          </div>
265|          {revealed && (
266|            <div className="flex animate-fade-up items-end gap-5 rounded-3xl border border-ok/30 bg-deep/70 p-6" data-testid="classroom-explanation">
267|              <div className="h-28 w-20 shrink-0 overflow-hidden rounded-2xl bg-ice">
268|                <MascotImage pose="celebrate" className="h-full w-full object-cover object-top" />
269|              </div>
270|              <div>
271|                <p className="font-heading text-2xl font-bold text-ok">
272|                  {LETTERS[item.q.correct]}. {tr(item.q.options[item.q.correct])}
273|                </p>
274|                <p className="mt-1 text-lg text-white/90">{tr(item.q.explanation)}</p>
275|              </div>
276|            </div>
277|          )}
278|          <div className="mt-auto flex flex-wrap items-center justify-between gap-3">
279|            <p className="font-mono text-sm text-mist" data-testid="classroom-total-votes">
280|              {total} {t("classVotes")}
281|            </p>
282|            {!revealed ? (
283|              <Button size="lg" onClick={reveal} className="h-14 gap-2 px-8 text-lg font-semibold" data-testid="classroom-reveal-button">
284|                <Eye className="size-5" /> {t("classReveal")}
285|              </Button>
286|            ) : (
287|              <Button size="lg" onClick={next} className="h-14 gap-2 px-8 text-lg font-semibold" data-testid="classroom-next-button">
288|                {idx + 1 < items.length ? t("nextQuestion") : t("seeResult")} <ChevronRight className="size-5" />
289|              </Button>
290|            )}
291|          </div>
292|        </div>
293|      )}
294|    </div>
295|  );
296|}
297|
