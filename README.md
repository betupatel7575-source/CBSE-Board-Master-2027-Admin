# CBSE Board Master 2027 — Admin Panel

Yeh ek **mobile-friendly Admin Panel** hai jisse aap apne CBSE Board Master 2027 app/website ka content (Questions, Subjects, Chapters, Sample Papers, Mock Tests, Updates) manage kar sakte ho — seedhe apne **Android phone ke browser** se.

---

## 📦 Is ZIP mein kya hai (What's in this ZIP)

```
index.html   → Admin Panel ka structure (pages)
style.css    → Design / look (mobile-friendly, dark mode support)
app.js       → Saara logic — Firebase Login, Firestore CRUD, Search, Filters, CSV Import/Export
README.md    → Yeh file (instructions)
```

Yeh plain HTML/CSS/JS hai — koi build step nahi chahiye. Bas file kholiye aur chal jaata hai.

---

## 🚀 Step 1: Phone par test karna (Testing on your phone)

1. ZIP ko phone mein **extract** kariye (koi bhi File Manager app se).
2. `index.html` file ko **Chrome browser** mein khol dijiye.
3. Aapko Login screen dikhega.
4. Apne Firebase Admin account ke **email aur password** se login kariye — jo account aapne pehle se Firebase Authentication mein banaya hai.

⚠️ Agar login nahi ho raha:
- Check kariye ki internet chal raha hai.
- Check kariye ki email/password sahi hai.
- Agar "permission denied" jaisa error aaye, to Step 3 (Firestore Security Rules) dekhiye — ho sakta hai rules set nahi hue.

---

## 🌐 Step 2: Online host karna (so it works from anywhere, not just your phone's local file)

Sabse aasaan tareeka: **Firebase Hosting** (free hai, aur aapka Firebase project pehle se ready hai).

Phone se karne ke liye best option: pehle in files ko **Firebase Console → Hosting** section mein manually upload karne ka option dhoondiye, ya laptop/cyber cafe se ek baar yeh command chalwa lijiye:

```bash
npm install -g firebase-tools
firebase login
firebase init hosting
# Public directory: is folder ko select karein jisme yeh 4 files hain
firebase deploy
```

Deploy hone ke baad aapko ek link milega jaise:
`https://cbse-board-master-2027-ce8b3.web.app`

Yeh link aap apne phone se bhi khol sakte ho, kabhi bhi.

(Agar aap chahen to yeh ek baar kisi dost/cyber café se karwa sakte ho — uske baad roz ka kaam aap sirf apne phone se hi karoge, kyunki content add/edit karna is Admin Panel ke andar se hi hota hai.)

---

## 🔐 Step 3: Firestore Security Rules (VERY IMPORTANT)

Abhi tak Firestore database **khula (open)** ho sakta hai, jo unsafe hai. Firebase Console → Firestore Database → Rules mein jaakar yeh rules paste kariye:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Students: read-only access to study content
    match /questions/{docId} {
      allow read: if true;
      allow write: if request.auth != null && request.auth.token.admin == true;
    }
    match /subjects/{docId} {
      allow read: if true;
      allow write: if request.auth != null && request.auth.token.admin == true;
    }
    match /chapters/{docId} {
      allow read: if true;
      allow write: if request.auth != null && request.auth.token.admin == true;
    }
    match /samplePapers/{docId} {
      allow read: if true;
      allow write: if request.auth != null && request.auth.token.admin == true;
    }
    match /mockTests/{docId} {
      allow read: if resource.data.published == true;
      allow write: if request.auth != null && request.auth.token.admin == true;
    }
    match /mockQuestions/{docId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.token.admin == true;
    }
    match /updates/{docId} {
      allow read: if true;
      allow write: if request.auth != null && request.auth.token.admin == true;
    }

    // Each user can only read/write their own bookmarks and results
    match /bookmarks/{docId} {
      allow read, write: if request.auth != null && request.auth.uid == resource.data.userId;
      allow create: if request.auth != null && request.auth.uid == request.resource.data.userId;
    }
    match /results/{docId} {
      allow read: if request.auth != null && request.auth.uid == resource.data.userId;
      allow create: if request.auth != null && request.auth.uid == request.resource.data.userId;
    }

    // Users collection: a user can only read/write their own profile
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

### ⚠️ Important: `admin.token.admin == true` ka matlab

Yeh rule tabhi kaam karega jab aapke admin account par **Custom Claim** (`admin: true`) set ho. Yeh cheez **frontend se nahi** ho sakti — iske liye ek chhota **server-side script** (Firebase Admin SDK, Node.js) ek hi baar chalana padta hai, ya Firebase Console ke "Cloud Functions" se. Yeh security ke liye zaroori hai, kyunki koi bhi frontend button ya password sirf "chhupaya" nahi ja sakta — real security backend rules se hoti hai.

**Jab tak aap yeh custom claim set nahi karte**, ek simpler (thoda kam secure, lekin kaam-chalau) temporary rule use kar sakte ho jismein sirf ek specific admin email ko allow kiya jaaye:

