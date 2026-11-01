# Ochiq kodli AI vositalarini Claude ish oqimiga joriy qilish rejasi

**Sana:** 2026-11-01
**Manba:** Instagram karuseli (@delnov_mi) — 7 ta "killer" ochiq kodli loyiha
**Maqsad:** Ushbu vositalardan qaysilari Davlat Farmakopeyasi va ilmiy-tahririy ish
oqimiga real foyda berishini aniqlash va ularni Claude Code bilan bog'lash.

---

## 0. Qisqa xulosa (TL;DR)

| # | Loyiha | Nimaning o'rniga | Bizga mos? | Ustuvorlik |
|---|--------|------------------|-----------|-----------|
| 02 | **Presenton** | Gamma / Canva | **Ha — eng kuchli nomzod** | 🟢 1-bosqich |
| 03 | **meetily** | Otter.ai | **Ha — maxfiylik sababli qimmatli** | 🟢 2-bosqich |
| 01 | **HeyGem (Duix-Avatar)** | HeyGen | Shartli — faqat o'quv video uchun | 🟡 3-bosqich (pilot) |
| 05 | **Vane (ex-Perplexica)** | Perplexity | Ixtiyoriy — mavjud MCP'lar kuchliroq | 🟡 3-bosqich |
| 04 | **agenticSeek** | Manus | **Yo'q** — Claude Code buni allaqachon qiladi | 🔴 o'tkazib yuborish |
| 06 | **harper** | Grammarly | **Yo'q** — faqat ingliz tili | 🔴 o'tkazib yuborish |
| 07 | **opencode** | Cursor | **Yo'q** — Claude Code o'rnini bosuvchi, qo'shimcha emas | 🔴 o'tkazib yuborish |

**Halol baho:** karuselda ko'rsatilgan "oyiga $138 tejash" — marketing raqami.
7 tadan faqat 2 tasi bizning ishimizga aniq mos keladi; xarajat obunadan
apparat + elektr + sozlash vaqtiga ko'chadi. Lekin **maxfiylik** bo'yicha yutuq
real: nashrdan oldingi farmakopeya matnlari va ishchi guruh yozuvlari
hech qayerga yuklanmaydi.

---

## 1. Presenton — taqdimot generatori (1-ustuvorlik)

**Repo:** `presenton/presenton` · Apache-2.0 · 9.5k★

### Nega aynan bu mos
- REST API bor: `POST /api/v1/ppt/presentation/generate`
  (`content`, `slides_markdown`, `instructions`, `tone`, `n_slides`,
  `language`, `template`, `export_as: pptx|pdf`).
- **O'zining MCP serveri bor** — Claude Code'ga to'g'ridan-to'g'ri ulanadi.
- LLM provayderi sifatida **Anthropic (Claude)** qo'llab-quvvatlanadi →
  slayd matnini Claude yozadi, ya'ni terminologiya bizning nazoratimizda.
- Mavjud brendlangan `.pptx` fayldan **shablon yaratish** mumkin (vazirlik/
  Farmakopeya qolipi).
- `language` parametri bor — o'zbek/rus tilida chiqarish mumkin.

