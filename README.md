# Patent OA Analyzer — Project

כלי לניתוח דו"חות בחינה (Office Actions) וניסוח תגובות מקצועיות.

---

## מבנה הפרויקט

```
patent-oa-platform/
│
├── platform/
│   └── index.html          ← הפלטפורמה המלאה (פתח בדפדפן / Claude artifact)
│
├── skill/
│   ├── SKILL.md            ← הסקיל המלא לשימוש ישיר עם Claude
│   └── legal-references.md ← פסיקה, MPEP, EPO Guidelines לפי מדינה
│
├── docs/
│   └── architecture.md     ← הסבר ארכיטקטורה ואיך הכל עובד
│
├── examples/
│   └── (הכנס כאן דוגמאות OA + תשובות לבדיקה)
│
└── README.md               ← זה הקובץ
```

---

## איך להשתמש

### אפשרות 1 — פלטפורמה עצמאית (מומלץ)
1. פתח `platform/index.html` בדפדפן כלשהו (Chrome / Firefox)
2. העלה: דו"ח בחינה (PDF) + בקשת פטנט (PDF)
3. הוסף prior art references (PDF) או Google Patents URLs
4. בחר רשות (EP / US / CN / JP)
5. לחץ "נתח" — הפלטפורמה תזהה תחום, תבקש אישורך, ואז תנתח

> ⚠️ הפלטפורמה קוראת ל-Anthropic API ישירות מהדפדפן.
> נדרש: API key בתוך הקוד (ראה סעיף "הגדרת API Key" למטה).

### אפשרות 2 — סקיל עם Claude ישיר
1. העלה את `skill/SKILL.md` ל-Claude כ-Custom Skill
2. שוחח עם Claude ישירות — הסקיל יופעל אוטומטית כשתדבר על OA / prior art / claims

---

## הגדרת API Key

הפלטפורמה שולחת בקשות ל-`https://api.anthropic.com/v1/messages`.

בגרסה הנוכחית ה-API key מוזרק אוטומטית על ידי Claude.ai כשרץ כ-Artifact.
**אם רץ מחוץ ל-Claude.ai** — צריך להוסיף בקובץ `platform/index.html`:

```javascript
// מצא את כל fetch("https://api.anthropic.com/v1/messages", ...
// והוסף header:
headers: {
  "Content-Type": "application/json",
  "x-api-key": "sk-ant-YOUR-KEY-HERE",        // ← הוסף זאת
  "anthropic-version": "2023-06-01",
  "anthropic-dangerous-direct-browser-access": "true"  // ← נדרש לדפדפן
}
```

> 🔒 לעולם אל תשמור API key בקוד שמועלה ל-git ציבורי.
> השתמש ב-`.env` + server-side proxy לפריסה בייצור.

---

## ארכיטקטורה — תהליך הניתוח

```
שלב 1: העלאת חומרים
  ├── OA (PDF)
  ├── Patent application (PDF)
  ├── Prior art references (PDF / URLs)
  └── בחירת jurisdiction (EP/US/CN/JP)

שלב 2: זיהוי תחום (API call קטן)
  ├── קורא עמוד ראשון של הבקשה
  ├── מחזיר: domain + IPC class + keywords
  └── מציג למשתמש → אישור / תיקון ידני

שלב 3: ניתוח מלא (API call ראשי)
  ├── System prompt = SKILL.md + domain expert instructions
  ├── Documents = כל הקבצים שהועלו
  └── מחזיר JSON מובנה

שלב 4: Output — 3 לשוניות
  ├── ניתוח בעברית (סיכום, ניתוח per rejection, אסטרטגיה)
  ├── טיוטת תשובה (אנגלית, בסגנון המשתמש)
  └── הצעות לתיקון תביעות (diff ויזואלי + support)
```

---

## מומחי תחום מובנים

| תחום | Domain Key | IPC אופייני |
|------|-----------|------------|
| מכנית | `mechanical` | F, B (non-printing) |
| הדפסה | `printing` | B41 |
| ביו-רפואי | `biomedical` | A61, G01N |
| תוכנה | `software` | G06, H04 |
| כימיה | `chemistry` | C07, C08, C09 |
| אלקטרוניקה | `electronics` | H01, H02, H03 |
| כללי | `general` | כל שאר |

להוספת תחום חדש — ראה `docs/architecture.md`.

---

## Roadmap — שיפורים עתידיים

- [ ] Server-side proxy לניהול בטוח של API key
- [ ] שמירת היסטוריית ניתוחים (localStorage / DB)
- [ ] ייצוא תשובה ל-Word (.docx)
- [ ] תמיכה ב-CN / JP — תרגום אוטומטי של OA
- [ ] הוספת תחומים: nanotechnology, telecommunications
- [ ] מצב "השוואה" — OA מול תשובה קודמת

---

## קרדיטים

נבנה בשיחה עם Claude (Anthropic) · February 2026
