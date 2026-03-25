# Architecture — Patent OA Analyzer

## סקירה כללית

הפלטפורמה בנויה כ-**single HTML file** שרץ בדפדפן ללא שרת.
כל הלוגיקה נמצאת בצד הלקוח — קריאת קבצים, שליחה ל-API, הצגת תוצאות.

---

## שכבות המערכת

### שכבה 1 — UI (HTML/CSS/JS)
- קובץ יחיד: `platform/index.html`
- ללא framework — vanilla JS בלבד
- עיצוב: CSS variables, dark theme, RTL

### שכבה 2 — הסקיל (Skill Layer)
- מוגדר ב: `skill/SKILL.md`
- **מוטמע בתוך הקוד** כ-`const PATENT_SKILL` בתוך `index.html`
- כשמעדכנים את `SKILL.md` — צריך לסנכרן ידנית ל-`index.html`

> TODO: ניתן לאחד — לטעון SKILL.md דינמית בעתיד

### שכבה 3 — מומחי תחום (Domain Experts)
- מוגדרים כ-`const DOMAIN_EXPERTS` ב-`index.html`
- כל domain מכיל: icon, label, instructions (נכנסות ל-system prompt)
- הזיהוי: API call נפרד + הצגה למשתמש + אישור

---

## זרימת נתונים

```
קבצי PDF
    │
    ▼ FileReader.readAsDataURL()
base64 strings
    │
    ▼ API Call #1 (domain detection)
    │   model: claude-sonnet-4-20250514
    │   input: application PDF only
    │   output: {domain, ipc, keywords}
    │
    ▼ Modal — אישור משתמש
    │
    ▼ API Call #2 (full analysis)
    │   system: PATENT_SKILL + DOMAIN_EXPERT[domain]
    │   input: OA + application + prior art PDFs
    │   output: JSON {summary, rejections, strategy, response_draft, claim_amendments}
    │
    ▼ parseOutput() → JSON
    │
    ▼ renderAll() → 3 panels
```

---

## הוספת תחום חדש

1. פתח `platform/index.html`
2. מצא `const DOMAIN_EXPERTS = {`
3. הוסף entry חדש:

```javascript
pharma: {
  icon: '💊',
  label: 'פרמה / Pharmaceuticals',
  instructions: `PHARMACEUTICAL DOMAIN CONTEXT:
- Key distinctions: formulations, dosage forms, active ingredients vs. excipients.
- Prior art: exact compound claims vs. genus claims, Markush structures.
- Unexpected technical effects: bioavailability, stability, efficacy data are critical.
- IPC classes: A61K (preparations), A61P (therapeutic activity).
- EP: "same therapeutic effect" standard for obviousness of formulations.`
}
```

4. הפלטפורמה תזהה אוטומטית IPC class A61K → יופעל expert זה

---

## עדכון הסקיל

כשמשנים `skill/SKILL.md`:

1. פתח `platform/index.html`
2. מצא `const PATENT_SKILL = \``
3. החלף את התוכן בין ה-backticks בתוכן החדש של SKILL.md
4. שמור

---

## הוספת Server-Side Proxy (לפריסה בייצור)

לפריסה בסביבת ייצור, מומלץ לא לחשוף API key בצד הלקוח.

**Node.js Express proxy (דוגמה):**
```javascript
// server.js
const express = require('express');
const app = express();
app.use(express.json({ limit: '50mb' }));

app.post('/api/analyze', async (req, res) => {
  const response = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': process.env.ANTHROPIC_API_KEY,
      'anthropic-version': '2023-06-01'
    },
    body: JSON.stringify(req.body)
  });
  const data = await response.json();
  res.json(data);
});

app.listen(3000);
```

ואז בקובץ `index.html` — החלף:
```javascript
// מ:
fetch("https://api.anthropic.com/v1/messages", ...)
// ל:
fetch("/api/analyze", ...)
```

---

## ייצוא ל-Word (Roadmap)

ניתן להוסיף בעתיד ייצוא של `response_draft` ל-Word:

```javascript
// npm: docx
import { Document, Paragraph, TextRun, Packer } from 'docx';

async function exportToWord(draft) {
  const doc = new Document({
    sections: [{
      children: draft.split('\n').map(line =>
        new Paragraph({ children: [new TextRun(line)] })
      )
    }]
  });
  const blob = await Packer.toBlob(doc);
  // download...
}
```