### Qayerda ishlatamiz
1. Xalqaro tadbirlar va konferensiya ma'ruzalari.
2. Mintaqaviy laboratoriyalar uchun GMP / farmakopeya o'quv slaydlari.
3. ICH Q2 validatsiya hisobotlarining taqdimot versiyasi
   (`analytical-validation-ich-q2` ko'nikmasi chiqishi asosida).
4. Ishchi guruh yakuniy hisobotlari.

### Joriy qilish qadamlari
```bash
# 1) Ishga tushirish (Docker)
docker run -d --name presenton -p 5001:80 \
  -e PRESENTATION_GENERATION_MODE=both \
  -e ANTHROPIC_API_KEY=... \
  -v "./app_data:/app_data" \
  ghcr.io/presenton/presenton:latest
```
2. Brend shabloni: mavjud rasmiy `.pptx` ni yuklab, shablon sifatida saqlash.
3. Claude Code'ga MCP sifatida ulash (`.mcp.json`).
4. Yangi ko'nikma (skill) yozish: **`monograph-to-deck`**
   - kirish: tayyor monografiya `.docx` yoki validatsiya hisoboti;
   - Claude avval `slides_markdown` ni **o'zi tayyorlaydi** (termbase-manager
     va agreed-terms-checker qoidalari bo'yicha), keyin API'ga uzatadi;
   - chiqish: brendlangan `.pptx` + `.pdf`.

> **Muhim:** Presenton'ga "mavzu berib, o'zi yozsin" demaymiz. Matnni Claude
> bizning terminologiya bazamiz asosida yozadi, Presenton faqat dizayn va
> eksport qiladi. Aks holda kelishilgan atamalar buziladi.

### Muvaffaqiyat mezoni (2 hafta)
Bitta real monografiya → 12–15 slaydli brendlangan `.pptx`, terminologiya
tekshiruvidan 0 ta xato bilan o'tadi.

---

## 2. meetily — majlis yozuvi va bayonnoma (2-ustuvorlik)

**Repo:** `Zackriya-Solutions/meetily` · 29k★

### Nega mos
- 100% lokal: Whisper yoki Parakeet transkripsiya + Ollama xulosalash.
- Xulosalovchi model sifatida **Claude'ni ham ulash mumkin**.
- Ovoz hech qachon noutbukdan chiqmaydi — bu nashrdan oldingi farmakopeya
  muhokamalari uchun **majburiy shart**.
- Lokal koordinator API va veb-interfeysi bor (dev rejimida `localhost:3118`).

### Qayerda ishlatamiz
- Farmakopeya qo'mitasi va ishchi guruh majlislari;
- EAEU / JSST yo'nalishidagi onlayn uchrashuvlar;
- ekspert muhokamalari → rasmiy bayonnoma loyihasi.

### Joriy qilish qadamlari
1. O'rnatish + model tanlash (`whisper large-v3` yoki Parakeet).
2. **Pilot — avval rus va ingliz tilidagi majlislarda.**
3. Yangi ko'nikma: **`meeting-to-protocol`**
   - meetily transkripti + xulosasini oladi;
   - `manual-rewriter` uslubida uchinchi shaxs, idoraviy registrga o'tkazadi;
   - "Qaror / Topshiriq / Mas'ul / Muddat" jadvalini shakllantiradi;
   - `termbase-manager` orqali atamalarni kanonik shaklga keltiradi.

### Ochiq risk
Whisper **o'zbek tilida zaif**, ayniqsa o'zbekcha-ruscha aralash nutqda.
Shuning uchun: pilotda WER (xato foizi) o'lchanadi; o'zbek tilidagi majlislar
uchun transkript Claude tomonidan termbase asosida post-tahrir qilinadi.
Agar WER 25% dan yuqori bo'lsa — o'zbek tilida faqat qo'llab-quvvatlovchi
vosita sifatida ishlatiladi, asosiy manba sifatida emas.

---

## 3. HeyGem (Duix-Avatar) — pilot, shartli (3-bosqich)

**Repo:** `duixcom/Duix-Avatar` · 14.4k★

### Apparat talablari (jiddiy)
- NVIDIA **RTX 4070** darajasidagi videokarta;
- **32 GB** operativ xotira;
- 30 GB dan ortiq bo'sh disk; Windows yoki Ubuntu 22.04 + Docker + NVIDIA
  Container Toolkit.

Ya'ni bu mavjud noutbukda ishlamaydi — alohida ish stansiyasi kerak.

### Qayerda ma'noli bo'lishi mumkin
- `Instruktor.ai` ta'lim kanali uchun videolar (mascot/brending ko'nikmasi
  allaqachon mavjud);
- yangi farmakopeya maqolalari bo'yicha qisqa tushuntirish roliklari;
- mintaqaviy laboratoriyalar uchun masofaviy o'quv moduli.

