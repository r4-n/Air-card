# AirCard 1.3.4 — بناء وتجربة iOS 27.0.1

## ما الذي تغيّر

هذه النسخة مبنية فوق AirCard 1.3.3 الذي يستبدل تسلسل AirTraffic القديم
بـ `RequestingSync` يحتوي بيانات Grappa، ويجمع ملفات البطاقة في جلسة Books
واحدة. أضفنا إصلاحًا مهمًا في استدعاء Windows:

```text
ATHostConnectionCreateWithLibrary(UDID, LibraryID, 0)
```

كان الاستدعاء يمرر `LibraryID` ثم `UDID`. لأن النوعين متطابقان، لا يحصل crash
ولا يظهر خطأ اتصال واضح، لكن يمكن أن تتوقف الجلسة بعد `SyncAllowed` بلا
`ReadyForSync`.

تتحقق هذه النسخة أيضًا من نتيجة `HostInfo` و `RequestingSync` و
`MetadataSyncFinished`. إذا رفضت مكتبة Apple الطلب، ستظهر المرحلة والرمز
مباشرة بدل انتظار timeout.

## بناء الملف التنفيذي

1. ثبّت Rust المستقر من https://rustup.rs، واختر toolchain الافتراضي
   `stable-x86_64-pc-windows-msvc`.
2. ثبّت **Visual Studio Build Tools 2022** واختر workload **Desktop development
   with C++** مع Windows SDK. هذا مطلوب لربط ملف EXE على Windows.
3. أغلق PowerShell وافتحه من جديد. من داخل مجلد هذا المشروع شغّل:

   ```powershell
   cargo test --locked
   cargo build --release --locked
   ```

4. سيكون البرنامج هنا:

   ```text
   target\release\aircard.exe
   ```

البرنامج يحتاج Apple Mobile Device Support من iTunes أو Apple Devices. لا
تنسخ DLLs الخاصة بـ Apple إلى مجلد البرنامج.

## تجربة آمنة أولًا

1. أوصل آيفون واحد فقط بكيبل USB، وافتحه ووافق على Trust.
2. افتح Books مرة واحدة، ثم أغلق iTunes وApple Devices وأي برنامج يدير الآيفون.
3. شغّل أولًا:

   ```powershell
   .\target\release\aircard.exe --probe
   ```

4. يجب أن تنتهي برسالة `probe DONE — ReadyForSync received`. هذا لا يكتب أي
   ملف في Wallet.
5. فقط بعد نجاح Probe، افتح البرنامج واستخدم Scan ثم Save original قبل Apply.
   بعد Apply أغلق Wallet بالقوة ثم افتحه من جديد.

إذا ظهر رفض في `RequestingSync` أو لم يصل `ReadyForSync`، لا تجرّب Apply
مرات متتالية. احفظ سجل Probe مع موديل الآيفون وإصدار Apple Mobile Device
Support؛ فالمشكلة حينها توافق DLL/النظام وليس ملف الصورة.

## ملاحظة مهمة

هذه أداة غير رسمية وقد تعيد Apple بناء واجهة البطاقة بعد تحديث iOS. احتفظ
بالنسخة الأصلية باستخدام Save original قبل أول تغيير. لا يمكن ضمان دعم
iOS 27.0.1 بلا اختبار فعلي على آيفون بهذا الإصدار؛ نجاح Probe هو بوابة
السلامة قبل كتابة أي أصل للبطاقة.
