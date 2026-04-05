# Інструкція: Як підключити справжній ШІ до сайту Capingo

Ви успішно додали візуальний віджет **КапіШІ** на ваш сайт! Наразі він працює у "демо-режимі": виклик інтерфейсу спрацьовує, але штучний інтелект завжди відповідає стандартною запланованою фразою.

Щоб "оживити" його, потрібно замінити цю заглушку на реальний запит (HTTP Request) до API (наприклад, OpenAI `gpt-4o-mini` або Google Gemini API).

Через те, що ваш сайт працює як статичний HTML (`index.html`), використовувати секретні ключі (API Keys) **безпосередньо у коді дуже небезпечно**. Їх можуть викрасти. Нижче описані 2 надійні способи підключення.

---

## Спосіб 1: Використання сервера (Бекенду) — Надійний метод 🛡️

Найкращий і найбезпечніший спосіб — створити проміжний сервер (наприклад, за допомогою Node.js/Express, Python/FastAPI, або безкоштовних "безсерверних" функцій типу Vercel Functions / Cloudflare Workers). 

Цей сервер зберігатиме ваш API-ключ в таємниці і відправлятиме запити до ChatGPT замість клієнта.

### Як це інтегрувати в `index.html`:
У вашому файлі `index.html` знайдіть функцію `sendAIMessage()` і замініть "емуляцію" (блок із `setTimeout`) на реальний запит на ваш бекенд:

```javascript
/* ЗАМІНИТИ ДЕМО-БЛОК НА НАСТУПНЕ: */

async function sendAIMessage() {
  const input = document.getElementById('ai-input');
  const text = input.value.trim();
  if(!text) return;
  
  const msgs = document.getElementById('ai-messages');
  
  // Додаємо повідомлення юзера
  const uMsg = document.createElement('div');
  uMsg.className = 'ai-msg ai-user';
  uMsg.textContent = text;
  msgs.appendChild(uMsg);
  
  input.value = '';
  msgs.scrollTop = msgs.scrollHeight;
  
  // Показуємо завантаження
  const sysMsg = document.createElement('div');
  sysMsg.className = 'ai-msg ai-sys';
  sysMsg.innerHTML = '<span class="spinner-dots" style="display:inline-flex;"><span></span><span></span><span></span></span>';
  msgs.appendChild(sysMsg);
  msgs.scrollTop = msgs.scrollHeight;

  // РОБИМО ЗАПИТ НА ВАШ СЕРВЕР 
  try {
    const response = await fetch('https://ВАШ-СЕРВЕР.com/api/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ message: text }) // відправляємо питання юзера
    });

    const data = await response.json();
    
    // Відображаємо відповідь ШІ
    sysMsg.innerHTML = data.reply; // відповідь від вашого сервера
  } catch (error) {
    sysMsg.innerHTML = 'Виникла помилка підключення. Спробуйте пізніше.';
  }
  
  msgs.scrollTop = msgs.scrollHeight;
}
```

---

## Спосіб 2: Пряме використання OpenAI API (Лише для приватного/навчального використання) ⚠️

Якщо ви не хочете створювати сервер, і цей сайт використовується _тільки вами_ або обмеженим колом довірених осіб, ви можете налаштувати запит напряму до OpenAI.
> **Важливо:** Якщо ви опублікуєте цей код у відкритому доступі, ваш API-ключ можуть швидко скопіювати і вичерпати ваші кошти!

Щоб використати цей метод:
1. Зареєструйтесь на [платформі OpenAI](https://platform.openai.com/).
2. Створіть свій секретний **API Key**.
3. Замініть код у `index.html` (функція `sendAIMessage()`) на цей:

```javascript
/* ПРОСТИЙ, АЛЕ МЕНШ БЕЗПЕЧНИЙ СПОСІБ: */

async function sendAIMessage() {
  const input = document.getElementById('ai-input');
  const text = input.value.trim();
  if(!text) return;

  const msgs = document.getElementById('ai-messages');

  // Додаємо повідомлення юзера
  const uMsg = document.createElement('div');
  uMsg.className = 'ai-msg ai-user';
  uMsg.textContent = text;
  msgs.appendChild(uMsg);

  input.value = '';
  msgs.scrollTop = msgs.scrollHeight;

  // Показуємо завантаження
  const sysMsg = document.createElement('div');
  sysMsg.className = 'ai-msg ai-sys';
  sysMsg.innerHTML = '<span class="spinner-dots" style="display:inline-flex;"><span></span><span></span><span></span></span>';
  msgs.appendChild(sysMsg);
  msgs.scrollTop = msgs.scrollHeight;

  const OPENAI_API_KEY = "sk-ВАШ_СЕКРЕТНИЙ_КЛЮЧ_ТУТ"; // Вставте ваш ключ ТУТ

  try {
    const response = await fetch("https://api.openai.com/v1/chat/completions", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${OPENAI_API_KEY}`
      },
      body: JSON.stringify({
        model: "gpt-4o-mini", // Використовуємо найшвидшу і найдешевшу модель
        messages: [
          { role: "system", content: "Ти — КапіШІ, привітний і розумний вчитель англійської мови. Допомагай учням, перекладай слова і пояснюй граматику коротко та зрозуміло." },
          { role: "user", content: text }
        ]
      })
    });

    const data = await response.json();
    
    if (data.choices && data.choices.length > 0) {
      // Змінюємо переноси рядків на <br> для правильного відображення в HTML
      const formattedReply = data.choices[0].message.content.replace(/\n/g, '<br>');
      sysMsg.innerHTML = formattedReply;
    } else {
      sysMsg.innerHTML = 'Виникла непередбачувана помилка OpenAI.';
    }
  } catch (error) {
    sysMsg.innerHTML = 'Помилка з\'єднання з сервером OpenAI. Перевірте ключ або інтернет.';
  }
}
```

### Наступні кроки для покращення:
Коли основний функціонал запрацює, ви також зможете реалізувати **історію чату**, зберігаючи попередні повідомлення у вигляді масиву `[{role: "user", content: "..."}, {role: "assistant", content: "..."}]` і відправляючи їх усі разом з кожним новим запитом. Це дозволить штучному інтелекту розуміти контекст розмови.