### Cheklovlar va shartlar
1. **Ovoz — asosiy to'siq.** Lip-sync audio bo'yicha ishlaydi, lekin o'zbek
   tilidagi sifatli TTS/ovoz klonlash alohida masala. Birinchi bosqichda
   **o'z ovozingizni yozib**, HeyGem faqat lab harakatini moslashi kerak.
2. **Oshkoralik majburiy.** Davlat mansabdor shaxsining sintetik avatari
   oshkor qilinmasdan ishlatilmasligi lozim — har bir videoda
   "AI-avatar" belgisi.
3. Rasmiy hujjat, bayonot yoki normativ xabar uchun **ishlatilmaydi** —
   faqat ta'limiy kontent.

**Tavsiya:** 3-bosqichda, 1 va 2-bosqich natija bergandan keyin, 1 ta sinov
roliki bilan baholash.

---

## 4. Vane (ex-Perplexica) — ixtiyoriy

**Repo:** `ItzCrazyKns/Vane` · MIT · 36.1k★ · Docker + ichki SearxNG.

Bizda allaqachon kuchliroq qidiruv zanjiri bor: PubMed, Consensus,
Firecrawl, WebSearch MCP'lari. Vane ochiq internetni qidiradi — Ph. Eur.,
USP, EAEU normativ bazasi uchun aniqlik bermaydi.

**Yagona real foyda:** so'rovlar tashqariga chiqmaydi (mavzu maxfiyligi).
Agar bu talab qo'yilsa — 3-bosqichda ko'rib chiqiladi, aks holda kerak emas.

---

## 5. Nega 3 tasi rad etiladi

| Loyiha | Sabab |
|--------|-------|
| **agenticSeek** | Claude Code allaqachon brauzer, fayl va tadqiqot agenti. Lokal modellar bilan sifat pastroq, GPU talab qiladi. Takrorlanish. |
| **harper** | **Faqat ingliz tili** (rasman tasdiqlangan). Bizning matnlar — o'zbek kirill + rus. Mavjud `multilingual-proofreader` plagini bu vazifani allaqachon bajaradi. Maksimal foyda: maqola/patent inglizcha annotatsiyasini oldindan filtrlash. |
| **opencode** | Claude Code'ning muqobili, qo'shimchasi emas. Mavjud 13+ plagin va ko'nikma ekotizimi Claude Code'da qurilgan — ko'chirish zarar. |

---

## 6. Bosqichma-bosqich jadval

| Bosqich | Muddat | Ish | Natija |
|---------|--------|-----|--------|
| **0** | 1 hafta | Apparat inventarizatsiyasi, API kalitlari, maxfiylik siyosati | Qaror varaqasi |
| **1** | 2 hafta | Presenton: Docker + brend shabloni + MCP + `monograph-to-deck` ko'nikmasi | 1 ta real taqdimot |
| **2** | 2–3 hafta | meetily: o'rnatish + rus/ingliz pilot + `meeting-to-protocol` ko'nikmasi | 1 ta real bayonnoma loyihasi |
| **3** | shartli | HeyGem pilot (GPU bo'lsa) / Vane (maxfiy qidiruv talab qilinsa) | Baholash hisoboti |

---

## 7. Yaratiladigan artefaktlar

1. `.mcp.json` — Presenton MCP yozuvi.
2. Yangi plagin: **`deck-builder`** → ko'nikma `monograph-to-deck`.
3. Yangi plagin: **`meeting-minutes`** → ko'nikma `meeting-to-protocol`.
4. `docs/ai-stack-plan.md` (ushbu hujjat).
5. Maxfiylik qoidasi: qaysi matnlar lokal, qaysilari bulutli ishlanadi.

---

## 8. Keyingi qadam

Agar ma'qul bo'lsa, 1-bosqichdan boshlaymiz: Presenton MCP konfiguratsiyasi va
`monograph-to-deck` ko'nikmasining birinchi versiyasini yozib beraman.
Buning uchun kerak:
- taqdimotlar uchun rasmiy brendlangan `.pptx` namunasi;
- Presenton qayerda ishlashi (lokal noutbuk / server) haqida qaror.
