[khatm.md](https://github.com/user-attachments/files/33906310/khatm.md)
---
title: Hifz Planner – Join a group khatm
permalink: /khatm/
---

# Join a group khatm

**English** · [Français](#rejoindre-un-khatm-en-groupe) · [العربية](#الانضمام-إلى-ختمة-جماعية)

Someone invited you to read the Quran together in **Hifz Planner** ("خطة الحفظ"): each member
takes one or more juz, and the group finishes the khatm together.

<p id="code-line" style="display:none">Group code: <strong id="code"></strong></p>

1. Install Hifz Planner from
   [Google Play](https://play.google.com/store/apps/details?id=com.beyapps.hifzplanner) (free).
2. <a id="open-link" href="hifzplanner://khatm">Open the invitation in the app</a>, or in the app go
   to **Settings > Group khatm** and enter the code.

No sign-up is needed. Members see the group, who took which juz, and the name each member chose.

## Rejoindre un khatm en groupe

On vous invite à lire le Coran ensemble dans **Hifz Planner** : chaque membre prend un ou
plusieurs juz, et le groupe termine le khatm ensemble.

1. Installez Hifz Planner depuis
   [Google Play](https://play.google.com/store/apps/details?id=com.beyapps.hifzplanner) (gratuit).
2. Ouvrez l'invitation dans l'application (lien ci-dessus), ou allez dans **Réglages > Khatm en
   groupe** et saisissez le code.

## الانضمام إلى ختمة جماعية

<div dir="rtl" markdown="1">

دعاك أحدهم إلى قراءة القرآن معًا في تطبيق **خطة الحفظ**: يأخذ كل عضو جزءًا أو أكثر، وتختم المجموعة
معًا.

1. ثبّت التطبيق من
   [Google Play](https://play.google.com/store/apps/details?id=com.beyapps.hifzplanner) (مجانًا).
2. افتح الدعوة في التطبيق (الرابط أعلاه)، أو اذهب إلى **الإعدادات > ختمة جماعية** وأدخل الرمز.

</div>

<script>
  // The code comes from the link (?code=...): shown here and passed to the app.
  var code = (new URLSearchParams(location.search).get('code') || '').replace(/[^A-Za-z0-9]/g, '').toUpperCase();
  if (code) {
    document.getElementById('code').textContent = code;
    document.getElementById('code-line').style.display = 'block';
    document.getElementById('open-link').href = 'hifzplanner://khatm/' + code;
  }
</script>