```
allow write: if request.auth != null && request.auth.token.email == "aapka-admin-email@example.com";
```

Isse replace `"aapka-admin-email@example.com"` apne actual admin email se. Yeh temporary solution hai — jaise hi possible ho, custom claims (`admin: true`) set karwa lena better hai, kisi developer ki help se.

---

## 🗂️ Firestore Collections (Database Structure)

Admin Panel yeh collections use karta hai:

| Collection      | Kya store hota hai |
|-----------------|---------------------|
| `questions`     | Har question: class, subject, chapter, question, answer, solution, year, marks, questionType, difficulty, repeated, source |
| `subjects`      | Subject list: name, class, stream, order |
| `chapters`      | Chapter list: name, class, subject, order |
| `samplePapers`  | Sample paper details: title, class, subject, timeAllowed, maxMarks, difficulty, instructions, answerKey |
| `mockTests`     | Mock test metadata: title, class, subject, durationMinutes, totalMarks, negativeMarking, published |
| `mockQuestions` | Har mock test ke andar ke actual questions: mockTestId (link), questionNumber, question, optionA-D, correctAnswer, explanation, marks, order |
| `updates`       | Latest Updates: title, category, date, description |
| `users`         | Student accounts (Android app/website banaayega) |
| `bookmarks`     | Student bookmarks (Android app/website banaayega) |
| `results`       | Student mock test results (Android app/website banaayega) |

Yeh Admin Panel `questions`, `subjects`, `chapters`, `samplePapers`, `mockTests`, aur `updates` — in 6 collections ko fully manage karta hai (Add/Edit/Delete/Search/Filter). `users`, `bookmarks`, `results` — yeh student-facing Android app/website banayega jab hum woh part banayenge.

---

## ✅ Admin Panel mein kya-kya kaam karta hai (What actually works)

- ✅ Login / Logout (Firebase Authentication)
- ✅ Dashboard — total questions, class-wise count, repeated count, subjects, chapters, papers, mock tests
- ✅ Questions: Add, Edit, Delete, Search (keyword), Filter (class/subject/chapter/year/marks/type/repeated)
- ✅ CSV Export (filtered questions ko CSV file mein download)
- ✅ CSV Import (bulk questions add karne ke liye — required columns: class, subject, chapter, question)
- ✅ Subjects: Add, Edit, Delete, Order set karna
- ✅ Chapters: Add, Edit, Delete, Subject se link
- ✅ Sample Papers: Add, Edit, Delete (title, marks, time, instructions, answer key)
- ✅ Mock Tests: Add, Edit, Delete, Publish/Unpublish
- ✅ Mock Test Questions: Har mock test kholke "Manage Questions" se uske andar ke actual questions add kar sakte ho — Question, 4 Options (A/B/C/D), Correct Answer, Explanation, Marks, Question Number. Edit, Delete, aur ↑/↓ buttons se order badal sakte ho.
- ✅ Updates: Post karna, Edit, Delete
- ✅ Mobile-responsive design + Dark mode (phone ki system setting follow karta hai)
- ✅ Error messages: login failed, saved, updated, deleted, permission denied, etc.
- ✅ Empty states: "No questions available yet." jaisa

### ⚠️ Abhi kya missing hai (honestly listing what's not built yet)

- Custom admin claims (`admin: true`) set karne ka script is ZIP mein nahi hai — yeh ek baar ka backend setup hai (Step 3 dekhiye).
- Android App aur Website is ZIP mein **nahi** hain — jaisa aapne kaha tha, Admin Panel pehle bana diya hai. Website aur Android App agle steps mein banenge.

---

## 📱 Roz ka istemal (Daily use from your phone)

1. Deployed link (ya local `index.html`) kholiye Chrome mein.
2. Login kariye.
3. Bottom/side menu se **Questions** par jaayiye.
4. "+ Add Question" dabaake naya question daaliye.
5. Same tarah **Subjects**, **Chapters**, **Sample Papers**, **Mock Tests**, **Updates** manage kar sakte ho.
6. Jo bhi aap yahan save karoge, woh seedha Firebase Firestore mein jaata hai — aur wahi data aapki future Website aur Android App mein automatically dikhega, bina app rebuild kiye.

---

## 🔜 Next Steps

1. Firestore Security Rules set kariye (Step 3) — yeh sabse zaroori hai.
2. Admin Panel ko Firebase Hosting par deploy kariye (Step 2) taaki link kahin se bhi khule.
3. Kuch demo questions add karke test kariye — is ZIP mein ek `demo-questions.csv` file bhi hai, use aap "Import CSV" button se seedha import kar sakte ho testing ke liye (3 sample Biology/Science questions).
4. Uske baad hum **Website** aur **Android App** par kaam shuru karenge — jo isi Firebase data ko dikhayenge.

Koi bhi cheez samajh na aaye to bataiye, step-by-step Hindi mein samjha dunga.
