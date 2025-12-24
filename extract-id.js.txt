javascript:(function(){
  const nick = prompt("🧠 ألصق هنا النك نيم الذي تريد استخراج الـ ID له:");
  if (!nick || nick.length < 2) {
    alert("❌ النك نيم غير صالح أو فارغ.");
    return;
  }

  const users = document.querySelectorAll('.uzr');
  let found = false;
  let extractedId = "";

  users.forEach(div => {
    const n = div.getAttribute('n');
    if (n && n.includes(nick)) {
      const uidClass = [...div.classList].find(c => c.startsWith('uid'));
      if (uidClass) {
        extractedId = uidClass.slice(3); // إزالة "uid"
        found = true;
      }
    }
  });

  if (!found) {
    alert("⚠ لم يتم العثور على أي عضوية بهذا النك نيم.");
    return;
  }

  // تشغيل اعتراض WebSocket باستخدام الـ ID المستخرج
  const myId = extractedId;
  const originalSend = WebSocket.prototype.send;
  WebSocket.prototype.send = function (...args) {
    const raw = args[0];
    if (typeof raw === "string" && raw.startsWith("42") && raw.includes('"msg"') && raw.includes(myId)) {
      try {
        const parsed = JSON.parse(raw.slice(2));
        const [type, payload] = parsed;
        if (type === "msg" && typeof payload === "object") {
          const data = payload.data || {};
          if (JSON.stringify(data).includes(myId)) {
            window._realSocket_ = this;
            console.log("✅ تم حفظ الاتصال الحقيقي في window._realSocket_");
          }
        }
      } catch (err) {
        console.warn("⚠ فشل في تحليل الرسالة:", raw);
      }
    }
    return originalSend.apply(this, args);
  };

  alert(`✅ تم استخراج الـ ID:\n${myId}\n\n📡 بدأ اعتراض WebSocket...`);
  console.log("📡 بدأ اعتراض WebSocket... بانتظار أول رسالة تحتوي على المعرف:", myId);
})();
